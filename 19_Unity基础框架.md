# Unity基础框架

# 单例模式 基类模块

不继承`MonoBehavior`

```C#
public class BaseManager<T> where T : new()
{
    private static T instance;
    public static T GetInstance()
    {
        if (instance == null)
            instance = new T();
        return instance;
    }
}
```

注意：不继承`MonoBehavior`即使切换场景也不会销毁



继承`MonoBehavior`

```C#
public class SingletonMono<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    public static T GetInstance()
    {
        if (instance == null)
        {
            GameObject obj = new GameObject();
            obj.name = typeof(T).ToString();
            GameObject.DontDestroyOnLoad(obj);
            instance = obj.AddComponent<T>();
        }
        return instance;
    }

    protected virtual void Awake()
    {
        instance = this as T;
    }
}
```



# 缓存池

创建和销毁对象会造成性能开销，频繁GC会造成卡顿

开辟一部分内存或创建一定数量的对象进行重复使用

```C#
public class PoolData
{
    //对象池中所有对象的父节点
    public GameObject parentObj;
    public Queue<GameObject> queue;
    public int Count => queue.Count;
    public PoolData(GameObject obj, GameObject root)
    {
        parentObj = new GameObject(obj.name);
        parentObj.transform.parent = root.transform;

        queue = new Queue<GameObject>() { };
        EnqueueObj(obj);
    }

    //将不用的物体放入对象池
    public void EnqueueObj(GameObject obj)
    {
        queue.Enqueue(obj);
        obj.transform.parent = parentObj.transform;
        obj.SetActive(false);
    }

    //从对象池中取出物体
    public GameObject DequeueObj()
    {
        GameObject obj = null;
        obj = queue.Dequeue();

        obj.SetActive(true);
        obj.transform.parent = null;

        return obj;
    }
}

public class PoolManager : BaseManager<PoolManager>
{
    public Dictionary<string, PoolData> poolDic = new Dictionary<string, PoolData>();

    private GameObject poolObj;

    //从缓存池中获取对象
    public GameObject GetObj(string name)
    {
        GameObject obj = null;
        if (poolDic.ContainsKey(name) && poolDic[name].Count > 0)
        {
            obj = poolDic[name].DequeueObj();
        }
        else
        {
            //没有对应物体的缓存池或缓存池用光，生成新的物体放入缓存池
            obj = GameObject.Instantiate(Resources.Load<GameObject>("name"));
            obj.name = name;    //实例化的物体会带(Clone)后缀，重新命名除去后缀
        }

        return obj;
    }

    //将暂时没有使用的东西重新放入对应的缓存池
    public void EnqueueObj(string name, GameObject obj)
    {
        if (poolObj == null)
        {
            poolObj = new GameObject("Pool");
        }

        //没有对应的缓存池，新建缓存池并将物体放入，新建的缓存池父物体为根节点
        if (!poolDic.ContainsKey(name))
        {
            poolDic.Add(name, new PoolData(obj, poolObj));
        }
        poolDic[name].EnqueueObj(obj);
    }

    /// <summary>
    /// 清空缓存池，一般用于切换场景时
    /// </summary>
    public void Clear()
    {
        poolDic.Clear();
        poolObj = null;
    }
}
```



# 事件中心

可以减小耦合，降低复杂性

采用观察者模式

事件的触发者不必关心有多少对象对自身进行监听，即使自身销毁也无所谓，只需要告诉事件中心触发事件即可

事件的监听者不需要获取监听的对象，只需要在事件中心监听某个事件以及传入事件触发的委托即可，可以有多个监听者监听同一事件

