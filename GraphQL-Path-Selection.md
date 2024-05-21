## RFC: GraphQL Path Selection

**Proposed by:** [Yaron Karni](mailto://yaron_kerni@intuit.com) - Intuit

### Motivation and Use Cases

GraphQL documents often need to be generated automatically and there are many cases where automated mutation is useful. For example, to develop a security testing tool that accepts GraphQL documents, modifies them and applies the resulting documents, two major components are needed:
* A powerful syntax to identify and select components of a GraphQL document.
* An API that traveses documents using these _path selection_ expressions and can flexibly make changes to the document.

This RFC addresses the first component by defining a path selection syntax, comparable to [XPath](https://www.w3.org/TR/xpath-30/) and [JSONPath](https://www.rfc-editor.org/rfc/rfc9535.txt). We note that while XML and JSON documents often represent data directly, GraphQL documents typically do not do that. Yet, automated generation and modification of GraphQL documents justifies a similar approach to addressing components of these documents.

### Terminology

* This document assumes that the GraphQL document has been parsed into an Abstract Syntax Tree (AST), and we use the term **Node** to refer to a node of that tree.
* The AST is ordered similarly to the original document. So when we refer to the **first node** that meets some condition, this assumes depth-first search of the tree, where a node always precedes all its children. Similarly when counting nodes that meet a condition, 
* Path selection expressions consist of a sequence of **path elements** separated by the slash (`/`) character.

### Examples

#### Simple Selection

In the following document, we have two nodes named 'name', but of different types: argument and field.

```graphql
query {
    Instrument(name: "1234") {
        Reference {
            name
            title
        }
    }
}
```

- Select all nodes *name* where *name* is an argument: `//query/Instrument/name[type=arg]`

- Select the first node *name* where *name* is an argument: `/query/Instrument/name[type=arg]`

- An ellipsis `...` matches any sequence of path elements (including the ==empty sequence==), so `//query/.../name[type=arg]` here is identical to the query `//query/Instrument/name[type=arg]`
  
- To select the *name* node which is a field under *Reference*, residing under *Instrumen*t, which is in turn under *query* `//query/Instrument/Reference/name`

- `//query/Instrument/.../name` is the same as `//query/Instrument/Reference/name`  
  
- `//.../name` returns the same node as `//query/Instrument/Reference/name`

#### More Complex Selection
```graphql
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
    friends @include(if: $withFriends) {
      name
    }
    friends @include(if: $withFriends) {
      name
    }
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

- Select all query nodes named *hero*: `//query[name=hero]` 

- Select the first query named *hero*: `/query[name=hero]`     

- Select all nodes named _name_ located under the _friends_ node `//query[name=hero]/hero/friends/name` 

- Select all nodes named _name_ within the range between index 0 and index 2 (inclusive), located under friends node `{0:2}//query[name=hero]/hero/friends/name`   

- Select the first node named _name_, located under any node which itself is located under the _hero_ node: `/query[name=hero]/hero/.../name `

- Select the _$withFriends_ variable located directly under the root node named _hero_: `//query[name=hero]/withFriends[type=var]`

- Select the first _include_ directive located under *friends*, which itself is located under the root query node named _hero_ `/query[name=hero]/hero/friends/include[type=direc] `

- Select the _if_ argument node which is located under the _@include_ directive `//.../include[type=direc]/if[type=arg]`  

- Select the *episode* variable `//.../episode[type=var]`


### High-Level Syntax Definition
The most useful path expressions are listed below:

| Expression | Description   |
|---------|---------------------------------------|
| `//`    | Path prefix: select all nodes from the root node, and use a slash as a separator between path elements.                            |
| `/`     | Path prefix: select the first node from the root node and use a slash as a separator between path elements.  A range is not supported when the first node is selected. |
| `{x:y}//` | Path prefix, select path node(s) in the range between *x* path node and *y* path node (inclusive). *x* and *y* are non-negative integers. The first node to meet the condition is numbered 0, so {0:2} refers to the first three nodes. |
| `{:y}//` | Path prefix, select path node(s) between the first path node to *y*. |
| `{x:}//` | Path prefix, select path node(s), between the *x* path node result to the last path node result. |
| `{:}//` | Path prefix, select all node(s) from the root node. |
| `...` | Support relative path "any" selection e.g. `{x:y}//a/b/.../f`.  Ellipsis can be used anywhere in the selection expression, except at the end of the path. Ellipsis can be used multiple times within the expression, this helps to avoid listing the entire node structure when dealing with deep GraphQL structures. |

### Formal Syntax
