---
layout: post
title: 线性回归
---

{{ page.title }}
================

<p class="meta"> 2014.11.01 - 北京</p>

<title>线型回归(OLS 最小二乘法)</title>
<body>
<h1>线型回归(OLS 最小二乘法)</h1>

<p>我们对回归的直观理解是这样的：</p>

<pre><code class="r">require(&quot;car&quot;)
</code></pre>

<pre><code>## Loading required package: car
</code></pre>

<pre><code class="r">scatterplot(weight ~ height, data = women, spread = F, pch = 19, main = &quot;Women Age 30-39&quot;, 
    xlab = &quot;Height(inches)&quot;, ylab = &quot;Weight(lbs.)&quot;)
</code></pre>


<p><strong>上图是身高和体重的散点图，直线为线性拟合，曲线为曲线平滑拟合，边界为箱线图。粗略看图可以得知，两个变量基本对称，且基本呈线性关系，曲线拟合的比直线要好。</strong></p>

<p>我们再看一个多元回归的例子，首先看各个变量之间的相关性，再看看拟合曲线的情况：</p>

<pre><code class="r">require(&quot;xtable&quot;)
</code></pre>

<pre><code>## Loading required package: xtable
</code></pre>

<pre><code class="r">states &lt;- as.data.frame(state.x77[, c(&quot;Murder&quot;, &quot;Population&quot;, &quot;Illiteracy&quot;, 
    &quot;Income&quot;, &quot;Frost&quot;)])
options(digits = 2)
corstates &lt;- cor(states)
xtablecs &lt;- xtable(corstates, colnames = T)
print(xtablecs, type = &quot;html&quot;)
</code></pre>

<!-- html table generated in R 3.0.2 by xtable 1.7-1 package -->

<!-- Mon Dec 30 17:59:18 2013 -->

<TABLE border=1>
<TR> <TH>  </TH> <TH> Murder </TH> <TH> Population </TH> <TH> Illiteracy </TH> <TH> Income </TH> <TH> Frost </TH>  </TR>
  <TR> <TD align="right"> Murder </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.34 </TD> <TD align="right"> 0.70 </TD> <TD align="right"> -0.23 </TD> <TD align="right"> -0.54 </TD> </TR>
  <TR> <TD align="right"> Population </TD> <TD align="right"> 0.34 </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.11 </TD> <TD align="right"> 0.21 </TD> <TD align="right"> -0.33 </TD> </TR>
  <TR> <TD align="right"> Illiteracy </TD> <TD align="right"> 0.70 </TD> <TD align="right"> 0.11 </TD> <TD align="right"> 1.00 </TD> <TD align="right"> -0.44 </TD> <TD align="right"> -0.67 </TD> </TR>
  <TR> <TD align="right"> Income </TD> <TD align="right"> -0.23 </TD> <TD align="right"> 0.21 </TD> <TD align="right"> -0.44 </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.23 </TD> </TR>
  <TR> <TD align="right"> Frost </TD> <TD align="right"> -0.54 </TD> <TD align="right"> -0.33 </TD> <TD align="right"> -0.67 </TD> <TD align="right"> 0.23 </TD> <TD align="right"> 1.00 </TD> </TR>
   </TABLE>

<p>用cor函数看各洲的谋杀、人口、文盲、收入、结霜天数之间的相关系数，看到谋杀和文盲呈强相关（一般相关系数&gt;0.7,我们可以称之为强相关）；谋杀和结霜天数，结霜天数和收入之间呈现一定的弱相关。</p>

<p>我们再用scatterplotMatrix来看一下，各个变量的分布情况：</p>

<pre><code class="r">scatterplotMatrix(states, spread = F, pch = 19, main = &quot;Example of Regression &quot;)
</code></pre>


<p><strong>上图是谋杀、人口、文盲率、收入、结霜天数之间的关系。其中直线是线性拟合，曲线是平滑拟合曲线。从图中我们大致可以看出谋杀和文盲呈现一定的正相关性，和冰冻天数呈现一定的负相关。谋杀随着人口和文盲的增加而增加，随着收入水平和结霜天数的增加而下降，同时越是寒冷的州，犯罪率越低，人口越少，文盲越少，收入越高等等的特征。</strong></p>

