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