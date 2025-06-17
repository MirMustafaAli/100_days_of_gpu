
# AMD CDNA 3 Archtecture WhitePaper
* The Instinct MI300X discrete GPU focuses
primarily on accelerator compute and incorporates 8 accelerator complex dies (XCD)
*  peak theoretical FP8 performance of 2.6
PFLOP/s.M
* The first set of chiplets is the accelerator complex dies (XCDs) which contain the computational elements of
the processor along with the lowest levels of the cache hierarchy
* each XCD contains a shared set of global resources such as the scheduler,
hardware queues, and four Asynchronous Compute Engines (ACE) that send compute shader workgroups
to the Compute Units (CUs) that are the computational heart of the AMD CDNA™ 3 architecture.
*  The 38 CUs all share a 4MB L2 cache that serves to coalesce all the memory
traffic for the die
* the processor includes 6-8
XCDs for as many as 304 CUs total, roughly 40% more than the Instinct MI250X GPU.



### XCD
* The instruction cache is shared between two CUs and doubles the capacity from the prior generation to
a 64KB and 8-way set-associative data array.
*  TF32 is somewhat of a
misnomer and it is actually a hybrid of FP16 and BF16. As Figure 5 below shows, the data format is 19-bits and
combines the more precise 10-bit mantissa of FP16 with the wider range 8-bit exponent from BF16 (along
with a sign bit). The name of the TF32 data type comes from the fact that for nearly all machine learning,
* The peak theoretical computational throughput for FP8 on the
Instinct MI300X and MI325X discrete GPUs are 16X the FP32 peak performance 
* The dense representation of the data fits directly into the Matrix core pipeline and enables doubling the computational
throughput up to an incredible 8K operations per clock for a CU.
* The LDS in the AMD CDNA 3 compute units remain at 64KB similar to AMD CDNA 2 compute units. The L1 vector data cache is responsible for feeding into the vector register file and LDS and keeping the execution units fully utilized.
* The cache line size has doubled to 128B, which also doubles L1 data cache capacity in tandem to 32KB,
* One thing that remains unchanged is that the vector data cache has a very relaxed coherency model that requires explicit synchronization to offer strong coherency and ordering guarantees



### Cache and it's coherence play
* redesign of the memory hierarchy truly starts with the shared L2 cache within the XCDs
* The role of the L2 cache has fundamentally changed with the addition of the
AMD Infinity Cache™ technology, which is a last level cache (LLC) located on the active I/O die (IOD)
* the L2 plays a critical new role since it is the lowest level of cache where coherency is automatically maintained by the hardware. 
* The L2 is a 4MB and 16-way set associative cache that is massively parallel with 16 channels that are each 256KB
* The L2 cache is shared by all 38 Compute Units and services requests from both the lower level instruction and data caches
* On the read side each channel can read out a 128-byte cache line and the L2
cache can sustain four requests from different CUs per cycle for a combined throughput of 2KBytes/clock for
each XCD
* The 16 channels only support a half-line 64-byte write each with one fill request from the Infinity
Fabric per clock.
*  AMD CDNA 3 architecture has collectively up to eight instances and up to
34.4 TB/s aggregate read bandwidth.
*  The L2 is a <b>writeback and write-allocate design</b>
* The L2 itself is coherent within an XCD.
* The Infinity Cache includes a snoop filter covering the multiple XCD L2 caches so that the vast majority of coherent requests from other other XCDs will be resolved at the Infinity Cache without disturbing


### CDNA Architecture Memory
*  vertically stacked beneath a pair of XCDs
* The L2 acts as a single point of interface for each XCD, coalescing all the local memory traffic to and from the 38 CUs before it spills out to the IOD.
* The concept of a channel originates in the L2 (each L2 comprising 16 channels), but is critical all throughout the rest of the memory hierarchy in the IOD and beyond.
* Each L2 is connected to the IOD across the set of sixteen channels and each channel is 64-bytes wide, for a total of 1K-bytes per XCD at the IOD interface.


