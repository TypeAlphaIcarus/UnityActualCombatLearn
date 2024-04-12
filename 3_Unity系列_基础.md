# Unity基础

## 3D数学基础

### 1、Math和Mathf

`Math`为C#中封装好的数学计算的工具 类

`Mathf`为unity中封装好的数学计算工具 结构体

区别：

`Mathf`为unity专门封装，包含`Math`中的方法，还额外提供一些适用于游戏开发的方法

`Mathf`中一般只调用一次的方法

```C#
Mathf.PI;
Mathf.Abs(x);			//绝对值
Mathf.CeilToInt(x);		//向上取整
Mathf.FloorToInt(x);	//向下取整
Mathf.Clamp(val, max, min);	//将value钳制在min~max之间，val>max，则val=max，val<min则val=min，min<val<max，则val不变
Mathf.Max(a,b,c...);	//取最大值
Mathf.Min(a,b,c...);	//取最小值
Mathf.Pow(a,b);			//a的b次幂，a^b
Mathf.RoundToInt(x);	//四舍五入
Mathf.Sqrt(x);			//x平方根
Mathf.IsPowerOfTwo(x);	//x是否为2的n次幂
Mathhf.Sign(x);			//信号函数，判断x正负，正数和0返回1，负数返回-1
```

`Mathf`中通常在`update`多次调用的方法

```C#
result = Mathf.Lerp(start, end, t);	//result = start + (end - start) * t，在start到end的连线中，从start向end方向，根据系数t来截取位置
//插值运用一
//每帧改变start的值，位置无限接近end，速度先快后慢
start = Mathf.Lerp(start, end, Time.deltaTime);

//插值运用二
//每帧改变t的值，位置每帧接近end，t>=1时到end，匀速变化
float time = 0;
time += Time.deltaTime;
result = Mathf.Lerp(start, end, time);	//得到从start到end的匀速变化(时间确定，速度不确定，start到end的路程长度会变化，time从0到1为固定一秒)
```



### 2、线性插值跟随移动

```C#
public class Follw : MonoBehaviour
{
    [SerializeField] private Transform targerObj;
    [SerializeField] private float speed;

    float t = 0;

    private Vector3 targetPos;
    private Vector3 startPos;

    void Update()
    {        
        //MoveConstantTime();
        MoveConstantSpeed();
    }

    //以固定时间匀速移动(速度不定)
    private void MoveConstantTime()
    {
        if (targetPos != targerObj.position)    //目标物体移动
        {
            t = 0;
            targetPos = targerObj.position;
            startPos = transform.position;
        }
        t += Time.deltaTime;
        transform.position = Vector3.Lerp(startPos, targetPos, t);	//始终从startPos开始插值，而非变化后的transform.position
    }

    //以固定速度移动(时间不定)
    private void MoveConstantSpeed()
    {
        transform.position = Vector3.Lerp(
        	transform.position,
        	targerObj.transform.position,
        	(Time.deltaTime) * speed / Vector3.Distance(transform.position, targerObj.position));
        	//x = x + (y - x) * (Time.deltaTime) * speed / (y - x) = x + (Time.deltaTime * speed)
        	//x = x + (Time.deltaTime * speed)，为匀速运动
        	//原本插值为x + (y - x) * (Time.deltaTime) * speed
        	//因为剩余距离(y-x)会越来越小，速度也会越来越小，除以剩余距离(y-x)就可以得到固定比值的速度
    }
}
```



### 3、三角函数

角度和弧度

360° = 2 PI rad

1° = PI / 180 rad

1rad = 180° / PI

快捷转换：

弧度 * 57.3 = 对应角度

角度 * 0.01745 = 对应弧度

```C#
float rad = 1;
float degree = rad * Mathf.Rad2Deg;

float degree = 1;
float rad = degree * Mathf.Deg2Rad;
```



三角函数

```C#
//unity中关于三角函数的运算使用的参数是弧度
float sin30 = Mathf.Sin(Mathf.Deg2Rad * 30f);
float cos30 = Mathf.Cos(Mathf.Deg2Rad * 30f);
```



反三角函数

```C#
//通过正弦余弦值计算弧度和角度
//如得到场景中直角三角形中两条线段的长度，可以计算其夹角弧度和角度
float rad = Mathf.Asin(0.5) ; //arcSin
float deg = rad * Mathf.Rad2Deg;

float rad = Mathf.Acos(0.5);
float deg = rad * Mathf.Rad2Deg;
```



