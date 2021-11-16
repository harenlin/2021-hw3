# System Programming 2021: HW3-PseudoThread
Due on 2021/12/14 23:59

## 1. Goal
0. Understand and simulate the concept of context switch.
1. Understand and learn some system calls about signals.
2. Understand and learn to use `setjmp()`, `longjmp()`.
3. Enjoy the journal in system programming.

## 1. Problem Description
In this programming assignment, we are going to simulate a user-thread library
 by using `setjmp()`, `longjmp()`, etc. For the simplicity, we use a function 
to represent a single thread. In other words, you are required to do 
"context switch" between functions. To do this, you need to do non-local jumps 
between functions, which is arranged by a `scheduler()`: each time a function 
needs to context switch to another, it needs to jump back to `scheduler()`,
and `scheduler()` will schedule next function to be executed and jump to it. 
Since non-local jump will not store local variables, we define a data structure,
 `TCB_NODE`, for each function to store data needed for computing. All of the 
`TCB_NODE`s will formulate a circular linked-list. As `scheduler()` schedules 
functions, it also needs to make sure `Current` pointer points to correct `TCB_NODE` 
for functions to output correct execution result.

You are expected to complete the following tasks:

1. Complete the user-thread library `threadtools.h`.
2. Complete the functions required in `scheduler.c`.
3. Implement three functions: `ReduceInteger()`, `MountainClimbing()`, and `OperationCount()` in `threefunctions.c`.

## 2. main.c
You **DON'T** have to change any code in `main.c`. But, it's 
better for you to understand the workflow in `main.c`. You 
should complete all your homework **WITHOUT** changing any 
variables or inserting any code segement in `main.c`.

## 3. threadtools.h
Here, we define the data structure of `TCB_NODE` and all global variables needed for `scheduler.c` 
and `threefunctions.c`. Moreover, you are required to complete some macro functions in this file. 
Please refer to the comments in `threadtools.h`.

