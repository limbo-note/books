- [2. Development Process](#2-development-process)
	- [Iterative and Waterfall Processes](#iterative-and-waterfall-processes)
	- [Predictive and Adaptive Planning](#predictive-and-adaptive-planning)
	- [Agile Processes](#agile-processes)
	- [Rational Unified Process](#rational-unified-process)
	- [Fitting a Process to a Project](#fitting-a-process-to-a-project)
	- [Fitting the UML into a Process](#fitting-the-uml-into-a-process)
- [3. Class Diagrams: The Essentials](#3-class-diagrams:-the-essentials)
	- [Properties](#properties)
	- [Multiplicity](#multiplicity)
	- [Bidirectional Associations](#bidirectional-associations)
	- [Operations](#operations)
	- [Generalization](#generalization)
	- [Note and Comments](#note-and-comments)
	- [Dependency](#dependency)
	- [Constraint Rules](#constraint-rules)
	- [A simple class diagram](#a-simple-class-diagram)
- [4. Sequence Diagrams](#4-sequence-diagrams)
	- [Creating and Deleting Participants](#creating-and-deleting-participants)
	- [Loops, Conditionals, and the Like](#loops,-conditionals,-and-the-like)
- [5. Class Diagrams: Advanced Concepts](#5-class-diagrams:-advanced-concepts)

# 2. Development Process

**Rational Unified Process (RUP)** is one process framework that you can use with the UML.
### Iterative and Waterfall Processes

- waterfall: requirements analysis -> design -> coding -> testing
- iterative: first iteration (complete software life cycle, including analysis, design, code, and test) -> second iteration (improve the software based on first iteration) -> ...

Recommend that projects **do not** use a pure waterfall approach, iterative approach can often involve rework (Refactoring) which helps work more efficiently

### Predictive and Adaptive Planning

1. Don't make a predictive plan until you have precise and accurate requirements and are confident that they won't significantly change.
2. If you can't get precise, accurate, and stable requirements, use an adaptive planning style.

### Agile Processes
strongly adaptive, people-oriented processes

### Rational Unified Process
RUP is essentially an iterative process. It should follow 4 phases:
- **Inception**. initial evaluation of a project
- **Elaboration**. have a good sense of the requirements and a skeletal working system that acts as the seed of development
- **Construction**. develop enough functionality to release
- **Transition**. 

### Fitting a Process to a Project

(.........)

### Fitting the UML into a Process

(.........)

# 3. Class Diagrams: The Essentials

### Properties
- Attributes (describes a property as a line of text within the class box)
	- form: `visibility name: type multiplicity = default {property-string}`
		- visibility: public(+), private(-)
		- name: name of the attribute
		- type: type of the attribute
		- multiplicity
		- default value
		- {property-string}: additional properties for the attribute
	- example: `name: String [1] = "Untitled" {readOnly}`
- Associations（实线箭头） (a solid line between two classes, directed from the source class to the target class)（包括自关联）
	- The name of the property goes at the target end of the association, together with its multiplicity
	- The target end of the association links to the class that is the type of the property
- examples:									
	![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-1.jpg)
	![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-2.jpg)

### Multiplicity
- `1` (exactly one)
- `0..1` (may or may not have a single)
- `*` (no upper limit)	
- `1`equals`1..1`, `0..*`equals`*`
- `Optional`: lower bound = 0
- `Mandatory`: lower bound >= 1
- `Single-valued`: upper bound = 1
- `Multivalued`: upper bound > 1

### Bidirectional Associations
![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-3.jpg)
or											
![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-4.jpg)
- The Car class has property `owner:Person[1]`, and the Person class has a property `cars:Car[*]`.
- When coding, let one side of the associations (single-side better if possible) control the relationship

### Operations
- form: `visibility name (parameter-list): return-type {property-string}`
- example: `+ balanceOn (date: Date) : Money`

### Generalization
- similar to **inheritance（实线空心三角）/interface（虚线空心三角）** in OO languages, for example, `Customer` is Generalization, with `Personal Customer` and `Corporate Customer` as subtypes

### Note and Comments

![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-5.jpg)

### Dependency

（虚线箭头）								
![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-6.jpg)

### Constraint Rules

### A simple class diagram

![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/3-7.jpg)

# 4. Sequence Diagrams

- centralized control & distributed control
	![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/4-1.jpg)
	![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/4-2.jpg)

### Creating and Deleting Participants

![](https://github.com/limbo-note/books/blob/master/UML_DISTILLED/4-3.jpg)

- To create a participant, you draw the message arrow directly into the participant box
- Deletion of a participant is indicated by big X

### Loops, Conditionals, and the Like

(.........)

# 5. Class Diagrams: Advanced Concepts
