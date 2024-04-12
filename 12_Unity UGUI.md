# Unity UGUI

# 基础组件

```C#
Canvas;
Canvas Scaler;
Graphic Raycaster;
RectTransfrom;
EventSystem;
Standalone Input Module;
```



## Canvas

Canvas负责渲染所有的UI子对象

```C#
Render Mode;	//渲染方式
	Screen Space - Overlay：屏幕空间，覆盖模式，UI始终在最前面;
	Screen Space - Camera：屏幕空间，摄像机模式，根据canvas的距离，物体可以在UI前面;	//建议新建UI Camera，设置为Overly，添加到主相机的Stack中
	World Space：世界空间，3D模式，和物体一样;
Pixel Perfect;	//是否开启无锯齿精确渲染，需要消耗性能
Additional Shader Channel;	//决定Shader可以读取哪些数据
```



## Canvas Scaler

Canvas Scaler用于分辨率自适应



### Constant Pixel Size

无法自适应分辨率

```C#
//无论屏幕大小分辨率如何，UI始终保持相同的像素大小
Scale Factor;	//缩放系数，会缩放Canvas中所有的UI
Reference Pixel Per Unit;	//多少像素对应Unity的一个单位，默认为100，会和图片的Pixel Per Unit一起计算
```

在设置`Scale Factor`后，Canvas的宽高将自动计算，`width/height * Scale Factor = 屏幕分辨率`

UI原始尺寸(`Set Native Size`) = 图片大小(像素) / (Pixel Per Unit / Reference Pixel Per Unit) = 图片大小(像素) * Reference Pixel Per Unit / Pixel Per Unit



### Scale With Screen Size

```C#
Reference Resolution;	//参考分辨率，一般为美术提供UI的标准分辨率
Screen Math Mode;	//当前屏幕分辨率宽高比不适应参考分辨率时，用于分辨率大小自适应的模式
	Match Width Or Height：以宽或高或二者比值来缩放画布;
	Expand：水平或垂直拓展画布，根据宽高比放大缩小画布，可能有黑边;
	Shrink：水平或垂直裁剪画布，根据宽高比放大缩小画布，可能会裁剪;
```



### Constant Physical Size

无论屏幕大小和分辨率如何，UI始终保持相同的物理大小



### 3D

 Canvas 为World Space自动切换

```C#
Dynamic Pixel Per Unit;	//越大UI越清晰，包括Text字体等
```



## Graphic Raycaster

用于检测UI输入事件的射线发射器，主要用于判断玩家是否点击到了UI，基于图形进行检测，而非碰撞器

```C#
Ignore Reversed  Graphics;	//是否忽略反转图形，x轴或y轴翻转，旋转值180
Blocking Objects;	//射线被哪些类型的碰撞器阻挡，Overlay模式无效
	None;
	Two D;
	Three D;
	All;
Blocking Mask;	//射线被哪些层级的碰撞器阻挡，Overlay模式无效
```



## Event System

管理玩家输入的事件，并分发给各个类型的UI控件，如点击、拖拽等

```C#
First Selected;	//首选对象，默认选择的对象，一般用于手柄操作
Send Navigation Events;	//是否允许导航事件，移动、按下、取消
Drag Threshold;	//拖拽操作的阈值，超过多少像素算拖拽
```



## Standalone Input Module

独立输入模块，主要监听鼠标、键盘、控制器、触屏的输入，依赖Event System，输入的事件通过Event System分发

参数对应InputManger中的参数



## Rect Transform

继承了Transform



# 控件

## Image

```C#
Source Image;	//图片来源，必须为Sprite
Use Sprite Mehs;	//使用Sprite网格
Image Type;	
	Simple：缩放整个图片;
	Sliced：九宫格拉伸，只拉伸中间区域;
	Tiled：平铺，重复中间部分;
	Filled：填充;
Preserve Aspect;	//保持图像现有尺寸
```



## Text

```C#
Rich Text;	//是否开启富文本，如<color = #ffffff>xxx</color>
Best Fit;	//忽略字体大小，始终把文本内容显示在矩形中，会自动调整大小
```

联动组件：OutLine，会为字体添加边缘线



## RawImage

原始图片，Source为Texture，一般用于显示大图



## Button

```C#
Interactable;	//是否能够点击
```

事件监听

```C#
button.OnClick.AddListener( () => {...} );
button.OnClick.RemoveListener(...);
button.OnClick.RemoveAllListeners(...);
```



## Toggle

用于制作开关，配合Toggl Group可以制作单选框

