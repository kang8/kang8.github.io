---
title: "使用在线算法求解样本方差 - 算法 (第 4 版) 1.2.18 练习题详解"
date: 2022-11-12T12:59:52+08:00
tags: [ "algorithm", "variance", "algs4" ]
draft: false
---

本文是对 [算法 (第 4 版)](https://book.douban.com/subject/19952400) 中[习题 1.2.18](https://github.com/kang8/algs4/blob/d9d9da0fe4d46053d8740c0f9a7ead835405f46a/src/main/java/kang/section_12oop/Ex_1_2_18.java#L6C2-L41) 分析，分析为什么题目中给出的代码可以求解方差。

## 名词解释

* [方差(variance)](https://en.wikipedia.org/wiki/Variance)：是描述一组数据的离散程度。比如比较两个班级的身高差异度，就可以使用方差。方差大的班级，高的更高，矮的更矮。计算方法为：先算出数据的平均值，在求每个数据与平均值差的平方的和，最后除以数据总量。
    * [总体方差(population variance)](https://en.wikipedia.org/wiki/Variance#Population_variance)：计算的数据是全部数据。
    * [样本方差(sample variance)](https://en.wikipedia.org/wiki/Variance#Sample_variance)：计算的数据是部分数据。且计算方法不一样，数据总量为 n，最后除以的是 n-1 而不是 n。n-1 被称为[贝赛尔矫正](https://en.wikipedia.org/wiki/Bessel%27s_correction)。

    既然有了总体方差，那为什么还有一个样本方差？那是因为统计全部数据难度高。比如计算全国身高的方差，就不可能把全国每一个人的身高都统计一遍。

* [标准差(standard deviation)](https://en.wikipedia.org/wiki/Standard_deviation)：标准差的平方是方差。方差可以描述一组数据的离散程度，但其单位与数据的单位不一致。这也是标准差出现的原因。比如，一个班级升高的身差的方差是 $ 100 cm^2 $，标准差就是 $ 10 cm $。

* [在线算法](https://en.wikipedia.org/wiki/Online_algorithm)：在输入的过程中就能完成的一系列算法。比如插入排序，它可以在输入的过程中进行排序，输入完毕也就排好序了。

## 求解

n 组数据的样本方差的公式为：

$$ S_{n} = \frac{1}{n-1} \sum_{i=1}^{n} (x_i - m_n)^{2} $$

其中：

* $S_{n}$ 为样本方差
* $x_i$ 为第 i 个数据
* $m_n$ 为样本的平均值，即： $ m_n = \frac{1}{n} \sum_{i=1}^{n} x_i $

根据 Kevin Wayne 在 [algs4 中的提示](https://github.com/kevin-wayne/algs4/blob/800d8809cac01e1b3ac11bf1fbd6f9ca01a331c8/src/main/java/edu/princeton/cs/algs4/Accumulator.java#L25)知道可以用 1962 年 B. P. Welford 在 Technometrics 发表的 [Note on a method for calculating corrected sums of squares and products](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#cite_note-5) 来解释。

### 前提准备

n 个数据的平均值与 n-1 个数据的平均值的关系为（其中 $x_n$ 为第 n 个数据）：

$$ m_n = \frac{1}{n} ((n-1) m_{n-1}+ x_n) $$

简化一下：

$$ m_n = \frac{n-1}{n} m_{n-1} + \frac{1}{n} x_n $$

这样就可以得出 $x_i - m_n$ 和 $x_n - m_n$ 的表达式：

$$ \begin{split}
x_i - m_n
&= x_i - (\frac{n-1}{n} m_{n-1} + \frac{1}{n} x_n )  \\\
&= x_i - m_{n-1} + \frac{1}{n} m_{n-1} - \frac{1}{n} x_n \\\
&= x_i - m_{n-1} - \frac{1}{n} (x_n - m_{n-1}) \\\
\end{split} $$

$$ \begin{split}
x_n - m_n
&= x_n - (\frac{n-1}{n} m_{n-1} + \frac{1}{n} x_n )  \\\
&= x_n - \frac{1}{n} x_n - \frac{n-1}{n} m_{n-1} \\\
&= \frac{n-1}{n} (x_n - m_{n-1}) \\\
\end{split} $$

### 开始证明

由公式可得：

$$ S_{n} = \frac{1}{n-1} \sum_{i=1}^{n} (x_i - m_n)^{2} $$

把 (n-1) 放到左边，将 $(n-1)S_n$ 看成一个整体：

$$
(n-1)S_n = \sum_{i=1}^{n} (x_i - m_n)^2
$$

拆分下，让其等于 (n-1) 的和加上最后一项：

$$
(n-1)S_n = \sum_{i=1}^{n-1} (x_i - m_n)^2 + (x_n - m_n)^2
$$

将上述的 $x_i - m_n$ 和 $x_n - m_n$ 代入：

$$
(n-1)S_n = \sum_{i=1}^{n-1} \left [(x_i - m_{n-1}) - \frac{1}{n}(x_n - m_{n-1}) \right ]^2 + \left [(\frac{n-1}{n})(x_n - m_{n-1}) \right ]^2
$$

求和公式满足分配律和结合律，且可直接提取常量。故可将式子拆分成：

$$
(n-1)S_n = \sum_{i=1}^{n-1} (x_i - m_{n-1})^2 - 2 (x_n - m_{n-1}) \sum_{i=1}^{n-1} (x_i - m_{n-1}) \\\
\+ \frac{1}{n^2} \sum_{i=1}^{n-1} (x_n - m_{n-1})^2 + (\frac{n-1}{n})^2(x_n - m_{n-1})^2
$$

其中 $x_n$ 和 $m_{n-1}$ 为常量，可以分解下第二个和第三个式子:

$$
\begin{split}
\sum_{i=1}^{n-1} (x_i - m_{n-1})
&= \sum_{i=1}^{n-1} x_i - \sum_{i=1}^{n-1} m_{n-1} \\\
&= \sum_{i=1}^{n-1} x_i - (n-1) m_{n-1} \\\
&= \sum_{i=1}^{n-1} x_i - (n-1) \frac{1}{n-1} \sum_{i=1}^{n-1} x_i\\\
&= 0
\end{split}
$$

$$
\frac{1}{n^2} \sum_{i=1}^{n-1} (x_n - m_{n-1})^2
= \frac{n-1}{n^2} (x_n - m_{n-1})^2
$$

代入上式可得：

$$
\begin{split}
(n-1)S_n
&= \sum_{i=1}^{n-1} (x_i - m_{n-1})^2 + \left [ \frac{n-1}{n^2} + \frac{(n-1)^2}{n^2} \right ](x_n - m_{n-1})^2  \\\
&= S_{n-1} + (\frac{n-1}{n})(x_n - m_{n-1})^{2}
\end{split}
$$

所以下列等式成立：

$$ \begin{split}
S_{n}
&= \frac{1}{n-1} \sum_{i=1}^{n} (x_i - m_n)^{2} \\\
&= S_{n-1} + (\frac{n-1}{n})(x_n - m_{n-1})^{2} \\\
\end{split} $$

其中 $S_{0} = S_{1} = 0$


这样就得到的 $S_{n}$ 与 $S_{n-1}$ 之间的关系。这样就可以在每读一个数据时，都可以使用之前的方差（$S_{n-1}$）加上 $(\frac{n-1}{n})(x_n - m_{n-1})^{2}$ 来计算当前的方差。达到来在线算法的标准。
