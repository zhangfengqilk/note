#  SpringBoot使用Delete方式接收请求问题总结

用惯了SpringMVC。SpringMVC中对PUT DELETE方式不支持，要加入HiddenHttpMethodFilter，在前端将PUT DELETE请求改为type:"POST",data:{_method:"DELETE}才可以接收（详见本博客另一篇博文）。

本以为SpringBoot继承了SpringMVC，一顿乱搞，碰了些壁，在这里和大家分享一下：



- 1.SpringBoot已经默认引入了HiddenHttpMethodFilter，可在SpringBoot启动日志里看到该Filter的启动信息。

- 2.Delete方式有Entity Body，但是该方法传递Entity Body没有明确定义的语义，所以有些服务器实现会丢弃/忽略DELETE请求的entity body，或者拒绝该请求。所以Delete请求中，不要把数据放到Entity Body中。

- 3.非常坑的一点是，Get没有Entity Body，在ajax中写入data来传参，会直接把参数拼接到url后面。Delete方式却不可以。所以使用DELETE方式请求时，要手动将参数拼接到url中，而非写到data中。

- 4.有个事情没搞明白，POST请求中，如果我把参数同时写到url和entity body中，后台程序会识别哪个？或优先识别哪个？我用restclient测试时，发现识别url后的参数而非entity body中的，显然是有问题的。所以想跟大家一起讨论下。