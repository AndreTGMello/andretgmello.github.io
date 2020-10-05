---
layout: post
title:  "Conheça Hy!"
date:   2020-10-04 18:00:00 -0300
categories: lisp,hy
lang: pt
lang-ref: say-hi-to-hy
---

Já quis desfrutar da velocidade do Python unido da legibilidade do Lisp/Clojure?

Se você respondeu Sim, então seus problemas acabaram! Eu lhe apresento a
linguagem [Hy](https://docs.hylang.org/en/stable/).

Hy é um dialeto Lisp que se traduz para a Abstract Syntax Tree to Python.

Maneiro, hein? Logo eu devo criar um Jupyter Notebook simples explorando o que
essa linguagem tem a oferecer.

Por ora, deixo aqui uma implementação do clássico problema FizzBuzz, pra que
você possa ter uma ideia de como a linguagem se parece primeiro.

Ah, mas um detalhe. Minhas duas soluções são um pouco "inusitadas".

Na primeira eu usei um esquema de "Divisão e Conquista" (mais ou menos).

```hy
;; The only way to tackle such a complex problem: Divide and Conquer (sort of).
;; You can call fizzbuzz with debug = True if it helps you grasp the solution.
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

Já a segunda é um código Hy com notação de tipo e paralelo. E é muito mais limpo
que o primeiro. Isso mostra como é trivial paralelizar um código que
embaraçosamente paralelo (*embarassingly parallel*) usando conceitos da
programação funcional. Mas isso não quer dizer que você deva sair implementando
isso pra qualquer coisa.

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

Você pode encontrar essas e outras implementações interessantes no repositório:

[](https://github.com/NLDev/Hacktoberfest-2020-FizzBuzz)

Envie a sua também! :)

PS: Os créditos para a piadinha no começo do texto vai para
[@hlissner](https://github.com/hlissner). Aliás, se você usa (ou está pensando
em usar) Emacs, dê uma olhada na sua distribuição de Emacs, o [Doom
Emacs](https://github.com/hlissner/doom-emacs). Tenho a usado por algum tempo e
recomendo fortemente!