<p>看了上面的例子，我们先来看看什么叫回归,从很多方面来看回归都是统计学的核心。回归是一个广义的概念，通指那些用一个或多个预测变量（也称自变量或解释变量）来预测响应变量（也称因变量或者结果变量）。对于回归，有很多种类型，比如有OLS 最小二乘回归、Logistic回归、泊松回归等。在本次介绍中我们先从普通的OLS最小二乘回归开始介绍，接下来再介绍Logistic 回归等方法。OLS最小二乘回归能解决哪些问题呢？从理想角度出发OLS可以帮我们回答以下一些问题：</p>

<pre><code>  1. 自变量和因变量之间到底有什么关系
  2. 哪些自变量相对比较重要，从而监控这些影响因变量的自变量
  3. 在建模过程中发现的离群点（异常值）往往会有一些重大发现
  4. 对不同的分类，都适用吗？是否不用的类型有不同的表现
</code></pre>

<p>既然OLS最小二乘回归是用来拟合线型模型，那么哪种方式叫线型模型，或者说线型模型的表达形式如何:</p>

<ol>
<li>线性：\( y_{i}=\beta_{0}+\beta_{1}x_{{1}{i}} \)<br/></li>
<li>线性：\( y_{i}=\beta_{0}+\beta_{1}x_{{1}{i}}+\cdots+\beta_{k}x_{{k}{i}} \)<br/></li>
<li>线性：\( y_{i}=\beta_{0}+\beta_{1}\lg x_{{1}{i}}+\beta_{2}\sin x_{{2}{i}} \) </li>
<li>非线性：\( y_{i}=\beta_{0}+\beta_{1}e^{x_{{1}{i}}} \)<br/></li>
</ol>

<p>OLS的目标是什么？是通过减少响应变量的真实值与预测值的差值来获得模型参数，具体而言，即使得残差平方和最小，即求</p>

<ol>
<li>\( \sum_{1}^{n}(y_{i}-\hat{y_{i}})^2=\sum_{1}^{n}\epsilon^2 \) 的最小值</li>
</ol>

<p>OLS回归的主要步骤有：</p>

<pre><code> 1. 读取数据
 2. 看数据的分布情况等特性
 3. 拟合并解释线性模型
 4. 验证模型假设
 5. 模型准确率评估
 6. 模型选择
</code></pre>

<p>知道了OLS回归的一些基本情况后，我们通过实际的例子从头至尾来做一遍回归：</p>

<p>1.读取数据，R提供了各种接口来读取数据，支持从excle、文本、oracle等各种数据库、网页、各种统计软件比如SPSS、SAS、键盘输入等,把txt文件读入对象codata中：</p>

<pre><code class="r">getwd()
</code></pre>

<pre><code>## [1] &quot;/Users/bluerays/work/regression&quot;
</code></pre>

<pre><code class="r">setwd(&quot;/Users/bluerays/work/regression&quot;)
codata &lt;- read.table(&quot;co.txt&quot;, head = T)
</code></pre>

<p>2.粗略了解数据特性,首先看描述性统计量，以便决定是否要进行缺省值补充、数据归一化等：</p>

<pre><code class="r">codata_s &lt;- summary(codata)
codata_s &lt;- xtable(codata_s)
print(codata_s, type = &quot;html&quot;)
</code></pre>

<!-- html table generated in R 3.0.2 by xtable 1.7-1 package -->

<!-- Mon Dec 30 17:59:18 2013 -->

<TABLE border=1>
<TR> <TH>  </TH> <TH>      Hour </TH> <TH>       CO </TH> <TH>    Traffic </TH> <TH>      Wind </TH>  </TR>
  <TR> <TD align="right"> 1 </TD> <TD> Min.   : 1.0   </TD> <TD> Min.   :1.2   </TD> <TD> Min.   : 10   </TD> <TD> Min.   :-0.2   </TD> </TR>
  <TR> <TD align="right"> 2 </TD> <TD> 1st Qu.: 6.8   </TD> <TD> 1st Qu.:2.9   </TD> <TD> 1st Qu.: 78   </TD> <TD> 1st Qu.: 0.0   </TD> </TR>
  <TR> <TD align="right"> 3 </TD> <TD> Median :12.5   </TD> <TD> Median :5.2   </TD> <TD> Median :178   </TD> <TD> Median : 0.8   </TD> </TR>
  <TR> <TD align="right"> 4 </TD> <TD> Mean   :12.5   </TD> <TD> Mean   :4.5   </TD> <TD> Mean   :159   </TD> <TD> Mean   : 2.0   </TD> </TR>
  <TR> <TD align="right"> 5 </TD> <TD> 3rd Qu.:18.2   </TD> <TD> 3rd Qu.:6.5   </TD> <TD> 3rd Qu.:248   </TD> <TD> 3rd Qu.: 4.0   </TD> </TR>
  <TR> <TD align="right"> 6 </TD> <TD> Max.   :24.0   </TD> <TD> Max.   :7.4   </TD> <TD> Max.   :308   </TD> <TD> Max.   : 5.9   </TD> </TR>
   </TABLE>

