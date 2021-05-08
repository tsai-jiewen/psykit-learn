---
title: ConQuest計分檔的分割函數
subtitle: ★本篇非常實用
date: 2021-03-26T13:13:33.949Z
draft: false
featured: false
author:
  - Jiewen Tsai
tags:
  - R
  - ConQuest

---

## 讀取計分檔。
記得用 `read_table()`，若用 `read_csv()` 因為是逗點切割，所以會全部分成一個欄位。

```r=
tds <- read_table('TDS.txt', col_names = FALSE)
```
ConQuest的計分檔（.txt）用R讀取後長這樣子：

```r
> tds
# A tibble: 413 x 3
      X1    X2 X3                    
   <dbl> <dbl> <chr>                 
 1     1     1 0122122211121231211113
 2     2     1 1111112212311221220121
 3     3     1 3233122012332333131001
 4     4     1 2212123221110332332212
 5     5     1 3320231030333322232332
 6     6     1 0232221222020232223222
 7     7     1 1111133223131223230012
 8     8     1 2223222222332322111221
 9     9     1 2131133231331132331111
10    10     1 0220122222330322123022
# … with 403 more rows
```

其中，X3欄位是分數，但是只有1個欄位。要切分成22個欄位（tidy）才能用`eRm`、`TAM`等pkg做後續處理。


另解：
當 CQ 的資料檔沒有空白格分開時，用 `read_table()` 讀取會變成數值，而不是 chr。
所以可以改用 `read_csv()` 指定欄位格式 `col_types = 'c'`。
```=
ch114_txt <- read_csv('CH11-4.txt', col_types = 'c', col_names = FALSE)
```

## 欄位切割函數

但一直沒找到可以做這個切分工作的函數。所以就自己寫一個吧！

```r=
item_split <- function(dat, n) {
  require(tidyverse)
  require(stringr)
  l <- list()
  for (i in 1:n) {
    l[[i]] <- dat %>% str_sub(i, i) %>% as.numeric()
  }
  i = i + 1
  
  df <- matrix(unlist(l), nrow = length(l), byrow = T) %>% t() %>% as_tibble()
  df
}

```
參數：
- dat：原資料欄位
- n：切分後欄位數 

概念上就是用 `stringr::str_sub()` 函數，直向抽取數值。並存到 list 裡面。
再轉存成tibble型態。

---

## 測試

測試檔：台灣憂鬱量表（TDS），共 413 個觀察值，22 個題目。
用自編的 `item_split()` 函數切割檔案 `tds$X3`，存成 A。

```r
> A <- item_split(tds$X3, 22) 
> A
# A tibble: 413 x 22
      V1    V2    V3    V4    V5    V6    V7    V8    V9   V10   V11   V12
   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
 1     0     1     2     2     1     2     2     2     1     1     1     2
 2     1     1     1     1     1     1     2     2     1     2     3     1
 3     3     2     3     3     1     2     2     0     1     2     3     3
 4     2     2     1     2     1     2     3     2     2     1     1     1
 5     3     3     2     0     2     3     1     0     3     0     3     3
 6     0     2     3     2     2     2     1     2     2     2     0     2
 7     1     1     1     1     1     3     3     2     2     3     1     3
 8     2     2     2     3     2     2     2     2     2     2     3     3
 9     2     1     3     1     1     3     3     2     3     1     3     3
10     0     2     2     0     1     2     2     2     2     2     3     3
# … with 403 more rows, and 10 more variables: V13 <dbl>, V14 <dbl>,
#   V15 <dbl>, V16 <dbl>, V17 <dbl>, V18 <dbl>, V19 <dbl>, V20 <dbl>,
#   V21 <dbl>, V22 <dbl>
```

用最簡單的 `eRm` 套件進行分析：

