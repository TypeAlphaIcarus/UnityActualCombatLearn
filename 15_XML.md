# XML

# 基础

XML结构是一种树形的结构

## 注释

```xml
<!---->
<!--注释-->
```



## 固定内容

第一行必须写

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--使用xml的版本和编码-->
```



## 基本语法

```xml
<Tag>xxx</Tag>
<!--<标签>元素</标签>-->
<PlayerInfo>
    <name>xxx</name>
    <health>10</health>
    <ItemList>
        <Item>
            <id>1</id>
            <num>10</num>
        </Item>
        <Item>
            <id>2</id>
            <num>5</num>
        </Item>
        <Item>
            <id>3</id>
            <num>2</num>
        </Item>
    </ItemList>
</PlayerInfo>
```



## 符号

```xml
&lt <!-- < -->
&gt <!-- > -->
&amp <!-- & -->
&apos <!-- ' -->
&quot <!-- " -->
```



# XML属性

```xml
<!--属性是在标签之后添加的内容，必须使用""和''包裹-->
<Enemy name = "name" age = '8'>xxx</Name>

<!--只使用属性记录信息，不使用元素记录-->
<Friend name = "name" age = '8'/>
```

如之前Item的信息可以修改为

```xml
<ItemList>
    <Item id = "1" num = "5"/>
    <Item id = "2" num = "10"/>
    <Item id = "3" num = "2"/>
</ItemList>
```

属性可以添加任意个

```xml
<Item id = "1" num = "5" C = "x" D = "x" E = "x" .../>
```



# C#读取XML

存储位置

1、只读的xml，存放在`Resources`或`StreamingAssets`下

2、动态存储的xml，存放在`Application.persistantDataPath`下，任意平台可读写



读取方式

C#读取xml的方式如下：

1、XmlDocument，将数据加载到内存中进行读取

2、XmlTextReader，以流的形式加载，内存占用更少，只能单向读取，非特殊情况，不使用

3、Linq

有一个xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Root>
    <name>Ikaros</name>
    <age>18</age>
    <ItemList>
        <Item id="1" num="5" />
        <Item id="2" num="10" />
        <Item id="3" num="2" />
    </ItemList>
    <Friend>
        <name>ToMoKi</name>
        <age>15</age>
    </Friend>
    <Friend>
        <name>Nyphm</name>
        <age>15</age>
    </Friend>
</Root>
```

使用`XmlDocument`读取

```C#
XmlDocument xmlDocument = new XmlDocument();

TextAsset textAsset = Resources.Load<TextAsset>("xmlTest");
xmlDocument.LoadXml(textAsset.text);

//xmlDocument.Load(Application.streamingAssetsPath + "/xmlTest.xml");

//根据名称，获取根节点
XmlNode rootNode = xmlDocument.SelectSingleNode("Root");

//通过根节点，获取子节点
XmlNode nameNode = rootNode.SelectSingleNode("name");

//获取节点元素
string name = nameNode.InnerText;

//获取节点属性
XmlNode itemListNode = rootNode.SelectSingleNode("ItemList");
XmlNodeList itemNodeList = itemListNode.SelectNodes("Item");
foreach (XmlNode xmlNode in itemListNode)
{
    Debug.Log(xmlNode.Attributes["id"].Value);
    Debug.Log(xmlNode.Attributes.GetNamedItem("num").Value);
}
```



# C#存储XML

存储在`Application.persistantDataPath`，任意平台可读写

