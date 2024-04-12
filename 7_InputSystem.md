# InputSystem

可以不必再对输入进行监听，直接在unity中进行配置即可

需要导入InputSystem

# 输入模式切换

在`File —— Build Setting —— Player Setting —— Other Setting —— Configuration —— Active Input Handling`即可切换



# 代码输入检测

## 键盘输入检测

```C#
private Keyboard keyboard;
private void Start()
{
    //新输入系统，提供了对应的输入设备的类
    keyboard = Keyboard.current;   //获取当前使用的键盘设备
}

void Update()
{
    //检测单个按键，对应的按键就是对应的xxKey
	//按下检测
	if (keyboard.spaceKey.wasPressedThisFrame)
	{
	    Debug.Log("按下space");
	}

	//抬起检测
	if (keyboard.spaceKey.wasReleasedThisFrame)
	{
	    Debug.Log("抬起space");
	}

	//长按检测
	if (keyboard.spaceKey.isPressed)
	{
	    Debug.Log("长按space");
	}
    
    //任意按键检测
    if(keyboard.anyKey.wasPressedThisFrame)
    {...}
    
    //通过事件监听
	keyboard.onTextInput += (key) =>
	{
	    Debug.Log(key);
	};
    
	keyboard.onTextInput += Keyboard_OnTextInput;
}

private void Keyboard_OnTextInput(char key)
{
    Debug.Log(key);
}
```



## 鼠标输入检测

```C#
private Mouse mouse;
void Start()
{
    mouse = Mouse.current;
}
void Update()
{
    //按下 wasPressedThisFrame
	//抬起 wasReleasedThisFrame
	//长按 isPressed
	//左键 leftButton
	//右键 rightButton
	//中键 middleButton
	//前后侧键 forwardButton, backButton
	if (mouse.leftButton.wasPressedThisFrame)
	{
	    Debug.Log("点击");
	}

	//当前鼠标位置
	Vector2 mousePos = mouse.position.ReadValue();

	//鼠标相对上一帧偏移
	//根据xy正负可以判断鼠标移动方向，根据值可以判断鼠标移动速度
	Vector2 mousedDelta = mouse.delta.ReadValue();

	//滚轮方向向量
	mouse.scroll.ReadValue();    
}
```



## 触屏输入检测

```C#
private Touchscreen touchscreen;
void Start()
{
    touchscreen = Touchscreen.current;
}

void Update()
{
    if (touchscreen == null)
        return;

    //获取点击数量
    int num = touchscreen.touches.Count;

    //获取单个手指点击
    if (touchscreen.touches.Count != 0)
    {
        TouchControl touch = touchscreen.touches[0];
    }

    //获取所有手指点击
    if (touchscreen.touches.Count != 0)
    {
        foreach (var touch in touchscreen.touches)
        {
            Vector2Control pos = touch.position;
        }
    }

    TouchControl tc = touchscreen.touches[0];
    //按下识别
    if (tc.press.wasPressedThisFrame)
    { }

    //长按识别
    if (tc.press.isPressed)
    { }

    //抬起识别
    if (tc.press.wasReleasedThisFrame)
    { }

    //点击手势
    if (tc.tap.isPressed)
    { }

    //连续点击次数
    IntegerControl tapNum = tc.tapCount;

    //获取手指实时位置
    Vector2 touchPos = tc.position.ReadValue();

    //第一次按下时的位置
    Vector2 startPos = tc.startPosition.ReadValue();

    //接触区域大小
    Vector2 area = tc.radius.ReadValue();

    //偏移位置
    Vector2 deltaPos = tc.delta.ReadValue();

    //获取当前手指状态
    //None, 无
    //Began,开始点击
    //Moved,移动拖拽
    //Ended,结束
    //Canceled,取消、离开屏幕
    //Stationary,静止/长按
    UnityEngine.InputSystem.TouchPhase touchPhase = tc.phase.ReadValue();
    switch (touchPhase)
    { }
}
```



## 手柄输入检测

