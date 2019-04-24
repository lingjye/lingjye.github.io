---
layout: post
title: "在WebView中添加长按手势,保存图片"
author: "lingjye"
header-style: text
tags:
  - WebView
  - iOS
---

首先要创建UIWebview, 然后直接添加长按手势如下:

```
//创建长按手势对象, 并添加到webview上
UILongPressGestureRecognizer *longtapGesture = [[UILongPressGestureRecognizer alloc]initWithTarget:self action:@selector(longtap:)];
[webView addGestureRecognizer:longtapGesture];
```

长按事件:

```
//长按下载图片
-(void)longtap:(UILongPressGestureRecognizer * )longtapGes {
    if (longtapGes.state == UIGestureRecognizerStateBegan) {
        //获取手势位置
        CGPoint pt = [longtapGes locationInView:appFrame.webView];
        pt = [appFrame.webView convertPoint:pt fromView:nil];
        CGPoint offset  = [appFrame.webView.scrollView contentOffset];
        CGSize viewSize = [appFrame.webView frame].size;
        CGSize windowSize = [appFrame.webView frame].size;
        CGFloat f = windowSize.width / viewSize.width;
        //如果不加判断,在webview滑动时将捕获不到image
        if ([[[UIDevice currentDevice] systemVersion] doubleValue] >= 5.0) {
            pt.x = pt.x * f;
            pt.y = pt.y * f;
        } else {
            // On iOS 4 and previous, document.elementFromPoint is not taking
            // offset into account, we have to handle it
            pt.x = pt.x * f + offset.x;
            pt.y = pt.y * f + offset.y;
        }
        [self openContextualMenuAt:pt];
    }
}
```

根据获取到的手势位置,通过注入一段js方法最终获取到该位置上的图片:

```
//根据长按位置获取当前图片
- (void)openContextualMenuAt:(CGPoint)pt {
	//加载本地js, 也可以直接加载js字符串
    NSString *path = [[NSBundle mainBundle] pathForResource:@"longPressJS" ofType:@"txt"];
    NSString *jsCode = [NSString stringWithContentsOfFile:path encoding:NSUTF8StringEncoding error:nil];
    [appFrame.webView stringByEvaluatingJavaScriptFromString: jsCode];
	//获取图片标签
    NSString *tags = [appFrame.webView stringByEvaluatingJavaScriptFromString:
    [NSString stringWithFormat:@"MyAppGetHTMLElementsAtPoint(%f,%f);",pt.x,pt.y]];
    UIActionSheet *sheet = [[UIActionSheet alloc] initWithTitle:@"保存图片到相册"
    delegate:self cancelButtonTitle:@"取消"
    destructiveButtonTitle:nil otherButtonTitles:nil];
	//判断是否获取到图片
    if ([tags rangeOfString:@",IMG,"].location != NSNotFound) {
        NSString *str = [NSString stringWithFormat:@"document.elementFromPoint(%f, %f).src", pt.x, pt.y];
        NSString *imgStr= [appFrame.webView stringByEvaluatingJavaScriptFromString: str];
        _imgUrl = imgStr;
        NSLog(@"imgUrl : %@",_imgUrl);
        [sheet addButtonWithTitle:@"保存图片"];
        [sheet showInView:self.view];
    }
}

#pragma mark UIActionSheetDelegate
- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex {
    NSLog(@"%li",buttonIndex);
    if (buttonIndex == 1) {
        NSLog(@"image url=%@", _imgUrl);
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:_imgUrl]];
        UIImage *image = [UIImage imageWithData:data];
        //UIImageWriteToSavedPhotosAlbum(image, nil, nil,nil);
        UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
    }
}

//保存图片
- (void)image:(UIImage *)image didFinishSavingWithError:(NSError*)error contextInfo:(void *)contextInfo {
    if (error) {
        NSLog(@"Error");
        [MBProgressHUD showError:@"保存失败" toView:self.view];
    }else {
        NSLog(@"OK");
        [MBProgressHUD showSuccess:@"保存成功" toView:self.view];
    }
}

```

需要用到longPressJS.txt 里面是一段js代码, 新建一个空白文档将下面代码复制到文档中保存,直接拖入工程即可:

```
function MyAppGetHTMLElementsAtPoint(x,y) {
    var tags = ",";
    var e = document.elementFromPoint(x,y);
    while (e) {
        if (e.tagName) {
            tags += e.tagName + ',';
        }
        e = e.parentNode;
        }
    return tags;
}

function MyAppGetLinkSRCAtPoint(x,y) {
    var tags = "";
    var e = document.elementFromPoint(x,y);
    while (e) {
        if (e.src) {
            tags += e.src;
            break;
        }
        e = e.parentNode;
    }
    return tags;
}

function MyAppGetLinkHREFAtPoint(x,y) {
    var tags = "";
    var e = document.elementFromPoint(x,y);
    while (e) {
        if (e.href) {
            tags += e.href;
            break;
        }
        e = e.parentNode;
    }
    return tags;
}
```
