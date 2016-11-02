# Pieces
## 在计算View的frame时，我们通常设置的是自适应，“sizeToFit”,或者是一些通过计算出来的frame，但是实际效果总是跟我们设置的值有些偏差，在经过测试分析后的发现系统在最后确定frame时，会进行舍入，并总是保留一位小数：当“.”后的值小于0.5时，会入成0.5；当等于0.5，会保持不变；当大于0.5，会入成1.0。
当知道这个后，自适应的时候可以做到精确计算每个控件的frame，不再有偏差。
# 转换方法
CGFloat frameFormatterNumber(CGFloat number){
    NSNumberFormatter * formatter = [[NSNumberFormatter alloc] init];
    //允许1位小数
    formatter.maximumFractionDigits = 1;
    //增量设置为0.5
    formatter.roundingIncrement = @0.5;
    //向上取整
    formatter.roundingMode = kCFNumberFormatterRoundUp;
    return [[formatter stringFromNumber:[NSNumber numberWithFloat:number]] floatValue];
}
当我们根据label中的文字,通过- (CGRect)boundingRectWithSize:(CGSize)size options:(NSStringDrawingOptions)options attributes:(nullable NSDictionary<NSString *, id> *)attributes context:(nullable NSStringDrawingContext *)context方法计算完之后，再转换，获得准确的label的frame。
