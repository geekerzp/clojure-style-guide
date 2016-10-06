# Clojure 风格指南

> 榜样很重要。 <br/>
> -- 《机械战警》 Alex J. Murphy 警官

这份 Clojure 旨在为 Clojure 程序员编写简洁易懂，易于维护的高质量 Clojure 代码提供一份最佳实践。无论多么好的风格或指南，但是过于理想化的结果导致大家拒绝使用或者可能根本没人用，毫无意义。

本指南分为几个小节，每一小节由几条相关的规则构成。我尽力在每条规则后面说明理由（如果省略了说明，那是因为其理由显而易见）。

这些规则不是我凭空想象出来的 --- 它们中的绝大部分来自我多年以来作为职业软件工程师的经验，来自 Clojure 社区成员的反馈和建议，以及许多备受推崇的 Clojure 编程资源，例如 ["Clojure Programming"](http://www.clojurebook.com/)
和 ["The Joy of Clojure"](http://joyofclojure.com/)。

这份指南还处于不断完善中，可能有一些部分是缺失的，可能有一些是不完善的，可能有一些规则缺少例子，可能有一些规例没有用例子阐述的足够清晰。要记住的是，随着时间的推移这些问题都会被解决。

请注意，Clojure 开发者社区同样维护了一份 [coding standards for libraries](http://dev.clojure.org/display/community/Library+Coding+Standards) 列表。

你可以使用 [Pandoc](http://pandoc.org/) 来生成本文的 PDF 或 HTML 版本。

本指南同时有以下语言的翻译版：

* [英文原版](https://github.com/bbatsov/clojure-style-guide)
* [日语](https://github.com/totakke/clojure-style-guide/blob/ja/README.md)
* [韩语](https://github.com/kwakbab/clojure-style-guide/blob/master/README-koKO.md)

## 目录

* [组织](#source-code-layout--organization)
* [语法](#syntax)
* [命名](#naming)
* [集合](#collections)
* [可变性](#mutation)
* [字符串](#strings)
* [异常](#exceptions)
* [宏](#macros)
* [注释](#comments)
    * [注解](#comment-annotations)
* [经验](#existential)
* [工具](#tooling)
* [测试](#testing)

## 组织

> 所有风格都又丑又难读，自己的除外。几乎人人都这样想。把 “自己的除外” 拿掉，<br>
> 他们或许是对的... <br>
> -- Jerry Coffin（论缩排）

* <a name="spaces"></a>
  使用 **空格** 进行缩进， 不要使用制表符。
  <sup>[[link](#spaces)]</sup>

* <a name="body-indentation"></a>
  使用两个空格缩进含有参数的 `form` 的内容。
  包括所有的 `def` `form`， 特殊 `form` (`special form`)，以及引入局域绑定的宏
  (例如 `loop`， `let`， `when-let`) 和例如 `when` ，
  `cond`， `as->`，`cond->`，`case`，`with-*` 的宏。
  <sup>[[link](#body-indentation)]</sup>

    ```Clojure
    ;; good
    (when something
      (something-else))

    (with-out-str
      (println "Hello, ")
      (println "world!"))

    ;; bad - four spaces
    (when something
        (something-else))

    ;; bad - one space
    (with-out-str
     (println "Hello, ")
     (println "world!"))
    ```

* <a name="vertically-align-fn-args"></a>
  函数/宏的多行参数缩排在同一层级。
  <sup>[[link](#vertically-align-fn-args)]</sup>

    ```Clojure
    ;; good
    (filter even?
            (range 1 10))

    ;; bad
    (filter even?
      (range 1 10))
    ```

* <a name="one-space-indent"></a>
  如果没有参数和函数名称在同一行，函数/宏的参数保持一个空格缩进。
  <sup>[[link](#one-space-indent)]</sup>

    ```Clojure
    ;; good
    (filter
     even?
     (range 1 10))

    (or
     ala
     bala
     portokala)

    ;; bad - two-space indent
    (filter
      even?
      (range 1 10))

    (or
      ala
      bala
      portokala)
    ```

* <a name="vertically-align-let-and-map"></a>
  `let` 绑定以及 `map` 关键字缩排在同一层级。
  <sup>[[link](#vertically-align-let-and-map)]</sup>

    ```Clojure
    ;; good
    (let [thing1 "some stuff"
          thing2 "other stuff"]
      {:thing1 thing1
       :thing2 thing2})

    ;; bad
    (let [thing1 "some stuff"
      thing2 "other stuff"]
      {:thing1 thing1
      :thing2 thing2})
    ```

* <a name="optional-new-line-after-fn-name"></a>
  当 `defn` 没有 `docstring` 时，函数名称和参数列表可以选择性的分布在同一行。
  <sup>[[link](#optional-new-line-after-fn-name)]</sup>

    ```Clojure
    ;; good
    (defn foo
      [x]
      (bar x))

    ;; good
    (defn foo [x]
      (bar x))

    ;; bad
    (defn foo
      [x] (bar x))
    ```

* <a name="multimethod-dispatch-val-placement"></a>
  `multimethod` 的 `dispatch-val` 要和函数名称保持在同一行。
  <sup>[[link](#multimethod-dispatch-val-placement)]</sup>

    ```Clojure
    ;; good
    (defmethod foo :bar [x] (baz x))

    (defmethod foo :bar
      [x]
      (baz x))

    ;; bad
    (defmethod foo
      :bar
      [x]
      (baz x))

    (defmethod foo
      :bar [x]
      (baz x))
    ```

*   <a name="docstring-after-fn-name"></a>
    当增加一个 `docstring` 的时候，尤其是对于使用这个 `docstring` 的函数，
    注意正确的位置应当是函数名称之后，而不是参数列表之后。
    后者没有语法错误并且不会引发异常，
    但是并没有作为文档绑定到函数名称对应的 `var`，
    仅仅是成为了函数内容的一部分。
    <sup>[[link](#docstring-after-fn-name)]</sup>

    ```Clojure
    ;; good
    (defn foo
      "docstring"
      [x]
      (bar x))

    ;; bad
    (defn foo [x]
      "docstring"
      (bar x))
    ```

*   <a name="oneline-short-fn"></a>
    对于内容较短的函数，可以选择单行定义。
    <sup>[[link](#oneline-short-fn)]</sup>

    ```Clojure
    ;; good
    (defn foo [x]
      (bar x))

    ;; good for a small function body
    (defn foo [x] (bar x))

    ;; good for multi-arity functions
    (defn foo
      ([x] (bar x))
      ([x y]
       (if (predicate? x)
         (bar x)
         (baz x))))

    ;; bad
    (defn foo
      [x] (if (predicate? x)
            (bar x)
            (baz x)))
    ```

*   <a name="multiple-arity-indentation"></a>
    函数的多组定义要和对应的参数列表保持同层缩进。
    <sup>[[link](#multiple-arity-indentation)]</sup>

    ```Clojure
    ;; good
    (defn foo
      "I have two arities."
      ([x]
       (foo x 1))
      ([x y]
       (+ x y)))

    ;; bad - extra indentation
    (defn foo
      "I have two arities."
      ([x]
        (foo x 1))
      ([x y]
        (+ x y)))
    ```

*   <a name="multiple-arity-order"></a>
    函数的多组定义要按照参数的个数由少到多的顺序。
    通常的情况是，多元数函数的 K 个参数定义完整实现了函数的功能，
    N < K 的 N 个参数定义会部分应用 N 个参数去调用 K 个参数的实现，
    N > K 的 N 个参数定义会通过变长参数，提供一种 `fold` 实现。
    <sup>[[link](#multiple-arity-order)]</sup>

    ```Clojure
    ;; good - it's easy to scan for the nth arity
    (defn foo
      "I have two arities."
      ([x]
       (foo x 1))
      ([x y]
       (+ x y)))

    ;; okay - the other arities are applications of the two-arity
    (defn foo
      "I have two arities."
      ([x y]
       (+ x y))
      ([x]
       (foo x 1))
      ([x y z & more]
       (reduce foo (foo x (foo y z)) more)))

    ;; bad - unordered for no apparent reason
    (defn foo
      ([x] 1)
      ([x y z] (foo x (foo y z)))
      ([x y] (+ x y))
      ([w x y z & more] (reduce foo (foo w (foo x (foo y z))) more)))
    ```

*   <a name="align-docstring-lines"></a>
    缩进多行 `docstring` 的每一行。
    <sup>[[link](#align-docstring-lines)]</sup>

    ```Clojure
    ;; good
    (defn foo
      "Hello there. This is
      a multi-line docstring."
      []
      (bar))

    ;; bad
    (defn foo
      "Hello there. This is
    a multi-line docstring."
      []
      (bar))
    ```

*   <a name="crlf"></a>
    使用 Unix 风格的换行符。
    （*BSD/Solaris/Linux/OS X 系统的用户不需担心，Windows 用户则要格外小心。）
    <sup>[[link](#crlf)]</sup>

    * 如果你使用 Git，可用下面这个配置来保护你的项目不被 Windows 的换行符干扰：

    ```bash
    bash$ git config --global core.autocrlf true
    ```

*   <a name="bracket-spacing"></a>
    如果段落后边紧跟着左括号 (`(`, `{` 和
    `[`) 或者前面紧跟着右括号 (`)`, `}` and `]`),
    使用空格进行分隔。 相反地, 左括号的右边和右括号的左边忽略空格。
    <sup>[[link](#bracket-spacing)]</sup>

    ```Clojure
    ;; good
    (foo (bar baz) quux)

    ;; bad
    (foo(bar baz)quux)
    (foo ( bar baz ) quux)
    ```

> 语法糖会导致分号癌。 <br/>
> -- Alan Perlis

* <a name="no-commas-for-seq-literals"></a>
  集合字面量的元素之间，不要使用逗号。
  <sup>[[link](#no-commas-for-seq-literals)]</sup>

    ```Clojure
    ;; good
    [1 2 3]
    (1 2 3)

    ;; bad
    [1, 2, 3]
    (1, 2, 3)
    ```

* <a name="opt-commas-in-map-literals"></a>
  在 `map` 字面量中适当的使用括号，可以提高 `map` 的可读性。
  <sup>[[link](#opt-commas-in-map-literals)]</sup>

    ```Clojure
    ;; good
    {:name "Bruce Wayne" :alter-ego "Batman"}

    ;; good and arguably a bit more readable
    {:name "Bruce Wayne"
     :alter-ego "Batman"}

    ;; good and arguably more compact
    {:name "Bruce Wayne", :alter-ego "Batman"}
    ```

* <a name="gather-trailing-parens"></a>
  尾部的括号保持在同一行。
  <sup>[[link](#gather-trailing-parens)]</sup>

    ```Clojure
    ;; good; single line
    (when something
      (something-else))

    ;; bad; distinct lines
    (when something
      (something-else)
    )
    ```

* <a name="empty-lines-between-top-level-forms"></a>
  顶级 `form` 之间使用空行分隔。
  <sup>[[link](#empty-lines-between-top-level-forms)]</sup>

    ```Clojure
    ;; good
    (def x ...)

    (defn foo ...)

    ;; bad
    (def x ...)
    (defn foo ...)
    ```

    这个规则的一个例外是，将相关的 `def` 放在一起。

    ```Clojure
    ;; good
    (def min-rows 10)
    (def max-rows 20)
    (def min-cols 15)
    (def max-cols 30)
    ```

* <a name="no-blank-lines-within-def-forms"></a>
  函数或者宏的定义中不要有空行，一个例外的情况是，
  使用空行指示出成对结构的分组，例如 `let`, `cond`。
  <sup>[[link](#no-blank-lines-within-def-forms)]</sup>

* <a name="80-character-limits"></a>
  每行尽量避免超过80个字符。
  <sup>[[link](#80-character-limits)]</sup>

* <a name="no-trailing-whitespace"></a>
  避免尾部空白符。
  <sup>[[link](#no-trailing-whitespace)]</sup>

* <a name="one-file-per-namespace"></a>
  每一文件使用单独的命名空间。
  <sup>[[link](#one-file-per-namespace)]</sup>

* <a name="comprehensive-ns-declaration"></a>
  使用一个全面的 `ns` `form` 来定义命名空间，
  包含 `refer`, `require`, `import`，并且按照前面的顺序。
  <sup>[[link](#comprehensive-ns-declaration)]</sup>

    ```Clojure
    (ns examples.ns
      (:refer-clojure :exclude [next replace remove])
      (:require [clojure.string :as s :refer [blank?]]
                [clojure.set :as set]
                [clojure.java.shell :as sh])
      (:import java.util.Date
               java.text.SimpleDateFormat
               [java.util.concurrent Executors
                                     LinkedBlockingQueue]))
    ```

* <a name="prefer-require-over-use"></a>
  在 `ns` `form` 中，使用 `:require :as` 优于 `:require :refer`，
  `:require :refer` 优于 `:refer :all`，不建议使用 `:use`。
  <sup>[[link](#prefer-require-over-use)]</sup>

    ```Clojure
    ;; good
    (ns examples.ns
      (:require [clojure.zip :as zip]))

    ;; good
    (ns examples.ns
      (:require [clojure.zip :refer [lefts rights]))

    ;; acceptable as warranted
    (ns examples.ns
      (:require [clojure.zip :refer :all]))

    ;; bad
    (ns examples.ns
      (:use clojure.zip))
    ```

* <a name="no-single-segment-namespaces"></a>
  避免单段命名空间。
  <sup>[[link](#no-single-segment-namespaces)]</sup>

    ```Clojure
    ;; good
    (ns example.ns)

    ;; bad
    (ns example)
    ```

* <a name="namespaces-with-5-segments-max"></a>
  避免使用过长的命名空间 (例如，超过5段的命名) 。
  <sup>[[link](#namespaces-with-5-segments-max)]</sup>

* <a name="10-loc-per-fn-limit"></a>
  函数的定义避免超过 10 行 (LOC)。大多数情况下，函数的定义应该少于 5 行 (LOC) 。
  <sup>[[link](#10-loc-per-fn-limit)]</sup>

* <a name="4-positional-fn-params-limit"></a>
  避免超过 3 个或者 4 个位置参数的参数列表。
  <sup>[[link](#4-positional-fn-params-limit)]</sup>

* <a name="forward-references"></a>
  避免向前引用。向前引用在某些情况下是必要的，但是这些情况在实践中微乎其微。
  <sup>[[link](#forward-references)]</sup>

## 语法

* <a name="ns-fns-only-in-repl"></a>
  避免使用 `require` 和 `refer` 等命名空间操作函数，
  在 `REPL` 的环境之外，这些函数是完全没有必要的。
  <sup>[[link](#ns-fns-only-in-repl)]</sup>

* <a name="declare"></a>
  必要时使用 `declare` 声明向前引用。
  <sup>[[link](#declare)]</sup>

* <a name="higher-order-fns"></a>
  倾向于使用类似 `map` 和 `loop/recur` 等高阶函数。
  <sup>[[link](#higher-order-fns)]</sup>

* <a name="pre-post-conditions"></a>
  倾向于使用前置和后置条件进行函数的检查。
  <sup>[[link](#pre-post-conditions)]</sup>

    ```Clojure
    ;; good
    (defn foo [x]
      {:pre [(pos? x)]}
      (bar x))

    ;; bad
    (defn foo [x]
      (if (pos? x)
        (bar x)
        (throw (IllegalArgumentException. "x must be a positive number!")))
    ```

* <a name="dont-def-vars-inside-fns"></a>
  不要在函数内定义 `var` 。
  <sup>[[link](#dont-def-vars-inside-fns)]</sup>

    ```Clojure
    ;; very bad
    (defn foo []
      (def x 5)
      ...)
    ```

* <a name="dont-shadow-clojure-core"></a>
  避免局部绑定覆盖 `clojure.core` 中的命名。
  <sup>[[link](#dont-shadow-clojure-core)]</sup>

    ```Clojure
    ;; bad - you're forced to use clojure.core/map fully qualified inside
    (defn foo [map]
      ...)
    ```

* <a name="alter-var"></a>
  使用 `alter-var-root` 代替 `def` 修改 `var` 的值。
  <sup>[[link]](#alter-var)</sup>

    ```Clojure
    ;; good
    (def thing 1) ; value of thing is now 1
    ; do some stuff with thing
    (alter-var-root #'thing (constantly nil)) ; value of thing is now nil

    ;; bad
    (def thing 1)
    ; do some stuff with thing
    (def thing nil)
    ; value of thing is now nil
    ```

* <a name="nil-punning"></a>
  使用 `seq` 作为终止条件去测试序列是否为空 (这种技术通常被称为 *nil punning*)。
  <sup>[[link](#nil-punning)]</sup>

    ```Clojure
    ;; good
    (defn print-seq [s]
      (when (seq s)
        (prn (first s))
        (recur (rest s))))

    ;; bad
    (defn print-seq [s]
      (when-not (empty? s)
        (prn (first s))
        (recur (rest s))))
    ```

* <a name="to-vector"></a>
  当你需要将序列 (`sequence`) 转换为矢量 (`vector`) 时，使用 `vec` 优于 `into`。
  <sup>[[link](#to-vector)]</sup>

    ```Clojure
    ;; good
    (vec some-seq)

    ;; bad
    (into [] some-seq)
    ```

* <a name="when-instead-of-single-branch-if"></a>
  使用 `when` 代替 `(if ... (do ...))`。
  <sup>[[link](#when-instead-of-single-branch-if)]</sup>

    ```Clojure
    ;; good
    (when pred
      (foo)
      (bar))

    ;; bad
    (if pred
      (do
        (foo)
        (bar)))
    ```

* <a name="if-let"></a>
  使用 `if-let` 代替 `let` + `if`。
  <sup>[[link](#if-let)]</sup>

    ```Clojure
    ;; good
    (if-let [result (foo x)]
      (something-with result)
      (something-else))

    ;; bad
    (let [result (foo x)]
      (if result
        (something-with result)
        (something-else)))
    ```

* <a name="when-let"></a>
  使用 `when-let` 代替 `let` + `when`。
  <sup>[[link](#when-let)]</sup>

    ```Clojure
    ;; good
    (when-let [result (foo x)]
      (do-something-with result)
      (do-something-more-with result))

    ;; bad
    (let [result (foo x)]
      (when result
        (do-something-with result)
        (do-something-more-with result)))
    ```

* <a name="if-not"></a>
  使用 `if-not` 代替 `(if (not ...) ...)`。
  <sup>[[link](#if-not)]</sup>

    ```Clojure
    ;; good
    (if-not pred
      (foo))

    ;; bad
    (if (not pred)
      (foo))
    ```

* <a name="when-not"></a>
  使用 `when-not` 代替 `(when (not ...) ...)`。
  <sup>[[link](#when-not)]</sup>

    ```Clojure
    ;; good
    (when-not pred
      (foo)
      (bar))

    ;; bad
    (when (not pred)
      (foo)
      (bar))
    ```

* <a name="when-not-instead-of-single-branch-if-not"></a>

  使用 `when-not` 代替 `(if-not ... (do ...))`。
  <sup>[[link](#when-not-instead-of-single-branch-if-not)]</sup>

    ```Clojure
    ;; good
    (when-not pred
      (foo)
      (bar))

    ;; bad
    (if-not pred
      (do
        (foo)
        (bar)))
    ```

* <a name="not-equal"></a>

  使用 `not=` 代替 `(not (= ...))`。
  <sup>[[link](#not-equal)]</sup>

    ```Clojure
    ;; good
    (not= foo bar)

    ;; bad
    (not (= foo bar))
    ```

* <a name="printf"></a>

  使用 `printf` 代替 `(print (format) ...)`。
  <sup>[[link](#printf)]</sup>

    ```Clojure
    ;; good
    (printf "Hello, %s!\n" name)

    ;; ok
    (println (format "Hello, %s!" name))
    ```

* <a name="multiple-arity-of-gt-and-ls-fns"></a>

  当进行比较的时候，记住，Clojure 的函数，例如 `<`, `>` 等，可以接受多个参数。
  <sup>[[link](#multiple-arity-of-gt-and-ls-fns)]</sup>

    ```Clojure
    ;; good
    (< 5 x 10)

    ;; bad
    (and (> x 5) (< x 10))
    ```

* <a name="single-param-fn-literal"></a>

  当函数字面量只有一个参数的时候，使用 `%` 优于 `%1`。
  <sup>[[link](#single-param-fn-literal)]</sup>

    ```Clojure
    ;; good
    #(Math/round %)

    ;; bad
    #(Math/round %1)
    ```

* <a name="multiple-params-fn-literal"></a>
  当函数字面量多于一个参数的时候，使用 `%1` 优于 `%`。

  <sup>[[link](#multiple-params-fn-literal)]</sup>

    ```Clojure
    ;; good
    #(Math/pow %1 %2)

    ;; bad
    #(Math/pow % %2)
    ```

* <a name="no-useless-anonymous-fns"></a>

  在非必要的情况下，不要把函数包裹在匿名函数中。
  <sup>[[link](#no-useless-anonymous-fns)]</sup>

    ```Clojure
    ;; good
    (filter even? (range 1 10))

    ;; bad
    (filter #(even? %) (range 1 10))
    ```

* <a name="no-multiple-forms-fn-literals"></a>

  当函数的定义多于一个 `form` 时，不要使用函数字面量。
  <sup>[[link](#no-multiple-forms-fn-literals)]</sup>

    ```Clojure
    ;; good
    (fn [x]
      (println x)
      (* x 2))

    ;; bad (you need an explicit do form)
    #(do (println %)
         (* % 2))
    ```

* <a name="complement"></a>

  倾向使用 `complement` 而不是匿名函数。
  <sup>[[link](#complement)]</sup>

    ```Clojure
    ;; good
    (filter (complement some-pred?) coll)

    ;; bad
    (filter #(not (some-pred? %)) coll)
    ```

    如果反向谓词作为一个独立函数存在时（例如，`event?` 和 `odd?`），
    这条规则可以忽略。

* <a name="comp"></a>

  利用 `comp` 让代码变得简洁。
  <sup>[[link](#comp)]</sup>

    ```Clojure
    ;; Assuming `(:require [clojure.string :as str])`...

    ;; good
    (map #(str/capitalize (str/trim %)) ["top " " test "])

    ;; better
    (map (comp str/capitalize str/trim) ["top " " test "])
    ```

* <a name="partial"></a>

  利用 `partial` 让代码变得简洁。
  <sup>[[link](#partial)]</sup>

    ```Clojure
    ;; good
    (map #(+ 5 %) (range 1 10))

    ;; (arguably) better
    (map (partial + 5) (range 1 10))
    ```

* <a name="threading-macros"></a>

  当 `form` 深度嵌套式，使用 `threading`宏 `->` (`thread-first`)
  和 `->>` (`thread-last`)。
  <sup>[[link](#threading-macros)]</sup>

    ```Clojure
    ;; good
    (-> [1 2 3]
        reverse
        (conj 4)
        prn)

    ;; not as good
    (prn (conj (reverse [1 2 3])
               4))

    ;; good
    (->> (range 1 10)
         (filter even?)
         (map (partial * 2)))

    ;; not as good
    (map (partial * 2)
         (filter even? (range 1 10)))
    ```

* <a name="else-keyword-in-cond"></a>

  在 `cond` 中使用 `:else` 捕获所有没有匹配的表达式。
  <sup>[[link](#else-keyword-in-cond)]</sup>

    ```Clojure
    ;; good
    (cond
      (< n 0) "negative"
      (> n 0) "positive"
      :else "zero"))

    ;; bad
    (cond
      (< n 0) "negative"
      (> n 0) "positive"
      true "zero"))
    ```

* <a name="condp"></a>

  当谓词或表达式没有变化时，使用 `condp` 优于 `cond`。
  <sup>[[link](#condp)]</sup>

    ```Clojure
    ;; good
    (cond
      (= x 10) :ten
      (= x 20) :twenty
      (= x 30) :thirty
      :else :dunno)

    ;; much better
    (condp = x
      10 :ten
      20 :twenty
      30 :thirty
      :dunno)
    ```

* <a name="case"></a>

  当测试表达式在编译时是常量时，使用 `case` 优于 `cond` 和 `condp`。
  <sup>[[link](#case)]</sup>

    ```Clojure
    ;; good
    (cond
      (= x 10) :ten
      (= x 20) :twenty
      (= x 30) :forty
      :else :dunno)

    ;; better
    (condp = x
      10 :ten
      20 :twenty
      30 :forty
      :dunno)

    ;; best
    (case x
      10 :ten
      20 :twenty
      30 :forty
      :dunno)
    ```

* <a name="shor-forms-in-cond"></a>

  在 `cond` 及相关的宏中，使用简短的 `form`，
  否则应该使用注释或者空行进行分组来进行视觉上的提示。
  <sup>[[link](#shor-forms-in-cond)]</sup>

    ```Clojure
    ;; good
    (cond
      (test1) (action1)
      (test2) (action2)
      :else   (default-action))

    ;; ok-ish
    (cond
      ;; test case 1
      (test1)
      (long-function-name-which-requires-a-new-line
        (complicated-sub-form
          (-> 'which-spans multiple-lines)))

      ;; test case 2
      (test2)
      (another-very-long-function-name
        (yet-another-sub-form
          (-> 'which-spans multiple-lines)))

      :else
      (the-fall-through-default-case
        (which-also-spans 'multiple
                          'lines)))
    ```

* <a name="set-as-predicate"></a>

  适当使用 `set` 作为谓词。
  <sup>[[link](#set-as-predicate)]</sup>

    ```Clojure
    ;; good
    (remove #{1} [0 1 2 3 4 5])

    ;; bad
    (remove #(= % 1) [0 1 2 3 4 5])

    ;; good
    (count (filter #{\a \e \i \o \u} "mary had a little lamb"))

    ;; bad
    (count (filter #(or (= % \a)
                        (= % \e)
                        (= % \i)
                        (= % \o)
                        (= % \u))
                   "mary had a little lamb"))
    ```

* <a name="inc-and-dec"></a>

  使用 `(inc x)` 和 `(dec x)` 代替 `(+ x 1)` 和 `(- x 1)`。
  <sup>[[link](#inc-and-dec)]</sup>

* <a name="pos-and-neg"></a>

  使用 `(pos? x)`, `(neg? x)` 和 `(zero? x)` 代替 `(> x 0)`, `(< x 0)` 和 `(= x 0)`。
  <sup>[[link](#pos-and-neg)]</sup>

* <a name="list-star-instead-of-nested-cons"></a>

  使用 `list*` 代替一系列嵌套 `cons` 调用。
  <sup>[[link](#list-star-instead-of-nested-cons)]</sup>

    ```Clojure
    ;; good
    (list* 1 2 3 [4 5])

    ;; bad
    (cons 1 (cons 2 (cons 3 [4 5])))
    ```

* <a name="sugared-java-interop"></a>
  使用 `java` 语法糖 `form`。
  <sup>[[link](#sugared-java-interop)]</sup>

    ```Clojure
    ;;; object creation
    ;; good
    (java.util.ArrayList. 100)

    ;; bad
    (new java.util.ArrayList 100)

    ;;; static method invocation
    ;; good
    (Math/pow 2 10)

    ;; bad
    (. Math pow 2 10)

    ;;; instance method invocation
    ;; good
    (.substring "hello" 1 3)

    ;; bad
    (. "hello" substring 1 3)

    ;;; static field access
    ;; good
    Integer/MAX_VALUE

    ;; bad
    (. Integer MAX_VALUE)

    ;;; instance field access
    ;; good
    (.someField some-object)

    ;; bad
    (. some-object someField)
    ```

* <a name="compact-metadata-notation-for-true-flags"></a>
  当 `metadata` 槽中的元素仅仅是键为 `keyword`，
  值为布尔值 `true`的键值对时，使用 `metadata` 的简写形式。
  <sup>[[link](#compact-metadata-notation-for-true-flags)]</sup>

    ```Clojure
    ;; good
    (def ^:private a 5)

    ;; bad
    (def ^{:private true} a 5)
    ```

* <a name="private"></a>
  指明代码中的私有部分。
  <sup>[[link](#private)]</sup>

    ```Clojure
    ;; good
    (defn- private-fun [] ...)

    (def ^:private private-var ...)

    ;; bad
    (defn private-fun [] ...) ; not private at all

    (defn ^:private private-fun [] ...) ; overly verbose

    (def private-var ...) ; not private at all
    ```

* <a name="access-private-var"></a>
  通过 `@#'some.ns/var` 形式的 `form` 访问私有 `var` (例如，进行测试) 。
  <sup>[[link](#access-private-var)]</sup>

* <a name="attach-metadata-carefully"></a>
  注意 `metadata` 的正确附加对象。
  <sup>[[link](#attach-metadata-carefully)]</sup>

    ```Clojure
    ;; we attach the metadata to the var referenced by `a`
    (def ^:private a {})
    (meta a) ;=> nil
    (meta #'a) ;=> {:private true}

    ;; we attach the metadata to the empty hash-map value
    (def a ^:private {})
    (meta a) ;=> {:private true}
    (meta #'a) ;=> nil
    ```

## 命名

> 程序设计的真正难题是替事物命名以及缓存失效。<br/>
>
> -- Phil Karlton

*   <a name="ns-naming-schemas"></a>
    使用下面两种模式对命名空间进行命名:
    <sup>[[link](#ns-naming-schemas)]</sup>

    * `project.module`
    * `organization.project.module`

*   <a name="lisp-case-ns"></a>
    命名空间片段使用 `Lisp` 小写 (`lisp-case`) (例如，`bruce.project-euler`)。
    <sup>[[link](#lisp-case-ns)]</sup>

*   <a name="lisp-case"></a>
    函数，变量名使用 `Lisp` 小写 (`lisp-case`)。
    <sup>[[link](#lisp-case)]</sup>

    ```Clojure
    ;; good
    (def some-var ...)
    (defn some-fun ...)

    ;; bad
    (def someVar ...)
    (defn somefun ...)
    (def some_fun ...)
    ```

*   <a name="CamelCase-for-protocols-records-structs-and-types"></a>
    协议 (`protocols`)，纪录 (`records`)，结构 (`structs`)，
    和类型 (`types`)， 使用驼峰式大小写（`CamelCase`）
    （HTTP、RFC、XML 等首字母缩写应该仍旧保持大写形式）。
    <sup>[[link](#CamelCase-for-protocols-records-structs-and-types)]</sup>

*   <a name="pred-with-question-mark"></a>
    谓词方法的名称 (返回布尔值的方法) 应当以问号结尾 (例如，`even?`)。
    <sup>[[link](#pred-with-question-mark)]</sup>

    ```Clojure
    ;; good
    (defn palindrome? ...)

    ;; bad
    (defn palindrome-p ...) ; Common Lisp style
    (defn is-palindrome ...) ; Java style
    ```

*   <a name="changing-state-fns-with-exclamation-mark"></a>
    `STM` 事务中非安全的方法或宏，名字以感叹号结尾 (例如，`reset!` ) 。
    <sup>[[link](#changing-state-fns-with-exclamation-mark)]</sup>

*   <a name="arrow-instead-of-to"></a>
    转换方法的名称中，使用 `->` 代替 `to`。
    <sup>[[link](#arrow-instead-of-to)]</sup>

    ```Clojure
    ;; good
    (defn f->c ...)

    ;; not so good
    (defn f-to-c ...)
    ```

*   <a name="earmuffs-for-dynamic-vars"></a>
    使用 `*earmuffs*` 为要重新绑定事物命名 (例如，动态全局变量)。
    <sup>[[link](#earmuffs-for-dynamic-vars)]</sup>

    ```Clojure
    ;; good
    (def ^:dynamic *a* 10)

    ;; bad
    (def ^:dynamic a 10)
    ```

*   <a name="dont-flag-constants"></a>
    不要为常量使用特殊记号，除了特殊情况，所有的事物都应该假定为一个常量。
    <sup>[[link](#dont-flag-constants)]</sup>

*   <a name="underscore-for-unused-bindings"></a>
    对于忽略没有被马上使用的解构对象 (`destructure targets`)
    和形式参数 (`formal argument`)，使用 `_` 进行命名。
    <sup>[[link](#underscore-for-unused-bindings)]</sup>

    ```Clojure
    ;; good
    (let [[a b _ c] [1 2 3 4]]
      (println a b c))

    (dotimes [_ 3]
      (println "Hello!"))

    ;; bad
    (let [[a b c d] [1 2 3 4]]
      (println a b d))

    (dotimes [i 3]
      (println "Hello!"))
    ```

*   <a name="idiomatic-names"></a>
    根据 `clojure.core` 中示例的惯例，例如，`pred` 和 `coll`，进行命名。
    <sup>[[link](#idiomatic-names)]</sup>

    *   在函数中:
        * `f`, `g`, `h` - 函数输入
        * `n` - 整数输入，通常代表大小
        * `index`, `i` - 整数索引
        * `x`, `y` - 数字
        * `xs` - 序列
        * `m` - 映射
        * `s` - 字符串输入
        * `re` - 正则表达式
        * `coll` - 集合
        * `pred` - 谓词闭包
        * `& more` - 变长参数
        * `xf` - xform, a transducer

        在宏中:
        * `expr` - 表达式
        * `body` - 宏定义
        * `binding` - 宏绑定矢量

## 集合

> 一百个函数去操作一个数据结构要优于十个函数去操作十个数据结构。  <br/>
> -- Alan J. Perlis

* <a name="avoid-lists"></a>

  避免使用列表 (`lists`) 保存常用数据结构 (除非真的需要列表) 。
  <sup>[[link](#avoid-lists)]</sup>

* <a name="keywords-for-hash-keys"></a>

  倾向于使用关键字类型 (`keywords`) 作为哈希键。
  <sup>[[link](#keywords-for-hash-keys)]</sup>

    ```Clojure
    ;; good
    {:name "Bruce" :age 30}

    ;; bad
    {"name" "Bruce" "age" 30}
    ```

* <a name="literal-col-syntax"></a>
  倾向于恰当的使用字面量集合语法，然而，定义集合 (`set`) 的时候，
  如果值是编译时常量，只使用字面量语法。
  <sup>[[link](#literal-col-syntax)]</sup>

    ```Clojure
    ;; good
    [1 2 3]
    #{1 2 3}
    (hash-set (func1) (func2)) ; values determined at runtime

    ;; bad
    (vector 1 2 3)
    (hash-set 1 2 3)
    #{(func1) (func2)} ; will throw runtime exception if (func1) = (func2)
    ```

* <a name="avoid-index-based-coll-access"></a>
  尽可能避免通过索引 (`index`) 获取集合 (`collection`) 的元素。
  <sup>[[link](#avoid-index-based-coll-access)]</sup>

* <a name="keywords-as-fn-to-get-map-values"></a>
  在适用的情况下，优先使用键作为函数，来获取映射 (`maps`) 的值。
  <sup>[[link](#keywords-as-fn-to-get-map-values)]</sup>

    ```Clojure
    (def m {:name "Bruce" :age 30})

    ;; good
    (:name m)

    ;; more verbose than necessary
    (get m :name)

    ;; bad - susceptible to NullPointerException
    (m :name)
    ```

* <a name="colls-as-fns"></a>
  利用大多数集合是其元素的函数这一事实。
  <sup>[[link](#colls-as-fns)]</sup>

    ```Clojure
    ;; good
    (filter #{\a \e \o \i \u} "this is a test")

    ;; bad - too ugly to share
    ```

* <a name="keywords-as-fns"></a>
  利用关键字可以用作集合的函数这一事实。
  <sup>[[link](#keywords-as-fns)]</sup>

    ```Clojure
    ((juxt :a :b) {:a "ala" :b "bala"})
    ```

* <a name="avoid-transient-colls"></a>
  避免使用短暂集合 (`transient collections`)，除非代码对性能有要求。
  <sup>[[link](#avoid-transient-colls)]</sup>

* <a name="avoid-java-colls"></a>
  避免使用 `Java` 集合。
  <sup>[[link](#avoid-java-colls)]</sup>

* <a name="avoid-java-arrays"></a>
  避免使用 `Java` 数组，除了和 `Java` 互操作的场景，
  或者大量原始类型 (`primitive types`) 操作的性能关键部分。
  <sup>[[link](#avoid-java-arrays)]</sup>

## 可变性

### Refs (引用)

* <a name="refs-io-macro"></a>
  倾向于将所有的 `I/O` 操作包裹到 `io!` 宏 (`macro`) 中，
  以防不小心在事务 (`transaction`) 中调用产生意外。
  <sup>[[link](#refs-io-macro)]</sup>

* <a name="refs-avoid-ref-set"></a>
  在任何情况下避免使用 `ref-set`。
  <sup>[[link](#refs-avoid-ref-set)]</sup>

    ```Clojure
    (def r (ref 0))

    ;; good
    (dosync (alter r + 5))

    ;; bad
    (dosync (ref-set r 5))
    ```

* <a name="refs-small-transactions"></a>
  尽量保持事务 (`transactions`) 小而紧凑 (事务中的逻辑) 。
  <sup>[[link](#refs-small-transactions)]</sup>

* <a name="refs-avoid-short-long-transactions-with-same-ref"></a>
  避免在同时在短时间事务和长时间事务中操作相同的 `Ref`。
  <sup>[[link](#refs-avoid-short-long-transactions-with-same-ref)]</sup>

### Agents (代理)

* <a name="agents-send"></a>
  只在 `CPU` 绑定 (`CPU bound`)，
  或者没有 `I/O` 阻塞 (`block on I/O`) 的时候使用 `send`。
  <sup>[[link](#agents-send)]</sup>

* <a name="agents-send-off"></a>
  对于可能阻塞，睡眠的操作，使用 `send-off`，或者以其它的方式配合线程。
  <sup>[[link](#agents-send-off)]</sup>

### Atoms (原子)

* <a name="atoms-no-update-within-transactions"></a>
  避免在 `STM` 事务 (`STM transactions`) 中更新原子 (`atom`)。
  <sup>[[link](#atoms-no-update-within-transactions)]</sup>

* <a name="atoms-prefer-swap-over-reset"></a>
  尽可能使用 `swap!` 而不是 `reset!`。
  <sup>[[link](#atoms-prefer-swap-over-reset)]</sup>

    ```Clojure
    (def a (atom 0))

    ;; good
    (swap! a + 5)

    ;; not as good
    (reset! a 5)
    ```

## 字符串

* <a name="prefer-clojure-string-over-interop"></a>
  使用 `clojure.string` 中的函数操作字符串，优于 `Java` 互操作
  (`Java interop`) 或者自定义函数。
  <sup>[[link](#prefer-clojure-string-over-interop)]</sup>

    ```Clojure
    ;; good
    (clojure.string/upper-case "bruce")

    ;; bad
    (.toUpperCase "bruce")
    ```

## 异常

* <a name="reuse-existing-exception-types"></a>
  重用现有的异常类型。地道的 `Clojure` 代码 &mdash; 当抛出异常时 &mdash;
  会抛出标准异常类型
  (例如， `java.lang.IllegalArgumentException`,
  `java.lang.UnsupportedOperationException`,
  `java.lang.IllegalStateException`, `java.io.IOException`)。
  <sup>[[link](#reuse-existing-exception-types)]</sup>

* <a name="prefer-with-open-over-finally"></a>
  使用 `with-open` 优于 `finally`。
  <sup>[[link](#prefer-with-open-over-finally)]</sup>

## 宏

* <a name="dont-write-macro-if-fn-will-do"></a>
  在函数可以实现功能的情况下不要使用宏。
  <sup>[[link](#dont-write-macro-if-fn-will-do)]</sup>

* <a name="write-macro-usage-before-writing-the-macro"></a>
  编写宏之前首先编写宏的用例。
  <sup>[[link](#write-macro-usage-before-writing-the-macro)]</sup>

* <a name="break-complicated-macros"></a>
  尽可能将复杂的宏拆为较小的函数。
  <sup>[[link](#break-complicated-macros)]</sup>

* <a name="macros-as-syntactic-sugar"></a>
  宏的核心应该是一个纯函数，宏通常仅仅提供了语法糖。这样可以提高组合性。
  <sup>[[link](#macros-as-syntactic-sugar)]</sup>

* <a name="syntax-quoted-forms"></a>
  使用语法引用 `forms` (`syntax-quoted forms`) 优于自己手动构建列表。
  <sup>[[link](#syntax-quoted-forms)]</sup>

## 注释

> 良好的代码自身就是最佳的文档。当你要添加一个注释时，
> 扪心自问，“如何改善代码让它不需要注释？” 改善代码，再写相应文档使之更清楚。<br/>
> -- Steve McConnell

* <a name="self-documenting-code"></a>
  努力让代码变得尽可能地自注释。
  <sup>[[link](#self-documenting-code)]</sup>

* <a name="four-semicolons-for-heading-comments"></a>
  头部注释至少保留四个分号。
  <sup>[[link](#four-semicolons-for-heading-comments)]</sup>

* <a name="three-semicolons-for-top-level-comments"></a>
  顶级注释至少保留三个分号。
  <sup>[[link](#three-semicolons-for-top-level-comments)]</sup>

* <a name="two-semicolons-for-code-fragment"></a>
  代码片段注释保留两个分号，并且和代码片段对齐。
  <sup>[[link](#two-semicolons-for-code-fragment)]</sup>

* <a name="one-semicolon-for-margin-comments"></a>
  单行尾部注释保留一个分号。
  <sup>[[link](#one-semicolon-for-margin-comments)]</sup>

* <a name="semicolon-space"></a>
  分号和注释正文之间总是至少保留一个空格。
  <sup>[[link](#semicolon-space)]</sup>

    ```Clojure
    ;;;; Frob Grovel

    ;;; This section of code has some important implications:
    ;;;   1. Foo.
    ;;;   2. Bar.
    ;;;   3. Baz.

    (defn fnord [zarquon]
      ;; If zob, then veeblefitz.
      (quux zot
            mumble             ; Zibblefrotz.
            frotz))
    ```

* <a name="english-syntax"></a>
  注释超过一个单词时，句首字母应当大写，并在语句停顿或结尾处使用标点符号。句号后添加
  [空格](http://en.wikipedia.org/wiki/Sentence_spacing)。
  <sup>[[link](#english-syntax)]</sup>

* <a name="no-superfluous-comments"></a>
  避免无谓的注释。
  <sup>[[link](#no-superfluous-comments)]</sup>

    ```Clojure
    ;; bad
    (inc counter) ; increments counter by one
    ```

* <a name="comment-upkeep"></a>
  及时更新注释。过时的注释比没有注释还要糟糕。
  <sup>[[link](#comment-upkeep)]</sup>

* <a name="dash-underscore-reader-macro"></a>
  当需要注释掉一个特定的 `form` 的时候，使用读取宏 `#_` 优于普通的注释。
  <sup>[[link](#dash-underscore-reader-macro)]</sup>

    ```Clojure
    ;; good
    (+ foo #_(bar x) delta)

    ;; bad
    (+ foo
       ;; (bar x)
       delta)
    ```

> 好的代码就像是好的笑话 —— 它不需要解释。<br/>
> -- Russ Olsen

* <a name="refactor-dont-comment"></a>
  避免替烂代码编写注释。重构它们使其变得一目了然。
  （要么做，要么不做，不要只是试试看。——Yoda）
  <sup>[[link](#refactor-dont-comment)]</sup>

### 注解

* <a name="annotate-above"></a>
  注解通常应该直接写在相关代码之前那行。
  <sup>[[link](#annotate-above)]</sup>

* <a name="annotate-keywords"></a>
  注解关键字后面，跟着一个冒号及空格，接着是描述问题的文本。
  <sup>[[link](#annotate-keywords)]</sup>

* <a name="indent-annotations"></a>
  如果需要用多行来描述问题，后续行要和第一行保持相同的锁进。
  <sup>[[link](#indent-annotations)]</sup>

* <a name="sing-and-date-annotations"></a>
  为了注解的相关信息得到验证，应该使用名字的缩写和日期进行标注。
  <sup>[[link](#sing-and-date-annotations)]</sup>

    ```Clojure
    (defn some-fun
      []
      ;; FIXME: This has crashed occasionally since v1.2.3. It may
      ;;        be related to the BarBazUtil upgrade. (xz 13-1-31)
      (baz))
    ```

* <a name="rare-eol-annotations"></a>
  当问题是显而易见时，任何文档都是多余的，
  注解应当放在有问题的那行末尾且不带任何多余说明。这个用法应该算是例外而不是规则。
  <sup>[[link](#rare-eol-annotations)]</sup>

    ```Clojure
    (defn bar
      []
      (sleep 100)) ; OPTIMIZE
    ```

* <a name="todo"></a>
  使用 `TODO` 标记应当加入的特征与功能。
  <sup>[[link](#todo)]</sup>

* <a name="fixme"></a>
  使用 `FIXME` 标记需要修复的代码。
  <sup>[[link](#fixme)]</sup>

* <a name="optimize"></a>
  使用 `OPTIMIZE` 标记可能引发性能问题的低效代码。
  performance problems.
  <sup>[[link](#optimize)]</sup>

* <a name="hack"></a>
  使用 `HACK` 标记代码异味，即那些应当被重构的可疑编码习惯。
  <sup>[[link](#hack)]</sup>

* <a name="review"></a>
  使用 `REVIEW` 标记需要确认与编码意图是否一致的可疑代码。
  比如，`REVIEW: Are we sure this is how the client does X currently?`。
  <sup>[[link](#review)]</sup>

* <a name="document-annotations"></a>
  适当情况下，可以自行定制其他注解关键字，
  但别忘记在项目的 `README` 或类似文档中予以说明。
  <sup>[[link](#document-annotations)]</sup>

## 经验

* <a name="be-functional"></a>
  用函数式的方式编码，只在显而易见的情况下使用可变性 (`mutation`) 。
  <sup>[[link](#be-functional)]</sup>

* <a name="be-consistent"></a>
  保持一致。在理想的情况下，和风格指南保持一致。
  <sup>[[link](#be-consistent)]</sup>

* <a name="common-sense"></a>
  利用常识。
  <sup>[[link](#common-sense)]</sup>

## 工具

下面是一些 Clojure 社区创建的工具，为你写出地道的 Clojure 助一臂之力。

* [Slamhound](https://github.com/technomancy/slamhound)
  是一个可以根据你现有代码，生成恰当的 `ns` 声明的工具。

* [kibit](https://github.com/jonase/kibit)
  是一个 Clojure 静态分析器，
  使用  [core.logic](https://github.com/clojure/core.logic) 通过搜索代码模式，
  来发现现有代码中函数或宏的更好的实现。

## 测试

*  <a name="test-directory-structure"></a>
   测试位于独立的文件夹中， 通常是 `test/yourproject/` (而不是 `src/yourproject/`)。
   构建工具保证了在需要它们的上下文中是可用的，
   大多数模版会自动完成这些功能。
   <sup>[[link](#test-directory-structure)]</sup>

*  <a name="test-ns-naming"></a>
   命名空间要命名为 `yourproject.something-test`， 对于的文件通常为
   `test/yourproject/something_test.clj` (或者 `.cljc`, `cljs`)。
   <sup>[[link](#test-ns-naming)]</sup>

*  <a name="test-naming"></a> 使用 `clojure.test` 时，
   使用 `deftest` 定义测试，并且命名为 `something-test`，例如：

   ```clojure
   ;; good
   (deftest something-test ...)

   ;; bad
   (deftest something-tests ...)
   (deftest test-something ...)
   (deftest something ...)
   ```

   <sup>[[link](#test-naming)]</sup>

# 贡献

这份指南中的任何规则都不是一成不变的。
我渴望和任何一位对 Clojure 风格指南的伙伴一起工作，
最终创造一份对整个 Clojure 社区都大有裨益的资源。

欢迎发起讨论或提交一个带有改进性质的更新请求。在此提前感谢你的帮助！

你也可以通过 [gittip](https://www.gittip.com/bbatsov) 对此项目提供财务方面的支持。

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

# 授权

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
本指南基于
[Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US) 授权许可。

# 口耳相传

一份社区驱动的风格指南，如果没多少人知道，
对一个社区来说就没有多少用处。微博转发这份指南，
分享给你的朋友或同事。我们得到的每个评价、建议或意见都可以让这份指南变得更好一点。
而我们想要拥有最好的指南，不是吗？

共勉之，<br/>
[Bozhidar](https://twitter.com/bbatsov)