### AMD Infinity Cache
* The AMD Infinity Cache was carefully designed as a shared memory-side cache, meaning that it caches the contents of memory and cannot hold dirty data evicted from a lower level cache.

<b>Advantages</b>
- First, the AMD Infinity Cache doesn’t participate in coherency and does not have to absorb or handle any snoop traffic, which significantly improves efficiency and reduces the latency of snooping from lower level caches.
- Second, it can actually hold nominally uncacheable memory such as buffers for I/O.

* the AMD Infinity Cache is 16-way set-associative, and it is built around the concept of
channels.
* Each stack of HBM memory is associated with 16 parallel channels.
* A channel is 64-bytes wide and connects to 2 MB of data arrays that are banked to sustain simultaneous reads and writes.
* In total, there are eight stacks of HBM across the four IODs, for 128 channels or 256MB of data.

* each IOD fans out through the package to two stacks of memory.
* For the Instinct MI300X and MI300A processors, the memory controllers drive a bus that operates at 5.2 Gbps and each stack contains 24GB on Instinct MI300X GPUs and 16GB on Instinct MI300A of memory.
* the HBM3 memory is 192GB on Instinct MI300X GPUs have an astounding 5.3 TB/s peak theoretical memory bandwidth.


### Partitioning
* The AMD Instinct MI300X discrete GPU can be configured with up to eight partitions, one per XCD.
* MD Instinct MI300 Series family of GPUs supports Single Root IO Virtualization (SR-IOV) that provides isolation of Virtual Functions (VF’s) and protect a VF from accessing information or state of the Physical Function (PF) or another VF.
* In addition to GPU partitons, the HBM can also be partitioned on the Instinct MI300X discrete GPU. 
* Known as NUMA partitions per socket or NPS, the HBM can be one or four partitions and is a separately configured from the GPU partitioning.
* The memory partitioning must be equal to or smaller than the number of GPU partitions. Example: NPS4 memory partitioning can be combined with four or eight GPU partitions.



### Communication and Scaling
* For AMD CDNA 3 architecture, the communication links have been systematically enhanced to operate at up to 32Gbps and redistributed across the IODs.
* Each IOD includes two 16-lane, bi-directional inter-package AMD Infinity Fabric links for connecting to other AMD accelerators.
* One of the links is multi-purpose and can be configured to act as a x16 PCIe® Gen 5 for pure I/O functionality.




Absolutely — here’s the entire AMD GPU Architecture Q&A in clean Markdown format, ready to paste into any .md file, documentation, or study notes:

⸻

🚀 AMD GPU Architecture — In-Depth Q&A

⸻

🎛️ 1️⃣ Compute Units (CUs) and Wavefront Execution

Q1: What is a Compute Unit (CU) in AMD GPUs?
A1: A CU is the fundamental hardware block in AMD GPUs. It contains SIMD lanes, scalar units, registers (VGPRs & SGPRs), Local Data Share (LDS), instruction and scalar caches, and schedulers.

⸻

Q2: What is a wavefront in AMD GPUs?
A2: A wavefront is a group of threads (usually 64) executing the same instruction simultaneously in SIMD fashion, similar to a warp in NVIDIA GPUs.

⸻

Q3: How does AMD handle divergence in wavefronts?
A3: If threads in a wavefront take different paths, the hardware uses a per-thread mask to disable inactive threads for each branch path, executing paths serially.

⸻

Q4: What are Scalar and Vector Units?
A4: Scalar units handle instructions common to all threads in a wavefront (like loop counters); Vector units execute SIMD instructions for each thread in parallel.

⸻

Q5: What are Special Function Units (SFUs)?
A5: SFUs handle complex math like sine, cosine, reciprocal, etc., offloading these from regular ALUs to save power and cycles.

⸻

💾 2️⃣ Memory Hierarchy

