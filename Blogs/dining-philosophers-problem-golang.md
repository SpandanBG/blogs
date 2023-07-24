
# Dining Philosophers Problem (GoLang) 

---

Multithreading and synchronization in Go using Dining Philosophers Problem

---

> "In computer science, the **dining philosophers problem** is an example problem often used in concurrent algorithm design to illustrate synchronization issues and techniques for resolving them." - [Wikipedia](https://en.wikipedia.org/wiki/Dining_philosophers_problem)

The problem states that five philosopher's sit in a round table with five bowls of spaghetti for each and a fork between each bowl (i.e. a total of five forks).

![image](https://drive.google.com/uc?export=view&id=1A6gmDMfjTGm65Y4bLSbC6V7sAC-gDFFT)

Each philosopher **needs both the right and left forks to eat the bowl** **of spaghetti,** but they **must alternate between eating and thinking.** After the philosopher finishes eating **only then** he can puts down **both** the forks, thus allowing other philosopher to pick it up.

Here each **philosopher can be regarded as a thread** and the **forks as resources** required by each thread to complete it's tasks. The challange is to avoid a deadlock amonst the threads. If each thread aquires a fork, no other forks will be left for the philosophers to pick and complete their tasks.

There are many solutions to this problem. The solution I will be discussing here is a quite a simple one called the **Arbitrator solution**. In this approch a philosopher can only pick up both the forks or non of them by introducing a arbitrator (eg. waiter). If a philosopher wants the forks, he must ask the permission from the waiter. This means each thread can either occupy all the resources it requires to complete it's task or non at all. Even though this is a logically viable solution, this isn't perfect solution for real world application, since modern applications can request for arbitary number of resources at any given instance of it's run time. In such cases many deadloack avoidance techniques are used.

Let's number each philosopher from 0 to 4, thus we have the set P = {0,1,2,3,4}, each representing one unquie philosopher. Let's number each forks from 0 to 4 too, thus we have the set F ={0,1,2,3,4}. Now, if P[0] need to eat the spaghetti, he'll need F[0] (his right) and F[1] (his left) forks free. This gives that **P[i] will need {F[i],  F[(i+1)%5]}** set of forks.

We are going to define three functons:

- **Waiter Task**: This implements the basic actions of the waiter, i.e. is to check if the forks are available (or not available) and assign them to the philosopher requesting it.
```golang
/*
       group="arbitrator_sol_1"
       title="Watier Funtion"
*/
func waiter_task(pick bool, pid int) bool {
       var i, j = pid, (pid+1)%5
       var has_value int = 1
       if pick {
              has_value = 0
       }
       if fork[i]==has_value &amp;&amp; fork[j]==has_value {
              fork[i] = (has_value+1)%2
              fork[j] = (has_value+1)%2
              return true
       }
       return false
}
```

- **Action**: This implements the basic actions of the philosopher,i.e. to pick up or put down the forks.
```golang
/*
       group="arbitrator_sol_2"
       title="Action Funtion"
*/
func Action(pick bool, pid int) {
       for true {
              // Global var: waiter -&gt; sync.Mutex
              waiter.Lock()
              if waiter_task(pick, pid) {
                     waiter.Unlock()
                     break
              }
              waiter.Unlock()
       }
}
```

- **Philosopher**: This implements the total actions of the philosopher, i.e. pick up the forks if both avaible, eat spaghetti and then drop them both.
```golang
/*
       group="arbitrator_sol_3"
       title="Philosopher Funtion"
*/
func Philosopher(id int) {
       Action(true, id)
       fmt.Println("Philosopher ", id, "pick up the forks, ate and dropped them.")
       Action(false, id)
       exit++
}
```

The source code:
```golang
/*
       group="arbitrator_sol"
       title="Arbitrator Solution"
*/
package main

import (
       "fmt"
       "sync"
)

var fork [5]int
var waiter sync.Mutex
var exit int

/*
       Method implements the waiter's tasks, 
       Either to pick up the fork
       Or to drop them.
*/
func waiter_task(pick bool, pid int) bool {
       var i, j = pid, (pid+1)%5
       var has_value int = 1
       if pick {
              has_value = 0
       }
       if fork[i]==has_value &amp;&amp; fork[j]==has_value {
              fork[i] = (has_value+1)%2
              fork[j] = (has_value+1)%2
              return true
       }
       return false
}

/*
       Method implements the actions to be
       Performed by the waiter.
*/
func Action(pick bool, pid int) {
       for true {
              waiter.Lock()
              if waiter_task(pick, pid) {
                     waiter.Unlock()
                     break
              }
              waiter.Unlock()
       }
}

/*
       Method implements the actions
       Of the philosopher.
*/
func Philosopher(id int) {
       Action(true, id)
       fmt.Println("Philosopher ", id, "pick up the forks, ate and dropped them.")
       Action(false, id)
       exit++
}

// Main
func main(){
       exit = 0
       fmt.Println("Dining Philosophers Problem - Arbitrator Solution")
       // Running five threads and supplying the IDs
       for i:=0; i&lt;5; i++ { go Philosopher(i) }
       for exit!=5 { /* wait for threads to get over */ }
}
```

One another solution involves logically ordering the actions of the philosophers such that no deadlock occurs. Here, the even philosophers will pick up the right fork first, and the odd philosophers will pick up left fork first. That means:
```generic
/*
       group="seperate_sol_algo"
       title="Algorithm"
*/

for Pi in Philosopher's Set:
    if Pi is even Philosopher, then:
        Pick up rigth fork
    else:
        Pick up left fork</pre>
```

The souce code:
```golang
/*
       group="seperate_sol"
       title="Dining Philosopher Solution"
*/
package main

import (
	"fmt"
	"sync"
	"time"
)

var fork [5]sync.Mutex
var exit int

func phil(id int) {
	for i := 0; i &lt; 2; i++ {
		if (id+i)%2 == 0 {
			fork[id].Lock()
			fmt.Println("P[", id, "] picked up Right Fork")
			time.Sleep(time.Duration(1) * time.Second)
		} else {
			fork[(id+1)%5].Lock()
			fmt.Println("P[", id, "] picked up Left Fork")
			time.Sleep(time.Duration(1) * time.Second)
		}
	}
	fork[id].Unlock()
	fork[(id+1)%5].Unlock()
	fmt.Println("P[", id, "] ate and dropped the forks")
	exit++
}

func main() {
	fmt.Println("Dining Philosophers Problem")
	for i := 0; i &lt; 5; i++ {
		go phil(i)
	}
	for exit != 5 { /* loop wait till all threads are over */
	}
}
```

---

Spandan Buragohain,
2017-12-30 06:27:51