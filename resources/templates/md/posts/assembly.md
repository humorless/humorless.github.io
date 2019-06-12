{:title "Using datomic with Luminus: Where to put our queries?"
 :layout :post
 :date "2019-06-12"
 :tags ["Datomic", "assembly", "namespace", "Luminus"]
}

If we build a Luminus project with db option other than datomic, for example `+postgres`, the code arrangement is much more straight forward. Open the file `resources/sql/queries.sql`, and put sql query and sql transaction command in this file. Then, we can just `require` the `xxx.db.core` namespace, the db queries or commands are totally available. 

## Where to put the db queries if we use db option as +datomic?

Put datomic queries in the same file with connection state in `xxx.db.core` is the first attempt I tried. However, the datomic queries actually execute in the application program runtime, not in the db server runtime. Also, if we design the query function to accept datomic db value as input argument, then our query function will become pure functions.

After discovering that our query functions are pure functions, I decide to arrange my application namespaces like this:

```
prj.[service].assembly ---> prj.db.core
                            ;; assembly only refers conn variable from prj.db.core
                       ---> datomic.api
                       ---> prj.db.query
                             ;; I make all the query functions as pure functions and put them here.
```

The namespace `[service].assembly` is used to *wire* utility funcitons (pure functions) and stateful things like datomic connection together.

## Where to put the db transactions?

Given that `[service].assembly` refers `conn`, I decide to call `(d/transact conn ... )` in this namespace. However, I still need to do some transformation to get proper transaction data that can directly put into `d/transact`. Therefore, the arrangement will be like:

```
prj.[service].assembly ---> prj.db.command
```

In `prj.db.command`, I put the transformation functions that used to create datomic transaction data. The transformation functions are also pure functions.

## Conclusion

Compared to traditional sql db option, the reasonable place to put database queries of datomic db option is totally different. 

In traditonal sql db options:
* We write HugSQL sql sourcre files with sql and tags.
* We need integration test to test these queries.
* We place our queries in `resource/sql/queries.sql`

In datomic db options:
* We write Clojure source files with data.
* We only need unit test to test these queries. 
* We place our queries in `prj.db.query` namespace.
