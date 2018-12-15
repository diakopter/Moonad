```
TODO:
no reduce function on formality!
instead, reduce = from_net(type, reduce_net(to_net(x)))
readback of pattern-matching is detected when a solid type is applied to something
```

## Moon: a Peer-to-Peer Operating System

**Abstract.** A purely peer-to-peer implementation of an operating system should allow online applications to operate perpetually with no downtime. Ethereum partly solved that problem in 2014, but its focus on smart-contracts posed challenges for everyday adoption. Building upon its ideas, we present the design of a complete decentralized operating system. Our low-level machine language, Nasic, is inherently parallel and adequate for reducing high-order programs efficiently, in contrast to register and stack machines. Our high-level language, Formality, is capable of exploiting Curry-Howard's isomorphism to verify security claims offline, allowing users to trust downloaded code without trusting its developers. An inductive datatype, DappSpec, is used to specify front-end user interfaces. A proof-of-work, tokenless blockchain, Bitlog, is used to aggregate and order global transactions, allowing applications to have a common back-end state. Our design is complete and final, making this a self-contained, timeless specification of the operating system. To facilitate independent implementations, each module is accompanied by commented Python code.

## 1. Nasic: a parallel, low-level machine language

A very simple graph-rewrite system with only 3 symbols and 6 rules has been shown to be a universal model of computation [1]. This system is particularly interesting for its perfect computational properties. Like a Turing machine, it can be evaluated as a series of atomic, local operations with a clear cost model. Like the λ-calculus, it is inherently high-order and has a nice logical interpretation. While, from the viewpoint of *computability*, those systems are equivalent (Turing complete), from the viewpoint of *computation*, interaction combinators is arguably superior, for being confluent and inherently parallel. In other words, while the Turing machine and other classical models can always be simulated by interaction combinators with no loss of efficiency, the converse isn't true. For those reasons, Moon uses an adaptation of it, Nasic (standing for n-ary symmetric interaction combinators), as its low-level machine language. This allows its applications to run in a model that, despite being so simple, is infinitely optimizable and extremelly powerful, thus, remarkably future-proof.

### Specification

Differently from conventional machine languages, which are usually described as bytecodes, Nasic programs are described as graphs in a specific format, which includes a single type of node with 3 outgoing edges and a symbolic label. That node is depicted below in a visual notation:

(image)

Any connected arrangement of those nodes forms a valid Nasic program. For example, those are valid Nasic programs:

(image)

Notice that the position of edges is important. For example, those graphs, while isomorphic, denote two different programs:

(image)

For that reason, the ports from which edges come are named. The port at the top of the triangle is the `main` port. The one nearest to it in counter-clockwise direction is the `aux1` port. The other one is the `aux2` port.

(image)

Moreover, there are 2 computation rules:

(image)

Those rewrite rules dictate that, whenever a sub-graph matches the left side of a rule, it must be replaced by its right side. For example, the graph below:

(image)

Is rewritten as:

(image)

Because (...). Rewritteable nodes are called redexes, or active pairs. 

This completes Nasic's specification. At this point, you might be questioning if such a simple graphical system can really replace Assembly and describe arbitrary programs that range from simple calculators to entire online games and social networks. While this may not be obvious or intuitive at first, rest assured the answer is yes, as will become clear through the paper. For now, we'll shift our focus to just implementing such system, as described. 

### Implementation


In our implementation, we will represent a Net as a table, where each row represents a Node, with its label, and 3 ports, each one including a Pointer to another port. As an example, consider the following net:

(image)

This could be represented as the following table:

addr | label | port 0 | port 1 | port 2
--- | --- | --- | --- | ---
0 | 0 | addr X, port X | addr X, port X | addr X, port X
1 | 0 | addr X, port X | addr X, port X | addr X, port X
2 | 1 | addr X, port X | addr X, port X | addr X, port X
3 | 0 | addr X, port X | addr X, port X | addr X, port X

For that, we use 3 classes: `Pointer`, holding an address and a port, `Node`, holding a node's label and ports, and `Net`, holding the table above, plus an array of active pairs, and reclaimable memory addresses. On the `Net` class, we implement 2 methods for space management: `alloc_node` and `free_node`, 3 methods for pointer wiring and navigation: `enter_port`, `link_ports` and `unlink_port`, and 2 methods for reductions: `rewrite` and `reduce`. Together, they form the core of our algorithm, encapsulating the fundamental rules of computation, annihilation, which captures interaction, functions, datatypes and pattern matching, and commutation, which captures loops and recursion. Parallelism and garbage-collection are naturally captured by the reduction machinery. A fully commented, executable example is available at [`nasic.py`](nasic.py). By importing this file, we're able to load the net above and reduce it:

```
net = Net.from_table(...)
```

This outputs the following table:

addr | label | port 0 | port 1 | port 2
--- | --- | --- | --- | ---
0 | 0 | addr X, port X | addr X, port X | addr X, port X
1 | 0 | addr X, port X | addr X, port X | addr X, port X
2 | 1 | addr X, port X | addr X, port X | addr X, port X
3 | 0 | addr X, port X | addr X, port X | addr X, port X

Which represents the reduced form of our graph:

(image)

### Performance considerations

While we've presented a complete Python implementation, it is clearly not the fastest one. In other of our implementations, we've observed remarkable performance characteristics. In ideal cases, Nasic managed to beat mature functional compilers such as GHC by orders of magnitude, due to optimal reductions and runtime fusion, which are unique to it. Even in worst cases, it didn't stand far behind in raw pattern-matches per second. Since those were naive implementations, though, there is still a lot of room for improvements. Nasic's structure makes it adequate for massively parallel architectures such as the GPU, which we did not fully explore yet. With proper miniaturization such as FPGAs and ASICs, we could have native Nasic processors as alternative for classical computer architectures. We leave it for client developers to creatively optimize their Nasic implementations as much as possible. 

## 2. Formality: a high-level language for programs and proofs

types as a language of specifications

programs as mathematical proofs

### Description

### Implementation

## 3. DappSpec: a specification format for decentralized applications

```haskell
data Uint : Type
| O : (x : Uint) -> Uint
| I : (x : Uint) -> Uint
| Z : Uint

data List<A : Type> : Type
| cons : (x : A, xs : List) -> List
| nil  : List

data Element : Type
| circle : (x : Uint, y : Uint, r : Uint) -> Element
| square : (x : Uint, y : Uint, r : Uint) -> Element

let Document List<Element>

data LocalEvent : Type
| MouseClick : (x : Uint, y : Uint) -> LocalEvent
| Keypress   : (k : Uint)           -> LocalEvent

data App : Type
| new : 
  ( LocalState     : Type
  , local_inistate : LocalState
  , local_transact : (event : LocalEvent, state : LocalState) -> LocalState
  , render         : (state : LocalState) -> Document)
  -> App
```

## 4. Bitlog: a token-less blockchain for global transactions

## References

[1] https://pdfs.semanticscholar.org/6cfe/09aa6e5da6ce98077b7a048cb1badd78cc76.pdf
