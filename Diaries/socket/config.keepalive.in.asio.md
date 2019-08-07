

You can configure both the send timeout and receive timeout using the setsockopt options. Here is some platform-dependent code to do this on both windows and linux / unix, the example sets both send and receive timeouts to the same ten second value:
```
// start IO service    
io_context = new boost::asio::io_context;        

// define a tcp socket object    
tcpsocket = new boost::asio::ip::tcp::socket(*io_context); 

// the timeout value
unsigned int timeout_milli = 10000;

// platform-specific switch
#if defined _WIN32 || defined WIN32 || defined OS_WIN64 || defined _WIN64 || defined WIN64 || defined WINNT
  // use windows-specific time
  int32_t timeout = timeout_milli;
  setsockopt(tcpsocket->native_handle(), SOL_SOCKET, SO_RCVTIMEO, (const char*)&timeout, sizeof(timeout));
  setsockopt(tcpsocket->native_handle(), SOL_SOCKET, SO_SNDTIMEO, (const char*)&timeout, sizeof(timeout));
#else
  // assume everything else is posix
  struct timeval tv;
  tv.tv_sec  = timeout_milli / 1000;
  tv.tv_usec = (timeout_milli % 1000) * 1000;
  setsockopt(tcpsocket->native_handle(), SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv));
  setsockopt(tcpsocket->native_handle(), SOL_SOCKET, SO_SNDTIMEO, &tv, sizeof(tv));
#endif
```


