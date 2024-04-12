# UnityCore

#  1、图片导入



## 图片格式

```C#
//BMP：windows标准图像格式，无压缩，占用空间大
//TIF：无损图片格式，体积大

//JPG：JPEG，有损压缩，会丢失部分图片数据，无透明(Alpha)通道
//PNG：无损压缩算法的位图格式，压缩比高，生成文件小，有透明(Alpha)通道

//TGA：支持压缩，使用不失真的压缩算法，支持编码压缩，体积小，效果清晰，兼备BMP的图像质量和JPG的体积优势，有透明通道
//PSD：PhotoShop专用格式，需要使用插件将PSD转为UI
//其它格式：EXR、GIF、IFF、PICT等

//最常用的为JPG，PNG，TGA
```



## 纹理类型

设置纹理的类型是为了让纹理图片有不同的用途，指明其用途

### Default

默认纹理，大部分模型贴图都是默认纹理

```C#
//sRGB(Color Texture)：开启后可以将纹理存储在伽马空间中(对每一个像素做一次幂函数运算)
因为人眼对光线反应不呈线性，从黑到白的线性渐变在人眼中不呈现线性，需要进行伽马矫正，让显示器发出看起来自然的图像
光照的计算在线性空间中进行以确保数学上的准确，显示结果在伽马空间中会让人眼觉得更正确
伽马空间公认标准为sRGB，定义了其与线性空间的映射，让人眼能充分利用8位通道的精度
    
//Alpha Source：如何根据图片数据生成Alpha通道
None:不生成Alpha通道;
Input Texture Alpha: 输入纹理的Alpha通道;
From Gray Scale: 从输入纹理的RGB通道取平均值生成Alpha;

//Alpha Is Transparency
Alpha是透明的，启用可以避免边缘过滤的瑕疵;

```



### Normal map

法线贴图是在原物体凹凸表面的每个点上均匀作法线，法线为垂直于某点的切线方向向量

```C#
//CreateFromGrayScale：从灰度高度图创建法线贴图
//Bumpiness：控制凹凸程度，值越大，凹凸感越强
//Filtering：如何计算Bumpiness；Smooth：标准算法生成法线贴图；Sharp：比标准模式更锐利的贴图

```



### Editor GUI &  Legacy GUI

用于GUI，编辑器开发



### Sprite(2D and UI)

2D游戏或UGUI使用的格式

```C#
//Sprite Mode：图像模式
	//Single：按原样单个使用
	//Multiple：瓦片模式，图集也使用该模式，会将一张图片分为多个小型图片
	//Polygon：多边形模式，网格模式

//Pixel Per Unit：世界空间中一个单位(1m)对应多少像素

//MeshType：网格类型，Single和Multiple模式才支持
	//Full Rect：创建四边形，将图像显示在四边形上
	//Tight：基于像素Alpha值来生成网格，更加贴合图片形状
	任何小于32*32的Sprite都将使用Full Rect模式，即使手动设置Tight也是;

//Extrude Edges：网格中图像周围流出的区域大小

//Pivot：Single模式才有

//Generate Physics Shape：根据图像轮廓，生成默认物理物理形状，仅Single和Multiple可用

//Sprite Editor：Sprite图像编辑器
```



### Cursor

用于鼠标光标



### Cookie

光源剪影

类似光源透过一张图片后的光照效果

```C#
//Light Type：光源类型，一般点光源需要立方体纹理，平行光和聚光灯需要2D纹理
```



### Lightmap

光照贴图，不会考虑法向贴图



### Directional Lightmap

定向光照贴图，会考虑法向贴图以及更多信息



### Shadowmask

阴影遮罩，用于烘焙阴影



### Single Channel

纹理只需要单通道的格式

```C#
//Channel：将纹理处理为Alpha通道还是Red通道
	//Alpha：使用Alpha通道，不允许压缩
	//Red：使用R通道
```



## 纹理形状

纹理除了可以制作贴图，还能制作天空盒和反射探针

```C#
//Texture Shape
	//2D：常用的2D纹理，通常用于模型和UI
		//Mapping：如何将纹理投影到物体上	
		Auto：根据纹理信息创建布局;
		6 Frames Layout：标准立方体贴图排列，六个图像;
		Latitude-Longitude Layout：将纹理映射到2D维度;
		Mirrored Ball：纹理映射到球体贴图上，类似立方体贴图;
            
	//Cube：立方体贴图，用于天空盒和反射探针
		//Convolution Type：纹理的过滤类型
		None：无过滤;
		Specular：将立方体贴图作为反射探针;
		Diffuse：将纹理进行过滤表示辐射照度，可作为光照探针;
		
		//Fixup Edge Seams：None和Diffuse才有，解决低端设备立方体贴图过滤错误
		
```



## 高级设置

```C#
//Non-Power of 2：纹理尺寸非2的幂如何处理
None：纹理尺寸不变;
To nearest：将纹理缩放到最接近2的幂的大小;(PVRTC要求纹理格式为正方形)
To larger：将纹理缩放到最大尺寸大小值的2的幂的大小;
To smaller：将纹理缩放到最小尺寸大小值的2的幂的大小;

//Read/Write Enable：启用可以使用Unity提供的方法从纹理中获取数据

//Streaming Mipmaps：启用可以使用纹理串流，主要用于在控制加载内存中的Mipmap级别，用于减少Unity对于纹理所需的内存，用性能换内存
Mip Map Priority：Mipmap优先分配那些层级;

//Generate Mip Maps：允许生成MipMap
```



Mipmaping：贴图渲染的常用技术，可以加快渲染速度，减少图像锯齿，需要将贴图提前进行处理和优化成一系列特定文件，称为Mipmap，会占用一定内存

Mipmap中每个图像是原始图像的一个特定缩放层级的一个复制品，Mipmap会根据渲染需求使用不同缩放等级的贴图

Mipmap是提前进行抗锯齿处理的

如贴图尺寸为256 x 256，Mipmap为128 x 128，64 x 64, 32 x 32, 16 x 16, 8 x 8, 4 x 4, 2 x 2, 1 x 1



## 平铺拉伸

设置纹理的平铺和拉伸规则

```C#
//Wrap Mode：平铺纹理的方式
Repeat：重复纹理;
Clamp：拉伸纹理边缘;
Mirror：整数边界上镜像创建纹理，重复图案;
Mirror Once：镜像创建一次，然后拉伸边缘纹理;
Per-axis：单独控制u-v上包裹的纹理的方式;

//Filter Mode：纹理在通过3D变化拉伸时如何进行过度
Point：纹理靠近时变为块状;
Bilinear：纹理靠近时变得模糊;
Trillinear：与Bilinear类似，也在不同Mipmap级别间模糊;

//Aniso Level：大角度查看纹理时提高纹理质量，会增加性能消耗
```





## 平台设置

平台设置主要设置最终打包时在不同平台的尺寸、格式、压缩方式

