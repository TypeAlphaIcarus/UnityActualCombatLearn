# [Unity系列]()

# 入门

## 1、工作原理

unity如何在Inspector窗口中显示信息？

通过反射，在将脚本添加到场景中对象时，获取到了脚本的类名，通过反射可根据类名获取其中的成员，获取后在Inspector面板显示成员的信息



场景的本质

场景本质是配置文件，包含场景设置信息和场景中所有对象的信息

通过场景配置文件的信息，利用反射，读取信息创建对象，再进行赋值



预制体

预制体本质也是配置文件，和场景类似，预制体放入场景中，利用反射创建对象



## 2、脚本基本规则

类名必须和文件名一致，因为会通过反射，用场景中文件名去寻找对应的类，并实例化对象



### MonoBehavior基类

继承了MonoBehavior才能将脚本挂载到物体上

继承了MonoBehavior只能挂载，不能通过new创建对象，只能通过AddComponent()

继承了MonoBehavior编写构造函数没有意义，因禁止通过new创建继承了MonoBehavior的对象

继承了MonoBehavior可以在一个物体上挂载多个，可以使用`[DisallowMultipleComponent]`禁止挂载多个相同的对象

继承了MonoBehavior的对象可以被继承



## 3、生命周期

Awake() -> OnEnable() -> Start() -> FixedUpdate() -> Update() -> LateUpdate() -> OnDisable() -> OnDestory()

OnEnable()和OnDisable()只有在激活和失活时调用，和循环无关

生命周期函数也是通过反射来调用的，通过方法的名称来获取对象的方法，进行方法的调用



## 4、Inspector显示

```C#
[SerializeField]
[HideInInspector]
[Serializable]

[Header("说明")]	//会直接在Inspector显示一行字
[Tooltip("说明")]	//鼠标悬浮显示说明

[Space()]	//让两个字段间显示间隔，参数为间隔距离
[Range(float a, float b)]	//限制遍历的取值范围，会显示为滑动条

[Multiline()]	//文本区域，参数为行数，默认为三行，可以让字符串进行多行显示
[TextArea(3, 4)]	//滚动条显示字符串，最少显示3行，最多4行，>=4行显示滚动条

[ContextMenuItem("重置金币", "Test")]	//为变量添加快捷方法，参数一：按钮名称，参数二：方法名称
public int money;					  //在变量点击右键，会在Inspector显示出按钮，点击按钮调用对应方法
public void Test()
{
    money = 0;
}

[ContextMenu("Test2")]	//点击挂载的脚本右键时，出现对应名称的选项，点击调用方法
public void Test2()
{
    ...
}
```



## 5、MonoBehavior

重要成员

```C#
GameObject gameObject;
gameObject.name;

Transform transform;
transform.position;

this.enable;
```

获取脚本

```C#
GetComponet("xxx");
GetComponet(typeof(T)) as T; //通过反射获取
GetComponet<T>();	//通过泛型获取

GetComponents<T>();
GetComponentInChildren<T>(); //包括自身也会尝试获取

List<T> t = new List<T>()
GetComponentsInChildren<T>(true, t);

GetComponentInParent<T>();

If(TryGetComponent<T>(out T t))
{...}
```



## 6、常用组件和API

### GameObject

GameObject中的常用成员变量

```C#
gameObject.name;
gameObject.activeSelf;	//是否激活
gameObject.isStatic;	//是否为静态对象
gameObject.tag;
gameObject.layer;
gameObject.transform;
```



静态方法

```C#
GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Cube);	//创建默认几何体

//无法找到失活的对象,只能找到激活的对象
//存在同名对象，会随机返回其中一个
GameObject.Find("name");	
GameObject.FindWithTag("tag");

GameObject[] objectArray = GameObject.FindGameObjectsWithTag("tag"); //寻找多个对象

//通过名称和tag寻找只是遍历对象，该方法还有遍历对象的脚本
T t = GameObject.FinObjectOfType<T>();	//该方法由Object提供，unity中的Object是自定义的类型，不是object，效率较低

//实例化对象
GameObject.Instantiate(prefab);

//删除对象
GameObject.Destory(obj);
GameObject.Destory(obj, 5);	//延迟删除
GameObject.Destory(this);	//会删除挂载的该脚本
obj.Destory();
//Destory()相当于是标记了对象，一般会在下一帧移除对象，并从内存中删除
GameObject.DestoryImmediate(obj);	//立即移除

//切换场景不会被移除
GameObject.DontDestoryOnLoad(obj);
```