```C#
string path = Application.persistentDataPath + "/xmlTest.xml";

XmlDocument xmlDocument = new XmlDocument();

//添加固定版本信息
XmlDeclaration xmlDeclaration = xmlDocument.CreateXmlDeclaration("1.0", "UTF-8", "");

//将版本信息添加进xmlDocument
xmlDocument.AppendChild(xmlDeclaration);

//添加根节点
XmlElement root = xmlDocument.CreateElement("Root");
xmlDocument.AppendChild(root);

//添加子节点
XmlElement name = xmlDocument.CreateElement("name");
name.InnerText = "Ikaros";  //包裹元素信息
root.AppendChild(name);

XmlElement age = xmlDocument.CreateElement("age");
name.InnerText = "10";  //包裹元素信息
root.AppendChild(age);

//添加多个元素
XmlElement NumList = xmlDocument.CreateElement("ItemList");
for (int i = 0; i < 3; i++)
{
    XmlElement num = xmlDocument.CreateElement("num");
    num.InnerText = (i + 1).ToString();
    NumList.AppendChild(num);
}
root.AppendChild(NumList);

//添加多个属性
XmlElement itemList = xmlDocument.CreateElement("ItemList");
for (int i = 0; i < 3; i++)
{
    XmlElement item = xmlDocument.CreateElement("Item");
    item.SetAttribute("id", i.ToString());
    item.SetAttribute("num", (i * 10).ToString());
    itemList.AppendChild(item);
}
root.AppendChild(itemList);

//存储
xmlDocument.Save(path);
```



# 修改XML文件

```C#
string path = Application.persistentDataPath + "/xmlTest.xml";
if (File.Exists(path))
{
    string xml = File.ReadAllText(path);
    XmlDocument xmlDocument = new XmlDocument();
    xmlDocument.LoadXml(xml);
    //xmlDocument.Load(path);

    //移除
    //XmlNode xmlNode = xmlDocument.SelectSingleNode("Root").SelectSingleNode("age");
    XmlNode root = xmlDocument.SelectSingleNode("Root");
    XmlNode xmlNode = xmlDocument.SelectSingleNode("Root/age");
    root.RemoveChild(xmlNode);

    //添加
    XmlElement xmlElement = xmlDocument.CreateElement("money");
    xmlElement.InnerText = "100";
    root.AppendChild(xmlElement);
    xmlDocument.Save(path);

    //修改
    XmlNode xmlNode1 = xmlDocument.SelectSingleNode("Root/money");
    xmlNode1.InnerText = "15";
    xmlDocument.Save(path);
}
```



# XML序列化

序列化：将对象转化为可传输字节序列的过程

反序列化：将字节序列还原为对象的过程

声明新类型

```C#
public class Data
{
    public Info info;
    public Data() { }
}

public class Info
{
    
    [XmlAttribute("name")]public int id;	//标记为[XmlAttribute]的成员将以属性存储，name可修改存储的名称
    [XmlElement("name")]public float val;	//[XmlElement("name")]将以节点存储，name可修改存储的名称
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



## xml序列化

```C#
Data data = new Data();
data.info = new Info(1, 1.0f, "1");

//string path = Application.persistentDataPath + "/XMLTest.xml";
string path = Application.dataPath + "/Xml/XmlTest.xml";

//写入文件流，有该文件，打开并修改，没有该文件，创建新文件
//using会在文件流写入结束后释放，一般用于读写或大内存操作
using (StreamWriter sw = new StreamWriter(path))
{
    //声明xml序列化对象，需要指定类型
    XmlSerializer xml = new XmlSerializer(typeof(Data));

    //将info1进行序列化，写入文件流
    xml.Serialize(sw, data);
}
```

结果

```xml
<?xml version="1.0" encoding="utf-8"?>
<Data xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <info id="1">
    <val>1</val>
    <name>1</name>
  </info>
</Data>
```

注意：信息中只有`public`声明的类型会被序列化，`private、protectd、internal`均不会被序列化

注意：引用类型如果值为空，不会序列化

注意：`XmlSerializer`不支持字典的序列化



## xml反序列化

```C#
Data data = new Data();
string path = Application.dataPath + "/Xml/XmlTest.xml";

if (File.Exists(path))
{
    using (StreamReader sr = new StreamReader(path))
    {
        XmlSerializer xml = new XmlSerializer(typeof(Data));
        data = xml.Deserialize(sr) as Data;
    }
}
```



## IXmlSerializable

让之前的类继承`IXmlSerializable`

```C#
public class Info : IXmlSerializable
{
    ...
    //返回结构
    public XmlSchema GetSchema()
    {
        return null;
    }

