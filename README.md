# My notes of reading source code of redis

## Server

We ignored sential mode here

* [main()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L5136)
    * initialize locale, time, random seed, hash function seed,
    * [initServerConfig()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L2338)
        * [populateCommandTable()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3109)
            * TODO
    * aclInit, module system, tls
    * parse args and configurations
    * print banner
    * [initServer()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L2820)
        * setup signals handler
        * fill `server` struct with default value
        * tls config (if needed)
        * create shared objects
        * adjust open files limit
        * [aeCreateEventLoop()](https://github.com/Xuyuanp/redis/blob/6.0/src/ae.c#L63)
            * [aeApiCreate()](https://github.com/Xuyuanp/redis/blob/6.0/src/ae_epoll.c#L39)
                * `epoll_create`
        * alloc db
        * [listenToPort()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L2706) if needed
            * [anetTcp(6)Server()](https://github.com/Xuyuanp/redis/blob/6.0/src/anet.c#L517)
                * [_anetTcpServer()](https://github.com/Xuyuanp/redis/blob/6.0/src/anet.c#L479)
                    * `socket`
                    * anetSetReuseAddr
                        * `setsockopt` SO_REUSEADDR
                    * [anetListen()](https://github.com/Xuyuanp/redis/blob/6.0/src/anet.c#L454)
                        * `bind`
                        * `listen`
            * anetNonBlock()
                * `fcntl` O_NONBLOCK
        * listen to unix socket if needed
        * create databases, and initialize other internal state
        * create timers
        * create event handlers ([aeCreateFileEvent()](https://github.com/Xuyuanp/redis/blob/6.0/src/ae.c#L153) AE_READABLE acceptTcpHandler)
            * [aeApiAddEvent()](https://github.com/Xuyuanp/redis/blob/6.0/src/ae_epoll.c#L73)
                * `epoll_ctl`
            * callback: [acceptTcpHandler](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L1001)
                * [anetTcpAccept()](https://github.com/Xuyuanp/redis/blob/6.0/src/anet.c#L562)
                    * [anetGenericAccept()](https://github.com/Xuyuanp/redis/blob/6.0/src/anet.c#L545)
                        * `accept`
                * [connCreateAcceptedSocket()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L95)
                    * [connCreateSocket()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L77)
                        * initialized with type [CT_SOCKET](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L349)
                    * conn.state = CONN_STATE_ACCEPTING
                * [acceptCommonHandler()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L929)
                    * assert connection state
                    * assert total connection count
                    * [createClient()]()
                        * TODO
                    * [connAccept()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.h#L104)
                        * [type->accept => CT_SOCKET.accept => connSocketAccept](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L199)
                            * state = CONN_STATE_CONNECTED
                            * call handler (clientAcceptHandler)
                        * callback [clientAcceptHandler](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L868)
                            * assert connection state (connected)
                            * [moduleFireServerEvent()](https://github.com/Xuyuanp/redis/blob/6.0/src/module.c#L7352)
                                * TODO
        * register before and after sleep handlers
        * open append-only-file (if needed)
    * module load from queue
    * [initServerLast()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3037)
        * bioInit
        * initThreadedIO
        * set_jemalloc_bg_thread
    * load data from disk
    * aeMain (blocking here)
    * aeDeleteEventLoop
