This document covers each of the core components that constitute a skill. It is important to note that Watson Assistant follows an ontology based paradigm, meaning that it derives understanding about the world based on a relationship of concepts. This is in contrast to most other systems which are intent based and depend on predicate logic.

At a high level, the assistant is composed of an [ontology](#ontology) of concepts that it is able to recognize and reason about. Natural langauge [patterns](#patterns) help the assistant to perform natural langauge understanding, linking language tokens to appropriate concepts. In the process of reasoning about a question, the system considers all possible outcomes it can take given a set of [actions](#actions) and [rules](#rules), ultimately selecting the best one or asking for user disambiguation in cases where it is fitting.

### Ontology

We begin our understanding of ontology by first looking at the notion of a concept. A concept is simply an object in your domain that the assistant is able to recognize. For example, in the marketing domain, we have concepts for mailings, open-rates, click-rates, etc. A concept in denoted by a colon `:`. For example `:Mailings` denotes the mailings concepts. Concepts should additionally be prefixed with the domain of your skill. For example, `marketing:Mailings` signifies the mailings concept in the marketing domain. 

Concepts and their relationship to other concepts are definined in an ontology. In fact, an ontology is simply a set of relationships between different concepts in rdf format, i.e. *subject-predicate-object* triples. Consider the following examples below. 

```
:OpenRate subClassOf :Showable
```

Here we specify that :OpenRate is a concept which subclasses or dervies from the :Showable concept. This system is able to visually render all :Showable objects, so we are effectively making our open rate concept viewable to the end user.


```
:Mailings isListOf :Mailing
```

Here we specify that the :Mailings concept is a list composed of individual :Mailing concepts. This is useful for actions that the assistant performs when working with collections.

#### Attributes

Typically concepts are used to model well formed business entities within our domain, and, consequently, the assistant is able to understand a concept as being composed of a set of attributes. For example, the :Mailing concept will contain attributes for subject line, recpeint, send date, etc. Such a relationship might be specified in the skill's ontology as follows.

```
:SubjectLine subClassOf :Attribute
:Mailing hasAttribute :SubjectLine
```


### Patterns

Patterns are simply natural langauge text samples annotated with concepts. When the assistant receives a question from the user, it is able to tokenize and parse this input into a tree. By using natural language patterns, it is able to annotate this tree with the appropriate concepts. For example consider the patterns below.

```
show {trending|wmt:Trending} {products|wmt:Products}
```

This pattern tells the system that trending and products tokens correspond to the concepts `wmt:Trending` and `wmt:Products` respectively. From now on, the system will recognize and consider "trending" and "products" approriately for all further input.


### Actions

Actions are the means by which the assistant can provide real data to corresponding concepts. Each skill will contain a set of actions designated by the following signature: `constraints => input -> output`. Given an input configuration of concepts subject to certain constraints, the system will be able to produce the specified output given the body of the action. In addition to this signature, an action will have a name as well as a function to be invoked when the action is selected. For example consider the action below. 

```
:Products(:Trending) -> data :Products
```

Note that this action does not have any constraints--constraints are optional. This action effectively states that if the assistant recognizes a :Product concept as well as a :Trending concept, then the system can produce real data for the :Products concepts. The data will be produced by the body of this action, which can query a database, call an api, etc.

[Node.js modules](NodeModules.md)

### Rules

Rewriting rules are a way to transform one single concept into a cluster of concepts assuming the context. Often times natural language can be very short and allow for the omission of certain words or concepts. Rewriting rules enable the assistant to handle such cases effectively. While our concepts are well formed, we anticipate that users will use imprecise langauge to ask about them. For example, we might expect a user to ask for "the best mailing" or "trending products". We can implement rewriting rules to reduce higher level notions such as "best" and "trending". For example consider the rewriting rule below.

```
:TheBest(:Mailing) -> :HighValue(:ClickToOpenRate, :Mailings)
```

Here we have defined the best mailing to be reproduced into lower level concepts. :HighValue is a concept used by the system to return highest quartile data points and :ClickToOpenRate is an attribute of :Mailing. We are effectively programming our assistant to recognize "the best mailings" as being equivalent to "mailings with clickToOpenRate attribute in the highest quartile".