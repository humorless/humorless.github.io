{
 :title "A Clojurian's idioms and patterns for ETL"
 :layout :post
 :date "2019-07-01"
 :tags ["idiom", "pattern", "let over map merge", "ETL", "Excel", "Datomic"]
 :toc true
}

## Background

I needed to do eight Excel ETLs at my project. At the beginning, I just implemented some of the ETLs without any design. I did not even implement schema validation, and then I felt the pain soon. After several re-writing, I abstracted out some idioms and patterns for ETL.

## Problems

We need to import data from several Excel files into Datomic database. There are several concerns with the ETL (extract-transform-load):

1. Schema validation:
   Can we have a validation function that we only need to inject the schema and then the validation function will handle all the rest for us? 
2. Transformation complexity:
   The transformation from Excel to Datomic table varies a lot. The simplest one is just copy data, but the complex ones need to look up tables in the database. How can we organize different type of transformation functions such that the functions can be more reusable and composable?
3. Database upsert semantic:
   The identity key of the database table may be compound fields, or there may be some cardinality-many fields in the database table. That is to say, the basic upsert semantic offered by Datomic is not enough.

### Solution for schema validation

The library `clojure.spec` is great for schema validation. 
```
;; library functions defined at utility namespace
(defn check-raw-fn
  "assemble schema and then create a validation fn"
  [schema]
  (fn check-raw
    [data]
    (if (spec/valid? schema data)
      data
      (let [desc (spec/explain-str schema data)]
        (throw (ex-info desc {:causes data :desc desc}))))))

;; Example application functions
(spec/def ::apply-time inst?)
(spec/def ::customer-id string?)
(spec/def ::lamp-customer-id string?)
(spec/def ::sales-name string?)
(spec/def ::source #{"agp" "lap"})

(spec/def ::mapping
  (spec/* 
    (spec/keys :req-un
               [::apply-time ::customer-id ::lamp-customer-id ::sales-name ::source])))

(def ^:private check-raw
  (utility/check-raw-fn ::mapping))
```

In this design:
- Even though I do not know how many rows an Excel file may have, I can still use `(spec/* ...)` to represent the schema for the Excel file. If the spec does not offer the semantic like `(spec/* ...)`, I have to write some loop logic in `check-raw-fn` function, which causes the context dependency.
- The spec names are just the same as the column names of Excel. Keep it simple making the program more robust.
- If a string has only a few possible options, represent it in the form as `#{option1 option2 ...}`
- When throwing exception, I use `(ex-info ...)` and I put the output of `(spec/explain-str ...)` into an exception. Then, I can find out what is wrong by just reading the exception message.  
 
Also, at the trigger API of ETL, the web API deliberately catches only certain types of Exception:
```
(try (if-let [r (etl/sync-data cmd filename)]
            (ok {:result :insert-done})
            (ok {:result :already-sync}))
          (catch clojure.lang.ExceptionInfo e
            (bad-request {:reason (ex-data e)}))
          (catch java.util.concurrent.ExecutionException e
            (bad-request {:reason (.getCause e)})))
```

The exception `clojure.lang.ExceptionInfo` only catches the schema validation error thrown by my application code. The exception `java.util.concurrent.ExecutionException` can catch the error from Datomic transaction. Other exceptions may happen with lower possibility, so I let them pass over and be recorded in log file.

### Solution for transformation complexity --- let over map merge

I propose a pattern, which I call it as `let over map merge` to handle the transformation complexity.

Consider a transformation function `data->txes`, both the input and the output are sequences of map:
- The single map in the input data represents the row in the Excel file.
- The single map in the output `txes` represents the row in the Datomic table. 


```
(defn- data->txes
  "data is a sequence of {HashMap}"
  [data]
  (let [db (d/db conn)
        table (utility/tax-id->c-eid db)]
    (map #(transformation-f table %) data)))
```

We can easily divide the transformation into two categories:
1. Basic transformation: Just copy the field, or with pure function transformation.
2. Complex transformation: When transforming the input data, we need to also look up the database content.

If we pull out `basic-mapping` and `complex-mapping` from `transformation-f`, we can change the original code into
```
(defn- data->txes
  [data]
  (let [db (d/db conn)
        table (utility/tax-id->c-eid db)]
    (let [basic-tx (map basic-mapping data)
          complex-tx (map #(complex-mapping table %) data)]
      (map merge basic-tx complex-tx))))
```

