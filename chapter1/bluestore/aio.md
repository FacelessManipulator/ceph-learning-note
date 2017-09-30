## OS/bluestore/aio

AIO指Asyncronic IO，异步IO，即提交IO请求后立即返回，IO请求完成后由系统中断等方式来执行回调。目前有三种AIO的方式

* 内核系统调用
* 用户空间实现上层，但在底层使用系统调用\(libaio.h\)
* 在用户空间完全模拟异步IO

BlueStore使用了libaio作为异步IO机制，在aio.h中主要定义了俩个数据结构，aio\_t以及aioqueue\_t

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

  aio_t(void *p, int f) : priv(p), fd(f), offset(0), length(0), rval(-1000) {
  }

  void pwritev(uint64_t _offset, uint64_t len) {
    offset = _offset;
    length = len;
    io_prep_pwritev(&iocb, fd, &iov[0], iov.size(), offset);
  }
  void pread(uint64_t _offset, uint64_t len) {
    offset = _offset;
    length = len;
    bufferptr p = buffer::create_page_aligned(length);
    io_prep_pread(&iocb, fd, p.c_str(), length, offset);
    bl.append(std::move(p));
  }

  int get_return_value() {
    return rval;
  }
};
```



