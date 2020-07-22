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