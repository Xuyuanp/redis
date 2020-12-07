# My notes of reading source code of redis

## Server

We ignored sential mode here

* [main()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L5136)
    * initialize locale, time, random seed, hash function seed,
    * [initServerConfig()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L2338)
        * [populateCommandTable()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3109)
            * TODO
    * aclInit
    * [moduleInitModulesSystem()](https://github.com/Xuyuanp/redis/blob/6.0/src/module.c#L7508)
        * [moduleRegisterCoreAPI()](https://github.com/Xuyuanp/redis/blob/6.0/src/module.c#L8066)
            * register all APIs with macro [REGISTER_API](https://github.com/Xuyuanp/redis/blob/6.0/src/module.c#L7502)
        * init server.module_blocked_pipe
        * create radix timer
        * lock GIL mutex
    * tlsInit
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
                    * [createClient()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L100)
                        * connNonBlock
                        * connEnableTcpNoDelay
                        * keepAlive
                        * [connSetReadHandler()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.h#L165) with callback
                            * [conn->type->set_read_handler => CT_SOCKET.set_read_handler => connSocketSetReadHandler()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.h#L239)
                                * if handler is NULL, remote read handler from event loop
                                * aeCreateFieEvent with [ae_handler => CT_SOCKET.ae_hander => connSocketEventHandler()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.h#L255)
                                    * call conn_handler and set it to NULL if state is CONN_STATE_CONNECTING and mask is AE_WRITEABLE and conn_handler is not NULL
                                    * call read_handler if AE_READABLE
                                    * call write_handler if AE_WRITABLE
                            * callback [readQueryFromClient](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L1956)
                                * return if [postponeClientRead()]()
                                    * TODO
                                * nread = [connRead()]()
                                    * [conn->type->read => CT_SOCKET.read => connSocketRead](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L182)
                                        * `read`
                                * if nread == 0 then client closed, we free client here and return;
                                * [processInputBuffer](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L1827)
                                    * some cluster or bulk commands checking
                                    * [processCommandAndResetClient()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L1940)
                                        * [processCommand()](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3540)
                                            * [lookupCommand](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3185)
                                                * lookup command from server.commands which defined in [populateCommandTable](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3109)
                                                * we use `setCommand` and `getCommand` as examples here.
                                                    * [setCommand](https://github.com/Xuyuanp/redis/blob/6.0/src/t_string.c#L97)
                                                        * parse args
                                                        * [setGenericCommand()](https://github.com/Xuyuanp/redis/blob/6.0/src/t_string.c#L68)
                                                            * [genericSetKey](https://github.com/Xuyuanp/redis/blob/6.0/src/db.c#L244)
                                                                * if key exists
                                                                    * [dbAdd](https://github.com/Xuyuanp/redis/blob/6.0/src/db.c#L179)
                                                                        * dictAdd db->dict
                                                                * else
                                                                    * [dbOverwrite](https://github.com/Xuyuanp/redis/blob/6.0/src/db.c#L214)
                                                                        * free old val and set new val
                                                            * [addReply](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L322)
                                                                * [prepareClientToWrite()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L233)
                                                                    * [clientInstallWriteHandler()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L192) if client has no pending data to write
                                                                        * set CLIENT_PENDING_WRITE to c.flags
                                                                        * add client to server.clients_pending_write
                                                                        * server.clients_pending_write is used by [handleClientsWithPendingWrites]() or [handleClientsWithPendingWritesUsingThreads]() which is called in [beforeSleep](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L2110) handler
                                                                * [_AddReplyToBuffer()](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L263)
                                            * [call](https://github.com/Xuyuanp/redis/blob/6.0/src/server.c#L3327)
                                                * c->cmd->proc(c)
                                        * [commandProcessed()]()
                                            * TODO
                        * [linkClient()]()
                            * TODO
                        * [initClientMultiState()]()
                            * TODO
                    * [connAccept()](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.h#L104)
                        * [type->accept => CT_SOCKET.accept => connSocketAccept](https://github.com/Xuyuanp/redis/blob/6.0/src/connection.c#L199)
                            * state = CONN_STATE_CONNECTED
                            * call handler (clientAcceptHandler)
                        * callback [clientAcceptHandler](https://github.com/Xuyuanp/redis/blob/6.0/src/networking.c#L868)
                            * assert connection state (connected)
                            * [moduleFireServerEvent()](https://github.com/Xuyuanp/redis/blob/6.0/src/module.c#L7352)
                                * find event listener by eid, construct context and module data
                                * call listener's callback with context, subid and module data
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
