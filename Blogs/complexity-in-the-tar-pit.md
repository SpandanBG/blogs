# Complexity in the Tar Pit

---

Over the past weeks, I went though the the paper "Out of the tar pit" written by Ben Moseley and Peter Marks in 2006.

---

- why we need DOD
	- flexibility reasons
	- performance reasons
- Reasons for complexity & why complexity arises
	- state & control
- How to identify complexity
	- approach to identify the different states and controls
- how to navigate the complexities
	-  essential and accidental splits

## Introduction
Over the past weeks I have been trying to learn about Data Oriented Designing and how and where it is applicable. On this pursuit, I got to read a paper written by Ben Moseley and Peter Marks in 2006 titled "*Out of the tar pit*". The paper talks about complexity in our software design and how it can be tackled. Following is a summary on it based on my understanding.
## Understanding a system
Before we can identify complexity, we need to understand how systems are understood. It can be done in two major ways:
- **Reading the source code**: The paper calls it *informal reasoning*. It is pretty straightforward, however, the more complex the system design is, the harder it is to understand.
- **Testing**: Here, we assume the system to be a *black box*. We then provide a set of inputs and validate the outputs. This however is limited, in the sense that a set of inputs does not tell us anything about the behaviour of the system on other sets of inputs.

With these in mind, we can say that with an improvement in the system's design will improve its comprehensibility via *informal reasoning*. And in comparison, better testing will only lead to more error being detected. In any system we, of course, need to work on both of these to build a more reliable software.
## Identification of complexity
In any given system, an inherent complexity exists due to how the problem statement needs to be solved. This complexity is unavoidable. We can call this `essential complexity`.

Apart from this a system can have complexity due to the infrastructure, tools and methods used to build the system. We can call this `accidental complexity` as it has nothing to do with the solution to the problem statement, rather it is there due to how the infrastructure and the tools are used to get to the required solution. This can be, for example, some language constructs that the developer needs to use that could have been avoided if another language would have been chosen.

As the system grows, this `accidental complexity` will only grow due to difficulty in understanding and navigating it. Complexity breeds complexity.

I would like to point out that by *infrastructure and tools*, I mean everything from the programming language to the type of server and database, etc.

In an ideal world, we would only have to deal with *essential complexity*. However, in a real world, we do have to deal with some *accidental complexity*. The idea is not to deny the latter, but to acknowledge them both.

To understand where these complexity arises from, let us break down a system into two parts:
- **State**: These are the data on which the system operates on to provide a required output. This can be anything from user inputs, system generated and, those obtained from other systems.
- **Control Flow**: These are the logic with which the system operates on the data to provide required outputs.
### Complexity due to State
To understand the complexity caused by *state* we have to look at the impact it has on both *testing* and *informal reasoning*.
#### Impact on testing
Testing a system on a given *state* does not reveal its behaviour in other *states*. As such, the system needs to be tested on all possible states. For example, imagine if a system flows through different states, say it can go from A to B or to C, the same has to be tested. As the number of *states* increases, the number of tests would also grow.
#### Impact on informal reasoning
*States* have an impact on our informal reasoning of the system since we have to understand an remember how they affect the system. As the number of *states* increases, we must consider equal or greater number of scenarios affecting the system. This makes it harder to reason about the system.
### Complexity due to Control Flow
A solution to a problem, if solved using a programing language, will need to be understood, via *informal reasoning*, in the context of that programming language. It may be that, due to the constructs of the chosen language, some complexity seeps into the solution's description.

Say, for example (taken from the paper itself), if we consider the code below
```cpp
int a = b + 3;
int c = d + 2;
int e = f + 4;
```
After going through the code snippet, we can clearly see that the ordering of these lines do not matter. It is the construct of the language that enforces this arbitrary ordering; which then, via further *informal reasoning*, we have to omit out this ordering as part of the requirements to the solution. This is a simple example. This can grow with more complex requirements, like polluting a solution with hooks in React, or solving a gameplay mechanics under the context of a game engine. These help with implementing the solution but now you are bound to understand the actual solution first by the context of the framework / language and then further by removing those.
#### Impact due to Concurrency
Concurrency, if present, also makes it difficult to reason about the control flow of the solution. Combined with *states*, it makes *informal reasoning* further complex. It also makes testing difficult. Testing a concurrent system does not tell us about the behaviour of the same system in subsequent runs.
## Recommended General Approach
We have now identified two types of complexities:
- **Essential Complexity**: This is the complexity in the problem statement itself. That it, this complexity cannot be avoided as it is brought in by the problem statement itself.
- **Accidental Complexity**: This is the rest of the complexity which can be brought in by the applied tools and methods to solve the given problem. In an ideal world, this complexity would not exist.

Now, in order to identify each of them in a solution to a problem statement we will go through a sequence of steps. The first of which is to iterate the solution in an ideal world.
### Ideal World
An ideal world in software development would be a world where there are no concerns over performance and, the language and infrastructure provides all the general support that is required. In such a world, we would try and remove all the complexities that can be avoided. What would remain will be the *essential complexity* of the solution.

In order to achieve this we gather the *informal requirements* from user's perspective. This gives us what inputs the users are expected to provide and what outputs the users are expects to see. This *informal requirements* would then be turned to *formal requirements* in order to develop a functioning software. The paper states that this formalization in an ideal world is essentially be there to ensure there wasn't any miss in the *informal requirements*. With the formalized requirements, we can supply this to our ideal world language and infrastructure and we would obtain our executable.

From this formalized *informal requirements*, we can identify the final states and control flows we would be left with.
#### States in Ideal World
The data in the system can either be an input from the user or something that is processed by the system itself (*derived data*). The *input data* cannot be avoided and is crucial to the user; and hence those are *essential states*. The *essential derived data* (mutable / immutable) can be considered *accidental state* since they can be re-derived whenever required. Similarly *accidental derived data* is also *accidental state*. In the ideal world, performance is of no concern and hence the re-derivation will cost nothing.

For *essential mutable derived data*, one thing that we need to note is that if the derived data from user input cannot be converted back to the derived data, then those are essentially can be considered as inputs. An example of such can be creating the hash of a password input.

| Data Essentiality | Data Type | Data Mutability | Classification |
| --- | --- | --- | --- |
| Essential | Input | - | Essential State |
| Essential | Derived | Mutable | Accidental State |
| Essential | Derived | Immutable | Accidental State |
| Accidental | Derived | - | Accidental State |
#### Control in Ideal World
The paper states that all logic in an ideal world should be considered as *accidental*. This is said under the pretext that the even though a software is expected to perform some operations on the input, the *result* should be independent of the the actual control flows used.

What it means is that the operations one can choose to get the result from the input can vary. As such one cannot call one control flow more essential than another. Thus all control flow are accidental.
### Practical World


---
Spandan Buragohain, 2023-10-08