```C#
//Max Size：纹理最大尺寸，即使是高分辨率贴图也能进行限制，常用2048

//Resize Algorithm：纹理尺寸大于指定的Max Size时，使用的缩小算法
Mitchell：米切尔算法，常用算法;
Bilinear：双线性插值，细节重要的图片使用，比Mitchell保留更多细节;

//Format：纹理格式
Automatic：根据平台使用默认设置，各个平台支持格式不同;
Ios推荐格式：默认(PVRTC)兼容性强，若无OpenGL ES 2支持，使用一种ASTC格式，有更好的质量和灵活性，压缩速度更快;
Android：因为安卓设备众多，会根据不同设备制作多个安装包
    目标为Open GL ES 3：在PlayerSetting中去除OpenGL ES 2;
	目标为Open GL ES 2：在PlayerSetting中去除OpenGL ES 3和Vulkan，添加OpenGL ES 2;
	ETC格式在所有安卓设备都支持，ETC2在Open GL ES 3支持，Open GL ES 2不支持;

//Compression：压缩类型
None：不压缩;
Low Quality：低质量压缩纹理;
Normal Quality：标准质量压缩纹理;
High Quality：高质量压缩纹理;

//Use Crunch Compression：启用后，使用Crunch压缩，基于DXT和ETC的有损压缩格式，压缩时间长，解压速度快

//Split Alpha Channel：分离Alpha通道，节约内存，会将一张纹理分成两张纹理，一张RGB，一张Alpha，在渲染时再合并

//Override ETC2 fallback：不支持ETC2的设备上使用的格式
```



# 2、Sprite

## Sprite Editor

需要2D Sprite才能使用

图片为Single类型：

```C#
//Sprite Editor：设置图片的属性

//Custom Outline：自定义边缘线位置，自定义Sprite轮廓形状，Sprite的透明区域会被渲染浪费性能，可以调小透明区域以提高性能
Snap：将控制点贴近最近的像素;
可以手动添加删除点;

//Custom Physics Shape：自定义物理形状，和Custom Outline类似

//Skinning Editor：2D骨骼编辑

//Secondary Texture：为图片添加特殊效果，可以添加次要纹理，通过着色器可以获取这些图片
```



图片为Multiple类型：

```C#
//Multiple图集元素分割
Automatic：自动分割;

Grid By Cell Size：按单元格大小分割;
	Pixel Size：单元格大小;
	Keep Empty Rects：是否保留空矩形;

Grid By Cell Count：按行列数分割;

Method：如何处理已经分割的矩形;
	Delete Existing：全部删除替换;
	Smart：尝试创建新的矩形同时保留或调整现有矩形;
	Safe：添加新矩形，不改变现有矩形;

Trim：修剪，减少透明区域;
```



图片为Polygon：

Change Shape：改变多边形形状

注意：修改图片类型后，Sprite的分割不会改变



## Sprite Renderer

2D游戏资源除了UI都是通过Sprite Renderer渲染



### 创建2D对象

直接拖入Sprite图片；右键创建；空物体添加脚本

```C#
//Sprite Renderer参数
//Draw Mode：绘制方式
Simple：缩放时，图片跟随缩放;
Sliced：切片九宫格模式，十字区域缩放，四个角不变，一般用于变化不大的纯色图，Sprite的网格类型(Mesh Type)需要设置为Full Rect;	
Tiled：平铺模式，将中建部分平铺而非缩放，与Sliced类似，Sprite的网格类型(Mesh Type)需要设置为Full Rect;
	例如2D地图，瓦片和有边框的UI面板，中间有大量的重复部分或纯色部分，这样就能用小图设置大的对象，可以节约大量资源，需要手动设置边缘线;
//Tile Mode：
	continuous：尺寸变化时，中间部分均匀平铺;
	Adaptive：尺寸变化时，当更改尺寸到达Stretch Value，中间才开始平铺，没有达到Stretch Value，会进行拉伸;

//Mask Interaction：
None：不和任何Sprite Mask (精灵遮罩)交互;
Visible inside Mask：遮罩覆盖的地方可见，遮罩外不可见;
Visible outside Mask：遮罩覆盖不可见，遮罩外可见;

//Sprite Sort Point：计算Sprite和摄像机间距离时，使用中心点Center还是轴心点Pivot
```



常用API

```C#
Sprite sprite = Resources.Load<Sprite>("name");	//加载单张图片
Sprite[] spriteArray = Resources.LoadAll<Sprite>("name");	//加载切分好的图集
```



### 案例

```C#
//创建一个工具类，可以更方便加载Multiple类型的图集资源
public class MultiSpriteManager
{
    private static MultiSpriteManager instance = new MultiSpriteManager();
    public static MultiSpriteManager Instance => instance;

    //存储加载过的MultipleSprite，外层字典存储图集的名称和图集，内层字典存储MultipleSprite中分割的单个Sprite的名称和Sprite对象
    private Dictionary<string, Dictionary<string, Sprite>> dic = new Dictionary<string, Dictionary<string, Sprite>>();

    private MultiSpriteManager() { }

    /// <summary>
    /// 获取Multiple图集中某张图片
    /// </summary>
    /// <param name="multiName">图集名称</param>
    /// <param name="spriteName">图集中单个图片名称</param>
    /// <returns></returns>
    public Sprite GetSprite(string multiName, string spriteName)
    {
        //判断是否加载过图集
        if (dic.ContainsKey(multiName))
        {
            //判断图集中是否有对应切片
            if (dic[multiName].ContainsKey(spriteName))
            {
                return dic[multiName][spriteName];
            }
        }
        else
        {
            Dictionary<string, Sprite> dicTmp = new Dictionary<string, Sprite>();
            Sprite[] spriteArray = Resources.LoadAll<Sprite>(multiName);

            for (int i = 0; i < spriteArray.Length; i++)
            {
                dicTmp.Add(spriteArray[i].name, spriteArray[i]);
            }

            dic.Add(multiName, dicTmp);

            if (dicTmp.ContainsKey(spriteName))
            {
                return dicTmp[spriteName];
            }
        }

        return null;
    }

    public void ClearInfo()
    {
        dic.Clear();
        Resources.UnloadUnusedAssets();
    }
}
```



## SpriteMask

SpriteMask为图片遮罩，可以只显示图片的一部分

需要修改Sprite的`Mask Interaction`后才能显示

Sprite的层级需要在Back~Front的层级之间，才会有遮罩效果

`SortingLyaer`每个新的层级都要比之前的层级高，新层级的`OrderInLayer`比上一次的所有`OrderInLayer`都要高

案例：实现类似放大镜的功能

```C#
//OrderInLayer1：原始缩放大小的Sprite，如果放大的图片没有完全遮住原始图片，建议Mask Interaction设置为Visible Outside Mask
//OrderInLayer2：放大后的Sprite
//OrderInLayer3：放大镜的遮罩，Sprite Mask
//OrderInLayer4：放大镜的图片，可以控制遮罩的位置，遮罩为放大镜图片的子物体
```



## SoringGroup

`SoringGroup`的子物体会受到`SoringGroup`排序的影响，优先级比`SortingLyaer`更高

`SoringGroup`子物体也能添加`SoringGroup`，优先级递减，父级更高

优先级`SoringGroup` >`子物体SoringGroup` > `SortingLyaer` > `OrderInLayer`



## SpriteAtlas

图集可以减少`DrawCall`，例如将许多较小的图片合并为一个较大的图片，那么CPU就不用一直依次发送多个命令去加载和渲染多个小图片，只需发送一次`DrawCall`，加载渲染合并的图片即可

需要在`Edit-ProjectSetting-Editor-Sprite Packer`中开启，非2D项目不会打开，其中各个模式的区别：