```C#
public class EventCenter : BaseManager<EventCenter>
{
    //用于存放事件，key为事件名称，value为监听该事件的所有对应的委托
    //如key怪物死亡，value为玩家升级，UI刷新
    //UnityAction<object>用于传递参数，使用object可以传递任意参数，提高通用性
    private Dictionary<string, UnityAction<object>> eventDic = new Dictionary<string, UnityAction<object>>();

    /// <summary>
    /// 添加对事件的监听，例如玩家对怪物死亡进行监听，触发怪物死亡后升级
    /// </summary>
    /// <param name="eventName">事件名称</param>
    /// <param name="action">触发事件执行的委托</param>
    public void AddEventListener(string eventName, UnityAction<object> action)
    {
        //判断有没有对应的事件监听
        if (!eventDic.ContainsKey(eventName))
        {
            //没有，添加对事件的监听，如eventName怪物死亡，action玩家升级，UI刷新
            eventDic.Add(eventName, action);
        }
        else
        {
            //有，添加委托，如在玩家升级，UI刷新的基础上再添加获取金币
            eventDic[eventName] += action;
        }

    }

    /// <summary>
    /// 移除事件监听
    /// </summary>
    /// <param name="eventName">事件名称</param>
    /// <param name="action">移除的委托</param>
    public void RemoveEventListener(string eventName, UnityAction<object> action)
    {
        //判断有没有对应的事件监听
        if (eventDic.ContainsKey(eventName))
            eventDic[eventName] -= action;
    }

    /// <summary>
    /// 清空所有事件
    /// 场景切换时使用，因为该管理器没有继承MonoBehavior，切换场景时不会销毁，需要手动清空
    /// </summary>
    public void Clear()
    {
        eventDic.Clear();
    }

    /// <summary>
    /// 触发事件/派发事件
    /// </summary>
    /// <param name="eventName">事件名称</param>
    /// <param name="info">传递的参数</param>
    public void EventTrigger(string eventName, object info)
    {
        //触发事件后，执行所有委托
        if (eventDic.ContainsKey(eventName))
        {
            eventDic[eventName]?.Invoke(info);
        }
    }
}
```

例子：

```C#
public enum E_MonsterType
{
    Goblin,
    Slim
}

public class Monster : MonoBehaviour
{
    protected E_MonsterType monsterType;
    [SerializeField] protected float exp;
    
    public E_MonsterType MonsterType => monsterType;
    public float Exp => exp;
}
```

```C#
public class Goblin : Monster
{
    void Start()
    {
        monsterType = E_MonsterType.Goblin;
        EventCenter.GetInstance().EventTrigger("OnMonsterDead", this);
    }
}
```

```C#
public class Slim : Monster
{
    void Start()
    {
        monsterType = E_MonsterType.Slim;
        EventCenter.GetInstance().EventTrigger("OnMonsterDead", this);
    }
}
```

```C#
public class Player : MonoBehaviour
{
    void OnEnable()
    {
        EventCenter.GetInstance().AddEventListener("OnMonsterDead", AddExp);
    }

    private void OnDisable()
    {
        EventCenter.GetInstance().RemoveEventListener("OnMonsterDead", AddExp);
    }

    //info为触发事件时传递的参数
    private void AddExp(object info)
    {
        Debug.Log($"消灭{(info as Monster).MonsterType}, 获取经验{(info as Monster).Exp}");
    }
}
```



# 公共Mono

让没有继承Mono的类可以开启协程，可以Update进行更新，来统一管理Update

过多的Update会造成一定的性能开销

```C#
public class MonoController : MonoBehaviour
{
    private event UnityAction updateEvent;

    void Start()
    {
        DontDestroyOnLoad(gameObject);
    }

    // Update is called once per frame
    void Update()
    {
        //每帧触发事件，调用所有监听者的委托
        updateEvent?.Invoke();
    }

    /// <summary>
    /// 让外部添加帧更新的方法
    /// </summary>
    /// <param name="action"></param>
    public void AddUpdateListener(UnityAction action)
    {
        updateEvent += action;

    }

    /// <summary>
    /// 让外部移除帧更新的方法
    /// </summary>
    /// <param name="action"></param>
    public void RemoveUpdateListener(UnityAction action)
    {
        updateEvent -= action;

    }
}
```

