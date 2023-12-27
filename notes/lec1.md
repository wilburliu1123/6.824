### What?
What is distributed systems? 
It is network connected machines that worked together
### Why?
A group of connected machines can achieve the following where single machine cannot:
- Increase capacity by parallelism
- Tolerate faults


### Historical context
The term `Distributed system` started when local area network started at universities

## Paper
2 Programming model
This section is about how `Map` and `Reduce` function is defined and what job do they perform. Along with examples. 

3 Implementation
This section is about how this MapReduce program implemented. Mainly from defining API to the logic behind it (how worker is assigned and general workflow etc)

3.1 Exec overview

First talked about automatic partitioning of data into M map job and R reduce jobs

Defined sequence of action and showed as Fig 1
1. Split input files into M pieces from 16-64MB. Then starts up many copies of the program on cluster of machines
2. One copy is the master who assign work to workers. Master picks idle workers and assigns each map task and reduce task to them
3. Worker read corresponding input split, parses k-v pairs, and pass it to map function. Map function will buffer output to current machine's memory
4. Buffered pairs will be periodically written to local disk, partitioned into R regions and locations are passed back to the master
5. Reduce worker is notified by the master about those locations. It uses RPC to read the buffered data from local disks of map workers. After it finish reading, it sorts it by the intermediate keys so all occurrence of the same key are grouped. This sorting is needed because amount of intermediate data is too large to fit in memory 
6. Reduce worker iterates over the sorted data and for each unique key, it pass the key and value to user's reduce function. The output is appended to the final output file for this reduce partition
7. When all map tasks and reduce tasks are done, master wakes up user program and `MapReduce` call in the user program returns back to the user code. 
![[map_reduce_fig1.png]]
#### 3.2 Master Data structure
For each map and reduce task, master store its state (*idle*, *in progress* or *completed*) and identity of worker machines (non idle tasks, *in progress* or *completed* workers)

For each *completed* map tasks, master stores the location of R files and push those incrementally to workers that have *in-progress* reduce tasks

#### 3.3 Fault tolerance
*Worker failure*
Master ping every worker periodically. If heartbeat is not responeded, master mark those map tasks back to *idle* state. Same for *in progress* tasks

When map task is switched from A to B, all workers executing reduce tasks are notified. Any reduce task not already read the data from worker A will switch to worker B
但这里如果已经读A的数据的reduce task 是否要重新开始？

*Master failure*
This paper assumes single master rarely fail and if master fails, Client can check for this condition and retry the MapReduce job if they desire

*Semantics in the presence of failures*
It rely on atomic commit to achieve fault tolerance

Question
> In the presence of non-deterministic operators, the output of a particular reduce task R1 is equivalent to the output for R1 produced by a sequential execution of the non-deterministic program.

what does this mean? 
#### Locality
Locality is another optimization to save network trip on MR
#### Backup tasks
Long tail problem where some tasks are just running longer (stragglers). MR uses backup (replicated tasks) tasks to speed up the process (if either stragglers or its backup finishes first, the MR job termintes)

### Refinements
Talked about additional extension on top of existing system to make it better.
#### Partitioning function 
Default and allow customization of partition function based on input. 
#### Combiner function 
Basically partial merge before send to reducer (saving network resources)
*Reduce vs Combiner*
Reduce job has final output where Combiner has intermediate output 
#### Side effects
MR assume application writer manages side effect of their MR job
No support for 2PC
#### Skipping bad record
Some record are determinitically failed (bad key or violate programming logic) then MR will skip those records by letting master to receive "last gasp" UDP packet more than once

#### Status information
Master implemented an HTTP server which allow user to check the status of the job (current progress and status of workers etc)

### Performance
2 results are shown, 1 for 1TB grep, 1 for 1TB sort job. Basically represented the results of given performance of those 2 jobs running on MR. Noted that sort job already know distribution of the keys. So in practice it might need additional sampling to avoid hot key problem 



What is the purpose of MapReduce?


Google need to build reverse index for web url in order to let user to search the web from their search engine. This kind of computation takes hours to complete and MapReduce is build for that kind of computation 

The goal for MapReduce is to make non expert easy to build fault tolerance system


How does MapReduce approach the problem to make non expert easy to build a fault tolerant system?

Provide `map` + `reduce` function (functional style which is stateless) for user to implement

MR deals with distribution (copy binary to worker machine, partition the data set, fault tolerance. basically solve common problem and abstract it away from user)


What are the main topics of distributed systems that often get addressed?

4 of them
1. Fault tolerance
	1. Availability (how to achieve it? techniques needed)
	2. Recoverbility (through relication, log/transaction to recover state from disk)
2. Consistency 
3. Performance
	1. Throughout
	2. Latency
4. Implmentation 

<iframe width="560" height="315" src="https://www.youtube.com/embed/WtZ7pcRSkOA?si=lsNsQC1-D1qHGrXL&amp;clip=UgkxVo5djVKS1V_cVWNWZEKpt0ze2KxI2Cze&amp;clipt=EISJkgEY5N2VAQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



### Performance

#### Sort
> Our partitioning function for this benchmark has built-in knowledge of the distribution of keys 

This paper already know the distribution of the key. Need sampling in real world data

