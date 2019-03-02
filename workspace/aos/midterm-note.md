
## spin (strongly typed interface, dynamically bind implementation)

### pros
 - protection domain enforced by strongly typed language. no cheating
 * entire OS in the same hardware address space. have shared address space to prevent border crossing
 * (key) applications can dynamically bind to different implementations of the interface functions providing flexibility

### cons
 - driver that need hardware access need to step out the language protection
 - difficult to force vendor to use the same specific language


## exokernel (allow arbitrary code downloaded to kernel, PE data structure store entriy point of handler, one PE structure per lib OS)
### compared to SPIN
 - exokernel exposes the hardware to the lib OS thru a secure binding which allows the lib OS to download arbitrary code into kernel and execute it
 * SPIN extend the logical protection domain which follows modula 3, strongly typed language. it enforces compile time checking and run time verification
 - exokernel, only selected code fragments are downloaded to kernel by lib OS thru security binding
 * SPIN, the entire OS is in same hardware address space

### page fault handling in exokernel (mention PE data structure for entry point of handler)
 - exokernel detects the pf
 - exokernel uses PE data structure for the library OS to make upcall to appropriate entry point in lib OS that handle pf
 - lib OS runs page relacement algo if needed
 - lib OS calls exokernel to delte the victim page
 - lib OS issue IO via exokernel to get faulting page from disk to PFN
 - upon io completion, exokernel upcalls the lib OS via entry point in PE data structure
 - lib OS calls exokernel to install translation in TLB
 - application to resume

# MS-DOS
 - no protection between application and os: everything is together. and errant application can corrupt os
 - good performance: system call is like procedure call
 - good extensibility: easy to add new service or replace existing service
 - MS-DOS is designed to run ONE application at a time. so protection is deemed unnecessary
 
# OS basics

different types of discontinuities
 - trap, syscall
 - fault, page fault, page not in memory
 - exception, divided by 0
 - interrupt, io completion/request
 

# virtualization

pysical page vs. machine page
 - pp: illusion of continuous phyiscal memory from gust OS's view
 - mp: it's the view of physical memory from hypervisor's view. the pp mapped to mp in shadow table

size of 1 shadow pt is proportional to the # of running processes in the guest os
number of shadow tables is the same as the number of guest os running

## para virtualization
How para virtualized performs better in IO 
 - hypervisor provide guest os clean and simple device abstraction
 - guest os can effiently transfer data without overhead of multiple copying thru APIs
 - hypercalls make control transfers efficient 
 - guest have some control in event delivery thru hypercall

### Xen
 - xenolinux runs on top of Xen
 - it installs a fast handler in Xen so that system calls by a process executing on top of XenoLinux gets directly handled by XenoLinux without a level of indirection from Xen
 - Xenolinux uses an I/O ring to convey disk I/O requests to Xen, and gets response back from Xen via I/O ring
 
 - via fast handler, xenolinux fields the fopen system call from Xen
 - Xenolinux populates **the I/O ring data structure with the details** of the fopen call
 - Xenolinux makes the I/O request by using a **hypercall**
 - **Xen** service the request
 - Xen fills the response into the **I/O ring data structure**
 - Xenolinux gets the response via **polling the I/O ring**
 - Xenolinux completes what it meeds to do and places the process back on the ready queue

### CMM - coordinated memory management
 1. Each VM has entry point called "report memory status"
 2. Each VM installs balloon driver
 3. Hypervisor calls "report memory status" on each donor vm
 4. each donor vm reports "Actual memory in use"
 5. hypervisor updates "memory-infor" for each donor vm
 6. calculate how much memory to ask from each donor vm based on the "memory-info"
 7. hypervisor instructs the balloon driver in each of the donor vm based to inflate commensurate with the decision in step 6
 8. once all drivers in donor vms have completed the inflate command, the hypervisor instructrs the ballon driver in the requesting vm to deflate the ballon by the amount of requested memory.
 
## Ballooning
 - hypervisor asks balloon drivers inflate for guest OS which are not actively using all the memory
 - hypervisor asks ballon driver to deflate for guest os which is under memory pressure


# Parallel systems:
 ## latency
  time it takes to get teh lock with **no contention**
  
 ## contention
   the interconnection network traffic generated unpon lock release by the competing process that are spinning to acquire the lock
   
 ## waiting time
  this is a property of an application ( e.g. duration of critical section ) and does not have anythin to acquire the lock 
 
 ## memory consistency: 
  it is a contract between hardware and software to reason about the behavior of a multi-thread program

 ## false sharing:
  independent variables are stored in the same location. multiple threads modifidied them can cause unnecessary coherence action. hurting performance

 ## barrier
 tournament and dissemination barriers do not rely on global shared data struture. each process know locally when it's done with a barrier and is in the next phase of completion. Thus each processor can locally flip its sense flag, and use this local information in its communication with the other processors.

 ## ticket lock
  - lock requestor wait for its own ticket to be served. it's FCFS order. 
  - it does not meet the goal of a scalable mutual exclusion lock in large-scale multiprocessor, because every lock release results in an **update to shared variable**
  - this results in contention in the network because all waiting processors need to get the up to date new value of L->now_serving
  
 ## things needed to exploit multi cores on multi-core cpu
  - user level thread libs to spawn a process upon each thread creation
  - os provide mechanism for user level thread library to register handler with os
  - os upcalls the associated user level threads library handler when a process make a *blocking system call* if its parent is a thread library
 
# LRPC
 steps in executing cross-domain calls when client and server are on different processors
 - client cp arguments to argument stack
 - client *trap* in kernel and **present binding boject for a call**
 - kernel *validates* client request and look up *entry point* into the server using **processor descriptor table**
 - kernel redirect the request to processor preloaded with the server
 - server copies the arguments on its e-stack and return result to client using argument stack
 - server trap and signal completion

# Tornado
 - Tornado use concepts of clustered objec. clustered boject has the same obj reference but may get de-referenced to a specific represention of that object. 
 - clustered object is internally used by tornado to optimize the implementation of the OS. applications don't have knowledge of clustered objects
 - use case of existence guarantee: processor object is in a chain of page fault handling. object itself is not going to be modified but the handler needs guarantee that the object will no "go away". So processor needs not be locked, but it needs to be there
 - use case of true locking: region object in the chain of page fault handling; the handler has to eventually modify specific entry in the object corresponding to the faulting VPN. the region object requires true locking
 
 
# Corey
## region vs. address range
 - difference:
  - region is internal to kernel to optimize the implementation of page fault handling to increase concurrency
  - address range is exposed to applications. it allows applications to give hints to OS to better optimize the coherence
  
 - similarity:
  - both serve to reduce locking requirement of kerneldata structure
  
  
 - 2018 midterm last question
 