```C#
Toggle tog;
tog.isOn = true;

//获取当前激活的toggle
foreach(Toggle tog in togGroup.ActiveToggles())
{...}

//事件监听，参数为开关改变后的状态
tog.onValueChanged.Addlistener((isOn) => 
{
    ...
});
```



## Input Field

```C#
Character Limit;	//最多输入多少字符
Content Type;	//限制输入字符的类型
Caret Blink Rate; //光标闪烁速率
Caret Width;	//光标宽度
```

事件监听

```C#
inputField.onValueChanged.AddListener((str) => {...});	//字符串改变时触发
inputField.onSubmit.AddListener((str) => {...});		//回车确认时触发，鼠标点击外部不会触发
inputField.onEndEdit.AddListener((str) => {...});		//编辑结束时触发，回车、鼠标点击外部都会触发
```



## Slider

```C
Whole Numbers;	//限制为整数
```

事件监听

```C#
slider.onValueChanged.AddListener((val) => {...});
```



## ScrollBar

一般不单独使用，配Scroll View使用

```C#
Value;	//当前进度，占总的百分比，可设置初始位置
Size;	//滚动条大小，页面占总的百分比
Number Of Steps;	//允许滚动多少次，滚动几次进度从0到1
```

事件监听

```C#
scrollBar.onValueChanged.AddListener((val) => {...});
```



## ScrollView

由Scroll Rect 和 ScrollBar 组成

```C#
//Scroll Rect参数
Content;	//所有子对象都将作为显示内容
Movement Type;	//移动类型，控制拖动时的反馈
	Unrealisticd：不受限制随便移动，一般不使用;
	Elastic：回弹，滚动出边缘后会回弹;
	Clamp：钳制效果，始终限制在显示范围内，没有回弹效果;
Inertia;	//移动惯性，拖动松开后的惯性
Deceleration Rate;	//减速率，0~1，0没有惯性，1不会停止
Scroll Sensitive;	//滚轮和触摸板的敏感度
Viewport;	//关联的内容窗口对象，一般是mask
H/V Scroll Bar;
	Visibility;	//滚动条自动隐藏模式
		Permanent：一直显示，不隐藏;
		Auto Hide：自动隐藏滚动条;
		Auto Hide And Expand Viewport：自动隐藏并拓展Viewport;
Spacing;	//滚动条和Viewport之间的间隔
```

常用API

```C#
//修改content大小
scrollRect.content.sizeDelta = new Vector2(100, 100);

//设置content位置
scrollRect.normalizedPosition = new Vector2(0.5f, 0.5f);

//事件监听
scrollRect.onValueChanged.AddListener((vector2) => { });
```



## DropDown

子对象:

Label：当前DropDonw的描述

Arrow：右侧小箭头

Template：下拉选项列表

```C#
Template;	//关联的下拉列表对象
Caption Text;	//关联当前显示的文本组件
Caption Image;	//关联的显示当前内容的图片
Item Text;	//关联的下拉列表选项用的文本控件
Item Image;	//关联的下拉列表选项用的图片
Value;	//当前选项的index，和数组一样，从0开始
Alpha Fade Speed;	//下拉选项淡入淡出的速度
Option;	//选项
```

常用API

```C#
//获取选项，options为List类型
dropdown.options[index];
dropdown.options[index].txt;
dropdown.options[index].image;

//添加选项
dropdown.options.Add(new Dropdown.OptionData("name", image));

//事件监听
dropdown.onValueChanged.AddListener((index) => { });
```



# 图集

打图集可以减小DrawCall

需要在Project Setting — Editor — Sprite Packer中开启，然后创建Sprite Atlas

```C#
SpriteAtlas sa = Resources.Load("path");
sa.GetSprite("name");
```



# 事件监听接口

常用事件接口

```C#
IPointerEnterHandler;
IPointerExitHandler;
IPointerDownHandler;
IPointerUpHandler;
IPointerCheckHandler;
IBeginDragHandler;
IDragHandler;
IEndDragHandler;
```

不常用

```C#
IInitializePotentialDragHandler;	//找到拖动目标时使用，可用于初始化值
IDropHandler;	//在拖动目标对象上调用
IScrollHandler;	//滚动鼠标时调用
IUpdateSelectedHandler;	//每次勾选时，在选定对象上调用
ISelectedHandler;	//当对象称为选定对象时调用
IDeselectedHandler;	//取消选定对象时调用
IMoveHandler;	//移动事件时调用，上下左右
ISubmitHandler;	//按下Submit时调用，如回车
ICancelHander;	//按下Cancel调用，如Esc
```

