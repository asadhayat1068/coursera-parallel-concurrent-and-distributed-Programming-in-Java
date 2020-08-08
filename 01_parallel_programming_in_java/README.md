**Course 1**  
###### Parallel Programming in Java

# Week 1
## Task Level Parallelism
### 1.1 Task Creation and Termination (Aysnc, Finish)
**async**: `async` notation is used to run a task asynchronously.  
For example `async <stament 1>` causes the parent task to create a new child task to execute the body of `async` asynchronously.

**finish**: `finish` makes the parent task to wait until the code execution in `finish` block is completed.  
`async` and `finish` constructs may be arbitrary nested.  

**Example:**
*[Code block from course notes @ Coursera](https://www.coursera.org/learn/parallel-programming-in-java/lecture/TLz5M/1-1-task-creation-and-termination-async-finish)*

```
    finish {
    async S1; // asynchronously compute sum of the lower half of the array
    S2;       // compute sum of the upper half of the array in parallel with S1
    }
    S3; // combine the two partial sums after both S1 and S2 have finished
```  

****

### 1.2 Tasks in Java's Fork/Join Framework  

Java's Fork/Join (FJ) is a standard java framework used to implement `async` and `finish` functionality. In **FJ**, a task can be specified in compute method of a user defined class that extends the standard `RecursiveAction` class of FJ framework. 

** Example: **  
*[<a href="https://www.coursera.org/learn/parallel-programming-in-java/supplement/wlDUr/1-2-lecture-summary" target="_blank">Example from coursera</a>]*

```
private static class ASum extends RecursiveAction {
  int[] A; // input array
  int LO, HI; // subrange
  int SUM; // return value
  . . .
  @Override
  protected void compute() {
    SUM = 0;
    for (int i = LO; i <= HI; i++) SUM += A[i];
  } // compute()
}
```  
In the example code above, we created `class ASum` with field `A` for the input array, `LO` and `HI` for the subrange for which the sum is to be computed, and `SUM` for the result for that subrange.

For any instance of `ASum`, say `L` (instace of ASum), `L.fork()` will create a new task that executes `L's compute()` method. The call to `L.join()` then waits until the task created by `L.fork()` is completed.  
* `L.fork()` --> `asyn`  

* `join()` is a lower level primitive than `finish` because `finish` waits for all tasks to complete while `join()` only wait specific task.  

FJ tasks are executed in a `ForkJoinPool`, which is a pool of java threads. This pool supports the `invokeAll()` method that combines both `fork()` and `join()` operations by executing a set of tasks in parallel, and waiting for their completion.  

For example, `ForkJoinTask.invokeAll(left,right)` implicitly performs `fork()` operations on `left` and `right`, followed by `join()` operations on both objects.

[Ref: <a href="https://www.coursera.org/learn/parallel-programming-in-java/supplement/wlDUr/1-2-lecture-summary" target="_blank">1.2 Lecture Summary</a>]

***

### 1.3 Computation Graphs, Work, Span
* #### Computational Graph  
  Computational Graph (CG) models the execution of a parallel program as a partially ordered set.  
  Specifically a CG consists of:  
  1. **A set of vertices or node**, in which each node represents a step consisting of an arbitraty sequential computation.  
  2. **A set directed edges**, that represent ordering constraints among group.

  For Fork-Join programs, it is useful to partition the edges into three cases:  
    1. **Continue Edges**, that capture sequencing of steps within a task.
    2. **Fork Edges**, that connect the fork operation to the first step of child tasks.
    3. **Join Edges**, that connect the last step of tasks to all join operations on that task.  

  CGs can be used to define data races. Data race is an important class of bugs in parallel programs. We say that a data race occurs on location `L` in a computation graph, `G`, if there exist steps `S1` and `S2` in `G` such that there is no path of directed edges from `S1` to `S2` or from `S2` to `S1` in `G`, and both `S1` and `S2` read or write `L` (*with at least one of the accesses being a write, since two parallel reads do not pose a problem*).  

  CGs can also be used to reason about the *ideal parallelism* of a parallel program as follows:

  * Define ***WORK(G)*** to be sum of execution times of all nodes in CG. [Total amount of work done by the program]  
  * Define ***SPAN(G)*** to be the length of longest path in G, when adding up the execution time of all nodes in path. The longest paths are known as *critical paths*, so ***SPAN*** also represents the `critical path length (CPL)` of **G**.

  Given above definitions of **WORK** and **SPAN**, we define the *ideal parallelism* of computation graph *G* as the ratio, ***WORK(G) / SPAN(G)***.  
  The *ideal parallelism* is an upper limit on the speed up factor that can be obtained from parallel execution of nodes in computation graph `G`. Note that ideal parallelism is only a function of the parallel program and does not depend on the actual parallelism available in physical computer.

    [Ref: <a href="https://www.coursera.org/learn/parallel-programming-in-java/supplement/EtCZK/1-3-lecture-summary" target="_blank">1.3 Lecture Summary</a>]

***

  ### 1.4 Multiprocessor Scheduling, Parallel Speedup

  In this lecture, we studied the possible executions of a Computation Graph (CG) on an idealized parallel machine with P processors. It is idealized because all processors are assumed to be identical, and the execution time of a node is assumed to be fixed, regardless of which processor it executes on. A legal schedule is one that obeys the dependence constraints in the CG, such that for every directed edge (A, B), the schedule guarantees that step B is only scheduled after step A completes. Unless otherwise specified, we will restrict our attention in this course to schedules that have no unforced idleness, i.e., schedules in which a processor is not permitted to be idle if a CG node is available to be scheduled on it. Such schedules are also referred to as "greedy" schedules.

  Let  
  `T_p` = Execution Time of `CG` on `p` processors,  
  then  
  `T_inf =< T_p =< T_1`  
  Also  
  `T_inf = Span`  
  `T_1 = Work`   

  We also saw examples for which there could be different values of `T_p` for different schedules of the same `CG` on `p` processors.

We then defined the parallel speedup for a given schedule of a `CG` on `P` processors as  
`Speedup(P) = T_1/T_p`  
​	
 , and observed that `Speedup(P) must be ≤ the number of processors P` , and also `≤` the `ideal parallelism, WORK/SPAN.`

