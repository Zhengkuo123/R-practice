前言
有很多方法能够将Table 1直接制作成为HTML格式，更方便在Word中进行修改，但这涉及到更多的编程知识和更复杂的代码；
因此，本笔记仅介绍将Table 1制作成为csv格式的方法，后续可能需要多一点修饰，但该方法简单且易于理解。

永远牢记：完成比完美更重要！
认识Table 1
Table 1中最核心的内容之一即为按暴露因素分层之后研究人群基线特征的描述性统计，另一个核心内容是差异比较，即为算出p值

核心代码
环境清空，加载包，导入数据，观察数据
```R 
rm(list = ls())
library(tableone) ##最核心的包 
aa <- read.csv("data.csv",sep = ",",header = T)
head(aa)
str(aa)
```

清洗数据
因子转换，加入标签
这一步可以使用循环完成，但牢记一个原则：完成比完美更重要！
```R 
aa$mucin <- 
    factor(aa$mucin,
           levels = c(1,0),##对应原数据框中的内容
           labels = c("positive","negative"))##生成的新的内容（标签）
aa$Gender<- 
    factor(aa$Gender,
           levels = c(1,2),
           labels = c("male","female"))
aa$age65<- 
    factor(aa$age65,
           levels = c(1,2),
           labels = c("≤ 65","＞65"))
aa$consi<- 
    factor(aa$consi,
           levels = c(1,0),
           labels = c("yes","no"))
aa$X1st_CEA<- 
      factor(aa$X1st_CEA,
             levels = c(2,1),
             labels = c("normal","up"))
aa$X1st_CA199<- 
      factor(aa$X1st_CA199,
             levels = c(2,1),
             labels = c("normal","up"))
aa$X2rd_CEA<- 
      factor(aa$X2rd_CEA,
             levels = c(0,1),
             labels = c("normal","up"))
aa$X2rd_CA199<- 
      factor(aa$X2rd_CA199,
             levels = c(0,1),
             labels = c("normal","up"))
aa$Low.5cm<- 
    factor(aa$Low.5cm,
           levels = c(1,2),
           labels = c("low","middle"))
aa$Clinical.stage<- 
    factor(aa$Clinical.stage,
           levels = c(2,3),
           labels = c("II","III"))
aa$TRG<- 
    factor(aa$TRG,
           levels = c(0,1,2,3),
           labels = c("0","1","2","3"))
aa$ypT<- 
    factor(aa$ypT,
           levels = c(0,1,2,3,4),
           labels = c("0","1","2","3","4"))
aa$ypN<- 
    factor(aa$ypN,
           levels = c(0,1,2),
           labels = c("0","1","2"))
aa$ypstage<- 
    factor(aa$ypstage,
           levels = c(0,1,2,3),
           labels = c("0","1","2","3"))
aa$death <- 
    factor(aa$death,
           levels = c(0,1),
           labels = c("alive","death"))
aa$recur <- 
    factor(aa$recur,
           levels = c(0,1),
           labels = c("no-recur","recur"))
label(aa$age1) <- "Age" 
```

```R 
检查计量资料是否为正态分布
shapiro.test(aa$age1)#p＞0.05才符合正态分布
shapiro.test(aa$DFS)
shapiro.test(aa$OS)
```

告诉R哪些变量希望在Table 1中出现
即为指定函数的参数
```R 
## 参数1：所有变量
myVars <- c(names(aa)[1:14])
## 参数2：哪些是计数资料
catVars <- myVars[c(1:2,4:14)]
## 参数3：指定非正态分布的计量资料，如果没有则无需指定
nonvar <- c("age1","OS","DFS")
## 参数5：指定需要Fisher精确检验的计数资料，如果没有则无需指定
exactvars <- c("a", "b")
```

构建Table函数
构建table函数，则需要指定分层参数，在本例中，即为mucinstrata="mucin"

```R 
table <- CreateTableOne(vars = myVars,##参数1
                          factorVars = catVars,##参数2
                          strata = "mucin",##参数4
                          data = aa##数据框
                       )
```
输出结果
这一步可以添加许多细节
```R 
table1<- print(table,##这里包含了参数1，2，4
               #nonnormal = nonvar,## 这里包含的是参数3，指定非正态分布的计量资料
               #exact = exactvars,## 这里包含了参数5，制定使用Fisher精确检验的计数资料
               catDigits = 2,##计数资料小数位数为2位
               contDigits = 3,##计量资料小数位数为3位
               pDigits = 3, ##p值小数位数为3位
               showAllLevels=TRUE,##显示所有变量 
               quote = FALSE, ##不显示引号
               noSpaces = TRUE, ##删除用于对齐的空格
               printToggle = FALSE ##不展示输出结果
              ) 
write.csv(table1,file = "table1.csv")
```
total一列的统计

