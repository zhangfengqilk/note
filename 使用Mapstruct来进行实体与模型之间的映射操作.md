# 使用Mapstruct来进行实体与模型之间的映射操作

在一个成熟可维护的工程中，细分模块后，domian工程最好不要被其他工程依赖，但是实体类一般存于domain之中，这样其他工程想获取实体类数据时就需要在各自工程写model，自定义model可以根据自身业务需要而并不需要映射整个实体属性。

  mapstruct这个插件就是用来处理domin实体类与model类的属性映射，定义mapper接口，mapstruct就会自动的帮我们实现这个映射接口，避免了麻烦复杂的映射实现。

  工程中引入mapstruct依赖

```html
<!-- mapstruct -->
	<dependency>
      <groupId>org.mapstruct</groupId>
      <artifactId>mapstruct-jdk8</artifactId>
     <version>${org.mapstruct.version}</version>
    </dependency>
  </dependencies>
```


这里定义实体Person

```java
public class Person {  
    private String name;  
    private int age;  
    private String phone;
} 
```


这里定义模型PersonModel 

```java
public class PersonModel { 
    private String name;  
    private int age;  
    private String phone; 
}
```
定义实体Person与模型PersonModel,这里两个类的属性一致。

定义映射可以使用接口也可以使用静态类。

<1>使用接口映射：

    <a>简单实体映射：

```java
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    PersonModel map( Person entity);
    List<PersonModel > map(List< Person> entity);
}
```

这里在PersonMapper 定义了两个map方法，第一个是单实体映射，第二个方法是List映射。在存盘之后，mapstruct会自动在target文件里为我们实现我们定义的映射接口。

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2017-01-18T11:53:32+0800",
    comments = "version: 1.0.0.Final, compiler: Eclipse JDT (IDE) 1.2.0.v20150514-0146, environment: Java 1.8.0_31 (Oracle Corporation)"
)

public class PersonMapperImpl implements PersonMapper {

    @Override
    public PersonModel ma(Person entity) {
        if ( entity == null ) {
            return null;
        }
        PersonModel personModel = new PersonModel();
        personModel.setName( entity.getName() );
        personModel.setAge( entity.getAge() );
        personModel.setPhone( entity.gePhone() );
        return personModel;
    }

    @Override
    public List<PersonModel> map(List<Person> entity) {
        if ( entity == null ) {
            return null;
        }

        List<PersonModel> list = new ArrayList<PersonModel>();
        for ( Person person : entity ) {
            list.add( map( person) );
        }
        return list;
    }
}
```

在目标工程使用实体的时候只需要new一个PersonMapper的实例INSTANCE，就可以调用map()方法映射实体属性到模型中去了。
  但是这是在实体与模型的属性命名一致的情况下，这种情况下映射基本上不需要我们指定模型的哪个属性对应实体的哪个属性，在模型属性命名与实体属性命名不一致的情况下，还可以使用@Mapping(target = "模型属性", source = "实体属性")来指定的映射某个属性
  重新定义PersonModel跟Person
  这里定义实体Person
```java
public class Person {  
    private String name;
    private int age;  
    private String phone; 
} 
```
这里定义模型PersonModel 
```java
public class PersonModel {  
    private String personName;  
    private int age;  
    private String phone;  
}  
```

其中将PersonModel中的name属性改为personName，这里的映射接口写法就可以写成

```java
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    @Mapping(target = "personName", source = "name")
    PersonModel map( Person entity);
    List<PersonModel > map(List< Person> entity);
}
```

如果model定义了在实体没有可以映射的属性时，就可以使用@Mapping(target = "模型属性", ignore = true)来跳过不需要映射的模型属性了。

  如下面重新定义实体Person和模型PersonModel

  这里定义实体Person

```java
public class Person {  

    private String name;  

    private int age;  

    private String phone;  
} 
```
这里定义模型PersonModel 
```java
public class PersonModel {  
    private String personName; 
    private int age;  
    private String phone;  
    private String hand;
}  
```
 这里的映射接口就应该改为:

```java
@Mapper

public interface PersonMapper {

    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);

    @Mapping(target = "personName", source = "name")
    @Mapping(target = "hand",  ignore = true)

    PersonModel map( Person entity);
    List<PersonModel > map(List< Person> entity);
}
```

 如果模型与实体均存在很多属性的情况下，映射接口的@Mapping注解很容易写得很长，比如：

```java
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    @Mapping(target = "personName", source = "name")
    @Mapping(target = "hand",  ignore = true)
     ..
     ..
     ..
     ..
    PersonModel map( Personentity);
    List<PersonModel > map(List< Person> entity);
}
```


 这样的程序就不可避免的写得很笨了。所以我们也可以使用default默认方法来定义映射接口，如：
```java
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);

    @Mapping(target = "hand",  ignore = true)
    PersonModel map(Person entity, Person personName, Person age, Person phone);
    default PersonModel map(Person entity) {
        return INSTANCE.map(entity, entity.getName(), entity.getAge(), entity.getPhone());
    }
    List<PersonModel> map(List<Person> entity);
    }
```


<2>使用静态类映射实体

```java
@Mapper(componentModel = "spring")
public abstract class PesonMapper {

   @Mapping(target = "personName", source = "name")
   @Mapping(target = "hand",  ignore = true)
   protected abstract PersonModel map( Person entity);
   protected abstract  List<PersonModel > map(List< Person> entity);
}
```


 虽然看起来写法差不多一直，但是使用静态类来映射有他的好处，最起码接口只能定义方法，无法写方法体，但是使用了静态类，就可以写上方法体了，比如：



```java
@Mapper(componentModel = "spring")
public abstract class PesonMapper {
   public PersonModel mapEighteen( Person entity) {
	  if (entity.getName.equals(18)) {
		   return map(entity);
      }
   };
   @Mapping(target = "personName", source = "name")
   @Mapping(target = "hand",  ignore = true)
   protected abstract PersonModel map( Person entity);
   protected abstract  List<PersonModel > map(List< Person> entity);
}
```



调用mapEighteen()就可以只映射实体属性age为18的实体了。