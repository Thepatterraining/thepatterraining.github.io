---
title: php-mpdf扩展包中文乱码问题
date: 2020-08-24 10:12:47
tags: ['php','mpdf']
category: php
article: php-mpdf扩展包中文乱码问题
---

# php-mpdf扩展包中文乱码问题

[mpdf](http://mpdf.github.io/fonts-languages/fonts-in-mpdf-7-x.html)是一个可以把html网页转换成pdf文件的扩展包。一开始使用的时候，发现中文乱码了。。在网上查了半天，好多方法都不管用。

最后，在他的文档里面找到了问题原因。

想要输出中文，有两个参数至关重要！！！

- autoLangToFont 这个值一定要设置为true才可以
- autoScriptToLang 这个值也一定要设置为true才可以

只要上面两个设置为true，那么你的中文就可以正常输出了。相信我，不能正常输出你来打我。

看一下mpdf文档上面的描述。

!(mpdf)[../images/mpdf1.png]

!(mpdf)[../images/mpdf2.png]

可以看到默认值是false，所以我们使用的时候需要改成true。

设置这两个值也很简单。

```php

use Mpdf\Mpdf;

function test() {
    $pdf = new Mpdf;
    $pdf->autoLangToFont = true;
    $pdf->autoScriptToLang = true;

    $pdf->writeHTML('<h1>123</h1>');

	return $pdf->output('./test.pdf', 'D');
}
```

其实，mpdf的文档最开始是有错误的，他的文档中写的默认值是`true`而不是现在的`false`。不过从他的源码上可以看到他的默认值其实是`false`。

源码位置：`vendor/mpdf/mpdf/src/Config/ConfigVariables.php`里面。
这个文件里面是很多变量的默认值，在这里面搜索可以看到这两个值是false。

```php

// AUTOMATIC FONT SELECTION
// Based on script and/or language
// mPDF 6.0 (similar to previously using function SetAutoFont() )
'autoScriptToLang' => false,

// mPDF 6.0 (similar to old useLang)
'autoLangToFont' => false,
```

我给他们的github上面提了一个issue，他们才把文档改过来了。

!(mpdf)[../images/mpdf3.png]

最后附上mpdf官方文档：

> http://mpdf.github.io/fonts-languages/fonts-in-mpdf-7-x.html

我给他们提的issue:

> https://github.com/mpdf/mpdf.github.io/issues/141