```C#
public class MonoManager : BaseManager<MonoManager>
{
    private MonoController monoController;

    //获取单例时会执行一次构造方法，创建一个新的MonoController，monoController是唯一的
    public MonoManager()
    {
        GameObject obj = new GameObject("MonoController");
        obj.AddComponent<MonoController>();
    }

    //对monoController的方法进行一次封装，方便使用单例进行调用
    public void AddUpdateListener(UnityAction action)
    {
        monoController.AddUpdateListener(action);
    }

    public void RemoveUpdateListener(UnityAction action)
    {
        monoController.RemoveUpdateListener(action);
    }

    //让外部执行协程的方法
    public Coroutine StartCoroutine(IEnumerator routine)
    {
        return monoController.StartCoroutine(routine);
    }

    public Coroutine StartCoroutine(string methodName, [DefaultValue("null")] object value)
    {
        return monoController.StartCoroutine(methodName, value);
    }

    public Coroutine StartCoroutine(string methodName)
    {
        return monoController.StartCoroutine(methodName);
    }

    public void StopCoroutine(IEnumerator routine)
    {
        monoController.StopCoroutine(routine);
    }
    public void StopCoroutine(Coroutine routine)
    {
        monoController.StopCoroutine(routine);
    }
}
```



# 场景切换管理器

```C#

public class ScenesManager : BaseManager<ScenesManager>
{
    public void LoadScene(string sceneName, UnityAction action)
    {
        SceneManager.LoadScene(sceneName);
        action();
    }

    //提供外部调用的异步加载场景方法
    public void LoadSceneAsync(string sceneName, UnityAction action)
    {
        MonoManager.GetInstance().StartCoroutine(CorLoadSceneAsync(sceneName, action));
    }

    //协程异步加载场景
    private IEnumerator CorLoadSceneAsync(string sceneName, UnityAction action)
    {
        AsyncOperation asyncOperation = SceneManager.LoadSceneAsync(sceneName);

        //获取加载进度，0~1，实际0.9时场景就加载完毕
        while (!asyncOperation.isDone)
        {
            //进度更新，使用了之前的事件中心
            EventCenter.GetInstance().EventTrigger("LoadingProgress", asyncOperation.progress);

            yield return asyncOperation.progress;
        }

        ////等待场景加载完毕
        //yield return asyncOperation;

        //场景加载完毕后执行委托
        action?.Invoke();
    }
}
```



# 资源加载管理器

```C#
public class ResourcesManager : BaseManager<ResourcesManager>
{
    public T Load<T>(string fileName) where T : Object
    {
        T res = Resources.Load<T>(fileName);

        //如果对象为GameObject，直接实例化并返回，外部直接使用即可
        if (res is GameObject)
        {
            return GameObject.Instantiate(res);
        }

        //不为GameObject，如json文件，返回对象
        return res;
    }

    //异步加载资源
    public void LoadAsync<T>(string fileName, UnityAction<T> callBack) where T : Object
    {
        MonoManager.GetInstance().StartCoroutine(CorLoadAsync<T>(fileName, callBack));
    }

    //使用协程异步加载，协程无法直接返回资源，需要使用委托返回资源，委托callBack参数为加载的资源
    IEnumerator CorLoadAsync<T>(string fileName, UnityAction<T> callBack) where T : Object
    {
        ResourceRequest request = Resources.LoadAsync<T>(fileName);

        //等待资源加载完成
        yield return request;

        if (request.asset is GameObject)
        {
            //如果资源为GameObject，实例化并返回
            callBack(GameObject.Instantiate(request.asset) as T);
        }
        else
        {
            //资源不是GameObject，直接返回
            callBack(request.asset as T);
        }
    }
}
```

使用

```C#
public class ResourcesTest : MonoBehaviour
{
    private GameObject cube;
    private GameObject sphere;
    void Start()
    {
        ResourcesManager.GetInstance().LoadAsync<GameObject>("Cube", (obj) =>
        {
            cube = obj;
            Debug.Log(cube.name);
        });

        ResourcesManager.GetInstance().LoadAsync<GameObject>("Sphere", LoadResCallBack);
    }

    //加载完成后执行方法，资源将作为委托的参数返回
    private void LoadResCallBack(GameObject obj)
    {
        //obj为异步加载并通过委托返回的资源对象
        sphere = obj;
        Debug.Log(sphere.name);
    }
}
```



# 缓存池优化

```C#
//异步加载不能立即返回，需要在加载完成后再返回，所有需要使用委托
public void GetObj(string name, UnityAction<GameObject> callBack)
{
    if (poolDic.ContainsKey(name) && poolDic[name].Count > 0)
    {
        callBack(poolDic[name].DequeueObj());
    }
    else
    {
        //改为异步加载
        ResourcesManager.GetInstance().LoadAsync<GameObject>(name, (newObj) =>
        {
            newObj.name = name;
            callBack(newObj);
        });
    }
}
```



