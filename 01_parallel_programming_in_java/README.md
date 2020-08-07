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