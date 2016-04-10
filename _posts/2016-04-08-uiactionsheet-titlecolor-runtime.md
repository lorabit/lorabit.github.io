---
layout:     	post
title:      	"通过运行时自定义UIActionSheet的字体颜色"
subtitle:   	"Objective-C Runtime Black Magic I"
date:       	2016-04-07 19:00:00
author:     	"LoraBiT"
header-img: 	"img/post/bg-alibaba-bus-stop.jpg"
header-mask: 	0.3
catalog: 		true
tags:
    - iOS
---

> 如果你只是来复制代码的，请直接跳到最后一节。
>
> <font style="float:right;">- LoraBiT</font>

# 「5分钟」的需求
前不久，产品经理发来一个视觉搞，说要做一个左图所示的菜单，我一看，并没有多想，这不就是5分钟搞定的需求吗。

![](/img/post/uiactionsheet-titlecolor/withme-actionsheet.png)

然而5分钟后，我发现按钮颜色居然不一样，UIActionSheet的按钮颜色都如右图所示那样。15分钟后都还没有找到设置按钮颜色的属性，没办法只好无耻地求助「度娘」和「栈溢出」。又经过了15分钟的信息检索后，得到如下信息：

- 苹果没有提供方法供定义红色、加粗之外的按钮样式
- iOS 8.0之前的版本可以通过遍历UIActionSheet的subviews来定制按钮样式
- 之后的版本如果有定制需求，基本都通过第三方的组件来实现

于是，半个多小时过去了还是没能解决这个「5分钟」的需求，虽然我通过这次的检索又知道了几个第三方组件的名字，但是因为原生组件的如下优点，让我决定再试试看能不能用原生组件解决问题。

1. App体积更小
2. 对不同设备的横向支持更好
3. 对未来的系统的纵向支持更好
4. 接口规范，文档丰富
5. 安全性更好
6. UI风格统一
5. ...

不妨，让我们先看看iOS 8.0的解决方法及思路。

# 自(丑)由(陋)时代


在iOS 8以前，常规的做法是在原生UI组件渲染之前再通过遍历视图层级的方式修改组件中的UILabel或者UIButton对象的。例如：我们可以实现`UIActionSheetDelegate`的如下方法。

```
- (void)willPresentActionSheet:(UIActionSheet *)actionSheet
{
    for (UIView *subViwe in actionSheet.subviews) {
        if ([subViwe isKindOfClass:[UIButton class]]) {
            UIButton *button = (UIButton*)subViwe;
            [button setTitleColor:[UIColor colorWithHexString:BLUE_GREEN_COLOR] forState:UIControlStateNormal];
        }
    }
} 
```

遗憾的是iOS 8之后，苹果似乎不太满意许多的原生UI被过度定制，于是便把遍历子视图获取UIButton、UILabel的方法给禁用了，好在苹果没有直接让subviews方法报错，而是十分友好的返回了个空数组，于是上述方法已经是一个只有「不能用」这一个缺点的方法了。

事实上，我们只要能拿到我们所需要修改的UIButton、UILabel实例即可，如果不用遍历子视图的方式，还有什么其他方式获取呢？

# 暴(优)力(雅)时代

熟悉「运行时（Runtime）」的同学应该立刻就能想到，只要混淆UILabel的init方法就能拿到整个App内所有的UILabel实例引用了，剩下的问题就只是在合适的时机来调用这些实例的`setTextColor:`和`setTintColor:`方法了。我能想象到的最暴力的方法应该就是维护一个App内所有UILabel实例的集合，然后开一个定时器不停地去枚举这些实例，找到符合条件（比如文本一致）的实例并调用他们的`setTextColor:`和`setTintColor:`方法。

如果苹果的程序员确实是通过UILabel来构建UIActionSheet的按钮文字，那么我们也有理由相信这位「神秘（不开源）」的程序员一定也是通过`setTextColor:`和`setTintColor:`的方法来把那些文字的颜色改成蓝色、蓝色、蓝色和蓝色的。简单的混淆这些方法后并打印`self.text`我们就可以验证这个假设了 - 果然苹果的程序员也很会重（偷）用（懒）。

剩下的就不用我说了吧，在混淆代码里面加一堆if，通过`self.text`判断就可以改颜色了。


<font style="text-decoration:line-through;">本文结束。</font>


感觉X还没有装够，而且你们也还没有看到代码。

遍历subviews的方法在我看来已经对业务逻辑代码造成了极大地混乱，然而这个运行时的方法里面得把所有的颜色修改都硬编码在`setTextColor:`和`setTintColor:`方法里面更是难以想象。

所幸机智的我，又想到了通过给NSString扩展属性的方式。
让我们给NSString扩展一个tintColor的属性，之后只需要给UIActionSheet提供带tintColor属性的NSString就可以实现定制按钮颜色了。下面的代码展示的是通过该方法修改“取消”按钮颜色的例子。是不是比遍历subviews的方式更加简洁明了？


```
NSString *cancelStr = @"取消";
    cancelStr.tintColor = [UIColor hexColorFloat:@"333333"];
    UIActionSheet *action = [[UIActionSheet alloc]initWithTitle:nil delegate:self cancelButtonTitle:cancelStr destructiveButtonTitle:nil otherButtonTitles:nil, nil];
```

下面是使用上述方法所需要的代码。

别忘了`import <objc/runtime.h>`。

    
```

@interface NSString(ActionSheetAdditions)

@property(nonatomic,strong)UIColor* tintColor;

@end

@implementation NSString(ActionSheetAdditions)

-(UIColor *)tintColor{
  return objc_getAssociatedObject(self, _cmd);
}

-(void)setTintColor:(UIColor *)tintColor{
  objc_setAssociatedObject(self, @selector(tintColor), tintColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end

@interface UILabel(Custom)

@property(strong,nonatomic) UIColor * forcedTintColor;
@end

@implementation UILabel(Custom)

-(UIColor *)forcedTintColor{
  return objc_getAssociatedObject(self, _cmd);
}

-(void)setForcedTintColor:(UIColor *)forcedTintColor{
  objc_setAssociatedObject(self, @selector(forcedTintColor), forcedTintColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

+(void)load{
    
    swizzled_Method([UILabel class], @selector(setTintColor:), @selector(setLabelTintColor:));
    swizzled_Method([UILabel class], @selector(setTextColor:), @selector(setLabelTextColor:));
}

-(void)setLabelTintColor:(UIColor *)tintColor{
    
  if(self.forcedTintColor){
    [self setLabelTintColor:self.forcedTintColor];
    return;
  }
  if(self.text.tintColor){
      
    [self setLabelTintColor:self.text.tintColor];
    self.forcedTintColor = self.text.tintColor;
    return;
  }
  [self setLabelTintColor:tintColor];
}



-(void)setLabelTextColor:(UIColor *)textColor{
    
    if(self.forcedTintColor){
        [self setLabelTextColor:self.forcedTintColor];
        return;
    }
    if(self.text.tintColor){
        
        [self setLabelTextColor:self.text.tintColor];
        self.forcedTintColor = self.text.tintColor;
        return;
    }
    [self setLabelTextColor:textColor];
}



@end
```