```C#
private Gamepad gamepad;
void Start()
{
    gamepad = Gamepad.current;
}
void Update()
{
    if (gamepad == null)
        return;

    //摇杆
    //左摇杆
    Vector2 leftStick = gamepad.leftStick.ReadValue();

    //右摇杆
    Vector2 rightStick = gamepad.rightStick.ReadValue();

    //摇杆按下
    if (gamepad.leftStickButton.wasPressedThisFrame)
    { }
    //摇杆抬起
    if (gamepad.leftStickButton.wasReleasedThisFrame)
    { }
    //摇杆长按
    if (gamepad.leftStickButton.isPressed)
    { }

    //左侧方向键,dpad = direction pad
    //left、right、up、down
    if (gamepad.dpad.left.wasPressedThisFrame)
    { }

    //右侧按键
    //Y，▲：buttonNorth
    //X，■：buttonWest
    //A，×：buttonSouth
    //B，●：buttonEast
    if (gamepad.buttonSouth.wasPressedThisFrame)
    { }

    //中央按键
    if (gamepad.startButton.wasPressedThisFrame)
    { }
    if (gamepad.selectButton.wasPressedThisFrame)
    { }

    //上方肩键
    if (gamepad.leftShoulder.wasPressedThisFrame)
    { }

    //扳机键
    float leftTrigger = gamepad.leftTrigger.ReadValue();
}
```



## 其他输入

传感器、陀螺仪等，参考官方文档

https://docs.unity3d.com/Packages/com.unity.inputsystem@1.8/manual/Sensors.html



# InputAction

声明一个`public InputAction inputAction`，即可在Inspector窗口绑定和编辑输入

```C#
//可以声明不同的对象来绑定不同的动作
public InputAction move;
public InputAction fire;
```

点击设置按钮后出现窗口

```C#
Action Type;	//动作类型
	Value：值类型，用于连续输入，如鼠标移动、手柄摇杆，若有多个设备绑定该action，只会发送一个设备的输入;
	Button：按钮类型，每次按下时触发action;
	Pass Through：直通类型，和Value一样，区别是若有多个设备输入，会发送所有设备的输入;

Control Type;	//各个设备属性的返回值，没有选择的类型将不会在按键设置中出现，相当于在筛选输入设备
	Any：任意值;
	Analog： 模拟值，float;
	Axis：一维浮点数，如手柄扳机;
	Bone：骨骼;
	Digital：数字;
	Double：浮点;
	Dpad：方向按键;
	Eyes：VR相关;
	Integer：整数;
	Pose：;
	Quaternion：四元数;
	Stick：摇杆;
	Touch：触屏;
	Vector2：;
	Vector3：;

Interactions;	//相互作用设置，用于特殊输入，如长按、连点等，可以设置长按时间和连点次数
//以下操作都包含started、performed、canceled事件
	Hold：按下触发started，在松开按钮前，按住时间>=Hold Time，触发performed，否则触发canceled;
	Tap：按下触发started，在MaxTapDuration之内松开，触发performed，否则触发canceled;
	Slow Tap：类似Hold，按住时间>=MaxTapDuration时，不会立即触发performed，松开时触发performed;
	Multi Tap：多次点击，TapCount为点击次数，MaxTapSpacing为连击间隔，MaxTapDuration为每次点击持续时间，当每次点击时间<MaxTapDuration且点击间隔时间<MaxTapSpacing，点击TapCount次，触发performed;
	Press;
		Press Only：按下触发started和performed，不触发canceled;
		Release Only：按下触发started，松开触发performed;
		Press And Release：按下触发started和performed，松开再次触发started和performed，不触发canceled;
		Press Point：每个按钮有对应的浮点值，设置一个阈值，大于该阈值认为按下按钮;
		
Processors;	//值加工处理，对得到的值进行处理
	Clamp：将输入值进行钳制;
	Invert：将值进行反转;
	Invert Vector2：将两个值都反转，invertX为true，反转x轴，invertY为true，反转y轴;
	Invert Vector3：同理;
	Normalize：归一化输入值，若值<0，归一化为[-1,1];
	Normalize Vector2：归一化;
	Normalize Vector3：归一化;
	Scale：将输入值乘以系数;
	Scale Vector2：将输入值乘以系数;
	Scale Vector3：将输入值乘以系数;
	Axis Deadzone：设置死区;
```

添加绑定

```C#
Add Binding;	//添加单个按键输入
Add Positive/Negative Binding;	//添加一维的数轴组合，返回[-1,1]，如绑定← →
Add Up/Down/Left/Right Binding;	//添加二维向量组合，如绑定WASD
Add Up/Down/Left/Right/Forward/Back Binding;	//添加三维向量组合
Add Binding With One Modifier;	//添加一个复合按键，如Ctrl C，Ctrl V
Add Binding With Two Modifier;	//添加两个复合按键，如Ctrl Shift S
```