```r
> eRm::PCM(A)

Results of PCM estimation: 

Call:  eRm::PCM(X = A) 

Conditional log-likelihood: -7139.254 
Number of iterations: 26 
Number of parameters: 65 

Item (Category) Difficulty Parameters (eta):
             V1.c2     V1.c3      V2.c1     V2.c2     V2.c3      V3.c1
Estimate 2.9373691 5.5004023 -0.5026313 0.5687929 2.7167951 -1.5891000
Std.Err  0.2080597 0.2905613  0.1378363 0.1852051 0.2379407  0.1445418
              V3.c2     V3.c3      V4.c1     V4.c2    V4.c3      V5.c1
Estimate -0.8842344 0.8283488 -0.3306218 1.2087767 3.667884 -0.8075350
Std.Err   0.1886138 0.2256532  0.1342302 0.1896171 0.258974  0.1355271
             V5.c2     V5.c3      V6.c1      V6.c2     V6.c3      V7.c1
Estimate 0.4579519 2.6152032 -1.5040463 -0.9743534 0.9923865 -3.1159230
Std.Err  0.1873378 0.2405321  0.1447854  0.1847781 0.2293673  0.1990869
              V7.c2     V7.c3      V8.c1       V8.c2     V8.c3      V9.c1
Estimate -3.1577746 -1.809449 -1.1221083 -0.05397499 2.1979582 -2.8551508
Std.Err   0.2259857  0.250966  0.1374213  0.18499937 0.2409469  0.1870406
              V9.c2      V9.c3     V10.c1    V10.c2    V10.c3     V11.c1
Estimate -2.9527754 -1.5478484 -0.5974519 0.4555994 2.8793634 -0.7175797
Std.Err   0.2148776  0.2424618  0.1371351 0.1829607 0.2470719  0.1349900
            V11.c2    V11.c3     V12.c1     V12.c2    V12.c3     V13.c1
Estimate 0.7249513 2.5141733 -1.6997940 -1.2915658 0.3104866 -0.2294872
Std.Err  0.1932825 0.2319935  0.1487864  0.1885285 0.2233758  0.1365726
            V13.c2   V13.c3     V14.c1     V14.c2     V14.c3    V15.c1
Estimate 1.3097317 2.977850 -1.1828484 -1.1855359 -0.3675684 -2.437145
Std.Err  0.1975586 0.231255  0.1515045  0.1893085  0.2119602  0.176375
             V15.c2     V15.c3     V16.c1     V16.c2    V16.c3     V17.c1
Estimate -2.8811047 -1.3132456 -2.0019023 -1.3900573 0.7428773 -1.1841216
Std.Err   0.2009328  0.2338507  0.1513078  0.1901427 0.2381854  0.1401928
             V17.c2    V17.c3     V18.c1     V18.c2   V18.c3     V19.c1
Estimate -0.3707148 1.5191147 -1.6189003 -0.7536189 1.222898 -0.8050713
Std.Err   0.1855372 0.2282181  0.1432545  0.1881159 0.232705  0.1341039
            V19.c2   V19.c3     V20.c1    V20.c2    V20.c3     V21.c1
Estimate 0.7225196 2.649148 -0.6201066 0.2792573 2.6452431 -1.0266341
Std.Err  0.1932131 0.237228  0.1385045 0.1818370 0.2426987  0.1392295
             V21.c2    V21.c3     V22.c1     V22.c2   V22.c3
Estimate -0.1349207 1.6591516 -1.2801121 -0.4725497 1.514371
Std.Err   0.1865880 0.2263306  0.1405882  0.1851690 0.230810

```

可以哦！

person-item map

![](https://i.imgur.com/PxN5CPk.png)



```r=
mod1 <- tam.mml(A)
persons.mod1 <- tam.wle( mod1 )
thresholds.mod1 <- tam.threshold(mod1)

wrightMap(persons.mod1$theta, thresholds.mod1, item.side = itemClassic)
```

![itemClassic](https://i.imgur.com/H5UKnVk.png)

![itemModern](https://i.imgur.com/9vfW3LB.png)


$$\dfrac{1}{2}$$

