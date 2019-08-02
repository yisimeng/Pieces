# ARKit

ARKit 底层通过 AVFoundation 和 Core Motion 捕捉并提供图像和运动数据。


ARSessionConfiguration --> ARSession --> ARFrame


1. 首先创建 ARSession。
2. 创建 ARSessionConfiguration 决定使用哪种追踪（平面/面部/世界）
3. 通过 ARSession 的 run 方法开始运行。

AVCaptureSession 和 CMMotionManager 已经在 ARSession底层创建好，用这两个提供图像捕捉和运动数据。

ARFrame 是一个时事快照，包含回话的全部状态，渲染场景所需要的一切。可从ARSession中的 currentFrame 获取，或者通过delegate方法回调中实时获取。

### ARSessionConfiguration

ARSessionConfiguration 提供自由追踪的三个角度，设备的方向。

子类

* ARWorldTrackingSessionConfiguration 提供6个维度追踪。除了设备的方向，还有设备在现实世界中的位置。（使用前判断设备是否支持）

### ARSession

管理增强现实应用中全部进程。

run 启用配置，pause 暂停进程。视图不可见时，可以暂停进程，停止调用CPU，暂停之后恢复进程，再次调用 run即可。

也可以多次调用run，在多个配置中进行无缝切换，不同配置间不会丢帧。

run(configuration, options: .resetTracking) 重设应用。

Session Delegate： 获取最后一帧，错误处理。

### ARFrame

包含用于渲染增强现实场景的所有信息。

* capture image：相机图像，渲染背景
* Tracking information：追踪信息，包含设备方向、位置、追踪状态等。
* Scene information： 场景信息，特征点，空间中的物理位置，光照等。 

### ARAnchor

ARKit 在空间中物理位置的呈现方式是通过 ARAnchor。

* Real-world posiotion and orientation: ARAnchor是一个相对位置，在现实世界中的位置和方向
* ARSession remote/add
* delegate 添加/移除 回调

### ARCamera

每个ARFrame 都包含一个ARCamera，代表虚拟相机对象，呈现设备的方向和位置。

* Transform 提供物理设备的移动旋转的矩阵转换。
* Tracking state 追踪状态
* Camera intrinsics 相机内参，匹配设备的物理相机

### Tracking State

Not Available(首次启动，配置没有完成) ---> Normal ---> Limited（特征不足等） ---> Normal ····


## Scene Understanding

Plane detection
Hit-testing

1. 检测平面
2. 使用碰撞测试，计算物体3D坐标。从设备发送一条摄像，将其与现实世界交叉。
3. 光预计，匹配现实的光照情况

取消识别平面：将planeDetection 设置为none，重新 run。

### ARPlaneAnchor

ARPlaneAnchor 是 ARAnchor 的子类，平面在现实世界中的位置和方向，

delegate：当范围更新时会回调didupdate，可以用于更新视觉展现。

多个平面会合并成一个大的平面，并移除之前的，移除会回调delegate方法 didRemove方法。


ARSCNViewDelegate 


向 Session 中添加新的锚点，ARSCNView会创建一个新的SCNNode。


## ARWorldMap

持久化，多人协同

**World Mapping Status**：not available --> limited --> extending(扩展) --> Mapped

开启环境纹理效果，玻璃制品有反射周围物体的效果。


### Image Detection

首先加载资源图片ARReferenceImage，赋值给ARWorldTrackingConfiguration的detectionImages属性，或者赋值给ARImageTrackingConfiguration的trackingImages属性，然后run session。之后不断回调didUpdateFrame方法，如果检测到image，ARFrame中将包含ARImageAnchor对象。

WorldTracking和ImageTracking 在追踪图像上的区别是跟踪图像的最大数量。

detectionImages 设置为

#### ARReferenceImage

资源文件，从File或者 Asset Catalog中加载 


### Object Detection

**ARReferenceObject**，使用worldTracking，与image detection类似，检测到属性后回调的 ARFrame中携带 ARObjectAnchor.


#### Object Scanning

检测之前需要先扫描object

#### ARObjectScanningConfiguration

为物体识别专门的configuration


### FaceTracking

ARFaceGeometry 包含渲染面部所有信息。


### AROrientationTrackingConfiguration

方向追踪


### BitStream

多点传输WorldMap


## ARKit 3.0

新特性

* people occlusion 任务遮挡
* motion capture 运动捕捉
* collaborative session 协作会话
* simultaneous use of the front and back camera 同时开启前后摄像头
* tracking multiple faces 跟踪不同脸

支持 AR12 及以后的处理器

#### ARConfiguration 的新属性

```
var frameSemantics: ARConfiguration.FrameSemantics {get set}
// 在指定configuration是否可用
class func supportsFrameSemantics(ARConfiguration.FrameSemantics) -> Bool
```

#### ARConfiguration.FrameSemantics

