+++
image = "/img/post-bg.jpg"
categories = ["Flow-based Programming","Computer Science"]
tags = ["fbp","dataflow","development"]
date = "2012-06-03T15:56:29+01:00"
title = "Comparison of dataflow models"
comments = false
draft = false
description = "Comparison of Flow-based programming, Actor model and Communicating Sequential Processes in a scientific manner"

+++

# Comparison of dataflow models

## Mathematical model of choice

Comparison criteria and analysis methods should be chosen before performing actual analysis.

We will solve the task of parallel computation model evaluation as a multiple criteria choice. In order to do this, a set of particular criteria is outlined and for each of them a mapping to the real numbers set is defined:

```
k(x):i(x)→R(1)
```

where `x` - is alternative choice (model);
`i(x)` - is a particular factor value;
`R` - is the set of real numbers;
`k` - is the particular criterion number, `k∈[1,K]`;
`K` - total number of particular criteria.

The following relations between possible particular criterion values `yi` are used:

1. `yi≈yj` - equivalence relation. Two values are equivalent if none of them is more preferable than the other. Every value is equivalent to itself.
2. `yi<yj,i≠j` - preference relation. Value `yj` is more preferable than `yi` if it has benefits over it for a decision maker.
3. `yi≪yj,i≠j` - strong preference relation. Value `yj` is a lot more preferable than `yi` if it has benefits of new quality over it for a decision maker. Strong preference is a type of preference relation.
4. `yi≤yj` -  non-strict preference relation. Value `yj` is more preferable or equivalent to `yi`.

Then `k`'th criterion numeric evaluation function (1) can be written as the following recursive formula:

```
k(yi)=0,∀j:yi≤yjk(yj)+P,yj<yik(yj)+Q,yj≪yi(2)
```

Here `P` and `Q` — are constant weights set by a decision-maker which determine the extent of difference between particular choices. The most trivial example is `P=1, Q=2`.

For multi-criteria decision making we will use a simple *weighted sum model* (WSM).  We will use *linear convolution method* to get a scalar quality estimate `Y` of a model `x` because the importance of each of the particular criteria is defined by a decision maker. Then generalized quality factor looks like:

```
Y(x)=∑i=1Kαi⋅i(x)→max(3)
```

where `αi>0`  is the `i`'th criterion significance (weight) defined by a decision maker. If a normalized estimate is being calculated, the following limitation takes place:

```
∑i=1Kαi=1
```

We will group particular criteria into several semantic groups. Then we can calculate values for denormalized factors for each group and then sum up all particular sums to get the generalized factor as shown in formula (4):

```
Y(x)=∑m=1Mβm⋅Ym(x)(4)
```

where `M` — is the quantity of semantic groups of criteria;
`βm>0` - significance (weight) of the `m`'th group.

## Particular model evaluation factors

Below there is a multilevel list of evaluated properties of the dataflow models. At the top level there are groups of criteria, then there are factors themselves nested and lists of possible values inside them. Numeric scale constants calculated using formula (2) are given in parenthesis.

1. Ease of development — these factors are related to how easy, quickly and convenient application development is.

    1. *Structure decomposition* — means explicit application structure definition as a graph, which allows the use of methods and utilities of structural analysis and significantly simplifies perception of the model by both specialists and non-specialists:

        1. *not emphasized (0)* — means that structure is not emphasized explicitly which is usual for models supporting text notation only;

        2. *2-3 levels (P)* — models with a fixed number of decomposition levels are relatively simple but they are not so powerful in terms of complex systems design as multilevel models;

        3. *multilevel (Q)* — especially effective to design large and complex software systems.

    2. *Notation support* — determines which kind of a language the structure and behavior of the system can be defined in:

        4. *text (0)* — textual languages are usually effective for behavior definition but it is not so for application structure definition and message routing;

        5. *visual (0)* — visual notation is very convenient for structural analysis and drawing of communication graphs but may be too cumbersome for behavior definition;

        6. *double (Q)* — usually means that the structure is represented visually while behavior of particular components is defined with a textual algorithmic language which is the most practical solution.

    3. *Iteration facilities* — presence of special elements and tools to implement iteration and recursion which simplifies application development:

        7. *not emphasized (0)* — iteration is implemented manually using complex constructs with back-coupling loops;

        8. *recursion* (P) — iteration can only be implemented using recursion;

        9. *control structures* (P) — iteration can be implemented using control structures within component blocks;

        10. *specific elements (Q)* — the language has specific elements to implement iteration easily as well as recursion and built-in control structures.

