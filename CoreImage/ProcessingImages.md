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

#重点#：CIFilter对象是可变的，所以是线程不安全的。所以每个线程必须要创建自己的CIFilter对象。然而滤镜的输入和输出的CIImage对象是不可变的，可以安全的在不同线程中访问。

# 复杂的滤镜链效果

每一个Core Image滤镜可以输出一个CIImage对象，所以你可以把它作为其他滤镜的输入参数，可以达到滤镜链的效果。因为只是标识了处理的步骤，等到最终要渲染显示或者输出的时候，才会进行处理。

每一个CIImage对象在滤镜链中不是一个完全渲染的图像，而是一个渲染的方法。Core Image不需要单独的处理每一个滤镜，浪费渲染内存和时间。Core Image把滤镜结合成单个操作，甚至重新组织滤镜以不同的顺序更有效的产生相同的效果

# 使用更多选项的特殊过滤器类型

* 合成滤镜根据预设公式组合两个图像。

  > CISourceInCompositing:合并两个图片，使得只有两个输入图像中不透明的区域在输出图像中可见。

  > CIMultiplyBlendMode：将两个图像的像素颜色相乘，产生变暗的输出图像。

* 生成器滤镜不需要输入图片，它会通过其他参数从头创建图片。
>  CIQRCodeGenerator和CICode128BarcodeGenerator生成输入数据的条形码图像。
> CIConstantColorGenerator、CICheckerboardGenerator和CILinearGradient生成指定颜色的图像。可以将它和其他滤镜结合生成有趣的效果。 CIRadialGradient可以创建一个和CIMaskedVariableBlur一起使用的滤镜。
>  CILenticularHaloGenerator和CISunbeamsGenerator创建独立的视觉效果。把它们和合成滤镜结合可以生成特殊的效果。

* 还原滤镜是在输入图像上操作，而不是去生成一个新的输出图像。所有的滤镜都需要一个CIImage的输出，所以这个输出的信息仍然是一个图片。然而一般不会去显示这个图片，可以读取图片的一个像素或者一行的颜色信息，或者作为其他滤镜的输入参数。
> CIAreaMaximum滤镜输出图片指定区域的最亮的颜色值。
> CIAreaHistogram滤镜输出图像的指定区域中每个像素的饱和度信息。

* 过渡滤镜需要两个输入图像，并根据独立变量改变它们之间的输出。通常这个变量是时间，所以你可以使用过渡滤镜创建一个动画，从一个图片开始到另一个图片结束。
  > CIDissolveTransition淡入淡出。

  > CICopyMachineTransition模拟扫描，从一个图像上滑动明亮的光线，到显示出另一个图像。


  # AVFoundation处理图片

  ```
  let filter = CIFilter(name: "CIGaussianBlur")!

  let composition = AVVideoComposition(asset: asset, applyingCIFiltersWithHandler: { request in


      // Clamp to avoid blurring transparent pixels at the image edges

      let source = request.sourceImage.clampingToExtent()

      filter.setValue(source, forKey: kCIInputImageKey)


      // Vary filter parameters based on video timing

      let seconds = CMTimeGetSeconds(request.compositionTime)

      filter.setValue(seconds * 10.0, forKey: kCIInputRadiusKey)


      // Crop the blurred output to the bounds of the original image

      let output = filter.outputImage!.cropping(to: request.sourceImage.extent)


      // Provide the filter output to the composition

      request.finish(with: output, context: nil)

  })

  ```

  使用AVVideoComposition类合成音频和视频，videoCompositionWithAsset:applyingCIFiltersWithHandler:方法在创建时可以给视频帧添加滤镜。

  Tips：默认情况下模糊滤镜会通过将图像像素与图像周围（在滤镜处理的范围内）的透明像素模糊化来软化图像的边缘。一般在给视频添加滤镜的时候，是不期望出现这个效果的。为了避免这个效果可以使用imageByClampingToExtent方法来延伸图片，这个方法会不断的复制边缘像素来表示图片是无限大的。因为它是创建的无限大的图片，所以需要在模糊后进行裁剪。

# 用Core Image上下文构建自己的工作流程

Core Image context表示执行滤镜和生成图像所需的CPU或GPU计算技术，资源和设置。

#重点：#Core Image context是管理大量资源和状态的重量级对象。重复创建和销毁上下文会有性能的成本，所以你计划执行不同的图片处理操作，提早创建一个上下文并存储备用。

## 自动渲染上下文

如果使用init 或者 initWithOptions:初始化上下文，Core Image会基于当前设备和指定的options自动在内部管理资源，并选择适当的或者最好的CPU或者GPU渲染技术。这个方法非常适合将处理后的图片渲染为文件。

##注意：## 没有明确指明渲染结果的上下文不能使用```drawImage:inRect:fromRect:```方法，因为该方法根据渲染结果而改变。使用以render或者create开头的CIContext方法来指定一个显示的结果。

#Tips:#当你尝试使用这个方法去实时渲染一个Core image结果，不断修改滤镜的参数，然后生成动画的过渡效果，或者是处理视频，或者是每秒需要渲染多次内容的时候，要小心。尽管通过这个方法创建的CIContext对象会自动通过GPU渲染，显示这个渲染的结果可能会涉及到很多从CPU到GPU内存的的复制操作。

## Metal的实时渲染

Metal框架提供对GPU的低开销访问，实现图形渲染和并行计算工作流程的高性能。利用Metal获取实时性能来实现滤镜输出或滤镜动画输入（如实时视频）。
MetalKit调用```drawInMTKView:```方法来绘制view显示的每一刻（默认是一秒60次）。

## OpenGL或者OpenGL ES的实时渲染

Core Image可以高性能的使用基于GPU的OpenGL ES渲染。使用```imageWithTexture:size:flipped:colorSpace:```方法从OpenGLES纹理中初始化CIImage对象，使用GPU内存中的图像数据可以通过删除冗余复制操作来提高性能。要在OpenGLES渲染Core Image的输出，需要将GL当前上下文设置为目标帧缓冲区，然后调用```drawImage:inRect:fromRect:```方法。

## 基于CPU的渲染 Quartz2D

如果不需要渲染的实时性可以使用CoreGraphics进行绘制。
