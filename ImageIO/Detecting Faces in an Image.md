# 在图片中检测人脸

Core Image可以在图片中检测并找到人脸。是检测，不是识别。人脸检测是识别包含人脸特征的矩形，而脸部识别是识别特定的人脸。检测到人脸之后会提供脸部特征的信息，像眼嘴的位置等。Core Image也可以跟踪视频中的人脸移动。

获取图片中人脸的位置后可以进行一些有趣的操作，例如调整图片中脸部的质量（色调平衡，红眼等）。或者给脸部打马赛克，给脸部添加一些装饰效果。

## 脸部识别

使用CIDetector来识别图片中的脸部信息。
```
CIContext *context = [CIContext context];

NSDictionary *opts = @{ CIDetectorAccuracy : CIDetectorAccuracyHigh };

CIDetector *detector = [CIDetector detectorOfType:CIDetectorTypeFace
                                          context:context
                                          options:opts];

opts = @{ CIDetectorImageOrientation :
          [[myImage properties] valueForKey:kCGImagePropertyOrientation] };

NSArray *features = [detector featuresInImage:myImage options:opts];

```

## 获取脸部和脸部特征的边界

脸部特征包括：左右眼，嘴，Core Image用于跟踪视频片段中的脸部的ID和帧数。

获取脸部特征数组之后，通过遍历得到每一个特征的信息。