Q6: Describe the memory hierarchy in AMD GPUs.
A6:
	•	Global Memory: HBM or GDDR, large capacity, high latency.
	•	L2 Cache: Shared across all CUs.
	•	L1 Cache: Per-CU data cache; feeds vector units.
	•	Instruction Cache: Per-CU, for fetching instructions.
	•	Scalar Cache: Stores scalar values common to all threads.
	•	Local Data Share (LDS): On-chip shared memory for fast inter-thread data exchange within a workgroup.
	•	Register File: VGPRs for vector data; SGPRs for scalar data.

⸻

Q7: What is LDS and why is it important?
A7: LDS is AMD’s equivalent of CUDA’s shared memory. It’s fast, user-managed storage on-chip for data reuse, crucial for blocking, tiling, and avoiding global memory reads.

⸻

Q8: How does cache coherence work in AMD GPUs?
A8: AMD uses a relaxed coherency model for performance. Coherence is strong within each CU but not guaranteed globally; explicit barriers (memory fences) are needed for ordering between wavefronts or CUs.

⸻

Q9: What is the cache line size on AMD GPUs?
A9: Typically 64 bytes for older architectures, up to 128 bytes in CDNA 3 for high throughput.

⸻

Q10: How does memory coalescing work in AMD GPUs?
A10: Memory accesses by threads are combined into as few cache lines as possible. Coalesced loads reduce bandwidth waste; scattered accesses cause more transactions.

⸻

⏱️ 3️⃣ Scheduling and Occupancy

Q11: How does the CU schedule wavefronts?
A11: Each CU has multiple wavefront slots; the scheduler switches wavefronts in and out to hide latency from memory or SFU ops.

⸻

Q12: What limits occupancy in an AMD CU?
A12: Number of VGPRs per thread, total LDS used by the workgroup, and the hardware limit on concurrent wavefronts per CU (often 40–64 wavefronts).

⸻

Q13: How does AMD handle register spilling?
A13: If a kernel uses more VGPRs than available, excess data spills to global memory, drastically hurting performance.

⸻

Q14: How is latency hidden on AMD GPUs?
A14: By having multiple wavefronts ready to run; if one wavefront stalls, the scheduler switches to another.

⸻

🧩 4️⃣ Synchronization and Communication

Q15: How do threads in a workgroup synchronize?
A15: Using barrier() in HIP or OpenCL; it ensures all threads reach the barrier before continuing.

⸻

Q16: How do wavefront-level shuffles work?
A16: AMD has Data Share (DS) instructions for shuffle-like operations within a wavefront for fast intra-wavefront communication without LDS.

⸻

Q17: How is inter-CU synchronization handled?
A17: Using atomics in global memory, fences, and streams. No direct hardware barrier exists between CUs.

⸻

🏎️ 5️⃣ Performance and Optimization

Q18: What is the purpose of coarsening and loop unrolling?
A18: Coarsening: each thread does more work to amortize overhead. Loop unrolling reduces branch instructions and increases ILP (instruction-level parallelism).

⸻

Q19: What are non-temporal stores?
A19: Stores that bypass caches to avoid polluting them when data won’t be reused, improving cache efficiency for large writes.

⸻

Q20: How can LDS bank conflicts be avoided?
A20: By aligning data so consecutive threads access different banks, or padding arrays to avoid multiple threads hitting the same bank simultaneously.

⸻

Q21: What is the role of occupancy calculators?
A21: They predict how many wavefronts fit per CU given register and LDS usage, guiding kernel tuning for maximum parallelism.

⸻

🔌 6️⃣ Interconnect and Multi-GPU

Q22: How do AMD GPUs communicate in multi-GPU setups?
A22: Using PCIe or Infinity Fabric. Infinity Fabric enables high-bandwidth, low-latency peer-to-peer access between GPUs and to the CPU in CDNA/Instinct architectures.

⸻

Q23: What is Unified Memory in AMD ROCm?
A23: A memory model where the CPU and GPU share a single address space, with on-demand paging and migration, simplifying programming but adding overhead compared to explicit transfers.

⸻

⚙️ 7️⃣ Compiler and ISA