<p>3.不同的变量之间量级差别比较大，对数据进行归一化处理：</p>

<pre><code class="r">codata_z &lt;- subset(codata, select = c(&quot;Hour&quot;, &quot;Traffic&quot;, &quot;Wind&quot;))
codata_z &lt;- scale(codata_z, center = T, scale = T)
codata_z &lt;- cbind(subset(codata, select = c(&quot;CO&quot;)), codata_z)
codata_z_s &lt;- summary(codata_z)
codata_z_s &lt;- xtable(codata_z_s)
print(codata_z_s, type = &quot;html&quot;)
</code></pre>

<!-- html table generated in R 3.0.2 by xtable 1.7-1 package -->

<!-- Mon Dec 30 17:59:18 2013 -->

<TABLE border=1>
<TR> <TH>  </TH> <TH>       CO </TH> <TH>      Hour </TH> <TH>    Traffic </TH> <TH>      Wind </TH>  </TR>
  <TR> <TD align="right"> 1 </TD> <TD> Min.   :1.2   </TD> <TD> Min.   :-1.63   </TD> <TD> Min.   :-1.52   </TD> <TD> Min.   :-0.95   </TD> </TR>
  <TR> <TD align="right"> 2 </TD> <TD> 1st Qu.:2.9   </TD> <TD> 1st Qu.:-0.81   </TD> <TD> 1st Qu.:-0.83   </TD> <TD> 1st Qu.:-0.86   </TD> </TR>
  <TR> <TD align="right"> 3 </TD> <TD> Median :5.2   </TD> <TD> Median : 0.00   </TD> <TD> Median : 0.20   </TD> <TD> Median :-0.52   </TD> </TR>
  <TR> <TD align="right"> 4 </TD> <TD> Mean   :4.5   </TD> <TD> Mean   : 0.00   </TD> <TD> Mean   : 0.00   </TD> <TD> Mean   : 0.00   </TD> </TR>
  <TR> <TD align="right"> 5 </TD> <TD> 3rd Qu.:6.5   </TD> <TD> 3rd Qu.: 0.81   </TD> <TD> 3rd Qu.: 0.90   </TD> <TD> 3rd Qu.: 0.86   </TD> </TR>
  <TR> <TD align="right"> 6 </TD> <TD> Max.   :7.4   </TD> <TD> Max.   : 1.63   </TD> <TD> Max.   : 1.52   </TD> <TD> Max.   : 1.68   </TD> </TR>
   </TABLE>

<p>4.看各变量之间的相关性以及分布情况：</p>

<pre><code class="r">cor_codata_z &lt;- cor(codata_z)
cor_codata_z_t &lt;- xtable(cor_codata_z)
print(cor_codata_z_t, type = &quot;html&quot;)
</code></pre>

<!-- html table generated in R 3.0.2 by xtable 1.7-1 package -->

<!-- Mon Dec 30 17:59:18 2013 -->

<TABLE border=1>
<TR> <TH>  </TH> <TH> CO </TH> <TH> Hour </TH> <TH> Traffic </TH> <TH> Wind </TH>  </TR>
  <TR> <TD align="right"> CO </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.43 </TD> <TD align="right"> 0.96 </TD> <TD align="right"> 0.71 </TD> </TR>
  <TR> <TD align="right"> Hour </TD> <TD align="right"> 0.43 </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.43 </TD> <TD align="right"> 0.42 </TD> </TR>
  <TR> <TD align="right"> Traffic </TD> <TD align="right"> 0.96 </TD> <TD align="right"> 0.43 </TD> <TD align="right"> 1.00 </TD> <TD align="right"> 0.61 </TD> </TR>
  <TR> <TD align="right"> Wind </TD> <TD align="right"> 0.71 </TD> <TD align="right"> 0.42 </TD> <TD align="right"> 0.61 </TD> <TD align="right"> 1.00 </TD> </TR>
   </TABLE>

