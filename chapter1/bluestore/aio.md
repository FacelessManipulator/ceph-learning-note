## OS/bluestore/aio

AIO指Asyncronic IO，异步IO，即提交IO请求后立即返回，IO请求完成后由系统中断等方式来执行回调。目前有三种AIO的方式

* 内核系统调用
* 用户空间实现上层，但在底层使用系统调用\(libaio.h\)
* 在用户空间完全模拟异步IO

BlueStore使用了libaio作为异步IO机制，在aio.h中主要定义了俩个数据结构，aio\_t以及aioqueue\_t

aio\_t是对iocb\(io controll block，用来把IO操作的基本信息传输给syscall\)的包装，代码如下:

```
# include <libaio.h>

struct aio_t {
  struct iocb iocb;  // must be first element; see shenanigans in aio_queue_t
  void *priv;
  int fd;
  boost::container::small_vector<iovec,4> iov;
  uint64_t offset, length;
  int rval;
  bufferlist bl;  ///< write payload (so that it remains stable for duration)

  boost::intrusive::list_member_hook<> queue_item;

  aio_t(void *p, int f) : priv(p), fd(f), offset(0), length(0), rval(-1000);
  void pwritev(uint64_t _offset, uint64_t len);
  void pread(uint64_t _offset, uint64_t len);
  int get_return_value();
};
```

* iocb是AIO控制体，用来向系统内核传递IO操作参数
* fd是文件描述符，在初始化时赋值，一般为待操作的文件
* iov在pwritev中使用，是基于Scatter IO的思想，将多个buffer中的数据存储到一个文件中。small\_vector是boost对vector的包装，与vector不同的是，smallvector会预申请内存\(第二个模板参数N\)来减少allocate的开销，通常适用于数量较少的vector。
* offset指文件的偏移量，length指内存的长度
* rval用来存储文件异步IO返回结果
* bufferlist bl是异步IO读取操作的buffer列表，异步IO读请求构建以后，buffer地址将会被添加到bl中
* pwritev构造iocb，请求为把iovector中的多个buffer写入fd中，函数返回后请求既未执行也未提交给内核
* pread构造iocb，请求为把fd中的数据读入buffer中，并将buffer加入bufferlist等待IO
* get\_return\_value直接返回rval，函数不保证IO已经完成，所以rval可能仍为-1000（初始值）

```
struct aio_queue_t {
  int max_iodepth;
  io_context_t ctx;

  typedef list<aio_t>::iterator aio_iter;

  explicit aio_queue_t(unsigned max_iodepth)
    : max_iodepth(max_iodepth),
      ctx(0) {
  }
  ~aio_queue_t() {
    assert(ctx == 0);
  }

  int init();
  void shutdown();

  int submit_batch(aio_iter begin, aio_iter end, uint16_t aios_size, 
           void *priv, int *retries);
  int get_next_completed(int timeout_ms, aio_t **paio, int max);
};
```

* max\_iodepth是调用setup时需要的参数，含义是在可以同时在context中驻留的请求的最大数量
* ctx存储调用io\_setup时产生的IO Context\(异步调用实体\)，用来区分每次IO结果，其实质是一个unsigned long
* init函数执行io\_setup函数初始化ctx, shutdown销毁ctx
* submit\_batch是真正将构造好的请求提交给内核执行的函数，该函数前半部分将iocb从aio列表中提取出来，后半部分内核系统调用io submit不断提交iocb请求。如果失败将在一段delay后再次尝试submit，最多尝试16次，delay每次加倍，最大时延16s
* 