Q24: What is GCN vs RDNA vs CDNA?
A24:
	•	GCN (Graphics Core Next): Older compute architecture (Vega, etc.).
	•	RDNA: Designed for gaming GPUs, better graphics pipeline.
	•	CDNA: Pure compute, high FP64, Matrix MFMA instructions for AI/HPC workloads.

⸻

Q25: How do wavefront predication and vector predication work in GCN/CDNA?
A25: The hardware uses EXEC mask registers; instructions execute on active lanes based on predicates.

⸻

Q26: What is MFMA?
A26: Matrix Fused Multiply-Add — specialized instructions for high-throughput matrix operations, similar to NVIDIA’s Tensor Cores.

⸻

🧪 8️⃣ Profiling and Debugging

Q27: What tools does AMD offer for profiling?
A27:
	•	rocprof — low-level profiler for kernel timing, instruction mix, and occupancy.
	•	rocm-smi — for power and temperature monitoring.
	•	Radeon Compute Profiler (RCP) and CodeXL (legacy).
	•	Nsight Systems supports some ROCm kernels with limited support.

⸻

Q28: How do you check cache hit rates?
A28: Using rocprof counters: L1/L2 hit/miss ratios can be measured to diagnose global memory efficiency.

⸻

Q29: What are common performance pitfalls?
A29:
	•	Divergence within wavefronts
	•	LDS bank conflicts
	•	Uncoalesced global memory accesses
	•	Register spilling
	•	Too few wavefronts to hide latency.

⸻

✅ 9️⃣ Advanced Workloads

Q30: How does FP8 E4M3 format help?
A30: Reduces memory footprint and bandwidth while keeping enough dynamic range for deep learning workloads; hardware matrix units (MFMA) can accelerate FP8 ops.

⸻

Q31: How are multi-level tiling and prefetching used?
A31: For large GEMM: tiles are loaded from global → LDS → registers, with overlapping computation and data loading to hide latency.




# 📘 AMD GPU Architecture — Full Interview Q&A

## ✅ Beginner Level

### 1️⃣ What is the AMD CDNA architecture designed for?
**Answer:**  
CDNA is designed for high-performance, data-parallel computing workloads like scientific simulations, AI training, and large matrix operations. It focuses on maximizing throughput for data-intensive tasks.

### 2️⃣ What is a Compute Unit (CU) in AMD GPUs?
**Answer:**  
A Compute Unit is the basic processing block containing multiple SIMD pipelines, scalar and vector ALUs, local data share (LDS), and caches. It executes wavefronts in parallel.

### 3️⃣ What is a wavefront?
**Answer:**  
A wavefront is a group of 64 work-items (threads) that execute the same instruction in lockstep on SIMD units.

### 4️⃣ What is the difference between VGPR and SGPR?
**Answer:**  
- **VGPR (Vector GPR):** Holds data for each work-item in a wavefront; used by vector ALU.  
- **SGPR (Scalar GPR):** Holds data common across a wavefront; used by scalar ALU.

### 5️⃣ What is LDS in AMD GPUs?
**Answer:**  
Local Data Share (LDS) is a 64 KB scratchpad memory in each compute unit. It enables fast data sharing and synchronization among work-items in a workgroup.

### 6️⃣ What is the EXEC mask?
**Answer:**  
The EXEC mask is a 64-bit mask that controls which threads in a wavefront are active and execute a given instruction.

### 7️⃣ What does the Scalar ALU (SALU) do?
**Answer:**  
SALU performs operations that affect all work-items uniformly, such as controlling branches and calculating addresses.

### 8️⃣ What is the function of the command processor in CDNA GPUs?
**Answer:**  
The command processor manages commands sent from the host CPU, configures registers, controls execution, and handles interrupts.

### 9️⃣ What is the role of the Global Wave Sync (GWS)?
**Answer:**  
GWS synchronizes multiple workgroups across the entire device for global coordination.

### 🔟 What are Matrix Core operations in CDNA3?
**Answer:**  
These are hardware-accelerated matrix multiply-accumulate instructions (MFMA) for efficient large matrix computations, crucial for AI workloads.

