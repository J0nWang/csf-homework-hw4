CONTRIBUTIONS
Jonathan Wang
-completed the whole assignment.

REPORT
Dataset: 16 MB random 64-bit integers
System: multi-core Linux host ugrad12.cs.jhu.edu
Program: parsort.c using shared memory mapping and fork-based parallel recursion

Data table, elements vs real time elapsed (full data at the bottom of the report):
| Parallel Threshold (elements) | Test 1 | Test 2 | Test 3 | Average (s) |
| ----------------------------- | ------ | ------ | ------ | ----------- |
|             2 097 152 (≈ 2 M) |  0.394 |  0.377 |  0.388 |    0.386    |
|             1 048 576 (≈ 1 M) |  0.226 |  0.213 |  0.213 |    0.217    |
|                       524 288 |  0.159 |  0.155 |  0.157 |    0.157    |
|                       262 144 |  0.126 |  0.131 |  0.126 |    0.128    |
|                       131 072 |  0.119 |  0.109 |  0.111 |    0.113    |
|                        65 536 |  0.116 |  0.110 |  0.113 |    0.113    |
|                        32 768 |  0.128 |  0.114 |  0.118 |    0.120    |
|                        16 384 |  0.132 |  0.124 |  0.124 |    0.127    |

Observations:
Large thresholds performed mostly sequential executions. With parThreshold at around 2 
million, the whole 16MB array fits into one recursion branch before even hitting the 
threshold. This means that the sort doesn't spawn any or very few child processes, so 
the sort behaves more or less like a single-process quicksort. This is reflected it's 
runtime, being the highest at roughly 0.39.

At moderate thresholds, parrellism can be observed. Between 128 thousand and 1 million 
elements, each partition spawns a few child processes whose memory slices point to 
different non overlapping slices of the same map_shared region. So, those sub processes 
could execute concurrently on different CPU cores. The kernel's scheduler interleaves 
them, resulting in large reductions in time (down to the 0.11 to 0.13 range).

However, at very small thresholds, we get diminishing returns. When the threshold falls 
below 64 thousand elements, the system now spends more time performing fork() and 
waitpid() calls, managing page tables, and switching between short lived processes that 
the overhead overtakes the optimizations from parallelism, and the elasped time 
stabalizes and even slightly rises (notice 0.113->0.120->.127 in the table).

test 1 data:
real    0m0.394s
user    0m0.376s
sys     0m0.011s

real    0m0.226s
user    0m0.395s
sys     0m0.019s

real    0m0.159s
user    0m0.429s
sys     0m0.041s

real    0m0.126s
user    0m0.444s
sys     0m0.052s

real    0m0.119s
user    0m0.458s
sys     0m0.056s

real    0m0.116s
user    0m0.473s
sys     0m0.083s

real    0m0.128s
user    0m0.498s
sys     0m0.106s

real    0m0.132s
user    0m0.531s
sys     0m0.159s

______
test 2 data:

real    0m0.377s
user    0m0.362s
sys     0m0.008s

real    0m0.213s
user    0m0.369s
sys     0m0.024s

real    0m0.155s
user    0m0.399s
sys     0m0.049s

real    0m0.131s
user    0m0.442s
sys     0m0.054s

real    0m0.109s
user    0m0.430s
sys     0m0.064s

real    0m0.110s
user    0m0.478s
sys     0m0.070s

real    0m0.114s
user    0m0.490s
sys     0m0.110s

real    0m0.124s
user    0m0.532s
sys     0m0.153s
______
test 3 data:
real    0m0.388s
user    0m0.370s
sys     0m0.012s

real    0m0.213s
user    0m0.374s
sys     0m0.020s

real    0m0.157s
user    0m0.415s
sys     0m0.044s

real    0m0.126s
user    0m0.432s
sys     0m0.053s

real    0m0.111s
user    0m0.435s
sys     0m0.063s

real    0m0.113s
user    0m0.467s
sys     0m0.070s

real    0m0.118s
user    0m0.485s
sys     0m0.117s

real    0m0.124s
user    0m0.522s
sys     0m0.157s