```C#
//V1为旧版Sprite图集，V2为新版，V1可以设置图片的间隔，为2的n次方
//Sprite Atlas V1 - Enable For Builds：在打包时构建图集，Editor模式下不打包图集
//Sprite Atlas V1 - Always Enabled：打包时构建图集，Editor模式下，运行之前会打包图集

//Sprite Atlas V2 - Enable For Builds：打包时构建图集，Editor模式下不打包图集
//Sprite Atlas V2 - Enabled：打包时构建图集，Editor模式下，运行之前会打包图集
```

参数：

```C#
//Type
Maseter：主图集;
Variants：图集变体;

//Include In Build：构建项目时打包图集
//Allow Rotation：打包图集时是否允许旋转，可以增加打包图片的数量，UI需要禁用该选项，会导致UI旋转
//Tight Packing：打包图集时使用图片的轮廓而非图片的矩形，可以提高图片密度

//Variants图集变体参数

//Master Atlas：关联的主图集
//Include In Build：打包变体图集，使用变体图集需要关闭主图集的该选项
//Scale：缩放大小，变体图集在主图集的基础上进行缩放(0 ~ 1)，生成新的图集
```

打包图集后，拖拽UI和Sprite即可使用

若需要单独加载某个图集的某张图片：

```C#
SpriteAtlas sa = Resources.Load<SpriteAtlas>("name");
sa.GetSprite("spriteName");
Sprite[] spriteArray = sa.GetSprites();
```

注意：若三张Spire重叠放置，最上方和最下方图片在同一图集，中间图片在另一图集，启动后，DrawCall为3，若没有中间的图片，DrawCall为1，只渲染两次无法遮挡中间的图片

渲染时会从下向上渲染，会依次判断图片的层次和图集



# 3、2D物理系统

## 刚体

刚体用于物理模拟力的效果

```C#
//Body Type：刚体的类型
Static：静态物体;
	
	

Kinematic：运动学刚体，固定路径或规则运动的物体，不受力的影响，如移动的平台，墙壁，GTA的火车等;
	//Simulated：开启后会当作无限质量的物体
	//Use Full Kinematic Contacts：开启后，会和所有的2D刚体碰撞；禁用后，只和Dynamic刚体碰撞，不和Kinematic和Static刚体碰撞

Dynamic：动态物体;
	//Use Auto Mass：自动设置物体质量，根据物体和子物体所有碰撞体面积大小确定
	//Mass：质量
	//Linear Drag：阻力系数
	//Angular Drag：角阻力
	//Collision Detection：
		Discrete：离散检测，只检测每帧的位置，速度快可能穿过对象;
		Continuous：连续检测，会检测上一(几)帧的位置，穿过物体会回退;
	//Sleeping Mode：对象静止时，进入睡眠模式
		Never Sleep：从不休眠，一直检测;
		Start Awake：一开始处于唤醒状态，静止后睡眠;
		Start Asleep：一开始处于睡眠状态，被碰撞后唤醒;
	//Interpolate：物理更新间的插值运算
		None：无;
		Interpolate：根据前一帧进行平滑;
		Extrapolate：根据后一帧进行平滑;

//通用参数
	//Material：物理材质
	不设置材质会使用Editor-ProjectSetting-Physics 2D中的默认物理材质，子物体的Collider不设置物理材质，会使用父物体RigidBody的物理材质;
	物理材质的优先级：Collider 2D > RigidBody 2D > Physics 2D;
	
	//Simulated：开启，2D刚体和所有子对象2D碰撞器、2D关节(Joint)都能模拟物理效果
	
```

常用API

```C#
Rigidbody2D rb = GetComponent<Rigidbody2D>();
rb.AddForce(new Vector2(1, 1));
rb.AddVelocity(new Vector2(1, 1));
rb.AddRelativeForce(new Vector2(1, 1));
rb.AddTorque(1);
rb.AddTorque(-1);
```



## 碰撞器

碰撞器用于表示物体的体积和形状

```C#
//Circle Collider 2D
	//Used By Effector：是否被附加的2D效应器使用

//Box Collider 2D
	//Used By Composite：勾选，该碰撞器会添加到2D复合碰撞器中使用，会丢失一些属性，这些属性统一在复合碰撞器中设置
	//Auto Tiling：Sprite为Tiled平铺模式，勾选后，改变Sprite大小时，将自动更新碰撞器大小
	//Edge Radius：圆角半径

//Polygon Collider 2D
	//Points：多边形顶点

//Edge Collider 2D：边界碰撞器，由线段组成，常用于地形，可以修改点

//Capsule Collider 2D
	
//Composite Collider 2D：复合碰撞器，会将子物体的碰撞器合并生成新的碰撞器，避免重复计算，可以节约性能
	//Geometry Type：几何学类型，合并碰撞器时，碰撞器顶点的组合方式
		OutLine：空心轮廓;
		Polygons：实心多边形;
	//Generation Type：生成类型，复合碰撞器什么时候生成新的几何体
		Synchronous：对2D复合碰撞器或使用其他碰撞器修改时，立即同步生成几何体;
		Manual：手动生成，会添加按钮;
	//Vertex Distance：复合碰撞器收集的顶点的最小距离，小于该距离会合并顶点
```



## 物理材质

物理材质用于控制物体碰撞的弹性系数和摩擦力

具体效果由两个碰撞物体的物理材质共同计算，其中一个摩擦系数为0，那就是光滑无摩擦，碰撞的弹性系数会叠加计算，两个都为1会越弹越高



## 恒定力

`Constant Force 2D`

恒定力会持续为2D物体施加力，可以用于制作如火箭发射等效果

恒定里会线性为对象施加力和扭矩，让其移动或旋转

```C#
//Force：施加于物体上的线性力，方向为世界坐标
//Relative Force：施加于物理的线性力，用刚体的本地坐标计算方向
//Torque：施加的扭矩，面向Z轴方向，> 0 为逆时针方向，< 0 为顺时针方向，因为Unity为左手系，正方向为X轴转向Y舟，拇指朝Z轴方向
```



## 效应器

2D效应器配合2D碰撞器使用，可以让对象在接触时产生一些特殊的作用力，如传送带、互斥、吸引、漂浮、单向碰撞等

