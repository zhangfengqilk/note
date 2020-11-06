# springboot DELETE 方法传参的方式



## 1 spingboot对DELETE和PUT支持的有限

如果非要使用put 和delete方法，可以用Body的json格式传参，也可以使用resturl传参

![image-20200720112218237](D:\笔记\图片\image-20200720112218237.png)

# 2 spring boot 接受Json传的参数

- 可以用map<String,Object>来接收

- 也可以用List<>来接收，不过发送的Json需要是List类型的

  ```json
  [
     4,5
  ]
  ```

- 也可以用对象来接收

``` json
{
    "id":[1,2,3,4],
    "isDealed":"afdsadsf"
}
```

```java
public class DealWarnRecordParam {
    private List<Long> id;
    private String isDealed;
}

```



## 3 使用DELETE方式进行交互

说明：ResponseData为自定义返回体{String code, String msg, List<?> data}

​      PollutionData 为一个entity  属性部分包含{String id, String name}

​      CodeEnum、MsgEnum为自定义枚举类，定义了一些常量

​      两种方式皆测试过

​      环境：win7+idea2018.2+jdk10.0.2+springboot  前端编辑工具为hbuilder

两种方式：

1、



```
//方法一 使用POST+ _method:"DELETE" + filter（springboot不需要我们配置） 
//这里的传输对象为json对象，后台直接接受
var r=confirm("方法一：确认删除该条数据？");
if(r){
    //var data = {_method:"DELETE", id:"456456",name:"征集"};
    var data = {_method:"DELETE"};//_method:"DELETE"必须,其他属性看你需求
    $.ajax({
        url:"http://192.168.2.116:8080/pollution/delete/1786vdsds863",
        type:"POST",
        data:data,
        dataType:"json",
        success:function(result){
            alert(result.msg);
        }

    });        
}
```



```
@DeleteMapping("/pollution/delete/{id}")
public ResponseData deletePollutionById(@PathVariable("id")String id, @RequestBody PollutionData data){
    System.out.println(id);
    System.out.println(data);
    return new ResponseData(CodeEnum.SUCCESS.getCode(),MsgEnum.SUCCESS.getMsg(),null);
}
```

 

2、



```
//方法二 使用DELETE请求  
//这是的传输对象为json字符串  后台使用@RequestBody注解解析该字符串并将字符串映射到对应实体上
var r=confirm("方法二：确认删除该条数据？");
if(r){
    var id = "123133";
    var jsonstr = { id: id,
                   name: "12345"};
    console.log(jsonstr);
    $.ajax({
        url:"http://192.168.2.116:8080/pollution/delete/" + id,
        type:"DELETE",
        contentType:"application/json",//设置请求参数类型为json字符串
        data:JSON.stringify(jsonstr),//将json对象转换成json字符串发送
        dataType:"json",
        success:function(result){
            alert(result.msg);
        }

    });        
}

/**如果不需要传递参数，可以不写下面的几项
* contentType:"application/json",//设置请求参数类型为json字符串
  data:JSON.stringify(jsonstr),//将json对象转换成json字符串发送
  dataType:"json",
*/
```



```
@DeleteMapping("/pollution/delete/{id}")
public ResponseData deletePollutionById(@PathVariable("id")String id, @RequestBody PollutionData data){
    System.out.println(id);
    System.out.println(data);
    return new ResponseData(CodeEnum.SUCCESS.getCode(),MsgEnum.SUCCESS.getMsg(),null);
}
```

