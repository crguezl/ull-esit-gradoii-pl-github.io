# PEGJS

PEG.js is a parser generator for JavaScript that produces parsers.

It generates a parser from a Parsing Expression Grammar describing a
language.

We can specify what the parser returns (using semantic actions on
matched parts of the input).

## Installation

To use the pegjs command, install globally:

    $ npm install -g pegjs

To use the JavaScript API, install locally:

    $ npm install pegjs

To use it from the browser, Install it using [Bower](https://bower.io/):

```
$ bower install pegjs
```

## Command Line Options

```
[~/.../pegjs/examples(master)]$ pegjs --help
Usage: pegjs [options] [--] [<input_file>]

Options:
      --allowed-start-rules <rules>  comma-separated list of rules the generated
                                     parser will be allowed to start parsing
                                     from (default: the first rule in the
                                     grammar)
      --cache                        make generated parser cache results
  -d, --dependency <dependency>      use specified dependency (can be specified
                                     multiple times)
  -e, --export-var <variable>        name of a global variable into which the
                                     parser object is assigned to when no module
                                     loader is detected
      --extra-options <options>      additional options (in JSON format) to pass
                                     to peg.generate
      --extra-options-file <file>    file with additional options (in JSON
                                     format) to pass to peg.generate
      --format <format>              format of the generated parser: amd,
                                     commonjs, globals, umd (default: commonjs)
  -h, --help                         print help and exit
  -O, --optimize <goal>              select optimization for speed or size
                                     (default: speed)
  -o, --output <file>                output file
      --plugin <plugin>              use a specified plugin (can be specified
                                     multiple times)
      --trace                        enable tracing in generated parser
  -v, --version                      print version information and exit
```

## Simple Example

```
~/.../pegjs/examples(master)]$ node
Welcome to Node.js v12.10.0.
Type ".help" for more information.
```
```js
> PEG = require("pegjs")
{
  VERSION: '0.10.0',
  GrammarError: [Function: GrammarError],
  parser: {
    SyntaxError: [Function: peg$SyntaxError] { buildMessage: [Function] },
    parse: [Function: peg$parse]
  },
  compiler: {
    visitor: { build: [Function: build] },
    passes: { check: [Object], transform: [Object], generate: [Object] },
    compile: [Function: compile]
  },
  generate: [Function: generate]
}
> parser = PEG.generate("start = ('a' / 'b')+")
{
  SyntaxError: [Function: peg$SyntaxError] { buildMessage: [Function] },
  parse: [Function: peg$parse]
}
```

On the top level, the grammar consists of _rules_ (in our example, there is only one). Each rule has 

- a _name_ (e.g. `start`) that identifies the rule, and 
- a _parsing expression_ (e.g. `('a' / 'b')+`) that defines a pattern to match against the input text and 

An expression matching a literal string produces a JavaScript string containing matched text (`'a'` matches `a`).

In PEG.js an expression like $$expression_1 / expression_2 / ... / expression_n$$
tries to match the first expression, if it does not succeed, try the second one, etc. Return the match result of the first successfully matched expression. If no expression matches, the match fails.

Also an expression like `expression +`
matches one or more repetitions of the expression and **return their match results in an array**. The matching is greedy, i.e. the parser tries to match the expression as many times as possible. Unlike in regular expressions, **there is no backtracking**.

Since an expression matching repeated occurrence of some subexpression produces a JavaScript array with all the matches, the expression `('a' / 'b')+` matches `abb` and has as match result the array `['a', 'b', 'b']`.

Using the generated parser is simple — just call its `parse` method and
pass an input string as a parameter.

```js
> parser.parse("abba")
[ 'a', 'b', 'b', 'a' ]
```

The method will return

-   a parse result (the exact value depends on the grammar used to build
    the parser) or

-   throw an exception if the input is invalid.

    The exception will contain 
    `location`, 
    `expected`,
    `found` and `message` 
    properties with more details about the error.

If an expression successfully matches a part of the text when running the generated parser, it produces a **match result**, which is a JavaScript value. For example:

**The match results** propagate through the rules when the **rule names** are used in **expressions**, up to the **start rule**. The generated parser returns start rule's match result when parsing is successful.

That is why in our example:

```js
> parser = PEG.generate("start = ('a' / 'b')+")
```

The **match result** for input `abba` is `[ 'a', 'b', 'b', 'a' ]`:

```
> parser.parse("abba")
[ 'a', 'b', 'b', 'a' ]
```

## The OR and Backtracking

PEG parsers don't backtrack like regexps and other recursive-descent parsers or Prolog do. 
Rather, when confronted with a choice, a PEG parser will try every option until one succeeds.
**Once one succeeds, it will commit to it** no matter how the rule was invoked.

El siguiente ejemplo ilustra el `/` y el backtracking en los PEGs:

```javascript
[~/.../pegjs/examples(master)]$ cat backtracking.js
const PEG = require ("pegjs");
const grammar1 = `
start = "test" / "test ;"
`;
let parser = PEG.generate(grammar1);
let input = 'test';
console.log("OK: "+parser.parse(input)); // OK: test
try {
  // This input will not be accepted
  const input = 'test ;';
  console.log(parser.parse(input));
}
catch (e) { // Expected end of input but " " found
  console.log(e.message);
}
const grammar2 = `
start = "test" !" ;" / "test ;"
`;
parser = PEG.generate(grammar2);
input = 'test ;';
console.log("OK: "+parser.parse(input)); // OK: test ;
```

The expression `! ";"` is like a regexp negative lookahead: 
tries  to match `";"`. If the match does not succeed, just
return `undefined` and does not advance the parser position, otherwise
considers the match failed giving opportunity for the next choice `"test ;"` to work.

## PEGs versus Gramáticas {#subsection:pegvsgrammars}

Una gramática y un PEG con las mismas reglas no definen el mismo
lenguaje. Véase este ejemplo:

```
[~/.../pegjs/examples(master)]$ cat grammarvspegjs.js
```
```js
const PEG = require('pegjs');
const grammar = `
  A =  B 'c'
  B = 'b' / 'b' 'a'
`;
const parser = PEG.generate(grammar);
let r = parser.parse("bc");
console.log("r = " + r);
try {
  r = parser.parse("bac");
  console.log("r = " + r);
} catch(e) { console.log(e.message); }
```
```
[~/.../pegjs/examples(master)]$ node grammarvspegjs.js
r = b,c
Expected "c" but "a" found.
```

Obsérvese que la correspondiente gramática genera el lenguaje:

    { 'bc', 'bac' }

Mientras que el PEG acepta el lenguaje `'bc'`.

En efecto, probemos con una herramienta como [Jison](https://zaa.ch/jison/) que permite procesar gramáticas:

```
[~/.../pegjs/examples(master)]$ cat grammarvspeg.jison
```
```
%lex
%%
.                 { return yytext; }
/lex

%%
A:  B 'c'
;
B: 'b' | 'b' 'a'
;
```
Cuando parseamos la entrada `bac`con esta gramática obtenemos:
```
[~/.../pegjs/examples(master)]$ jison grammarvspeg.jison
[~/.../pegjs/examples(master)]$ ls -ltr |tail -n 1
-rw-r--r--   1 casiano  staff   20615 10 may 09:18 grammarvspeg.js
[~/.../pegjs/examples(master)]$ ./use.js grammarvspeg bac
Processing <bac>
true
```

## Options

You can tweak parser behavior by passing a second parameter with an
`options` object to the `parse` method.

The following options are supported:

`startRule`

Name of the rule to start parsing from.

`tracer`

Tracer to use. See [this example of use](tracer)

Parsers can also support their own custom options.

## allowedStartRules

Specifying `allowedStartRules` we can set the rules the parser will be allowed to start
parsing from (default: the first rule in the grammar).

```
[~/.../pegjs/examples(master)]$ cat allowedstartrules.js
```
```js
"use strict";
const util = require('util');
const PEG = require("pegjs");
const grammar = `
  a = 'hello' b
  b = 'world'
`;
console.log(grammar);

const parser = PEG.generate(grammar,{ allowedStartRules: ['a', 'b'] });

let r = parser.parse("helloworld", { startRule: 'a' });
console.log(r); // [ 'hello', 'world' ]

r = parser.parse("helloworld")
console.log(r); // [ 'hello', 'world' ]

r =  parser.parse("world", { startRule: 'b' })
console.log(r); // 'world'

try {
  r = parser.parse("world"); // Throws an exception
}
catch(e) {
  r = e;
}
console.log(`The error object:`);
console.log(util.inspect(r, {depth: null}));
/*
{
  message: 'Expected "hello" but "w" found.',
  expected: [ { type: 'literal', text: 'hello', ignoreCase: false } ],
  found: 'w',
  location: {
    start: { offset: 0, line: 1, column: 1 },
    end: { offset: 1, line: 1, column: 2 }
  },
  name: 'SyntaxError'
}
*/
```

## The error object properties

```
[~/.../pegjs/examples(master)]$ node allowedstartrules.js

  a = 'hello' b
  b = 'world'

[ 'hello', 'world' ]
[ 'hello', 'world' ]
world
The error object:
peg$SyntaxError: Expected "hello" but "w" found.
    at peg$buildStructuredError (eval at compile (/Users/casiano/local/src/javascript/PLgrado/pegjs/examples/node_modules/pegjs/lib/compiler/index.js:67:29), <anonymous>:277:14)
    at Object.peg$parse [as parse] (eval at compile (/Users/casiano/local/src/javascript/PLgrado/pegjs/examples/node_modules/pegjs/lib/compiler/index.js:67:29), <anonymous>:336:13)
    at Object.<anonymous> (/Users/casiano/local/src/javascript/PLgrado/pegjs/examples/allowedstartrules.js:22:14)
    at Module._compile (internal/modules/cjs/loader.js:936:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:947:10)
    at Module.load (internal/modules/cjs/loader.js:790:32)
    at Function.Module._load (internal/modules/cjs/loader.js:703:12)
    at Function.Module.runMain (internal/modules/cjs/loader.js:999:10)
    at internal/main/run_main_module.js:17:11 {
  message: 'Expected "hello" but "w" found.',
  expected: [ { type: 'literal', text: 'hello', ignoreCase: false } ],
  found: 'w',
  location: {
    start: { offset: 0, line: 1, column: 1 },
    end: { offset: 1, line: 1, column: 2 }
  },
  name: 'SyntaxError'
}
```

The error object `e` contains

-   message
-   expected,
-   found
-   location (`start` and `end` objects with offset, line and column)
-   name

and properties with more details about the error.

## The *output* Option of  generate

-   When `output` is set to `'parser'`, the method will return generated parser
    object; if set to `'source'`, it will return parser source code as a string

```js
> grammar = "a = 'hello' b\nb='world'"
"a = 'hello' b\nb='world'"
> parser = PEG.generate(grammar,{ output: "parser"})
{
  SyntaxError: [Function: peg$SyntaxError] { buildMessage: [Function] },
  parse: [Function: peg$parse]
}
> let parser = PEG.generate(grammar,{ output: "source"})
undefined
> typeof parser
'string'
> parser.substring(0,100)
'/*\n' +
  ' * Generated by PEG.js 0.10.0.\n' +
  ' *\n' +
  ' * http://pegjs.org/\n' +
  ' */\n' +
  '(function() {\n' +
  '  "use strict";\n' +
  '\n' +
  '  funct'
```

The default value is: `parser`.

## Plugins

La opción `plugins` o en línea de comando `plugin` indica que plugin se van a usar.
Véase [la lista de plugins](https://github.com/pegjs/pegjs/wiki/Plugins)

```coffee
[~/.../pegjs/examples(master)]$ cat plugin.coffee
#!/usr/bin/env coffee
PEG = require 'pegjs'
coffee = require 'pegjs-coffee-plugin'
grammar = """
a = 'hello' _ b { console.log 3; "hello world!" }
b = 'world'     { console.log 2 }
_ = [ \t]+      { console.log 1 }
"""
parser = PEG.generate grammar, plugins: [coffee]
r = parser.parse "hello world"
console.log("r = #{r}")
```
One special case of parser expression is a _parser action_ — a piece of JavaScript code inside curly braces (`{` and `}`) that takes **match results** of some of the the preceding expressions **and returns a JavaScript value**. This value is considered match result of the preceding expression (in other words, **the parser action is a match result transformer**).

La ejecución nos muestra además el orden de abajo - arriba y de
izquierda -derecha en la ejecución de las **parser actions** o  acciones semánticas:

```
[~/.../pegjs/examples(master)]$ npm i pegjs-coffee-plugin
[~/.../pegjs/examples(master)]$ npm i -g coffeescript
[~/.../pegjs/examples(master)]$ ./plugin.coffee
1
2
3
r = hello world!
```


## A Very Simple Calculator {#section:UnEjemploSencillo}

The PEG.js syntax is similar to JavaScript in that it is not
line-oriented and ignores whitespace between tokens.

You can also use JavaScript-style comments (`// ...` and `/* ... */`).

Let’s look at example grammar that recognizes simple arithmetic
expressions like `2*(3+4)`.

A parser generated from this grammar computes their values.

```
 [~/.../pegjs/examples(master)]$ cat arithmetics-with-minus.pegjs
```
```
/*
 * Classic example grammar, which recognizes simple arithmetic expressions like
 * "2*(3+4)". The parser generated from this grammar then computes their value.
 */

start
  = additive

additive
  = left:multiplicative PLUS right:additive { return left + right; }
  / left:multiplicative MINUS right:additive { return left - right; }
  / multiplicative

multiplicative
  = left:primary MULT right:multiplicative { return left * right; }
  / left:primary DIV right:multiplicative { return left / right; }
  / primary

primary
  = integer
  / LEFTPAR additive:additive RIGHTPAR { return additive; }

integer "integer"
  = NUMBER

_ = $[ \t\n\r]*

PLUS = _"+"_
MINUS = _"-"_
MULT = _"*"_
DIV = _"/"_
LEFTPAR = _"("_
RIGHTPAR = _")"_
NUMBER = _ digits:$[0-9]+ _ { return parseInt(digits, 10); }
```

On the top level, the grammar consists of _rules_ (in our example, there are five of them). Each rule has 
- a _name_ (e.g. `integer`) that identifies the rule, and 
- a _parsing expression_ (e.g. `digits:[0-9]+`) that defines a pattern to match against the input text and 
- possibly contains some JavaScript code that determines what happens when the pattern matches successfully (e.g. `{ return parseInt(digits.join(""), 10); }`). 

A rule can also contain _human-readable name_ that is used in error messages (in our example, only the `integer` rule has a human-readable name). 

The parsing starts at the first rule, which is also called the _start rule_.

* A rule name must be a JavaScript identifier. 
* It is followed by an equality sign `=` and a parsing expression.
*  If the rule has a human-readable name, it is written as a JavaScript string between the name and separating equality sign. 
* Rules need to be separated only by whitespace (their beginning is easily recognizable), but a semicolon (`;`) after the parsing expression is allowed.

In PEG.js the peg expression `$ expression` tries to match the `expression`. If the match succeeds, return the **matched text** instead of the match result.

That is why we use the `$`inside the rule:

```
_ = $[ \t\n\r]*
```

This way, instead of returning an array of individual whites, the rule returns the matched string of whites.

We do the same in:

```
$[0-9]+
```

And thus, instead of an array of digits we have the string with the digits, simplifying the parsing to `parseInt(digits, 10)`.

In our example there are many parser actions. Consider the action in expression 

`left:multiplicative PLUS right:additive { return left + right; }`

It takes the **match result** of the appearance of the rule `multiplicative`, which is provided in a variable named `left` an the **match result** associated with the appearance of the rule `additive`. They must be numbers and the action sets as **match result** 
of the rule the sum of the two numbers `{ return left + right; }`

In this example you see many examples of the syntax:

`label : expression`

The meaning is that it matches the expression and remembers its match result under given `label`.

-   The label must be a JavaScript identifier.
-   Labeled expressions are useful together with actions, where  saved match results can be accessed by action’s JavaScript code.

To produce the parser we compile it with `pegjs`:

```
[~/.../pegjs/examples(master)]$ pegjs arithmetics-with-minus.pegjs
[~/.../pegjs/examples(master)]$ ls -ltr | tail -1
-rw-r--r--   1 casiano  staff   18688  9 may 14:36 arithmetics-with-minus.js
```

The output file `arithmetics-with-minus.js` is a COMMON.js module that we can use
with a `require`:

```
[~/.../pegjs/examples(master)]$ cat use-arithmetics-with-minus.js
```
```js
var PEG = require("./arithmetics-with-minus.js");
var input = process.argv[2] || "5+3*2";
console.log(`Processing <${input}>`);
var r = PEG.parse(input);
console.log(r);
```
The execution gives:
```
[~/.../pegjs/examples(master)]$ node use-arithmetics-with-minus.js
Processing <5+3*2>
11
```

## Asociación Incorrecta para la Resta y la División

Una Gramática Independiente del Contexto se dice *Recursiva por la Izquierda*
cuando existe una derivación $$A \stackrel{*}{\Longrightarrow} A \alpha$$.

En particular, es *recursiva por la izquierda* si contiene una regla de
producción de la forma $$A \rightarrow A \alpha$$. En este caso se dice
que la recursión por la izquierda es directa.

Cuando la gramática es recursiva por la izquierda, un método de análisis recursivo descendente
como los de los PEGs no funciona. En ese caso, el procedimiento `A` asociado con
$$A$$ ciclaría para siempre sin llegar a consumir ningún terminal.

Es por eso que en el ejemplo hemos empezado escribiendo las reglas de la calculadora con
recursividad a derechas,

```
    additive
      = left:multiplicative PLUS right:additive { return left + right; }
      / left:multiplicative MINUS right:additive { return left - right; }
      / multiplicative
```

pero eso da lugar a árboles hundidos hacia la derecha y a una aplicación
de las reglas semánticas errónea:

```
[~/.../pegjs/examples(master)]$ node use-arithmetics-with-minus.js 5-3-2
Processing <5-3-2>
4
```

## Initializers

The first rule can be preceded by an _initializer_ — a piece of JavaScript code in curly braces (`{` and `}`). 

1. This code is executed before the generated parser starts parsing. 
2. All variables and functions defined in the initializer are accessible in rule actions and semantic predicates. 
3. The code inside the initializer can access options passed to the parser using the `options` variable. 
4. Curly braces in the initializer code must be balanced. 

## Solving the Associativity Problem

Here is an example of use of initializers that also solves the associativity problem 
posed in the section [a very simple calculator](#section:UnEjemploSencillo):

```
[~/.../pegjs/examples(master)]$ cat simple_reduce.pegjs
```
```
{
  function reduce(left, right) {
    return right.reduce((sum, [op, num]) => eval(`sum ${op}= num`), left);
  }
}

sum     = left:product right:($[+-] product)* { return reduce(left, right); }
product = left:value   right:($[*/] value)*   { return reduce(left, right); }
value   = number:$[0-9]+                      { return parseInt(number,10); }
        / '(' sum:sum ')'                     { return sum; }
```

We have written a helper script `use.js` to load  a parser and execute it on a
given input:

```
[~/.../pegjs/examples(master)]$ cat use.js
```
```js
#!/usr/bin/env node
const [peg, input] = process.argv.splice(2);
const parse = require('./'+peg).parse;
console.log(`Processing <${input}>`);
const r = parse(input);
console.log(r);
```

Now we have an arithmetics calculator that correctly computes substractions and divisions:

```
[~/.../pegjs/examples(master)]$ ./use.js 'simple_reduce' 5-3-2
Processing <5-3-2>
0
```

## Passing Options to the Parser Initializer from Outside

In this example the variable `g `declared inside the initializer is visible 
in all the actions. The variable `options` contains the options argument 
used in the call to the parser:

```
[~/.../pegjs/examples(master)]$ cat initializer.js
```
```ruby
"use strict";
const PEG = require("pegjs");
const grammar = `
 {
   const util = require("util");
   const g = "visible variable";
   console.log("Inside Initializer! options = "+util.inspect(options));
 }
 start = 'a' { console.log('g = '+g); return 1; }
       /  &  { console.log('inside predicate: g = '+g); return true; } 'b' { return 2; }
`;

const parser = PEG.generate(grammar);
console.log("PARSING 'a'");
let r = parser.parse("a", { x: 'hello' });
console.log(r);
console.log("PARSING 'b'");
r = parser.parse("b");
console.log(r);
```
```
[~/.../pegjs/examples(master)]$ node initializer.js
PARSING 'a'
Inside Initializer! options = { x: 'hello' }
g = visible variable
1
PARSING 'b'
Inside Initializer! options = {}
inside predicate: g = visible variable
2
```

## The `i` option for "literal"

Appending `i` right after the literal makes the match case-insensitive:

```js
[~/.../pegjs/examples(master)]$ cat ignorecasejs.js
"use strict"
const PEG = require('pegjs');
const grammar = `start = a: 'a'i `;
const parser = PEG.generate(grammar);
let r = parser.parse('A');
console.log(r);
```

## DOT  `.`

Match exactly one character (including `\n`) and return it as a string:

```js
[~/.../pegjs/examples(master)]$ cat dotjs.js
const PEG = require('pegjs');
const grammar = 'start = a: ..';
const parser = PEG.generate(grammar);
const r = parser.parse("\n\t");
console.log(r);
[~/.../pegjs/examples(master)]$ node dotjs.js
[ '\n', '\t' ]
```

## Classes  `[characters]`

Match one character from a set and return it as a string.

The characters in the list can be escaped in exactly the same
way as in JavaScript string.

The list of characters can also contain ranges (e.g. `[a-z]`
means all lowercase letters).

Preceding the characters with `^` inverts the matched set (e.g.
`[^a-z]` means <span>*"all character but lowercase letters*</span>).

Appending `i` right after the class makes the match case-insensitive.

Example:

```js
[~/.../pegjs/examples(master)]$ cat regexpjs.js
const PEG = require('pegjs');
grammar = 'start = a: [aeiou\u2661]i . [^x-z] ';
parser = PEG.generate(grammar);
r = parser.parse('Abr');
console.log(r);
r = parser.parse('♡br');
console.log(r);
```
```
[~/.../pegjs/examples(master)]$ node regexpjs.js
[ 'A', 'b', 'r' ]
[ '♡', 'b', 'r' ]
```

## Looking Forward: AND an d NOT operators

`& expression`

Try to match the expression. If the match succeeds, just return `undefined` and do not advance the parser position, otherwise  consider the match failed.

`! expression`

Try to match the expression. If the match does not succeed, just
return `undefined` and do not advance the parser position, otherwise
consider the match failed.

## Parsing a Non Context Free Language

The language $$\{ a^n b^n c^n / n \in \mathcal{N} \}$$ can't be generated by 
a context free grammar.

```
[~/.../pegjs/examples(master)]$ cat anbncn.pegjs
/*
  The following parsing expression grammar describes the classic
  non-context-free language :
               { a^nb^nc^n / n >= 1 }
*/

S = &(A 'c') 'a'+ B:B !.  { return B; }
/* S = &A 'a'+ B:B !.   does not work since accepts abbcc */
A = 'a' A:A? 'b' { if (A) { return A+1; } else return 1; }
B = 'b' B:B? 'c' { if (B) { return B+1; } else return 1; }
```

```
[~/.../pegjs/examples(master)]$ pegjs anbncn.pegjs
[~/.../pegjs/examples(master)]$ ./use.js anbncn aabcc
Processing <aabcc>
Expected , or undefined but "a" found.
[~/.../pegjs/examples(master)]$ ./use.js anbncn aabbcc
Processing <aabbcc>
2
[~/.../pegjs/examples(master)]$ ./use.js anbncn aabccc
Processing <aabccc>
Expected , or undefined but "a" found.
```

See also the [old version of this example](anbncn)

## Not Predicate: Comentarios Anidados

The following recursive program matches Pascal-style nested comment
syntax:

    (* which can (* nest *) like this *)

```
[~/.../pegjs/examples(master)]$ cat pascal_comments.pegjs
/* Pascal nested comments */

P     =   prog:N+                       { return prog; }
N     =   chars:$(!Begin .)+            { return chars;}
        / C
C     = Begin chars:$T* End             { return "C: "+chars; }
T     =   C
        / !Begin !End char:.           { return char;}
Begin = '(*'
End   = '*)'
```

```js
$ cat use_pascal_comments.js 
var PEG = require("./pascal_comments.js");
var r = PEG.parse(
  "not bla bla (* pascal (* nested *) comment *)"+
  " pum pum (* another comment *)");
console.log(r);
```

```
[~/.../pegjs/examples(master)]$ node use_pascal_comments.js
[
  'not bla bla ',
  'C:  pascal (* nested *) comment ',
  ' pum pum ',
  'C:  another comment '
]
```

## Looking Forward: JavaScript Comments

Here is an example recognizing JavaScript whitespaces and comments:

```
[~/srcPLgrado/pegjs/examples(master)]$ cat notpredicate.pegjs 
__ = (whitespace / eol / comment)*

/* Modeled after ECMA-262, 5th ed., 7.4. */
comment "comment"
  = singleLineComment
  / multiLineComment

singleLineComment
  = "//" (!eolChar .)*   { return text(); }

multiLineComment
  = "/*" (!"*/" .)* "*/" { return text(); }


/* Modeled after ECMA-262, 5th ed., 7.3. */
eol "end of line"
  = "\n"
  / "\r\n"
  / "\r"
  / "\u2028"
  / "\u2029"

eolChar
  = [\n\r\u2028\u2029]

whitespace "whitespace"
  = [ \t\v\f\u00A0\uFEFF\u1680\u180E\u2000-\u200A\u202F\u205F\u3000]
```

Once it is compiled we can call it from our main program:

```js
[~/srcPLgrado/pegjs/examples(master)]$  cat mainnotpredicate.js 
var PEG = require("./notpredicate.js");
var input = process.argv[2] || "// one comment\n"+
                                "// another comment \t/\n"+
                                "/* a\n"+
                                "   third comment */";
console.log("\n*****\n"+input+"\n*****\n");
var r = PEG.parse(input);
console.log(r);
```

This is the output:

```
[~/srcPLgrado/pegjs/examples(master)]$ pegjs notpredicate.pegjs 
[~/srcPLgrado/pegjs/examples(master)]$ node mainnotpredicate.js 

*****
// one comment
// another comment      /
/* a
    third comment */
*****

[ '// one comment',
  '\n',
  '// another comment \t/',
  '\n',
  '/* a\n   third comment */' ]
```


## Predicate `& { predicate }`

The predicate is a piece of JavaScript code that is executed as if it was inside a function. It gets the match results of labeled expressions in preceding expression as its arguments. It should return some JavaScript value using the `return` statement. If the returned value evaluates to `true` in boolean context, just return `undefined` and do not consume any input; otherwise consider the match failed.

```
[~/.../pegjs/examples(master)]$ cat semantic_intermediajs.js
const PEG = require('pegjs');

const grammar = `
a = a:$'a'+
    & { console.log("acción intermedia. a = "+a); return true; }
    b:$'b'+ {
             console.log("acción final. b = "+b);
             return text();
          }
`;

parser = PEG.generate(grammar);
r = parser.parse("aabb");
console.log("r = " + r);
```

```
[~/.../pegjs/examples(master)]$ node semantic_intermediajs.js
acción intermedia. a = aa
acción final. b = bb
r = aabb
```

The code inside the predicate can access all variables and functions defined in the initializer at the beginning of the grammar.

The code inside the predicate can also access location information using the `location` function. It returns an object like this:

```js
{
  start: { offset: 23, line: 5, column: 6 },
  end:   { offset: 23, line: 5, column: 6 }
}
```

The `start` and `end` properties both refer to the current parse position. 

The `offset` property contains an offset as a zero-based index and `line` and `column` properties contain a line and a column as one-based indices.

The code inside the predicate can also access `options` passed to the parser using the options variable.

Note that curly braces in the predicate code must be balanced.

```
[~/.../pegjs/examples(master)]$ cat semantic_predicatejs.js
```
```js
let PEG, grammar, parser, r;
PEG = require('pegjs');
grammar = `
 {
   let util = require("util")
   let g = "visible variable"
   console.log("Inside Initializer! options = "+util.inspect(options))
 }
 start = 'a' {  console.log(g); return 1 }
       / c:'c' '\\n' &   {
                     console.log("inside predicate: g = "+g+" c = "+c);
                     console.log("options = "+util.inspect(options));
                     console.log("location = "+util.inspect(location()));
                     return true
                   } 'b' { return 2 }

`;
parser = PEG.generate(grammar);
r = parser.parse('a', { x: 'hello' });
console.log(r);
r = parser.parse("c\nb", { y: 'world' });
console.log(r);
```
Execution:
```
[~/.../pegjs/examples(master)]$ node semantic_predicatejs.js
Inside Initializer! options = { x: 'hello' }
visible variable
1
Inside Initializer! options = { y: 'world' }
inside predicate: g = visible variable c = c
options = { y: 'world' }
location = {
  start: { offset: 2, line: 2, column: 1 },
  end: { offset: 2, line: 2, column: 1 }
}
2
```


## The Dangling else: Asociando un else con su if mas cercano

The dangling else is a problem in computer programming in which an
optional `else clause` in an `If–then(–else)` statement results in
nested conditionals being ambiguous.

Formally, the reference context-free grammar of the language is
ambiguous, meaning there is more than one correct parse tree.

In many programming languages one may write conditionally executed code
in two forms:

the `if-then` form, and the `if-then-else` form – the `else` clause is
optional:

```pascal
                  if a then s
                  if a then s1 else s2
```

This gives rise to an ambiguity in interpretation when there are nested
statements, specifically whenever an `if-then` form appears as `s1` in
an `if-then-else` form:

```pascal
                  if a then if b then s else s2
```

In this example, `s` is unambiguously executed when `a` is `true` and
`b` is `true`, but one may interpret `s2` as being executed when `a` is
`false`

-   (thus attaching the `else` to the first `if`) or when

-   `a` is `true` and `b` is `false` (thus attaching the `else` to the
    second `if`).

In other words, one may see the previous statement as either of the
following expressions:

```pascal
    if a then (if b then s) else s2
```

or

```pascal
    if a then (if b then s else s2)
```

This is a problem that often comes up in compiler construction,
especially scannerless parsing.

The convention when dealing with the dangling `else` is to attach the
`else` to the nearby `if` statement.

Programming languages like Pascal and C follow this convention, so there
is no ambiguity in the semantics of the language, though the use of a
parser generator may lead to ambiguous grammars. In these cases , such
as `begin...end` in Pascal and `{...}` in C.

Here follows a solution in PEG.js:

```js
$ cat danglingelse.pegjs 
/*
S ← 'if' C 'then' S 'else' S / 'if' C 'then' S
*/

S =   if C:C then S1:S else S2:S { return [ 'ifthenelse', C, S1, S2 ]; }
    / if C:C then S:S            { return [ 'ifthen', C, S]; }
    / O                          { return 'O'; }
_ = ' '*
C = _'c'_                        { return 'c'; }
O = _'o'_                        { return 'o'; }
else = _'else'_                 
if = _'if'_
then = _'then'_
```

```
$ cat use_danglingelse.js
```
```js
var PEG = require("./danglingelse.js");
var r = PEG.parse("if c then if c then o else o");
console.log(r);

$ ../bin/pegjs danglingelse.pegjs 
$ node use_danglingelse.js 
[ 'ifthen', 'c', [ 'ifthenelse', 'c', 'O', 'O' ] ]
```

Si invertimos el orden de las alternativas:

    [~/srcPLgrado/pegjs/examples(master)]$ cat danglingelse2.pegjs 
    /*
    S ← 'if' C 'then' S 'else' S / 'if' C 'then' S
    */

    S =   if C:C then S:S            { return [ 'ifthen', C, S]; }
        / if C:C then S1:S else S2:S { return [ 'ifthenelse', C, S1, S2 ]; }
        / O                          { return 'O'; }
    _ = ' '*
    C = _'c'_                        { return 'c'; }
    O = _'o'_                        { return 'o'; }
    else = _'else'_                 
    if = _'if'_
    then = _'then'_

el lenguaje reconocido cambia (vease el ejemplo en la sección
[subsection:pegvsgrammars]):

```
[~/srcPLgrado/pegjs/examples(master)]$ pegjs danglingelse2.pegjs 
[~/srcPLgrado/pegjs/examples(master)]$ cat use_danglingelse2.js 
```
```js
var PEG = require("./danglingelse2.js");
var r = PEG.parse("if c then if c then o else o");
console.log(JSON.stringify(r));
```

    [~/srcPLgrado/pegjs/examples(master)]$ node use_danglingelse2.js 

    /Users/casiano/local/src/javascript/PLgrado/pegjs/examples/danglingelse2.js:513
    SyntaxError: Expected " " or end of input but "e" found.

## PegJS en los Browser

Vease [PegJS en los Browser](peg-browser)

## Lab: Ambiguity in C++

This lab illustrates a problem that arises in C++. The C++ syntax does
not disambiguate between expression statements (`stmt`) and declaration
statements (`decl`). The ambiguity arises when an expression statement
has a function-style cast as its left-most subexpression. Since C does
not support function-style casts, this ambiguity does not occur in C
programs. For example, the phrase

`int (x) = y+z;`

parses as either a `decl` or a `stmt`.

The disambiguation rule used in C++ is that <span>*if the statement can
be interpreted both as a declaration and as an expression, the statement
is interpreted as a declaration statement.* </span>

The following examples disambiguate into <span>*expression*</span>
statements when the potential <span>*declarator*</span> is followed by
an operator different from equal or semicolon (`type_spec` stands for a
type specifier):

```
    type_spec(i)++;      
    type_spec(i,3)<<d;  
    type_spec(i)->l=24;
```

```
    type_spec(*i)(int); 
    type_spec(j)[5];   
    type_spec(m) = { 1, 2 }; 
    type_spec(a);              
    type_spec(*b)();          
    type_spec(c)=23;         
    type_spec(d),e,f,g=0;   
    type_spec(h)(e,3);     
```

Regarding to this problem, Bjarne Stroustrup remarks:

> Consider analyzing a statement consisting of a sequence of tokens as
> follows:
>
>                   type_spec (dec_or_exp) tail
>
> Here `dec_or_exp` must be a declarator, an expression, or both for the
> statement to be legal. This implies that `tail` must be a semicolon,
> something that can follow a parenthesized declarator or something that
> can follow a parenthesized expression, that is, an initializer,
> `const`, `volatile`, `(`, `[`, or a postfix or infix operator. The
> general cases cannot be resolved without backtracking, nested grammars
> or similar advanced parsing strategies. In particular, the lookahead
> needed to disambiguate this case is not limited.

The following grammar depicts an oversimplified version of the C++
ambiguity:

    $ cat CplusplusNested.y 
    %token ID INT NUM

    %right '='
    %left '+'

    %%
    prog:
        /* empty */
      | prog stmt
    ;

    stmt: 
        expr ';' 
      | decl    
    ;

    expr:
        ID 
      | NUM
      | INT '(' expr ')' /* typecast */ 
      | expr '+' expr
      | expr '=' expr
    ;

    decl:
        INT declarator ';'
      | INT declarator '=' expr ';'
    ;

    declarator:
        ID 
      | '(' declarator ')'
    ;

    %%

Escriba un programa PegJS  que distinga correctamente
entre declaraciones y sentencias. Este es un ejemplo de un programa que
usa una solución al problema:

    [~/Dropbox/src/javascript/PLgrado/pegjs-coffee-plugin/examples(master)]$ cat use_cplusplus.coffee 
    PEG = require("./cplusplus.js")
    input = "int (a); int c = int (b);"

    r = PEG.parse(input)
    console.log("input = '#{input}'\noutput="+JSON.stringify r)

    input = "int b = 4+2  ;  "
    r = PEG.parse(input)
    console.log("input = '#{input}'\noutput="+JSON.stringify r)

    input = "bum = caf = 4-1;\n"
    r = PEG.parse(input)
    console.log("input = '#{input}'\noutput="+JSON.stringify r)

    input = "b2 = int(4);"
    r = PEG.parse(input)
    console.log("input = '#{input}'\noutput="+JSON.stringify r)

    input = "int(4);"
    r = PEG.parse(input)
    console.log("input = '#{input}'\noutput="+JSON.stringify r)

Y este un ejemplo de salida:

    $ pegcoffee cplusplus.pegjs 
    $ coffee use_cplusplus.coffee 
    input = 'int (a); int c = int (b);'
    output=["decl","decl"]
    input = 'int b = 4+2  ;  '
    output=["decl"]
    input = 'bum = caf = 4-1;
    '
    output=["stmt"]
    input = 'b2 = int(4);'
    output=["stmt"]
    input = 'int(4);'
    output=["stmt"]

## La Gramática de PEG.js

See the [PEG.js](pegjs-grammar) grammar here

## Referencias 

* [Apuntes 2015/2016](http://crguezl.github.io/pl-html/node30.html)
* [Repo crguezl/pegjs](https://github.com/crguezl/pegjs)
* [The examples in this section](https://github.com/crguezl/pegjs/tree/master/examples)
* [PEG.js Documentation](https://pegjs.org/documentation)
* [PEG.js online Editor](https://pegjs.org/online)

### Para Localizar Ficheros (Profesor)

```
    [~/srcPLgrado/pegjs/examples(master)]$ pwd -P
    /Users/casiano/local/src/javascript/PLgrado/pegjs/examples
```
```
[~/.../pegjs/examples(master)]$ git remote -v
2020	https://github.com/pegjs/pegjs (fetch)
2020	https://github.com/pegjs/pegjs (push)
dmajda	https://github.com/dmajda/pegjs.git (fetch)
dmajda	https://github.com/dmajda/pegjs.git (push)
origin	git@github.com:crguezl/pegjs.git (fetch)
origin	git@github.com:crguezl/pegjs.git (push)
```