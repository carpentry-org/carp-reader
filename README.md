# carp-reader

in which Carp gets parsed into a structured `Form` AST. Built on
[`carpentry-org/parsec`](https://github.com/carpentry-org/parsec).

## Install

```clojure
(load "git@github.com:carpentry-org/carp-reader@0.5.0")
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

## Editing documents

`Reader.read-syntax` returns a `ReaderDocument` for one exact UTF-8 buffer,
including incomplete or invalid input:

```clojure
(let [document (Reader.read-syntax "(let [x 1] x")]
  (match (ReaderDocument.status &document)
    (ReaderStatus.Complete) (IO.println "complete")
    (ReaderStatus.Incomplete) (IO.println "needs more input")
    (ReaderStatus.Invalid) (IO.println "invalid")))
```

The document retains source, nodes, and issues. A `ReaderNode` carries its
start and end `Info`, a kind, and children. Groups retain delimiter punctuation;
reader macros retain their prefix and operand. Whitespace and comments remain
in the tree. Leaves partition every source byte, with additional zero-width
`Missing` nodes where syntax is absent. `Reader.node-category` returns lexical
categories chosen by the actual atom parsers.

Each `ReaderIssue` distinguishes incomplete from invalid syntax and retains
the ordinary `ParseErr`. Definite invalid syntax takes precedence when a
document also has missing syntax.

`Reader.forms` lowers a document only when it has no issues. It returns the
same located forms as `Reader.parse`, including reader-macro lowering.
`Reader.parse` now reads this tree and calls `Reader.forms`; strict parsing and
editing share one grammar. Recovery never makes invented syntax executable.

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