### 4、坐标系

世界坐标

原点为世界的中心点，三个轴向固定

```C#
//常用参数
transform.position;
transform.rotation;
transform.eulerAngles;
transform.lossyScale;
```



物体坐标

原点为物体中心点，中心点和轴向在建模时决定

```C#
//常用参数
transform.localPosition;
transform.localEulerAngles;
transform.localRotation;
transform.localScale;
```



屏幕坐标

原点为屏幕左下角，向右x正方向，向上y正方向

x、y的长宽和分辨率有关

```C#
//常用参数
Input.mousePosition;
Screen.width;
Screen.height;
```



视口坐标系

原点为屏幕左下角，向右x正方向，向上y正方向

左下角坐标固定(0，0)，右上角坐标固定(1，1)，将屏幕坐标归一化

```C#
//常用参数
//摄像机的视口范围
Camera.main.rect = new Rect(0, 0, .5f, .5f);
```



### 5、坐标转换

世界坐标转本地坐标

```C#
transform.InverseTransformDirection();
transform.InverseTransformVector();
transform.InverseTransformPoint();
```



本地坐标转世界坐标

```C#
transform.TransformDirection();
transform.TransformVector();
transform.TransformPoint();
```



世界坐标与屏幕坐标相互转换

```C#
Camera.main.WorldToScreenPoint();
Camera.main.ScreenToWorldPoint();
```



世界坐标与视口坐标转换

```C#
Camera.main.WorldToViewPortPoint;
Camera.main.ViewportToWorldPoint;
```



屏幕坐标与视口坐标转换

```C#
Camera.main.ViewportToScreenPoint;
Camera.main.ScreenToViewportPoint;
```



### 6、向量

向量可以随意移动，有长度和方向即可

`Vector3`可以表示位置，也可以表示向量

```C#
//两点计算向量
Vector3 A, B;
Vector3 AB = B - A;
Vector3 BA = A - B;
```



零向量

唯一一个大小为0的向量



负向量

A 和 -A大小相等，方向相反



向量模长

即向量的长度

```C#
dir.magnitude;	//向量模长为向量的长度
point.magnitude;	//点的模长为原点到该点的长度
Vector3.Distance(A,B);	//AB(BA)向量的长度
```



单位向量

模长为1的向量

在只需要方向，不想让模长影响计算结果时使用

```C#
dir.normalized;	//向量归一化 
dir / dir.magnitude;
```



### 7、向量相关运算

加

位置+位置 没有意义

向量+向量 = 新向量

位置+向量 = 新位置



减

位置 - 位置 =  新向量

向量 - 向量 = 新向量

位置 - 向量 = 加上(-向量)，新位置	

向量 - 位置没有意义



乘除

与标量(常数)相乘除，放大缩小长度，乘以0得0



### 8、向量点乘

A · B = |AB|，相乘后为标量，一个向量在另一个向量的投影长度，可以用于判断敌人大致位置

向量AB夹角θ

```C#
θ = Mathf.Acos(Mathf.Dot(A,B));
```



计算向量尖的夹角

```C#
[SerializeField] private Transform target;
private Vector3 dir;

void Update()
{
    dir = target.position - transform.position;
    float degree = Mathf.Acos(Vector3.Dot(transform.forward, dir.normalized)) * Mathf.Rad2Deg;	//与正前方向量的夹角角度
    
    //unity为此提供了方法
    Vector3.Angle(transform.forward, dir);	//计算向量的夹角
}
```



范围索敌

```C#
[SerializeField] private Transform target;
[SerializeField] private float checkDistance;
[SerializeField] private float checkAngle;
private Vector3 dir;

void Update()
{
    dir = target.position - transform.position;
    if (dir.magnitude < checkDistance)
    {
        float dot = Vector3.Dot(transform.forward, dir.normalized);
        if (Mathf.Acos(dot) * Mathf.Rad2Deg <= checkAngle)
        {
            Debug.Log("发现入侵者");
        }
    }
    //等效于
    if(dir.magnitude < checkDistance && Vector3.Angle(dir) <= checkAngle)
    {
         Debug.Log("发现入侵者");
    }
}
```



### 9、向量叉乘

向量A×B得到垂直于AB向量平面的新向量(法向向量)

A(Xa, Ya, Za)

B(Xb, Yb, Zb)

A×B = (YaZb - ZaYb，ZaXb - XaZb，XaYb - YaXb) = -(B × A)

