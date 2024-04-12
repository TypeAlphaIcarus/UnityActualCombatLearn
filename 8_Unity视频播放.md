# Unity视频播放

# 视频格式和编解码器

视频文件包含音频、字幕、轨道信息(不同语言配音)

为了节省带宽和存储，需要使用编解码器

常见编码格式：H.264

常见解码格式：AAC

软件解码可以解码所有格式，但会增加CPU负担

硬件解码使用GPU进行解码



# 视频文件的参数

```C#
sRBG;	//是否开启sRGB，sRBG为色彩标准，可以避免不同设备出现色差
Transcode;	//是否转码
Dimension;	//画面尺寸
	Origin Size：原尺寸;
	Half Res：一半尺寸;	//长宽都是一半
	...
Codec;	//
	Auto：根据目标平台自动设置转码方式;
	H264：mp4解码器，大多数平台支持;
	H265;
	VP8;
Bitrate Mode;	//比特率，影响清晰度
Spatial Quality;	//空间质量，图像在转码时是否压缩，缩小在转码时会压缩，同时播放时会产生模糊
Keep Alpha;	//保留视频的透明通道，有Alpha才显示
Dinterlace;	//去交错设置，如何解除隔行扫描
	off：不进行隔行扫描;
	Even：采用每个帧的偶数行;
	Odd：采用每个帧的奇数行;
```

VP8可以跨平台，安卓原生支持VP8，比H.264消耗更多性能



# VideoPlayer

## 参数

```C#
RenderMode;	//渲染方式
	Camera Far Plane：摄像机的远平面;
	Camera Near Plane：摄像机的近平面;
	RenderTexture：渲染到纹理;
	Material Override：渲染到材质;
	API Only：根据代码设置;
Audio Output Mode;	//播放视频中音频的方式
	Audio Source：会添加Audio Source组件;
	Direct：绕过unity音频处理，直接发送给硬件;
```



## 常用API

```C#
//是否自动播放
videoPlayer.playerOnAwake = false;

//渲染模式
videoPlayer.renderMode = VideoRenderMode.RenderTexture;
videoPlayer.targetTexture = ...

//透明度
videoPlayer.targetCameraAlpha = 0.5f;

//视频源
videoPlayer.source = VideoSource.VideoClip;
videoPlayer.clip = ...
videoPlayer.source = VideoSource.Url;
videoPlayer.url = ...
    
//是否循环
videoPlayer.isLooping = true;

//视频长度
videoPlayer.length;

//当前播放长度
videoPlayer.time;

//总帧数
videoPlayer.frameCount;
    
//当前播放帧数
videoPlayer.frame;

//播放
videoPlayer.Play();

//暂停
videoPlayer.Pause();

//停止
videoPlayer.Stop();

//事件
videoPlayer.prepareComplete += () => {};	//准备完成
videoPlayer.started += () => {};	//开始播放
videoPlayer.loopPointReached += () => {};	//播放到结尾
```



# 全景视频

unity支持的全景视频

1、等距圆柱投影布局，长宽比为2：1的360°内容，和1：1的180°内容

2、立方体贴图布局，宽高比为1：6、3：4、4：3、6：1



使用RenderTexture，RenderTexture分辨率要和视频一样，将DepthBuffert设为No depth buffer

两种布局都使用天空盒，将RenderTexture赋值给天空盒

1、等距圆柱投影布局，将mapping设置为Latitude Longitude Layout

2、立方体贴图布局，将mapping设置为6 Frames Layout