```C#
//区域效应器：Area Effector 2D
//添加后还需要Collider 2D，打开Used By Effector，并设为Trigger
	//Use Collider Mask：开启后可以选择作用的层级Layer
	//Use Global Angel：使用全局坐标，还是效应器所在物体的本地坐标
	//Force Angel：力的角度(方向)，0为x轴方向
	//Force Magnitude：力的大小
	//Force Variation：力的随机变化
	//ForceTarget：效应器施加力的作用点
		RigidBody：施加在质心，不会产生扭矩;
		Collider：施加在碰撞器中心，会产生扭矩;
	//Damping：
		Drag：阻力;
		Angular Drag：扭矩阻力;

//浮力效应器：Buoyancy Effector 2D，模拟流体行为，浮力阻力，看起来像在水中移动
//添加后还需要Collider 2D，打开Used By Effector，并设为Trigger
	//Density：流体密度，密度越大，浮力越大
	//Surface Level：流体表面的位置，高于表面值，不会受到浮力，沿着世界坐标Y轴偏移
	//Flow Angel：流体方向
	//Flow Magnitude：流体力的大小
	//Flow Variation：流体力的随机变化
	
//点效应器：Point Effector 2D
//用于模拟磁铁的相吸相斥
//添加后还需要Collider 2D，打开Used By Effector，并设为Trigger
	//Distance Scale：产生力距离缩放，会参与计算，影响力的大小
	//ForceMode：
		Constanct：范围内固定力，忽略距离影响;
		Inverse Liner：范围内力线性变化，越远力越小;
		Inverse Squared：范围内力反平方变化，越远力越小，呈现指数下降;

//平台效应器：Platform Effector 2D，可以从平台下方跳到平台上
//用于移动平台或攀爬跳跃的墙壁，可以从效应器一边穿透到另一边，Collider 2D 需要关闭Trigger
	//Rotational Offset：旋转偏移量，控制平台的角度
	//Use One Way：是否开启单向碰撞，开启后能从一边穿透到另一边，不产生碰撞，关闭后不能穿透
	//Use One Way Grouping：平台有多个碰撞器时，可以将所有碰撞器都设为单向碰撞
	//Surface Arc：能够穿透和不能穿透的范围，范围内是禁止穿透
	//Use Side Friction：是否在平台两侧应用摩擦，关闭后可以防止角色卡在平台边缘
	//Use Side Bounce：是否在平台两侧应用碰撞，关闭后可以防止角色被平台边缘撞飞
	//Side Arc：定义平台两侧的作用弧度范围

//表面效应器：Surface Effector 2D
//用于模拟传送带，Collider 2D 需要关闭Trigger
	//Speed：传送带表面速度
	//Force Scale：传送带作用力的缩放，越大越快
	//Use Contact Force：是否开启接触面的力，开启，对象会旋转，关闭不会
	//Use Friction：是否开启摩擦力
	//Use Bounce：是否开启弹力


```



# 4、Sprite Shape

Sprite Shape方便以节约美术资源，可以用单张图片，重复拼接成游戏场景地形或背景，可以弯曲，类似Tilemap

需要在`Package Manager`中导入`2D Sprite Shape`

## Sprite Shape Profile

`Sprite Shape Profile`：`Sprite Shape`的配置文件

```C#
//Use Sprite Border：是否使用Sprite的边框，用于Sprite的九宫格拉伸
//Texture：Texture图片的模式Wrap Mode必须是Repeat，用于填充封闭图形内部
//Angle Range：角度范围，一般用于封闭的图形，可以设置不同角度范围使用的Sprite
	Start：起始角度;
	End：结束角度;
	Order：Sprite接触时的优先级，谁在上方;	
//Sprites：角度范围内的Sprite列表，角度范围内使用那些Sprite图片
//Corners：用于封闭图形转折四个角的图片
```



## Sprite Shape Renderer

```C#
//Mask Interaction：遮罩
```



## Sprite Shape Controller

```C#
//Adaptive UV：自适应UV，开启后会自动判断Sprite为平铺还是拉伸
//Enable Tangents：启用切线计算，shader中需要切线信息开启
//Cache Geometry：缓存集合体数据，会增加内存消耗
//Corner Threshold：角阈值，当拐角达到特新角度时，使用边角图片
//Stretch UV：拉伸内部图片的UV，不会进行平铺
```



## 生成碰撞器

1、使用边界碰撞器

添加边界碰撞器后，碰撞的边界会自动匹配Sprite Shape，弧形也能匹配

但是内部空心



2、使用多边形碰撞器 + 复合碰撞器

添加多边形碰撞器后也会自动匹配，内部实心

再添加复合碰撞器，多边形碰撞器开启`Used By Composite`可以合并多边形碰撞器的三角面，减少性能消耗

两者同时添加后效果类似边界碰撞器

注意：复合碰撞器会添加刚体，需要设置刚体的类型





# 5、Tilemap

需要在`PackgeManager`中导入`2D Tilemap Editor`



## 瓦片资源

`widnow-2D-Tile Palette`打开后，创建`new Palette`，将图片拖入即可创建瓦片资源，可以多选批量拖入



## 调色板

橡皮：画笔+`shift`

注意：制作45度俯视角时，需要在`Edit-Graphics-Transparency Sort Mode`中修改为`Custom Axis`，参数为`(0, 1, -0.26)`，然后将`Tilemap Renderer`的`Mode`设置为`Individual`；在画板下方，勾选`Can Change Z Position`，即可不修改Sprite中心点的位置，设置地面的图片的高度，快捷键：`+` `-`，在修改Z高度后，可以在同一个网格位置绘制不同Z高度的图片，类似漂浮了一个平台或地下平台



## 碰撞器

为`Tilemap`添加`Tilemap Collider 2D`，将瓦片资源的`Collider Type`设置为Sprite，会根据Sprite形状生成碰撞器，Grid会根据格子大小生成碰撞器

再添加`Composite Collider 2D`可以复合多个网格，减少消耗



## 拓展包

https://github.com/Unity-Technologies/2d-extras

拓展包为Tilemap添加了新的瓦片类型和笔刷类型

2022.3版本已经自带拓展



## 新增瓦片类型



### Rule  Tile

定义不同方向是否存在连接图片的规则，根据邻近图片的类型，自动设置当前图片

根据九宫格周围是否有图片进行判断，绿色箭头为其余地方有图片，红色×为没有图片

```C#
//Default Sprite：不满足规则时，默认显示的图片
//Default GameObject：默认关联的游戏对象，模型、Sprite都可以，例如地面上关联一棵树
```



### Animated  Tile

可以播放序列帧动画的瓦片



### Pipeline  Tile

根据相邻瓦片的数量更改瓦片

可以快速绘制管道



### Random Tile

随机瓦片



### Terrain Tile

地形瓦片

类似Rule Tile，不过提前定义好规则



### Weighed Random Tile

权重随机瓦片



### Rule Override Tile

高级规则覆盖瓦片

改变图片或指定启用的规则瓦片

相当于重载一个Rule Tile



## 新增笔刷



### Prefab Brush

快速绘制预制体，3D、2D都可以，可以携带脚本



### Random Prefab Brush

随机预制体笔刷



### Random Brush

随机笔刷，可以指定瓦片进行关联，随机刷出瓦片，可以自定义瓦片的大小、数量



## 常用API

```C#
Tilemap map;	//瓦片地图
TileBase tileBase;  //瓦片资源的基类;
Grid grid;	//格子，可以进行坐标转换

//清空瓦片地图
map.ClearAllTiles();

//获取指定坐标的格子
tileBase = map.GetTile(new Vector3Int(1, 1, 1));

//设置瓦片
map.SetTile(new Vector3Int(0, 0, 0), tileBase);

//删除瓦片
map.SetTile(new Vector3Int(0, 0, 0), null);

//坐标转换
grid.WorldToCell(new Vector3(0, 0, 0));	//世界坐标转格子坐标，可用于鼠标点击-转屏幕坐标-转世界坐标-转格子坐标
```



# 动画



## 动画状态机

参数

```C#
Has Exit Time;	//当前动画播放完毕，才会进行转换，动画硬直

```

常用API

```C#
Animator animator;

animator.SetTrigger("name");
animator.SetBool("name", true);
animator.SetInt("name", 1);
animator.SetFloat("name", 1.0f);

animator.GetBool("name");
animator.GetInt("name");
animator.GetFloat("name");

animator.Play("name");


```



