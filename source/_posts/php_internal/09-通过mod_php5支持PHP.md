---
title: 09-通过mod_php5支持PHP
tags: php_internal
categories: php_internal
date: 2018-01-01 20:07:09
updated: 2018-01-01 20:07:09
---

# 09-通过mod_php5支持PHP
Apache对PHP的支持是通过Apache的模块mod_php5来支持的。如果希望Apache支持PHP的话，在./configure步骤需要指定--with-apxs2=/usr/local/apache2/bin/apxs 表示告诉编译器通过Apache的mod_php5/apxs来提供对PHP5的解析。

在最后一步make install的时候我们会看到将动态链接库libphp5.so(Apache模块)拷贝到apache2的安装目录的modules目录下，并且还需要在httpd.conf配置文件中添加LoadModule语句来动态将libphp5.so 模块加载进来，从而实现Apache对php的支持。

由于该模式实在太经典了，因此这里关于安装部分不准备详述了，相对来说比较简单。我们知道nginx一般包括两个用途HTTP Server和Reverse Proxy Server(反向代理服务器)。在前端可以部署nginx作为reverse proxy server，后端布置多个Apache来实现机群系统server cluster架构的。

因此，实际生产中，我们仍旧能够保留Apache+mod_php5的经典App Server，而仅仅使用nginx来当做前端的reverse proxy server来实现代理和负载均衡。 因此，建议nginx(1个或者多个)+多个apache的架构继续使用下去。

Apache2的mod_php5模块包括sapi/apache2handler和sapi/apache2filter两个目录 在apache2_handle/mod_php5.c文件中，模块定义的相关代码如下：

    AP_MODULE_DECLARE_DATA module php5_module = {
        STANDARD20_MODULE_STUFF,
            /* 宏，包括版本，小版本，模块索引，模块名，下一个模块指针等信息，其中模块名以__FILE__体现 */
        create_php_config,      /* create per-directory config structure */
        merge_php_config,       /* merge per-directory config structures */
        NULL,                   /* create per-server config structure */
        NULL,                   /* merge per-server config structures */
        php_dir_cmds,           /* 模块定义的所有的指令 */
        php_ap2_register_hook
            /* 注册钩子，此函数通过ap_hoo_开头的函数在一次请求处理过程中对于指定的步骤注册钩子 */
    };

它所对应的是Apache的module结构，module的结构定义如下：

    typedef struct module_struct module;
    struct module_struct {
        int version;
        int minor_version;
        int module_index;
        const char *name;
        void *dynamic_load_handle;
        struct module_struct *next;
        unsigned long magic;
        void (*rewrite_args) (process_rec *process);
        void *(*create_dir_config) (apr_pool_t *p, char *dir);
        void *(*merge_dir_config) (apr_pool_t *p, void *base_conf, void *new_conf);
        void *(*create_server_config) (apr_pool_t *p, server_rec *s);
        void *(*merge_server_config) (apr_pool_t *p, void *base_conf, void *new_conf);
        const command_rec *cmds;
        void (*register_hooks) (apr_pool_t *p);
    }

上面的模块结构与我们在mod_php5.c中所看到的结构有一点不同，这是由于STANDARD20_MODULE_STUFF的原因， 这个宏它包含了前面8个字段的定义。STANDARD20_MODULE_STUFF宏的定义如下：

    /** Use this in all standard modules */
    #define STANDARD20_MODULE_STUFF MODULE_MAGIC_NUMBER_MAJOR, \
                    MODULE_MAGIC_NUMBER_MINOR, \
                    -1, \
                    __FILE__, \
                    NULL, \
                    NULL, \
                    MODULE_MAGIC_COOKIE, \
                                    NULL      /* rewrite args spot */

