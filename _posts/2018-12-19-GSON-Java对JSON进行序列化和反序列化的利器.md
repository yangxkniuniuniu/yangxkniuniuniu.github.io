---
layout:     post
title:      Gson Java中对JSON对象进行序列化和反序列化的利器
subtitle:   Gson
date:       2018-12-19
author:     owl city / covered from BY
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Java
    - Json
    - Gson
---


## java解析json文件（org.josn和gson）
### org.json
- 读取json
```
String json = "{\"name\":\"Tom\",\"age\":\"12\",\"info\":{\"id\":\"144\",\"uuid\":112}}";
JSONObject jsonObject = new JSONObject(json);
String name = jsonObject.getString("name");
String age = jsonObject.getString("age");
JSONObject object1 = jsonObject.getJSONObject("info");
String id = object1.getString("id");
Integer uuid = object1.getInt("uuid");
System.out.println("name:"+name+"...age:"+age+"...info:>id:"+id+"..>uuid:"+uuid);
```
- 生成json
```
JSONObject jsonObject1 = new JSONObject();
jsonObject1.put("title","数学");
jsonObject1.put("score",90);
System.out.println(jsonObject1.toString());
```
> 也可以使用JSONArray的方式读取和生成json
> [详细使用方法](https://blog.csdn.net/Zen99T/article/details/50351637)

### Gson
- 读取json
    - 生成java Bean
```
public class MytestBean {
    public String name;
    public String age;
    public Details details = new Details();
    public class Details{
        public String color;
        public String city;
    }
}
```
    - 读取json文件
```
public class JsonTest {
    public static void main(String[] args) throws  Exception{
        FileReader fr=new FileReader("/Users/cody/Desktop/test/mytest.json");
            BufferedReader br = new BufferedReader(fr);
            String line = "";
            while((line=br.readLine())!=null) {
                Gson gson = new Gson();
                MytestBean mytestBean = gson.fromJson(line, MytestBean.class);
                MytestBean.Details ss = mytestBean.details;
                System.out.println(ss.city + "  " + ss.color);

            }
    }
}
```
- 生成json文件
```
Writer writer = new FileWriter("/Users/cody/Desktop/test/writetest.json");
    Gson gson = new Gson();
    gson.toJson("hello",writer);
    gson.toJson("123",writer);
    writer.close();
```
### Gson高级使用
- 通过自定义序列化和反序列化过程来完成一些想做的事
**SessionInfoDeserializer**
```
public class SessionInfoDeserializer implements JsonDeserializer<SessionInfoParse> {

    @Override
    public SessionInfoParse deserialize(JsonElement jsonElement, Type type, JsonDeserializationContext jsonDeserializationContext) throws JsonParseException {
        final JsonObject jsonObject = jsonElement.getAsJsonObject();
        final String launch_session_id = getNotNull(jsonObject.get("launch_session_id"));
        final String client_launch_session_id = getNotNull(jsonObject.get("client_launch_session_id"));
        final String associate_session_id = getNotNull(jsonObject.get("associate_session_id"));
        final String client_associate_session_id = getNotNull(jsonObject.get("client_associate_session_id"));
        final String login_session_id = getNotNull(jsonObject.get("login_session_id"));
        final String client_login_session_id = getNotNull(jsonObject.get("client_login_session_id"));
        final String appInfo = getNotNull(jsonObject.get("app_info"));
        final String deviceInfo = getNotNull(jsonObject.get("device_info"));
        final String userInfo = getNotNull(jsonObject.get("user_info"));
        final String netWorkInfo = getNotNull(jsonObject.get("network_info"));

        final SessionInfoParse sessionInfoParse = new SessionInfoParse();
        sessionInfoParse.deviceInfo = deviceInfo;
        sessionInfoParse.appInfo = appInfo;
        sessionInfoParse.userInfo = userInfo;
        sessionInfoParse.networkInfo = netWorkInfo;
        sessionInfoParse.launch_session_id = launch_session_id;
        sessionInfoParse.client_launch_session_id = client_launch_session_id;
        sessionInfoParse.associate_session_id = associate_session_id;
        sessionInfoParse.client_associate_session_id = client_associate_session_id;
        sessionInfoParse.login_session_id = login_session_id;
        sessionInfoParse.client_login_session_id = client_login_session_id;
        return sessionInfoParse;
    }

    public String getNotNull(JsonElement str) {
        if (str.isJsonNull()){
        return "";}else{
            return str.getAsString();
        }
    }
}

```
**SessionInfoParse**
```
...
@Expose
public String client_login_session_id;

@Expose(serialize = false, deserialize = true)
public String appInfo;
@Expose(serialize = true, deserialize = false)
public AppInfo app_info;
...
```
**主程序,解析并发送到es**
```
/*
    1.映射反序列化成Java对象,这个对象里的DeviceInfo,appInfo,networkInfo,userInfo是字符串类型,
    它们在序列化的时候是不被暴露的,只在反序列化的暴露.
    2.将这4个String对象再经过一次反序列化,并且将得到的bean赋值给上一层得到的java对象的device_info,
    device_info,netWork_info,user_info.它们4个是另外定义的bean对象,在序列化的时候暴露,反序列化的时候隐藏
    3.再将重新赋值后的java对象进行序列化,得到需要的json对象

    notes:
    1.这里使用实现JsonDeserializer<SessionInfoParse>接口来自定义反序列化的过程,可以自定义
    java对象的属性名(如果使用默认方法,反序列化得到的json key(device_info)是和原来的(device_info)一样,
    再反序列化,还想使用(device_info)这个名词的时候,就会出现key出现多次的问题,因此自定了第一次反序列化得到的String的属性
    名称),保证最后仍能使用和原始格式一样的key.
    2.如果在最终序列化的时候不想暴露某个属性,可以设置它的expose参数
    3.在自定义反序列化的过程中,将原始json中的null替换成了"",不然会出现JsonNull的错误
 */
public void getMessage() throws Exception {
    GsonBuilder gsonBuilder = new GsonBuilder();
    gsonBuilder.excludeFieldsWithoutExposeAnnotation();
    gsonBuilder.registerTypeAdapter(SessionInfoParse.class, new SessionInfoDeserializer());
    Gson gson = gsonBuilder.create();

    Duration duration = Duration.of(1000, ChronoUnit.MILLIS);
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
    consumer.subscribe(topics);
    System.out.println("Start to subscribe topic :" + topics.toString());
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(duration);
        for (ConsumerRecord<String, String> record : records) {
            String message = record.value();
            EventParse event1 = gson.fromJson(message, EventParse.class);
            event1.data.session_info.device_info = gson.fromJson(event1.data.session_info.deviceInfo, DeviceInfo.class);
            event1.data.session_info.app_info = gson.fromJson(event1.data.session_info.appInfo, AppInfo.class);
            event1.data.session_info.netWork_info = gson.fromJson(event1.data.session_info.networkInfo, NetWorkInfo.class);
            event1.data.session_info.user_info = gson.fromJson(event1.data.session_info.userInfo, UserInfo.class);
            System.out.println(gson.toJson(event1));
            sendMessageToEs(gson.toJson(event1), event1.data.session_info.app_info.getToken());
        }
    }
}

/*
    将消息发送到ElasticSearch
 */
public void sendMessageToEs(String message, String token) throws Exception {
    IndexRequest indexRequest = new IndexRequest(index, type, token).source(message, XContentType.JSON);
    IndexResponse indexResponse = client.index(indexRequest);
    System.out.println(indexResponse.toString());
}
```