# 2D动画

## 序列帧动画

为了避免帧丢失，创建动画后，先修改动画播放的帧率，再将序列帧Sprite拖入`Animation`窗口



## 2D骨骼动画

需要导入`2D Animation`

在SpriteEditor中选择Skinning Editor来编辑Sprite的骨骼

 编辑完成后，拖入场景，添加`Sprite Skin`组件，点击`Create Bones`，即可生成并控制对象的骨骼

图集类型的Sprite需要在编辑完成骨骼后，自行组合身体的各个部分，将各个部分拖入到场景中并自己分层，每个部分都要单独添加`Sprite Skin`



Psb文件编辑骨骼

PSD和PSB都为PS的专用格式，没有太大区别，PSD兼容一些其他软件，PSB只能用PS打开

unity官方建议使用PSB格式

需要导入`2D PSD Importer`

PSB文件已经分层，各个Sprite的位置已经设置，便于使用

文件参数

```C#
//Extrude Edges：图片边缘延申网格
//Import Hidden：是否导入PSB中隐藏的图层
//Moszic：开启后，生成Sprite，并将各个图层合并为单个纹理
//Character Rig：是否使用人物已经绑定的骨骼
//Use Layer Grouping：使用PSB中的图层进行分组
//Reslice：重新生成Sprite，清除对Sprite的所有修改，启用Moszic后可开启
```

可以设置每个Sprite受影响的骨骼



### IK

需要在PackageManager导入`2D IK`

编辑完骨骼后，将对象拖入场景，添加`IK Manager 2D`

点击加号，添加不同的IK

```C#
//Chain (CCD)：
//Chain (FABRIK)：
//Limb：
```

使用CCD后，会创建一个子物体CCD Solver用于设置IK，在Effector中放入末端受影响的节点，设置好参数后，点击生成，即可生成末端的IK对象，通过控制该对象的位置，即可控制IK

Farbik和CCD的区别是，可以反向拖动，可以上下颠倒

Limb默认会关联三个骨骼节点，常用于制作四肢的IK

每个IK节点需要单独创建，例如四肢需要单独创建四个IK节点



IK常用于有指向性的功能，如瞄准，头部朝，地面矫正向等



## 换装

#### 换装资源在同一文件

在PS制作时，将所有的换装资源在一个PSB文件中，都摆放好位置

在SpriteEditor中分别编辑骨骼即可

同一部位的不同装备需要进行分组



将对象拖入场景后，会自动添加`Sprite Library`组件，用于设置分组的信息

每个部位会添加`Sprite Skin`和`Sprite Resolver`，用于切换不同的Sprite

常用API

```C#
public SpriteResolver sr;
sr.SetCategoryAndLabel(sr.GetCategory(), "name");
```



#### 换装资源在不同文件

各个部位在PS中需要统一，不同文件的骨骼信息必须统一，不同套装，隐藏主体骨骼后，分别导出为不同的PSB文件

因为不同文件的骨骼信息必须统一，所以编辑好基本的骨骼后，复制到其他装备文件，然后重新关联骨骼

需要手动创建`Sprite Library Asset`，然后将不同装备按位置部位进行分组

场景对象需要添加`Sprite Library`，并关联之前创建的`Sprite Library Asset`

然后在场景对象的各个部位添加`Sprite Resolver`，选择对应的图片



### Spin动画

在Spine官网下载并导入unity的运行库，将下载的packge文件拖入asset即可



Spine骨骼

Spine导出的Unity资源有三个文件：.json，存储骨骼信息；.png，存储图片图集；.atlas/.txt，图片在图集中的位置信息

把这三个资源导入unity后，会自动生成 Atlas、Material、SkeletonData文件

将SkeletonData拖入场景即可创建对象



文件参数

骨骼数据文件

```C#
//SkeletonData JSON：对应的骨骼数据文件
//Atlas Asset：图集资源
//Mix Setting
	Animation State Data：动画状态数据;
	Default Mix Duration：默认混合(过渡)持续时间;
	Add Custom Mix：添加自定义混合(两个动画间的状态过渡)
```



常用API

```C#
public SkeletonAnimation sa;

//播放动画
sa.loop = false;	//先修改循环，再设置动画
sa.AnimationName = "name";

sa.AnimationState.SetAnimation(0, "name", false);	//参数三为是否循环播放
sa.AnimationState.AddAnimation(0, "name", false, 0f);	//当前动画播放完毕后，切换动画，参数四为延迟播放时间

//转向
sa.skeleton.ScaleX = -1;

//动画事件
//参数为动画的相关信息
sa.AnimationState.Start += (trackEntry) =>
{
    Debug.Log(sa.AnimationName + "开始播放");
    ...
};

sa.AnimationState.End += ...;	//动画中断或清除
sa.AnimationState.Dispose += ...;
sa.AnimationState.Complete += ...;	
//制作动画时，添加的自定义事件
sa.AnimationState.Event += (trackEntry, event) =>
{
    ...
};	

//便捷特性
[SpineAnimation]
public string animationName;	//添加特性后，可以直接通过枚举的方式，在Inspector窗口中选择动画
[SpineBone]	//添加特性后，可以直接通过枚举的方式，在Inspector窗口中选择骨骼、IK
[SpineSlot]	//添加特性后，可以直接通过枚举的方式，在Inspector窗口中选择插槽
[SpineAttachment]	//添加特性后，可以直接通过枚举的方式，在Inspector窗口中选择附件

//获取骨骼
Bone b = sa.skleton.FinBone("boneName");

//设置插槽附件
sa.skeleton.SetAttachment("slotName", "attachmentName");
```

注意：对象可以在UI中使用，拖入到场景中时选择UI类型即可，可以根据UI来缩放

# 3D动画

## 模型导入

模型软件导出时，任务面朝Z轴方向，头顶为Y轴方向，人物右侧为X轴方向

模型文件夹层级

主文件夹 —— 名称

- Animation
- Animator
- Materials
- Models
- Prefabs
- Textures



## 模型文件

### Model

参数

```C#
Convert Units;	//将模型文件的比例转换为Unity的比例，.fbx .max .jas = 0.01, .3ds = 0.1, .mb .ma .blend .dfx .dae = 1
Import BlendShaps; //是否导入混合形状，将使用Skinned MeshRenderer
Preserve Hierarchy;	//始终创建一个显示预制体根，通常导入时，FBX会将模型中空的根节点去掉，但若多个FBX文件包含同一空节点，可以勾选来保留
				  //例如：两个fbx文件，1有骨骼和网格，2只有骨骼动画，若不启用该参数，那么unity会剥离根节点，导致动画无法正常播放
Mesh Compression;	//网格压缩
Read/Write Enabled;	//开启后，unity将网格数据传递给GPU后，在CPU中还会保留可寻址的内存，即可以通过代码访问网格数据进行处理
				   //读取写入网格数据、合并网格、使用网格碰撞、使用NavMesh时，开启
Optimize Mesh;	//确定三角形在网格中列出的顺序以提高GPU性能，Everything：对顶点和多边形索引重新排序，其余两个仅为单独排序
Generate Colliders;	//生成网格碰撞体，会移动的对象不要开启

Weld Vertices;	//合并空间中相同位置的顶点，前提是这些顶点的UV、法线、切线等属性相同
```



