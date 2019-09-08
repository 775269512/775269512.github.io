---
layout: post
title: 用列主元Gauss消去法求解方程组
date: 2019-09-09
tags: Mathematics
---

# 高斯消元思想

## 背景
人们在应用高斯消去法求解的过程中发现他的计算具有一定的局限性，只有在≠0的情况下才可以正常进行计算，是按照方程组中方程给定的顺序进行的，但是当=0时，消元过程就无法进行。另外，即使当≠0时，如果其他值很小，用它做除数时，根据数值运算应遵循的原则知道这种情况舍入误差会增大，严重影响计算结果的精度。为克服这一局限性，我们便可以引入本论文所要讨论的方法—列主元高斯消去法。

## 消元过程
![img](/images/posts/matlab/9.png)
![img](/images/posts/matlab/10.png)
![img](/images/posts/matlab/11.png)
![img](/images/posts/matlab/12.png)

## 回代过程
![img](/images/posts/matlab/13.png)

## 乘除运算量
![img](/images/posts/matlab/14.png)

## 列主元优化高斯消去
![img](/images/posts/matlab/15.png)


# 编程思想

* 1、输入矩阵阶数n,增广矩阵A(n,n+1)

* 2、对k=1,2,…n：(a)按列主元，选取l使； (b)如果l≠k，交换A(n,n+1)的第k行与第l行元素；(c)消元计算：
              i=k+1,…,n；
          i=k+1,…,n  j=k+1,…,n+1

* 3、回代计算： i=n,n-1,…,1

* 4、输出解向量。


## 代码实现

```matlab
% GaussElimination.m
%% Gauss消元法的Matlab实现
function [r_matrix] = GaussElimination(Coefficient_matrix,Load_matrix)
%  inputs:
%         Coefficient_matrix:系数矩阵，为n*n维方阵
%         Load_matrix       :载荷矩阵，为n*1维矩阵
%  outputs：
%         r_matrix:计算结果向量,为n*1为矩阵
 
% 判断输入矩阵维度是否满足要求
[row_coeff,col_coeff] = size(Coefficient_matrix);
[row_load,~] = size(Load_matrix');
 
% 初始化r_matrix矩阵
r_matrix = zeros(row_load,1);
% 判断输入的维度是否满足要求
if (row_coeff ~= col_coeff) || (row_coeff ~= row_load)
    % 不满足则输出错误提示
    print('输入错误！');
else
    % 满足则进行下一步运算
    for k = 1:row_coeff-1
        % 检查主对角元素第i行的第i个元素是否为0
        if Coefficient_matrix(k,k) == 0
            print('主对角元素错误！');
        else
            % 循环计算第k+1行到最后一行
            for i = k+1:row_coeff
               L_ik = Coefficient_matrix(i,k) / Coefficient_matrix(k,k); %更新L_ik
               % 更新每一行从第i个元素开始后的所有元素
               for j = k+1:col_coeff
                   Coefficient_matrix(i,j) = Coefficient_matrix(i,j) - ...
                       L_ik*Coefficient_matrix(k,j); % 更新a(i,j)
               end
               Load_matrix(i) = Load_matrix(i) - Load_matrix(k)*L_ik; % 更新b(i)
               Coefficient_matrix(i,1) = 0; 
            end %for循环结束
        end % 条件2结束
    end % for循环结束
    % 回代过程
    r_matrix(row_coeff) = Load_matrix(row_coeff)/Coefficient_matrix(row_coeff,col_coeff);
    for k = row_coeff-1:-1:1
        sum_temp = 0;
        for j = k+1:col_coeff
            sum_temp = sum_temp + Coefficient_matrix(k,j)*r_matrix(j);
        end
        r_matrix(k) = (Load_matrix(k) - sum_temp)/Coefficient_matrix(k,k);
    end
end % 条件判断结束
end % 函数结束
```

# 测试分析

设输入a = [1,2;5,6], b = [12,24],则计算结果为result=[-6,9]。通过手算验证该结果为正确结果。
 
下面对算法的时间代价进行测试，输入维度分别取100,200,300,400,500,600,700,800,900,1000，输入元素采用1-100的整数，计算时间代价如下表所示。


![img](/images/posts/matlab/16.png)
 