---

## ✅ Intermediate Level

### 1️⃣ What happens if an SGPR or VGPR index is out-of-range?
**Answer:**  
- For SGPRs: If a source is out-of-range, SGPR0 is used; if a destination is out-of-range, no write occurs.
- For VGPRs: Same; source uses VGPR0; out-of-range destination means the instruction is a NOP.

### 2️⃣ Explain the trap and exception handling in CDNA3.
**Answer:**  
Trap registers store fault information (trap PC, trap ID) when exceptions occur (like illegal instructions, memory violations). The TRAPSTS and TMA registers help manage these events, allowing wavefront context save and recovery.

### 3️⃣ How does the GPR indexing work?
**Answer:**  
An M0 register holds an index offset that modifies base addresses for source or destination VGPRs dynamically during execution. It allows indirect addressing within VGPR files.

### 4️⃣ What does the program counter (PC) do in a wavefront?
**Answer:**  
The PC points to the next instruction for the wavefront. Instructions like S_GET_PC, S_SET_PC, and branches update the PC for control flow.

### 5️⃣ How are vector memory operations synchronized with execution?
**Answer:**  
Counters like VM_CNT, LGKM_CNT, and EXP_CNT track outstanding operations. `S_WAITCNT` ensures operations complete before dependent instructions proceed.

### 6️⃣ What are the main types of ALU instructions in CDNA3?
**Answer:**  
- **SALU:** Scalar ALU ops for control flow, address calculations.
- **VALU:** Vector ALU ops for per-thread arithmetic.
- **MFMA:** Matrix fused-multiply-add for AI workloads.

### 7️⃣ Explain the concept of MFMA blocks and registers.
**Answer:**  
MFMA uses Accumulation VGPRs separate from normal VGPRs. Registers must be aligned and contiguous for matrix blocks, following a strict storage layout for input/output tiles.

### 8️⃣ How does the EXEC mask interact with branching?
**Answer:**  
Conditional vector instructions depend on EXEC: only active threads execute. Scalar branches can modify EXEC by setting it based on vector condition codes.

### 9️⃣ What is the role of the trap handler’s TTMP registers?
**Answer:**  
TTMP0-TTMP15 are reserved SGPRs used during trap handling for storing temporary state, PC, and trap-specific data.

### 🔟 What is the impact of denormals and rounding modes in CDNA3 matrix instructions?
**Answer:**  
Some MFMA instructions ignore MODE register settings for denormals and rounding (e.g., XF32). Others like FP64 respect IEEE compliance. This ensures predictable hardware behavior for precision-critical workloads.

---

## ✅ Advanced Level

### 1️⃣ How does the CDNA3 instruction scheduler optimize latency hiding?
**Answer:**  
By issuing instructions out-of-order within wavefronts, scheduling independent instructions first, and using register scoreboarding to prevent hazards. Multiple ready wavefronts ensure the SIMD lanes remain busy.

### 2️⃣ Explain the difference between DPP and SDWA modifiers in vector instructions.
**Answer:**  
- **DPP (Data Parallel Primitive):** Reorders, rotates, or shifts vector lanes for intra-wavefront communication without LDS.
- **SDWA (Sub-DWord Addressing):** Allows operations on subfields (byte, half-word) within VGPRs, useful for packing/unpacking data.

### 3️⃣ How are multiple workgroups globally synchronized in CDNA3?
**Answer:**  
Via Global Wave Sync (GWS) hardware primitives and atomic memory operations on global memory, combined with explicit synchronization barriers in kernels.

### 4️⃣ What is the role of scalar vs vector memory instructions?
**Answer:**  
Scalar memory ops fetch values identical for all threads, saving bandwidth. Vector memory ops fetch per-thread data in parallel. Scalar caching optimizes repeated fetches.

### 5️⃣ How does the CDNA3 MFMA instruction pipeline differ from older architectures?
**Answer:**  
It supports higher operand precision (FP64), larger operand tiles, improved accumulation registers, and fine-grained control of output layouts, with optimized scheduling for deep learning kernels.

