# [springboot中的@DeleteMapping注解无法获取参数值](https://segmentfault.com/q/1010000011211490)

[springboot](https://segmentfault.com/t/springboot)

> 最近在试试使用springboot编写一个demo进行测试，测试过程中发现@DeleteMapping注解有一些问题，现在汇总如下，有大神指点一下

示例1：
问题：无法获取参数id的值

```
    @DeleteMapping(value = "userinfo")
    public void deleteUserinfo(Integer id) {
        System.out.println("========= id : " + id);
        this.dao.delete(id);
    }
```

在spring-mvc中，经常使用上面的方法获取参数，无论是get还是post方法都可以获取的到，但是在springboot中这种写法得到的id却是null，然后方法就抛出异常，因为delete方法的参数值不能为null；

为什么这种方式获取不到参数值呢？

示例2：
问题：无法执行方法，无法获取参数值

```
    @DeleteMapping(value = "userinfo3")
    public void deleteUserinfo3(@RequestParam("id") int id) {
        System.out.println("========= id : " + id);
        this.dao.delete(id);
    }
```

在postman中进行的测试，无论是 form-data还是x-www-form-unlencoded类型，都无法进入方法体，连第一句打印都不执行，直接报**400**的错误，错误信息如下

```
{
  "timestamp": 1505653583069,
  "status": 400,
  "error": "Bad Request",
  "exception": "org.springframework.web.bind.MissingServletRequestParameterException",
  "message": "Required int parameter 'id' is not present",
  "path": "/userinfo/userinfo3"
}
```

很不理解，put方法可以通过修改x-www-form-unlencoded方式，然后通过@RequestParam方法获取参数值，但是delee却不行，不知为何

示例3：
问题：获取不到参数值

```
    @DeleteMapping(value = "userinfo4")
    public void deleteUserinfo4(Userinfo userinfo) {
        System.out.println(userinfo);
        this.dao.delete(userinfo);
    }
```

在post方法中可以使用entity来接受参数，但是delete方法却不行；
上面方法虽然可以执行到方法里面，第一行打印也有内容，但是userinfo对象是空的，没有获取到任何参数，不知为何！
后台日志如下：

```
{"age":0,"id":0}
Hibernate: insert into userinfo (age, cup_size, name) values (?, ?, ?)
Hibernate: delete from userinfo where id=?
```

> 极其匪夷所思，我只是执行了一个delete操作，为什么日志会打印insert into 语句呢？

示例4：
问题 : 无法获取参数值，方法直接进不到方法体中

```
    @DeleteMapping(value = "userinfoMap")
    public void deleteUserinfoMap(@RequestBody Map<String, String> map) {
        System.out.println(map);
    }
```

示例5：

```
    @DeleteMapping(value = "userinfo/{id}")
    public void deleteUserinfo2(@PathVariable("id") int id) {
        System.out.println("========= id : " + id);
        this.dao.delete(id);
    }
```

上面的5个例子，只有这种情况下，通过restful的方法才能获取deletemapping的参数值，实在是费解。

另外的一个问题 :
在示例5中，delete操作竟然不能执行2次，当第二次执行的时候，由于数据已经被删除，导致程序直接抛出异常，错误信息如下

```
{
  "timestamp": 1505654465700,
  "status": 500,
  "error": "Internal Server Error",
  "exception": "org.springframework.dao.EmptyResultDataAccessException",
  "message": "No class com.zzg.springboot.firstbootweb.entity.Userinfo entity with id 2 exists!",
  "path": "/userinfo/userinfo/2"
}
```

阅读 19.1k

 赞 1踩

 收藏 2关注 7

[评论](javascript:;) 更新于 2017-09-17



![img](https://sponsor.segmentfault.com/lg.php?bannerid=51&campaignid=1&zoneid=3&loc=https%3A%2F%2Fsegmentfault.com%2Fq%2F1010000011211490&referer=https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3Dj2Hmj-IKB94pj4xQf4qek9WfD8LmebDZ-iM03Q6un4nvVk3226a9Y7LtPHGY1MWo5auuRVoShc6OZXqkWDdYOK%26wd%3D%26eqid%3Dac37a5dd00022b64000000025f3a45e5&cb=240a219840)

5 个回答

[得票](https://segmentfault.com/q/1010000011211490#comment-area)[时间](https://segmentfault.com/q/1010000011211490?sort=created#comment-area)

[![avatar](https://avatar-static.segmentfault.com/380/923/3809239748-59bfa75261934_big64)**krun**](https://segmentfault.com/u/krun)

-  **6.6k**

情况5中的 `@PathVariable` 注解是从请求路径中获取值，与 `RequestBody` 无关，所以能取到。

`@DeleteMapping` 注解的参数其实有办法取到，方法的参数加个 `@RequestBody` 注解，这样就是直接获取整个原始请求体，就要你自己解析了。
postman的话，body类型设置为raw即可：

![clipboard.png](https://segmentfault.com/img/bVVcYr?w=492&h=627)

![clipboard.png](https://segmentfault.com/img/bVVcZk?w=349&h=170)

搜索了一下网上的资料
[这是一种做法](https://segmentfault.com/q/1010000011211490#http://blog.csdn.net/qq_37545366/article/details/76132040)，但是我不清楚这适不适合你。

记得很久以前看到过关于这个问题的说明，好像是说 Spring 的 `@RequestMapping` 对 `GET/POST` 以外的请求方式没有做完整的解析，当时那篇文章说要自己手动实现，现在找不到那篇文章了，也可能是我记忆有误。

然后关于你 情况5 的 "二次删除数据报错"
你的确是已经删除了id为2的数据啊，怎么可能再删除得了呢？
你可以把delete操作catch起来，检查是否为这个已知异常，是的话传回错误信息 "id 不存在"

 赞 2

 已采纳

[评论](javascript:;) [赞赏](javascript:;) [更新于 2017-09-17](https://segmentfault.com/q/1010000011211490/a-1020000011212299)

- [**上邪**](https://segmentfault.com/u/dayanda)： 

  谢谢你，把这么长一个问题全看完了！！
  @RequestBody 应该能获取的到，我上面有一个示例说这种获取不到，是因为我突然想起来，当时没有使用raw的方式传递json数据！

  [ ](javascript:;) [回复](javascript:;) 2017-09-18

- [**krun**](https://segmentfault.com/u/krun)： 

  哈哈 只是要自己解析了 不过应该也不算太复杂

  [ ](javascript:;) [回复](javascript:;) 2017-09-18

[![avatar](https://cdn.segmentfault.com/v-5f0a9217/global/img/user-64.png)**阿克蒙德**](https://segmentfault.com/u/akemengde)

-  **2**
- **新人请关照**

不知道楼主有没有解决
用 x-www-form-uilencodeded 是可以的

 赞

[评论](javascript:;) [赞赏](javascript:;) [更新于 2018-05-25](https://segmentfault.com/q/1010000011211490/a-1020000015032134)

[![avatar](https://cdn.segmentfault.com/v-5f0a9217/global/img/user-64.png)**石头剪子贝儿**](https://segmentfault.com/u/shitoujianzibeier)

-  **3**
- **新人请关照**

使用@requestBody注解后,并没有解决该问题

 赞

[评论](javascript:;) [赞赏](javascript:;) [更新于 2019-01-22](https://segmentfault.com/q/1010000011211490/a-1020000017980010)

[![avatar](https://avatar-static.segmentfault.com/172/216/1722162001-5c8ef2faac5ee_big64)**main**](https://segmentfault.com/u/main_5c8ef1d2b6ba4)

-  **1**
- **新人请关照**

楼主解决了吗，我传的是一个UserList的一个id串，PostMan我用x-www-form-uilencodeded传的，后端用的@deleteMapping然后接收不到参数

 赞

[评论](javascript:;) [赞赏](javascript:;) [发布于 2019-03-18](https://segmentfault.com/q/1010000011211490/a-1020000018550400)

- [**上邪**](https://segmentfault.com/u/dayanda)： 

  最后没有再跟进这个问题，因为当时也是做测试写的demo

  [ ](javascript:;) [回复](javascript:;) 2019-03-18

[![avatar](https://cdn.segmentfault.com/v-5f0a9217/global/img/user-64.png)**chilion**](https://segmentfault.com/u/chilion)

-  **1**
- **新人请关照**

很好，这个坑今天被我踩到了，接收到的参数变成null 是因为
这个value属性，把它干掉就可以接收到数据了。具体什么原因我还没来得及看，我也是对比了好久才试出来的 ![clipboard.png](https://segmentfault.com/img/bVbs5dT?w=405&h=36)

![clipboard.png](https://segmentfault.com/img/bVbs5ef?w=320&h=19)

![clipboard.png](https://segmentfault.com/img/bVbs5em?w=189&h=24)

- 