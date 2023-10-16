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
Over the past weeks I have been trying to learn about Data Oriented Designing and how and where it is applicable. On this pursuit, I got to read a paper written by Ben Moseley and Peter Marks in 2006 titled "*Out of the tar pit*". Following is the summary on how complexity is present in out system design and how we can tackle it.
## Understanding a system
Before we can go ahead to identify complexity we need to understand how systems are understood. It can be done in two major ways:
- **Reading the source code**: The paper calls it *informal reasoning*. It is pretty straightforward, however, the more complex the system design is, the harder it is to understand.
- **Testing**: Here, we assume the system to be a *black box*. We then provide a set of inputs and validate the outputs. This however is limited, in the sense that a set of inputs does not tell us anything about the behaviour of the system on other sets of inputs.

With these in mind, we can say that with an improvement in the system's design will improve its comprehensibility via *informal reasoning*. And in comparison, better testing will only lead to more error being detected. In any system we, of course, need to work on both of these to build a more reliable software.
## Identification of complexity
In any given system, an inherent complexity exists due to how the problem statement needs to be solved. This complexity is unavoidable. We can call this `essential complexity`.

Apart from this a system can have complexity due to the infrastructure and the tools used to build the system. We can call this `acciental complexity` as it has nothing to do with the solution to the problem statement, rather it is there due to how the infrastructure and the tools are used to get to the required solution. This can be, for example, some language constructs that the developer needs to use that could have been avoided if another language would have been chosen.

As the system grows, this `accidental complexity` will only grow due to difficulty in understanding and navigating it. Complexity breeds complexity.

I would like to point out that by *infrastructure and tools*, I mean everything from the programming language to the type of server and database, etc.

In an ideal world, we would only have to deal with *essential complexity*. However, in a real world, we do have to deal with some *accidental complexity*. The idea is not to deny the latter, but to acknowledge them both.

To understand where these complexity arises from, let us break down a system into two parts:
- **State**: These are the data on which the system operates on to provide a required output. This can be anything from user inputs, system generated and, those obtained from other systems.
- **Control Flow**: These are the logic with which the system operates on the data to provide required outputs.
### Complexity due to State
To understand the complexity caused by *state* we have to look at the impact it has on both *testing* and *informal reasoning*.
#### Impact on testing
Testing a system on a given *state* will tell you nothing about the behaviour of the same system on a different *state*. As such, the system needs to be tested on all possible states. For example, imagine if a system flows through different states, say it can go from A to B or to C, the same has to be tested. As the number of *states* increases, the number of tests would also grow.
#### Impact on informal reasoning
*States* have an impact on our informal reasoning of the system since we have to understand an remember how they affect the system. As the number of *states* grows, we have to remember the as many or more scenarios the system can be affected in. This makes it harder to reason about the system.
### Complexity due to Control Flow
A solution to a problem, if solved using a programing language, will need to be understood, via *informal reasoning*, in the context of that programming language. It may be, that due to the construct of chosen language, some complexity from it will sip into the description of the solution.

Say, for example (taken from the paper itself), if we consider the code below
```cpp
int a = b + 3;
int c = d + 2;
int e = f + 4;
```
After going through the code snippet, we can clearly see that the order of these lines do not matter. It is the construct of the language that enforces this arbitrary ordering; which then, via further *informal reasoning*, we have to omit out this ordering as part of the requirements to the solution.
#### Impact due to Concurrency
Concurrency, if present, also makes it difficult to reason about the control flow of the solution. Combined with *states*, it makes *informal reasoning* further complex. It also makes testing difficult. Testing a concurrent system does not tell us about the behaviour of the same system in subsequent runs.
## Recommended General Approach


---
Spandan Buragohain, 2023-10-08