### 6️⃣ How is register spilling minimized in high-performance kernels?
**Answer:**  
By carefully blocking kernels to fit within VGPR limits, loop unrolling to reuse registers, and minimizing dynamic indexing which may force spills.

### 7️⃣ Describe how wavefront context save and restore works during traps.
**Answer:**  
Trap handlers save EXEC, PC, VGPR state into trap-specific memory, service the exception, and restore context to resume interrupted computation transparently.

### 8️⃣ How do denormals propagate in mixed-precision operations?
**Answer:**  
Denormals may flush to zero in FP16/FP32 for performance but are preserved in FP64. MFMA units often flush subnormal inputs to zero to reduce hardware cost.

### 9️⃣ How is the global data cache hierarchy tuned for matrix-heavy workloads?
**Answer:**  
L2 is large and shared across compute units for high reuse; scalar and vector caches minimize redundant loads. Matrix operations exploit high bandwidth and cache locality for tile re-use.

### 🔟 What are the practical impacts of relaxed cache coherency in CDNA3?
**Answer:**  
Programmers must use explicit fences or barriers when synchronizing data between wavefronts and workgroups to ensure correctness, avoiding stale data or race conditions.



## Cache Coherence, Consistency & AMD Instruction Set — Q&A

---

## 📌 1️⃣ Beginner Questions & Answers

| # | Question | Answer |
|---|-------------------------------|-------------------------------|
| 1 | What is cache coherence in multiprocessor systems? | It ensures that all processors observe the same value for the same memory location in their caches. |
| 2 | Why do we need cache coherence? | To avoid stale or inconsistent data when multiple cores access and modify shared variables. |
| 3 | What is a memory consistency model? | A contract that defines the legal orderings of memory operations visible to processors. |
| 4 | What is the difference between coherence and consistency? | Coherence: agreement on a single location's value. Consistency: ordering of reads/writes across multiple locations. |
| 5 | What is write propagation? | Mechanism to ensure that a write by one processor is visible to others by invalidating or updating other caches. |
| 6 | Can you name a basic coherence protocol? | MSI or MESI are classic examples. |
| 7 | What is false sharing? | When multiple processors write to different variables that share the same cache line, causing unnecessary coherence traffic. |
| 8 | What happens if there is no cache coherence mechanism? | Processors may read stale values, leading to incorrect program execution. |

---

## 📌 2️⃣ Intermediate Questions & Answers

| # | Question | Answer |
|---|-------------------------------|-------------------------------|
| 1 | Explain the MESI protocol in detail. | MESI: Modified, Exclusive, Shared, Invalid. Tracks if a cache line is modified, exclusive to one cache, shared among caches, or invalid. |
| 2 | What is MOESI protocol? | Extends MESI by adding Owned: a state where a block is dirty but can be shared, reducing unnecessary write-backs. |
| 3 | Compare snooping vs. directory-based coherence. | Snooping: all caches monitor a broadcast bus. Directory: a central table tracks which caches have copies to direct coherence actions, reducing broadcast traffic. |
| 4 | What are transient states? | Temporary states during protocol transitions, e.g., waiting for ownership before writing. |
| 5 | Explain sequential consistency with an example. | Every processor sees all memory operations in the same order, consistent with each program’s order. |
| 6 | What is weak consistency? | Only guarantees consistency at synchronization points; allows more reordering for better performance. |
| 7 | Difference: write-invalidate vs. write-update. | Invalidate: writer invalidates other copies. Update: writer broadcasts new data to other caches. |
| 8 | Snooping hardware mechanisms? | Bus snoopers, bus arbiters, cache controllers watching bus traffic. |
| 9 | Role of a coherence directory? | Tracks which caches have copies; directs invalidations/updates without broadcasting to all nodes. |

---

## 📌 3️⃣ Advanced Questions & Answers