成员方法

```C#
//创建空物体
GameObject obj = new GameObject();
GameObject obj = new GameObject("name");
GameObject obj = new GameObject("name", typeof(T1), typeof(T2));	//创建空物体，并添加T脚本，可添加多个

//添加脚本
T t = obj.AddComponent(typeof(T)) as T;
T t = obj.AddComponent<T>();

//获取脚本，与继承Mono的类获取的方法一样
...
    
//标签比较
if(obj.CompareTag("tag"))
{...}
if(obj.tag == "tag")
{...}

//激活失活
obj.SetActive(true);

//以下方法不建议使用，效率较低

//命令自身去执行方法，会在自身挂载的所有脚本中去查找该名称的方法
obj.SendMessage("TestMethod"， 1);	//后续参数为方法的参数

//广播，让自己和自己的子对象执行方法
obj.BroadcastMessage("TestMethod");

//向自己和父对象发送消息执行方法
obj.SendMessageUpwards("TestMethod");
```



### Time

用于计时、暂停、位移等

```C#
//时间缩放
Time.timeScale = 0;

//帧间隔时间
Time.deltaTime;	//受scale影响
Time.unscaledDeltaTime;	//不受scale影响，不受时间暂停影响

Time.time;	//从游戏开始到现在的时间

Time.dixedDeltaTime;	//物理帧间隔时间，写在FixedUpdate()中
Time.dixedUnscaledDeltaTime;

//帧数
Time.frameCount; //总帧数

```



### Transform

位置和位移

`Vecator3`用于表示一个点或向量

```C#
Vector3 pos = new Vector3(x, y, z);
//可以进行+-*/运算

//常用变量
Vector3.zero;
Vector3.left;

Vecator3.Distance(v1, v2);

//位置和位移
transform.position;	//世界空间下的位置，不受父子关系影响
transform.localPosition;	//相对位置

//只改变其中一个值
//直接赋值
transform.position = new Vector3(1, transform.position.y, transform.position.z);
//先取值再赋值
Vector3 pos = transform.position;
pos.x = 1;
transform.position = pos;

//朝向
transform.forward;	//本地朝向

```

位移

```C#
//路程 = 方向 * 速度 * 时间
transform.position = transform.position + transform.forward * speed * Time.deltaTime;

//参数二为相对坐标系，默认为自身坐标
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.World); 	//会朝世界坐标Z轴移动
trasnform.Translate(tramsform.forward * speed * Time.deltaTime, Space.World);	//朝自己面朝向移动

transform.Translate(tramsform.forward * speed * Time.deltaTime, Space.Self);	//如果物体旋转，不会朝自身前方移动
//tramsform.forward获取的是在世界空间下的相对坐标方向，如果从世界坐标转换到本地坐标，需要进行相应的旋转
//本地坐标和世界坐标的方向暂时只考虑旋转，tramsform.forward相对世界坐标的方向是固定的，世界坐标旋转成为了本地坐标，tramsform.forward会经历相同的旋转

transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.Self);	//朝自身坐标下自身面朝向移动，自身坐标的Vector3.forward就是面朝向 
```



角度和旋转

```C#
//相对世界坐标角度
transform.eulerAngels;	//欧拉角
transform.rotation;	//四元数

//相对角度
transform.localEulerAngels;
transform.localRotation;

//旋转
transform.Rotate(Vector.up * speed * Time.deltaTime);				//沿自身y轴旋转
transform.Rotate(Vector.up * speed * Time.deltaTime, Space.World);	//沿世界坐标y轴旋转

//绕着某个点旋转,均为世界坐标
transform.RotateAround(Vector3.right, Vector.up, speed * Time.deltaTime);	
```



缩放和朝向

注意：代码缩放只能一起改，xyz不能单独改

```C#
//相对世界坐标
transform.lossyScale;

//本地坐标
transform.localScale;

//朝向
transform.LookAt(Vector3.zero);	//朝向某点或某对象
```



父子关系

