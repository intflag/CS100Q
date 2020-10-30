# Java Code Snippet
## JSON
### 1）排除 JSON 中多余的字段
```java
obj转json
@param object
@param excludeProperties 需要排除的属性


//排除不需要转换成为json的字段

PropertyPreFilters filters = new PropertyPreFilters();
PropertyPreFilters.MySimplePropertyPreFilter excludefilter = filters.addFilter();
excludefilter.addExcludes(excludeProperties);
String jsonStr = JSONObject.toJSONString(object, excludefilter, SerializerFeature.PrettyFormat);
return jsonStr;

json转obj

T t=new T();

//忽略json中多出的字段
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
t = objectMapper.readValue(jsonStr, T.class);
String json=obj2Json(t,excludeProperties);
t = JSON.parseObject(json,T.class);

return t;
```

### 2）jackson 数组格式 json 字符串转换成 List
```java
//类主要是
import org.codehaus.jackson.type.TypeReference;
import org.codehaus.jackson.map.ObjectMapper;

[{"id":"36CD0224C1ED25F5E0538A3B0B7A8190","catgId":null,"matcAmont":50000,"lendPoolId":"36CD0224C1C225F5E0538A3B0B7A8190","balance":50000,"matchFlag":1,"isLock":1,"createTime":1467576000000,"userId":0,"mltCustLendPool":{"id":"36CD0224C1C225F5E0538A3B0B7A8190","userId":157020,"state":1,"isLock":1,"balance":50000,"curTotaAmt":50000,"syncDate":1467561600000,"inviFlag":2,"investTime":1467595065000,"deadline":1499131065000,"billDate":1467681465000,"billDay":5,"productId":"6","productName":"xxx","investAmt":50000,"prodOrderId":38662,"userTelephone":"15922166933","userName":"张三","idcardNum":"1111111111111","projectNum":null,"matchModelCode":null,"creditProduct":{"id":"290AA19B1134838EE053A716C0769130","swldid":"6","prodName":"xxx","prodType":null,"prodRate":0.13,"synFeeRate":null,"payCapitalType":null,"freezeTime":12,"prodCode":6,"prodCatgory":1,"unit":1},"subject":null},"orderFlag":null,"subject":0,"projectNum":null,"matchModeCode":"1"},{"id":"36DA5B50E54E2790E0538A3B0B7A2261","catgId":null,"matcAmont":50000,"lendPoolId":"36CD0224C1C225F5E0538A3B0B7A8190","balance":50000,"matchFlag":1,"isLock":1,"createTime":1467748800000,"userId":0,"mltCustLendPool":{"id":"36CD0224C1C225F5E0538A3B0B7A8190","userId":157020,"state":1,"isLock":1,"balance":50000,"curTotaAmt":50000,"syncDate":1467561600000,"inviFlag":2,"investTime":1467595065000,"deadline":1499131065000,"billDate":1467681465000,"billDay":5,"productId":"6","productName":"xxx","investAmt":50000,"prodOrderId":38662,"userTelephone":"15922166933","userName":"王dd","idcardNum":"2222222222222","projectNum":null,"matchModelCode":null,"creditProduct":{"id":"290AA19B1134838EE053A716C0769130","swldid":"6","prodName":"dfdfd","prodType":null,"prodRate":0.13,"synFeeRate":null,"payCapitalType":null,"freezeTime":12,"prodCode":6,"prodCatgory":1,"unit":1},"subject":null},"orderFlag":null,"subject":0,"projectNum":null,"matchModeCode":"1"}]

ObjectMapper mapper = new ObjectMapper();
List<MltWaitLendReco> lendReco = mapper.readValue(listStr,new TypeReference<List<MltWaitLendReco>>() { });
System.out.println(lendReco.get(0).getId());
```


## Java 8 Stream
### 1）List 过滤
```java
List<Student> collect = valueList.stream().filter(stu -> "tom".equals(stu.getName())).collect(Collectors.toList());
```
### 2）List 转 Map
```java
public static Map<Object, Object> getStudentObjectMap(List<Student> list) {
    Map<Object, Object> map = list.stream().collect(Collectors.toMap(Student::getStuId, student -> student));
    map.forEach((key, value) -> {
        System.out.println("key:" + key + ",value:" + value);
    });
    return map;
}
```
## Java 8 LocalDateTime
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMMddHHmm");
String time = LocalDateTime.now().format(formatter);

//获取秒数
Long second = LocalDateTime.now().toEpochSecond(ZoneOffset.of("+8"));
//获取毫秒数
Long milliSecond = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();


// 01. java.util.Date --> java.time.LocalDateTime
public void UDateToLocalDateTime() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
}

// 02. java.util.Date --> java.time.LocalDate
public void UDateToLocalDate() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
    LocalDate localDate = localDateTime.toLocalDate();
}

// 03. java.util.Date --> java.time.LocalTime
public void UDateToLocalTime() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
    LocalTime localTime = localDateTime.toLocalTime();
}


// 04. java.time.LocalDateTime --> java.util.Date
public void LocalDateTimeToUdate() {
    LocalDateTime localDateTime = LocalDateTime.now();
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDateTime.atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}


// 05. java.time.LocalDate --> java.util.Date
public void LocalDateToUdate() {
    LocalDate localDate = LocalDate.now();
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDate.atStartOfDay().atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}

// 06. java.time.LocalTime --> java.util.Date
public void LocalTimeToUdate() {
    LocalTime localTime = LocalTime.now();
    LocalDate localDate = LocalDate.now();
    LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDateTime.atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}
```