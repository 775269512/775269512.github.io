---
layout: post
title: Cholesky分解法求解三对角方程组
date: 2019-09-11
tags: Mathematics
---

# Cholesky分解

## 背景
**Cholesky分解法**又叫平方根法，是求解对称正定线性方程组最常用的方法之一。对于一般矩阵，为了消除LU分
解的局限性和误差的过分积累，采用了选主元的方法，但对于对称正定矩阵而言，选主元是不必要的。


## Cholesky分解的基本原理

![img](/images/posts/matlab/18.png)

现在的关键问题是对进行Cholesky分解。假设：
![img](/images/posts/matlab/19.png)
![img](/images/posts/matlab/20.png)




## Cholesky 分解在计算马氏距离时的作用
![img](/images/posts/matlab/21.png)

## 代码实现

```matlab
%Cholesky分解法
%若X为非对称正定，则输出错误信息
%在matlab中chol(X)用于对矩阵X进行分解，其格式如下
%R = chol(X):产生一个上三角矩阵R,使得R'R=X 。
function [X]=pingfg(A,b)
[N,N]=size(A);
X=zeros(N,1);
Y=zeros(N,1);
%A = L*L^T
for i=1:N
    A(i,i)=sqrt(A(i,i)-A(i,1:i-1)*A(i,1:i-1)');
    if A(i,i)==0
        disp('矩阵没有唯一解方案，为非对称正定');
        break
    end
    for j=i+1:N
        A(j,i)=(A(j,i)-A(j,1:i-1)*A(i,1:i-1)')/A(i,i);
    end
end

%前代法
%求y
for j=1:N
    Y(j)=(b(j)-A(j,1:j-1)*Y(1:j-1))/A(j,j);
end
%求x
A=A';
for k=N:-1:1
    X(k)=(Y(k)-A(k,k+1:N)*X(k+1:N))/A(k,k);
end
```

## 运行及结果
命令行输入： A =[1     2     3;2     8     8;3     8    35];b = [3 4 5];
x = pingfg(A,b)


x =

    4.2400
   -0.4400
   -0.1200

