#### nginx -s reload 执行过程

执行./nginx -s reload 向master进程发送重启信号

**1、nginx程序解析命令行参数**

```
ngx_get_options(argc, argv)
```

此函数进去可以看到

ngx_signal这个静态变量会被赋值reload

即：ngx_signal="reload"

**2、如果用了-s参数，那么就要发送reload/stop等信号，然后结束**

```
 if (ngx_signal) {
    return ngx_signal_process(cycle, ngx_signal);
}
```

接着在调用它ngx_os_signal_process(cycle, sig, pid) 可以看到最后调用kill(pid, sig->signo)向
master进程的pid发送信号

**3、信号捕获**
之前nginx在启动的时候会调用ngx_init_signals(cycle->log)这个函数给信号绑定这个处理器ngx_signal_handler。
当调用kill的时候会触发ngx_signal_handler使ngx_reconfigure = 1

**4、创建新的work进程 cache进程**
从这个函数ngx_master_process_cycle(cycle)最后可以看到

```
 // 收到了-s reload重新配置
        if (ngx_reconfigure) {
            ngx_reconfigure = 0;

            if (ngx_new_binary) {
                ngx_start_worker_processes(cycle, ccf->worker_processes,
                                           NGX_PROCESS_RESPAWN);
                ngx_start_cache_manager_processes(cycle, 0);
                ngx_noaccepting = 0;

                continue;
            }

            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "reconfiguring");
            cycle = ngx_init_cycle(cycle);
            if (cycle == NULL) {
                cycle = (ngx_cycle_t *) ngx_cycle;
                continue;
            }
            ngx_cycle = cycle;
            ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx,
                                                   ngx_core_module);
            ngx_start_worker_processes(cycle, ccf->worker_processes,
                                       NGX_PROCESS_JUST_RESPAWN);
            ngx_start_cache_manager_processes(cycle, 1);

            /* allow new processes to start */
            ngx_msleep(100);
            live = 1;
            ngx_signal_worker_processes(cycle,
                                       		 ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
        }
```

nginx开始进行创建新的进程关闭老的进程