<pre><code class="r">scatterplotMatrix(codata_z, spread = F, pch = 19, main = &quot;Carbon Monoxide from a Freeway&quot;)
</code></pre>

<p>从散点图可以看出，CO和Traffic似乎有些线性关系，CO和Hour则有些类似于正弦曲线的关系，Hour和另外两个变量也有些周期形式的关系，CO和Wind的关系比较复杂,很难看出来是什么关系。</p>

<p>5.做全部变量回归</p>

<pre><code class="r">fit_all &lt;- lm(CO ~ ., data = codata_z)
summary(fit_all)
</code></pre>

<pre><code>## 
## Call:
## lm(formula = CO ~ ., data = codata_z)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.7503 -0.3328 -0.0902  0.2265  1.2511 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(&gt;|t|)    
## (Intercept)   4.5375     0.1040   43.62  &lt; 2e-16 ***
## Hour         -0.0402     0.1207   -0.33   0.7423    
## Traffic       1.8039     0.1385   13.03  3.1e-11 ***
## Wind          0.4157     0.1381    3.01   0.0069 ** 
## ---
## Signif. codes:  0 &#39;***&#39; 0.001 &#39;**&#39; 0.01 &#39;*&#39; 0.05 &#39;.&#39; 0.1 &#39; &#39; 1
## 
## Residual standard error: 0.51 on 20 degrees of freedom
## Multiple R-squared:  0.95,   Adjusted R-squared:  0.942 
## F-statistic:  126 on 3 and 20 DF,  p-value: 3.68e-13
</code></pre>

<p>从结果来看，常量、Traffic、Wind 通过了显著性T、F的显著性检验，R squared 的值是0.95，拟合效果还好。但是变量Hour没有通过显著性检验，似乎这个值应该去掉。</p>

<p>6.逐步回归，介于上一个步骤中，全部变量的回归有一些变量没有通过显著性检验，我们通过逐步回归来进行操作</p>

<pre><code class="r">fit_step &lt;- step(fit_all, direction = &quot;backward&quot;)
</code></pre>

<pre><code>## Start:  AIC=-29
## CO ~ Hour + Traffic + Wind
## 
##           Df Sum of Sq  RSS   AIC
## - Hour     1       0.0  5.2 -30.6
## &lt;none&gt;                  5.2 -28.7
## - Wind     1       2.4  7.5 -21.8
## - Traffic  1      44.1 49.3  23.3
## 
## Step:  AIC=-31
## CO ~ Traffic + Wind
## 
##           Df Sum of Sq  RSS   AIC
## &lt;none&gt;                  5.2 -30.6
## - Wind     1       2.4  7.6 -23.7
## - Traffic  1      46.1 51.3  22.2
</code></pre>

<pre><code class="r">summary(fit_step)
</code></pre>

<pre><code>## 
## Call:
## lm(formula = CO ~ Traffic + Wind, data = codata_z)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.7286 -0.3171 -0.0963  0.2241  1.2655 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(&gt;|t|)    
## (Intercept)    4.538      0.102   44.57  &lt; 2e-16 ***
## Traffic        1.793      0.132   13.62  6.8e-12 ***
## Wind           0.405      0.132    3.08   0.0057 ** 
## ---
## Signif. codes:  0 &#39;***&#39; 0.001 &#39;**&#39; 0.01 &#39;*&#39; 0.05 &#39;.&#39; 0.1 &#39; &#39; 1
## 
## Residual standard error: 0.5 on 21 degrees of freedom
## Multiple R-squared:  0.95,   Adjusted R-squared:  0.945 
## F-statistic:  197 on 2 and 21 DF,  p-value: 2.42e-14
</code></pre>

<p>逐步回归中根据AIC， 删除了Hour变量，我们看到Traffic、Wind变量都通过了显著性检验，且R squared 的值是0.95，拟合效果算是不错。接下来进行回归诊断，回归诊断主要包括残差检验、离群点检验等</p>