2. Performance and resource consumption — this group of factors affects application scalability, response time, throughput, CPU time consumption, RAM consumption and other performance characteristics.

    4. *Message passing* — depending on whether a handshake from both sides is required to finish the message transfer successfully:

        11. *synchronous (0)* — in this case the sender waits until the receiver receives the whole message or until the message is buffered. This mode is easier to control but it may result into higher latency;

        12. *asynchronous (P)* — the sender does not wait until the message is received in full and proceeds to its further actions. This mode tends to use CPU time more effectively especially in presence of long input/output operations.

    5. *Message buffering* — defines whether the runtime buffers the messages:

        13. *none (0)* — if the message is passed from the sender to the receiver directly and both processes are blocked during message transfer;

        14. *limited (P)* — if non-blocking message passing is supported using bounded buffers; if the queue length reaches the buffer capacity limit, then the sender is blocked;

        15. *unlimited (Q)* — if the system provides unbounded buffer size in an explicit or an implicit way. This may result into higher memory consumption but guarantees that the message is eventually delivered while the sender can act without being blocked.

    6. *Process persistence* — this aspect is related to a fact whether application components are resident in memory during its lifetime or not:

        16. *non-resident* (0) — in such systems processes are spawned to process a new incoming portion of data and terminated when the result of processing is sent along the communication channel. It preserves from memory leaks and saves memory for otherwise rarely used subnets;

        17. *resident* (0) — such processes reside in memory for all the time that application runs which lets them respond faster (they don't need to spend time for initialization) but this increases memory consumption and leak risk (if a garbage collector is not used);

        18. *by choice* (P) — the paradigm and the framework allows a user to develop programs consisting either from resident or from non-resident components depending on what the particular application requires;

        19. *combined* (Q) — in this case some of the components can be made resident  (e.g. if a fast response is required from them) while some of the components within the same network can be non-resident (for example if it is rarely used and its resources can be effectively allocated on demand).

    7. *Parallelism facilities* — which mechanism of parallelism is used on operating system (OS) level:

        20. *processes (0)* — if there are multiple OS processes using interprocess communication (IPC) signals;

        21. *threads (0)* — OS threads communicating via shared memory;

        22. *coroutines (0)* — lightweight processes, "green" threads or coroutines are effectively multiplexed within one OS thread/process;

        23. *combined (Q) —* a combination of facilities from a) to c) are used, e.g. threads + lightweight processes.

3. Flexibility and extensibility — these factors affect the variety of tasks that can be covered by a particular model and how easily the existing programs which have been built using that model can be extended in future.

    8. *Structure changes* — means the possibility to create and modify subnets in application graph during runtime:

        24. *static* (0) — such a structure is defined by an engineer once during design/implementation stage and requires the application to be recompiled and restarted in case of a change;

        25. *dynamic* (Q) — such a structure allows the system to change during runtime, up to continuous evolution of the system.

    9. *Feedback loops* — defines whether the model supports feedback loops, i.e. signals can be passed from ports down the data stream to the ports up the regular data stream:

        26. *no* (0) — some models forbid feedback loops explicitly to avoid possible deadlocks a priori. This results into a smaller application domain and complicates implementation of iterative algorithms;

        27. *yes* (P) — feedback loops may cause deadlocks in some cases but allow an engineer to solve a wider variety of problems in a more simple way.

    10. *Control signals* — represents the emphasis of control signals as a separate type of signals:

        28. *not emphasized* (0) — in this case control parameter values are passed directly to regular input and a component should distinguish them and ensure a correct response itself;

        29. *initial* (0) — in this case control parameter values are entered only once during application initialization which allows a developer to modify system's behavior from one run to another;

        30. *dynamic* (P) — in this case control signals can arrive to a component from both system's environment and other higher-level components during runtime which provides a higher level of flexibility and parametrization.

    11. *Non-determinism* — on the one hand it affects the spectrum of solvable tasks and on the other it affects system's reliability ambiguously [32]:

        31. *deterministic* (0) — deterministic systems are simple and easy to detect errors but they don't ensure guaranteed service and refuse requests more often;

        32. *bounded nondeterminism* (P) — systems with bounded nondeterminism solve a wider spectrum of tasks and have a lower fault probability (usually by means of message queues);

        33. *unbounded nondeterminism* (Q) — models supporting unbounded nondeterminism in theory solve the widest variety of tasks possible while being able to guarantee that a request will be served even if the system load continues to grow.