* .personSegmentation 将人物分割渲染到相机图像。可用与虚拟物体在两个人物之间。
* .personSegmentationWithDepth 提供额外的景深，预估人物到相机的距离。

#### ARBodyTrackingConfiguration

#### ARBodyAnchor

#### BodyTrackedEntity


### 同时开启前后摄像头追踪

使用ARWorldTrackingConfiguration，设置 faceTrackingEnabled 为 true，回调 ARFaceAnchor。

使用ARFaceTrackingConfiguration，设置 worldTrackingEnabled为 true，回调 didUpdate frame。

#### Collaborative Session Data 协作会话数据

* 自动交换用户创建的 ARAnchor
* 每个锚点都有会话ID标识，anchor来自与哪个会话ID的哪个设备
* ARParticipantAnchor 代表实时参与者的位置


### AR Coaching View

* Add as a child of any UI View
* Connect to the ARSession
* Optionally set a delegate
* Specify coaching goals in source code

### AR Face Tracking 


### ARPositionalTrackingConfiguration

* 仅适用于跟踪用例
* 低消耗
* 我们可以通过仍然保持渲染率为60赫兹来降低捕获帧速率和相机分辨率，从而实现低功耗


### Scene Understanding Improvements 场景理解改进

图像识别

* 同时识别多达100张图片
* 自动缩放预估，可以预估物理大小，并自动缩放
* 当您想要创建新的AR参考图像时，能够在运行时查询您传递给ARKit的图像的质量

机器学习

* 增强机器学习的物体识别算法
* 快速识别
* 提供了更多的特定场景以训练机器学习。

平面检测

* 机器学习支持平面预估
* 更多的平面模型训练，以更快识别平面
* 不只能识别地面，还能识别墙壁


#### Plane Classification 平面分类

```
enum ARPlaneAnchor.Classification {
	case wall // 墙
	case floor //地面
	case ceiling // 天花板
	case table //桌面
	case seat // 椅子

	// ARKit3.0新增
	case door //门
	case window //窗
}
```

#### RayCasting 

* 更真实的放置物体
* 垂直水平表面对齐
* 光线实时捕捉

### visual coherence enhancements

```
// ARKit3.0新增
class ARWorldTrackingConfiguration {
	var wantsHDREnvironmentTextures: Bool
}

class ARFrame {
	var cameraGrainIntensity: Float
	var cameraGrainTexture: MTLTexture?
}
```



### People Occlusion

* Occlusion between people and rendered content 在人物中间渲染物体
* Supported in RealityKit with ARView 支持ARView
* Backwards compatible with ARSCNView 支持ARSCNView
* Also enables custom composition through ARMatteGenerator 可以通过ARMatteGenerator启用自定义合成


### Motion Capture

* 实时跟踪人体动作
* 集成于ARKit3和RealityKit
* 借助于机器学习
* 支持A12

ARBodyTrackingConfiguration

Extracting Data from 3D Skeleton

BodyTrackedEntity

1. load character
2. get the location where you would like to put character

ARBodyAnchor {
	ARSkeleton{
		Definition{
			Joints Name
			Parent-Child Relationship
		}
		Joint Transforms
	}
	Transform
}

Extracting Data from 2D Skeleton

ARBody2D{
	Skeleton{
		Definition{
			Parent-Child Relationship
			Joint-Names
		}
		Joint Landmarks
	}
}

获取 ARBody2D 对象

* 通过 didUpdateFrame 回调，获取 frame.detectedBody
* ARBodyAnchor 属性 referenceBody


### Collaborative AR Experiences

ARCollaborationData

### AR Scene

什么是Scene

* Anchor
* Objects
* Behaviors
* Physics World


## Tips

### 1. worldPosition VS position vs simdPosition

* worldPosition：相对于场景世界原点（0，0，0）的位置。
* position：如果父节点是根节点，那么等同于worldPosition，否则是相对于父节点的位置。
* simdPosition：是position的simd类型数据，使用simd相关函数进行大量向量计算性能更好。






34:30
Most of the time your experience require a furface to place content on to it.
So if you enable plane detection on your configuration ,then this overlay will automatically show up.

34:42
Secondly,we have another overlay that provides the user with the ability to understand that they have to move around a little bit more to gather additional features so that tracking can works best.

34:54
And then finally, we have another overlay which helps your user relocalize against certain environments in case of your lost tracking for example, or if app went into the background.

35:40 
you have to set it up as a child of another UI view. Ideally, you set it as a child of the AR view.

35:49
Then, you need to connect this session to the coaching view so that the coaching view knows what events to react to.

35:57
Or you need to connect the sesison provider oultet of the coaching view to the session provider itself if you're using a storyboard for example.

36:08
Optionally, you can set a bunch of delegates if you want to react to certain events that the view is giving you.

36:17
And finally, you can also provide a set of specific coaching goals if you want to disable certain functionalities.