在InputAction绑定好之后进行输入检测

```C#
//可以声明不同的对象来绑定不同的动作
public InputAction move;
public InputAction fire;

void Start()
{
    //启用InputAction
    move.Enable();

    move.started += Move_started;   //开始操作
    move.performed += Move_performed;   //真正触发
    move.canceled += Move_canceled;   //结束操作
}

private void Move_started(InputAction.CallbackContext context)
{

    Vector2 vector2 = context.ReadValue<Vector2>();
    Debug.Log("开始" + vector2);
}
private void Move_performed(InputAction.CallbackContext context)
{
    //获取当前状态
    //没有启用：Disabled
    //等待：Waiting
    //开始：Started
    //触发：Performed
    //结束：Canceled
    Debug.Log(context.phase);

    //动作行为信息
    Debug.Log(context.action.name);

    //控件(设备)信息
    Debug.Log(context.control.name);

    //获取值
    Vector2 vector2 = context.ReadValue<Vector2>();
    Debug.Log("触发" + vector2);

    //持续时间
    float duration = (float)context.duration;
    Debug.Log(duration);

    //开始时间
    float startTime = (float)context.startTime;
    Debug.Log(startTime);
}
private void Move_canceled(InputAction.CallbackContext context)
{
    Vector2 vector2 = context.ReadValue<Vector2>();
    Debug.Log("结束" + vector2);
}
```



# 输入配置文件

右键创建`InputActions`文件，本质是json文件，双击进行配置

参数

```C#
Action Maps;	//行为映射，表示多套操作规范
Actions;	//InputAction的集合，输入行为窗口
Properties;	//输入属性
```

## 配置文件生成C#代码

点击配置文件的`Generate C# Class`即可设置生文件的名称、位置等参数，点击apply即可生成

直接使用该类即可，这里命名为`PlayerInput`

```C#
public class InputActionAssetTest : MonoBehaviour
{
    private PlayerInput playerInput;
    private void Start()
    {
        //创建生成的代码对象
        playerInput = new PlayerInput();

        //激活
        playerInput.Enable();

        //事件监听
        //这里PlayerAction为ActionMaps中其中一个映射配置的名称，可以直接获取
        //Fire为PlayerAction该ActionMaps中的其中一个InputAction，可以直接获取
        playerInput.PlayerAction.Fire.performed += (context) =>
        {
            Debug.Log("Fire");
        };
        playerInput.PlayerAction.Jump.performed += (context) =>
        {
            Debug.Log("Jump");
        };

        //获取InpuAction的值
        playerInput.PlayerAction.Move.ReadValue<Vector2>();
    }
}
```



## PlayerInput

InputSystem提供用于处理玩家输入的组件

直接关联配置文件即可，甚至不用生成类和new对象

参数

```C#
Actions;	//关联的配置文件，点击Create可以自动创建
Camera;		//关联的相机，分屏时才修改该选项
UI Input Module;	//关联UI的EventSystem，需要更新为InputSystem的新组件
Behavior;	//如何通知游戏对象执行对应逻辑
	Send Messages：将逻辑脚本挂载到Player Input同一对象上，通过SendMessage通知方法运行;
	Boardcast Message：将逻辑脚本挂载到Player Input同一对象或子对象上，通过Boardcast Message通知方法运行;
	Invoke Unity Actions：通过拖拽脚本指定执行方法;
	Invoke C Sharp Events：通过C#事件监听进行处理，需要获取PlayerInput进行监听;
```



## 行为执行模式

### Send Messages

```C#
//触发输入会自动调用对应的方法，On + "行为名称"
private void OnMove(InputValue value)
{
    Debug.Log(value);
    Debug.Log(value.Get<Vector2>());
}

private void OnFire(InputValue value)
{
    if(value.isPressed)
    {
     	Debug.Log("Fire");   
    }    
}

private void OnDeviceLost(PlayerInput playerInput)
{
    Debug.Log("设备丢失");
}

private void OnDeviceRegained(PlayerInput playerInput)
{
    Debug.Log("设备重连");
}

private void OnControlsChanged(PlayerInput playerInput)
{
    Debug.Log("设备切换");
}
```



### Boardcast Message

