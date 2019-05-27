先从[github](https://raw.githubusercontent.com/Bioconductor/BiocWorkshops/master/100_Morgan_RBiocForAll/ALL-phenoData.csv)中复制数据

自己在本地创建一个csv，放到文件夹中

> 要复制不带逗号的数据进行粘贴，还要注意删除空白行

当你文件不是在project文件夹中创建，而是需要在别的路径里找的时候，可以使用一下语法:

```r
fname<- file.choose() #弹出文件选择框，将路径存储到fname中
fname
# 利用file.exists()检查文件是否存在
file.exists(fname)
#读入文件
pdata <- read.csv(fname)

# 然后还是标准的检查
dim(pdata)
head(pdata)
summary(pdata)
tail(pdata)
```

导入数据时指定每一列数据的类型：
通过指定 `colClasses` 参数
```r
pdata<-read.csv(fname,colClasses = c("character","factor","integer","factor"))
```
### 关于子集(subsetting)的操作
#### 1.提取
```r
pdata[1:5, c("sex","mol.biol")] #提取1-5行，sex和mol.bilo列
pdata[1:5,] #也可以省略其中一个，代表全部都要
pdata$age #等同于
pdata[[age]]
```

##### other
> table的意思就是 tabular summary

```r
table(pdata$mol.biol) #table函数用于统计该向量中，每中因子的数量；
table(is.na(pdata$age)) # 用于统计空白值的数量，is.na是判断语句，返回布尔值，最后table统计布尔值就好了
levels(pdata$sex) # 显示性别的因子名字和排列顺序，可以手动修改
```
#### 2.对比操作(返回布尔值）)
```r
table(pdata$sex == "F")  #性别中每一个向量都跟 F 作对比,统计输出的布尔值
```

#### 3.判断特定的列中是否含有某些特定向量（返回布尔值）

> `A %in% B` 表示向量A中的元素是否在向量B中出现

```r
pdata$mol.biol %in% c("BCR/ABL", "NEG")
```

#### 4.按两个条件筛选：（返回布尔值）
```r
get1<- (pdata$sex == "F")&(pdata$age >50)
#选出性别为女性并且年纪大于50的,同时满足才返回True
```

#### 5.用subset函数取子集 (得到数据集而非布尔值）)
##### `subset`函数

`subset(dataframe,choices)`
```r
subset1<-subset(pdata,sex =="F" & age >50)
subset1

# 同样也可以筛选出含有BRC/ABL或NEG的子集
subset2<- subset(pdata,mol.biol%in%c("BCR/ABL","NEG"))
```

```r
bcrabl <- subset(pdata, mol.biol %in% c("BCR/ABL", "NEG"))
dim( bcrabl )

# 然后table看一些(table的意思就是 tabular summary)
table(bcrabl$mol.biol)

# 因为我们只选了BCR/ABL和NEG两项，因此其他的都是空白，想要只看选出来的这两项信息，可以
factor(bcrabl$mol.biol)

# 更新一下bcrabl的mol.biol列（看前后Levels的变化）
# (之前)
bcrabl$mol.biol

# (之后)
bcrabl$mol.biol <- factor(bcrabl$mol.biol)
bcrabl$mol.biol

```

#### `formula`公式的使用
这个在R中是一个非常常用的概念，`formular` 的表达式是 `varA ~ varB` 左边是**因变量**，右边是**自变量**，例如`age ~ mol.biol`就表示年龄随着细胞学表征的变化。
> 自变量需要是**因子**，这样在x轴上才能分组

从箱线图中就可以清楚看出，BCR/ABL 的人平均年龄比NEG偏高，可以再结合`t检验`进行验证:
```r
t.test(age ~ mol.biol, bcrabl)
#用ggplot2作图
library(ggplot2)
ggplot(bcrabl, aes(x = mol.biol, y = age)) + geom_boxplot()
```

## 关于bioconductor

> Bioconductor是一个包含了150多个统计和组学**数据包的集合**，最初为芯片数据开发，现在支持 bulk and single-cell RNA-seq, ChIP seq, copy number analysis, microarray methylation and classic expression analysis, flow cytometry等等

### Bioconductor官网

在[官网](https://bioconductor.org/packages)中你可以查询到不同package的详细内容和安装方法

### 如何通过 biocondunctor 安装包

#### 首先安装 BiocManager
```r
if (!"BiocManager" %in% rownames(intalled.packages()))
    install.packages("BiocManager", repos="https://cran.r-project.org")
# 从 CRAN 中下载安装
```

#### 安装相应的包


语句`BiocManager::install(c("package1","package2",...,...))`
```r
BiocManager::install(c("rtracklayer", "GenomicRanges"))
# 安装"rtracklayer", "GenomicRanges"这两个包
# R会自动安装这两个包所依赖的包，超方便
# 安装DESeq包，后续会用到很重要的一个分析包
BiocManager::install("DESeq2")
```

`bioconductor` 中有一个包很重要 `vignettes`,这个包的功能可以用来定义**某一个包的用法**

####  学习与基因组坐标相关的用法
