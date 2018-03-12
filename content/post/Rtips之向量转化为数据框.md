---
title: Rtips之向量转化为数据框
author: Delta
date: '2018-03-01'
categories:
  - R tips
tags:
  - R
  - R tips
  - 数据框
toc: yes
description: '向量与数据框更配哦'
---

## 前言
前段时间在和小伙伴一起研究自动输出三线表时，遇到一个问题，就是将所得的一系列指标p值和指标summary表一一对应。说着有点抽象，简单点说就是怎么将一个向量根据名字合并到一个数据集中作为一列。

<!-- more-->

#### 举个例子
如下表df1所示，需要添加一列p值，而这个值已经提取在了一个向量p中，该怎么做呢？


```R
# create example df1
df1 <- data.frame(
  variables = c('x1', 'x2', 'x3'),
  mean1 = c(5.6, 12, 24) ,
  mean2 = c(6.5, 10, 30),
  stringsAsFactors = FALSE
)
df1
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>variables</th><th scope=col>mean1</th><th scope=col>mean2</th></tr></thead>
<tbody>
    <tr><td>x1  </td><td> 5.6</td><td> 6.5</td></tr>
    <tr><td>x2  </td><td>12.0</td><td>10.0</td></tr>
    <tr><td>x3  </td><td>24.0</td><td>30.0</td></tr>
</tbody>
</table>
<!--/html_preserve-->



```R
(p <- c(0.02, 0.04, 0.001))
```

<!--html_preserve-->
<ol class=list-inline>
    <li>0.02</li>
    <li>0.04</li>
    <li>0.001</li>
</ol>
<!--/html_preserve-->


**PS**:在R中可以通过将变量括号括起来，然后就会直接打印出来，在学习过程中可节省时间。

有小伙伴会说直接将p转为数据框然后再merge一下就可以嘛。是的，但是如果我们得到的p值是x1和x3的p值，即0.02和0.001，那该怎么加进去呢。之所以有这种情况是因为我们输出三线表时有些变量时需要用多行的，如性别，就有男女两行来展现其数据。下面我们就来演示怎么将两个p值加入到数据框。

## 开工


```R
(p1 <- c(0.02, 0.001))
```

<!--html_preserve-->
<ol class=list-inline>
    <li>0.02</li>
    <li>0.001</li>
</ol>
<!--/html_preserve-->


先把p值和变量名字对应，对向量进行命名有多种形式：


```R
(p11 <- c(x1 = 0.02, x3 = 0.001))
```

<!--html_preserve-->
<dl class=dl-horizontal>
    <dt>x1</dt>
        <dd>0.02</dd>
    <dt>x3</dt>
        <dd>0.001</dd>
</dl>
<!--/html_preserve-->



```R
# 或者
(p11 <- setNames(p1, c('x1', 'x3')))
```

<!--html_preserve-->
<dl class=dl-horizontal>
    <dt>x1</dt>
        <dd>0.02</dd>
    <dt>x3</dt>
        <dd>0.001</dd>
</dl>
<!--/html_preserve-->



```R
# 再或者
p11 <- c(0.02, 0.001)
names(p11) <- c('x1', 'x3')
p11
```

<!--html_preserve-->
<dl class=dl-horizontal>
    <dt>x1</dt>
        <dd>0.02</dd>
    <dt>x3</dt>
        <dd>0.001</dd>
</dl>
<!--/html_preserve-->


三种方法都可以的得到我们想要的。传说中的条条大路通罗马。那我们接下来把p11转化为数据框：


```R
(p11.df <- as.data.frame(p11))
# p11.df结构
str(p11.df)
```

<!--html_preserve-->
<table>
<thead><tr><th></th><th scope=col>p11</th></tr></thead>
<tbody>
    <tr><th scope=row>x1</th><td>0.020</td></tr>
    <tr><th scope=row>x3</th><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


    'data.frame':   2 obs. of  1 variable:
     $ p11: num  0.02 0.001
    

我们可以看到数据框有一个变量名叫p11，也就是我们原来的向量名字。如果这时候我们直接合并是做不到的，因为p11.df没有可以索引的，无法和df1中的variables对应。所以我们要添加一列把p11.df的行名字作为一列。

这里要引入来自Hadley大神的`tibble`包，其中有个函数`rownames_to_column()`可以将行名字转换为列，与之对应的是`column_to_rownames()`我们看一下：


```R
# 载入包
library("tibble")
# 将行名转化为列
(p11.df <- rownames_to_column(p11.df, var = 'variables'))
str(p11.df)
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>variables</th><th scope=col>p11</th></tr></thead>
<tbody>
    <tr><td>x1   </td><td>0.020</td></tr>
    <tr><td>x3   </td><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


    'data.frame':   2 obs. of  2 variables:
     $ variables: chr  "x1" "x3"
     $ p11      : num  0.02 0.001
    