# 输入控制管理器

结合事件中心和公共Mono，分发输入事件

```C#
public class InputManager : BaseManager<InputManager>
{
    private bool enable;
    public InputManager()
    {
        enable = true;
        MonoManager.GetInstance().AddUpdateListener(Update);
    }

    public void SetActive(bool active)
    {
        enable = active;
    }

    private void CheckKey(KeyCode keyCode)
    {
        if (Input.GetKeyDown(keyCode))
        {
            EventCenter.GetInstance().EventTrigger("GetKeyDown", keyCode);
        }
        if (Input.GetKeyUp(keyCode))
        {
            EventCenter.GetInstance().EventTrigger("GetKeyUp", keyCode);
        }
    }
    private void Update()
    {
        if (!enable)
            return;
        CheckKey(KeyCode.W);
        CheckKey(KeyCode.S);
        CheckKey(KeyCode.A);
        CheckKey(KeyCode.D);
    }
}
```

监听测试

```C#
public class InputTest : MonoBehaviour
{
    private KeyCode keyCode;
    void Start()
    {
        InputManager.GetInstance().SetActive(true);
        EventCenter.GetInstance().AddEventListener("GetKeyDown", Test);
    }

    private void Test(object o)
    {
        keyCode = (KeyCode)o;
        switch (keyCode)
        {
            case KeyCode.W:
                Debug.Log("W");
                break;
            case KeyCode.S:
                Debug.Log("S");
                break;
            case KeyCode.A:
                Debug.Log("A");
                break;
            case KeyCode.D:
                Debug.Log("D");
                break;
        }
    }
}
```



# 事件中心优化 ！

避免事件中心装箱拆箱

将泛型委托作为泛型类的成员变量，再让泛型类继承接口，这样就能使用接口引用任意类型的泛型类

```C#
//利用接口基类来存放子类泛型，里氏替换原则
public interface IEventInfo
{	}

//泛型类继承空接口
public class EventInfo<T> : IEventInfo
{
    //因为UnityAction<T>不能直接继承空接口，所以让其作为成员变量，进行封装
    public UnityAction<T> action;
    public EventInfo(UnityAction<T> action)
    {
        this.action += action;
    }
}

//非泛型类，用于传递无参委托
public class EventInfo : IEventInfo
{
    public UnityAction action;
    public EventInfo(UnityAction action)
    {
        this.action += action;
    }
}

public class EventCenter : BaseManager<EventCenter>
{
    //用空接口来存放泛型类
    private Dictionary<string, IEventInfo> eventDic = new Dictionary<string, IEventInfo>();

    public void AddEventListener<T>(string eventName, UnityAction<T> action)
    {
        if (!eventDic.ContainsKey(eventName))
        {
            //里氏替换，可以直接创建新的泛型类并添加委托，直接传入dic，用IEventInfo来存放
            eventDic.Add(eventName, new EventInfo<T>(action));
        }
        else
        {
            //用里氏替换，将接口引用取出转换为泛型类型，添加委托
            (eventDic[eventName] as EventInfo<T>).action += action;
        }
    }
    
    //无参重载
    public void AddEventListener(string eventName, UnityAction action)
    {
        if (!eventDic.ContainsKey(eventName))
        {
            eventDic.Add(eventName, new EventInfo(action));
        }
        else
        {
            (eventDic[eventName] as EventInfo).action += action;
        }
    }

    public void RemoveEventListener<T>(string eventName, UnityAction<T> action)
    {
        if (eventDic.ContainsKey(eventName))
            //用里氏替换，将接口引用取出转换为泛型类型，移除委托
            (eventDic[eventName] as EventInfo<T>).action -= action;
    }
    
    //无参重载
	public void RemoveEventListener(string eventName, UnityAction action)
    {
        if (eventDic.ContainsKey(eventName))
            (eventDic[eventName] as EventInfo).action -= action;
    }
    
    public void Clear()
    {
        eventDic.Clear();
    }

    public void EventTrigger<T>(string eventName, T info)
    {
        if (eventDic.ContainsKey(eventName))
        {
            if ((eventDic[eventName] as EventInfo<T>) != null)
            {
                (eventDic[eventName] as EventInfo<T>).action?.Invoke(info);
            }
        }
    }
    
    //无参重载
    public void EventTrigger(string eventName)
    {
        if (eventDic.ContainsKey(eventName))
        {
            if ((eventDic[eventName] as EventInfo) != null)
            {
                (eventDic[eventName] as EventInfo).action?.Invoke();
            }
        }
    }
}
```