With this `let over map-merge` pattern, we can make the granularity of the transformation function smaller so as to make them more reusable and composable. In certain cases, basic-mapping only needs to change the key-name in the hash map, so we can use `clojure.set/rename-keys` to implement the basic-mapping.

### Solution for database upsert semantic

In Datomic, we can use the `:db.unique/identity` to make certain schema work like primary key in traditional RDBMS. 

#### Compound primary key
Consider tha table with compound primary key as `stream-unique-id, writing-time, source`. How to do upsert when we have `txes` like below?
```
 [#:rev-stream{:stream-unique-id "AA"
               :writing-time #inst "2019-04-01T02:39:00.000-00:00"
               :source :etl.source/agp
               :campaign-name "BB"}]
```

With a db transaction function `upsert-rev-stream`, we can simply write `txes` as 
```
 [[:fn/upsert-rev-stream 
   #:rev-stream{:stream-unique-id "AA
                :writing-time #inst "2019-04-01T02:39:00.000-00:00"
                :source :etl.source/agp
                :campaign-name "BB"}]]
```

The transaction function `:fn/upsert-rev-stream` handles the upsert complexity.
```
 {:db/id #db/id [:db.part/user]
  :db/ident :fn/upsert-rev-stream
  :db/doc "The primary key of rev-stream is compound key"
  :db/fn #db/fn
  {:lang :clojure
   :params [db m]
   :code (if-let [id (ffirst
                      (d/q '[:find ?e
                             :in $ ?u ?t ?s
                             :where
                             [?e :rev-stream/stream-unique-id ?u]
                             [?e :rev-stream/writing-time ?t]
                             [?e :rev-stream/source ?s]]
                           db (:rev-stream/stream-unique-id m)
                           (:rev-stream/writing-time m)
                           (:rev-stream/source m)))]
           [(-> (dissoc m :rev-stream/stream-unique-id
                        :rev-stream/writing-time
                        :rev-stream/source)
                (assoc :db/id id))]
           [m])}}
```

#### Cardinality many
Consider tha table with a cardinality-many schema `:order/accounting-data` and `:order/product-unique-id` with `:db.unique/identity`. How to do upsert when we have `txes` like below?
```
[#:order{:io-writing-time #inst "2019-04-01T02:39:00.000-00:00",
         :service-category-enum :product.type/today,
         :accounting-data
         [#:accounting{:month "2019-04", :revenue -2}
          #:accounting{:month "2019-05", :revenue -3}
          #:accounting{:month "2019-02", :revenue 4}
          #:accounting{:month "2019-01", :revenue 5}]}]
```

With a db transaction function `upsert-order`, we can simply write `txes` as
```
  [[:fn/upsert-order
    #:order{:io-writing-time #inst "2019-04-01T02:39:00.000-00:00",
            :service-category-enum :product.type/today,
            :accounting-data
            [#:accounting{:month "2019-04", :revenue -2}
             #:accounting{:month "2019-05", :revenue -3}
             #:accounting{:month "2019-02", :revenue 4}
             #:accounting{:month "2019-01", :revenue 5}]}]]
```
 
The transaction function `:fn/upsert-order` handles the upsert complexity.
```
 {:db/id #db/id [:db.part/user]
  :db/ident :fn/upsert-order
  :db/doc "The :order/accounting-data is cardinality many.
  When insert semantic, transact `[m]`
  When update semantic, do retraction of :order/accounting-data first and then transact `m`  "
  :db/fn #db/fn
  {:lang :clojure
   :params [db m]
   :code (if-let [eid (ffirst
                      (d/q '[:find ?e
                             :in $ ?u
                             :where
                             [?e :order/product-unique-id ?u]]
                           db (:order/product-unique-id m)))]
           (let [ad-refs (d/q '[:find [?a ...]
                                :in $ ?e
                                :where [?e :order/accounting-data ?a]]
                              db eid)
                 retracts (mapcat (fn [r]  [[:db/retractEntity r]
                                            [:db/retract eid :order/accounting-data r]]) ad-refs)]
             (conj (vec retracts) m))
           [m])}}
```

## Conclusions

From abstracting out idioms and patterns of ETL, I understand that context dependency is the primary cause of the complex application code. Both Datomic transaction functions and regular expression syntaxes of `clojure.spec` can help to remove the context dependency of our application code. Use them wisely!