在php5_module定义的结构中，php_dir_cmds是模块定义的所有的指令集合，其定义的内容如下：

    const command_rec php_dir_cmds[] =
    {
        AP_INIT_TAKE2("php_value", php_apache_value_handler, NULL,
            OR_OPTIONS, "PHP Value Modifier"),
        AP_INIT_TAKE2("php_flag", php_apache_flag_handler, NULL,
            OR_OPTIONS, "PHP Flag Modifier"),
        AP_INIT_TAKE2("php_admin_value", php_apache_admin_value_handler,
            NULL, ACCESS_CONF|RSRC_CONF, "PHP Value Modifier (Admin)"),
        AP_INIT_TAKE2("php_admin_flag", php_apache_admin_flag_handler,
            NULL, ACCESS_CONF|RSRC_CONF, "PHP Flag Modifier (Admin)"),
        AP_INIT_TAKE1("PHPINIDir", php_apache_phpini_set, NULL,
            RSRC_CONF, "Directory containing the php.ini file"),
        {NULL}
    };

这是mod_php5模块定义的指令表。它实际上是一个command_rec结构的数组。 当Apache遇到指令的时候将逐一遍历各个模块中的指令表，查找是否有哪个模块能够处理该指令， 如果找到，则调用相应的处理函数，如果所有指令表中的模块都不能处理该指令，那么将报错。 如上可见，mod_php5模块仅提供php_value等5个指令。

php_ap2_register_hook函数的定义如下：

    void php_ap2_register_hook(apr_pool_t *p)
    {
        ap_hook_pre_config(php_pre_config, NULL, NULL, APR_HOOK_MIDDLE);
        ap_hook_post_config(php_apache_server_startup, NULL, NULL, APR_HOOK_MIDDLE);
        ap_hook_handler(php_handler, NULL, NULL, APR_HOOK_MIDDLE);
        ap_hook_child_init(php_apache_child_init, NULL, NULL, APR_HOOK_MIDDLE);
    }

以上代码声明了pre_config，post_config，handler和child_init 4个挂钩以及对应的处理函数。 其中pre_config，post_config，child_init是启动挂钩，它们在服务器启动时调用。 handler挂钩是请求挂钩，它在服务器处理请求时调用。其中在post_config挂钩中启动php。 它通过php_apache_server_startup函数实现。php_apache_server_startup函数通过调用sapi_startup启动sapi， 并通过调用php_apache2_startup来注册sapi module struct（此结构在本节开头中有说明）， 最后调用php_module_startup来初始化PHP， 其中又会初始化ZEND引擎，以及填充zend_module_struct中 的treat_data成员(通过php_startup_sapi_content_types)等。

到这里，我们知道了Apache加载mod_php5模块的整个过程，可是这个过程与我们的SAPI有什么关系呢？ mod_php5也定义了属于Apache的sapi_module_struct结构：

    static sapi_module_struct apache2_sapi_module = {
    "apache2handler",
    "Apache 2.0 Handler",

    php_apache2_startup,                /* startup */
    php_module_shutdown_wrapper,            /* shutdown */

    NULL,                       /* activate */
    NULL,                       /* deactivate */

    php_apache_sapi_ub_write,           /* unbuffered write */
    php_apache_sapi_flush,              /* flush */
    php_apache_sapi_get_stat,           /* get uid */
    php_apache_sapi_getenv,             /* getenv */

    php_error,                  /* error handler */

    php_apache_sapi_header_handler,         /* header handler */
    php_apache_sapi_send_headers,           /* send headers handler */
    NULL,                       /* send header handler */

    php_apache_sapi_read_post,          /* read POST data */
    php_apache_sapi_read_cookies,           /* read Cookies */

    php_apache_sapi_register_variables,
    php_apache_sapi_log_message,            /* Log message */
    php_apache_sapi_get_request_time,       /* Request Time */
    NULL,                       /* Child Terminate */

    STANDARD_SAPI_MODULE_PROPERTIES
    };

这些方法都专属于Apache服务器。以读取cookie为例，当我们在Apache服务器环境下，在PHP中调用读取Cookie时， 最终获取的数据的位置是在激活SAPI时。它所调用的方法是read_cookies。

    SG(request_info).cookie_data = sapi_module.read_cookies(TSRMLS_C);

对于每一个服务器在加载时，我们都指定了sapi_module，而Apache的sapi_module是apache2_sapi_module。 其中对应read_cookies方法的是php_apache_sapi_read_cookies函数。 这也是定义SAPI结构的理由：统一接口，面向接口的编程，具有更好的扩展性和适应性。