和Send Messages类似，除了挂载在自身对象上，还可以挂载到子节点上

其余规则相同



### Invoke Unity Events

在Inspector窗口拖拽指定对应的方法，方法参数类型需要修改为`InputAction.CallBackContext`

```C#
public void Move(CallbackContext context)
{
    Debug.Log(context.action.ReadValue<Vector2>());
}

public void Fire(CallbackContext context)
{
    Debug.Log("Fire");
}

public void DeviceLost(CallbackContext context)
{
    Debug.Log("设备丢失");
}

public void DeviceRegained(CallbackContext context)
{
    Debug.Log("设备重连");
}

public void ControlsChanged(CallbackContext context)
{
    Debug.Log("设备切换");
}
```



### Invoke C Sharp Events

```C#
private PlayerInput playerInput;
private void Start()
{
    playerInput = GetComponent<PlayerInput>();
    playerInput.onDeviceLost += OnDeviceLost;
    playerInput.onDeviceRegained += OnDeviceRegained;
    playerInput.onControlsChanged += OnControlsChanged;

    playerInput.onActionTriggered += OnActionTriggered;
}

//所有行为都在这里触发
public void OnActionTriggered(CallbackContext context)
{
    switch (context.action.name)
    {
        case "Move":
            Debug.Log(context.action.ReadValue<Vector2>());
            break;
        case "Fire":
            //触发阶段才执行逻辑
            if (context.phase == InputActionPhase.Performed)
                Debug.Log(context.action.ReadValue<float>());
            break;
    }
}

public void OnDeviceLost(PlayerInput playerInput)
{
    Debug.Log("设备丢失");
}

public void OnDeviceRegained(PlayerInput playerInput)
{
    Debug.Log("设备重连");
}

public void OnControlsChanged(PlayerInput playerInput)
{
    Debug.Log("设备切换");
}
```



## 直接从Update中获取输入

```C#
void Update()
{
    playerInput.currentActionMap["Move"].ReadValue<Vector2>();
}
```



# PlayerInputManager

PlayerInputManager主要管理本地多人输入，管理玩家的离开和加入

新建空物体`PlayerInputManager`添加组件`PlayerInputManager`

参数

```C#
Notification Behavior;	//玩家进入时通知的方式，PlayerInputManager如何通知关联的对象，方式和Player Input中相同
	Send Messages;
	Boardcast Message;
	Invoke Unity Events;
	Invoke C Sharp Events;
Join Behavior;	//玩家加入机制
	When Button Is Pressed：新设备按下任意按键;
	When Join Action Is Triggered：新设备按下指定按键;
	Manually：不会自动加入玩家，需要手动加入玩家;
Player Prefab;	//挂载PlayerInput的对象
Joining Enabled By Default;	//开启后，玩家按照Join Behavior的规则加入
Limit Number Of Players;	//是否开启玩家数量限制
	Max Player Count：允许参加游戏的最大玩家数量;
```



使用

```C#
PlayerInputManager.instance.onPlayerJoined += (playerInput) => { };
PlayerInputManager.instance.onPlayerLeft += (playerInput) => { };
```

 

# UGUI配合使用

新的输入系统不支持GUI，但编辑器不受影响

新输入系统支持UGUI，但需要使用新的输入模块`Input System UI Input Module`

若VR项目需要使用，要在Canvas上添加`Tracked Device Raycaster`组件

多人游戏使用多套UI，需要将`EventSystem`替换为`MultiplayerEventSystem`



## On Screen组件

On Screen组件可以模拟UI和用户操作的交互

On-Screen Button：按钮交互

On-Screen Stick：摇杆交互

为Image添加On-Screen Stick，为Button添加On-Screen Button并设置对应触发输入即可



# Input Debug

Input Debug是输入调试器，通过Window — Analysis — Input Debug打开



# 任意键输入

将InputAction绑定键盘的任意按键

```C#
inputAction.Enable();
inputAction.performed += (context)=> { Debug.Log("按下任意按键"); };	//不能获取具体按下哪个按键
```



键盘按下任意字符

```C#
Keyboard.current.onTextInput += (c) => { Debug.Log(c) };	//可以获取字符，但无法获取空格、回车等按键
```

无法监听手柄、鼠标输入



使用InputSystem中专门处理任意键的方案