```C#
//设置父对象
transform.parent = obj.transform;
transform.SetParent(obj.transform);
transform.SetParent(obj.transform, true);	//参数二为是否保留世界坐标的位置旋转缩放信息

//移除子对象
transform.DetachChildren();	//移除所有一层子对象，放置到上一层空间中

//获取子对象
transform.Find("name"); //只能找儿子，不能找孙子
transform.childCount;	//包括失活的对象，只儿不孙
transform.GetChild(0);	//根据index获取，超出范围报错

for(int i = 0; i < transform.childCount; i++)
{
    transform.GetChild(i);	
}

//判断父对象
transform.IsChildOf(obj.transform);
transform.GetSiblingIndex();	//获取作为子对象的编号
transform.SetAsFirstSibling();	//设为第一个子物体
transform.SetAsLastSibling();	//设为最后一个子物体
transform.SetSiblingIndex(index);	//设为第index个子物体
```



坐标转换

世界转本地

```C#
//世界坐标点转本地坐标点
Vector3 pos = new Vecrtor3(0,0,1);
transform.InverseTransformPoint(pos);	//转换后的点

//世界坐标方向转本地坐标方向
Vector3 dir = new Vector3(0,0,1);
transform.InverseTransformDirection(dir);	//转换后的向量，不受缩放影响
transform.InverseTransformVector(dir);	//转换后的向量，受到缩放影响，会比较转换前后的transform缩放
```



本地转世界

```C#
//本地坐标点转世界坐标点
transform.TransformPoint(Vector3.forward);

//本地坐标方向转世界坐标方向
transform.TransformDirection(Vector3.forward);	//不受缩放影响
transform.TransformVector(dir);	//受缩放影响
```



### Input

```C#
//鼠标
Input.mousePosition;	//鼠标屏幕位置，以左下角为原点

Input.GetMouseButtonDown(0); //鼠标按键，0左键，1右键，2中建
Input.GetMouseButtonUp(0);
Input.GetMouseButton(0);	//长按，按下、抬起都会响应

Input.mouseScrollDelta(0);	//滚轮，-1向下，0，1向上

//键盘
Input.GetKeyDown(KeyCode.A);
Input.GetKeyUp(KeyCode.A);
Input.GetKey(KeyCode.A);	//长按
Input.GetKeyDown("a");	//字符串只能小写

//输入轴
Input.GetAxis("Vertical");		//返回-1~1
Input.GetAxisRaw("Vertical");	//返回-1或1或0

Input.GetAxis("Mouse X");
Input.GetAxis("Mouse Y");
```



其他

```C#
//任意按键或鼠标长按
Input.anyKey;
Input.anyKeyDown;

//这一帧键盘输入，与anyKeyDown配合可以获取单帧输入
Input.inputString;

//手柄相关
//获取手柄所有按键名称
string[] stringArr = Input.GetJoystickNames();

//某手柄按键按下
Input.GetButtonDown("name");
Input.GetButtonUp("name");
Input.GetButton("name");

//触摸
//单点触摸
if(Input.touchCount > 0)
{
    Touch t1 = Input.touches[0];
    t1.position;
    t1.deltaPosition;	//相对上一次位置变化
}

//是否启用多点触控
Input.multiTouchEnabled = true;

//陀螺仪
Input.gyro.enable = true;
Input.gyro.gravity;	//重力加速度向量
Input.gyro.rotationRate;	//旋转速度
Input.gyro.attitude;	//当前旋转的四元数
```



### Screen

```C#
//静态属性
//屏幕分辨率，Resolution有宽、高、刷新率
Resolution res = Screen.currentResolution;	

//屏幕当前宽高，游戏窗口的宽高
Screen.width;
Screen.height;

//屏幕休眠
Screen.sleepTimeout = SleepTimeout.NeverSleep;	
Screen.sleepTimeout = SleepTimeout.SystemSetting;

//全屏
Screen.fullScreen = true;

//窗口模式
Screen.fullScreenMode = FullScreenMode.ExclusiveFullScreen;	//全屏独占窗口
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;	//全屏窗口
Screen.fullScreenMode = FullScreenMode.MaximizedWindow;		//最大化窗口
Screen.fullScreenMode = FullScreenMode.Windowed;		    //窗口模式

//移动设备屏幕转向
Screen.autorotateToLandScapeLeft;	//Home键在左
Screen.autorotateToLandScapeRight;	//Home键在右
Screen.autorotateToPortrait;		//竖屏
Screen.autorotateToPortraitUpsideDown;//倒着

//指定屏幕方向
Screen.orientation = ScreenOrientation.Landscape;

//静态方法
Screen.SetResolution(width, height, true);	//设置分辨率，参数三为是否全屏
```



### Camera

