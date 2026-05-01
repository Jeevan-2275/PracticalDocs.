# Operating System Practical Exam Questions and Solutions

## Q1. Priority CPU Scheduling Algorithm
**Problem Statement:** Write a program to simulate the Priority CPU scheduling algorithm.

**Solution / Implementation:**
The following is a C program that implements the Non-Preemptive Priority CPU Scheduling algorithm. (Note: A lower priority number implies a higher priority).

```c
#include <stdio.h>

struct Process {
    int pid;        // Process ID
    int bt;         // Burst Time
    int priority;   // Priority
    int wt;         // Waiting Time
    int tat;        // Turnaround Time
};

// Function to sort processes based on priority
void sortProcesses(struct Process p[], int n) {
    struct Process temp;
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            // Lower number means higher priority
            if (p[i].priority > p[j].priority) {
                temp = p[i];
                p[i] = p[j];
                p[j] = temp;
            }
        }
    }
}

int main() {
    int n = 3; // Example with 3 processes
    struct Process p[] = { {1, 10, 3}, {2, 5, 1}, {3, 8, 2} }; 
    
    sortProcesses(p, n);

    // Calculate waiting time and turnaround time
    p[0].wt = 0; 
    p[0].tat = p[0].bt;
    
    float total_wt = 0, total_tat = p[0].tat;

    for (int i = 1; i < n; i++) {
        p[i].wt = p[i - 1].wt + p[i - 1].bt;
        p[i].tat = p[i].wt + p[i].bt;
        total_wt += p[i].wt;
        total_tat += p[i].tat;
    }

    printf("PID\tBurst Time\tPriority\tWaiting Time\tTurnaround Time\n");
    for (int i = 0; i < n; i++) {
        printf("%d\t%d\t\t%d\t\t%d\t\t%d\n", p[i].pid, p[i].bt, p[i].priority, p[i].wt, p[i].tat);
    }
    printf("\nAverage Waiting Time: %.2f", total_wt / n);
    printf("\nAverage Turnaround Time: %.2f\n", total_tat / n);

    return 0;
}
```

---

## Q2. Round Robin Scheduling (Time Quantum = 2)
**Problem Statement:** Calculate Completion Time, Turnaround Time, and Waiting Time using Round Robin (Time Quantum = 2).
* Process | Arrival Time (AT) | Burst Time (BT)
* P1 | 0 | 5
* P2 | 1 | 4
* P3 | 2 | 2
* P4 | 4 | 1

**Solution / Calculation:**

**Gantt Chart & Execution Execution:**
- **Time 0:** P1 arrives and starts execution (Queue: P1). Runs until Time 2.
- **Time 1:** P2 arrives. 
- **Time 2:** P1 finishes its quantum. P3 arrives. (Queue: P2, P3, P1). P2 starts execution. Runs until Time 4.
- **Time 4:** P4 arrives. P2 finishes its quantum. (Queue: P3, P1, P4, P2). P3 starts execution. Runs until Time 6.
- **Time 6:** P3 completes its burst! (Queue: P1, P4, P2). P1 starts execution. Runs until Time 8.
- **Time 8:** P1 finishes its quantum. (Queue: P4, P2, P1). P4 starts execution. Runs until Time 9.
- **Time 9:** P4 completes its burst! (Queue: P2, P1). P2 starts execution. Runs until Time 11.
- **Time 11:** P2 completes its burst! (Queue: P1). P1 starts execution. Runs until Time 12.
- **Time 12:** P1 completes its burst! (All processes done).

**Final Calculations table:**
* **Completion Time (CT):** The time when the process fully completes execution.
* **Turnaround Time (TAT):** CT - AT
* **Waiting Time (WT):** TAT - BT

| Process | AT | BT | Completion Time (CT) | Turnaround Time (TAT) | Waiting Time (WT) |
|---------|----|----|----------------------|-----------------------|-------------------|
| **P1**  | 0  | 5  | 12                   | 12 - 0 = **12**       | 12 - 5 = **7**    |
| **P2**  | 1  | 4  | 11                   | 11 - 1 = **10**       | 10 - 4 = **6**    |
| **P3**  | 2  | 2  | 6                    | 6 - 2 = **4**         | 4 - 2 = **2**     |
| **P4**  | 4  | 1  | 9                    | 9 - 4 = **5**         | 5 - 1 = **4**     |

* **Average Turnaround Time:** (12 + 10 + 4 + 5) / 4 = **7.75**
* **Average Waiting Time:** (7 + 6 + 2 + 4) / 4 = **4.75**

---

## Q3. FIFO Page Replacement Algorithm & Belady's Anomaly
**Problem Statement:** Using the FIFO algorithm, calculate the page faults with 3 frames and then with 4 frames for the Reference String `3, 2, 1, 0, 3, 2, 4, 3, 2, 1, 0, 4`. Demonstrate if Belady's Anomaly occurs.

**Solution / Calculation:**

**Scenario A: With 3 Frames (FIFO)**

| String | 3 | 2 | 1 | 0 | 3 | 2 | 4 | 3 | 2 | 1 | 0 | 4 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Frame 1** | 3 | 3 | 3 | **0** | 0 | 0 | **4** | 4 | 4 | **1** | 1 | 1 |
| **Frame 2** |   | 2 | 2 | 2 | **3** | 3 | 3 | 3 | 3 | 3 | **0** | 0 |
| **Frame 3** |   |   | 1 | 1 | 1 | **2** | 2 | 2 | 2 | 2 | 2 | **4** |
| **Fault/Hit**| **F** | **F** | **F** | **F** | **F** | **F** | **F** | H | H | **F** | **F** | **F** |

*Total Page Faults with 3 frames = **9***

**Scenario B: With 4 Frames (FIFO)**

| String | 3 | 2 | 1 | 0 | 3 | 2 | 4 | 3 | 2 | 1 | 0 | 4 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Frame 1** | 3 | 3 | 3 | 3 | 3 | 3 | **4** | 4 | 4 | 4 | **0** | 0 |
| **Frame 2** |   | 2 | 2 | 2 | 2 | 2 | 2 | **3** | 3 | 3 | 3 | **4** |
| **Frame 3** |   |   | 1 | 1 | 1 | 1 | 1 | 1 | **2** | 2 | 2 | 2 |
| **Frame 4** |   |   |   | 0 | 0 | 0 | 0 | 0 | 0 | **1** | 1 | 1 |
| **Fault/Hit**| **F** | **F** | **F** | **F** | H | H | **F** | **F** | **F** | **F** | **F** | **F** |

*Total Page Faults with 4 frames = **10***

**Conclusion: Demonstration of Belady's Anomaly**
* **Belady's Anomaly** is a phenomenon where increasing the number of page frames results in an increase in the number of page faults.
* In our calculations:
  * With **3 frames**, we got **9 page faults**.
  * With **4 frames**, we got **10 page faults**.
* Since $10 > 9$, this perfectly demonstrates that **Belady's Anomaly OCCURS** for this specific reference string when using the FIFO page replacement algorithm.
