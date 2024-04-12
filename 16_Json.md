# Json

# 基础

## 注释

```json
//
/* */
```



## 基本语法

json的结构为键值对结构

```json
"name" : value
```



### 符号

```json
{}	//对象
[]	//数组
:	//键值对对应关系
,	//数据分割
""	//键、字符串
```



```json
{
    "name" : "xxx",
    "age" : 10,
    "sex" : true,
    "money" : 1.5
    
    //数组
    "nums" : [1,2,3,4],
	
	//List
	"friends" : [{"name":"A", "age":10, "sex":false},
                 {"name":"B", "age":10, "sex":false}],
	//字典
	"dic":{"1":"123", "2":"456"}
}
```



# JsonUtility

继续使用之前的类

```C#
public class Data
{
    public Info info;
    public Data() { }
}

[Serializable]
public class Info
{
    public int id;
    public float val;
    public string name;
    
    public Info() { }

    public Info(int id, float val, string name)
    {
        this.id = id;
        this.val = val;
        this.name = name;
    }
}
```

序列化

```C#
Data data = new Data();
string path = Application.dataPath + "/jsonTest.json";
string s = JsonUtility.ToJson(data);
File.WriteAllText(path, s);
```

反序列化

```C#
Data data = new Data();
string path = Application.dataPath + "/jsonTest.json";
string s = File.ReadAllText(path);
data = JsonUtility.FromJson<Data>(s);
```



# LitJson

序列化

```C#
Data data = new Data();
data.info = new Info(1, 10, "123");
string path = Application.dataPath + "/jsonTest.json";
string s = JsonMapper.ToJson(data);
File.WriteAllText(path, s);
```



反序列化

```C#
Data data = new Data();
string path = Application.dataPath + "/jsonTest.json";
if (File.Exists(path))
{
    string s = File.ReadAllText(path);
    data = JsonMapper.ToObject<Data>(s);
}
```

注意：使用字典是，键需要使用string类型，然后再转换为其他类型

注意：类需要有无参构造方法，否则不能被反序列化