```C#
Camera.main;	//获取主摄像机
Camera.allCamerasCount;	//所有摄像机数量
Camera[] cameraArray = Gamera.allCameras;	//获取所有相机

Camera.onPreCull += (c) => {...};	//相机剔除前处理委托函数,c为Camera
Camera.onPreRender += (c) => {...}	//渲染前处理委托函数
Camera.onPostRender += (c) => {...} //相机渲染后委托函数

//世界坐标转屏幕坐标
//返回的Z值是距离摄像机Z方向的距离，如0，0，0转换后返回0，0，10，默认相机z坐标为0，0，-10
Camera.main.WorldToScreenPoint(pos); 

//屏幕坐标转世界坐标
//会返回(0,1,-10)，因为鼠标坐标位于屏幕上，没有ｚ坐标，屏幕可以视为视角锥的交汇点，无论如何变化鼠标位置，都是(0,1,-10)，即摄像机位置
//需要添加ｚ轴距离
Camera.main.ScreenToWorldPoint(Input.mousePosition);
```



## 7、核心系统

### 光源

略



### 碰撞

碰撞检测的条件：两个物体都有collider，至少一个物体有RigidBody



刚体 RigidBody

IsKinematic：启用后，不受物理引擎的控制，只能`Transform()`操作

Interpolate：插值运算，Interpolate根据前一帧来平滑变换，Extrapolate根据下一帧来估计平滑变化



CollisionDetection：用于防止快速移动的对象穿过其他对象而不发生碰撞检测

Discrete：离散检测，对场景中其他所有碰撞体进行离散检测

Continuous：连续检测，对动态(有rigidbody)使用离散检测，对没有刚体的使用连续碰撞检测，Continuous Dynamic与该刚体碰撞使用连续碰撞检测，性能消耗高

Continuous Dynamic：连续动态检测，对Continuous和Continuous Dynamic的对象使用连续碰撞检测，对静态碰撞体使用连续碰撞检测，性能消耗高

Continuous Speculative：连续推测检测，对刚体和碰撞体使用推测碰撞检测，比连续碰撞检测消耗低

|                                   | Discrete               | Continuous             | Continuous Dynamic     | Continuous Speculative | 无刚体的碰撞盒         |
| --------------------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- |
| Discrete 离散检测                 | Discrete 离散检测      | Discrete 离散检测      | Discrete 离散检测      | Continuous Speculative | Discrete               |
| Continuous 连续检测               | Discrete 离散检测      | Discrete 离散检测      | Continuous  连续检测   | Continuous Speculative | Continuous             |
| Continuous Dynamic 连续动态检测   | Discrete 离散检测      | Continuous 连续检测    | Continuous  连续检测   | Continuous Speculative | Continuous             |
| Continuous Speculative 续推测检测 | Continuous Speculative | Continuous Speculative | Continuous Speculative | Continuous Speculative | Continuous Speculative |
| 无刚体的碰撞盒                    | Discrete               | Continuous             | Continuous             | Continuous Speculative | 不检测                 |

性能消耗：Continuous Dynamic > Continuous Speculative > Continuous  > Discrete 



#### 碰撞器

注意：父对象添加了刚体之后，所有子物体的碰撞器都会进行碰撞检测，所有子物体的碰撞体将视为一个整体，子物体间不会发生碰撞

OnCollision和OnTrigger在FixedUpdate之后，Update之前执行

```C#
private void OnCollisionEnter(Collision collision)	//collision为另一个物体的碰撞信息
{
    collision.collider;
    collision.gameobject;
    collision.contactCount;	//接触点数量
    ContactPoint[] points = collision.contacts;	//获取所有碰撞点
}

//两个物体相互摩擦时，不停调用，停止摩擦后不会调用
private void OnCollisionStay(Collision collision)
{...}

private void OnCollisionExit(Collision collision)
{...}

//触发器
private void OnTriggerEnter(Collider other) //Collider为另一个物体的碰撞器
{
    
}
private void OnTriggerStay(Collider other)
{...}
private void OnTriggerExit(Collider other)
{...}
```



#### 刚体加力

