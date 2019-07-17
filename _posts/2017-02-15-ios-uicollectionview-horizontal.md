---
layout: post
title: UICollectionView：支持横向滑动分页
date: 2017-02-15 18:32:24.000000000 +09:00
tag: iOS
---

最近做直播APP，坑爹的产品想要实现一个礼物列表，列表中有多种类型的礼物，每个类型下面又有n个礼物，礼物分页展示，可以左右滑动，当滑到分类最后一页自动切换到下个分类，且礼物列表支持横竖屏切换。
> 备注：下文中“每一类”、“组”和“类别”概念相同，都指一类礼物

##### 横竖屏切换：
**竖屏时**，每个分类要显示当前多少页，以及定位当前第几页，当`pageCount`<=1时，不显示页码。
**横屏时**，不显示页码，左右滑动可以切换类别，同一类别可以上下滑动。

工程地址：https://github.com/ma762614600/MAHorizontalCollectionView

先看一下效果图gif：
![demo-show.gif](/images/posts/ios-uiicollectionview/ios-uiicollectionview-demo-show.gif)


##### 实现思路：
第一眼看到这个需求，就确定通过`UICollectionView`实现，但是`UICollectionView`并不支持横向换行分页，后来通过网上找了个第三方类进行修改，支持了横向换行分页，但是滑到分类最后一页不能自动切换到下个分类，分类之间只能通过点击tab切换，由于向产品各种保证二期修改 o(╯□╰)o，第一版就这样勉强上线了。

二期：经过仔细分析需求和对以后产品扩展，以及一些零碎的用户体验等综合考虑，决定采用下面实现方法，也即是目前实现方法。

##### 实现方法：
1.横屏和竖屏各写一套布局，`MAItemListPortraitView`是竖屏布局，`MAItemListLandscapeView`是横屏布局，将两个布局加到父视图`MAItemListView`上，默认是竖屏布局。当用户旋转手机屏幕，触发方法，动态切换布局：横屏时隐藏竖屏布局，显示横屏布局；竖屏时隐藏横屏布局，显示竖屏布局。

2.竖屏时`MAItemListPortraitView`：

声明两个参数：
`kRowCount`：每一页显示多少行
`kItemCountPerRow`：每一行显示多少个

为了既支持换行横向分页滑动，又支持不同类别间能互相滑动，在竖屏时使用一个UICollectionView，如果某个类别中最后一页的礼物数小于`kRowCount`与`kItemCountPerRow`的乘积，缺少多少，就补上相应数目的假数据，凑成整页，整合完每个类别的数据，将所有类别数据加到同一个数组中，供`UICollectionView`使用。

3.横屏时`MAItemListLandscapeView`：
为了既能左右滑动切换不同类别礼物，又能在同一类别中上下滑动查看该类别全部礼物，最底层是一个`UIScrollView`，设置只能横向滑动，然后为每个类别初始化一个`UICollectionView`，将`UICollectionView`加到`UIScrollView`上，`UICollectionView`设置只能竖向滑动。


##### 具体实现 及 核心代码

UIViewController接受sizeClass变化回调，在其中触发`MAItemListView `的布局改变：
```
- (void)willTransitionToTraitCollection:(UITraitCollection *)newCollection withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator
{
    [super willTransitionToTraitCollection:newCollection
                 withTransitionCoordinator:coordinator];
    [coordinator animateAlongsideTransition:^(id <UIViewControllerTransitionCoordinatorContext> context)
     {
         if (newCollection.verticalSizeClass == UIUserInterfaceSizeClassCompact) {
             //紧凑的高 横屏
             [_itemListView setItemListLayoutType:ItemListLayoutTypeLandscape];
         } else {
             //正常的高
             [_itemListView setItemListLayoutType:ItemListLayoutTypePortrait];
         }
         [self.view setNeedsLayout];
     } completion:nil];
}

```

1.父视图`MAItemListView`的头文件
```
#import <UIKit/UIKit.h>
#import "MAApiItemListModel.h"

typedef NS_ENUM(NSInteger, ItemListViewLayoutType) {
    ItemListLayoutTypePortrait = 0,  //竖屏
    ItemListLayoutTypeLandscape  //横屏
};

//礼物列表
@interface MAItemListView : UIView
//切换横竖屏
@property (nonatomic, assign) ItemListViewLayoutType itemListLayoutType;
//数据源
@property (nonatomic, strong) NSArray *listsData;
@end
```
> 备注：重写`itemListLayoutType `的setter方法，以切换横竖屏布局



