## R package: votesys
## 花式计票方法，用于选举、决策等
## by：Wu Jiang （吴江，首都师范大学）
## 微信号：theblackriver
##
## CRAN: http://mirrors.ustc.edu.cn/CRAN/web/packages/votesys/index.html
## 英文手册更加详细：http://mirrors.ustc.edu.cn/CRAN/web/packages/votesys/votesys.pdf
***

### 人们常常从三个方面批评投票制度（不论是用来决定人选的投票还是用来选择政策的投票）：一是它不能表达人们的真实偏好（比如，人们批评相对多数制中的胜者事实上只符合少数人的偏好）；二是投票结果会出现阿罗不可能性定理所描述的情况；三是选举容易被操纵。

### 为了解决这些问题，人们发明了多种合理的或奇葩的方法来计票。

### 以前计算机还不够快的时候，实际运用这些方法是个问题。但是现在计算机跑得快了，只要有了代码，分秒间可以得到多种计票方法的结果。本R包就是用来做这个的。当然，这些算法也已经出现在其他程序、平台甚至网页上了；但本R包中的函数为计算提供了一定灵活性，并且输出结果也包含更多细节信息供后续分析。

## 一、本包特点

### （1）多种计票方法

除了常用方法外，还包括像ranked pairs、Schulze、Kemeny-Young这种有些复杂的计票方法。

### （2）检测并灵活处理错误选票、有重复值的选票、有缺失值的选票；有助于方便地比较多种计票方法的结果。

错误选票是指写了不该写的人名或数值、长度不对的选票。

重复值是指一个人名写了好几次，或把同样的分数给了两个人。

缺失值是指未对一些候选人排序或打分；很多选举都是允许选民这样的，但主要看如何处理。

### （3）节省数据清理的时间。

使用本包中的函数计票地，第一步是把不同格式的选票转化成一个"vote"对象，这个对象可以作为输入放到各种用来计票的函数中，以便节省数据清理的时间。

## 二、本包中的函数

### 计票函数

- plurality_method    多数制

- approval_method    赞成投票制（在选票上可以写好几个名字，看谁的名字出现次数多）

- borda_method    波达计数法，也就是打分法，含两种方法

- dowdall_method    Dowdall法，也是一种计数法

- irv_method    排序复选制，instant-runoff voting (IRV)

- cdc_simple    普通孔多塞

- cdc_copeland    Copeland孔多塞，含两种变体

- cdc_dodgson    Dodgson法，含Tideman和DQ两种

- cdc_minmax    最小最大法，含三种变体

- cdc_schulze    Schulze法，也就是解一个widest path问题

- cdc_rankedpairs    ranked-paris法

- cdc_kemenyyoung    Kemeny-Young方法，候选人人数上限为8人

- star_rating    星级评分法，也就是让用户用1星到5星给电影打分，然后对电影进行排序的方法

### 数据处理函数

- list2ballot    把列表或字符转化成列表格式的选票

- check_dup_wrong    找出有重复值的、错误的、全是NA的选票

- create_vote    生成"vote"对象，同时生成孔多塞矩阵

-as_complete  把不完整选票补充成完整选票

## 三、举例

### list2ballot

```R
# 4种有着独特排序的选票，投这4种票的人数为1、2、3、0
# 用list2ballot生成原始选票
unique_ballot <- list(
    c("A", "B", "C"), c("F", "A", "B"),
    c("E", "D", "C", "B", "F", "A"), c("x","x", "A")
)
r <- c(1, 2, 3, 0)
y <- list2ballot(unique_ballot, r)
#
# 用"string=..."格式的字符生成选票
# 每一个字符的第一部分必须是数字，即选票数，后边是排序
# 数字和名字之间，以及名字和名字之间，要用空格或标点隔开，但是随便加什么其他无用的东西也无所谓，会自动处理掉
# 比如下边这个例子，写得很随意，加了各种标点，但是没问题
unique_ballot <- c(
	"2, Bob, Mike Jane", 	"3: barack_obama;;Bob>Jane", 
	"0 Smith   Jane", "  1 Mike???!!!"
)
y <- list2ballot(string = unique_ballot)
```

### check_dup_wrong

这个函数并不直接处理数据，而只是告诉你哪些选项有错误、有重复值，或全是NA。

```R
raw=list(
    c('a', 'e', 'c', 'd', 'b'), 
    c('b', 'a', 'e'), 
    c('c', 'd', 'b'), 
    c('d', 'a', 'b'), 
    c('a', 'a', 'b', 'b', 'b'), 
    c(NA, NA, NA, NA), 
    v7=NULL, 
    v8=c('a', NA, NA, NA, NA, NA, NA), 
    v9=rep(" ", 3)
)
y=check_dup_wrong(raw, xtype=3, candidate=letters[1: 5])
y$row_with_dup
# [1] 5
y$row_with_wrong
# [1] 8
y$row_all_na
# [1] 6 7 9
```

### create_vote

用create_vote函数生成"vote"对象。原始数据可以是（1）列表、（2）用列表示偏好（第1列表示最喜欢，第2列表示第2喜欢，最后一列表示最讨厌，单元格里是相应的候选人的名字或编号）的数据框或矩阵，还可以（3）本身就是打分矩阵。

create_vote生成的对象中包含经过处理的选票。

