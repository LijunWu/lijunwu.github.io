---
layout: post
title: "UITextView 限制字符输入"
date: 2015-09-16 20:48:54 +0800
comments: true
categories: iOS

---

最近项目里有个小需求，就是在文本输入时要限制输入的字数。文本输入控件用的是`UITextView`和`UITextField`，通常的做法是，在内容改变时，在这两个类相应的回调方法中检测字数，如果大于某一个值则当前输入的内容不会被显示到控件中。以`UITextView`为例，我们基本只需要在下面两个回调方法中检测即可：

{% codeblock lang:objc %}
// 第一个方法：

- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text
{
    if (textView.text.length == WordLimitOfTitle && ![text isEqualToString:@""]) {
        // 标题字数已达最大，仍然继续输入
        return NO;
    } else {
        // copy and paste 的情况
        NSUInteger totalLength = textView.text.length;
        totalLength += text.length;
        if (totalLength > WordLimitOfTitle) {
            text = [text substringWithRange:NSMakeRange(0, WordLimitOfTitle - textView.text.length)];
            return YES;
        } else {
            return YES;
        }
    }
}
{% endcodeblock %}

当用户在键盘上键入一个新的字符或者删除一个字符都会调用这个方法，另外当复制粘贴内容到`textView`时这个方法也会被调用。删除一个字符时，`range`的长度为1，`text`内容为空。

{% codeblock lang:objc %}
// 第二个方法：

- (void)textViewDidChange:(UITextView *)textView
{
    if (textView.text.length > WordLimitOfTitle) {
        NSString *title = textView.text;
        title = [title substringWithRange:NSMakeRange(0, WordLimitOfTitle)];
        textView.text = title;
    }
}
{% endcodeblock %}

这个方法在文本框内容发生改变时被调用，注意，这个方法只会响应由用户引起的内容改变，不会响应代码发起的内容改变。其实只要在第一个方法中截获内容变化再截断超出部分，基本就能达到我们的预期。

### 中文输入法

下面我们考虑中文输入的问题。（这里我把`return`健设置为`完成`，但是貌似只是文字变成了`完成`而已，实际上不管你设为什么，你按下这个健后拿到的字符都是一样的`\n`）

这里先说说`UITextInput`里的`markedTextRange`属性，就是被标记的文本,它的类型是`UITextRange`。

`UITextInput`是一个`protocol`，`UITextView`、`UITextField`等文本输入控件都采纳了这个协议。官方文档对`markedTextRange`的解释如下：
	
> If text can be selected, it can be marked. Marked text represents provisionally inserted text that has yet to be confirmed by the user.  It requires unique visual treatment in its display.  If there is any marked text, the selection, whether a caret or an extended range, always resides witihin.
> Setting marked text either replaces the existing marked text or, if none is present,inserts it from the current selection. 
	
要注意的是：

标记的文本表示键入的文本只是临时性的，还没有被用户确定。外观上它需要与普通文本显示不一样。比如下面这样
<div>
<img src = "https://farm6.staticflickr.com/5707/20845977654_d26e389f3e_z.jpg"  width = 300px>
</div>
除了上面两个回调方法，我们再增加一个回调方法：

	- (void)textViewDidChangeSelection:(UITextView *)textView;
在这三个方法中，我们分别打印`textView.text`和`textView.markedTextRange`值。

现在我们切换到中文输入法：