    //反序列化
    public void ReadXml(XmlReader reader)
    {
        //读取属性
        this.id = int.Parse(reader["Id"]);

        //读取节点
        //每次读取，读取的是<>的内容<>value</>需要读取三次，其中的节点内容也需要读取一次
        reader.Read();
        reader.Read();
        this.val = float.Parse(reader.Value);
        reader.Read();
        reader.Read();
        reader.Read();
        this.name = reader.Value;
        
        //方法二循环读取节点
        while (reader.Read())
        {
            if (reader.NodeType == XmlNodeType.Element)
            {
                //判断<>中的名称，EndElement是判断</>的名称
                switch (reader.Name)
                {
                    case "Value":
                        reader.Read();  //读取<>后面的值
                        this.val = float.Parse(reader.Value);
                        break;

                    case "Name":
                        reader.Read();
                        this.name = reader.Value;
                        break;
                }
            }
        }
        
        //读取包裹节点
        XmlSerializer xml = new XmlSerializer(typeof(int));
        reader.Read();  //跳过根节点
        reader.ReadStartElement("Id");
        id = (int)xml.Deserialize(reader);
        reader.ReadEndElement();
    }

    //序列化
    public void WriteXml(XmlWriter writer)
    {
        //自定义序列化的规则，需要使用XmlWriter的方法

        //以属性写入元素a
        writer.WriteAttributeString("Id", this.id.ToString());

        //以节点写入
        writer.WriteElementString("value", this.val.ToString());
        writer.WriteElementString("Name", this.name);
        
        //写包裹节点
        XmlSerializer xmlSerializer = new XmlSerializer(typeof(int));
        
        //开始包裹
        writer.WriteStartElement("xxx");

        //在中间编写的内容会被包裹

        //直接序列化一个可序列化的对象，不必再重复编写
        xmlSerializer.Serialize(writer, this.id);

        //结束包裹
        writer.WriteEndElement();
    }
}
```



## 序列化字典

新建类，继承`Dictionary`

```C#
public class SerializableDictionary<TKey, TValue> : Dictionary<TKey, TValue>, IXmlSerializable
{
    public XmlSchema GetSchema()
    {
        return null;
    }

    public void ReadXml(XmlReader reader)
    {
        XmlSerializer keyXmlSer = new XmlSerializer(typeof(TKey));
        XmlSerializer valXmlSer = new XmlSerializer(typeof(TValue));

        reader.Read();

        //因为所有键值对都包括在<Dic></Dic>中，读取到</Dic>才结束
        //每次读取一行，不会读取到每行键值对的的</>
        while (reader.NodeType != XmlNodeType.EndElement)
        {
            TKey key = (TKey)keyXmlSer.Deserialize(reader);
            TValue val = (TValue)valXmlSer.Deserialize(reader);
            this.Add(key, val);
        }
    }

    public void WriteXml(XmlWriter writer)
    {
        XmlSerializer keyXmlSer = new XmlSerializer(typeof(TKey));
        XmlSerializer valXmlSer = new XmlSerializer(typeof(TValue));
        foreach (KeyValuePair<TKey, TValue> keyValuePair in this)
        {
            //直接对键值对进行序列化，将键值对写入writer
            keyXmlSer.Serialize(writer, keyValuePair.Key);
            valXmlSer.Serialize(writer, keyValuePair.Value);
        }
    }
}
```





# XML管理类

```C#
public class XmlDataManager
{
    private static XmlDataManager instance = new XmlDataManager();
    public static XmlDataManager Instance => instance;
    private XmlDataManager() { }

    public void SaveData(object data, string fileName)
    {
        string path = Application.persistentDataPath + "/" + fileName + ".xml";
        using (StreamWriter sw = new StreamWriter(path))
        {
            XmlSerializer xml = new XmlSerializer(data.GetType());
            xml.Serialize(sw, data);
        }
    }

    public object LoadData<T>(string fileName)
    {
        string path = Application.persistentDataPath + "/" + fileName + ".xml";
        if (!File.Exists(path))
        {
            path = Application.streamingAssetsPath + "/" + fileName + ".xml";
            if (!File.Exists(path))
            {
                return Activator.CreateInstance(typeof(T));
            }
        }

        using (StreamReader sr = new StreamReader(path))
        {
            XmlSerializer xml = new XmlSerializer(typeof(T));
            return xml.Deserialize(sr);
        }

    }
}
```



