# Cylon Yolol AST spec

> Written by Oscar (Matrixmage).
>
> This spec was primarily designed by the following Cylon users (in alphabetical order):  
> `Chthonium#3988`  
> `Martin#2468`  
> `Matrixmage#4830`  
> `Pyry#6210`  
> `rad dude broham#2970`  

**Version 0.1.0**

To allow for interoperability between community yolol tools, we've designed the Cylon Yolol AST spec to provide a specification for what an abstract syntax tree (AST) of yolol should look like. This specification is something we hope all community yolol tools will follow, so that we can have all of our tools work together and make more awesome things than we could make alone.

# Design goals

The Cylon Yolol AST ("Cylon AST" from now on) specification is designed with the following design goals in mind:

* Being useful in powering syntax highlighting
* Allow practical reconstruction of the original source
* Permit optional information to be included

# Specification

A compliant Cylon AST is a UTF-8 encoded text file formatted in JSON. A single Cylon AST JSON file is designed to represent a single program, which has some number of lines within it.

**Every** key in a compliant Cylon AST is in lowercase, with snake_case used to separate words. This is to make things as simple and unambiguous as possible.

## Nodes

A Cylon AST is made up of many nodes, each representing an aspect of the program. All nodes have certain required keys, which must be included for maintaining compliance. Nodes may not contain any keys which aren't required or optional; containing arbitrary keys is non-compliant.

**Most** nodes contain an explicitly required `type` key. This type is the same as the name of the node in the node list below. Some nodes have sub-types, for example `goto` is a sub-type of `statement`. In this case, the `type` key would have the value `statement::goto`, which is reflected in the node list below.


### Metadata
All nodes may optionally contain a `metadata` key, allowed to contain arbitrary keys and values for consumption by other tools.

### Node types

Below is an exhaustive list of all the node types that can exist in a compliant Cylon AST. The required keys list for each node has the keys written in the form `<name>: <type>`, to make things more explicit.

---

### `root`
**Required keys:** `version: String`, `program: Node<program>`

This is the outermost enclosing node in a Cylon AST. Everything is within the root node. Note that the `root` node is **unnamed** by virtue of being the enclosing JSON object itself. It is also one of the only nodes without a `type` key.

The `version` key is a [SemVer](https://semver.org/) encoded string representing what version of the Cylon AST spec the output was designed for.

---

### `program`
**Required keys:** `type: String`, `lines: Array<Node<line>>`

A program in yolol. The `lines` key is an array of `line` nodes. Although yolol programs may never exceed 20 lines, there is no limit on the size of the `lines` key.

---

### `line`
**Required keys:** `type: String`, `code: Array<Node<statement>>`  
**Optional keys:** `comment: String`

A singular line of yolol code. The `code` key is an array of `statement` nodes. Although real lines in yolol may not exceed 70 characters, there is no restriction on the size of the `code` key or nodes within it.

The optional `comment` key is a string which contains the exact text of a comment, which is everything after the leading `//`. This comment is by definition assumed to exist at the end of the line, since comments may never exist in the middle of a valid yolol line. It is compliant to have an empty `code` array but still have a `comment` string.

---

### `statement`
**Required keys:** `type: String`, others as dictated by the chosen sub-type

A singular statement in yolol. The `type` key dictates which kind of statement the node represents, and may contain one of the below sub-type names. The statement node will have other required keys as dictated by which sub-type the node is.

### `statement::goto`
**Required keys:** `expression: Node<expression>`

A goto statement. Contains the expression which evaluates to the destination line.

### `statement::if`
**Required keys:** `condition: Node<expression>`, `body: Array<Node<statement>>`, `else_body: Array<Node<statement>>`

An if statement. The `condition` key is the expression which evaluates to the control condition. `body` and `else_body` contain arrays of the statements to execute based on the `condition` value.

### `statement::assignment`
**Required keys:** `identifier: String`, `operator: String`, `value: Expression`

An assignment operation. `identifier` contains the name of the identifier to modify, prefixed by a colon if it's a data field. `operator` is a string which contains the textual representation (`+=` for example) of the actual operator being applied. `value` is an expression which evaluates to the value to be assigned.

### `statement::expression`
**Required keys:** `expression: Node<expression>`

A wrapper around an expression type node. `expression` is the expression being wrapped.

---

### `expression`
**Required keys:** `type: String`, others as dictated by the chosen sub-type

An expression in yolol. The `type` key dictates which kind of expression the node represents, and may contain one of the below sub-type names. The expression node will have other required keys as dictated by which sub-type the node is.

### `expression::group`
**Required keys:** `group: Node<expression>`

A wrapper for an expression wrapped in parenthesis. The `group` key contains said wrapped expression.

### `expression::binary_op`
**Required keys:** `operator: string`, `left: Node<expression>`, `right: Node<expression>`

A binary operation in yolol. The `operator` key is a string which contains the textual representation (`+` for example) of the actual operator being applied. `left` and `right` contain the expressions that evaluate to the left and right hand sides, respectively, of the operator.

### `expression::unary_op`
**Required keys:** `operator: String`, `operand: Expression`

A unary operation in yolol. The `operator` key is a string which contains the textual representation (`-` for example) of the operator being applied. Due to the prefix/postfix operators being the same textually, they have the special representation of `++a` and `a++`, to use prefix/postfix increment as an example. `operand` contains the expression that evaluates to the value that the operator is applied to.

### `expression::value`
**Required keys:** `value: Node<value>`

A wrapper around a value type node. `value` is the value being wrapped.

---

### `value`
**Required keys:** `type: String`, other keys as dictated by the chosen sub-type

A value in yolol. The `type` key dictates which kind of value the node represents, and may contain one of the below sub-type names. The value node will have other required keys as dictated by which sub-type the node is.

### `value::identifier`
**Required keys:** `name: String`

Either a local variable or a data field in yolol. Data field identifiers are prefixed by a colon in their string value.

### `value::number`
**Required keys:** `num: String`

A number in yolol. Is encoded as a string due to the hyper-specific decimal rules binding compliant yolol numbers.