### Rig

Animation Type

Legacy：旧版动画系统，一般不用

Humanoid：人形模型

Generic：通用，非人形模型，需要设置骨骼根节点



### Animation

基础信息

```C#
Anim.Compression;	//导入动画的压缩类型
	off;	//关闭压缩
	KeyFrame Reduction;	//减少冗余帧
	Optimal；	//让unity自动决定如何压缩，仅适合Generic和Humanoid
Animated Custom Properties;	//导入自定义的fbx属性
```



动画剪辑片段属性设置

一个模型的动画可以很长，动画剪辑从中截取一段作为动画(Animation Clip)

可以自定义剪辑，自定义动画的开始和结束

```C#
Loop Time;
	Loop Pose;	//播放动画时无缝循环，勾选LoopTime后才有，会自动过度AnimationClip的头尾
	Cycle Offset;	//第二次及之后循环时，动画开始播放的位置，例如跑步动作有起步动作，后面循环跑步可以跳过起步动作

Root Transform Rotation;	//根节点位置的角度相关
	Bake Into Pose;	//将旋转烘焙到骨骼移动，关闭时存储为根节点移动，当模型自带旋转时，例如走路右转，勾选后，只有模型的动作，不会让根节点产生位移
					//关闭后，根节点会旋转和位移
	Base Upon;	//根节点旋转的基础
		Original：保持文件中的原始旋转;	//模型Z轴正方向不会跟随模型的转动改变，始终朝向一个方向
		Root Node Rotation：使用根节点的选择，仅Generic可用;	//模型Z轴正方向会跟随模型的转动改变
		Body Orientation：保持上半身朝向，仅Humanoid可用;	//模型Z轴正方向会跟随模型的转动改变
	Offset;	//根旋转偏移角度，偏移动画的朝向
		
Root Transform Position(Y);	//根垂直运动相关
	Bake Into Pose;	//动画如果存在Y方向的位移，关闭选项，会应该模型的根节点，让模型上下位移，transform.position.y会上下变化，
				  //勾选后，不会影响模型Y方向的位置，只会让动画上下位移，transform.position.y不变
	Base Upon(at Start);	
		Origin：保持原始文件中的垂直位置;
		Root Node Position：使用垂直根节点位置，仅Genetic可用;
		Center of Mass：保持质心与根节点对其，仅Humanoid可用;
		Feet：保持双脚与根节点对其，仅Humanoid可用;
	Offset;	//在Y方向上的位置偏移

Root Transform Position(XZ);	//根水平运动相关
	Bake Into Pose;	//同理，动画如果存在XZ方向的水平位移，关闭选项，会应该模型的根节点，让模型水平位移，transform.position.x/z会变化，
				  //勾选后，不会影响模型XZ方向的位置，只会让动画水平位移，transform.position.x/z不变
	Base Upon(at Start);	
		Origin：保持原始文件中的水平位置;
		Root Node Position：使用水平根节点位置，仅Genetic可用;
		Center of Mass：保持质心与根节点对其，仅Humanoid可用;
	Offset;	//在XZ方向上的位置偏移、

Mirror;	//镜像动画，仅Humanoid有
Additive Reference Pose;	//启用后，可以设置附加动画层基础参考姿势的帧，附加动画层在状态机中可以添加
	Pose Frame;	//启用参考姿势时，用的哪一帧
```

其它参数

```C#
Curve;	//设置动画在动画状态机里对应的参数变化，由曲线来控制，参数的值会根据曲线来变化，添加更多的曲线，会添加更多的参数，只读
Events;	//动画事件，动画播放到指定位置时，会寻找指定脚本中的同名方法
Mask;	//动画遮罩，让哪些骨骼、IK不受动画的影响，例如腿部受伤，可以让一只腿不受走路动画的影响，非人形动画，需要单独创建并指定骨骼
Motion;	//指定骨骼节点作为动画的根节点
```



### Materials

```C#
Material Creation Mode;
	Standard：使用Unity默认规则生成材质;
	Import Via Material Description：使用模型嵌入材质;	//支持广、结果准确

sRGB Albedo Colors;	//是否在伽马空间中使用反射率着色，线性空间的项目关闭该选项

Location;	//如何访问材质和纹理
	Use Embedded Material：将导入的材质保持到模型资源中;	//也可点击按钮导出到项目文件夹中
	Use External Material：将导入的材质提取为外部资源;

Naming;	//材质的命名规则
	By Base Texture Name：根据材质的纹理来命名材质;
	From Model's Material：使用导入材质的名称;
    Model Name + Model Material：模型名称 + 材质名称;

Search;	//使用Naming选项时定义的名称查找文件位置的规则
	Local Materials Folder：模型所在文件夹的Materials文件夹;
	Recursive-Up：所在父文件夹(一直追溯到Assets文件夹)的所有Materials文件夹;
	project-Wide：所有项目文件夹查找材质;
```



## 3D动画基础

动画状态机中动画的参数

```C#
Multiplier;	//控制动画速度的乘数，需要开启Parameter并绑定float类型的参数，例如物体移动速度越快，动画播放越快，坦克履带动画、跑步动画等
Motion Time;	//运动的时间，需要开启Parameter并绑定float类型的参数，强制控制当前动画播放的时间，会暂停动画的播放
Mirror;	//镜像播放动画，可以绑定Bool类型的参数，仅适用Humanoid
Cycle Offset;	//循环偏移时间，可以绑定float类型的参数，循环播放动画时，从哪里开始继续播放
Foot IK;	//是否遵循Foot IK，仅Humanoid可用
Write Default;	//Animator States是否为其运动执行未动画化的属性写回默认值
Transition;
	Solo：仅播放该过渡;	//有相同参数条件时，优先播放该动画
	Mute：禁用过渡;	//即使满足参数的条件，动画也不会切换
	Solo & Mute：Mute优先;
```

动画连线过渡参数

```C#
//动画间的连线
Has Exit Time;	//勾选时，即使满足切换条件，动画必须播放到设定的ExitTime才切换动画，例如闪避或技能后摇
Exit Time;	//退出动画的时间，<1为播放动画的百分比退出，>1为循环几次后退出
Fixed Duration;	//勾选后，Transition Duration将以秒为单位，不勾选，以百分比为单位
Transition Duration;	//动画过渡持续的时间，为下方两个蓝色箭头之间的区域
Transition Offset;	//过渡到目标状态的起始播放的时间偏移，如0从目标状态开头播放，0.5从一半开始播放
Interruption Source;	//过渡中断，状态切换过快时采用的方案
	None：无过渡;
	Current State：将当前过渡排队;
	Next State：使用下一个状态进行过渡;
	Current State Then Next State：当前状态过渡，下一状态过渡依次排队;
	Next State Then Current State：下一状态过渡，当前状态过渡依次排队;

//AnyState和动画的连线
Can Transition To self;	//是否可以过渡到自己，任意动作是否包含动画状态本身，从自身动画切换到自身动画
Preview Source State;	//预览过渡状态，因为AnyState代表任意一种动画状态，需要设置每个状态的切换
```







## 动画分层和遮罩

动画分层的目的：两套不同动作切换，动画组合播放，节约资源

例子：角色血量正常，播放正常移动的动画，角色血量低，播放虚弱状态的动画