<p>7.回归诊断</p>

<pre><code class="r">par(mfrow = c(2, 2))
shapiro.test(fit_step$residual)
</code></pre>

<pre><code>## 
##  Shapiro-Wilk normality test
## 
## data:  fit_step$residual
## W = 0.92, p-value = 0.05601
</code></pre>

<pre><code class="r">plot(fit_step)
</code></pre>



<p>从对残差的检验上看，效果并不是很理想，图一表示残差散点图，理想情况应该是均匀分布，且我们看到第11号点和第12号点远离其他样本点。图二是残差的正态性检验图，如果残差符合正态分布，理想情况下应该在直线上，从图中我们很难得结论说残差符合正态分布。图三是标准化残差绝对值得开方的残差图，第11号标准化残差绝对值开方大于1.5，说明11号点在95%的置信区间之外。第四幅图是看强影响点，我们看到11号点是强影响点，说明11号值很有可能是拟合的离群值。解释最开始我们看的散点图，我们尝试另外一个的模型，即把Traffic、Wind的一二三次项都考虑进去，再包括谐波分析中的p=1,2项。这个模型明显偏大，我们可以用逐步回归来根据AIC选择变量。</p>

<p>8.考虑把一二三次项都考虑进去</p>

<pre><code class="r">attach(codata_z)
cor_codata_z_1 &lt;- cor(cbind(CO, Traffic, Tsq = Traffic^2, Tcub = Traffic^3, 
    Hour, Hsq = Hour^2, Hcub = Hour^3, Wind, Wsq = Wind^2, Wcub = Wind^3))
cor_codata_z_1
</code></pre>

<pre><code>##            CO Traffic   Tsq  Tcub     Hour      Hsq     Hcub  Wind    Wsq
## CO       1.00    0.96 -0.28  0.89  4.3e-01 -7.4e-01  3.1e-01  0.71  0.337
## Traffic  0.96    1.00 -0.18  0.95  4.3e-01 -6.8e-01  3.2e-01  0.61  0.331
## Tsq     -0.28   -0.18  1.00 -0.21 -5.0e-01  1.9e-01 -5.1e-01 -0.22  0.114
## Tcub     0.89    0.95 -0.21  1.00  5.5e-01 -4.9e-01  4.4e-01  0.54  0.288
## Hour     0.43    0.43 -0.50  0.55  1.0e+00  1.1e-20  9.2e-01  0.42  0.061
## Hsq     -0.74   -0.68  0.19 -0.49  1.1e-20  1.0e+00  1.2e-20 -0.61 -0.362
## Hcub     0.31    0.32 -0.51  0.44  9.2e-01  1.2e-20  1.0e+00  0.17 -0.104
## Wind     0.71    0.61 -0.22  0.54  4.2e-01 -6.1e-01  1.7e-01  1.00  0.715
## Wsq      0.34    0.33  0.11  0.29  6.1e-02 -3.6e-01 -1.0e-01  0.72  1.000
## Wcub     0.58    0.52 -0.13  0.45  3.4e-01 -5.1e-01  1.2e-01  0.92  0.892
##          Wcub
## CO       0.58
## Traffic  0.52
## Tsq     -0.13
## Tcub     0.45
## Hour     0.34
## Hsq     -0.51
## Hcub     0.12
## Wind     0.92
## Wsq      0.89
## Wcub     1.00
</code></pre>

<pre><code class="r">fit_all_1 &lt;- lm(CO ~ Traffic + Wind + I(Wind^2) + I(Wind^3) + sin((2 * pi/24) * 
    Hour) + cos((2 * pi/24) * Hour) + sin((4 * pi) * Hour) + cos((4 * pi/24) * 
    Hour))
summary(fit_all_1)
</code></pre>

