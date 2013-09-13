![Miniup logo](res/miniup.png)
# Miniup - the pragmatic parsing library
[Try now!](TODO link to test page)
## A short introduction to miniup
Welcome to Miniup, the pragmatic parsing library. Miniup is a pragmatic parsing library that is strongly inspired by context-free grammar parsing libraries such as Stratego/XT. Miniup offers grammars in a pragmatic and therefor easy-to-use way.

### The pragmatism of Miniup:

* _Easy-to-write grammars._ Miniup offers built-in support for common concepts such as lists, operators and choices. These concepts avoids the need of the grammar writer to be aware of the underlying parsing algorithm and its typical drawbacks.
* _Context-free parsing algorithm._ Miniup applies no lexical analysis before parsing and is therefor a context-free parser. This means the parser can switch between different languages during the parse process
* _Friendly AST._ The AST generated by the Miniup parser is very intuitive.
* _No intermediate code._ Miniup does not generate a parser, it is a parser. You can hook up miniup directly in your java / javascript / typescript environment without needing intermediate compilation steps.

* _ **No** support for semantics _ Many parser generators allow semantic actions do be defined inside a grammar. However, this kills the readability of a grammar, makes interoperability with other environments very hard and prevent parser to do some automatic refactorings on the grammar. In fact, the allowance for semantic actions mixes two concerns which should be separated in my humble opininion: Parsing input to an AST, and interpreting an AST to do something sensible with it. An other import feature of semantic actions is that it allows to build 'nice' AST's. However, in miniup this is done automatically thanks to the common concepts.

automatic whitespace and built in token definitions

### How does miniup do that?

In contrast to LL or LR based parsing algorithms, rules in Miniup are not defined by using BNF. In miniup, rules are defined by referring to common concepts typically encountered in most programming languages. Concepts such as 'infix operator' or 'list'. This gives Miniup an edge over parsers that do not support this concept. First of all, writing grammars is easier, the author is not required to know which kind of parsing algoritm is applied to be able to write a working grammar. Second, the AST produced by miniup is built up in such a way that it reflects these concepts.

#Mention somewhere:
Left recursion
1 Kloc
Test page
Built in tokens + conversion functions
Build instructions
PEGJS compatibility.

Extended and meta AST information
TODO: convert every example in a unit test.

# PEG grammar constructions

Construction a grammar file:

## /\* comment \*/
Java style multi-line comment

## // comment <sub>END OF LINE</sub>
Java style single line comment

