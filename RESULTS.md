# Lottery Scheduler Results

## Setup
I modified xv6 to use scheduling instead of round-robin. Each process was assigned a comfigurable number of tickets using the 'settickets(unt n)' system call.

To test I moved testlottery from the user folder to the xv6 module which:
- Validates 'settickets'
- Burns CPU in a loop
- Allows testing under different ticket counts

## Test 1: Validation
Command: 
    testlottery 10
Output: 
    PASS: settickets validation
    testlottery: done

Comfirms settickets(0) correctly fails, settickets(n >= 1) succeeds, ans the system call is properly wired

## Test 2: Proportional Fairness Experiment

To observe lottery scheduling behavior, two CPU-bound processes were created
using a custom test program (`lotterytest`).

### Test Setup

- Child 1: `settickets(10)`
- Child 2: `settickets(40)`
- Both perform identical CPU-intensive work.
- xv6 was run with a single CPU:
    make qemu-nox CPUS=1 QEMU=/usr/libexec/qemu-kvm

A start barrier (pipe synchronization) ensured both processes began
their CPU work at the same time, preventing head-start bias.

The parent process waited for both children and recorded which finished first.

### Observed Behavior

Across repeated runs:

- The 40-ticket process finished first more frequently.
- Occasionally, the 10-ticket process finished first.

This is expected since lottery scheduling is probabilistic.

Since 40 tickets represent 4× the weight of 10 tickets,
the 40-ticket process received more scheduling opportunities
and therefore tended to complete its CPU work sooner.

### Key Observations
- Ticket count affects **selection probability**, not workload size.
- With only one runnable process, ticket count has no observable effect.
- On multiple CPUs, the effect is less visible because processes can run simultaneously.
- On a single CPU with synchronized start, proportional behavior is clearly observable.

### Conclusion
The implemented lottery scheduler behaves as expected:
- Processes are scheduled proportionally to their ticket counts.
- `settickets` correctly updates scheduling weight.
- Over repeated trials, higher-ticket processes receive more CPU time.

The scheduler successfully replaces round-robin behavior with
probabilistic proportional scheduling.