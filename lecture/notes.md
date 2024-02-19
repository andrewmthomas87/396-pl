# Avoiding Catastrophic Interference: Protection, Abstraction, and Typing

## Papers

1. Protection in Programming Languages, Morris (1973)
2. Programming with Abstract Data Types, Liskov (1974)
3. Typeful Programming, Caredelli (1989)

## Protection in Programming Languages

- How can a growing set of modules coexist and cooperate?
- "Although the need for protection is usually inspired by situations involving hostility (e.g. competing government agencies sharing a computer system), experienced programmers will attest that hostility is not a necessary precondition for catastrophic interference between programs. An undebugged program, coexisting  other programs, might as well be regarded as having been written by a malicious enemy--even if all the programs have the same author!"
- "A programmer should be able to prove that his programs have various properties and do not malfunction, solely on the basis of what he can see from his private bailiwick. To make "his bailiwick" more precise we simply equate it with a single textual region of a larger program"
- "we are only interested in those properties of a subprogram which are invariant over all the possible program texts into which the subprogram might be inserted"

### Procedures as Objects

- "The procedure or subroutine has always been a convenient device for one programmer to make his work available to others. The essence of its power is that the user of a procedure need not be concerned with the details of its operation, only the effect."
- "in order to exploit this device to its fullest extent it is useful to make procedures full-fledged objects in the sense that they can be passed as parameters or values of other procedures and be assigned to variables"
- LISP, ISWIM, PAL, GEDANKEN
- "Our example programs will be written in GEDANKEN as its definition is probably the most readily accessible to the reader"
- Example 1

```
[Count is Ref(O);
  λ( ){Count := Count + 1;
    Val(Count)}]
```

### Local Objects

- "An object is local to a textual region R if the only expressions that will ever have it as a value are part of R"
- "The advantage of an object's being local is that one can enumerate all the situations in which it plays a role by looking at a single (hopefully small) region of the program"
- Strategy to determine: for each ref, the value can escape if:
  1. Passed as a parameter
  2. Returned
  3. Assigned to non-local reference
- Check the above to show it does escape

### Memory Protection

- Can provide restricted access to local references w/ procedures
- Fig. 1
- Call `Createtable`, returns `Item, Index, Save`
- References `Table, Count` & procedures `Locate, Move` local
- Add *s, make assertion `Table` remains in ascending order
- FORTAN's local vars & ALGOL's own vars are limited, can't have multiple instances

### Type Protection

- Representation of a type of object
- Fig. 2
- Example: interval: `Createint(x,y), Max, Min, Sum`
- It's just a pair, so the user can interact w/ it, notion of "type violations":
  1. Alteration
    - Manual changes w/o provided primitive functions
    - Violate -> primitive function might malfunction, "embarrass their programmer"
    - Argument for defensive programming (checking params), what if you have a sorted vector & a function that uses binary search, checking for sorted defeats the purpose
  2. Discovery
    - Look at the data w/o provided primitive functions
    - Violate -> no direct threat, more subtle considerations
  3. Impersonation
    - Call provided primitive function w/ a compatible object, not constructed properly
    - Violate -> same as alteration
    - Could be a mistake, inform ASAP, or unplanned, but useful

- Rest of paper: mechanisms to prevent the above
    - Generalization of type tags from implementation level of dynamic langauges, made available to user
    - New:
      1. cascadable: reflects our belief that representation can be a multilevel affair
      2. completely dynamic: a running program could generate an arbitrarily large number of different type tags

### Seals

- Universally available `Createseal` returns `Seal_i, Unseal_i`
- `i` is a serial number which changes every time `Createseal` is called
- `Seal_i(x)` yields a new object `x'` such that any operation applied to `x'` except `Unseal_i` results in an error
- `Unseal_i(x')` yields `x`, if `x'` was produced by `Seal_i`, and
results in error otherwise
- Fig. 3 alter Fig. 2
- The `Seal` mechanism provides 2 separable functions of protection
  1. Authentication: embodied in (1)
  2. Limited access: embodied in (2)

### Trademarks

- If we are not concerned with limiting access to a class of objects, we could make the `Unseal` operation publicly available. The same effect can bc achieved in a more convenient way by use of a new primitive function, `Createtrademark`.
- `Createtrademark` returns `Mark_i, i`
- `i` is an ever-changing serial number as before
- `Mark_i(x)` yields an object `x'` that behaves exactly like `x`
except that it bears trademark `i`
- `Trademarklist(x')` is a publicly available operation which yields a list of the trademarks borne by `x'`
- `Mark_i` can be used in place of `Seal_i`
- `if i \in Trademarklist(x) then x else go to Error` in place of `Unseal_i(x)`.
- Consider how trademarks and other authenticating marks are used in the commercial/bureaucratic sphere. A letter saying, "You have been drafted," is much more likely to be believed if it bears the mark of a draft board.
- Access-limiting ability using local variable conventions of GEDANKEN
- Fig. 4
    1. Authentication: argument to `Unseal` must have `I1`, `Mark1` is local, so `Seal1` must have been called
    2. Limited access: `Y` must come from `Seal`, `Mark2, Password` are local