# 音效管理器

副本、怪物、玩家等等都会使用音效模块

```C#
public class MusicManager : BaseManager<MusicManager>
{
    private AudioSource bgmSource = null;
    private List<AudioSource> soundSourceList = new List<AudioSource>();
    private float bgmVolume = 1;
    private float soundVolume = 1;
    private GameObject globalSoundObj;

    public MusicManager()
    {
        MonoManager.GetInstance().AddUpdateListener(Update);
    }

    private void Update()
    {
        //反向检测，尾部移除可以不调整元素位置或调整更少元素的位置
        for (int i = soundSourceList.Count - 1; i >= 0; --i)
        {
            if (!soundSourceList[i].isPlaying)
            {
                StopSoundEffect(soundSourceList[i]);
            }
        }
    }

    public void PlayBGM(string name)
    {
        if (bgmSource == null)
        {
            GameObject obj = new GameObject();
            obj.name = "BGM";
            bgmSource = obj.AddComponent<AudioSource>();
        }

        StopBGM();
        //异步加载BGM并播放
        ResourcesManager.GetInstance().LoadAsync<AudioClip>("Sound/BGM/" + name, (audioClip) =>
        {
            bgmSource.clip = audioClip;
            bgmSource.volume = bgmVolume;
            bgmSource.loop = true;
            bgmSource.Play();
        });
    }

    public void PauseBGM()
    {
        bgmSource?.Pause();
    }

    public void StopBGM()
    {
        bgmSource?.Stop();
    }

    public void ChangeBGMVolume(float volume)
    {
        if (bgmSource != null)
        {
            bgmVolume = volume;
            bgmSource.volume = bgmVolume;
        }
    }

    //播放音效
    public void PlaySoundEffect(string name, bool loop = false, Transform soundParent = null, UnityAction<AudioSource> callBack = null)
    {
        GameObject obj = new GameObject();
        AudioSource newAudioSource = obj.AddComponent<AudioSource>();

        //有专门挂载AudioSource的对象
        if (soundParent != null)
        {
            obj.transform.parent = soundParent;
        }

        //没有专门挂载音效的对象，全都放到globalSoundObj上
        if (globalSoundObj == null)
        {
            globalSoundObj = new GameObject("Sound");
        }

        obj.transform.parent = globalSoundObj.transform;

        ResourcesManager.GetInstance().LoadAsync<AudioClip>("Sound/SoundEffect/" + name, (clip) =>
        {
            newAudioSource.clip = clip;
            newAudioSource.loop = loop;
            newAudioSource.volume = soundVolume;
            newAudioSource.Play();
            soundSourceList.Add(newAudioSource);

            callBack?.Invoke(newAudioSource);
        });
    }
    public void StopSoundEffect(AudioSource audioSource)
    {
        if (soundSourceList.Contains(audioSource))
        {
            audioSource.Stop();
            soundSourceList.Remove(audioSource);
            GameObject.Destroy(audioSource);
        }
    }

    public void ChangeSoundVolume(float volume)
    {
        soundVolume = volume;
        for (int i = 0; i < soundSourceList.Count; i++)
        {
            soundSourceList[i].volume = soundVolume;
        }
    }
}
```



# UI管理器

UI基类