PointerEventData参数

```C#
//父类为BaseEventData

PointerEventData data;
//参数
data.pointerId;	//点击时的鼠标id，判断是左键还是中建、右键
data.Position;	//当前指针位置
data.PressPosition;	//按下时指针位置
data.delta;	//指针移动增量
data.ClickCount;	//连击次数
data.ClickTime;		//点击时间
data.pressEventCamera;	//最后一个OnPointerPress事件关联的摄像机
data.enterEventCamera;	//最后一个OnPointerEnter事件关联的摄像机
```



# EventTrigger

EventTrigger集成了所有的事件接口，添加Event Trigger组件即可

EventTrigger触发的事件，将由EventSystem进行分发

手动添加

添加对应的事件，编写对应的方法即可

```C#
//将该方法的对象拖入EventTrigger组件添加的接口即可
//参数为BaseEventData
public void Method(BaseEventData data)
{
    //需要转换到所需的类型
    PointerEventData data = data as PointerEventData;
    ...
}
```



代码添加

```C#
public EventTrigger et;

EventTrigger.Entry entry = new Entry();	//Entry为EventTrigger的内部类，包含事件类型(枚举)、触发事件的回调方法
entry.eventID = EventTriggerType.PointerDown;
entry.callBack.AddListener(() => {...});

//建议先遍历是否有相同类型的Entry，直接获取给Entry添加callBack即可，不必再新建
et.triggers.Add(entry);		//triggers为List<Entry>
```

## 案例：制作虚拟摇杆

```C#
public class JoyStick : MonoBehaviour
{
    [SerializeField] private RectTransform handle;	
    [SerializeField] private float handleMoveRadius = 100;	//摇杆移动范围
    
    private EventTrigger eventTrigger;
    
    private Vector3 posDelta;	//拖动时的位置增量
    
    private Vector2 handleDir;
    public Vector2 HandleDir => handleDir;
    
    private void Awake()
    {
        eventTrigger = gameObject.AddComponent<EventTrigger>();

        //添加事件监听
        EventTrigger.Entry entry = new EventTrigger.Entry();
        entry.eventID = EventTriggerType.Drag;
        entry.callback.AddListener(OnDrag);
        eventTrigger.triggers.Add(entry);

        entry = new EventTrigger.Entry();
        entry.eventID = EventTriggerType.EndDrag;
        entry.callback.AddListener(OnEndDrag);
        eventTrigger.triggers.Add(entry);
    }
    
    private void OnDrag(BaseEventData eventData)
    {
        PointerEventData data = eventData as PointerEventData;
        posDelta.x = data.delta.x;
        posDelta.y = data.delta.y;
        //handle.localPosition += new Vector3(data.delta.x, data.delta.y, 0);，节省性能，不要每次都new
        
        //位置移动
        handle.localPosition += posDelta;

        //范围钳制
        if (handle.anchoredPosition.magnitude >= handleMoveRadius)
        {
            handle.anchoredPosition = handle.anchoredPosition.normalized * handleMoveRadius;
        }

        //anchoredPosition是相对于锚点(中心点)的位置，该值=向量，因为已经钳制，直接/(handleMoveRadius就能得到摇杆的向量(方向)和大小(0~1)
        handleDir = handle.anchoredPosition / handleMoveRadius;
    }

    private void OnEndDrag(BaseEventData eventData)
    {
        handle.anchoredPosition = Vector2.zero;
        handleDir = Vector2.zero;
    }
}

```



# 屏幕坐标转UI坐标

RectTransformUtility为RectTransform的辅助类，主要用于进行坐标转换等操作

```C#
//屏幕坐标转UI本地坐标
//一般配合拖拽事件使用，例如将UI图片拖入某个UI容器，摇杆拖拽等
public class Test : MonoBehaviour, IDragHandler
{
    public void OnDrag(PointerEventData eventData)
    {
        Vector2 curentPos;
        RectTransformUtility.ScreenPointToLocalPointInRectangle(
            transform.parent as RectTransform, //返回值curentPos相对位置的父对象
            eventData.position, //拖动时鼠标的位置
            eventData.enterEventCamera, //触发事件的摄像机，有UI摄像机就是UI摄像机
            out curentPos);

        transform.localPosition = curentPos;
    }
}
```

这样更准确

## 优化后的摇杆