只需要在CreateTableOne()中添加一个新的参数addOverall=TRUE(参数6)即可
```R 
table <- CreateTableOne(vars = myVars,##参数1
                        factorVars = catVars,##参数2
                        strata = "mucin",##参数4
                        addOverall = TRUE,##参数6
                        data = aa##数据框
                       )

##print()不变
table1<- print(table,##这里包含了参数1，2，4
               #nonnormal = nonvar,## 这里包含的是参数3，指定非正态分布的计量资料
               #exact = exactvars,## 这里包含了参数5，制定使用Fisher精确检验的计数资料
               catDigits = 2,##计数资料小数位数为2位
               contDigits = 3,##计量资料小数位数为3位
               pDigits = 3, ##p值小数位数为3位
               showAllLevels=TRUE,##显示所有变量 
               quote = FALSE, ##不显示引号
               noSpaces = TRUE, ##删除用于对齐的空格
               printToggle = FALSE ##不展示输出结果
              ) 
write.csv(table1,file = "table1.csv")
```R 

核心函数和的深入学习
CreateTableOne()
没有介绍等级资料的比较方法
```R 
table <- CreateTableOne(
  vars,#指定哪些变量是Table 1需要汇总的变量
  strata,#指定进行分类的变量，不写则仅列出Overall列
  data,#变量的数据集名称
  factorVars,#指定分类变量，应是vars中的变量
  includeNA = F,#为T时则将缺失值作为因子处理，仅对分类变量有效
  test = T, #默认为T，当有2个或多个分组时，自动进行组间比较
  testApprox = chisq.test,#默认卡方检验
  argsApprox = list(correct = TRUE),# 进行连续校正的chisq.test
  testExact = fisher.test,#进行fisher精确检验
  argsExact = list(workspace = 2 * 10^5),# 指定fisher.test分配的内存空间
  testNormal = oneway.test, 
  #连续变量为正态分布进行的检验，默认为oneway.test，两组时相当于t检验
  argsNormal = list(var.equal = TRUE),
  testNonNormal = kruskal.test,#默认为Kruskal-Wallis秩和检验
  argsNonNormal = list(NULL),#传递给testNonNormal中指定的函数的参数的名列表
  smd = TRUE,#如果为TRUE（如默认值）并且有两个以上的组，则将计算所有成对比较的标准化均值差
  addOverall = FALSE#仅在分组时使用）将整个列添加到表中。Smd和p值计算仅使用分层的列阵进行
)
print()
table1<- print(
  x, #CreateTableOne()的 <- 前的名字(x=table)
  catDigits = 1, #连续变量小数位1位
  contDigits = 2,#分类变量保留2位
  pDigits = 3,#p值保留3位
  quote = FALSE,#默认值为FALSE。如果为TRUE#则包括行名和列名在内的所有内容都用引号引起来，以便您可以轻松地将其复制到Excel。
  missing = FALSE,#是否显示丢失的数据信息
  explain = TRUE,#显示百分比时是否在变量名称中添加解释，即（％）添加到变量名称中。
  printToggle = TRUE,#如果为FALSE，则不输出
  test = TRUE,#是否显示p值。默认为TRUE。
  smd = FALSE,#是否显示标准化均值差异。默认为FALSE。如果存在多个对比，则显示所有可能的标准化均值差的平均值。
  noSpaces = FALSE,#是否删除为对齐而添加的空格。
  padColnames = FALSE,#是否用空格填充列名以居中对齐。默认值为FALSE。如果noSpaces = TRUE，则不进行
  varLabels = FALSE,#是否用从labelled :: var_label（）函数获得的变量标签替换变量名。
  format = c("fp", "f", "p", "pf")[1],#默认值为“ fp”频率（百分比）。您也可以选择仅“ f”频率，“仅p”百分比和“ pf”百分比（频率）。
  showAllLevels = FALSE,#是否显示所有级别。
  cramVars = NULL,#字符向量，用于指定两个级别的分类变量，对于这两个级别的变量，应在一行中显示两个级别。
  dropEqual = FALSE,#是否删除“ =第二级名称”描述，指示为两级分类变量显示哪个级别。
  exact = NULL,#字符向量，用于指定p值应为精确测试值的变量。
  nonnormal = NULL,#字符向量，用于指定p值应为非参数检验的变量的变量。
  minMax = FALSE#对于非正态变量，是否使用[min，max]而不是[p25，p75]。默认值为FALSE。
)
```