```C#
RigidBody rb = GetComponent<RigitBody>();
float force = 100;

rb.AddForce(dir * force);	//瞬时加力
rb.AddRelativeForce(dir * force); //根据本地坐标，添加瞬时力
rb.AddForceAtPosition(dir * force, pos);

rb.AddTorque(dir * force);	//添加扭矩，瞬时
rb.AddRelativeTorque(dir * force);	//根据本地坐标，添加瞬扭矩

rb.velocity = dir * 100;	//直接设置速度

//爆炸、范围受力
rb.AddExplosionForce(force, Vector3 pos, float radius);	//在某个位置添加爆炸力，不是全局的
```



#### 力的模式

动量定理

```C#
Ft = mv;	//F：力，t：时间，m：质量，v：速度
v = Ft/m;
```

```C#
//v = Ft/m，若dir * force = (0,0,10)，t=0.02s，Acceleration模式会忽略质量，默认m=1kg
//则v=Ft/m = 10*0.02/1 = 0.2m/s，每个物理帧移动0.004m
rb.AddForce(dir * force, ForceMode.Acceleration);	

//v = Ft/m，若dir * force = (0,0,10)，t=0.02s，Force模式不会忽略质量，若m=2kg
//则v=Ft/m = 10*0.02/2 = 0.1m/s，每个物理帧移动0.002m
rb.AddForce(dir * force, ForceMode.Force);			

//v = Ft/m，若dir * force = (0,0,10)，t=0.02s，Impulse模式会忽略时间，默认为1，若m=2kg
//则v=Ft/m = 10*1/2 = 5m/s，每个物理帧移动0.1m
rb.AddForce(dir * force, ForceMode.Impulse);

//v = Ft/m，若dir * force = (0,0,10)，t=0.02s，VelocityChange模式会忽略时间与质量，t默认为1，m默认为1
//则v=Ft/m = 10*1/1 = 10m/s，每个物理帧移动0.2m
rb.AddForce(dir * force, ForceMode.VelocityChange);
```



#### 持续力

添加Constant Force组件



刚体休眠：方块落到平面之后，旋转平面，刚体不会下落，为节省性能会将刚体休眠

```C#
if(rb.IsSleeping())
{
    rb.WakeUp();
}
```



### 音效

```C#
AudioSource audioSource = GetComponent<AudioSource>();
audioSource.Play();
audioSource.Stop();
audioSource.Pause();
audioSource.UnPause();

//检测音效是否播放完毕
if(audioSource.isPlaying)
{...}

//动态控制音效播放
//1、挂载脚本控制播放
//2、实例化挂载了音源的预制体，用的比较少
//3、用一个AudioSource控制播放不同音效

audioSource.clip = clip;
audioSource.Play();
```



#### 麦克风输入

```C#
//获取麦克风设备信息
string[] s = Mircophone.devices;
AudioClip clip = new AudioClip();

//开始录制
//参数一：设备名称，参数二为超过设定长度后是否从头录制，重写录制会覆盖之前录制的内容
clip = Mircophone.Start(null, false, 10, 44100);	//参数三：10s，参数四：采样率：44100

//结束录制
Mircophone.End(null);	//参数为设备名称，null为默认设备
audioSource.clip = clip;
audioSource.Play();

//获取音频用于存储或传输
float[] f = new float[clip.channels * clip.samples];	//数据长度 = 声道数 * 采样数
clip.GetData(f, 0);	//参数二为偏移，将clip的音频数据存入float数组
```



## 8、项目实践

坦克大战

### 场景切换和游戏退出

```C#
SceneManager.LoadScene(index);
SceneManager.LoadScene("name");

Application.Loadlevel("name");	//弃用

Application.Quit();	//退出游戏
```



### 鼠标隐藏和锁定

```C#
//隐藏鼠标
Cursor.visible = false;	

//锁定鼠标
Cursor.lockState = CursorLockMode.None;	//不锁定
Cursor.lockState = CursorLockMode.Locked;	//锁定到中心
Cursor.lockState = CursorLockMode.Confined;	//锁定到窗口范围内

//设置鼠标图片
Cursor.SetCursor(Texture2D tex, Vector2 offset, CursorMode.Auto);
```



### 随机数

```C#
//Unity中的随机数
UnityEngine.Random random = Random.Range(min, max); //[)

//C#中的随机数
System.Random random = new Random();
random.Next(min, max);
```



### Unity自带委托

```C#
//C#自带委托
Action action = () => {...}
Action action<int,string> = (i,s) => {...}

Func<int> func = () => { return 1; };
Func<int, string> func = (i) => { return "s"; };

//Unity自带委托
UnityAction action = () => {...}
UnityAction<string> action = (s) => {...}
```