## Comparison analysis of interaction models based on data flows

Below there are tables representing the results of comparison of Flow-Based Programming (FBP), Actor model (Actor) and Communicating Sequential Processes (CSP) as theoretic component interaction models and practical frameworks. The "Ideal" column is somewhat what a perfect hypothetical model would provide.

The marks specified  in parenthesis are the points values of the scale:

* 0 (not specified) -  no certain advantage;
* P – definite advantage;
* Q – higher order advantage which gives a more possibilities than 0 and P.

FBP is considered as Morrison's JavaFBP/C#FBP in first place, Actor model is mostly analyzed from its Erlang implementation, CSP example is the default Go language's runtime.

### Table 1 — Comparison by ease of development criteria

<table>
  <tr>
    <td>Factor</td>
    <td>Wt.</td>
    <td>FBP</td>
    <td>Actor</td>
    <td>CSP</td>
    <td>Ideal</td>
  </tr>
  <tr>
    <td>Structure decomposition</td>
    <td>1</td>
    <td>Multilevel (Q)</td>
    <td>Not emphasized</td>
    <td>Not emphasized</td>
    <td>Multilevel (Q)</td>
  </tr>
  <tr>
    <td>Notation support</td>
    <td>1</td>
    <td>Double(Q)</td>
    <td>Text</td>
    <td>Text</td>
    <td>Double (Q)</td>
  </tr>
  <tr>
    <td>Iteration facilities</td>
    <td>1</td>
    <td>Not emphasized</td>
    <td>Recursion (P)</td>
    <td>Control structures (P)</td>
    <td>Specific elements (Q)</td>
  </tr>
  <tr>
    <td>Total</td>
    <td></td>
    <td></td>
    <td>2Q</td>
    <td>P</td>
    <td>P</td>
  </tr>
</table>


### Table 2 — Performance and resource consumption related factors comparison

<table>
  <tr>
    <td>Factor</td>
    <td>Wt.</td>
    <td>FBP</td>
    <td>Actor</td>
    <td>CSP</td>
    <td>Ideal</td>
  </tr>
  <tr>
    <td>Message passing</td>
    <td>1</td>
    <td>Synchronous</td>
    <td>Asynchronous (P)</td>
    <td>Synchronous</td>
    <td>Asynchronous (P)</td>
  </tr>
  <tr>
    <td>Message buffering</td>
    <td>1</td>
    <td>Limited (P)</td>
    <td>Unlimited (Q)</td>
    <td>Limited (P)</td>
    <td>Unlimited (Q)</td>
  </tr>
  <tr>
    <td>Process persistence</td>
    <td>1</td>
    <td>By choice (P)</td>
    <td>Non-resident</td>
    <td>Combined (Q)</td>
    <td>Combined (Q)</td>
  </tr>
  <tr>
    <td>Parallelism facilities</td>
    <td>0</td>
    <td>Threads</td>
    <td>Combined (Q)</td>
    <td>Combined (Q)</td>
    <td>Combined (Q)</td>
  </tr>
  <tr>
    <td>Total</td>
    <td></td>
    <td></td>
    <td>2P</td>
    <td>P+Q</td>
    <td>P+Q</td>
  </tr>
</table>


### Table 3 — Flexibility and extensibility criteria comparison

