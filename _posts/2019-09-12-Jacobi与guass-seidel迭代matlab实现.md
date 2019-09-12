---
layout: post
title: Jacobi与guass-seidel求解线性方程组近似解
date: 2019-09-12
tags: Mathematics
---

# 雅可比迭代

## 引例

![img](/images/posts/matlab/23.png)


## 雅可比迭代的基本原理

![img](/images/posts/matlab/22.png)


## 代码实现

```matlab
function [x,n] = jacobi(A,b,x0,eps,it_max)
%  求线性方程组的Jacobi迭代法，调用格式为
%  [x, k] = jacobi(A,b,x0,eps,it_max)
%  其中, A 为线性方程组的系数矩阵，b 为常数项，eps 为精度要求，默认为1e-6,
%  it_max 为最大迭代次数，默认为200
%  x 为线性方程组的解，k迭代次数
if nargin ==3
    eps = 1.0e-6;
    M = 200;
elseif nargin<3
    disp('输入参数数目不足3个');
    return
elseif nargin ==5
    M = it_max;
end
D = diag(diag(A));%求A的对角矩阵
L = -tril(A,-1);%求A的下三角矩阵
U = -triu(A,1);%求A的上三角矩阵
B = D\(L+U);
f = D\b;
x = B*x0+f;
n = 1;%迭代次数
while norm(x-x0)>=eps
    x0 = x;
    x = B*x0+f
    n = n+1;
    if(n>=M)
        disp('Warning:迭代次数太多,可能不收敛!')
        return;
    end
end
```

## 运行及结果
命令行输入： 

```
clear
clc
A = [8 -3 2;4 11 -1;6 3 12];
b = [20;33;36];
[x, n] = jacobi(A,b,[0,0,0]',1.0e-7,30)
```
*结果:迭代次数18次*

| x1   |  x2     |  x3 |....|  x17     |  x18 |
|:---:|:---:|:------:|:---:|:---:|:------:|
| 2.8750 |  3.1364 | 3.0241|....|3.0000|3.0000|
| 2.3636| 2.0455   |  1.9478 |....|2.0000|2.0000|
|1.0000 |  0.9716 |   0.9205|....|1.0000|1.0000|


# 高斯-赛德尔迭代

## 高斯-赛德尔迭代的基本原理

![img](/images/posts/matlab/24.png)



## 代码实现

```matlab
function [x,n] = guaseidel(A,b,x0,eps,it_max)
% 求线性方程组的Gauss-Seidel迭代法，调用格式为
%  [x, k] = guaseidel(A,b,x0,eps,it_max)
%  其中, A 为线性方程组的系数矩阵，b 为常数项，eps 为精度要求，默认为1e-5,
%  it_max 为最大迭代次数，默认为100
%  x 为线性方程组的解，k迭代次数
if nargin == 3
    eps = 1.0e-6
    it_max= 200
elseif nargin == 4
    it_amx = 200
elseif nargin <3
    disp('输入参数个数不足3个');
    return;
end
D = diag(diag(A));%求A的对角矩阵
L = -tril(A,-1);%求A的下三角矩阵,不带对角线
U = -triu(A,1);%求A的上三角矩阵
G = (D-L)\U;
f = (D-L)\b;
x = G*x0+f;
n=1;  %迭代次数
while norm(x-x0)>=eps
    x0 = x;
    x = G*x0+f
    n = n+1;
    if(n>=it_max)
        disp('Warning:迭代次数太多,可能不收敛');
        return;
    end
end
```

## 运行及结果
命令行输入： 

```
clear
clc
A = [8 -3 2;4 11 -1;6 3 12];
b = [20;33;36];
[x, n] = guaseidel(A,b,[0,0,0]',1.0e-7,30)
```
*结果:迭代次数10次*

| x1   |  x2     |  x3 |....|  x9     |  x10 |
|:---:|:---:|:------:|:---:|:---:|:------:|
| 2.9773|    3.0098 | 2.9998|....|3.0000|3.0000|
|  2.0289|   1.9968 |   1.9997 |....|2.0000|2.0000|
| 1.0041|    0.9959 |    1.0002|....|1.0000|1.0000|


# 比较

以上介绍了求解线性方程组的两种迭代法，一般来说，高斯-赛德尔迭代要比雅可比迭代好。但情况并不总是这样，有时候G-S迭代收敛的慢或者反而发散。