{% img https://farm1.staticflickr.com/652/20845977174_ec03afca87_z.jpg  300 %}

在上面的输入框里我继续按下 <img style = "border:none" src = "https://farm1.staticflickr.com/694/21280634390_7f46dd49b3_z.jpg"" width = 50 >,这时输入框的内容变成了

<img src = "https://farm1.staticflickr.com/686/21468673185_679ed4fcc0_z.jpg" width = 300px>

并且在`tian`与`a`之间多出了一个空格。

我们先来看看这几个方法执行顺序：

- 首先触发的是 <code>- (BOOL)textView:(UITextView \*)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text;</code> 这个方法。打印结果如下：

<img src = "https://farm1.staticflickr.com/654/21468672825_45945d13d6_z.jpg" width = 468px>

此时`markedTextRange`表示从第三个字符开始，长度为9所表示的区域（包括中间的空格）。

如果方法返回YES，表示<code>text</code>所表示的文本将会替代指定位置，即<code>range</code>所表示的区域的文本。这里<code>range</code>的<code>length=0</code>，表示这个<code>range</code>是光标插入的位置。这里打印出<code>text</code>的值，却是 <img style = "border:none"  src =" https://farm1.staticflickr.com/675/21477271371_0e05ce20f8_t.jpg" width = 12 >。猜测这里应该会在底层有一套处理逻辑，包括字符转换、联想等。因为我们看到的并不是这个奇怪的字符啊，而是在输入框看到了<code>a</code>。

- 然后被触发的是 <code>- (void)textViewDidChangeSelection:(UITextView *)textView;</code> 这个方法.

<img src = "https://farm1.staticflickr.com/746/21442460826_d74163b3f5_o.png" width = 486>

这时候<code>a</code>已经被追加到文本框中了，<code>textView.text</code>以及<code>textView.markedTextRange</code>都已经发生了变化。这个方法会被调用，主要也是因为<code>textView.markedTextRange</code>发生了变化吧。

注意，这是<code>a</code>跟前面的字符之间还没有出现空格，也就是说 这里只是简单的把一个字符放进了<code>markedTextRange</code>中。

- 紧接着，<code>- (void)textViewDidChange:(UITextView *)textView;</code>
这个方法被调用

<img src = "https://farm1.staticflickr.com/746/21442460826_d74163b3f5_o.png" width = 486>

这个方法里打印结果与前一个方法一样，是因为<code>textView.text</code>改变了所以被调用。

以为到这里结束了？然而，并！没！有！

在上面那个方法执行完了之后，底层应该又去默默地做了一些事情，比如，它觉得"tiana"这一坨好像并不
能组成一个中文字或词组，所以应该把他们分开？结果，它真的这么做了。

这时候，<code>- (void)textViewDidChangeSelection:(UITextView *)textView;</code>这个方法被调用了。

来看看发生了什么？

<img src = "https://farm1.staticflickr.com/716/20845975984_2418dd4a05_o.png" width = 476>

一个空格出现了！

<code>textView.text</code>以及<code>textView.markedTextRange</code>都发生了变化，所以不出意外，接下来还会调用<code>- (void)textViewDidChange:(UITextView *)textView</code>,这里不再写了。

然后我们再看点别的。

假设我现在已经输入了这一段

<img src = "https://farm6.staticflickr.com/5760/21280849488_7dbbfdcd13_o.png" width = 407>

当选择<code>我是</code>后，这几个方法又是怎么调用的呢？

这次因为没有键入新的内容，所以<code>- (BOOL)textView:(UITextView \*)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text;</code> 不会被调用。

先被调用的是<code>- (void)textViewDidChangeSelection:(UITextView *)textView；</code>方法：

<img src = "https://farm6.staticflickr.com/5830/21468671185_c92fde6005_o.png" width = 472>

这个方法中，<code>markedTextRange</code>已经被更改了，<code>textView.text = “我是ddmgwgmmgddwgpgjamg”</code>, 所以这个方法之后会调用<code>- (void)textViewDidChange:(UITextView *)textView；</code>这个方法，打印内容与这里一样，不赘述。

等等，为什么<code>textView.text</code>的值是<code>我是ddmgwgmmgddwgpgjamg</code>而不是<code>我是fengzhongdeyipilang"</code>?! 而且空格也不见了?! 咕~~(╯﹏╰)b，这里输入法已经将之前所有预测的字符全部改成了按键里的第一个字母了，并去掉了空格，也就是这里去掉了所有它自己推测的部分。因为在用户确认了部分文字后，它要对剩下的字符重新预测，因此，先要让所有输入都回到初始状态，然后对剩下的未确定的部分重新推算并在合适的位置加入空格。这样又会导致这两个方法再被调用一遍。

<code>- (void)textViewDidChangeSelection:(UITextView *)textView</code>这个方法第二次被调用后，打印内容：

<img src = "https://farm1.staticflickr.com/770/21477270021_6dd106ba62_o.png" width = 474>

看，空格又回来了！但是，慢着，哪里不对？！输入法不仅加回了空格，还改变了文本框中显示的字符。因为这些字符是输入法推测出来的，所以每次输入一个字符、删除一个字符，输入法会根据当前剩下的所有未确定字符实时推测可能的组合情况，并加入空格。记住这一点很重要，因为我曾将尝试在<code>markedTextRange</code>不为空时去截断<code>textView.text</code>的内容（有个小需求是要检测特定字符，不让其显示到文本框），然后我发现所有一切都变得一团糟。比方说，当你获取到<code>textView.text</code>的值，并截取一段，然后重新设置<code>textView.text</code>之后，你会发现<code>markedTextRanged</code>变为nil，这时候还必须重新设置<code>markedTextRange</code>的值，否则你将会遇到一些奇奇怪怪的现象.

所以最终，我的方案是，只有在<code>markedTextRange==nil</code>时，才会对输入文本做检测，例如字数限制，字符检测，这种情况下再截断就不会有问题。