## rulename "displayname" = expr
Defines a rule with `rulename`. A rule with a certain name can be defined only once. `expr` refers to any of the constructions defined below. The `"displayname" string is a human-friendly name used in error reporting and is optional.

## rule
Searches for a rule named `rulename` and tries to match it. Rules can be used before they are defined (syntactically speaking).

Example:

All examples in this section are in the format `grammar` x `input` &raquo; `output`. The examples can be verified by running the command `miniup -c -g "grammar" "input"` which should produce the mentioned output.

`phone = number; number = [0-9]+;` x `45` &raquo; `"45"`

## 'literal'
Tries to match `literal` literally in the input. Both single- and double quotes are allowed to define the literal. Normal java(script) escaping is allowed within the literal (e.g. `'quote\' and newline\n'`). Unicode, hexadecimal and octal escape sequences are allowed as well. The `i` flag can be added after the closing quote to perform the match case-insensitive.

Example:`foo = "baR"i` x `BAr` &raquo; `"BAr"`

## [characterset]
Tries to match exactly one character from the given characterset. Ranges and negations can be used, similar to regular expressions. For example `[^0-9 ]` matches everything but a digit or a space. The `i` flag can be added after the closing bracket to perform the match case-insensitive. Within the characterset, the slash (`\`) can be used as escape character.

Example: `foo = [^bar]` x `R` &raquo; `"R"`

## expr<sub>1</sub> expr<sub>2</sub> ... expr<sub>n</sub>
Tries to match all the expressions or fails. Returns an object containing all submatches. If the `extendedAST` option is enabled, each submatch is available under its (zero-based) index. Otherwise only labeled items are available (see `label`)

Example: `foo = 'bar' name:'baz'` x `barbaz` &raquo; `{ name: "baz" }`

## expr<sub>1</sub> / expr<sub>2</sub> ... / expr<sub>n</sub>
Tries to match either `expr1`, `expr2` or `expr-n`. The choice rule returns the first successful matches and does not attempt to match any subsequent choices. This is unlike the behavior of the `|` operator in context free grammars.

Example: `foo = 'a' / 'b' / 'b' / 'c'` x `'b'` &raquo; `"b"`

## (expr<sub>1</sub> expr<sub>2</sub> ... expr<sub>n</sub>)
Groups the list of expressions. Behavior is identical to not using parentheses. But parentheses are very useful to make quantifiers or predicates match multiple expressions at the same time.

Example:

    decl = @whitespace-on modifiers:(pub:'public'? stat:'static'?) name: IDENTIFIER '(' ')'

x `static foo()` &raquo; `{ modifiers : { pub : null, stat: 'static' }, name: "foo" }`

## label: expr
Matches `expr` and stores it under `label` in the resulting AST.

Items will not be available in the result AST unless they are either labeled or matched by a Regex or Characterclass (todo: verify:) or Literal matcher. If the `extendedAST` parse option (`--extended`) is used, all items are available in the resulting AST.

Example: `abc = a:'a' 'b' c:'c'` x `abc` &raquo; `{ a: "a", c: "c"}`

Example (with extended AST enabled): `abc = a:'a' 'b' c:'c'` x `abc` &raquo; `{ a: "a", c: "c", 0: "a", 1: "b", 2: "c", length: 3 }`

## expr?
Optionally matches `expr`. If `expr` is not found, the parse is still considered to be successful.

Example: `foo = bar:'bar'? baz:baz` x `baz` &raquo; `{ bar: null, baz: 'baz'}`

## expr\*
Matches `expr` as many times as possible, but no matches are fine as well. The match will always be performed greedy. The matches will be returned as array.

Example: `foo = 'a'*` x `aaaa` &raquo; `['a', 'a', 'a', 'a']`

## expr\+
Matches `expr` as many times as possible, but at least once. The match will always be performed greedy. The matches will be returned as array.

Example: `foo = 'a'+` x `aaaa` &raquo; `['a', 'a', 'a', 'a']`

## &expr
Positive predicate. Tries to match `expr`. If `expr` is found, the match is considered successful, but the rule does not match anything. Can be used as 'lookahead'.

Example: `foo = &'0' [0-9]+` x `017` &raquo; `"017"`. But, this rule will fail on the input `117`.

## \!expr
Negative predicate. Tries to match `expr`. If `expr` is *not* found, the match is considered successful and parsing will continue. But fails if `expr` is found.

Example: `foo = 'idontlike ' !'coffee' what:[a-z]*` x `idontlike tea` &raquo; `{ what: tea }`. But, this rule will fail on `idontlike coffee`. And rightfully so.

# Miniup PEG extensions

## (expr<sub>1</sub> ... expr<sub>n</sub> separator)\*?
Matches all items between the parentheses zero or more times. However, the last item of the sequence is used as separator to initiate the repitition.

Example: `expr = 'dummy'; args = args:(expr ',')*?` x `dummy,dummy,dummy` &raquo; `{ args: ["dummy", "dummy", "dummy"] }`

## (expr<sub>1</sub> ... expr<sub>n</sub> separator)\*?
Behavios the same as the `*?` operator, but requires at least one match.

Example: `expr = 'dummy'; args = args:(expr ',')*?` x `dummy` &raquo; `{ args: ["dummy"] }`

(stuf)\# match a set

@import module.name

@whitespace-on itemx itemy itemz
@whitespace-off itemx itemy itemz

@left-associative operand operator operand

@right-associative operand operator operand



# built-in tokens

# left recursion

# adept grammar on the fly

### Show me a grammar!

Some grammar

--

Run from command line
--

Some AST

### Use from javascript

## Miniup reference

### Language options

### Miniup grammar concepts

#### List

#### Choice

#### Token

#### Import

#### Operator

# Run Miniup from commandline

Miniup can read input from files, standard input stream or arguments. To parse input from the input stream use the following command:

`echo "+31 2344567" | miniup -rc -g "phone = '+' countrycode:[0-9]+ ' ' phonenumber:[0-9]+"`

Which (TODO) outputs:


	{
		country : "31",
		phonenumber: "2344567"
	}

Or, if you provide invalid input:

`echo "+31 23oops7" | miniup -rc -g "phone = '+' countrycode:[0-9]+ ' ' phonenumber:[0-9]+"`

TODO: error output

All available command line options can be listed by using `miniup --help`

# Run Miniup from javascript

Build a grammar object from a grammar string:

	var grammar = miniup.Grammar.load("phone = '+' countrycode:[0-9]+ ' ' phonenumber:[0-9]+");`

To parse input, use `grammar.parse(input /*string*/ [, options /*options object*/])`

	var ast = grammar.parse("+31 2344567");
	console.log("Country: " + ast.country + " Phone: " + ast.phone)


The following options can be provided to the parse function:


	{
		debug: true, /* default: false, prints the complete parse strategy*/
		cleanAST: true, /* default: false, does not include additional match information such as $text, $rule and $pos*/
		extendedAST: true, /* default: false, if true, unlabeled items are added to the resulting AST as well */
		startSymbol: 'rulename', /* use alternative start symbol. Default: first rule that is defined in the grammar */
		inputName: 'string' /* default: 'input', name of the input to use in error reporting, such as a filename */
	}