<pre><code>## 
## Call:
## lm(formula = CO ~ Traffic + Wind + I(Wind^2) + I(Wind^3) + sin((2 * 
##     pi/24) * Hour) + cos((2 * pi/24) * Hour) + sin((4 * pi) * 
##     Hour) + cos((4 * pi/24) * Hour))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.3944 -0.0799 -0.0281  0.1019  0.3300 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(&gt;|t|)    
## (Intercept)              690.5025    80.2829    8.60  3.5e-07 ***
## Traffic                    1.6392     0.0694   23.62  2.8e-13 ***
## Wind                       0.3847     0.1273    3.02   0.0086 ** 
## I(Wind^2)                 -0.8772     0.1601   -5.48  6.4e-05 ***
## I(Wind^3)                  0.3023     0.1154    2.62   0.0193 *  
## sin((2 * pi/24) * Hour)   -0.3715     0.2703   -1.37   0.1895    
## cos((2 * pi/24) * Hour) -928.6201   108.6744   -8.54  3.8e-07 ***
## sin((4 * pi) * Hour)      -0.0212     0.0545   -0.39   0.7030    
## cos((4 * pi/24) * Hour)  243.9350    28.4663    8.57  3.7e-07 ***
## ---
## Signif. codes:  0 &#39;***&#39; 0.001 &#39;**&#39; 0.01 &#39;*&#39; 0.05 &#39;.&#39; 0.1 &#39; &#39; 1
## 
## Residual standard error: 0.19 on 15 degrees of freedom
## Multiple R-squared:  0.995,  Adjusted R-squared:  0.992 
## F-statistic:  366 on 8 and 15 DF,  p-value: 8.86e-16
</code></pre>

<pre><code class="r">fit_step_1 &lt;- step(fit_all_1)
</code></pre>

<pre><code>## Start:  AIC=-74
## CO ~ Traffic + Wind + I(Wind^2) + I(Wind^3) + sin((2 * pi/24) * 
##     Hour) + cos((2 * pi/24) * Hour) + sin((4 * pi) * Hour) + 
##     cos((4 * pi/24) * Hour)
## 
##                           Df Sum of Sq   RSS   AIC
## - sin((4 * pi) * Hour)     1      0.01  0.53 -75.4
## &lt;none&gt;                                  0.53 -73.6
## - sin((2 * pi/24) * Hour)  1      0.07  0.59 -72.8
## - I(Wind^3)                1      0.24  0.77 -66.6
## - Wind                     1      0.32  0.85 -64.2
## - I(Wind^2)                1      1.05  1.58 -49.3
## - cos((2 * pi/24) * Hour)  1      2.57  3.09 -33.2
## - cos((4 * pi/24) * Hour)  1      2.58  3.11 -33.1
## - Traffic                  1     19.60 20.12  11.8
## 
## Step:  AIC=-75
## CO ~ Traffic + Wind + I(Wind^2) + I(Wind^3) + sin((2 * pi/24) * 
##     Hour) + cos((2 * pi/24) * Hour) + cos((4 * pi/24) * Hour)
## 
##                           Df Sum of Sq   RSS   AIC
## &lt;none&gt;                                  0.53 -75.4
## - sin((2 * pi/24) * Hour)  1      0.07  0.61 -74.3
## - I(Wind^3)                1      0.24  0.77 -68.6
## - Wind                     1      0.33  0.87 -65.7
## - I(Wind^2)                1      1.05  1.58 -51.3
## - cos((2 * pi/24) * Hour)  1      2.56  3.09 -35.2
## - cos((4 * pi/24) * Hour)  1      2.57  3.11 -35.1
## - Traffic                  1     20.25 20.78  10.5
</code></pre>

<pre><code class="r">summary(fit_step_1)
</code></pre>

<pre><code>## 
## Call:
## lm(formula = CO ~ Traffic + Wind + I(Wind^2) + I(Wind^3) + sin((2 * 
##     pi/24) * Hour) + cos((2 * pi/24) * Hour) + cos((4 * pi/24) * 
##     Hour))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.4024 -0.0857 -0.0224  0.0887  0.3146 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(&gt;|t|)    
## (Intercept)              689.2429    78.0603    8.83  1.5e-07 ***
## Traffic                    1.6437     0.0666   24.67  3.7e-14 ***
## Wind                       0.3899     0.1232    3.16    0.006 ** 
## I(Wind^2)                 -0.8749     0.1557   -5.62  3.9e-05 ***
## I(Wind^3)                  0.2992     0.1120    2.67    0.017 *  
## sin((2 * pi/24) * Hour)   -0.3872     0.2601   -1.49    0.156    
## cos((2 * pi/24) * Hour) -926.8746   105.6617   -8.77  1.6e-07 ***
## cos((4 * pi/24) * Hour)  243.4431    27.6734    8.80  1.6e-07 ***
## ---
## Signif. codes:  0 &#39;***&#39; 0.001 &#39;**&#39; 0.01 &#39;*&#39; 0.05 &#39;.&#39; 0.1 &#39; &#39; 1
## 
## Residual standard error: 0.18 on 16 degrees of freedom
## Multiple R-squared:  0.995,  Adjusted R-squared:  0.993 
## F-statistic:  442 on 7 and 16 DF,  p-value: &lt;2e-16
</code></pre>

