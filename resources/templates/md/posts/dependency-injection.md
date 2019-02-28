{:title "dependency injection with Clojure"
 :layout :post
 :date "2017-07-12"
 :tags ["unit test", "dependency-injection", "clojure"]}

寫 clojure 的時候，雖然套用了 REPL-driven development 的開發方式，已經相對可以讓大多數的函數很快地做過測試。但是，隨著要開發的專案愈來愈大，還是一樣需要用標準的寫法來寫單元測試 (unit test) 。有一個非正規的統計，如果是 Ruby on Rail 的專案，一般而言，90% 的函數都是有副作用的。然而， clojure 語言的專案，往往只有 40% 的函數帶有副作用。

即使是寫 clojure 語言，還是會遇到有 side effect 的函數，那比較好的寫法是怎麼樣呢？

<!--more-->

我查了一下 stackoverflow 之後，很快就找到了一個很好用的函數 `with-redefs` 。 stackoverflow 上的答案大意如下： 由於 clojure 語言有 Dynamic binding 的特性，使用 `with-redefs` 就可以實現同樣的語意了。

我試了一下，還真的管用，範例如下：

```clojure
(deftest platform-contact-test
  (testing "platform-contact"
    ; use the DI technique to test the function platform-contact
    (is (= 170
           (with-redefs [get-platform-contact (fn [_] (slurp "./resources/contact_data.txt"))]
             (count (platform-contact (temp-platform-all))))))))

```

在這個範例中，原本的 `get-platform-contact` 函數是一個有副作用的函數，它會被 `platform-contact` 函數呼叫。 `get-platform-contact` 函數會發出一個 http request ，並且傳回遠端 server 上的資料，所以如果沒有加以代換，單元測試就會非常慢。用了 `with-redefs` 之後，就可以輕易地將 `get-platform-contact` 代換成一個會傳回固定檔案資料的函數，如此就可以執行快速的單元測試了。

對於 clojure 這種先進的特性， stackoverflow 上有一句評論： Needing a framework for DI is really just compensating for a lack of sufficient features in the language itself.
