# carp-reader

in which Carp gets parsed into a structured `Form` AST. Built on
[`carpentry-org/parsec`](https://github.com/carpentry-org/parsec).

## Install

```clojure
(load "git@github.com:carpentry-org/carp-reader@0.4.0")
```

## Example

```clojure
(match (Reader.parse "; greeting\n(defn hello [name] (println &name))")
  (Result.Success forms)
    (for [i 0 (Array.length &forms)]
      (IO.println &(Form.str (Located.form (Box.peek (Array.unsafe-nth &forms i))))))
  (Result.Error e)
    (IO.errorln &(Parser.format-error &e)))
```

`Reader.parse` returns `(Result (Array (Box Located)) ParseErr)`.

## Located

Every node the parser emits — top-level and nested alike — is a `Located`,
not a bare `Form`:

```clojure
(deftype Located
  [form Form
   info Info    ; position of the form's first byte
   end  Info])  ; position just past its last byte

(deftype Info
  [pos  Int     ; byte offset
   line Int
   col  Int])
```

`Located.form` unwraps it, so it is the accessor you reach for in almost any
use of this library:

```clojure
(let [l (Box.peek (Array.unsafe-nth &forms 0))]
  (println* (Form.str (Located.form l))
            " at line " @(Info.line (Located.info l))
            ", column " @(Info.col (Located.info l))))
```

`Located.str` is shorthand for `(Form.str (Located.form l))` when you do not
need the position. Nodes that reader macros synthesize rather than read from
source carry `Info.synthetic`, whose fields are all `0`.

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
  (Lst        [(Array (Box Located))]) ; (...)
  (Arr        [(Array (Box Located))]) ; [...]
  (StaticArr  [(Array (Box Located))]) ; $[...]
  (Dict       [(Array (Box Located))]) ; {...}
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
