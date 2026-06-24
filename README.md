# carp-reader

in which Carp gets parsed into a structured `Form` AST. Built on
[`carpentry-org/parsec`](https://github.com/carpentry-org/parsec).

## Install

```clojure
(load "git@github.com:carpentry-org/carp-reader@0.3.6")
```

## Example

```clojure
(match (Reader.parse "; greeting\n(defn hello [name] (println &name))")
  (Result.Success forms)
    (for [i 0 (Array.length &forms)]
      (println &(Form.str (Box.peek (Array.unsafe-nth &forms i)))))
  (Result.Error e)
    (IO.errorln &(Parser.format-error &e)))
```

`Reader.parse` returns `(Result (Array (Box Form)) ParseErr)`.

## Form shape

```clojure
(deftype Form
  (Itg        [Int])
  (Byt        [Byte])
  (Lng        [Long])
  (Flt        [Float])
  (Dbl        [Double])
  (Bol        [Bool])
  (Chr        [Char])
  (Str        [String])
  (Pat        [String])                ; #"..." pattern literal
  (Sym        [(Array String)])        ; ["Foo" "Bar" "baz"]
  (Lst        [(Array (Box Form))])    ; (...)
  (Arr        [(Array (Box Form))])    ; [...]
  (StaticArr  [(Array (Box Form))])    ; $[...]
  (Dict       [(Array (Box Form))])    ; {...}
  (Cmt        [String]))               ; ; line comment
```

Reader macros lower to `(Lst [(Sym [name]) form])`:

| Source | Form |
|---|---|
| `&x` | `(ref x)` |
| `~x` | `(deref x)` |
| `@x` | `(copy x)` |
| `'x` | `(quote x)` |
| `` `x `` | `(quasiquote x)` |
| `%x` | `(unquote x)` |
| `%@x` | `(unquote-splicing x)` |

<hr/>

Have fun!
