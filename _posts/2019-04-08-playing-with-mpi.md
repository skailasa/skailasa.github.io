---
layout: post
title: "Getting Used to MPI"
date: 2019-04-08
---

Regular listeners, if there are any, might know that I study Scientific Computing.
A field all about simulating real world phenomena using the latest technology. As
this is largely dominated by old-school maths and physics types, we are resultantly
studying some fairly old school tech, and in one of my courses we've been studying MPI
(Message Passing Interface). It's basically a C++ library that allows you to model 
distributed memory systems, with 90s supercomputers in mind. This has begun to draw
[criticism](https://www.dursi.ca/author). This is fair enough, the prevalence of entirely
new technologies for solving large numerical systems (Hadoop, Spark) and the ubiquity
of cloud compute make HPC seem quite antiquated, and definitely at the wrong level of
abstraction (why are research scientists focussing on messaging, when they should be
focusing on the algorithms?). However, it's ubiquity as the tool of choice amongst HPC 
scientists makes knowledge of it important (for now). Parallel programming concepts are 
pretty agnostic to tools, so I thought I'd go through a simple MPI example demonstrating 
some parallellism.

We're gonna go through a dummy program that parallelises the finding of a prime number within
some user defined range of numbers. The fundamental computational complexity of this problem
is O(N), we basically are gonna check every number in some range, but we can speed up all of
this checking by chopping up this range and feeding it to different processes to go off and 
do *in parallel*.

Consider the following function which checks for primality.

```c++
#include <math.h>

namespace num 
{
bool IsPrime(unsigned candidate)
{
  bool isPrime = true;
  if (candidate == 0 || candidate == 1)
  {
    isPrime = false;
  }
  else if (candidate == 2)
  {
    isPrime = true;
  }
  else
  {
    unsigned limit = static_cast<unsigned>(ceil(sqrt(static_cast<double>(candidate))));
    for (unsigned factor(2); factor <= limit; ++factor)
    {
      if (candidate % factor == 0)
      {
        isPrime = false;
        break;
      }
    }
  }
  return isPrime;	
}
```

## Communication in MPI

MPI, as the name suggests, is all about controlling how processs communicate (pass messages)
to each other. These messages could indicate instructions such as the starting and stopping
of a computation, or contain data. Furthermore, there are two paradigms in which MPI handles
communications, 'point to point' and 'collective'. Point to point, as the name indicates, is
about specifying the communication between two named processes, similarly collective
communication is about specifying communications across groups of processes.

## Point to Point Communication

A common pattern is have a 'master' process tell a bunch of 'slave' processes what to do. All
the compute happens at the slaves, with the master just choraling the action. A few points for
clarity, before we continue, 'rank' in MPI refers to the unique name of a process, it's common
to have the rank=0 process be the master. Additionally 'size' refers to the total number of
processers available on your machine. My measly 2013 MacBook Air has just 2 cores, hence only
a master with a single slave.

```c++
#include <iostream>
#include <vector>

#include <mpi.h>

int main(int argc, char* argv[]) {
	/***********************Ignore this stuff***********************/
	MPI_Init(&argc, &argv);
	
	int world_rank, world_size;
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size)
	/****************************************************************/
	// Random number generation
 	std::minstd_rand generator;
	unsigned min(0), max(1000);
	std::uniform_int_distribution<> distribution(min, max);

	// Create buffer to store prime number candidates
	std::vector<unsigned> candidates(size - 1);

	// Buffer to store the prime that is found
	int found_prime(0);

	// Main loop continues until a prime is found
	while (found_prime == 0) {
    	if (rank == 0) {
            // Generate some candidate numbers for each process to test
            std::generate(candidates.begin(), candidates.end(),
                          [&]() { return distribution(generator); });

            // Send one to each worker, don't send one to master
            for (int worker(1); worker < size; ++worker) {
                MPI_Ssend(&candidates[worker - 1], 1, MPI_UNSIGNED,
                		  worker, 0, MPI_COMM_WORLD);
            }

            // Receive result of prime check
            for (int worker(1); worker < size; ++worker) {
                unsigned result;
                MPI_Recv(&result, 1, MPI_UNSIGNED, worker, 0,
                         MPI_COMM_WORLD, MPI_STATUS_IGNORE);
                if (result == 1) {
                    found_prime = candidates[worker - 1];
                    std::cout << "Worker " << worker << " found prime "
                              << found_prime << std::endl;
                }
            }
    	} else {
            // Receive the candidate to check
            unsigned candidate;
            MPI_Recv(&candidate, 1, MPI_UNSIGNED, 0, 0, 
            		 MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            // Do the check
            unsigned is_prime = mp::IsPrime(candidate) ? 1 : 0;
            // Return the result
            MPI_Ssend(&is_prime, 1, MPI_UNSIGNED, 0, 0, MPI_COMM_WORLD);
            if (is_prime == 1) {
                found_prime = candidate;
            }
        }
    }
    /***********************Ignore this stuff***********************/
    MPI_Finalize();
    /****************************************************************/
}
```

There's a huge amount to unpack above, so I'm going to focus on the important parts.
As mentioned, MPI consists of messages passed between processes, which they can either
be in the process of 'sending' or 'receiving'. In our algorithm, we want the master
process to send a prime it's generated to a slave process and command it to check whether
or not it is prime, returning the answer. Therefore it's extremely important that the
number of 'sends' and 'receives' exactly match, otherwise a process will be waiting
around for a signal to do something else, which will never actually arrive. This situation
is known as deadlock. Deadlock can also occur for other reasons too. Consider the following
pseudo-code.

```c++
// Check if we're on the master
if (rank == 0) {
	// If we're on the master send of message
	int msg = 1;
	int receipt;
	MPI_Ssend(&msg, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
	MPI_Recv(&receipt, 1, MPI_INT, 1, 0);
} else if (rank == 1) {
	// if we're on the single worker, unpack the message
	int msg;
	int receipt;
	MPI_Recv(&msg, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
	MPI_Ssend(&receipt, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);
}
```

Ignoring the actual syntax for a second, two things are clear from the above code. Both statements
are executed at the same time - but on different processors. The first block is executed on the
master, which we verify with a rank check, similarly for the slave. The master sends off a message
whilst the slave's first instruction is to wait to receive a message. If the slave's first action is
to also send a message, we'd get deadlock - as both master and slave are waiting around for a receipt.
The worst part about MPI is that all of these failures are completely silent. So we have to make sure
as developers that all of our communications are correct, manually, with no help from the compiler at
all.

So before even diving into syntax, we've found two potential pitfalls that could result in deadlock.
(1) the calls to send and receive don't match (2) messages are never received resulting in processes
running forever waiting for a receipt.

How do we solve these? In the above solution I've placed both rank checks within a while loop. This
ensures that as long as the for loops - sending off messages to the slaves - do NOT send a message
to the master itself, then the number of sends will always match the number of receives. A side 
effect of this is that the code will completely fail if we only have one process (though this can
be gotten around with collective communication as we shall see). Secondly, after we have a prime
we have to signal explicitly **to both the master and the slave(s)** that it's time to wrap up all
computations. Otherwise they'll be stuck in the while forever. This is done by checking the receipt
`is_prime`.

Moving onto syntax, MPI can at first seem incredibly intimidating. With massive function signatures
and confusing parameter names being passed around, and of course a massive amount of boilerplate
that doesn't really seem to do anything - but is vital for the whole thing not crashing. This is all
kind of annoying, but I've gotten used to it now after a few weeks of playing around with it, and I
guess this is one of the reasons actual MPI developers can't be bothered changing anything, habit...

In the above example we basically use two MPI function calls `MPI_Ssend` and `MPI_Recv`. Their 
parameters seem confusing, and I defer to the [docs](https://www.open-mpi.org/doc/) for what all of 
them even are, but for our purposes it's enough to know that Ssend is a **blocking-synchrnous send** 
which means that the sender waits for a receipt before moving onto another task. Similarly Recv is a
**blocking receive**, also waiting for the confirmation of receipt before moving onto another task. 
This gives us a sense of safety that our buffers won't be overwritten whilst our processes are 
communicating. However, in less trivial examples, could massively impact performance.


## Collective Communication

The above solution is quite fiddly, we had to be extremely careful to avoid
deadlock with the indices in the above loops. MPI maintainers have gotten around
this kind of problem though by wrapping this kind of functionality, where we need
to interact with all processes, in separate functions allowing the developer to
forget about what's actually going on in terms of the minutae of whether a single
process has received it's signal to terminate.

We can replace the loops above with 'Scatter' which is a wrapper function to
send of chunks of a buffer from a master process to different slave processes. Once
We've found a prime at a slave process, we tell all the other processes to stop whatever
they were doing and shut down.

This code may not look any simpler, but it is, and has a couple of big advantages. Firstly
the folks at MPI have made these functions clever enough that we can send a non-blocking
bit of info to the master node (rank = 0), meaning that it can be used for computation AND
coordination. Secondly, we've removed the burden of checking whether all of our receives match
our sends. We still have to make sure that all processes know when to shut down, but this is
bundled in a single `MPI_Allgather` function call rather than before when we had to be so
meticulous with the indices, making deadlock much easier to handle than before. This code 
should work for any number of processes, even with just one.

```c++

void computePrimesInParallel() {
    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Random number generation
    std::minstd_rand generator;
    unsigned min(22), max(100);
    std::uniform_int_distribution<> distribution(min, max);

    // Calculate elements per process
    unsigned long elementsPerProcess = 1;


    unsigned found_prime(0);
    while (found_prime == 0) {

        std::vector<unsigned> subsetOfArray(elementsPerProcess);
        std::vector<unsigned> candidates(size);

        // Generate candidates to check for primality
        // Create some candidate numbers for each process to test
        std::generate(candidates.begin(), candidates.end(),
                      [&]() { return distribution(generator); });

        std::cout << "number of candidate: " << candidates.size() << std::endl;

        // Scatter accross all processes including root
        MPI_Scatter(candidates.data(), elementsPerProcess, MPI_UNSIGNED,
                    subsetOfArray.data(), elementsPerProcess, MPI_UNSIGNED,
                    0, MPI_COMM_WORLD);

        // Loop is extraneous as we only have one element per process, but this is flexible
        for (int i(0); i < subsetOfArray.size(); i++) {

            std::cout << "checking: " << subsetOfArray[i] << " at proc: " << rank << std::endl;
            int isPrime = mp::IsPrime(subsetOfArray[i])  ? 1 : 0;
            std::cout << "result: " << isPrime << std::endl;

            if (mp::IsPrime(subsetOfArray[i])) {
                found_prime = subsetOfArray[i];
                std::cout << "Found prime: " << found_prime << " at proc: " << rank << std::endl;

                // One we've found a prime, broadcast this news back to all other procs
                MPI_Bcast(&found_prime, 1, MPI_UNSIGNED, rank, MPI_COMM_WORLD);
            }
        }

        // Check if there has been any news regarding primes, if so put into global prime
        int global_prime;
        MPI_Allgather(
                &found_prime,
                1,
                MPI_UNSIGNED,
                &global_prime,
                1,
                MPI_UNSIGNED,
                MPI_COMM_WORLD
        );

        // Break out of loop if we've found a prime globally
        if (global_prime != 0) {
            std::cout << "global prime: " << global_prime << std::endl;
            found_prime = global_prime;
        }
    }
}

void main(int argc, char* argv[])
{
   /***********************Ignore this stuff***********************/
    MPI_Init(&argc, &argv);
	
    int world_rank, world_size;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
    MPI_Comm_size(MPI_COMM_WORLD, &world_size)
    /****************************************************************/
    computePrimesInParallel();
    /***********************Ignore this stuff***********************/
    MPI_Finalize();
    /****************************************************************/
}
```

The only thing we have to keep in mind here is that the default behaviour of MPI is
to return from all of your processes, even non-root processes, this is something you
have to keep in mind (to test for rank) if you want to use the results of these computations
further downstream - rather than just printing it out as I've done above.

Though the worst part of MPI, the fact that you always fail silently, is still an issue
despite these useful wrappers, and this has made me realise how much I rely on both the
compiler and IDE to put me right when coding anything.

## Conclusions

There wasn't a great deal of syntax explanation up there, but to be honest with MPI
you have to just get stuck in to get your head around the syntax. More importantly
we've seen a few different ways of parallelising a simple problem using MPI, and how
deadlock can be a serious issue. I think that these two implementations are fairly
decent templates for large variety of problems though, so yeah copy and paste away.