```C#
//CallOnce只执行一次，Call每次按下都执行
InputSystem.onAnyButtonPress.CallOnce((inputControl) =>
{
    Debug.Log("按下任意按键" + inputControl.path + inputControl.name);
});
InputSystem.onAnyButtonPress.Call((inputControl) =>
{
    Debug.Log("按下任意按键" + inputControl.path + inputControl.name);
});
```

任意设备按下任意键都能获取到



# 通过Json手动加载输入配置

将自动创建的PlayerInputAsset复制一份，改为json后缀

将`PlayerInput`组件的Actions置空，将通过代码加载

```C#
private PlayerInput playerInput;
private void Awake()
{
    playerInput = GetComponent<PlayerInput>();
}
void Start()
{
    string s = Resources.Load<TextAsset>("Json/InputSystem").text;
    InputActionAsset inputActions = InputActionAsset.FromJson(s);
    playerInput.actions = inputActions;

    playerInput.onActionTriggered += OnActionTriggered;
}

private void OnActionTriggered(CallbackContext context)
{
    switch (context.action.name)
    {
        case "Move":
            Debug.Log(context.action.ReadValue<Vector2>());
            break;
        case "Fire":
            Debug.Log(context.action.ReadValue<float>());
            break;
    }
}
```



# 按键绑定修改

步骤：

使用InputActrionAsset生成按键绑定，复制一份存储为json文件

获取输入配置json信息

修改json数据，修改绑定的按键

生成`InputActionAsset`，赋值给`PlayerInput`

新建类，用于存储绑定按键

```C#
public enum E_ButtonAction
{
    Up,
    Down,
    Left,
    Right,
    Fire,
    Jump
}
public enum E_InputDevice
{
    Keyboard,
    Mouse,
    Gamepad
}

[Serializable]
public class KeyInfo
{
    public E_ButtonAction buttonAction;
    public E_InputDevice inputDevice;
    public string key;
    public KeyInfo() { }
    public KeyInfo(E_ButtonAction buttonAction, string key, E_InputDevice inputDevice)
    {
        this.buttonAction = buttonAction;
        this.key = key;
        this.inputDevice = inputDevice;
    }

    public override string ToString()
    {
        return $"<{inputDevice.ToString()}>/{key}";
    }
}
```

```C#
[Serializable]
public class InputInfo
{
    public List<KeyInfo> keyInfoList = new List<KeyInfo>()
    {
        new KeyInfo(E_ButtonAction.Up, "w", E_InputDevice.Keyboard) ,
        new KeyInfo(E_ButtonAction.Down, "s", E_InputDevice.Keyboard),
        new KeyInfo(E_ButtonAction.Left, "a", E_InputDevice.Keyboard),
        new KeyInfo(E_ButtonAction.Right, "d", E_InputDevice.Keyboard),
        new KeyInfo(E_ButtonAction.Fire, "leftButton", E_InputDevice.Mouse),
        new KeyInfo(E_ButtonAction.Jump, "space", E_InputDevice.Keyboard)
    };
}
```

```C#
[CreateAssetMenu(fileName = "InputData", menuName = "InputData/DataAsset")]
public class InputData : ScriptableObject
{
    public List<InputInfo> inputInfoList = new List<InputInfo>() { new InputInfo() };
}
```

