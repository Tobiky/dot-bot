# .BOT Specification

## Core Ideas

### Node Segregation

The process of .BOT is divided into nodes, of which there are two types. There
is the "Main" node which is the dispatcher and the "Execution" nodes which are
the handlers. Each execution node is marked with an identifier, an integer or a
string, that is either custom provided by the user or a random default one.

Additionally, each execution node has an identifier. As this is also a node
identifier, it too can be custom defined. All nodes with the same prefix belong
to the same "Cluster". A cluster is garantueed a thread (but not garantueed to
be a kernel thread). To each cluster, a dispatch node is also created as the
head of the cluster. This node is viewed as an execution node to dispatcher
nodes and as execution node to dispatcher nodes. If the prefix is empty or none,
which defaults to empty, the node belongs under the main node.

### Domain Language

When making a bot for Slack, Discord, or other it often involves defining the
structures and converstions to and from the built in structures before any
actual functionality can be added. Thus one of the most important features of
.BOT is to decrease the time spent doing this as much as possible.

Additionally, the usage of the domain language should also be embedded into the
.BOT script when used. If you constantly have to apply converstion from some
`String` and `Int` to your DSL `ID` and back with casting or, god forbid,
seperate functions, there wouldn't be much reason to create a DSL for bots.

## Grammar

### Keywords

- context
- fnc
- cmd
- string
- character
- integer
- float

### Examples

Examples to showcase how the language should aim to look like and how it should
be used.

#### Simple add function

##### Inline

```none
fnc add a b is a + b.
```

#### Explicit

```none
fnc add a b
    a + b.
```

or

```none
fnc add a b
    a + b
.
```

#### Simple command for replying to user

```none
cmd hello context.reply "Hello, $(context.author.name)".
```

#### Simple call to DSL integration

```none
users = $sql(SELECT * FROM users)
```

#### DSL integration call with options

```none
user = $sql(SELECT * FROM users WHERE users.id = 0)
value = $mongosh{
    host="localhost:8080"
}(
    use valueDb;
    db.valueCollection.find({
        userId: $(user.id)
    });
)
```

#### Meta configuration

Prefix for commands (when called by user on platform) using explicit and inline format.

```none
set {
    prefix = "$".
}

set prefix = "/"
```

#### Handling internal and external naming collision

```none
use util.{add, div}.

fnc add a b is a + b.
fnc mul a b is a * b.
fnc h_sub a b is a - b.

cmd add a b
    context.reply $(use local fnc add a b).
cmd div a b
    context.reply $(use foreign fnc mul a b).
cmd mul a b
    context.reply $(use fnc mul a b).
cmd sub a b
    context.reply $(h_sub a b).
```

#### Function with type annotation

```none
fnc add <a: integer> <b: integer> is a + b.
fnc sub <a: integer> <b: integer>
    a - b.
```

### Productions

**executionType**:\
| "cmd"
| "fnc"
| "<" executionType ":" type ">"

**primitiveType**:\
| "string"\
| "character"\
| "integer"\
| "float"

**dataType**:\
| primitiveType\
| identity

**type**:\
| executionType\
| dataType

**booleanBinaryOperator**:\
| "=="
| "&&"
| "||"
| "^^"

**binaryOperator**:\
| "+"
| "-"
| "*"
| "/"
| "%"
| "&"
| "|"
| "^"
| "<<"
| ">>"
| "!" booleanBinaryOperator

**booleanUnaryOperator**:\
| "!"

**leftUnaryOperator**:\
| "~"
| "!" booleanUnaryOperator

**expression**:\
| expression binaryOperator expression
| leftUnaryOperator expression

**statement**:\
| expression
| statement ";"

**argument**:\
| "<" identity ":" type ">"
| identity

**function**:\
| executionType identity {argument} "is" statement "."
| executionType identity {argument} "\n" statement {statement} "."

### Informal Semantics

- Main node is responsible for resolving function and commands. Each node and
file are given exploration value, publically visisble as an incrementing number.

- All functions marked as command are exported from the file, visible to all
nodes. When a function is executed and its identifier cannot be found in the
current context, it is looked for globally by asking the main node for it. The
order preference is: shortest graph distance to requesting node, shortest graph
distance to main node, lowest exploration value.

- Commands and functions can have the same name but in that case will require
additional identification in such cases.

- `$OPT_IDENTIFIER{opts}(expr)` prioritizes evaluating the expression. This can
be used for both the typical usage of parenthesis for prioritizing expressions
as well as for string interpolation in bash. Optional identifiers and further
options can be used in some cases. For example when including an SQL-language
or MongoDB Shell integration where a specific host or other configuration might
necessary to run commands.

- `type(literal)` evaluates or converts the literal into the type. While this
should work for datatypes its primarly part of .BOT to allow for natural
integration of things like SQL into the script as well as syntax highlighting
for those situations. The script engine treat the literal mostly as a regular
`String`.

- Statically typed but support for integrating converstion naturally. Also
include natural dynamic type for cases when JSON-like values are returned
or used (such as MongSh).

- Evaluation of types is lazy (extrapolated) except when annotated. The
execution engine will attempt to solve (and remember) the type when needed.
