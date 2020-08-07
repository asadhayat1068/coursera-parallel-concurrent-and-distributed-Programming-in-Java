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
*[Example from coursera](https://www.coursera.org/learn/parallel-programming-in-java/supplement/wlDUr/1-2-lecture-summary)*

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

[Ref: [1.2 Lecture Summary](https://www.coursera.org/learn/parallel-programming-in-java/supplement/wlDUr/1-2-lecture-summary) ]