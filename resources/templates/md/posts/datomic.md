{:title "Using Datomic in my app"
 :layout :post
 :date "2019-04-27"
 :tags [ "Datomic", "tips", "experience"]}

## The background of this post
I began to use Datomic seriously in my project at work from February 2019. Now, it is time to write down certain experience. When I just began, I found a lot of documents talking about how to use Datomic. However, I still found certain points worth to mention from my project.

## Query API and Pull API are enough
When I just begin to write Datomic, soon I found [post](https://vvvvalvalval.github.io/posts/2016-07-24-datomic-web-app-a-practical-guide.html) from Val. In the post, Val used Entity API. 

In my project, I used only Query API and Pull API. Query API was for taking out entity id mostly and Pull API was for pulling out necessary field or sometimes doing some 'join'. I think the article [SEPARATION OF CONCERNS IN DATOMIC QUERY: DATALOG QUERY AND PULL EXPRESSIONS](http://blog.cognitect.com/blog/2017/4/21/separation-of-concerns-in-datomic-query-datalog-query-and-pull-expressions) has explained similar idea. Entity API is also good, but Pull API is even better.

## Occasionally, a generalized CAS (compare-and-swap) is needed, or you need to use stamp.
In my project, I need to use Datomic to model:

1. The user can propose request. Initially, the request is in open status.
2. The admin can approve/reject/modify the user request.

The request schema is like:
```
:req/status     ;; cardinality one. It can be - open, modified, approved, rejected
:req/things     ;; cardinality many. [thing-id ...]
```
The admin sees the user requests from a web application UI. There are three options for admin: approve, reject, modify. If a request is approved or rejected, then this request is no longer alive. It will disappear from admin UI. However, if a request is modified, it can still be approved, be rejected, or be modified again. When the request is modified, only the `req/things` can be modified. There may be multiple admins operating at the same time on the same request in this system.

The state diagram of request status is:
```
 open -> modified 
 modified -> modified 
 {modified, open} -> approved (done)
 {modified, open} -> rejected (done)
```

Consider a situation: Two admins A and B process on the same request and they do not sense each other. They push the button at the same time. One admin A approves the request and another admin B modifies the request. The request was originally modified before, so it is at the status `modified` when the two admins process it.

The correct behavior of the system could be two possibilities: Either operation of admin A is successful or operation of admin B is successful. If operation of admin A is successful first, then the request can not be modified anymore. If the operation of admin B is successful first, then the approval of A should not happen, because the `req/things` is already modified, but the admin A approved different set of `req/things`.

I consider to utilize `db.fn/cas` to guarantee that only one operation of admin A or admin B can succeed. However, `db.fn/cas` does not work on attributes with cardinality many.

I think there are two ways to solve this mutually exclusive concurrent operation problem:

1. Add an extra schema `req/stamp` into req. The stamp is initially 0. Every operation will increase it by 1. Then I can use this stamp and `db.fn/cas` to ensure the logically strictness of the operations.
2. Install some customized db function, which can do CAS on cardinality many to ensure the logically strictness.

## DB Enumeration

I use `:db/ident` to do enumerations in my project:
```
[:db/add #db/id [:db.part/user] :db/ident :product.type/account]
[:db/add #db/id [:db.part/user] :db/ident :product.type/display]
```

They are enumerations that represent the different products. Then, there are certain related issues associated with this modeling.

### How to pull out all the enumerations of the same type?

I deliberately set the enumeration of the same type with the same namespace, so I need to prepare a query that can filter based on the same namespace. It is very convenient that we can directly use Clojure function in Datomic query.

```
(defn product-enum-eids
  "all the product enumeration eids"
  [db]
  (d/q '[:find [?e ...]
         :in $ ?nsp
         :where [?e :db/ident ?attr]
         [(namespace ?attr) ?nsp]]     ;;Datomic Function expression binds the ?nsp variable
       db "product.type"))
```

### How to store the external string and enumeration mapping in Datomic?

Once again, I use simple schema with no magic.

```
   {:db/doc "External name associated with a db enumeration value"
    :db/ident :enum/name
    :db/valueType :db.type/string
    :db/cardinality :db.cardinality/one
    :db/unique :db.unique/identity
    :db/id #db/id [:db.part/db]
    :db.install/_attribute :db.part/db}

   {:db/doc "db enumeration value"
    :db/ident :enum/value
    :db/valueType :db.type/ref
    :db/cardinality :db.cardinality/one
    :db/unique :db.unique/identity
    :db/id #db/id [:db.part/db]
    :db.install/_attribute :db.part/db}
```

When we need to import data from files and we need to map external names to DB enumeration values, we can pull out all the mapping at once. 

```
(defn name2enum-table
  "create a mapping table that can lookup enumeration from string name."
  [db]
  (into {}  (d/q '[:find ?k ?enum
                   :where
                   [?e :enum/name ?k]
                   [?e :enum/value ?v]
                   [?v :db/ident ?enum]]
                 db)))
```
