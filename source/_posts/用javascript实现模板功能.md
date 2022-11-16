---
title: 用Javascript实现模板功能
tags:
  - javascript
  - 模板
categories:
  - - JavaScript
date: 2018-04-23 14:09:10
---
最近在后台做一个功能，传给后台API的是一个多层的js对象。 

如果用Vue做的话，只需要操作变量，模板就会自动更新内容了，十分方便。 

但是当前后台用的是JQuery，因此打算自己实现一个模板替换JS变量的功能。 

搜索了一些资料，发现js的replace替换字符串功能有限，反而用正则就灵活的多。 

一种比较推荐的写法是：

```javascript
var str = "string1 is a boy,string2 is a girl";
str = str.replace(/string1string2/gi,function(mached){
    var map = {
        'string1':'xxxx',
        'string2':'xxxxx',
    };
    return map[mached];
});
```

通过 这样来实现多个变量的替换。看到这种代码，我觉得用PHP还真是方便许多。 后来又查了一些资料，发现有人也写了相似的功能，文章链接： [http://www.hsfzxjy.site/a-simple-javascript-template-language/](http://www.hsfzxjy.site/a-simple-javascript-template-language/) 

博主仅用17行代码就搞定了，十分方便。不过貌似忘了右边反斜杠转译字符的替换了，所以我简单修改一下，代码如下：

```javascript
function render(template, context) {
    var tokenReg = /(\\)?\{([^\{\}\\]+)(\\)?\}/g;
    return template.replace(tokenReg, function (word, slash1, token, slash2) {
        if (slash1  slash2) {
            return word.replace(/\\/g, '');
        }
        var variables = token.replace(/\s/g, '').split('.');
        var currentObject = context;
        var i, length, variable;
        for (i = 0, length = variables.length; i < length; ++i) {
            variable = variables[i];
            currentObject = currentObject[variable];
            if (currentObject === undefined  currentObject === null) return '';
        }
        return currentObject;
    })
}

String.prototype.render = function (context) {
    return render(this, context);
};

//使用方法
//支持变量里有空格，被转译的分隔符{和}不会被渲染
"{   greeting   }! My name is { author.name } \\{aaa \\}.".render({
    greeting: "Hi",
    author: {
        name: "hsfzxjy"
    }
});
// "Hi! My name is hsfzxjy {aaa }."
```

  需要的朋友请拿走，不谢咯~