| # | Question | Answer |
|---|-------------------------------|-------------------------------|
| 1 | How do multi-socket NUMA systems handle coherence? | Use extended directory protocols or coherence domains to manage remote cache interactions efficiently. |
| 2 | How does directory-based coherence reduce bus bandwidth? | Targets only relevant sharers rather than broadcasting to all processors. |
| 3 | How do GPU memory models differ from CPUs? | GPUs use weaker consistency to maximize throughput; explicit synchronization is needed for correctness. |
| 4 | Trade-offs between strict and relaxed consistency? | Strict: easier to reason but slower. Relaxed: higher performance, more programmer responsibility. |
| 5 | Difference: release vs. processor consistency. | Release: consistency at lock acquire/release. Processor: writes seen in issue order by other processors but reads/writes can reorder. |
| 6 | Handling coherence races? | Use atomic transactions, locks, or unique request IDs to serialize conflicting operations. |
| 7 | Optimizations: silent stores, migratory sharing? | Avoid sending coherence messages for redundant writes (silent stores); efficiently transfer ownership if only one processor frequently accesses a block (migratory sharing). |
| 8 | Impact on compiler/memory ordering? | Compilers insert memory barriers/fences to preserve required ordering. |
| 9 | Designing coherence for heterogeneous CPU+GPU? | Separate coherence domains or explicit programmer-managed coherence; hybrid solutions for unified memory. |

---

## 📌 4️⃣ Coherence & Consistency Protocols

| Type | Examples | Notes |
|----------------|-------------------------------|-------------------------------|
| Coherence | MSI, MESI, MOESI, MESIF | Classic CPU protocols. |
| Directory-Based | Full map, limited pointer | Used in large multi-socket systems. |
| Consistency | Strict, Sequential, Processor, Weak, Release | Defines memory operation ordering guarantees. |
| Real-world | TSO (x86), PSO (SPARC), RC (Release Consistency) | Actual hardware implementations. |
| GPU | Often weak or relaxed; explicit sync primitives. |

---

## 📌 5️⃣ AMD Instruction Set / GPU Architecture — Q&A

| # | Question | Answer |
|---|-------------------------------|-------------------------------|
| 1 | What are VALU and SALU in AMD GPUs? | VALU: Vector ALUs for SIMD operations. SALU: Scalar ALUs for scalar instructions. |
| 2 | What is a wavefront in AMD architecture? | A group of threads (usually 64) executing in lockstep within a SIMD unit. |
| 3 | How is LDS used in AMD compute units? | Local Data Share is on-chip shared memory for fast inter-thread communication within a wavefront/block. |
| 4 | Purpose of L1 vector data cache? | Feeds vector register files and LDS efficiently to keep SIMD units busy. |
| 5 | How does AMD handle vector/scalar cache coherency? | Scalar caches are coherent globally; vector caches use relaxed coherency with explicit synchronization if needed. |
| 6 | Typical cache line size in AMD CDNA GPUs? | Usually 128 bytes (e.g., CDNA 2 & 3). |
| 7 | Impact of relaxed coherency model? | Programmers must use barriers or fences to ensure correct data ordering when needed. |
| 8 | What are SFUs? | Special Function Units for transcendental math operations (sin, cos, exp). |
| 9 | How does AMD support mixed-precision ops? | Supports FP32, FP16, BF16, and FP8 with specialized vector units. |
| 10 | VGPRs vs. SGPRs? | VGPRs: Vector registers for SIMD lanes. SGPRs: Scalar registers shared by a wavefront. |
| 11 | How to optimize kernel occupancy on MI300X? | Balance register usage, block size, LDS usage; avoid register spilling. |
| 12 | Wavefront-level sync? | Use `s_barrier` or `ds_barrier` instructions. |
| 13 | Profiling cache hits/misses? | Use `rocprof`, `rocminfo`, `rocm-smi` for hardware counters and events. |
| 14 | How does the vector cache feed LDS and registers? | By staging data close to the SIMD cores, reducing latency for repeated data reuse. |
| 15 | Non-temporal writes trade-off? | Bypass caches for streaming writes to avoid polluting caches with write-only data. |

---

