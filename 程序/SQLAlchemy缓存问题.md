## 问题描述

最近在排查一个问题，为了方便说明，我们假设现在有如下一个API：



```python
@app.route("/sqlalchemy/test", methods=['GET'])
def sqlalchemy_test_api():
    data = {}
    # 获取商品价格
    product = Product.query.get(1)
    data['old_price'] = product.present_price
    # 休眠10秒，等待外部修改价格
    time.sleep(10)
    product = Product.query.get(1)
    data['new_price'] = product.present_price
    return jsonify(status='ok', data=data)
```

这里我们的后台使用了[Flask](https://link.jianshu.com?t=http://flask.pocoo.org/)作为服务端框架，[SQLAlchemy](https://link.jianshu.com?t=https://www.sqlalchemy.org/)作为数据库ORM框架。Product是一张商品表的ORM模型，假设原来id=1的商品价格为10，在程序休眠的10秒内价格被修改为20，那么你觉得返回的结果是多少？

old_price显然是10，那么new_price呢？讲道理的话由于外部修改价格为20了，同时程序在sleep后立刻又query了一次，你可能觉得new_price应该是20。但结果并不是，真实测试的结果是10，给人感觉就像是SQLAlchemy“缓存”了上一次的结果。

另外在测试的过程还发现一个现象，虽然在第一次API调用时两个price都是10，但是在第二次调用API时，读到的price是20。也就是说，在一个新的API开始时，之前“缓存”的结果被清除了。

## SQLAlchemy的session状态管理

之前我们提出了一个猜测：第二次查询是否“缓存”了第一次查询。为了验证这个猜想，我们可以把`SQLALCHEMY_ECHO`这个配置项打开，这是个全局配置项，官方文档定义如下：

| 配置项            | 说明                                                         |
| ----------------- | :----------------------------------------------------------- |
| `SQLALCHEMY_ECHO` | If set to True SQLAlchemy will log all the statements issued to stderr which can be useful for debugging. |

在这个配置项打开的情况下，我们可以看到查询语句输出到终端下。我们再次调用API，可以发现第一次查询会输出类似`SELECT * FROM product WHERE id = 1`的语句，而第二次查询则没有这样的输出。如此看来，SQLAlchemy确实缓存了上次的结果，在第二次查询的时候直接使用了上次的结果。

实际上，当执行第一句`product = Product.query.get(1)`时，product这个对象处于持久状态(persistent)了，我们可以通过一些工具看到ORM对象目前处于的状态。详细的状态列表可在[官方文档](https://link.jianshu.com?t=http://docs.sqlalchemy.org/en/latest/orm/session_state_management.html#quickie-intro-to-object-states)中找到。



```python
>>> from sqlalchemy import inspect
>>> insp = inspect(product)
>>> insp.persistent
True
>>> product.__dict__
{
  'id': 1, 'present_price': 10,
  '_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x1106a3350>,
}
```

为了清除该对象的缓存，程度从低到高有下面几种做法。`expire`会清除对象里缓存的数据，这样下次查询时会直接从数据库进行查询。`refresh`不仅清除对象里缓存的数据，还会立刻触发一次数据库查询更新数据。`expire_all`的效果和`expire`一样，只不过会清除session里所有对象的缓存。`flush`会把所有本地修改写入到数据库，但没有提交。`commit`不仅把所有本地修改写入到数据库，同时也提交了该事务。



```python
db.session.expire(product)
db.session.refresh(product)
db.session.expire_all()
db.session.flush()
db.session.commit()
```

我们对这几种方法依次做实验，结果发现这5个操作都会让下次查询直接从数据库进行查询，但只有`commit`会读到最新的price。那这个又是什么原因呢，我们已经强制每次查询走数据库，为何还是读到“缓存”的数据。这个就要用数据库的事务隔离机制来解释了。

## 事务隔离

在数据库系统中，事务[隔离级别](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/Isolation_(database_systems))(isolation level)决定了数据在系统中的可见性。隔离级别从低到高分为四种：未提交读(Read uncommitted)，已提交读(Read committed)，可重复读(Repeatable read)，可串行化(Serializable)。他们的区别如下表所示。

| 隔离级别     |   脏读 | 不可重复读 |   幻读 |
| ------------ | -----: | ---------: | -----: |
| 未提交读(RU) |   可能 |       可能 |   可能 |
| 已提交读(RC) | 不可能 |       可能 |   可能 |
| 可重复读(RR) | 不可能 |     不可能 |   可能 |
| 可串行化     | 不可能 |     不可能 | 不可能 |

脏读(dirty read)是指一个事务可以读到其他事务还未提交的数据。不可重复读(non-repeatable read)是指在一个事务中同一行被读取了多次，可以读到不同的值。幻读(phantom read)是指在一个事务中执行同一个语句多次，读到的数据行发生了改变，即可能行数增加了或减少了。

前面提到的问题其实就涉及到不可重复读这个特性，即在一个事务中我们query了product.id=1的数据多次，但读到了重复的数据。对于MySQL来说，默认的事务隔离级别是RR，通过上表我们可知RR是可重复读的，因此可以解释这个现象。

| 事务A                                                        | 事务B                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `BEGIN;`                                                     | `BEGIN;`                                                     |
| `SELECT present_price FROM product WHERE id = 1; /* id=1的商品价格为10 */` |                                                              |
|                                                              | `UPDATE product SET present_price = 20 WHERE id = 1; /* 修改id=1的商品价格为20 */` |
|                                                              | `COMMIT;`                                                    |
| `SELECT present_price FROM product WHERE id = 1; /* 再次查询id=1的商品价格 */` |                                                              |
| `COMMIT;`                                                    |                                                              |

对于前面的问题，我们可以把两个事务的执行时序图画出来如上所示。因此为了使第二次查询得到正确的值，我们可以把隔离级别设为RC，或者在第二次查询前进行`COMMIT`新起一个事务。

## Flask-SQLAlchemy的自动提交

前面还遗留一个问题没有搞清楚：在一个新的API开始时，之前“缓存”的结果似乎被清除了。由于打开了`SQLALCHEMY_ECHO`配置项，我们可以观察到每次API结束的时候都会自动触发一次`COMMIT`，而正是这个自动提交清空了所有的“缓存”。通过查找源代码，我们发现是下面这段代码在起作用：



```python
@teardown
def shutdown_session(response_or_exc):
    if app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN']:
        if response_or_exc is None:
            self.session.commit()
    self.session.remove()
    return response_or_exc
```

如果配置项`SQLALCHEMY_COMMIT_ON_TEARDOWN`为`True`，那么首先触发`COMMIT`，最后统一执行`session.remove()`操作，即释放连接并回滚事务操作。

有意思的是，这个配置项在Flask2.0版本的Changelog中被移除了。

![img](https:////upload-images.jianshu.io/upload_images/2245716-7b55f7db34ac4c33.png?imageMogr2/auto-orient/strip|imageView2/2/w/710/format/webp)

Flask2.0 Changelog

关于删除的原因，作者在[stackoverflow](https://link.jianshu.com?t=https://stackoverflow.com/questions/23301968/invalid-transaction-persisting-across-requests)的一个帖子里进行了说明。这个帖子同时也解释了为什么在我们的生产环境中经常报这个错误：
 `InvalidRequestError: This session is in 'prepared' state; no further SQL can be emitted within this transaction.`，而且只有重启才能解决问题。有兴趣的同学可以深入阅读一下。

## 总结

在MySQL的同一个事务中，多次查询同一行的数据得到的结果是相同的，这里既有SQLAlchemy本身“缓存”结果的原因，也受到数据库隔离级别的影响。如果要强制读取最新的结果，最简单的办法就是在查询前手动`COMMIT`一次。根据这个原则，我们可以再仔细阅读下自己项目中的代码，看看会不会有一些隐藏的问题。