- Occam's razor, use this impl
- More motivation, easy to add many trademarks to one objecjt
  - number pairs -> intervals, cartesian coords, polar coords -> complex number, vertex in a geometry
  - Only have to check for the mark you care about
  - Whereas, w/ seals you'd need every unseal and apply in correct order

### Access Keys

- `Seal` public, `Unseal` private

### Authentication vs. Access Control

- Chicken-egg relationship
  - Access to authentication operation must be controlled
  - Gaining access sometimes requires providing authenticated evidence of its access rights
- Access control is necessary for protection
- We believe that authentication is a notion orthogonal to limitation of access. Our belief rests upon intuition and the fact that the former seems difficult to simulate with the latter.
- Ideas presented very general, admit substantial runtime costs

## Programming with Abstract Data Types

- "This paper presents an approach which allows the set of built-in abstractions to be augmented when the need for a new data abstraction is discovered", a "computer representation of abstraction"
- "If a language is to be used at all, it is likely to be used to solve problems which its designer did not envision, and for which the abstractions embedded in the language are not sufficient."
- Trend of very-high-level languages, which "attempts to present the user with the abstractions (operations, data structures, and control structures) useful to his application area"
- Structured programming: "a problem is solved by means of a process
of successive decomposition"
  - "write a program which solves the problem but which runs on an abstract machine, one which provides just those data objects and operations which are ideally suited to solving the problem"
  - "The programmer is initially concerned with satisfying himself (or proving) that his program correctly solves the problem. In this analysis he is concerned with the way his program makes use of the abstractions, but not with any details of how those abstractions may be realized."
  - "Each abstraction represents a new problem, requiring additional programs for its solution", may introduce "further abstractions"
  - "The original problem is completely solved when all abstractions generated in the course of constructing the program have been realized by further programs"
- Relation between structured programming & very-high-level languages:
  - "each is based on the idea of making use of those abstractions which are correct for the problem being solved"
  - "rationale for using the abstractions is the same... to free the programmer from concern with details not relevant to the problem he is solving"
- "A structured programming language... contains no preconceived notions about the particular set of useful abstractions... must provide a mechanism whereby the language can be extended to contain the abstractions which the user requires"
- "a general-purpose, indefinitely-high-level language"

## The Meaning of Abstraction

- "a mechanism which permits the expression of relevant details and the suppression of irrelevant details"
- Like previous paper, procedures get us pretty far, but not a "sufficiently rich vocabulary of abstractions"
- "An abstract data type defines a class of abstract objects which is completely characterized by the operations available on those objects"
- Comparison to built-in types: used at one level, realized at a lower level
  - built-in types: the language itself & compiler
  - ADTs: operation cluster: defines the type in terms of the operations
  which can be performed on it
    - language allows use of ADT w/o requiring defn
    - language processor builds links between use and defn
- Most operations will belong to clusters, remaining operations given term functional abstraction

## The Programming Language

- Clu
  - developed at MIT
  - derived from PASCAL
  - strongly typed
- Module system, no conventionally free variables, only names of other modules

## The Example

- `Polish_gen` translate from an infix language to a Polish post-fix language
- Assumptions:
  1. The input language has an operator precedence grammar
  2. Symbols are arbitrary strings of alphanumeric characters OR a single non-alphanumeric character
- Example:
  - Input: a + b * (c + d)
  - Output: a b c d + * +
- Chosen:
  - Familiar to people interested in PL
  - Sufficiently complex to use many abstractions
- Fig. 1, 2
- Abstractions:
  - ADTs:
    - `infile`: holds the sentence of the input language
    - `outfile`: accept a sentence of the output language
    - `grammar`: recognize symbols of the input language and determine their precedence relations
    - `stack`
    - `token`
  - Functional abstractions:
    - `scan`
- Strong typing:
  1. Use ADT operations
  2. Pass ADT to procedure that has param of exact type
  3. Assigned to variable if has exact type
- operation call: compound name type_name$operation_name
  - type name included reduces ambiguity, avoids clash of identifiers, enhance understandability
- in ">" case, "Since that operator token may have a higher precedence than t, the boolean variable mustscan is used to prevent a new symbol from being scanned and to insure the next comparison is with the current value of t"
- types `infile` and `outfile`, which are used to shield `Polish_gen` from any physical facts concerning its input and output, respectively

## Defining Abstract Data Types

- Fig. 3
- Consider `stack`, type is not known in advance, parameter `element_type` indicates the type of element a particular stack object is to contain
  - `is`: list of the operations defining the type which the cluster implements
  - Remaining:
    1. Object representation
      - Outside, indivisible, inside, decomposable based on rep
    2. Code to create objects
      - Reserved word `create`
    3. Operation defns
      - Have access to rep: "the cluster may simultaneously support many objects of its defined type, this parameter tells the operation the particular object on which to operate"

## Controlling the Use of Information

- the programs which result are more modular, and easier to understand, modify~ maintain and prove correct
- "The proof of a program is divided into two parts: a proof that the cluster correctly implements the type, and a proof that the program using the type is correct. Only in the former proof need details of the implementation of type objects be considered; the latter proof is based only on the abstract properties of the types, which may be expressed in terms of relations among the characterizing operations for each type."