<table>
  <tr>
    <td>Factor</td>
    <td>Wt.</td>
    <td>FBP</td>
    <td>Actor</td>
    <td>CSP</td>
    <td>Ideal</td>
  </tr>
  <tr>
    <td>Structure changes</td>
    <td>1</td>
    <td>Static</td>
    <td>Dynamic (Q)</td>
    <td>Dynamic (Q)</td>
    <td>Dynamic (Q)</td>
  </tr>
  <tr>
    <td>Feedback loops</td>
    <td>1</td>
    <td>Yes (P)</td>
    <td>Yes (P)</td>
    <td>Yes (P)</td>
    <td>Yes (P)</td>
  </tr>
  <tr>
    <td>Control signals</td>
    <td>1</td>
    <td>Initial (P)</td>
    <td>Not emphasized</td>
    <td>Not emphasized</td>
    <td>Dynamic (Q)</td>
  </tr>
  <tr>
    <td>Non-determinism</td>
    <td>1</td>
    <td>Bounded (P)</td>
    <td>Unbounded (Q)</td>
    <td>Unbounded (Q)</td>
    <td>Unbounded (Q)</td>
  </tr>
  <tr>
    <td>Total</td>
    <td></td>
    <td></td>
    <td>3P</td>
    <td>P+2Q</td>
    <td>P+2Q</td>
  </tr>
</table>


Note that Parallel facilities is implementation-specific, so we exclude them from the Total sums. Most of the other criteria are model-specific.

Now we can sum the group values and get the final scalar estimates. In order to get them as simple integers we can specify P and Q scale constants. In this simple example we set all Weights (Wt.) to 1, `P = 1` and `Q = 2`.

### Table 4 — Consolidated comparison results table

<table>
  <tr>
    <td>Group of factors</td>
    <td>Wt.</td>
    <td>FBP</td>
    <td>Actor</td>
    <td>CSP</td>
    <td>Ideal</td>
  </tr>
  <tr>
    <td>Ease of development</td>
    <td>1</td>
    <td>2Q (4)</td>
    <td>P (1)</td>
    <td>P (1)</td>
    <td>3Q (6)</td>
  </tr>
  <tr>
    <td>Performance and resources</td>
    <td>1</td>
    <td>2P (2)</td>
    <td>P+Q (3)</td>
    <td>P+Q (3)</td>
    <td>P+2Q (5)</td>
  </tr>
  <tr>
    <td>Flexibility and extensibility</td>
    <td>1</td>
    <td>3P (3)</td>
    <td>P+2Q (3)</td>
    <td>P+2Q (3)</td>
    <td>P+3Q (7)</td>
  </tr>
  <tr>
    <td>Total</td>
    <td></td>
    <td></td>
    <td>5P+2Q (9)</td>
    <td>3P+3Q (9)</td>
    <td>3P+3Q (9)</td>
  </tr>
</table>


By customizing Weight values, P and Q constants you can get different estimates which match your own priorities better.

There is no actual "best" model, all these models are in the Pareto set.

The FBP model is the most comfortable to use at software design stage and then during implementation because it supports structure analysis and decomposition, double notation (textual description of component behavior and graphical representation of overall application). But actor and CSP models, which are very close to each other, can handle a wider class of tasks and support dynamic restructuring at run-time up to unbounded nondeterminism. The difference between CSP and Actor models is in message passing semantics details, but they are almost identical when using large buffers in CSP.

Let's outline the weak points and the ways that each of these models or platforms can be improved.

**FBP**:

* Static structure. For the model to be really flexible and universal, it needs to support dynamic restructuring so that an application would be capable of changing entire subnets at run-time. Another major problem is that it is hard to display such dynamic structure (especially if changes are frequent and of variable structure themselves)  on graphical diagrams, which are the strongest points of this model.

* As a consequence, this model supports bounded nondeterminism. Shortly, this means that the system cannot guarantee normal functioning behind the boundaries where its behavior is determined.

* Facilities for recursion and iteration handling would also improve this model, so that it would make it clear and easy how to implement cyclic and iterative structures.

**Actor model:**

* The main disadvantage of most existing actor model implementation is that the only notation supported is textual. Historically this model has been utilized in functional programming languages which rarely have a visual presentation. This makes it hard to apply structure decomposition techniques and maintain the system. A possible solution is to split the system into 2 levels using different notations: high level coordination language (visual or textual) and low level textual component behavior description.

**CSP** has the same problems as the actor model in general. But another thing a CSP developer has to care about is capacity of each buffer in communication channels, which in reality depends on an exact application and current inbound data intensity.

