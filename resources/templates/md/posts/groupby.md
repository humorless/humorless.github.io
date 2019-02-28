{:title "groupby"
 :layout :post
 :date "2017-05-21"
 :tags ["groupby" "database"]}

一開始是我在寫 [4clojure](http://www.4clojure.com/problem/63) 的練習題的時候，寫到了一個題目，要重新實現 clojure 語言的 groupby 函數。我糾結了好一陣子，又查了不少資料，才勉強用 reduce 寫出來。然而，最近卻在工作中，用上了 groupby 。

```clojure
(fn f [k coll]
  (reduce
    (fn [c v]
      (update-in c [(k v)] (fnil conj []) v))
    {} coll))

```
<!--more-->
工作上遇到的問題是要重構同事寫的程式碼。程式碼做的事情是：「接受資料庫 dump 的 json 輸出，跑兩層很複雜的迴圈，對原始的資料做主鍵交換的處理，然後將資料存入 mysql 資料庫。」資料庫 dump 出來的 json 大概長成如下的樣子：

```json
  "result": [
    {
      "platform": "c01.i01",
      "ip_list": [
        {
          "ip": "192.168.0.1",
          "hostname": "ggyy6699"
        },
        {
          "ip": "192.169.1.1",
          "hostname:": "ggyy7700"
        }
      ]
    },
    {
      "platform": "c01.i05",
      "ip_list": [
        {
          "ip": "192.168.0.2",
          "hostname": "ggkk8899"
        },
        {
          "ip": "192.169.1.2",
          "hostname:": "ggkk9900"
        }
      ]
    }
  ]
}

```

從這個 json 來看的話，`platform` 是主鍵 (primary key) 。而每一個 `platform` 下之下會有多個 `hostname` 。而程式碼做的事情是，先解析這個 json ，重新整理之後，讓 `hostname` 變成主鍵 (primary key) ，並且做成一行又一行的 row ，最後要存入關聯式資料庫。讓我感到困擾的地方是因為整理屬性與屬性之間複雜關系的程式碼，都塞在雙重迴圈裡頭，所以雙重迴圈就變得很複雜，而且這一段雙重迴圈的程式碼也無法複用，難以修改、難以維護。

轉換成用資料庫的觀點來看待這個問題之後，就得到了還不錯的解法：
*  資料庫的 dump 輸出，本質上也是 join 兩張資料表的結果輸出，所以主鍵 (primary key) 本來就有可能交換。
*  既然要解析的資料是 join 之後的結果，所以有效的處理方式是這樣子：
   1. 先將 json 的資料跑完簡單的雙重迴圈，雙重迴圈只做一件事，只將將資料做展開 (unfolding)，變成 join 完成的樣子。
   2.  python 的 `itertools.groupby` ，可以讓資料表 (table) 重新整理，產生出以任意的 column 做為主鍵 (primary key) 的新資料表 (table)。


程式碼如下：
```python
def get_h_platforms(res):
    """ sample output
    ctl-zj-061-130-028-019 ['c01.p02', 'c01.p02-kugou']
    ctl-zj-061-130-028-020 ['c01.p02', 'c01.p02-kugou']
    ctl-zj-061-130-028-022 ['c01.p02', 'c01.p02-kugou']
    """
    product = [(p["platform"], device["hostname"])
               for p in res["result"] for device in p["ip_list"]]
    data = sorted(product, key=lambda x: x[1])
    for key, grp in itertools.groupby(data, key=lambda x: x[1]):
        print(key, list(map(lambda x: x[0], set(grp))))
```
