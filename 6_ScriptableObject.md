# ScriptableObject

# 概念

ScriptableObject是unity提供的数据配置存储基类，可以用于保存大量的数据，可以自定义数据资源文件

主要作用：

1、数据复用，多个对象使用同一数据，如同一类型的敌人，子弹的数据等

2、配置文件，配置游戏的数据，如玩家的基础属性、音量大小等

3、编辑器模式下的数据持久化

例子：如果为子弹预制体添加脚本，每次实例化子弹，生成新的子弹对象，都会都会分配一次内存，使用ScriptableObject可以避免内存浪费，只分配一次内存，然后一直复用



# 创建

自定义`ScriptableObject`

通过Asset Menu创建

```C#
//fileName为默认的文件名称，MyData为创建该资源的路径，order为菜单页面的顺序
[CreateAssetMenu(fileName = "MyData", menuName = "MyData", order = 0)]
public class MyData : ScriptableObject
{
    public int i;
    public float f;
    public string s;
}
```

通过ScriptableObject的静态方法创建

```C#
public class ScriptableObjectTool
{
    [MenuItem("ScriptableObject/Create MyData")]
    public static void CreateMyData()
    {
        MyData myData = ScriptableObject.CreateInstance<MyData>();

        //根据对象，创建文件
        AssetDatabase.CreateAsset(myData, "Assets/Data/MyData.asset");  //需要添加后缀

        //保存文件
        AssetDatabase.SaveAssets();

        //刷新界面
        AssetDatabase.Refresh();
    }
}
```



# 使用

```C#
public class Test : MonoBehaviour
{
    public MyData myData;	//直接在Inspector窗口关联，多个对象关联同一ScriptableObject，关联的是同一个对象

    private void Start()
    {
        myData = Resources.Load<MyData>("Path");	//通过Resources加载，AB包、Addressable都支持加载ScriptableObject
    }
}
```



# 生命周期

```C#
public class MyData : ScriptableObject
{
    ...
    private void Awake()
    {...}
    
    //创建或加载对象时使用
    private void OnEnable()
    {...}
    
    //对象销毁或重新加载脚本程序集时调用
    private void OnDisable()
    {...}
    
    //对象销毁时调用
    private void OnDestroy()
    {...}
    
    //只会在编辑器中调用，当加载脚本，或数值改变时调用
    private void OnValidate()
    {...}
}
```



# 非持久数据

编辑器和发布后，都不会持久化的数据

利用ScriptableObject的静态方法`CreateInstance<>()`创建对象，该对象创建在内存中，可以被GC

```C#
MyData myData = ScriptableObject.CreateInstance<MyData>();	//类似使用new创建，但ScriptableObject不能new，只能用CreateInstance
MyData myData = ScriptableObject.CreateInstance("MyData") as MyData;	
```



# 持久化ScriptableObject

使用Json结合ScriptableObject存储数据

```C#
public class Test : MonoBehaviour
{
    public string path;
    private MyData myData;
    private void Start()
    {
        //persistentDataPath在windows上位于AppData-LocalLow
        path = Application.persistentDataPath + "/MyData.json";

        //读取
        //Directory.Exists(path)判断是否有文件夹,File.Exists(path)判断是否有文件(任意后缀)
        if (File.Exists(path))
        {
            string json = File.ReadAllText(path);
            JsonUtility.FromJsonOverwrite(json, myData);    //将解析出的数据赋值给myData变量
        }
        else
        {
            myData = ScriptableObject.CreateInstance<MyData>();
        }

        //存储
        string str = JsonUtility.ToJson(myData);
        File.WriteAllText(path, str);
    }
}
```



# 配置数据

```C#
[CreateAssetMenu(fileName = "CharacterData", menuName = "Create CharacterData")]
public class CharacterData : ScriptableObject
{
    public List<Characterinfo> characterDataList;
}

[Serializable]
public class Characterinfo
{
    public int id;
}
```



# 复用数据

不同对象使用相同数据时，ScriptableObject可以复用节约内存



# 数据的多态行为

随机音效、物品拾取音效、AI的不同行为模式，物品的buff等

```C#
public abstract class AudioPlayBase : ScriptableObject
{
    public abstract void Play(AudioSource audioSource);
}

//播放随机音效，可以创建多个数据文件，如不同枪械的音效，移动的音效等
[CreateAssetMenu()]
public class RandomAudioPlayer : AudioPlayBase
{
    public List<AudioClip> audioClipList;

    public override void Play(AudioSource audioSource)
    {
        if (audioClipList.Count == 0)
            return;

        audioSource.clip = audioClipList[Random.Range(0, audioClipList.Count)];
        audioSource.Play();
    }
}

public class Test : MonoBehaviour
{
    public RandomAudioPlayer randomAudioPlayer;	//不同的类，绑定不同的音效数据文件即可
    private AudioSource audioSource;
    
    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }
    
    private void Start()
    {
        randomAudioPlayer.Play(audioSource);
    }
}
```



# 单例模式获取数据

经常使用，并且要复用的数据，如配置文件，使用单例模式获取，非常方便

```C#
public class SingletonScriptableObjectBase<T> : ScriptableObject where T : ScriptableObject
{
    private static T instance;
    public static T Instance
    {
        get
        {
            //为空，加载对应的资源文件
            if (instance == null)
            {
                //自定义规则:
                //1、所有资源文件都放在Resources/ScriptableObject文件夹下
                //2、需要复用的唯一的资源文件名称与类名相同
                instance = Resources.Load<T>("ScriptableObject/" + typeof(T).Name);
            }

            //再次处理，如果没有加载文件
            if (instance == null)
            {
                //创建默认数据
                instance = ScriptableObject.CreateInstance<T>();
                
                //throw new ArgumentException("没有对应资源文件");
            }

            //从json中读取也可以，但不建议用ScriptableObject来做数据持久化

            return instance;
        }
    }
}
```

继承该基类的数据资源，可以用过单例的Instance获取

```C#
[CreateAssetMenu()]
public class TestData : SingletonScriptableObjectBase<TestData>
{
    public List<TestInfo> testInfoList;
}

[Serializable]
public class TestInfo
{
    public int i;
}

public class Test : MonoBehaviour
{
    private void Start()
    {
        TestInfo testInfo = TestData.Instance.testInfoList[0];
    }
}
```