## 4. scheduler.c
You need to implement `sighandler()` and `scheduler()` in `scheduler.c`. The `sighandler` is the 
requirement as a signal handler in system call `sigaction()`. 
(To know more: https://pubs.opengroup.org/onlinepubs/007904875/functions/sigaction.html)
Please refer to the comments in `scheduler.c`.

## 5. Rule of "contect switch"
We introduce the rule of "context switch" in detail:
1. "Context switch" means that function jumps back to `scheduler()`, the `scheduler()` 
schedules next function in the circular linked-list to be executed. 
2. "Context switch" may occur in two different scenarios:
    - After each iteration in a function. (Please refer to the spec in next section.)
    - Signal caught (we only consider `SIGTSTP`), timeslice reached.
2. If you're doing context switch after each iteration, you should change the function 
you're executing at the end of each iteration.
3. If you're doing context switch on signal caught or timeslice reached, you should 
check the pending signals at the end of each iteration. Once you get the pending signal(s), 
you should do "context switch".
4. Timeslice is the time that a function can execute between context switch, 
which is implemented by `alarm()` system call. Typically the timeslice is set to 3.
5. If multiple signals are pending after one iteration, `SIGTSTP` has the top priority. 

## 6. threefunctions.c
0. Function name refers to ReduceInteger, MountainClimbing, or OperationCount.
1. Basically, functions are all in the form mentioned below; in the meanwhile you 
are required to solve all of the problem within **`O(n)`** time complexity.
2. Functions should at least print an output line during its timeslice.
3. Functions only context switch at the end of each iteration.
4. Functions terminate on `maxiter` exceeded or stop condition occurred.  
5. You are required to solve all functions via iterative method, instead of recursion.
6. The details of each functions and corresponding steps you need to follow are down below.
 - `ReduceInteger()`: Given a positive integer n: if n is even, replace n with n/2; otherwise, 
you can replace n with either n + 1 or n - 1. Please find the minimum number of replacements 
needed for n to become 1. The function stops when the calculation is done.

 - `MountainClimbing()`: Imagine you are climbing a mountain. Each step you can either climb 1 
or 2 units of height. Please compute how many distinct ways you climb to the top? The function 
stops when the calculation is done.

 - `OperationCount()`: You have an array `arr` of length n where `arr[idx] = (2*idx)+3` for all 
valid values of i (i.e. 0 <= idx < n). In 1 operation, you are allowed choose two indices x and y 
where 0 <= p, q < n and subtract 3 from `arr[p]` and add 3 to `arr[q]` (i.e. 
`arr[p] -= 3` & `arr[q] += 3`). The goal is to make all the elements of the array equal. 
We promised that all the numbers of an array can be made equal using this operations. 
Please calculate the minimum number of operations needed to make all the elements of 
array equal with given integer n, the length of the array. The function stops when the calculation is done.
```
void FunctionName(int thread_id, int init, int maxiter){
    ThreadInit(thread_id, init, maxiter);
    /* Some initilization if needed. */
    for(Current->i = 0; Current->i < Current->N; ++Current->i){
        sleep(1);
        /*  Do the computation, then output result.
            Call ThreadExit() if the work is done. */      
        ThreadYield();
    } ThreadExit();
}
```
In each timeslice, functions should output its current result by:
```
printf("FunctionName: %d\n", your_output_variable);
```

## 7. Execution
This homework will only be executed by the following command:
```
$ ./main {ri_init} {ri_maxiter} {mc_init} {mc_maxiter} {oc_init} {oc_maxiter} {timeslice} {switchmode}
```
Below are argument explainations:
```
ri_init = The initial value pass into ReduceInteger
ri_maxiter = The max iteration for ReduceInteger to process
mc_init = The initial value pass into MountainClimbing
mc_maxiter = The max iteration for MountainClimbing to process
oc_init = The initial value pass into OperationCount
oc_maxiter = The max iteration for OperationCount to process
timeslice = time limit for a function to process until next context switch
switchmode = 0 for context switch after each iteration; 1 for context switch due to signal caught and timeslice reached
```
Sample execution 1
```
$ ./main 0 8 1 10 0 6 3 0
Reduce Integer: 1
Mountain Climbing: 2
Operation Count: 1
Reduce Integer: 2
Mountain Climbing: 3
Operation Count: 4
Reduce Integer: 3
Mountain Climbing: 5
Operation Count: 9
Mountain Climbing: 8
Mountain Climbing: 13
Mountain Climbing: 21
Mountain Climbing: 34
Mountain Climbing: 55
Mountain Climbing: 89
```

Sample execution 2
```
$ ./main 0 8 1 10 0 6 3 1
Reduce Integer: 1
Reduce Integer: 2
Reduce Integer: 3
ALRM signal!
Mountain Climbing: 2
Mountain Climbing: 3
Mountain Climbing: 5
ALRM signal!
Operation Count: 1
Operation Count: 4
Operation Count: 9
Mountain Climbing: 8
Mountain Climbing: 13
Mountain Climbing: 21
ALRM signal!
Mountain Climbing: 34
Mountain Climbing: 55
Mountain Climbing: 89
ALRM signal!
```

Sample execution 3
```
$ ./main 0 8 1 10 0 6 3 1
Reduce Integer: 1
Reduce Integer: 2
Reduce Integer: 3
ALRM signal!
^ZMountain Climbing: 2
TSTP signal!
Operation Count: 1
Operation Count: 4
Operation Count: 9
^ZMountain Climbing: 3
TSTP signal!
^ZMountain Climbing: 5
TSTP signal!
Mountain Climbing: 8
ALRM signal!
Mountain Climbing: 13
^ZMountain Climbing: 21
TSTP signal!
Mountain Climbing: 34
ALRM signal!
Mountain Climbing: 55
Mountain Climbing: 89
```
Note: ^Z is where `SIGTSTP` delivered to process, 
it is not output by process itself.

## 8. Grading
0. (1 pt) Your `threadtools.h` and `scheduler.c` supports context switch after each iteration.
1. (2 pt) Your `threadtools.h` and `scheduler.c` supports context switch by time slice.
2. (2 pt) Your `threadtools.h` and `scheduler.c` supports context switch by signal caught.
3. (2 pt) Your `threefunctions.c` works correctly in O(n) time complexity.
4. (1 pt) Your `threadtools.h`, `scheduler.c` and `threefunctions.c` work fine together.

For all tasks, your code will be run on **CSIE workstation**. In the meanwhile, 
your code will be complied by `MakeFile` in this repostiory. Please make sure your codes are runnable on it.
- In 1. 2. 3., your `threadtools.h` and `scheduler.c` will complied with TA's `main.c` and `threefunctions.c`.
- In 4., your `threefunctions.c` will complied with TA's `main.c` and `threadtools.h` and `scheduler.c`.
- In 5., your `threadtools.h`, `scheduler.c` and `threefunctions.c` will be compiled with TA's `main.c`.
- TA's `threefunctions.c` is pre-compiled as `threefunctions.o` in this repository, use it well. **BUT**, this **DOESN'T** imply that you will get full credit of 0. 1. 2. if your code works fine with TA's `threefunctions.o`. (why?)

## 9. Submission
Your assignment should be submitted to github before deadline. The submission should include three files: 
`threadtools.h`, `scheduler.c`, and `threefunctions.c`.
Your repository may contain other files, but TA will **ONLY** score your homework based on three files mentioned above, please make sure that you named your files correctly.

## 10. Reminder
1. Plagiarism is **STRICTLY** prohibited.
2. Your credits will be **deducted 10% for each day's delay**. Late submission is better than an absence.
3. If you have any question, please leave comments on NTU COOL's Discussion Area.
4. For some code details, please feel free to contact us via email ntucsiesp@gmail.com or come during TA hours.
5. Please start your work as soon as possible, **DON'T** leave it until the last day! 
