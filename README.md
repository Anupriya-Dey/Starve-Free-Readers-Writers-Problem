# Starve-Free-Readers-Writers-Problem
## CSN-232 Operating System Course Assignment
The readers-writers problem is a classical problem of process synchronization which deals with a data set that is shared by multiple processes at once, like a file. These distinct processes are categorized into two types:
- Readers: They can only read the data set and cannot change it
- Writers: They can read and update the data set

The objective is to allow multiple readers to access the dataset simultaneously while only a single writer can update the data set at a time.
The immediate solution to this problem gives priority to either readers or writers. Thus, it can resut in starvation of either writers or readers in the queue respectively. Below is the implementation of Starve-Free readers writers solution where no reader and writer is allowed to wait indefinitely for a resource.

## Starve-Free solution

### Initialisation of Global Variables 
```
in_mx = Semaphore(1)
rw_mx = Semaphore(1)
mutex = Semaphore(1)

int cnt = 0
```

### Implementation (Reader)
```
while(true) {

    //Entry Section
    wait(in_mx)
    wait(mutex)
    cnt++
    if (cnt == 1)
       wait(rw_mx);
    signal(mutex)
    signal(in_mx)

    /*
        *Critical Section
    */

   //Exit Section
   wait(mutex)
   cnt--
   if (cnt == 0)
      signal(rw_mx);
   signal(mutex)

   //Remainder Section
}
```

### Implementation (Writer)
```
while (true) {
    //Entry Section
    wait(in_mx)
    wait(rw_mx)
    
    /*
        *Critical Section
    */
    
    //Exit Section
    signal(rw_mx)
    signal(in_mx)
    
    //Remainder Section
}
```

## Correctness of Starve-free Solution

### Mutual Exclusion
- `in_mx`: ensures mutual exclusion among all processes.
- `rw_mx`: ensures mutual exclusion among all writers and first reader.
- `mutex`: ensures mutual exclusion between readers trying to access variable `cnt`. 

### Bounded Waiting
The processes must wait in a queue before `in_mx` allows a process to enter the critical section. Hence, the waiting time for a process in queue will be bounded for finite number of processes.

### Progress
There will be progress as long as execution times of all the processes' are finite, as they will continue to execute after waiting in the queue. The given solution will not enter a state of deadlock.


## More optimized Starve-Free Solution
The essential idea is that a Writer informs Readers of their need to enter the working space. Also, no new Readers can begin working at this time. Once a reader starts executing it increments variable `cnt_in`. Each Reader which leaves the working area increments variable `cnt_out` and checks to see if a Writer is waiting. When the final Reader is to leave then `cnt_in` will be equal to `cnt_out` and alerts the Writer that it is safe to move further by giving semaphore `rw_mx`.

The writers wait on `in_mx` and `out_mx`. After acquiring both semaphores, the writes compare the value of `cnt_in` and `cnt_out`. If it is equal then no readers are executing in their critical section annd writer will continue to its critical section. If it is not equal then it waits for all readers processes to complete execution and turn `wrt_wait` to true to show that a writer process is waiting in queue. Once it gets `rw_mx` it proceeds to its critical section and changes   wrt_wait` to false. After access to the working area by writer is complete, the writer notifies any waiting readers that they are now able to access the working area once more.
### Initialisation of Global Variables 
```
in_mx = Semaphore(1)
out_mx = Semaphore(1) 
rw_mx = Semaphore(0) //Semaphore to write

int cnt_in = 0 // Number of readers who have already started reading
int cnt_out = 0 // Number of readers who have completed reading

boolean wrt_wait = false // Shows if a writer is waiting
```

### Implementation (Reader)
```
while(true) {

    //Entry Section
    wait(in_mx)
    cnt_in++
    signal(in_mx)
    
    /*
        *Critical Section
    */
    
    //Exit Section
    wait(out_mx)
    cnt_out++
    if (wrt_wait == 1 && cnt_in == cnt_out)
        signal(rw_mx);
    signal(out_mx)

    //Remainder Section
}
```

### Implementation (Writer)
```
while (true) {

    //Entry Section
    wait(in_mx)
    wait(out_mx)    
    if (cnt_in == cnt_out)
        signal(out_mx);        
    else
        wait = 1;
        signal(out_mx)
        wait(rw_mx)
        wait = 0
        
    /*
        *Critical Section
    */
    
    //Exit Section
    signal(in_mx)
    
    //Remainder Section
}
```

## Correctness of Optimised Starve-free Solution

### Mutual Exclusion
- `in_mx`: ensures mutual exclusion among all processes.
- `out_mx`: ensures mutual exclusion for variables `cnt_in` and `cnt_out`. 
- `rw_mx`: ensures mutual exclusion between readers and waiting writer. 


### Bounded Waiting
The processes must wait in a queue before `in_mx` allows a process to enter the critical section. Hence, the waiting time for a process in queue will be bounded for finite number of processes.

### Progress
There will be progress as long as execution times of all the processes' are finite, as they will continue to run after waiting in the queue. The solution will not enter a state of deadlock.
