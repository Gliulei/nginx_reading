####nginx源码分析一
#####main函数关键启动分析
**1、解析命令行参数,显示帮助信息**
```
if (ngx_get_options(argc, argv) != NGX_OK) {
        return 1;
    }
```
**2、初始化操作系统调用接口函数**
```
 if (ngx_os_init(log) != NGX_OK) {
        return 1;
    }
```
**3、根据命令行参数等建立一个基本的cycle**
```
 ngx_memzero(&init_cycle, sizeof(ngx_cycle_t));
 init_cycle.log = log;
 ngx_cycle = &init_cycle;
```
**4、初始化静态模块数组ngx_modules**
```
if (ngx_preinit_modules() != NGX_OK) {
        return 1;
    }
 ```   
**5、核心操作，调用ngx_init_cycle创建进程使用的cycle,解析配置文件,启动监听端口**
```
cycle = ngx_init_cycle(&init_cycle);
```
**6、启动单进程或多进程**
```
 if (ngx_process == NGX_PROCESS_SINGLE) {
        ngx_single_process_cycle(cycle);
    } else {
        ngx_master_process_cycle(cycle);
    }
```