# Say Hi to Hy!

Ever wanted the speed of Python plus the readability of Lisp/Clojure?

Look no further! I present to you the Hy language.

Hy is a Lisp dialect that translates itself into the Python Abstract Syntax Tree.

Pretty cool huh? Soon enough I should come up with a simple notebooks for exploring what this language has to offer.

For now, I'll leave you with the classic FizzBuzz exercise, so you can get a sense of what the language looks like first.

Oh but there's a twist. I implemented it in two "unusual" ways.

In the first one I went for a "Divide and Conquer" (sort of) approach.

```hy
;; The only way to tackle such a complex problem: Divide and Conquer (sort of).
;; You can call fizzbuzz with dbug = True if it helps you grasp the solution.
;; Author: @atgmello

(require [hy.contrib.walk [let]])
(import [math [floor]])

(defn fizz-buzz [until &optional [debug False]]
  (setv result (list (range 0 until)))
  (defn devide-fizz-conquer-buzz [ini end]
    (if (= ini end)
        (let [res (list)]
          (do
            (if debug
                (print (.format "Leaf reached.\nValue: {}\n" ini)))
            (if (zero? (% ini 3))
                (.append res "Fizz"))
            (if (zero? (% ini 5))
                (.append res "Buzz"))
            (if (zero? (len res))
                (.append res (str ini)))
            (setv (get result (- ini 1)) (str.join "" res))
            (if debug
                (print (.format "Results so far:\n{}\n" result)))))
        (let [pivot (floor (/ (+ ini end) 2))]
           (do
             (if debug (print
                         (.format
                           "Dividing:\nIni: {} Pivot: {} End: {}\n"
                           ini pivot end)))
             (if debug (print
                         (.format
                           "First recursion:\nIni: {}. End: {}\n"
                           ini pivot end)))
             (devide-fizz-conquer-buzz ini pivot)
             (if debug (print
                         (.format
                           "Second recursion:\nIni: {}. End: {}\n"
                           (+ pivot 1) end)))
             (devide-fizz-conquer-buzz (+ pivot 1) end))))
    (return result))
  (for [n (devide-fizz-conquer-buzz  1 until)] (print n)))

(fizz-buzz 100)
```

The second one is a type annotated, parallel code. And it's actually a lot cleaner than the first one.
This goes to show how parallelizing embarrassingly parallel code is trivial using some functional programming concepts.
Doesn't mean you should always do it, though.

```hy
;; Type annotated, parallel implementation of the fizzbuzz problem.
;; Author: @atgmello

(import [concurrent.futures [ProcessPoolExecutor]])
(import [os [cpu-count]])

(defn fizzbuzz? [^int n] (zero? (+ (% n 3) (% n 5))))
(defn fizz? [^int n] (zero? (% n 3)))
(defn buzz? [^int n] (zero? (% n 5)))

(defn check-which [^int n] (cond [(fizzbuzz? n) "FizzBuzz"]
                                 [(fizz? n) "Fizz"]
                                 [(buzz? n) "Buzz"]
                                 [True n]))

(defn fizzbuzz [^int n]
  (setv ^ProcessPoolExecutor executor
        (ProcessPoolExecutor :max-workers (cpu-count)))
  (setv ^list res (.map executor check-which (range 1 (+ n 1))))
  (.shutdown executor)
  (for [r res] (print r)))

(fizzbuzz 100)
```

You can check these and other crazy FizzBuzz implementations over at:

https://github.com/NLDev/Hacktoberfest-2020-FizzBuzz

Submit yours as well! :)

PS: The credits for the pun in the first paragraph goes to [@hlissner](https://github.com/hlissner).
Check out his Emacs distribution, [Doom Emacs](https://github.com/hlissner/doom-emacs). I highly recommend it!