```R
# 以下原始数据raw是一个包含人名的矩阵，其第1列表示最喜欢，第2列表示第2喜欢……
raw <- c(
    rep(c('m', 'n', 'c', 'k'), 42), 
    rep(c('n', 'c', 'k', 'm'), 26), 
    rep(c('c', 'k', 'n', 'm'), 15), 
    rep(c('k', 'c', 'n', 'm'), 17)
) 
raw <- matrix(raw, ncol = 4, byrow = TRUE)
vote <- create_vote(raw, xtype = 2, candidate = c('m', 'n', 'k', 'c'))
vote$ballot # 提取转化成分数矩阵的选票
```

as_complete用于将包含缺失值的分数矩阵补充完整，补充方法有三种

```R
r1=c(1, 2, 3, 4, NA, NA)
r2=c(2, 2, 4, NA, NA, NA)
r3=c(1, 4, NA, NA, 2, 3)
r123=rbind(r1, r2, r3)
colnames(r123)=LETTERS[1: 6]
as_complete(r123, method="len", n=7)
as_complete(r123, method="max", plus=1)
as_complete(r123, method="valid", plus=1)
```

### 计票

先说孔多塞家族的吧。所有以cdc开头的都是孔多塞家族的方法。相应的函数的特点是，它们不但接受"vote"对象，而且还接受用户直接输入的孔多塞矩阵（不需要原始选票！），以及其他孔多塞计算方法的结果。

```R
raw <- list2ballot(
    x = list(
        c('a', 'c', 'b', 'e', 'd'), 
        c('a', 'd', 'e', 'c', 'b'), 
        c('b', 'e', 'd', 'a', 'c'), 
        c('c', 'a', 'b', 'e', 'd'), 
        c('c', 'a', 'e', 'b', 'd'), 
        c('c', 'b', 'a', 'd', 'e'), 
        c('d', 'c', 'e', 'b', 'a'), 
        c('e', 'b', 'a', 'd', 'c')
    ), 
    n = c(5, 5, 8, 3, 7, 2, 7, 8)
)
vote <- create_vote(raw, xtype = 3, candidate = c('a', 'b', 'c', 'd', 'e'))
win1 <- cdc_simple(vote) # 用一般孔多塞法没有胜者
win2 <- cdc_schulze(vote) # 用Schulze法，胜者是e
win2 <- cdc_schulze(win1) # 用win1这个condorcet对象当输入
```

Kemeny-Young方法

```R
# 可以直接拿孔多塞矩阵当输入，这样一来我们随便弄个孔多塞矩阵放里边就行了
# Kemeny-Young的候选人人数上限为8人，应该够用了吧
m <- matrix(c(0, 58, 58, 58, 42, 0, 32, 32, 42, 68, 0, 17, 42, 68, 83, 0), nr = 4)
colnames(m) <- c('m', 'n', 'c', 'k')
rownames(m) <- c('m', 'n', 'c', 'k')
y <- cdc_kemenyyoung(m, keep_all_link = TRUE) # n是胜者
```

Dodgon法的不同计分方法会得到不同的结果

```R
raw <- list2ballot(
    x = list(
        c('A', 'B', 'C', 'D', 'E', 'F'), 
        c('F', 'A', 'B', 'C', 'D', 'E'),
        c('E', 'D', 'C', 'B', 'F', 'A'),
        c('B', 'A', 'C', 'D', 'E', 'F'),
        c('F', 'E', 'D', 'C', 'B', 'A'),
        c('F', 'B', 'A', 'C', 'D', 'E'),
        c('E', 'D', 'C', 'A', 'F', 'B'),
        c('E', 'B', 'A', 'C', 'D', 'F'),
        c('F', 'D', 'C', 'A', 'E', 'B'),
        c('D', 'B', 'A', 'C', 'E', 'F'),
        c('F', 'E', 'C', 'A', 'D', 'B')
    ), 
    n = c(19, 12, 12, 9, 9, 10, 10 , 10 , 10, 10, 10)
)
vote <- create_vote(raw, xtype = 3, candidate = c('A', 'B', 'C', 'D', 'E', 'F'))
win1 <- cdc_simple(vote) # 一般孔多塞法，没有胜者
win2 <- cdc_dodgson(vote, dq_t = "dq") # DQ法，A是胜者
win3 <- cdc_dodgson(win2, dq_t = "t") # Tideman法，B是胜者
```

波达计数法

```R
raw <- c(
    rep(c('m', 'n', 'c', 'k'), 42), 
    rep(c('n', 'c', 'k', 'm'), 26), 
    rep(c('c', 'k', 'n', 'm'), 15), 
    rep(c('k', 'c', 'n', 'm'), 17)
) 
raw <- matrix(raw, ncol = 4, byrow = TRUE)
vote <- create_vote(raw, xtype = 2, candidate = c('m', 'n', 'c', 'k'))
y <- plurality_method(vote) # 多数制法，胜者是m
y <- borda_method(vote) # 波达法，胜者是n
```

IRV法

```R
raw <- c(
   rep(c('m', 'n', 'c', 'k'), 42), rep(c('n', 'c', 'k', 'm'), 26), 
   rep(c('c', 'k', 'n', 'm'), 15), rep(c('k', 'c', 'n', 'm'), 17)
) 
raw <- matrix(raw, ncol = 4, byrow = TRUE)
vote <- create_vote(raw, xtype = 2, candidate = c('m', 'n', 'k', 'c'))
y <- irv_method(vote) # 用IRV法，胜者是k
y <- cdc_simple(vote) # 用一般孔多塞法，胜者是n
```

# 祝大家投票愉快！