2.竖屏实现：
① 在拿到数据源后，首先进行计算数据源信息：
* `groupPageIndexs`：每一类(组)的索引
* `groupPageCounts`：每一类(组)的页数
* `groupTotalPageCount`：总页数

② 整合数据：
循环遍历每一组数据，将每一组最后一页补齐，用于补齐的假数据打上`isMock `标，用于区分真伪数据，并将整合后的数据加到数组`totalItemList`中。
```
- (void)reDefineData
{
    NSMutableArray *tempArray = [NSMutableArray array];
    for (MAApiItemListModel *listModel in _listsData) {
        
        // 理论上每页展示的item数目
        NSInteger itemCount = kItemCountPerRow * kRowCount;
        // 余数（用于确定最后一页展示的item个数）
        NSInteger remainder = listModel.itemList.count % itemCount;
        
        NSMutableArray *arr = [NSMutableArray arrayWithArray:listModel.itemList];
        
        if (remainder > 0 && remainder < itemCount) {
            //如果不够一页，就加人假数据，凑成一页
            for (NSInteger i = 0; i < itemCount - remainder; i++) {
                MAApiItemItemModel *itemModel = [[MAApiItemItemModel alloc] init];
                itemModel.isMock = YES;//标记这条是假数据
                [arr addObject:itemModel];
            }
        }
        [tempArray addObjectsFromArray:arr];
    }
    self.totalItemList = tempArray;
}

```

3.横屏实现：
在拿到数据源后，根据`listsData.count`初始化对应数目的`UICollectionView`，并设置不同的tag，注册不同的重用标识`ReuseIdentifier`。
```
- (void)initCollectionViews
{
    _bgScrollView.contentSize = CGSizeMake(LandscapeItemListViewWidth * _listsData.count, self.height_t);
    

    CGFloat x_offset = 0;
    for (NSInteger i = 0; i < _listsData.count; i++) {
        
        UICollectionViewFlowLayout *flowLayout = [[UICollectionViewFlowLayout alloc] init];
        flowLayout.itemSize = CGSizeMake(75, 75);
        flowLayout.minimumInteritemSpacing = 25/4;
        
        CGRect frame = CGRectMake(x_offset, 15, LandscapeItemListViewWidth, _bgScrollView.height_t - 15);
        UICollectionView *landscapeCollectionView = [[UICollectionView alloc] initWithFrame:frame collectionViewLayout:flowLayout];
        landscapeCollectionView.tag = 100+i;
        landscapeCollectionView.dataSource = self;
        landscapeCollectionView.delegate = self;
        landscapeCollectionView.alwaysBounceHorizontal = NO;
        landscapeCollectionView.alwaysBounceVertical = YES;
        landscapeCollectionView.backgroundColor = [UIColor colorWithWhite:0 alpha:0];
        landscapeCollectionView.showsHorizontalScrollIndicator = NO;
        landscapeCollectionView.showsVerticalScrollIndicator = NO;
        [_bgScrollView addSubview:landscapeCollectionView];
        
        NSString *identifier = [NSString stringWithFormat:@"ItemLandscapeCollectionCellIdentifier_%ld",landscapeCollectionView.tag];
        [landscapeCollectionView registerClass:[MAItemCell class] forCellWithReuseIdentifier:identifier];
        
        x_offset += LandscapeItemListViewWidth;
        [landscapeCollectionView reloadData];
    }
    [self layoutIfNeeded];
}
```
4.滑动ScrollView，实现分页效果：
在`scrollViewDidEndDecelerating`方法中计算UIScrollView或UICollectionView的偏移量，来计算当前是第几组、该组有多少页以及当前是第几页。

##### 注意事项
1.使用```- (void)scrollRectToVisible:(CGRect)rect animated:(BOOL)animated;```方法设置滚动时，一定要设置UIScrollView的`contentSize`。但在我的开发过程中，这个方法有时会失灵(原因不详)，推荐使用``` - (void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;```设置滚动。

2.在Autolayout与frame混用时，在设置完约束后要紧接着调用```[self layoutIfNeeded]```进行重绘，否则会出现UIView的frame为(0,0,0,0)的情况。

##### 备注
1.demo中的```- (void)locatedItem:(MAApiItemItemModel *)itemModel positionBlock:(void(^)(CGRect frame,CGPoint point))block```方法是为了计算竖屏(横屏)选中的cell在横屏(竖屏)中的位置。

2.不经常写文章，在这班门弄斧了，水平有限，如若发现有错误或者有更好的实现方法，欢迎留言沟通。