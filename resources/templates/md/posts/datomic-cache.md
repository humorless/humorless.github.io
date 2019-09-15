{:title "Using Datomic with disk cache and LU cache"
 :layout :post
 :date "2019-09-15"
 :tags [ "Datomic", "tips", "performance"]
 :toc true}

## The background of this post
I began to use Datomic seriously in my project at work from February 2019. I encountered certain performance issues and I solved them through disk cache and LU cache.

## Analytical queries need pre-computation
My project had several analytical queries which implemented the business rules. With Datomic expressive query power, it was very easy to implement queries that closely related to domain model. Great for expressiveness, but the query speed was quite slow, I needed to do some pre-computation.  

How to save the query results? My query results were in the form of EDN format. Should I prepare a key-value database to cache it? Or, should I use just Datomic to serve as the key-value database?

## Using Datomic as key-value store
In my use cases, I used Datomic as key-value store with the following schema.

```
| schema name | :booking/tx  | :booking/team      | :booking/bytes |
|-------------|--------------|--------------------|----------------|
| data type   | long         |  string            |  bytes         |

```

The `:booking/tx` and `:booking/team` served as keys and `:booking/bytes` served as value. Before I stored the EDN format value into `:booking/bytes`, I first required a smart library: [Nippy](https://github.com/ptaoussanis/nippy). Nippy helped to transform Clojure composite data structure into plain Java bytes.

Here came another question: Was there any size limit with the Datomic schema type: `db.type/bytes`? I spent some time to find the answer in Datomic google group.

> I believe the rule of thumb is that values stored in Datomic — strings, bytes, etc. — should not exceed one kilobyte. Nothing will break if they do, but Datomic's storage layout is optimized for values this size or smaller.

Great! Nothing will break if they do.

## OutOfMemory Error occurred

Some of my queries used `not-join` syntaxes. At the beginning, `not-join` looked like great things because with `not-join` I could express my intent without any low level interpretation. Soonly, the queries with `not-join` threw an `OutOfMemory` error. Therefore, I decided to do some optimizations.

## Extract not-join out of query and use LU-cached memoize

The original query was like this:

```
(d/q '[:find (count ?r) .
       :where [?r :release/name "Live at Carnegie Hall"]
              (not-join [?r]
                [?r :release/artists ?a]
                [?a :artist/name "Bill Withers"])]
       db)
```

The modified equivalent queries were:

```
(def B (into #{}
             (d/q '[:find ?r
                    :where [?r :release/artist ?a]
                           [?a :artist/name "Bill Withers"]]
                    db)))

(d/q '[:find (count ?r) .
       :in $a $b
       :where
       [$a ?r :release/name "Live at Carnegie Hall"]
       ($b not [?r])]
     db B)
```

After I extracted `not-join` part out of the original query, I discovered that some of my new query would be called with similar inputs across several queries. It would save some computation resources if I used `memoize` to modify the new query.

However, the standard version `clojure.core/memoize` would cause memory leak. I chose the Clojure contrib library [core.memoize](https://github.com/clojure/core.memoize) to cache the query result with LU cache.

The to be memoized query functions were like this:
```
(defn memo-q*
  [db t]
  (into #{}
        (d/q '[:find ?r
               :where [?r :release/artist ?a]
                      [?a :artist/name "Bill Withers"]]
             db)))

(def memo-q (clojure.core.memoize/lu memo-q* {} :lu/threshold 2))
```


The memoized query function was called like this:
```
(memo-q db (d/basis-t db))
```

The Datomic t of the most recent transaction reachable via the db value served as the input parameter to decide whether the cached result was out of date.
