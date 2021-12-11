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
- function
- command
- string
- character
- integer
- float

### Productions

**executionType**:\
| "command"
| "function"
| identity

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

**typeAnnotation**:\
| "<" dataType ":" identity ">"

**expression**:\
|

**statement**:\
| expression

**argument**:\
| typeAnnotation
| identity

**function**:\
| executionType identity {argument} ["is"] {statement} "."

### Examples

Because I'm only a robot, here are a few examples of what the grammer should parse.

```
function add a b is a + b.
```
or
```
function add a b
    a + b.
```
or
```
function add a b
    a + b
.
```

```
command hello context.reply "Hello, $(context.author.name)".
```

```
user = sql(SELECT * FROM users)
value = mongosh(mongodb://localhost:28015, 
    use valueDb
    db.valueCollection.find({
        userId: $(user.id)
    })
)
```

```
set {
    prefix = "$".
    identifier = "numbers".
    Number as Int as String.
}

function add a b is a + b.
function mul a b is a * b.
function h_sub a b is a - b.

command add a b
    context.reply $(use function add a b).
command mul a b
    context.reply $(use function mul a b).
command sun a b
    context.reply $(h_sub a b).
```

## Semantics

- All functions marked as command are exported from the file, visible to all nodes. When a function is executed and its identifier cannot be found in the current context, it is looked for globally by asking the main node for it.

- `$(expr)` prioritizes evaluating the expression. This can be used for both the typical usage of parenthesis for prioritizing expressions as well as for string interpolation in bash.

- `type(literal)` evaluates or converts the literal into the type. While this should work for datatypes its primarly part of .BOT to allow for natural integration of things like SQL into the script as well as syntax highlighting for those situations. The script engine treat the literal mostly as a regular `String`.

- Statically typed but support for integrating converstion naturally.

- Evaluation of types are lazy except when annotated.