动画分层需要和动画遮罩结合使用

例子：站立射击，移动射击，下蹲射击，开枪的动画只影响上半身，和下半身无关，这样就不必制作三种开枪动作，只用一种即可



动画层级的参数

```C#
Weight;	//权重，当动画播放时，如果层级为叠加状态，会根据权重来设置不同层级动画叠加的比例
Mask;	//动画遮罩，该层所有动画能够影响的范围，未被影响的位置会播放上一层级的动画
Blending;	//混合方式
	Override：覆盖，会覆盖其他层的动画，只播放当前层的动画;
	Additive：叠加，会和其他层级的动画像混合叠加;
Sync;	//是否同步其他层，用于直接从其他层级复制过来，在该层进行修改，如正常状态下的移动动画和受伤状态下的移动动画
		//开启后会多一个Source Layer表示复制哪一层
Timing;	//开启，采取折中方案调整同步层上的动画时长，关闭，动画时长和原始层级相同
IK Pass;

animator.SetLayerWeight(animator.GetLayerIndex("name"), 0.5f);	//层级索引，权重
```



## 动画混合

创建动画混合树Blend Tree，在动画状态机中右键点击创建即可

双击即可进行编辑

### 1D混合

1D混合自带一个`float`参数

```C#
Threshold;	//对应动画的阈值，当参数等于该值时，该动作权重最大，可以随意设置该阈值
时钟图案;	//动画播放的速度
人体图像;	//是否镜像
Automate Threshold;	//自动设置阈值，取消后可以手动设置
Compute Threshold;	//计算阈值的方式
	Speed：速度;	//会从Animation Clip的根节点中去获取数据
	Velocity X：X方向上的速度;		
	Velocity Y：Y方向上的速度;	
	Velocity Z：Z方向上的速度;	
	Angular Speed(Rad)：角速度(弧度);	//会从Animation Clip的根节点中去获取数据
	Angular Speed(Degree)：角速度(角度);
Adjust Time Scale;	//调整时间刻度
	Homogeneous Speed：均匀速度;	//设置不同动画Motion的播放速度，会让移动速度一致，依此来设置动画的速度
	Reset Time Scale：重置时间刻度;	//重置为1
```



### 2D混合

```C#
2D Simple Directional;	//2D简单定向，运动表示不同方向时使用，如前后左右移动
2D Freeform Directional;	//2D自由形式定向，同上，但在一个方向可以有多个运动。如走和跑
2D Freeform Cartesian;	//2D自由形式笛卡尔坐标模式，运动不表示不同方向时使用，如向前走/跑不拐弯，向前左右转
Direct	//直接模式，自由控制每个节点权重，一般用作表情控制
```



## 动画子状态机

子状态机为状态机里的状态机，用于实现多种状态组合而成的复杂状态，如一个技能由三段动画组成，跳起、攻击、落下，释放该技能时会连续播放这三个动作，可以将这三个动作放入子状态机中

在动画状态机中点击右键`-Create Sub-State Machine`

子状态机和外部的连接方式：

```C#
返回Base Layer;	//会从外部状态机的Entry重新进入状态机，播放默认动画
返回指定State状态;	//直接切换到外部状态机对应的状态，不再从外部状态机的Entry进入
```



## 动画IK控制

Inverse Kinematics，反向动力学，由骨骼末端影响父骨骼



如何控制IK

在状态机的层级中，开启IK Pass

继承MonoBehavior，Unity定义了一个IK回调方法：OnAnimatorIK，在该方法中使用Unity提供的IK相关的API来控制IK

```C#
[SerializeField] private Transform target;
private Animator animator;

private void OnAnimatorIK(int layerIndex)	//会传入当前播放动画的层级
{
    //全局权重，
    //身体权重，
    //头部权重，
    //眼部权重，    
    //ClampWeight:0无限制运动，看向目标时，会转180°，1固定无法移动，不执行LookAt，如防止角色身体头部转180°，可以设置为0.5、0.6
    //在编辑Humanoid时，可以控制骨骼的移动范围
    //SetLookAtWeight只能控制角色的上半身
    //如鼠标控制角色上半身进行射击瞄准    
	animator.SetLookAtWeight(1f, 0.5f, 1f, 0f, 0.5f);
    //如角色头部看向目标位置
    animator.SetLookAtWeight(1f, 0f, 1f, 0f, 0.5f);
    animator.SetLookAtPosition(target.position);
    
    //设置四肢IK
    //设置IK位置权重
    animator.SetIKPositionWeight(AvatarIKGoal.LeftHand, 1f);	//AvatarIKGoal为四肢的枚举
    //设置IK角度权重
    animator.SetIKRotationWeight(AvatarIKGoal.LeftHand, 1);
    //设置IK位置
    animator.SetIKPosition(AvatarIKGoal.LeftHand, target.position);
    //设置IK旋转
    animator.SetIKRotation(AvatarIKGoal.LeftHand, target.rotation);
}
```

IK的用处：

拾取物品

射击瞄准

楼梯、斜面脚部对齐

`OnAnimatorIK`在`Update`和`LateUpdate`之间调用，在每帧状态机和动画处理完之后调用，`OnAnimatorIK`在`OnAnimatorMove`之前调用

`OnAnimatorMove`用于动画存在位移，同时需要用代码添加位移时使用



## 动画目标位置匹配

跳跃或移动时，需要到达或限制在指定位置

```C#
[SerializeField] private Transform matchingPos;
private Animator animator;

private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        animator.SetTrigger("Jump");
    }
}

//动画添加事件并调用，在开始位移之前调用
public void MatchPos()
{
    //1、目标位置
    //2、目标角度
    //3、匹配的骨骼位置
    //4、位置角度权重
    //5、开始位移动作的百分比
    //6、结束位移动作的百分比
    animator.MatchTarget(
        matchingPos.position,
        matchingPos.rotation,
        AvatarTarget.LeftFoot,
        new MatchTargetWeightMask(Vector3.one, 1),    //xyz位置的权重；旋转角度权重；
        0.4f,   //动画播放到40%时产生位移
        0.65f   //动画播放到65%时结束位移
    );
}
```

注意：

1、调用匹配动画时，必须保证动画已经切换到目标动画

2、必须保证调用时动画不是处于过渡阶段，而是在播放目标动画

3、需要开启Apply Root Motion



## 状态机行为脚本

状态机行为脚本是一类特殊的脚本，继承指定的基类，主要用于关联到状态机的状态上，当进入、退出、保持在特定状态时可以进行逻辑处理

就是为Animator Controller状态机窗口中的某一状态添加脚本，可用于：

进入和退出状态时播放声音

仅在某些状态下检测一些逻辑，如是否接触地面

激活和控制一些状态相关的特效



创建新脚本，继承`StateMachineBehaviour`，并在状态机中某个动画上点击`AddBehaviour`添加该行为脚本

