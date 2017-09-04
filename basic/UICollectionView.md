# UICollectionView


### shouldInvalidateLayoutForBoundsChange(newBounds: CGRect) -> Bool

```
override func shouldInvalidateLayoutForBoundsChange(newBounds: CGRect) -> Bool {
  return true
}
```

返回```true```，在collectionView bounds改变的时候强制重新进行布局运算，然后会调用```layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]?```方法进行计算。在滑动的时候会引起bounds的改变。
