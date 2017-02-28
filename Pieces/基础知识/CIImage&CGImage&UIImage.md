Core Image:CIImage是不能直接通过UIImage().ciImage获取的。官方定义:
![CIImage](/images/6F6D2776-B64C-4D01-A7C6-A531E050C9C3.png)    

UIImage， CGImageRef， CIImage 三者之间可以通过一些联系进行转换   
### 1、 UIImage 转换CGImageRef
UIImage 类当中包含了CGImage的属性，所以很方便地就能转换，方法如下：
```
UIImage *image =[ UIImage imageNamed:@"xx.png"];
CGImageRef imageRef = [image CGImage];
```
### 2 CGImageRef 转换UIImage
UIImage里面包含了一个方法imageWithCGImage，如果知道了CGImage，则这样子也可以创建得到UIIamge类，在上面我们可以看到关系 UIImage 通过属性得到CGImageRef，同样地两者也可以关联起来。  
UIImage—>CGImageRef   
CGImageRef –> UIImage   
```UIImage *uiImage =[UIImage imageWithCGImage:cgImage];```

### 3 CIImage 转换CGImageRef  
CIContext 当中有一个方法createCGImage，可以创建得到CGImageRef，换句话可知道CIImage 可以通过其他方式转换CGImageRef:
```
CIContext *context = [CIContext contextWithOptions:nil];
CIImage *ciImage = [CIImage imageWithContentsOfURL:myURL];
filter = [filterWithName:@"CISepiaTone"];
[filter setValue:ciImage forKey:kCIInputImageKey];
[filter setValue:@0.8f forKey:kCIInputIntensityKey];
CIImage *outputImg = [filter outputImage];
CGImageRef cgImage = [context createCGImage:outputImg fromRect:[outputImg extent]];
```
### 4 UIImage 转换CIImage
```
CIImage *ciImage = [UIImage imageNamed:@"test.png"].CIImage
UIImage *image = [[UIImage alloc] initWithCIImage:ciImage];
```
但是采用这种方式转换，CIImage的值会是nil，
相反 采用CIImage 的initWithCGImage初始化的方式 则有值，很奇怪
```
UIImage *image = [UIImage imageNamed:@"test.png"];
CIImage *ciImage = [[CIImage alloc]initWithCGImage:image.CGImage];
```
由此可见，三者都可以实现转换了，通过直接或者间接把他们联系起来。
UIImage –> CGImageRef –> CIImage    
UIImage <– CGImageRef <– CIImage