```
dir = Vector3.Cross(A,B);
```



叉乘的意义

A×B得到新向量C

C.y > 0，B在A的右侧，这里y为垂直于AB平面的向量，根据y的值判断向量方向

C.y < 0，B在A的左侧

A×B != B×A，得到的向量方向相反

```C#
[SerializeField] private Transform targetA;
[SerializeField] private Transform targetB;
private Vector3 dirA;
private Vector3 dirB;
private Vector3 dirC;
void Update()
{
    dirA = targetA.position - transform.position;
    dirB = targetB.position - transform.position;
    dirC = Vector3.Cross(dirA, dirB);
    Debug.DrawLine(transform.position, dirC, Color.red);
    if (dirC.y > 0)
    {
        Debug.Log("B在A右侧");
    }
    else
    {
        Debug.Log("B在A左侧");
    }
}
```



可以与点乘相结合，判断目标的位置

若以`transform.forward`为例，点乘可以判断角度，叉乘可以判断左右

例子：判断左侧和右侧一定角度范围内是否有敌人

```C#
[SerializeField] private Transform target;	//目标位置
[SerializeField] private float checkDistance;	//监测范围
[SerializeField] private float leftDegree;	//左侧角度
[SerializeField] private float rightDegree;	//右侧角度

private Vector3 dir;

private void Update()
{
    if (Vector3.Distance(transform.position, target.position) <= checkDistance)	//距离判断
    {
        dir = target.position - transform.position;	//朝向目标的向量
        
        //判断左侧
        //判断角度范围 && 判断是在Vector3.forward该向量的左侧还是右侧，如果是在左侧，叉乘出的向量在世界空间的y轴分量会小于0
        //因为目标位置可能不确定，叉乘的结果垂直于物体的某个向量和朝向目标向量所在的平面，这个平面可能是任意平面，不一定是平行于世界空间的XOZ的平面
        //但是无论目标如何变化，其叉乘结果在世界空间y轴方向上的分量会以0为分界线，可以以此判断两个向量的位置情况
        if (Vector3.Angle(transform.forward, dir) <= leftDegree && Vector3.Cross(Vector3.forward, dir).y < 0)
        {
            Debug.Log("检测到左侧敌人");
        }

        //判断右侧
        if (Vector3.Angle(transform.forward, dir) <= rightDegree && Vector3.Cross(Vector3.forward, dir).y > 0)
        {
            Debug.Log("检测到右侧敌人");
        }
    }

}
```



### 10、向量插值

1、线性插值，Lerp就是线性插值

`result = start + (end - start) * t，(0 <= t <= 1)`

移动到目标位置

```C#
[SerializeField] private Transform target;
[SerializeField] private float speed;

private Vector3 startPos;
private float t;

void Update()
{
    //先快后慢
    transform.position = Vector3.Lerp(transform.position, target.position, Time.deltaTime * speed);
    
    //匀速运动(固定时间)
    if (targetPos != target.position)
    {
        //目标位置改变，重置时间和位置
        t = 0;
        targetPos = target.position;
        startPos = transform.position;
    }
    t += Time.deltaTime;
    transform.position = Vector3.Lerp(startPos, targetPos, t * speed);
    
    //匀速运动(固定速度)
    transform.position = Vector3.Lerp(
        transform.position,
        targerObj.transform.position,
        (Time.deltaTime) * speed / Vector3.Distance(transform.position, targerObj.position));
        //x = x + (y - x) * (Time.deltaTime) * speed / (y - x) = x + (Time.deltaTime * speed)
        //x = x + (Time.deltaTime * speed)，为匀速运动
        //原本插值为x + (y - x) * (Time.deltaTime) * speed
        //因为剩余距离(y-x)会越来越小，速度也会越来越小，除以剩余距离(y-x)就可以得到固定比值的速度
}
```



2、球形插值

`Vector3.Slerp(start, end, t)`

线性插值为线性距离，球形插值为线性弧度

```C#
[SerializeField] private Transform t_Target;
 private float t;

 void Update()
 {
     t += Time.deltaTime;
     transform.position = Vector3.Slerp(Vector3.right, Vector3.forward, t);
 }
```



### 11、四元数

#### 基本概念

欧拉角用于描述物体的旋转，简单易操作，但会出现

1、旋转的表示不唯一，由一个状态到另一个旋转状态的过程变化方式不唯一

