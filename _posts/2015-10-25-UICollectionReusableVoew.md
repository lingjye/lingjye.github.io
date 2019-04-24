---
layout: post
title: "UICollectionReusableView与UITableViewHeaderFooterView的用法"
subtitle: 'UICollectionReusableView与UITableView的区头'
author: "lingjye"
header-style: text
tags:
  - iOS
---

UICollectionReusableView与UITableViewHeaderFooterView都继承自UIView, 显示效果类似常见的UITableView Group分组区头区尾

### How to use?

#### UICollectionReusableView用法

创建一个继承UICollectionReusableView的MyCollectionReusableView

```
//MyCollectionReusableView.h

#import <UIKit/UIKit.h>
 
@interface MyCollectionReusableView : UICollectionReusableView

@property (nonatomic, copy) NSString *labelText;

@end
```

```
//MyCollectionReusableView.m

#import "MyCollectionReusableView.h"

@interface MyCollectionReusableView()

@property (nonatomic, strong) UILabel *textLabel;

@end

@implementation MyCollectionReusableView

- (instancetype)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        [self configSubviews];
    }
    return self;
}
 
- (void)configSubviews {
	[self addSubview:self.textLabel];
}

- (void) setLabelText:(NSString *)labelText {
    self.label.text = text;
}

- (UILabel *)textLabel {
    if (!_textLabel) {
        _textLabel = [[UILabel alloc] init];
    }
    return _textLabel;
}
 
@end
```

UICollectionReusableView 同 UICollectionViewCell 一样需要提前注册:
```
[self.collectionView registerClass:[MyCollectionReusableView class] forSupplementaryViewOfKind:UICollectionElementKindSectionHeader withReuseIdentifier:@"MyCollectionReusableViewHeaderID"];
[self.collectionView registerClass:[MyCollectionReusableView class] forSupplementaryViewOfKind:UICollectionElementKindSectionFooter withReuseIdentifier:@"MyCollectionReusableViewFooterID"];

```

然后实现UICollectionViewDataSource协议方法:
```
-(UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath{
    if (kind == UICollectionElementKindSectionHeader) {
        MyCollectionReusableView *reusableHeaderView = [collectionView dequeueReusableSupplementaryViewOfKind:kind withReuseIdentifier:@"MyCollectionReusableViewHeaderID" forIndexPath:indexPath];
        reusableHeaderView.labelText = [NSString stringWithFormat:@"%tu", indexPath.section];
        return reusableHeaderView;
    }
    MyCollectionReusableView *reusableFooterView = [collectionView dequeueReusableSupplementaryViewOfKind:kind withReuseIdentifier:@"MyCollectionReusableViewFooterID" forIndexPath:indexPath];
    reusableFooterView.labelText = [NSString stringWithFormat:@"%tu", indexPath.section];
    return reusableFooterView;
}
```

最终效果:
<img src="https://img-blog.csdn.net/20151025140109764?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="50%" height="50%">
<img src="https://img-blog.csdn.net/20151025140141985?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="50%" height="50%">


#### UITableViewHeaderFooterView用法

创建一个继承UITableViewHeaderFooterView的MyTableViewHeaderFooterView

```
//MyTableViewHeaderFooterView.h

#import <UIKit/UIKit.h>
 
@interface MyTableViewHeaderFooterView : UITableViewHeaderFooterView

@property (nonatomic, copy) NSString *labelText;

@end
```

```
//MyTableViewHeaderFooterView.m

#import "MyTableViewHeaderFooterView.h"

@interface MyTableViewHeaderFooterView()

@property (nonatomic, strong) UILabel *textLabel;

@end

@implementation MyTableViewHeaderFooterView

- (instancetype)initWithReuseIdentifier:(NSString *)reuseIdentifier {
    if (self = [super initWithReuseIdentifier:reuseIdentifier]) {
        [self configSubviews];
    }
    return self;
}
 
- (void)configSubviews {
    [self addSubview:self.textLabel];
}

- (void) setLabelText:(NSString *)labelText {
    self.label.text = text;
}

- (UILabel *)textLabel {
    if (!_textLabel) {
        _textLabel = [[UILabel alloc] init];
    }
    return _textLabel;
}
 
@end
```
注意UITableViewHeaderFooterView的初始化方法为:
```
- (instancetype)initWithReuseIdentifier:(NSString *)reuseIdentifier;
```

UITableViewHeaderFooterView 同样也需要提前注册:
```
[self.tableView registerClass:[MyTableViewHeaderFooterView class] forHeaderFooterViewReuseIdentifier:@"MyTableViewHeaderFooterViewID"];
```

然后实现UITableViewDataSource协议方法:
```
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
    MyTableViewHeaderFooterView *headerView = [MyTableViewHeaderFooterView dequeueReusableHeaderFooterViewWithIdentifier:@"MyTableViewHeaderFooterViewID"];
    headerView.labelText = [NSString stringWithFormat:@"%tu", indexPath.section];
    return headerView;
}

- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section {
    MyTableViewHeaderFooterView *footerView = [MyTableViewHeaderFooterView dequeueReusableHeaderFooterViewWithIdentifier:@"MyTableViewHeaderFooterViewID"];
    footerView.labelText = [NSString stringWithFormat:@"%tu", indexPath.section];
    return footerView;
}

```
效果参考UICollectionReusableView


