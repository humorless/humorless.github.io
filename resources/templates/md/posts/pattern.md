{:title "pattern"
 :date "2017-02-28"
 :layout :post
 :tags  ["golang", "clojure"]}
## patterns = programming with abstactions that are not powerful enough

先來引述一下 Paul Graham 的句子
> When I see patterns in my programs, I consider it a sign of trouble. The shape of a program should reflect only the problem it needs to solve. Any other regularity in the code is a sign, to me at least, that I'm using abstractions that aren't powerful enough.
>- Paul Graham - Revenge of the Nerds

為了想出可以妥善解釋這段話的意思的 non-trivial 範例，其實我還想了滿久的。不料真的就在我學習 clojure 語言的過程之中找到了。這個範例是對某個 array 的每一個元素，做相同的運算處理：一個是循序處理、一個是平行處理。

<!--more-->

### golang 的兩個版本

循序處理的版本
```golang
res := make([]float, N);
for i,xi := range data {
    func (i int, xi float) {
        res[i] = doSomething(i,xi);
    } (i, xi);
}

```

平行處理的版本
```golang
type empty {}
...
data := make([]float, N);
res := make([]float, N);
sem := make(chan empty, N);  // semaphore pattern
...
for i,xi := range data {
    go func (i int, xi float) {
        res[i] = doSomething(i,xi);
        sem <- empty{};
    } (i, xi);
}
// wait for goroutines to finish
for i := 0; i < N; ++i { <-sem }
```

### clojure 的兩個版本 

循序處理的版本
```clj
(defn myfun [coll]
  (map doSomething coll))
```

平行處理的版本

```clj 
(defn myfun [coll]
  (pmap doSomething coll))

```
### 抽象層次的差異
比較這兩種語言寫的四段程式碼，很快可以發現，循序處理的範例都相當的簡單。然而，當換成平行處理的版本時， golang 的實作比 clojure 難多了。需要用 golang 的 channel 做出一個 semaphore 的 pattern 才能實現。而相較之下， clojure 把 map 換成 pmap 就可以了。由此可見， clojure 在這個例子之中，是一種足夠強的抽象層，可以輕易地去表達這個平行處理的語意。