此时已经增加了一列叫‘variables’，那接下来就简单了，直接将p11.df和df合并就可以，合并的方法很多，我们依旧使用Hadley开发的`dplyr`包中的`left_join`函数，当然基础包中的`merge`也能胜任。


```R
dplyr::left_join(df1, p11.df, by = 'variables')
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>variables</th><th scope=col>mean1</th><th scope=col>mean2</th><th scope=col>p11</th></tr></thead>
<tbody>
    <tr><td>x1   </td><td> 5.6 </td><td> 6.5 </td><td>0.020</td></tr>
    <tr><td>x2   </td><td>12.0 </td><td>10.0 </td><td>   NA</td></tr>
    <tr><td>x3   </td><td>24.0 </td><td>30.0 </td><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


大功告成！！！

其实，有更简单的方法，刚刚我们进行了两步，一步是将p11转化为数据框，再添加了一列变量名。而在大神`tibble`包中有个函数可以直接做到这步，就是`enframe`:


```R
p11
enframe(p11, name = 'variables', value = 'P value')
```

<!--html_preserve-->
<dl class=dl-horizontal>
    <dt>x1</dt>
        <dd>0.02</dd>
    <dt>x3</dt>
        <dd>0.001</dd>
</dl>
<!--/html_preserve-->


<!--html_preserve-->
<table>
<thead><tr><th scope=col>variables</th><th scope=col>P value</th></tr></thead>
<tbody>
    <tr><td>x1   </td><td>0.020</td></tr>
    <tr><td>x3   </td><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


可以看到不仅一步做到了，而且还可以对变量进行命名，感觉是非常的厉害了。

## 收工
这里面我们是将vector转化为数据框再合并，如果是list的话，需要进行转置，因为转为数据框后list名字变为了数据框的变量名：


```R
(q11 <- list(x1 = 0.02, x3 = 0.001))
str(q11)
```

<!--html_preserve-->
<dl>
    <dt>$x1</dt>
        <dd>0.02</dd>
    <dt>$x3</dt>
        <dd>0.001</dd>
</dl>
<!--/html_preserve-->


    List of 2
     $ x1: num 0.02
     $ x3: num 0.001
    


```R
as.data.frame(q11)
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>x1</th><th scope=col>x3</th></tr></thead>
<tbody>
    <tr><td>0.02 </td><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


这样我们就要再转置为宽数据格式，其实可以通过`reshape2`包中的`melt`函数完成：


```R
reshape2::melt(q11)
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>value</th><th scope=col>L1</th></tr></thead>
<tbody>
    <tr><td>0.020</td><td>x1   </td></tr>
    <tr><td>0.001</td><td>x3   </td></tr>
</tbody>
</table>
<!--/html_preserve-->


可以看到也是一步完成了，不过要再对数据框进行变量重新命名才能进行合并。那么这里大家肯定会想，如果用`enframe`函数会怎样呢？我们来看一看：


```R
enframe(q11)
str(enframe(q11))
```

<!--html_preserve-->
<table>
<thead><tr><th scope=col>name</th><th scope=col>value</th></tr></thead>
<tbody>
    <tr><td>x1  </td><td>0.02</td></tr>
    <tr><td>x3   </td><td>0.001</td></tr>
</tbody>
</table>
<!--/html_preserve-->


    Classes 'tbl_df', 'tbl' and 'data.frame':   2 obs. of  2 variables:
     $ name : chr  "x1" "x3"
     $ value:List of 2
      ..$ : num 0.02
      ..$ : num 0.001
    

也是一步完成，但是，这看上去是一步完成，我们仔细看下这个的数据结构，发现变量value下面的值是两个列表，厉害了~在数据框中的一个变量下面的值还可以是一个列表。其实这个函数最初开发出来的主要目的就是在为了将列表完美的储存在数据框中，而这个的好处呢，不在今天的话题，以后再说吧~

## 总结
说了这么多，今天的操作其实很简单，就是将向量或者列表转为数据框，然后顺带处理一下数据框的行名。下面是本文所涉及的重要函数：

```R
rownames_to_column()
enframe()
left_join()
melt()
```

### Peace out~