```C#
public class PlayerInputManager
{
    private static PlayerInputManager playerInputManager = new PlayerInputManager();
    public static PlayerInputManager GetInstance() => playerInputManager;

    public InputActionAsset inputActionAsset;
    private InputInfo inputInfo; //用于替换的按键，实际按键
    private InputData inputData;
    public PlayerInputManager()
    {

    }
    public InputInfo GetInputInfo()
    {
        if (inputInfo == null)
        {
            LoadInputInfo();
        }
        return inputInfo;
    }
    private InputInfo LoadInputInfo()
    {
        TextAsset textAsset = Resources.Load<TextAsset>("Json/InputData");
        string s = textAsset.text;
        inputData = ScriptableObject.CreateInstance<InputData>();
        JsonUtility.FromJsonOverwrite(s, inputData);

        inputInfo = inputData.inputInfoList[0];
        return inputInfo;
    }

    public void SaveInputData()
    {
        string path = Application.dataPath + "/Resources/Json/InputData.json";
        string s = JsonUtility.ToJson(inputData);
        File.WriteAllText(path, s);
    }

    public InputActionAsset LoadInputActionAsset()
    {
        string str = Resources.Load<TextAsset>("Json/InputSystem").text;
        if (inputInfo == null)
        {
            LoadInputInfo();
        }

        //替换按键映射
        foreach (KeyInfo keyInfo in inputInfo.keyInfoList)
        {
            switch (keyInfo.buttonAction)
            {
                case E_ButtonAction.Up:
                    str = str.Replace("<up>", keyInfo.ToString());
                    break;
                case E_ButtonAction.Down:
                    str = str.Replace("<down>", keyInfo.ToString());
                    break;
                case E_ButtonAction.Left:
                    str = str.Replace("<left>", keyInfo.ToString());
                    break;
                case E_ButtonAction.Right:
                    str = str.Replace("<right>", keyInfo.ToString());
                    break;
                case E_ButtonAction.Fire:
                    str = str.Replace("<Fire>", keyInfo.ToString());
                    break;
            }
        }
        inputActionAsset = InputActionAsset.FromJson(str);
        return inputActionAsset;
    }

    public InputActionAsset GetInputActionAsset()
    {
        if (inputActionAsset == null)
        {
            LoadInputActionAsset();
        }
        return inputActionAsset;
    }

    public void ChangeInputInfo(KeyInfo keyInfo)
    {
        for (int i = 0; i < inputInfo.keyInfoList.Count; i++)
        {
            if (keyInfo.buttonAction == inputInfo.keyInfoList[i].buttonAction)
            {
                inputInfo.keyInfoList[i] = keyInfo;
            }
        }
        //SaveInputData();

        LoadInputActionAsset();
    }
}
```

编辑json文件中的绑定字符串，用特定字符标记用于替换

如：

```c#
"path": "<Mouse>/leftButton",	//原本文件，替换为
"path": "<Fire>",	//用标记<Fire>该绑定的按键，之后替换为具体按键

"path": "<Keyboard>/w",
"path": "<up>",
```

流程：获取json -> 替换绑定按键 -> 生成 InputActionAsset -> 赋值给PlayerInput

关键：修改每个行为绑定的path(按键)

之后制作修改按键绑定的UI

```C#
public class KeyBindingUI : MonoBehaviour
{
    private InputInfo inputInfo;
    [SerializeField] private List<KeyBindingCell> keyBindingCellList;
    void Start()
    {
        inputInfo = PlayerInputManager.GetInstance().GetInputInfo();

        foreach (KeyBindingCell cell in keyBindingCellList)
        {
            for (int i = 0; i < inputInfo.keyInfoList.Count; i++)
            {
                KeyInfo keyInfo = inputInfo.keyInfoList[i];
                if (cell.buttonAction == keyInfo.buttonAction)
                {
                    cell.SetKeyInfo(keyInfo);
                }
            }
            cell.UpdateText();
        }
    }

    public void SaveInputData()
    {
        PlayerInputManager.GetInstance().SaveInputData();
    }
}
```

```C#
public class KeyBindingCell : MonoBehaviour
{
    public Text text_ActionName;
    public Text text_BindingKey;
    public Button button_ChangeBinding;

    public E_ButtonAction buttonAction;
    private KeyInfo keyInfo;

    private void Start()
    {
        button_ChangeBinding.onClick.AddListener(ChangeBinding);
    }
    private void ChangeBinding()
    {
        InputSystem.onAnyButtonPress.CallOnce(ChangeButtonBinding);
    }
    private void ChangeButtonBinding(InputControl inputControl)
    {
        //获取一次输入，为修改后的按键输入
        string path = inputControl.path;
        string[] s = path.Split('/');
        //path = $"<{s[1]}/{s[2]}>";

        keyInfo.inputDevice = s[1] == "Keyboard" ? E_InputDevice.Keyboard : E_InputDevice.Mouse;
        keyInfo.key = s[2];

        UpdateText();

        PlayerInputManager.GetInstance().ChangeInputInfo(keyInfo);
        LoadInputByJson.instance.UpdatePlayerInput();
    }

    public void SetKeyInfo(KeyInfo keyInfo)
    {
        this.keyInfo = keyInfo;
    }
    public KeyInfo GetKeyInfo() => keyInfo;

    public void UpdateText()
    {
        text_ActionName.text = buttonAction.ToString();
        text_BindingKey.text = keyInfo.key;
    }
}
```

