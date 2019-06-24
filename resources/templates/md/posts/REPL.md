{:title "REPL tips"
 :layout :post
 :date "2019-03-30"
 :tags ["repl", "tips"]}

從今年 2 月開始，接了一個公司內部應用軟體的專案開發，我用 clojure + luminus + datomic 來實作。不知不覺也就每天寫 clojure 的 REPL 近兩個月了。每天玩 REPL 之後，很快就發現一些過去我用 REPL 的盲點。

### 沒有善用 `clojure.repl/pprint`

  沒有善用的主要原因，自然是因為在 `fireplace.vim` 的環境下，一開始我沒有特別做一些設定時，直接做 cpp, cqp 之類 REPL 操作，並不會有 pretty print 的輸出。後來，我總算是下定決心，把 [leiningen profiles](https://github.com/humorless/dotfiles/blob/master/profiles.clj) 設定好，加入了一個叫 `vinyasa` 的 leiningen dependency

  設定好之後，就可以用 `(>pprint ...)` 來做 pretty print 。

### 沒有善用 `*1` `*2`

  過去，我在做 REPL 操作時，常常做的事情是這樣子：

  `(f1 a b c)` => 試到結果正確

  `(f2 (f1 a b c) d)` => 也是試到結果也正確

  `(f3 (f2 (f1 a b c) d) e)` => 然後指令就愈來愈長, 愈來愈難下

  其實不用這樣子麻煩，第二次可以這樣子下指令 `(f2 *1 d)` 。