```C#
public class BasePanel : MonoBehaviour
{
    private Dictionary<string, List<UIBehaviour>> controlDic;
    private void Awake()
    {
        controlDic = new Dictionary<string, List<UIBehaviour>>();
    }
    private void Start()
    {
        FindUIControlsInChildren<Button>();
        FindUIControlsInChildren<Image>();
        FindUIControlsInChildren<Text>();
        FindUIControlsInChildren<Toggle>();
        FindUIControlsInChildren<Slider>();
        FindUIControlsInChildren<ScrollRect>();
    }

    public virtual void Show()
    {

    }

    public virtual void Hide()
    {

    }

    protected T GetControl<T>(string name) where T : UIBehaviour
    {
        if (controlDic.ContainsKey(name))
        {
            for (int i = 0; i < controlDic[name].Count; ++i)
            {
                if (controlDic[name][i] is T)
                {
                    return controlDic[name][i] as T;
                }
            }
        }
        return null;
    }

    private void FindUIControlsInChildren<T>() where T : UIBehaviour
    {
        T[] controls = transform.GetComponentsInChildren<T>();
        string objName;
        for (int i = 0; i < controls.Length; ++i)
        {
            objName = controls[i].gameObject.name;
            if (controlDic.ContainsKey(objName))
            {
                controlDic[objName].Add(controls[i]);
            }
            controlDic.Add(objName, new List<UIBehaviour>() { controls[i] });
        };
    }
}
```



！将Canvas添加几个层级，设置参数，然后设置为预制体

注意修改层级锚点

注意修改Canvas缩放

UI管理器

```C#
public enum E_PanelLayer
{
    Bottom,
    Middle,
    Top,
    System
}

public class UIManager : BaseManager<UIManager>
{
    private Dictionary<string, BasePanel> panelDic = new Dictionary<string, BasePanel>();
    private RectTransform canvasTransform;
    public RectTransform CanvasTransform => canvasTransform;

    //UI面板层级
    private Transform bottom;
    private Transform middle;
    private Transform top;
    private Transform system;

    public UIManager()
    {
        GameObject obj = ResourcesManager.GetInstance().Load<GameObject>("UI/Canvas");
        canvasTransform = obj.GetComponent<RectTransform>();
        GameObject.DontDestroyOnLoad(obj);

        bottom = canvasTransform.Find("Bottom");
        middle = canvasTransform.Find("Middle");
        top = canvasTransform.Find("Top");
        system = canvasTransform.Find("System");
    }

    public Transform GetLayerRoot(E_PanelLayer panelLayer)
    {
        switch (panelLayer)
        {
            case E_PanelLayer.Bottom:
                return bottom;
            case E_PanelLayer.Middle:
                return middle;
            case E_PanelLayer.Top:
                return top;
            case E_PanelLayer.System:
                return system;
        }
        return null;
    }

    public void LoadPanel<T>(string panelName, E_PanelLayer panelLayer = E_PanelLayer.Middle, UnityAction<T> callBack = null) where T : BasePanel
    {
        if (!panelDic.ContainsKey(panelName))
        {
            ResourcesManager.GetInstance().LoadAsync<GameObject>("UI/Panel/" + panelName, (newPanel) =>
            {
                Transform panelParent = bottom;
                panelParent = GetLayerRoot(panelLayer);

                //初始化位置和大小
                newPanel.transform.SetParent(panelParent);
                newPanel.transform.localPosition = Vector3.zero;
                newPanel.transform.localScale = Vector3.one;
                (newPanel.transform as RectTransform).offsetMax = Vector2.zero;
                (newPanel.transform as RectTransform).offsetMin = Vector2.zero;

                //使用回调方法，在加载面板完毕之后可以通过回调获取面板，回调方法参数即为加载完成后的面板
                T panel = newPanel.GetComponent<T>();
                panelDic.Add(panelName, panel);

                panelDic[panelName].Show();
                callBack?.Invoke(panel);
            });
        }
        else
        {
            panelDic[panelName].Show();
            callBack?.Invoke(panelDic[panelName] as T);
        }
    }

    public void UnloadPanel(string panelName)
    {
        if (!panelDic.ContainsKey(panelName))
            return;

        panelDic[panelName].Hide();
        GameObject.Destroy(panelDic[panelName].gameObject);
        panelDic.Remove(panelName);
    }

    public T GetPanel<T>(string name) where T : BasePanel
    {
        if (panelDic.ContainsKey(name))
        {
            return panelDic[name] as T;
        }
        return null;
    }
}
```

注意：如何避免两处同时调用同一面板，面板可能没有加载完成就被调用，会再次加载

尝试上锁



# 优化面板基类事件监听