<p>做完逐步回归之后发现,sin((2 * pi/24) * Hour)没有通过显著性检验，而I(Wind<sup>3)</sup> 是弱显著，去掉这两项。</p>

<pre><code class="r">fit &lt;- lm(CO ~ Traffic + Wind + I(Wind^2) + cos((2 * pi/24) * Hour) + cos((4 * 
    pi/24) * Hour))
summary(fit)
</code></pre>

<pre><code>## 
## Call:
## lm(formula = CO ~ Traffic + Wind + I(Wind^2) + cos((2 * pi/24) * 
##     Hour) + cos((4 * pi/24) * Hour))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.4903 -0.1033  0.0352  0.1160  0.2546 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(&gt;|t|)    
## (Intercept)              688.2060    87.8411    7.83  3.3e-07 ***
## Traffic                    1.5997     0.0638   25.06  1.9e-15 ***
## Wind                       0.6058     0.0784    7.73  4.0e-07 ***
## I(Wind^2)                 -0.5011     0.0775   -6.47  4.4e-06 ***
## cos((2 * pi/24) * Hour) -926.3701   118.8278   -7.80  3.5e-07 ***
## cos((4 * pi/24) * Hour)  243.8539    31.0593    7.85  3.2e-07 ***
## ---
## Signif. codes:  0 &#39;***&#39; 0.001 &#39;**&#39; 0.01 &#39;*&#39; 0.05 &#39;.&#39; 0.1 &#39; &#39; 1
## 
## Residual standard error: 0.21 on 18 degrees of freedom
## Multiple R-squared:  0.993,  Adjusted R-squared:  0.99 
## F-statistic:  478 on 5 and 18 DF,  p-value: &lt;2e-16
</code></pre>

<p>通过显著性检验同时R-squared 为0.99 表示拟合效果很好。</p>

<p>9.再进行归回诊断：</p>

<pre><code class="r">par(mfrow = c(2, 2))
shapiro.test(fit$residual)
</code></pre>

<pre><code>## 
##  Shapiro-Wilk normality test
## 
## data:  fit$residual
## W = 0.94, p-value = 0.1811
</code></pre>

<pre><code class="r">plot(fit)
</code></pre>


<p>发现残差通过了正态检验的假设。</p>

<p>10.交叉验证</p>

<pre><code class="r">shrinkage &lt;- function(fit, k = 10) {
    require(&quot;bootstrap&quot;)
    thera.fit &lt;- function(x, y) {
        lsfit(x, y)
    }
    thera.predict &lt;- function(fit, x) {
        cbind(1, x) %*% fit$coef
    }
    x &lt;- fit$model[, 2:ncol(fit$model)]
    y &lt;- fit$model[, 1]
    results &lt;- crossval(x, y, thera.fit, thera.predict, ngroup = k)
    r2 &lt;- cor(y, fit$fitted.values)^2
    r2cv &lt;- cor(y, results$cv.fit)^2
    cat(&quot;Original R-square=&quot;, r2, &quot;\n&quot;)
    cat(k, &quot;Fold Cross-Validated R-square =&quot;, r2cv, &quot;\n&quot;)
    cat(&quot;Change=&quot;, r2 - r2cv, &quot;\n&quot;)
}
shrinkage(fit)
</code></pre>

<pre><code>## Loading required package: bootstrap
</code></pre>

<pre><code>## Original R-square= 0.99 
## 10 Fold Cross-Validated R-square = 0.99 
## Change= 0.0062
</code></pre>

<p>交叉验证后发现R-squared 为0.99 说明模型的泛化很好。最后我们得出最后的拟合方程为：CO=688+1.6Traffic+0.68Wind-0.50(Wind^ 2)-926cos((2pi/24)Hour)+244cos((4pi/24)Hour)</p>

</body>

</html>