```C#
public class JoyStick : MonoBehaviour
{
    [SerializeField] private RectTransform handle;
    [SerializeField] private float handleMoveRadius = 100;
    
    private EventTrigger eventTrigger;
    
    private Vector2 newPos;
    
    private Vector2 handleDir;
    public Vector2 HandleDir => handleDir;
    
    private void Awake()
    {
        ...
    }

    private void OnDrag(BaseEventData eventData)
    {
        PointerEventData data = eventData as PointerEventData;

        RectTransformUtility.ScreenPointToLocalPointInRectangle(
            handle.transform.parent as RectTransform,	//返回值newPos相对位置的父对象
            data.position,	//拖拽时鼠标的位置
            data.enterEventCamera,	//拖拽时触发事件的摄像机，有专门的UI摄像机，就用UI摄像机
            out newPos
        );

        handle.anchoredPosition = newPos;

        //范围钳制
        if (handle.anchoredPosition.magnitude >= handleMoveRadius)
        {
            handle.anchoredPosition = handle.anchoredPosition.normalized * handleMoveRadius;
        }
        
        handleDir = handle.anchoredPosition / handleMoveRadius;
    }
    ...
}
```



# Mask

只要父物体添加了Mask，所有子物体都会被遮罩

遮罩的图片，透明的地方会被遮罩，不透明的地方会显示子物体



# 模型和粒子在UI前显示

注意：canvas不能为Overlay模式

1、设置新的摄像机和层级

添加摄像机，并设置为Overlay模式，将新添加的摄像机添加到Main Camera的Stack中



2、将3D物体渲染到图片上

将渲染内容输入到RenderTexture上，将相机背景设置为黑色，黑色部分在将RenderTexture添加到RawImage后为透明



粒子甚至可以修改的Sorting Layer和Order In Layer让其始终显示在最前面



# 异形按钮

1、添加子对象

子对象图片的范围也会作为点击范围，可以通过拼凑子对象设置检测范围



2、改变图片透明度的相应阈值

开启图片Read/Write

```C#
//修改透明度相应阈值
image.alphaHitTestMinimumThreshold = 0.1f;
```

缺点：会增加内存消耗



# 自动布局组件

布局元素的布局属性，能自动布局的UI必须有以下属性，会根据这些属性计算位置

```C#
Minimum width;	//元素最小宽度
Minimum height;	//元素最小高度
Preferred width;	//分配额外可用宽度之前，此元素应具有的宽度
Preferred height;	//分配额外可用高度之前，此元素应具有的高度
Flexible width;		//此元素相对于同级而填充的额外可用宽度相对量
Flexible height;	//此元素相对于同级而填充的额外可用高度相对量
```

规则如下：

1、首先分配最小宽高

2、父类容器有足够空间，分配Preferred width / height

3、上两步分配完成还有额外空间，分配Flexible width / height

一般情况下这些都是0，image和Text会自动调整

一般情况不会手动去修改，有需求可以添加`LayoutElement`组件



## 水平/垂直布局组件

Horizontal / Vertical Layout Group

```C#
Reverse Arrangement;	//是否反向排列
Control Child Size;	//控制子对象的宽高，开启Child Force Expand，会根据父对象大小平均分配
Use Child Scale;	//设置子对象大小和布局时，是否考虑子对象缩放
Child Force Expand;	//是否强制子对象拓展以补充额外空间，平均排列
```



## 网格布局组件

Grid Layout Group



## 内容大小适配器

Content Size Filter

一般用于Text或配合其他组件使用，如Grid Layout Group

```C#
Unconstrained;	//不根据布局元素扩展
Min Size;	//根据元素最小宽高度扩展
Preferred Size;	//根据布局元素偏好宽高度扩展
```



## 宽高比适配器

Aspect Ratio Fitter

让元素按照一定比例自动调整大小，让布局元素根据父对象大小自动调整

```C#
None;	
Width Contorl Height;	//根据宽度，自动调整高度
Height Control Width;	//根据高度，自动调整宽度
Fit In Parent;	//自动调整宽高、位置、锚点，让矩形适应父对象矩形，同时保持宽高比，可能会出现黑边
Envelope Parent;	//自动调整宽高、位置、锚点，让矩形覆盖父对象矩形，同时保持宽高比，可能会出现裁剪
```

Aspect Ratio：宽 除以 高



# Canvas Group

控制所有子对象的Alpha

```C#
Interactable;	//启用禁用所有子对象
Blocks Raycasts;	//开关所有子对象射线检测
Ignore Parent Group;	//是否忽略父级对象的Canvas Group
```

