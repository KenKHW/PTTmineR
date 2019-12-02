
<!-- README.md is generated from README.Rmd. Please edit that file -->

# PTTmineR <img src="man/figures/logo.svg" align="right" alt="" width="120" />

簡單、快速地爬取 PTT 上的資料，讓使用者能更專注在分析工作上！

## Why `PTTmineR` ?

  - 友善使用者的語意化使用方式
  - 整合多種 PTT 文章搜尋方式
  - 內部高效率的資料處理( `data.table` )
  - 支持平行運算( `futrue` )

## Installation

安裝最新的 `PTTmineR` 開發版本：

``` r
if (!requireNamespace("remotes"))
  install.packages("remotes")

remotes::install_github("shihjyun/PTTmineR")
```

## Getting started

### 載入 `PTTmineR` 套件

``` r
library(PTTmineR)
```

### 使用 `PTTmineR$new` 創建第一個 miner

你可以幫這支 miner 所要執行的爬蟲任務取一個簡單的名稱

``` r
rookie_miner <- PTTmineR$new(task.name = "Mr. Meeseeks")
# inspect your miner's metadata
rookie_miner
#> *** PTTMINER ***
#> * task name: Mr. Meeseeks
#> * total posts: NA
#> * total comments: NA
#> * miner's size: NA
#> * last crawling date time: NA
```

接下來的任何操作都不需要再用到 `<-` 或是 `=` 綁定名字到物件上(除非你需要再創建新的 miner 物件)， 原因是在
`PTTmineR` 套件的設計上有 **Reference Semantics** 的特性，所以接下來所操作的 function 都會是
side-effect function，對這部分不理解的話並不會影響套件的使用，有興趣的話可以參考之後的技術文件會更深入說明，
而現在你只需要透過接下來的範例知道爬到的資料最後都到了哪裡就好了😁！

### 使用 `mine_ptt()` 獲得資料

在 `mine_ptt()` function 中除了 `ptt.miner` 需輸入剛剛創建 miner 變數外，也可以輸入一下參數：

  - `board` : 字串，必填，你想要爬的板 e.g. Gossiping, Beaudy, HatePolitics
  - `keyword` : 字串，選填，在板中透過標題關鍵字搜尋
  - `author` : 字串，選填，在板中透過文章作者搜尋
  - `recommend` : 數字，選填，在板中透過文章淨推文搜尋
  - `min.date` : 字串，選填，爬到什麼時間點為止 e.g. `2018-01-01`,
    `2019-11-01 15:00:01`
  - `last.n.page` : 數字，從最前面一頁開始爬多少頁

以上提到的各種搜尋條件都可以混搭使用！

``` r
# You can ...
mine_ptt(ptt.miner = rookie_miner,
         board = "Gossiping",
         last.n.page = 10)
# or ...(Using `%>%` is more semantic !!)
rookie_miner %>% 
  mine_ptt(board = "Gossiping",
           last.n.page = 10)

#> 🙉 PTTmineR mining from ptt on your setting ... DONE
  
```

雖然 `PTTmineR` 的 function 都是 side-effect function，但一樣可以支持 `%>%` 進行語意化操作：

「rookie\_miner 從 Gossiping 板中爬最新的十頁文章回來！」

``` r
rookie_miner

#> *** PTTMINER ***
#> * task name: Mr. Meeseeks
#> * total posts: 192
#> * total comments: 6487
#> * miner's size: 1.52 MB
#> * last crawling date time: 2019-12-02 15:45:03
```

### 使用 `export_ptt()` 輸出爬下來的資料

除了 `ptt.miner` 要放入 miner 物件外，`export.type` 參數目前接受三種資料輸出格式：

  - `"dt"` : `data.table` 格式，習慣 `data.tabla` 操作的使用者可以使用
  - `"tbl"` : `tibble` 格式，習慣 `tidyverse` 操作的使用者可以使用
  - `"nested_tbl"` : 巢狀 `tibble` 格式，只有一張表，各篇文章的推文內容會以單個 column 的形式儲存

而最後的 `obj.name` 要填入最後要回傳到全域環境的物件名稱

``` r
rookie_miner %>% 
  export_ptt(export.type = "tbl",
             obj.name = "tbl_result")

colnames(tbl_result$post_info_tbl)

#> [1] "post_id"        "post_author"    "post_category"  "post_title"     
#> [5] "post_date_time" "post_ip"        "post_country"   "post_content"  
#> [9] "post_board"  

colnames(tbl_result$post_comment_tbl)

#> [1] "post_id"        "push_type"      "push_id"        "push_content"   
#> [5] "push_ip"        "push_date_time"
```

一般來說在自己定義的物件名稱中會得到兩張表：

  - `post_info_tbl` : 文章基本資料
  - `post_comment_tbl` : 推文的基本資料

而以上所顯示的欄位就是現階段 `PTTmineR` 能夠爬取的資料

#### 真正的資料在哪裡？

其實爬下來的資料都是存放在 miner 物件中，但會有額外 `export_ptt()` 的設計是希望使用者 盡量不要去修改到 miner
物件中的原始資料！

``` r
rookie_miner$result_dt
```

### 使用 `update_ptt()` 更新已經爬的文章推文

我們知道推文是會動態增加的，所以我們要分析時有可能會想知道之前爬過的文章有沒有新的推文產生

``` r
rookie_miner

#> *** PTTMINER ***
#> * task name: Mr. Meeseeks
#> * total posts: 192
#> * total comments: 6487
#> * miner's size: 1.52 MB
#> * last crawling date time: 2019-12-02 15:45:03

update_id <- rookie_miner$result_dt$post_info_dt$post_id

rookie_miner %>% 
  update_ptt(update.post.id = update_id)

rookie_miner
```
