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