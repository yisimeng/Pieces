# 处理图片
处理图片意味着使用滤镜—图像滤镜是一个软件，其逐像素地检查输入图像，并且算法地应用一些效果以创建输出图像。在core image中处理图片依赖于描述他们输入输出的CIFilter和CIImage两个类。使用滤镜并显示或者输出结果，可以利用Core Image和其他系统框架的集成，或者使用CIContext类创建你自己的渲染流程，本章介绍了使用这些类应用滤镜和渲染结果的关键概念。
## Overview
这里有很多使用Core Image处理图片的方法
```
import CoreImage

let context = CIContext()                          // 1

let filter = CIFilter(name: "CISepiaTone")!                         // 2
filter.setValue(0.8, forKey: kCIInputIntensityKey)
let image = CIImage(contentsOfURL: myURL)                           // 3
filter.setValue(image, forKey: kCIInputImageKey)
let result = filter.outputImage!                                    // 4
let cgImage = context.createCGImage(result, from: result.extent)    // 5
```

这些代码做了啥：
1、创建上下文对象，你并不总是需要自己的CoreImage上下文—通常您可以与管理渲染的其他系统框架集成。创建你自己的上下文，创建自己的上下文可以更准确地控制渲染过程和渲染中涉及的资源。 上下文是重量级的对象，所以如果你创建一个对象，那么尽可能早地执行，并在每次需要处理图像时重用它。
2、实例化CIFilter，给参数赋值。
3、创建一个要处理的CIImage对象，把它作为filter的inputImage参数。
4、创建CIImage的输出对象，此时滤镜尚未执行—图片是指定如何使用指定的滤镜，参数和输入创建图像。 Core Image仅在请求渲染时才执行此配方。
5、渲染输出图像到一个你可以显示或者保存的CGImage。

# 滤镜的输入输出图片
滤镜处理并生成Core image图像。CIImage是表示图片的不可变对象。CIImage不是图片的位图数据，是表示生成图片的方法。
可能是从文件中加载图像的方法，或者是从滤镜输出图像的方法。Coreimage只在当你请求渲染显示或者输出图片的时候使用这个方法。
要应用滤镜，请创建一个或多个要由滤镜处理的图像的CIImage对象，并把它设置为滤镜的输入参数，几乎可以从任何图像数据源创建Coreimage图像对象。
* 从URL加载的图像文件，或者包含图像文件数据的NSData对象。
* Quartz2D,UIKit或者AppKit的图像 (CGImageRef, UIImage,  NSBitmapImageRep )。
* Metal,OpenGL,或者OpenGL ES纹理。
* CoreVideo图片或者像素缓存（CVImageBufferRef or CVPixelBufferRef）。
* 在进程之间共享图像数据的IOSurfaceRef对象
* 内存中的图像位图数据（指向这样的数据的指针，或者是根据需要提供数据的CIImageProvider对象）。

CIImage对象是描述该如何生成图片（而不是包含图像数据），也可以是滤镜的输出。当你访问CIFilter的outputImage属性时，CoreIamge仅仅是标识和存储执行滤镜的步骤。这一步只有当你请求渲染显示或者输出的时候才会执行。你可以使用CIContextrender或绘图方法显式请求呈现，也可以隐式地使用使用Core Image的许多系统框架显示图像。

延迟处理的渲染时间使Core image更加快速高效。渲染的时候，Core image可以看到是否有多个滤镜作用于一个图片。这样的话，它可以自动的整合多个处理方法并且管理他们消除冗余的操作，每个像素都只会被处理一次。
