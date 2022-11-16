---
title: ThinkPhp3.2中的not in Bug
tags:
  - thinkphp
id: '171'
categories:
  - - PHP
date: 2017-09-19 11:23:32
---
有时候在一些比较老的系统中做开发，用的thinkphp3.2框架。 

不得不说，thinkphp有它的优点 ，但也发现了不少缺陷。比如今天说的，就是进行sql查询时的bug。 

thinkphp支持where函数传入查询条件，当我要查询id在，或者不在一些数组里面的时候，代码可以这么写：

```
$ids = array();
$data = M('User')->where(array(
    'id'=>array('not in',$ids)
))->select();
```

注意，这里的$ids是一个空数组。这种情况，无论是not in，还是in，thinkphp拼装出来的sql是这样的：

```
SELECT * FROM `user` WHERE `id` NOT IN () ;
```

而这样的sql语句运行是会报错的。作为一个框架，在传入参数都合法的情况下，竟然生成一条报错的sql，实在是不应该。 

正确的SQL应该是这样的：

```
SELECT * FROM `user` WHERE `id` NOT IN ('') ;
```

所以，我对ThinkPhp3.2框架的DB层进行了简单的修改。 

在**Common/ThinkPHP/Library/Think/Db/Driver.class.php**的**parseWhereItem**方法里，可以找到这样的代码：

```
}elseif(preg_match('/^(notinnot inin)$/',$exp)){ // IN 运算
    if(isset($val[2]) && 'exp'==$val[2]) {
        $whereStr .= $key.' '.$this->exp[$exp].' '.$val[1];
    }else{
        if(is_string($val[1])) {
             $val[1] =  explode(',',$val[1]);
        }
        $zone      =   implode(',',$this->parseValue($val[1]));
        $whereStr .= $key.' '.$this->exp[$exp].' ('.$zone.')';
    }
}
```

我简单的修改后，变成了：

```
}elseif(preg_match('/^(notinnot inin)$/',$exp)){ // IN 运算
    if(isset($val[2]) && 'exp'==$val[2]) {
        $whereStr .= $key.' '.$this->exp[$exp].' '.$val[1];
    }else{
        if(is_string($val[1])) {
             $val[1] =  explode(',',$val[1]);
        }
        $zone      =   implode(',',$this->parseValue($val[1]));
        if(empty($zone)){
            $whereStr .= $key.' '.$this->exp[$exp].' (\'\')';
        }else{
            $whereStr .= $key.' '.$this->exp[$exp].' ('.$zone.')';
        }
    }
}
```

这样，即使传入一个空数组，也不会报错，而且还会返回你想要的查询结果。
