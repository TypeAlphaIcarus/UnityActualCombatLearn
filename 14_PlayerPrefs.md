# PlayerPrefs

### 基本功能

```C#
//PlayerPrefs可以存储int、float、string三种类型
//存储的方式类似键值对，键为string类型
PlayerPrefs.SetInt("id", 1);
PlayerPrefs.SetFloat("money", 12.5f);
PlayerPrefs.SetString("name", "hello");

PlayerPrefs.Save();	//立即存储数据

//参数二为没有获取到赋值的默认值
int id = PlayerPrefs.GetInt("id", 0);
float money = PlayerPrefs.GetFloat("money", 100f);
string name = PlayerPrefs.GetString("name", "");

//数据是否存在
if(PlayerPrefs.HasKey("id"))
{...}

//删除数据
PlayerPrefs.DeleteKey("id");
PlayerPrefs.DeleteAll();
```

Unity结束时，存储的值，将会存储到注册表中，游戏崩溃，值会丢失



### 存储位置

Windows：HKCU\Softwate\公司名称\产品名称\ 下的注册表中

安卓：/data/data/包名/shared_prefs/pkg-name.xml

IOS：/Library/Preferences/应用id.plist



### 快速存储任意数据

#### 反射补充

```C#
//判断一个类型对象是否能让另一个类型对象为自己分配控件
//即，父类装子类
public class Father
{}
public class Son : Father
{}

Type fatherType = typeof(Father);
Type sonType = typeof(Son);

//判断类型是否能为自己分配空间
if(fatherType.IsAssignableFrom(sonType))
{
    Father f = Activator.CreateInstance(sonType) as Father;
}

//通过反射获取泛型类型
List<string> list = new List<string>();
Type listType = list.GetType();
Type[] typeArray = listType.GetGenericArguments();	//获取泛型参数
```



#### 数据管理类

对常用数据进行存储

```C#
public class PlayerPrefsDataManager
{
    private static PlayerPrefsDataManager instance = new PlayerPrefsDataManager();

    public static PlayerPrefsDataManager Instance
    {
        get { return instance; }
    }
    private PlayerPrefsDataManager() { }

    /// <summary>
    /// 存储数据
    /// </summary>
    /// <param name="data">数据对象</param>
    /// <param name="keyName">数据名称</param>
    public void SaveData(object data, string keyName)
    {
        Type type = data.GetType();
        FieldInfo[] infoArray = type.GetFields(); //获取传入对象的所有字段(反射)

        string saveKeyName = "";
        FieldInfo info;
        //遍历所有字段
        for (int i = 0; i < infoArray.Length; i++)
        {
            info = infoArray[i];
            //对每一个字段进行存储
            saveKeyName = keyName + "_" + type.Name + "_" + info.FieldType.Name + "_" + info.Name;   //key名称_数据类型_字段类型_字段名称

            //通过PlayerPrefs存储
            object value = info.GetValue(data);    //获取字段的值
            SetValue(value, saveKeyName);
        }
        PlayerPrefs.Save();
    }

    /// <summary>
    /// 存储具体的数据值
    /// </summary>
    private void SetValue(object value, string keyName)
    {
        //判断传入数据的类型，为单个字段
        Type type = value.GetType();
        if (type == typeof(int))
        {
            PlayerPrefs.SetInt(keyName, (int)value);
        }
        else if (type == typeof(float))
        {
            PlayerPrefs.SetFloat(keyName, (float)value);
        }
        else if (type == typeof(string))
        {
            PlayerPrefs.SetString(keyName, value.ToString());
        }
        else if (type == typeof(bool))
        {
            //PlayerPrefs不能存储bool，需要进行转换
            PlayerPrefs.SetInt(keyName, (bool)value ? 1 : 0);   //将bool转为int存储，true存1，false存0
        }
        //存储List<>类型，所有的List<>都实现了IList接口
        else if (typeof(IList).IsAssignableFrom(type))  //判断type是否为IList子类，即type是否为List<>
        {
            //用父类引用子类
            IList list = value as IList;
            //存储List<>中元素的数量
            PlayerPrefs.SetInt(keyName, list.Count);
            int index = 0;
            foreach (object obj in list)
            {
                SetValue(obj, keyName + index);    //递归调用，判断List<>的泛型类型
                ++index;
            }
        }
        //存储Dictionary<,>类型，所有字典类型继承了IDictionary
        else if (typeof(IDictionary).IsAssignableFrom(type))
        {
            IDictionary dict = value as IDictionary;
            PlayerPrefs.SetInt(keyName, dict.Count);
            int index = 0;
            //遍历字典的key
            foreach (object key in dict.Keys)
            {
                SetValue(key, keyName + "_key_" + index);
                SetValue(dict[key], keyName + "_value_" + index);
                ++index;
            }
        }
        //不是基础类型，那可能是自定义类型
        else
        {
            //递归调用方法，再遍历自定义类型的所有成员变量，判断其类型并进行存储
            SaveData(value, keyName);
        }
    }

    /// <summary>
    /// 读取数据
    /// </summary>
    public object LoadData(Type type, string KeyName)
    {
        //根据Type创建对象，用于返回
        object data = Activator.CreateInstance(type);

        //获取所有字段信息
        FieldInfo[] infoArray = type.GetFields();
        //用于获取单个字段
        FieldInfo fieldInfo;
        //用于拼接key的字符串
        string loadKeyName = "";
        for (int i = 0; i < infoArray.Length; i++)
        {
            fieldInfo = infoArray[i];
            loadKeyName = KeyName + "_" + type.Name + "_" + fieldInfo.FieldType.Name + "_" + fieldInfo.Name;

            //将获取到的值赋值给新创建的对象的单个字段
            fieldInfo.SetValue(data, GetValue(fieldInfo.FieldType, loadKeyName));
        }

        return data;
    }

    /// <summary>
    /// 获取单个数据的值
    /// </summary>
    private object GetValue(Type type, string keyName)
    {
        if (type == typeof(int))
        {
            return PlayerPrefs.GetInt(keyName);
        }
        else if (type == typeof(float))
        {
            return PlayerPrefs.GetFloat(keyName);
        }
        else if (type == typeof(string))
        {
            return PlayerPrefs.GetString(keyName);
        }
        else if (type == typeof(bool))
        {
            return PlayerPrefs.GetInt(keyName) == 1 ? true : false;
        }
        else if (typeof(IList).IsAssignableFrom(type))
        {
            //获取数据长度
            int length = PlayerPrefs.GetInt(keyName, 0);
            //实例化List<>对象，用于存储数据
            IList list = Activator.CreateInstance(type) as IList;
            for (int i = 0; i < length; i++)
            {
                //需要获取List中泛型的类型
                list.Add(LoadData(type.GetGenericArguments()[0], keyName + i));
            }
            return list;
        }
        else if (typeof(IDictionary).IsAssignableFrom(type))
        {
            int count = PlayerPrefs.GetInt(keyName, 0);
            IDictionary dictionary = Activator.CreateInstance(type) as IDictionary;
            //获取键值对的泛型类型
            Type[] keyValueType = type.GetGenericArguments();
            for (int i = 0; i < count; i++)
            {
                dictionary.Add(LoadData(keyValueType[0], keyName + "_key_" + i),
                    LoadData(keyValueType[1], keyName + "_value_" + i));
            }
            return dictionary;
        }
        else
        {
            //递归读取自定义类型
            return LoadData(type, keyName);
        }
    }
}
```