```C#
public class BasePanel : MonoBehaviour
{
    private Dictionary<string, List<UIBehaviour>> controlDic;
    protected virtual void Awake()
    {
        controlDic = new Dictionary<string, List<UIBehaviour>>();
    }
    protected virtual void Start()
    {
        FindUIControlsInChildren<Button>();
        FindUIControlsInChildren<Image>();
        FindUIControlsInChildren<Text>();
        FindUIControlsInChildren<Toggle>();
        FindUIControlsInChildren<Slider>();
        FindUIControlsInChildren<ScrollRect>();
    }

    public virtual void Show()
    {	}

    public virtual void Hide()
    {	}

    public virtual void FadeIn()
    {	}
    public virtual void FadeOut()
    {	}

    //让所有按钮监听该方法，在点击时传入自身的名称，即可获取是哪个按钮进行点击
    protected virtual void OnClick(string buttonName)
    {	}
    //让所有Toggle监听该方法
    protected virtual void OnValueChanged(string toggleName, bool value)
    {	}
    
    //让所有Slider监听该方法
    protected virtual void OnValueChanged(string sliderName, float value)
    {	}

    protected T GetControl<T>(string name) where T : UIBehaviour
    {
        if (controlDic.ContainsKey(name))
        {
            for (int i = 0; i < controlDic[name].Count; ++i)
            {
                if (controlDic[name][i] is T)
                {
                    return controlDic[name][i] as T;
                }
            }
        }
        return null;
    }

    private void FindUIControlsInChildren<T>() where T : UIBehaviour
    {
        T[] controls = transform.GetComponentsInChildren<T>();

        for (int i = 0; i < controls.Length; ++i)
        {
            string objName = controls[i].gameObject.name;
            if (controlDic.ContainsKey(objName))
            {
                controlDic[objName].Add(controls[i]);
            }
            else
            {
                controlDic.Add(objName, new List<UIBehaviour>() { controls[i] });
            }

            if (controls[i] is Button)
            {
                (controls[i] as Button).onClick.AddListener(() =>
                {
                    OnClick(objName);
                });
            }
            if (controls[i] is Toggle)
            {
                (controls[i] as Toggle).onValueChanged.AddListener((value) =>
                {
                    OnValueChanged(objName, value);
                });
            }
            if (controls[i] is Slider)
            {
                (controls[i] as Slider).onValueChanged.AddListener((value) =>
                {
                    OnValueChanged(objName, value);
                });
            }
        };
    }
}

```

例子：

```C#
public class LoginPanel : BasePanel
{
    protected override void OnClick(string buttonName)
    {
        switch (buttonName)
        {
            case "btnLogin":
                Debug.Log("Login");
                break;
            case "btnQuit":
                Debug.Log("Quit");
                break;
        }
    }

    protected override void OnValueChanged(string toggleName, bool value)
    {
        switch (toggleName)
        {
            case "togMusic":
                Debug.Log("IsOpenMusic: " + value);
                break;
        }
    }

    protected override void OnValueChanged(string sliderName, float value)
    {
        switch (sliderName)
        {
            case "sliMusic":
                Debug.Log("Music Volume: " + (float)Math.Round(value, 1));
                break;
        }
    }
}
```



# 自定义事件监听

监听UI的自定义事件，如拖拽、点击等，需要添加`EventTrigger`

一般手动为UI控件手动添加，代码添加非常麻烦

为UIManager添加静态方法

```C#
public static void AddCustomEventListener(UIBehaviour control, EventTriggerType eventTriggerType, UnityAction<BaseEventData> callBack)
{
    EventTrigger eventTrigger = control.GetComponent<EventTrigger>();
    if (eventTrigger == null)
    {
        eventTrigger = control.gameObject.AddComponent<EventTrigger>();
    }

    EventTrigger.Entry entry = new EventTrigger.Entry();
    entry.eventID = eventTriggerType;
    entry.callback.AddListener(callBack);

    eventTrigger.triggers.Add(entry);
}
```

测试

```C#
public class LoginPanel : BasePanel
{
    protected override void Start()
    {
        base.Start();
        Test();
    }
    private void Test()
    {
        UIManager.AddCustomEventListener(GetControl<Button>("btnLogin"), EventTriggerType.Drag, (eventData) =>
        {
            Debug.Log(eventData);
            PointerEventData pointerEventData = eventData as PointerEventData;
            button.transform.position = pointerEventData.position;
        });
    }
}
```



