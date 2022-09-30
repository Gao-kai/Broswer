## Base64基础

### Base64是什么？
Base64是一种基于64个可打印字符(也就是ASCII字符)来表示二进制数据的编码方式，是将二进制数据转化为字符串的过程，原则上来说一些存储在计算机上的东西都是二进制的，都可以使用base64进行编码。

### Base64编码表
Base64编码之所以称为Base64，是因为其使用64个字符来对任意数据进行编码，这些字符对应的索引表是：
```js
1. A-Z 26个字符 (索引0-25)
2. a-z 26个字符 (索引26-51)
3. 0-9 10个字符 (索引52-61)
4. + (索引62)
5. / (索引63)
```

### Base64编码的原理
将8byte位为一个单元的字节数据拆分成为6个byte位为一个单元的二进制片段，然后将每6个byte位单元对应Base64索引表中的一个字符，最终将得到的字符拼接起来就是最终的Base64编码，举例：

下面是将字符串'MAN'进行Base64编码的过程
```bash
                  文本：M A N
每个字符对应的10进制ASCII码值：77 65 78
8字节的二进制值：01001101 01000001 01001110
base64编码后的6字节单元：010011 010100  000101  001110
base64编码索引：19  20  5  14
base64编码字符：T   U   F   O
```
经过上诉计算可知字符串MAN的Base64编码值为：'TUFO'
如何验证我们的计算是否正确呢？使用浏览器给的api：
```js
window.btoa("MAN"); // 'TUFO' 
```

### Base64编码索引表中没有=但是实际的Base64字符串中有=符号的问题
原因就是编码前的字节数如果刚好被3整除，那么转化为8位的2进制编码之后刚好是3*8 = 24的倍数，当然也就是可以被6整除，这种情况下是不存在补位的情况的。
但是如果字节数不被3整除，就在最后可能会多余出1或2个字节，此时我们就需要在末尾使用000000补足，而补位所需的6个0组成的比特单元就用=符号代替。
```js
window.btoa("MANA"); // 'TUFOQQ==' 不能被3整除 末尾补2个=
```

### base64编码的缺点
通过base64编码的原理可知base64字符串要比原来的体积大1/3，所以在用于data:URI插入资源的时候都选择较小的资源来嵌入文档，以减少http请求的次数。


## Base64编码与解码
在浏览器端为我们提供了2个API来进行Base64编码与解码：
1. 编码
window.btoa();

2. 解码
window.atob();

在node端如果我们想使用Base64编码，那么我们可以选择npm库：js-base64,它导出的对象上有encode和decode方法让我们编码解码。

## Data URI Scheme
data URI scheme（数据URI方案）是URI（统一资源标识符）方案，它提供了一种在网页中内联包含数据的方式，这些数据好像它们是外部资源一样。它目的是将一些小的数据，直接嵌入到网页中，这样就不用再从外部文件载入。常常用于把图片嵌入网页中

### Data URI的组成和写法
一个合格的Data URI由四个部分组成：data:[<MIME>][;base64],<data>
1. 前缀data:
2. MIME媒体类型
3. base64表示URI的数据内容是二进制数据，使用Base64方案以ASCII格式编码,一定记住前面有分号;
4. base64数据本身

比如一个png图片转化为base64字符串然后通过Data URI协议插入到文档的过程是：
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANwAAAAoCAIAAAAaOwPZAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7">
```

### Data URI支持的类型
```css
data:,                              文本数据
data:text/plain,                    文本数据
data:text/html,                     HTML代码
data:text/html;base64,              base64编码的HTML代码
data:text/css,                      CSS代码
data:text/css;base64,               base64编码的CSS代码
data:text/javascript,               Javascript代码
data:text/javascript;base64,        base64编码的Javascript代码
data:image/gif;base64,              base64编码的gif图片数据
data:image/png;base64,              base64编码的png图片数据
data:image/jpeg;base64,             base64编码的jpeg图片数据
data:image/x-icon;base64,           base64编码的icon图片数据
```

## Base64与Base64URL区别
由于标准的Base64编码后可能出现字符+和/，但是这两个字符在URL中就不能当做参数传递，所以就出现了Base64URL，下面是它们的区别：
1. Base64编码后出现的+和/在Base64URL会分别替换为-和_
2. Base64编码中末尾出现的=符号用于补位，这个字符和queryString中的key=value键值对会发生冲突，所以在Base64URL中=符号会被省略，去掉=后怎么解码呢？因为Base64是把3个字节变为4个字节，所以，Base64编码的长度永远是4的倍数，因此，需要加上=把Base64字符串的长度变为4的倍数，就可以正常解码了。

Base64与Base64URL互转在线地址：http://www.lzltool.com/base64url

## URL的几种编码方式
1. encodeURI和decodeURI
2. encodeURIComponent和decodeURIComponent
3. base64URL