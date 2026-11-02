# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->511.86<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->2.743<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->455320.2<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->13109.5<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.506<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
2733.167, 2161.596, 2225.652, 2542.776. The fastest : 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive version algorithm is very slow due to instruction dispatch overhead and sequential memory access latency caused by explicit looping over individual elements because of the loop. Whereas the vectorized version deleguates all the arithmetic to low-level BLAS libraries to load memory in blocks and perform matrix multiplications simultaneously across parallel execution units.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The contention point is when we reach 2 threads. After that, the execution time increases for several reasons. First, the standard Colab environment only provides 2 vCPUs, so beyond 2 threads, we are introducing CPU context-switching overhead. Additionally, threads must coordinate and compete for shared memory access and L3 cache bandwidth. As we increase the number of threads, this time increases too, and beyond the contention point, it outweighs any theoretical parallel processing benefit.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
The transformer training is really suitable for a GPU because it relies only on matrix multiplications. The GPUs are exactly built in order to do thousands of multiplications in parallel. Tasks which are not suitable for this speed-up are sequential tasks where each step relies on the previous.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism stops helping when bound by software, hardware, or coordination limits. 

First, explicit Python loops create work that cannot be easily divided, causing the naive attention to take 511.86 ms compared to 2.743 ms when vectorized across hardware execution units. 

Second, thread scaling is bound by missing hardware and coordination costs: beyond 2 threads, performance degraded from 2161.596 ms (2 threads) to 2542.776 ms (8 threads) because Colab only has 2 vCPUs, forcing context switching and memory contention. 

Finally, training speed is bound by hardware architecture: the CPU spent 455320.2 ms because its few core units cannot match the massive parallel compute required for matrix operations, whereas the GPU finished in 13109.5 ms by processing thousands of tensor operations concurrently.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would compare multi-processing against multi-threading on the CPU. If multi-processing achieved significant speedups beyond 2 workers, it would prove that Python's GIL was a primary bottleneck rather than hardware vCPU limits or memory bandwidth, changing my conclusion about CPU scalability.
<!--/answer-->
