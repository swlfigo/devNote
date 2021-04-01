## 去掉输入键盘上的自动联想部分，限制长度内容，以及禁止输入emoji表情等问题

当输入的信息只允许是数字或者字母的时候，自动联想的文本可以不点击进去，而这些文本有的时候不见得是我们希望出现的。所以，解决办法可以采用直接去掉自动联想功能。代码为：

```objective-c
textField.autocorrectionType = UITextAutocorrectionTypeNo;

textView.autocorrectionType = UITextAutocorrectionTypeNo;
```



2.1 限定某些特定的输入文本时，可以采用的方法是：

```objective-c
define kAlphaNum @"X0123456789"

NSCharacterSet *cs;

cs = [[NSCharacterSetcharacterSetWithCharactersInString:kAlphaNum]invertedSet];

NSString *filtered = [[stringcomponentsSeparatedByCharactersInSet:cs]componentsJoinedByString:@""];//按cs分离出数组,数组按@""分离出字符串

BOOL canChange = [stringisEqualToString:filtered];

return canChange;
```



2.2 由于在限制输入字符的时候有许多的需求，比如输入汉字，当在拼写的时候在邻近限制长度的时候，拼音一旦超过长度，就无法精确地将想要的汉字输入到最后限制位，针对这样的问题解决办法如下：注：textView同样同样适用与这个方法，不过并不是给textView添加触发事件而是添加监听，即NSUserDefault

1.给textField添加触发事件



```objective-c
[_hNewMsgTextField addTarget:self action:@selector(textChanged:)forControlEvents:UIControlEventEditingChanged];
```



2.完成方法



```objective-c
-(void)textChanged:(UITextField *)textField{
    
    NSString *lang = [[UITextInputMode currentInputMode]primaryLanguage];//键盘输入模式
    
    if ([lang isEqualToString:@"zh-Hans"]) {// 简体中文输入，包括简体拼音，健体五笔，简体手写
        
        UITextRange *selectedRange = [textField markedTextRange];
        
        //获取高亮部分
        
        UITextPosition *position = [textField positionFromPosition:selectedRange.start offset:0];
        
        //没有高亮选择的字，则对已输入的文字进行字数统计和限制
        
        if (!position) {
            
            if (textField.text.length >11) {
                //不能简单用 substringToIndex 来裁剪，如果最后一个字是 Emoji，裁剪可能会乱码
                //textField.text = [textField.text substringToIndex:11];
                NSRange rangeIndex = [textView.text rangeOfComposedCharacterSequenceAtIndex:11];
            		textView.text = [textView.text substringToIndex:rangeIndex.location];
            }
            
        }
        
        //有高亮选择的字符串，则暂不对文字进行统计和限制
        
        else{
            
        }
        
    }
    
    // 中文输入法以外的直接对其统计限制即可，不考虑其他语种情况
    
    else{
        
        if (textField.text.length >11) {
            //同上
            //textField.text = [textField.text substringToIndex:11];
            NSRange rangeIndex = [textView.text rangeOfComposedCharacterSequenceAtIndex:11];
            textView.text = [textView.text substringToIndex:rangeIndex.location];
        }
        
    }
    
}

```


​                        


3.禁止输入苹果键盘自带的emoji表情




```objective-c
if ([[[UITextInputMode currentInputMode]primaryLanguage] isEqualToString:@"emoji"]) {
        return NO;        
}
```







另外 ，**OC** 、**Swift** 判断字符(字节)长度不一样

![](http://sylarimage.oss-cn-shenzhen.aliyuncs.com/2021-04-01-063549.png)