2、万向节死锁问题

万向节死锁：物体的旋转轴向存在父子关系，当物体旋转时，旋转的轴向可能重合，失去一个维度的自由度，unity x轴旋转90°，会产生万向节死锁

解决方法：使用四元数



四元数是实数加上三个虚数组成的复数

四元数包含一个标量，一个3D向量

`[W,V]`，W为标量，V为向量，也可表示为`[W, (x,y,z)]`，任意一个四元数，表示3D空间中的一个旋转量



轴角对：3D空间中，任意旋转可以表示为 绕着某轴旋转某角度，该轴为任意轴，可以不是xyz轴



对于给定的一个旋转，假设绕着轴n，旋转β°，轴n为(x,y,z)，那么可以构成四元数Q

`Q = [ cos(β/2), sin(β/2)n ] = [ cos(β/2), sin(β/2)(x,y,z) ]`

四元数Q表示绕着`n`轴旋转`β`度



#### unity中四元数的初始化

unity中的四元数为结构体`Quaternion`

轴角对公式初始化

`Q = [ cos(β/2), sin(β/2)n ] = [ cos(β/2), sin(β/2)(x,y,z) ]`

```C#
Quaternion q = new Quaternion(sin(β/2)x, sin(β/2)y, sin(β/2)z), cos(β/2));
Quaternion q = new Quaternion(Mathf.Sin(30f * Mathf.Deg2Rad) * 1, 0, 0, Mathf.Cos(30f * Mathf.Deg2Rad));	//轴向(1，0，0)，角度60

//创建一个物体，然后让其旋转
GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Cube);
obj.transform.rotation = q;	//rotation为四元数，对旋转直接赋值
```



轴角对方法初始化

```C#
//								角度		轴
Quaternion q = Quaternion.AngleAxis(60, Vector3.right);
GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Cube);
obj.transform.rotation = q;
```



#### 欧拉角和四元数的转换

```C#
//欧拉角转四元数
Quaternion q = Quaternion.Euler(60, 0, 0);

//四元数转欧拉角
Vector3 euler = q.eulerAngles;
```



四元数相乘

四元数相乘表示旋转四元数，表示旋转一个旋转

```C#
 void Update()
{
    transform.rotation *= Quaternion.AngleAxis(10 * Time.deltaTime, Vector3.right);	//每帧绕Vector3.right旋转
}
```



注意：四元数转欧拉角范围始终在-180~180之间，不存在720之类的角度，而欧拉角60和420旋转的状态相同



### 12、四元数的常用方法

单位四元数

```C#
[1,(0,0,0)] [1,(0,0,0)]//为单位四元数，表示无旋转量
Quaternion.identity;
//使用
Instantiate(objPrefab, Vector3.zero, Quaternion.identity);
```



插值运算

```C#
[SerializeField] private Transform target;
[SerializeField] private float speed;

private Quaternion startRotation;
private Quaternion targetRotation;
private float t;

void Update()
{
    //普通插值，先快后慢
    transform.rotation = Quaternion.Slerp(transform.rotation, targert.rotation, Time.deltaTime);

    //匀速变化(控制时间)
    if (targetRotation != target.rotation)
    {
        t = 0;
        targetRotation = target.rotation;
        startRotation = transform.rotation;
    }
    t += Time.deltaTime;
    transform.rotation = Quaternion.Slerp(startRotation, targetRotation, t);
    
    //匀速变化(控制速度)
    transform.rotation = Quaternion.Slerp(transform.rotation,
                                          target.rotation,
                                          Time.deltaTime * speed / Quaternion.Angle(transform.rotation, target.rotation));
}
```



向量指向转四元数

LookRotation

```C#
//将传入的面朝向，转换为对应的四元数角度信息			返回一个四元数
transform.rotation = Quaternion.LookRotation(target.position - transform.position);	//和LookAt()类似
```





### 13、四元数的计算

四元数相乘

`q3 = q2 * q1`

四元数相乘得到新的四元数，代表两个旋转量的叠加，相当于旋转后再次旋转

```C#
Quaternion q = Quaternion.AngleAxis(60, Vector3.right);
transform.rotation *= q;
```



四元数乘以向量

四元数乘以向量会返回一个新的向量，相当于旋转向量

```C#
Vector3 dir = Vector3.forward;
Quaternion q = Quaternion.AngleAxis(45, Vector3.up);
dir = q * dir;	//四元数在向量左侧
```