```C#
public class PlayerAnimatorState : StateMachineBehaviour
{
    public string name;	//和在MonoBehavior相同，会显示出来
    
    //进入状态，播放动画时的第一个Update中调用
    public override void OnStateEnter(Animator animator, AnimatorStateInfo animatorStateInfo, int layerIndex)
    {
        //判断当前播放动画状态的名称
        if (animatorStateInfo.IsName(name))
        {
            Debug.Log($"进入{name}状态");
        }
    }

    //退出状态，播放动画时的最后一个Update调用
    public override void OnStateExit(Animator animator, AnimatorStateInfo animatorStateInfo, int layerIndex)
    {

    }

    //OnAnimatorIK之后调用，Update之后，LateUpdate之前
    public override void OnStateIK(Animator animator, AnimatorStateInfo animatorStateInfo, int layerIndex)
    {

    }

    //OnAnimatorMove之后调用
    public override void OnStateMove(Animator animator, AnimatorStateInfo animatorStateInfo, int layerIndex)
    {

    }

    //除了第一帧和最后一帧，播放动画时的每个Update调用
    public override void OnStateUpdate(Animator animator, AnimatorStateInfo animatorStateInfo, int layerIndex)
    {

    }

    //子状态机进入时调用，播放动画时的第一个Update调用
    public override void OnStateMachineEnter(Animator animator, int stateMachinePathHash)
    {

    }

    //子状态机退出时调用，播放动画时的最后一个Update调用
    public override void OnStateMachineExit(Animator animator, int stateMachinePathHash)
    {

    }
}
```



## 状态机复用

有n个玩家，n个怪物，他们的动画和状态机行为相同，只是动画不同

在Project窗口中右键创建`Animator Override Controller`，关联复用的状态机和对应的动画即可



# 角色控制器

角色控制器可以让角色受到碰撞，但不会被刚体所牵制，对角色添加刚体，可能会出现：

在斜坡往下划动、不添加约束的情况下可能被撞飞

unity提供了角色控制器`Character Controller`，添加了之后，不必再添加刚体，同时能检测碰撞函数、触发Trigger、可以被射线检测

注意：如果使用CharacterController控制角色移动，建议关闭Animator的Apply Root Motion

参数

```C#
Slope Limit;	//坡度限制，大于该坡度上不去
Step Offset;	//台阶偏移，单位为米，低于该值才能上去，不能大于角色控制器的高度
Skin Width;		//皮肤的宽度，两个碰撞体可以穿透彼此的最大的皮肤宽度，较大的值可以减少抖动，较小会让角色卡住，建议设置为半径的10%
Min Move Distance;	//最小移动距离，大多数情况为0，可以减少抖动

```

常用API

```C#
//受重力的移动
characterController.SimpleMove(transform.forward * speed * Time.deltaTime);

//不受重力的移动
characterController.Move(transform.forward * speed * Time.deltaTime);	//使用该方法会关闭重力影响，无向下方向的速度

//判断是否接触地面
if(characterController.isGrounded)
{...}

//碰撞检测
private void OnControllerColliderHit(ControllerColliderHit hit)
{
    //hit为与角色控制器碰撞的其他对象
}

//不会触发
private void OnCollisionEnter(Collision other)
{...}

//触发器触发
private void OnTriggerEnter(Collider other)
{
	//自身进入其他触发器，会触发，由characterController触发，和刚体无关
    //自身有触发器时，其他带刚体的物体进入，会触发
}
```



# 导航寻路系统

unity导航寻路，在A*基础上进行拓展和优化



## NavMesh

生成导航网格NavMesh

打开`window - AI - Navigation`

新版本的Navigation只有Agent和Areas两个设置

为地面添加`NavMeshSurface`，即可烘焙寻路网格

```C#
AgentType;	//烘焙时使用的Agent
Default Area;	//烘焙的区域的类型
Generate Links;	//生成链接，生成跳跃、下落区域连接
Use Geometry;	//烘焙时使用的网格
	Render Mesh：物体的网格;
	Physics Colliders：物体的碰撞器;
Collect Objects;	//烘焙时，需要处理的对象
	All Game Objects：处理所有对象，默认的物体会作为障碍物处理;
	Volume：处理体积范围内的物体;
	Current Object Hierachy：有NavMeshSurface的子物体;
	NavMeshModifierComponentOnly：只处理有NavMeshModifier的对象;
Include Layers;	//烘焙时处理的层级

```



NavMeshAgent

寻路对象

```C#
AgentType;	//代理类型
Base Offset;	//寻路碰撞器的偏移

Speed;	//寻路时最大速度
Angular Speed;	//寻路时转身最大速度
Acceleration;	//最大加速度
Stopping Distance;	//靠近目标多少距离停止
Auto Braking;	//自动减速，即将到达目标会减速，连续路径点移动建议关闭

Radius;		//用于计算碰撞的半径
Height;		//用于计算碰撞的高度
Quality;	//躲避品质，越高躲避障碍越精准，消耗越大
Priority;	//优先级，0~99，数字越小，优先级越高，优先级低的会忽略避障

Auto Traverse OffMesh Link;	//是否开启自动遍历网格外的其他网格连接，若需自定义判断，关闭
Auto Repeat;	//是否开启自动重设路线，开启时，到达路径后段时，会再次尝试寻路，没有到达目标的路径时，会生成一条到达最近位置的路径
Area Mask;	//寻路时使用的区域
```

常用API

```C#
private NavMeshAgent navMeshAgent;

//设置目标点
navMeshAgent.SetDestination(target.position);

//停止寻路
navMeshAgent.isStopped = true;	//再次移动需要设置为false

//常用参数
navMeshAgent.Speed;
navMeshAgent.acceleration;
navMeshAgent.angularSpeed;

//是否找到路径
navMeshAgent.hasPath;

//是否找到代理目标点
navMeshAgent.desitination;

//当前路径
navMeshAgent.Path;

//是否正在计算路径
if(navMeshAgent.pathPending)
{...}

//路径状态
snavMeshAgent.pathStatus;	//成功、部分(无法到达)、失败

//是否更新位置
navMeshAgent.updatePosition = false;	//不自动移动到目标点，例如先点击，显示移动路径，再点击确认，才移动

//是否更新角度
navMeshAgent.updateRotation = false;

//代理速度
navMeshAgent.velocity;	//可以判断是否停止

//手动寻路
//计算生成路径
NavMeshPath path = new NavMeshPath();
if(navMeshAgent.CaculatePath(Vector3.zero, path))	//计算结果会存储到path中
{
    //计算出路径
    //设置新路径
    if(navMeshAgent.SetPath(path))
    {
        ...
    }
}

//清除路径
navMeshAgent.ResetPath();

//调整到指定点位置
navMeshAgent.Wrap(Vector3.zero);	//直接移动到某个位置
```



OffMeshLink

网格外连接点，如跳跃点

```C#
Start;	//起始点
End;	//终点
Cost Override;	//消耗寻路的权重值，太高的话，会走另一条路，而非跳跃
Bi Directional;	//是否开启双向连接，开启后，可以双向移动，关闭后，只能从Start到End
Activated;	//是否激活
```



NavMeshObstacle

动态障碍物，可用于：

移动的障碍物，可以破坏的门或障碍

```C#
Shape;	//形状
Carve;	//是否雕刻，开启后，会在寻路网格中挖洞，形成不可移动的障碍，固定不动或很少移动的物体可以开启，频繁移动的物体建议关闭
Move Threshold;	//移动阈值，当障碍物移动超过该距离，会更新NavMesh
Time To Stationary;	//障碍物静止所需的时间，移动后，经过该时间，确认静止，更新NavMesh
Carve Only Stationary;	//静止时才更新NavMesh
```

