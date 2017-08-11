<p align="left"><img src="http://takenobu-hs.github.io/downloads/images/haskell-logo-s.png"/></p>

command-line Haskell with 'ghc -e'
==================================


Introduction
------------

The [GHC](https://www.haskell.org/ghc/) (Glasgow Haskell Compiler) is a enormous and powerful compiler of [Haskell](https://www.haskell.org/).  
However, it is also a handy tool like sed and awk. You can use GHC in your daily work :)  
For example, it can be embedded in a shell script, it can be executed immediately on your terminal.


One-liners mode (expression evaluation mode) of GHC
---------------------------------------------------

The GHC could evaluate a single expression on your shell ([see also](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/using.html#expression-evaluation-mode)):

```
$ ghc -e expr
```


Cooking recipes
---------------

Do you want a calculator? OK, here:
```
$ ghc -e '1+2'
3
```

Do you need sequence data?, please:
```
$ ghc -e '[1..10]'
[1,2,3,4,5,6,7,8,9,10]
```

List comprehension is OK ([see also](http://learnyouahaskell.com/starting-out#im-a-list-comprehension)):
```
$ ghc -e '[x^2 | x <- [1..10]]'
[1,4,9,16,25,36,49,64,81,100]
```

If you want to output per line ([see also](http://learnyouahaskell.com/input-and-output#hello-world)):
```
$ ghc -e 'mapM_ print [1..3]'
1
2
3
```


Actually, you can also use input here:
```
$ ghc -e 'getLine'
ABC
"ABC"
```

Of course you can also input and output:
```
$ ghc -e 'getLine >>= putStrLn'
ABC
ABC
```

You can also use the do notation to describe the above:
```
$ ghc -e 'do {x <- getLine; putStrLn x}'
ABC
ABC
```

Since you can input and output, you can also create a filter:
```
$ ghc -e 'interact $ unlines . map ("hello " ++) . lines'
John
hello John
Mike
hello Mike
```

Numeric filter is also OK:
```
$ cat test.dat    # data file for example
1
2
3
```

```
$ cat test.dat | ghc -e 'interact $ unlines . map (show . (*2) . (read::String -> Int)) . lines'
2
4
6
```

Pipeline between GHC and GHC :)
```
$ ghc -e 'mapM_ print [1..3]' | ghc -e 'interact $ unlines . map ("hello " ++) . lines'
hello 1
hello 2
hello 3
```


By the way, don't you want pair data? That's OK, too:
```
$ ghc -e "zip [1..3] ['A'..]"
[(1,'A'),(2,'B'),(3,'C')]
```

Not a pair, but a combination? OK easy ([see also](http://learnyouahaskell.com/functors-applicative-functors-and-monoids#applicative-functors)):
```
$ ghc -e "(,) <$> [1..3] <*> ['A'..'C']"
[(1,'A'),(1,'B'),(1,'C'),(2,'A'),(2,'B'),(2,'C'),(3,'A'),(3,'B'),(3,'C')]
```

Combination of character string and number:
```
$ ghc -e '(++) <$> ["R", "G", "B"] <*> map show [0..3]'
["R0","R1","R2","R3","G0","G1","G2","G3","B0","B1","B2","B3"]
```

Want to generate date strings? :
```
$ ghc -e '(++) <$> ["2016", "2017", "2018"] <*> ["Jan", "Feb", "Mar"]'
["2016Jan","2016Feb","2016Mar","2017Jan","2017Feb","2017Mar","2018Jan","2018Feb","2018Mar"]

```

Let's insert hyphens:
```
$ ghc -e '(\x y -> x ++ "-" ++ y) <$> ["2016", "2017", "2018"] <*> ["Jan", "Feb", "Mar"]'
["2016-Jan","2016-Feb","2016-Mar","2017-Jan","2017-Feb","2017-Mar","2018-Jan","2018-Feb","2018-Mar"]
```

Do you want to make CSV data?, OK:
```
$ ghc -e 'mapM_ putStrLn $ (\x y -> x ++ "," ++ y) <$> ["2016", "2017", "2018"] <*> ["Jan", "Feb", "Mar"]'
2016,Jan
2016,Feb
2016,Mar
2017,Jan
2017,Feb
2017,Mar
2018,Jan
2018,Feb
2018,Mar
```


Well, do you want to know the type in the expression? You can use typed holes with `_` ([see also](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/glasgow_exts.html#typed-holes)) :
```
$ ghc -e 'foldl _ 0 [1..5]'

<interactive>:0:7: error:
    • Found hole: _ :: b -> Integer -> b
      Where: ‘b’ is a rigid type variable bound by
               the inferred type of it :: Num b => b at <interactive>:0:1-16
    • In the first argument of ‘foldl’, namely ‘_’
      In the expression: foldl _ 0 [1 .. 5]
      In an equation for ‘it’: it = foldl _ 0 [1 .. 5]
    • Relevant bindings include it :: b (bound at <interactive>:0:1)
```

Actually, you can use the ghci command `:t`(:type) if you want to know the type ([see also](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/ghci.html#ghci-cmd-:type)):
```
$ ghc -e ':t foldl'
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b
```

If it is GHC 8.2 or later, you can use `:t +d`(:type +d) command for a more simple representation ([see also](https://downloads.haskell.org/%7Eghc/latest/docs/html/users_guide/ghci.html#ghci-cmd-:type%20+d%20%E2%9F%A8expression%E2%9F%A9)):
```
$ ghc -e ':t +d foldl'
foldl :: (b -> a -> b) -> b -> [a] -> b
```

You can also use the `:i`(:info) command:
```
$ ghc -e ':i foldl'
class Foldable (t :: * -> *) where
  ...
  foldl :: (b -> a -> b) -> b -> t a -> b
  ...
  	-- Defined in ‘Data.Foldable’
```

You can also know the kind by `:k`(:kind) command ([see also](http://learnyouahaskell.com/making-our-own-types-and-typeclasses#kinds-and-some-type-foo)):
```
$ ghc -e ':k Maybe'
Maybe :: * -> *
```


By the way, you can also use quasi-quarts with shell variables! Tiny templating :)
```
$ NUM=5
$ ghc -e "[1..$NUM]"
[1,2,3,4,5]
```

You can pass data easily:
```
$ X1="[1..3]"
$ X2="['A'..'C']"
$ ghc -e "zip $X1 $X2"
[(1,'A'),(2,'B'),(3,'C')]
```


Other miscellaneous recipes
---------------------------

```
$ NUMS="[1..3]"
$ ghc -e "sum $NUMS"
6
```

```
$ cat test.dat | ghc -e "lines <$> getContents"
["1","2","3"]
```

```
$ cat test.dat | ghc -e "(sum . map read .lines) <$> getContents"
6
```

```
cat test.dat | ghc -e "(filter (>=2) . map read .lines) <$> getContents"
[2,3]
```

```
$ ghc -e "let x = 1; y = 2 in x+y"
3
```

```
$ ghc -e 'Text.Printf.printf "%s %d\n" "abc" 1'
abc 1
```

```
$ cat .ghci
import Text.Printf

$ ghc -e 'printf "%s %d\n" "abc" 1'
abc
```

```
$ ghc -e 'replicate 5 "hello"'
["hello","hello","hello","hello","hello"]
```

Let's enjoy cooking!


References
----------

 * [GHC User’s Guide](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/)
 * [Haskell.org](https://www.haskell.org/)
 * [Haskell 2010 Language Report](https://www.haskell.org/onlinereport/haskell2010/)
 * [The Glasgow Haskell Compiler](https://www.haskell.org/ghc/)
 * [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)

