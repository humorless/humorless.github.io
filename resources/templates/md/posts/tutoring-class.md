{:title "Teaching Clojure programming class"
 :layout :post
 :date "2020-05-05"
 :tags [ "programming class", "tutoring", "experience"]}

When I told others that I am a Clojure programmer, they responded apathetically. Why so many people in Taiwan never heard of this great programming language? One day, an idea occurred to me that how about teaching some students? 

## The advertisement

I re-wrote my advertisement again and again. What kind of value proposition would be appreciated by my prospects? I actually did not know. At the end, I wrote 3 objectives in my [advertisement](https://medium.com/@replware/%E5%87%BD%E6%95%B8%E5%BC%8F%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88-functional-programming-%E5%BE%9E%E5%9F%BA%E7%A4%8E%E5%88%B0%E5%AF%A6%E5%8B%99-bc06f68c7bfb). 

1. Help you learn the Clojure programming language
2. Help you become the real senior programmer in the eyes of your colleagues. 
3. Help you become more confident whenever you want to ask for a raise.

My friends did not believe I could get students, and they tried to tell the uncomfortable truth mildly. They asked something like "Who is your target audience?"

Fortunately, I got two students just after I posted it. Two of my college classmates wanted to learn Clojure programming language from me. 

## The ways of teaching

At the very beginning, I asked my students what they want to learn. This was a one-on-one tutoring class, so it could be customized. I organized the class into 4 stages.

1. [Development environment setup](https://github.com/humorless/dotfiles). 
2. About the Clojure's productivity --- [The productivity brought by Clojure](https://www.slideshare.net/humorless/the-productivity-brought-by-clojure-149170292).
3. Practicing at the [4clojure](http://www.4clojure.com/) website.
4. Studying any specific topics they were interested. (Customization part)

## Lessions learned from teaching

Through the questions from my students I found out some obstacles in learning Clojure. 

### Switching between purpose view and implementation view

When I write recursion, I split it into three steps:
1. Choose the name of this recursion function. The name is about this function's purpose.
2. Think about the boundary condition of this function. When will it stop?
3. Write the implementation of this function. This function should be implemented by a call to itself with a different input argument and some connection code.

```
(defn range-to-zero [x] 
  (when (> x 0)
    (conj (range-to-zero (dec x)) x)))
```

Take the above code as an example. I think the output of `(range-to-zero 4)` is `'(4 3 2 1)`. When I want to define the `(range-to-zero 5)`, I just need to conj a `5` to the `'(4 3 2 1)`. 

My students did not think like this way: they simulated the execution of the code from the very top toward the boundary conditions. They organized their mind like an interpreter and they traced the code just like the interpreter did. I told my student that you need to switch your thinking between purpose view and implementation view.

### Different levels of complexity

After my students solved about 50 questions in 4clojure, they felt a sense of confidence to fly alone. However, when they just solved about 25 questions, they felt quite confused. They were very confused about the idiomatic ways to solve 4clojure questions. 

I considered there were 5 levels of complexity:
1. Solve the question by remembering the Clojure function name. For example, `frequencies`. 
2. Solve the question by using some sequence questions: `map`, `filter`, `mapcat`.
3. Solve the question by using `reduce`.
4. Solve the question by using recursion.
5. Solve the question by using mutual recursion.

I encouraged my students to solve any questions using as fewer level of complexity as possible. There were still certain special cases like `flatten`, which might not fit into my categories.

## Final notes

One of my students told me that he decided to learn the Clojure programming language because of Robert C. Martin's recommendation. Thanks to uncle Bob that he had done great marketing for me.
