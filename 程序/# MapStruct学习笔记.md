# MapStruct学习笔记

```
MapStruct版本号是1.2.0.Final
http://mapstruct.org/documentation/stable/reference/html/12
```

学习mapstruct主要就是看文档，还有一个小窍门是编译之后，会直接在相应的包下生成一个xxxImpl.class，   直接查看此类就可以了

# 项目配置

最小化的配置：

```GROOVY
...
plugins {
    ...
    id 'net.ltgt.apt' version '0.8'
}
dependencies {
    ...
    compile 'org.mapstruct:mapstruct-jdk8:1.2.0.Final'

    apt 'org.mapstruct:mapstruct-processor:1.2.0.Final'
}
...
compileJava {
    options.compilerArgs = [
        '-Amapstruct.suppressGeneratorTimestamp=true',
        '-Amapstruct.suppressGeneratorVersionInfoComment=true'
    ]
}
...12345678910111213141516171819
```

我的整个配置文件：

```
buildscript {
    ext {
        spring_boot_version = "1.5.9.RELEASE"
        mapstruct_version = "1.2.0.Final"
        apt_plugin_version = "0.15"
        lombok_version = "1.16.20"
    }
    repositories {
        mavenLocal()
        mavenCentral()
        maven { url "http://maven.aliyun.com/nexus/content/groups/public" }
    }
    dependencies {
        classpath("net.ltgt.gradle:gradle-apt-plugin:${apt_plugin_version}")
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${spring_boot_version}")
    }
}

apply plugin: "java"
apply plugin: "org.springframework.boot"
apply plugin: "net.ltgt.apt"

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile "org.springframework.boot:spring-boot-starter-web:${spring_boot_version}"
    compile "org.mapstruct:mapstruct-jdk8:${mapstruct_version}"
    compileOnly "org.projectlombok:lombok:${lombok_version}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstruct_version}"
    annotationProcessor "org.projectlombok:lombok:${lombok_version}"
    testCompile 'org.testng:testng:6.10', 'org.easytesting:fest-assert:1.4'
}

tasks.withType(JavaCompile) {
    options.compilerArgs = [
            '-Amapstruct.suppressGeneratorTimestamp=true',
            '-Amapstruct.defaultComponentModel=spring'
    ]
}

task wrapper(type: Wrapper) {
    gradleVersion = '3.4'
}

test {
    useTestNG()
}1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253
```

**MapStruct 全局配置**

- mapstruct. suppressGeneratorTimestamp：false/true
- mapstruct. suppressGeneratorVersionInfoComment：false/true
- mapstruct.defaultComponentModel：default/cdi/spring/jsr330，使用spring时，需要spring-boot支持
- mapstruct.unmappedTargetPolicy：ERROR/WARN/IGNORE

# 操作

## Mapper

### 基础操作

```java
@Mapper
public interface CarMapper {

    @Mappings({
        @Mapping(source = "make", target = "manufacturer"),
        @Mapping(source = "numberOfSeats", target = "seatCount")
    })
    CarDto carToCarDto(Car car);

    @Mapping(source = "name", target = "fullName")
    PersonDto personToPersonDto(Person person);
}123456789101112
```

在CarMapper中的方法`carToCarDto` 是转换的入口函数。`@Mapping` 注解中，source是Car类，target是CarDto类。注解含义是Car类的make变量，CarDto中manufacturer相互转换。其中Car中有一个driver变量，是一个Person类。做转换时，会自动调用`personToPersonDto` 做转换（具体原理需要查看源码）。

### interface和abstract

```java
@Mapper
public interface CarMapper {

    @Mappings({...})
    CarDto carToCarDto(Car car);

    default PersonDto personToPersonDto(Person person) {
        //hand-written mapping logic
    }
}12345678910
```

在Java 8时，可以在接口中指定default方法和实现转换逻辑，我们可以这样写mapper。他会调用默认方法去转换Person。

```java
@Mapper
public abstract class CarMapper {

    @Mappings(...)
    public abstract CarDto carToCarDto(Car car);

    public PersonDto personToPersonDto(Person person) {
        //hand-written mapping logic
    }
}12345678910
```

抽象类，也同样可以达到这个效果。

### 多参数

```java
@Mapper
public interface AddressMapper {

    @Mappings({
        @Mapping(source = "person.description", target = "description"),
        @Mapping(source = "address.houseNo", target = "houseNumber")
    })
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Address address);
}123456789
```

