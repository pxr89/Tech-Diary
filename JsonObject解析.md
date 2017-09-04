#JSONObject源码分析-构建有序的Json输出
##1.JSONObject是乱序的？
先来看一段代码:

```
  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        printJsonObject(new JSONObject());
    }

    void printJsonObject(JSONObject test) {
        try {
            test.put("command", "11121");
            test.put("custominfo", "http");
            test.put("name", "http");
            test.put("age", "http");
            test.put("hobby", "http");
            test.put("garb", "http");
            Log.e("test-----", test.toString());
        } catch (JSONException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
```
在4.4手机及以下，我们看到的日志输出如下：是乱序的
```
09-22 13:47:48.714: E/test-----(622): {"hobby":"http","command":"11121","garb":"http","custominfo":"http","age":"http","name":"http"}
```

但是在5.0的手机上，日志输出如下：是有序的

##2.源码查看：
5.0之前jsonobject存储使用的是hashmap，5.0之后使用的是linkedhashmap~


##3.JSONObject的顺序输出解决方案
###1.判断sdk版本，5.0之前反射修改jsonobject的成员 即可
 
 	5.0之后
 	private final LinkedHashMap<String, Object> nameValuePairs;