### 使用原有对象

```java
@Mapper
public interface CarMapper {

    void updateCarFromDto(CarDto carDto, @MappingTarget Car car);
}12345
```

### 直接访问变量

```java
@Mapper
public interface CustomerMapper {

    CustomerMapper MAPPER = Mappers.getMapper( CustomerMapper.class );

    @Mapping(source = "customerName", target = "name")
    Customer toCustomer(CustomerDto customerDto);

    @InheritInverseConfiguration
    CustomerDto fromCustomer(Customer customer);
}1234567891011
```

## 获取Mapper

### mapper factory

```java
@Mapper
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );

    CarDto carToCarDto(Car car);
}

Car car = ...;
CarDto dto = CarMapper.INSTANCE.carToCarDto( car );
1234567891011
```

### cdi

componentModel对应值参照前文表格。

```java
@Mapper(componentModel = "cdi")
public interface CarMapper {

    CarDto carToCarDto(Car car);
}

@Inject
private CarMapper mapper;
123456789
```

## 数值类型转换

### 隐式类型转换

支持的类型有很多种（Boolean，Integer，String，Date），详情见[文档](http://mapstruct.org/documentation/stable/reference/html/#implicit-type-conversions)

```java
@Mapper
public interface CarMapper {

    @Mapping(source = "price", numberFormat = "$#.00")
    CarDto carToCarDto(Car car);

    @IterableMapping(numberFormat = "$#.00")
    List<String> prices(List<Integer> prices);

    @Mapping(source = "power", numberFormat = "#.##E0")
    CarDto carToCarDto(Car car);

    @Mapping(source = "manufacturingDate", dateFormat = "dd.MM.yyyy")
    CarDto carToCarDto(Car car);

    @IterableMapping(dateFormat = "dd.MM.yyyy")
    List<String> stringListToDateList(List<Date> dates);
}
12345678910111213141516171819
```

### Object转换

```java
@Mapper
public interface CarMapper {

    CarDto carToCarDto(Car car);

    PersonDto personToPersonDto(Person person);
}1234567
```

转换的规则：

- source和target类型**相同**，直接拷贝。含有List类型的属性，会复制一个新的List到target
- source(Atype attr)和target(Btype attr)类型**不相同**，就去查找mapper（当前或其他）中，是否含有`Btype func(Atype attr)` ，找到就使用。
- 如果没有找到func函数，看是否有内置的转换(a built-in conversion)
- 如果还是没有，则尝试自动创建一个mapping去处理。此配置会取消自动创建`@Mapper( disableSubMappingMethodsGeneration = true )`
- 如果创建失败，则会出错

根据转换规则，可以很方便写出处理的方法，能满足任何定制化需求(也是mapstruct强大的地方，尽量让人避免编码。迫不得已进行编码时，也只需要定制化小部分就行，自动帮你组装)。

### 不同名称的变量转换

```java
@Mapper
public interface FishTankMapper {

    @Mappings({
        @Mapping(target = "fish.kind", source = "fish.type"),
        @Mapping(target = "fish.name", ignore = true),
        @Mapping(target = "ornament", source = "interior.ornament"),
        @Mapping(target = "material.materialType", source = "material"),
        @Mapping(target = "quality.report.organisation.name", source = "quality.report.organisationName")
    })
    FishTankDto map( FishTank source );
}

@Mapper
public interface FishTankMapperWithDocument {

    @Mappings({
        @Mapping(target = "fish.kind", source = "fish.type"),
        @Mapping(target = "fish.name", expression = "java(\"Jaws\")"),
        @Mapping(target = "plant", ignore = true ),
        @Mapping(target = "ornament", ignore = true ),
        @Mapping(target = "material", ignore = true),
        @Mapping(target = "quality.document", source = "quality.report"),
        @Mapping(target = "quality.document.organisation.name", constant = "NoIdeaInc" )
    })
    FishTankWithNestedDocumentDto map( FishTank source );

}12345678910111213141516171819202122232425262728
```

### 使用其他mapper

```java
public class DateMapper {

    public String asString(Date date) {
        return date != null ? new SimpleDateFormat( "yyyy-MM-dd" )
            .format( date ) : null;
    }

    public Date asDate(String date) {
        try {
            return date != null ? new SimpleDateFormat( "yyyy-MM-dd" )
                .parse( date ) : null;
        }
        catch ( ParseException e ) {
            throw new RuntimeException( e );
        }
    }
}

@Mapper(uses=DateMapper.class)
public class CarMapper {

    CarDto carToCarDto(Car car);
}1234567891011121314151617181920212223
```

### 传递目标类型@TargetType

```java
@ApplicationScoped // CDI component model
public class ReferenceMapper {

    @PersistenceContext
    private EntityManager entityManager;

    public <T extends BaseEntity> T resolve(Reference reference, @TargetType Class<T> entityClass) {
        return reference != null ? entityManager.find( entityClass, reference.getPk() ) : null;
    }

    public Reference toReference(BaseEntity entity) {
        return entity != null ? new Reference( entity.getPk() ) : null;
    }
}

@Mapper(componentModel = "cdi", uses = ReferenceMapper.class )
public interface CarMapper {

    Car carDtoToCar(CarDto carDto);
}

//生成的代码
@ApplicationScoped
public class CarMapperImpl implements CarMapper {

    @Inject
    private ReferenceMapper referenceMapper;

    @Override
    public Car carDtoToCar(CarDto carDto) {
        if ( carDto == null ) {
            return null;
        }

        Car car = new Car();

        car.setOwner( referenceMapper.resolve( carDto.getOwner(), Owner.class ) );
        // ...

        return car;
    }
}123456789101112131415161718192021222324252627282930313233343536373839404142
```

### 传递@Context到Mapper中

```java
public abstract CarDto toCar(Car car, @Context Locale translationLocale);

protected OwnerManualDto translateOwnerManual(OwnerManual ownerManual, @Context Locale locale) {
    // manually implemented logic to translate the OwnerManual with the given Locale
}

//生成后的代码
public CarDto toCar(Car car, Locale translationLocale) {
    if ( car == null ) {
        return null;
    }

    CarDto carDto = new CarDto();

    carDto.setOwnerManual( translateOwnerManual( car.getOwnerManual(), translationLocale );
    // more generated mapping code

    return carDto;
}12345678910111213141516171819
```

### @Qualifier指定处理

```java
import org.mapstruct.Qualifier;

@Qualifier
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface TitleTranslator {
}

import org.mapstruct.Qualifier;

@Qualifier
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface EnglishToGerman {
}

import org.mapstruct.Qualifier;

@Qualifier
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface GermanToEnglish {
}

@Mapper( uses = Titles.class )
public interface MovieMapper {

     @Mapping( target = "title", qualifiedBy = { TitleTranslator.class, EnglishToGerman.class } )
     GermanRelease toGerman( OriginalRelease movies );

}
可以这样使用
@TitleTranslator
public class Titles {

    @EnglishToGerman
    public String translateTitleEG(String title) {
        // some mapping logic
    }

    @GermanToEnglish
    public String translateTitleGE(String title) {
        // some mapping logic
    }
}
或者这样使用
@Named("TitleTranslator")
public class Titles {

    @Named("EnglishToGerman")
    public String translateTitleEG(String title) {
        // some mapping logic
    }

    @Named("GermanToEnglish")
    public String translateTitleGE(String title) {
        // some mapping logic
    }
}

@Mapper( uses = Titles.class )
public interface MovieMapper {

     @Mapping( target = "title", qualifiedByName = { "TitleTranslator", "EnglishToGerman" } )
     GermanRelease toGerman( OriginalRelease movies );

}12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667
```

## 集合的mapping

### maps

```java
map中的数值转换
public interface SourceTargetMapper {

    @MapMapping(valueDateFormat = "dd.MM.yyyy")
    Map<String, String> longDateMapToStringStringMap(Map<Long, Date> source);
}

//生成代码
@Override
public Map<Long, Date> stringStringMapToLongDateMap(Map<String, String> source) {
    if ( source == null ) {
        return null;
    }

    Map<Long, Date> map = new HashMap<Long, Date>();

    for ( Map.Entry<String, String> entry : source.entrySet() ) {

        Long key = Long.parseLong( entry.getKey() );
        Date value;
        try {
            value = new SimpleDateFormat( "dd.MM.yyyy" ).parse( entry.getValue() );
        }
        catch( ParseException e ) {
            throw new RuntimeException( e );
        }

        map.put( key, value );
    }

    return map;
}1234567891011121314151617181920212223242526272829303132
```

![这里写图片描述](https://img-blog.csdn.net/20180614164051323?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Z0bmV3cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180614164057999?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Z0bmV3cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## Stream的mapping

```java
@Mapper
public interface CarMapper {

    Set<String> integerStreamToStringSet(Stream<Integer> integers);

    List<CarDto> carsToCarDtos(Stream<Car> cars);

    CarDto carToCarDto(Car car);
}

//代码生成
@Override
public Set<String> integerStreamToStringSet(Stream<Integer> integers) {
    if ( integers == null ) {
        return null;
    }

    return integers.stream().map( integer -> String.valueOf( integer ) )
        .collect( Collectors.toCollection( HashSet<String>::new ) );
}

@Override
public List<CarDto> carsToCarDtos(Stream<Car> cars) {
    if ( cars == null ) {
        return null;
    }

    return integers.stream().map( car -> carToCarDto( car ) )
        .collect( Collectors.toCollection( ArrayList<CarDto>::new ) );
}
12345678910111213141516171819202122232425262728293031
```

## Mapping Values

### enum type

```java
@Mapper
public interface OrderMapper {

    OrderMapper INSTANCE = Mappers.getMapper( OrderMapper.class );

    @ValueMappings({
        @ValueMapping(source = "EXTRA", target = "SPECIAL"),
        @ValueMapping(source = "STANDARD", target = "DEFAULT"),
        @ValueMapping(source = "NORMAL", target = "DEFAULT")
    })
    ExternalOrderType orderTypeToExternalOrderType(OrderType orderType);
}

// GENERATED CODE
public class OrderMapperImpl implements OrderMapper {

    @Override
    public ExternalOrderType orderTypeToExternalOrderType(OrderType orderType) {
        if ( orderType == null ) {
            return null;
        }

        ExternalOrderType externalOrderType_;

        switch ( orderType ) {
            case EXTRA: externalOrderType_ = ExternalOrderType.SPECIAL;
            break;
            case STANDARD: externalOrderType_ = ExternalOrderType.DEFAULT;
            break;
            case NORMAL: externalOrderType_ = ExternalOrderType.DEFAULT;
            break;
            case RETAIL: externalOrderType_ = ExternalOrderType.RETAIL;
            break;
            case B2B: externalOrderType_ = ExternalOrderType.B2B;
            break;
            default: throw new IllegalArgumentException( "Unexpected enum constant: " + orderType );
        }

        return externalOrderType_;
    }
}

@Mapper
public interface SpecialOrderMapper {

    SpecialOrderMapper INSTANCE = Mappers.getMapper( SpecialOrderMapper.class );

    @ValueMappings({
        @ValueMapping( source = MappingConstants.NULL, target = "DEFAULT" ),
        @ValueMapping( source = "STANDARD", target = MappingConstants.NULL ),
        @ValueMapping( source = MappingConstants.ANY_REMAINING, target = "SPECIAL" )
    })
    ExternalOrderType orderTypeToExternalOrderType(OrderType orderType);
}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354
```

## 对象工厂

```java
public class DtoFactory {

     public CarDto createCarDto() {
         return // ... custom factory logic
     }
}

public class EntityFactory {

     public <T extends BaseEntity> T createEntity(@TargetType Class<T> entityClass) {
         return // ... custom factory logic
     }
}

@Mapper(uses= { DtoFactory.class, EntityFactory.class } )
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );

    CarDto carToCarDto(Car car);

    Car carDtoToCar(CarDto carDto);
}

//生成代码
public class CarMapperImpl implements CarMapper {

    private final DtoFactory dtoFactory = new DtoFactory();

    private final EntityFactory entityFactory = new EntityFactory();

    @Override
    public CarDto carToCarDto(Car car) {
        if ( car == null ) {
            return null;
        }

        CarDto carDto = dtoFactory.createCarDto();

        //map properties...

        return carDto;
    }

    @Override
    public Car carDtoToCar(CarDto carDto) {
        if ( carDto == null ) {
            return null;
        }

        Car car = entityFactory.createEntity( Car.class );

        //map properties...

        return car;
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758
```

## 高级技巧

### 默认值

```java
@Mapper(uses = StringListMapper.class)
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mappings( {
        @Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined"),
        @Mapping(target = "longProperty", source = "longProp", defaultValue = "-1"),
        @Mapping(target = "stringConstant", constant = "Constant Value"),
        @Mapping(target = "integerConstant", constant = "14"),
        @Mapping(target = "longWrapperConstant", constant = "3001"),
        @Mapping(target = "dateConstant", dateFormat = "dd-MM-yyyy", constant = "09-01-2014"),
        @Mapping(target = "stringListConstants", constant = "jack-jill-tom")
    } )
    Target sourceToTarget(Source s);
}
1234567891011121314151617
```

### 表达式

可以加入java代码：

```java
@Mapper
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "timeAndFormat",
         expression = "java( new org.sample.TimeAndFormat( s.getTime(), s.getFormat() ) )")
    Target sourceToTarget(Source s);
}

imports org.sample.TimeAndFormat;

@Mapper( imports = TimeAndFormat.class )
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "timeAndFormat",
         expression = "java( new TimeAndFormat( s.getTime(), s.getFormat() ) )")
    Target sourceToTarget(Source s);
}123456789101112131415161718192021
```

### 定义返回类型

```java
@Mapper( uses = FruitFactory.class )
public interface FruitMapper {

    @BeanMapping( resultType = Apple.class )
    Fruit map( FruitDto source );

}

public class FruitFactory {

    public Apple createApple() {
        return new Apple( "Apple" );
    }

    public Banana createBanana() {
        return new Banana( "Banana" );
    }
}123456789101112131415161718
```

### 异常

```java
@Mapper(uses = HandWritten.class)
public interface CarMapper {

    CarDto carToCarDto(Car car) throws GearException;
}

public class HandWritten {

    private static final String[] GEAR = {"ONE", "TWO", "THREE", "OVERDRIVE", "REVERSE"};

    public String toGear(Integer gear) throws GearException, FatalException {
        if ( gear == null ) {
            throw new FatalException("null is not a valid gear");
        }

        if ( gear < 0 && gear > GEAR.length ) {
            throw new GearException("invalid gear");
        }
        return GEAR[gear];
    }
}

// GENERATED CODE
@Override
public CarDto carToCarDto(Car car) throws GearException {
    if ( car == null ) {
        return null;
    }

    CarDto carDto = new CarDto();
    try {
        carDto.setGear( handWritten.toGear( car.getGear() ) );
    }
    catch ( FatalException e ) {
        throw new RuntimeException( e );
    }

    return carDto;
}123456789101112131415161718192021222324252627282930313233343536373839
```

### 装饰器

```java
@Mapper
@DecoratedWith(PersonMapperDecorator.class)
public interface PersonMapper {

    PersonMapper INSTANCE = Mappers.getMapper( PersonMapper.class );

    PersonDto personToPersonDto(Person person);

    AddressDto addressToAddressDto(Address address);
}

public abstract class PersonMapperDecorator implements PersonMapper {

    private final PersonMapper delegate;

    public PersonMapperDecorator(PersonMapper delegate) {
        this.delegate = delegate;
    }

    @Override
    public PersonDto personToPersonDto(Person person) {
        PersonDto dto = delegate.personToPersonDto( person );
        dto.setFullName( person.getFirstName() + " " + person.getLastName() );
        return dto;
    }
}
```

### before-mapping和after-mapping

```java
@Mapper
public abstract class VehicleMapper {

    @BeforeMapping
    protected void flushEntity(AbstractVehicle vehicle) {
        // I would call my entity manager's flush() method here to make sure my entity
        // is populated with the right @Version before I let it map into the DTO
    }

    @AfterMapping
    protected void fillTank(AbstractVehicle vehicle, @MappingTarget AbstractVehicleDto result) {
        result.fuelUp( new Fuel( vehicle.getTankCapacity(), vehicle.getFuelType() ) );
    }

    public abstract CarDto toCarDto(Car car);
}

// Generates something like this:
public class VehicleMapperImpl extends VehicleMapper {

    public CarDto toCarDto(Car car) {
        flushEntity( car );

        if ( car == null ) {
            return null;
        }

        CarDto carDto = new CarDto();
        // attributes mapping ...

        fillTank( car, carDto );

        return carDto;
    }
}

@Mapper
public abstract class VehicleMapper {

    @PersistenceContext
    private EntityManager entityManager;

    @AfterMapping
    protected <T> T attachEntity(@MappingTarget T entity) {
        return entityManager.merge(entity);
    }

    public abstract CarDto toCarDto(Car car);
}

// Generates something like this:
public class VehicleMapperImpl extends VehicleMapper {

    public CarDto toCarDto(Car car) {
        if ( car == null ) {
            return null;
        }

        CarDto carDto = new CarDto();
        // attributes mapping ...

        CarDto target = attachEntity( carDto );
        if ( target != null ) {
            return target;
        }

        return carDto;
    }
}
```

