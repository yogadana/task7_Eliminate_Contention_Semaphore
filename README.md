### Exercise 8: Eliminating Resource Contention Using Semaphores

---

#### **Introduction**
Resource contention in multitasking systems, as demonstrated in **Exercise 7**, can significantly degrade system performance. While disabling interrupts for critical sections is straightforward, it introduces severe disruptions by globally halting interrupt handling. This approach is inherently intrusive and unsuitable for general-purpose multitasking systems.

This exercise addresses resource contention in a more focused manner using **semaphores**—a synchronization mechanism that provides controlled access to shared resources with minimal disturbance.

---

### **Objective**
1. Replace interrupt disabling with semaphores for critical section management.
2. Demonstrate how semaphores can selectively ensure mutual exclusion without disrupting the entire system.
3. Maintain smooth task execution and predictable timing for all tasks while sharing resources.

---

### **Theoretical Background**
Semaphores provide a robust and scalable mechanism for managing access to shared resources in multitasking systems. A semaphore:
- Maintains a counter to track the availability of a resource.
- Blocks tasks attempting access when the resource is unavailable.
- Unblocks waiting tasks as the resource becomes available.

In this exercise, a **binary semaphore** is used:
- Counter value = 1: Resource is available.
- Counter value = 0: Resource is occupied, and any task attempting access must wait.

This approach confines access control to the specific critical section without impacting other parts of the system.

---

### **Implementation**

#### **Key Modifications from Exercise 7**

1. **Use a Semaphore for Resource Protection**
   - Replace `taskENTER_CRITICAL()` and `taskEXIT_CRITICAL()` with `osSemaphoreAcquire()` and `osSemaphoreRelease()` calls to manage resource access.
   - Ensure the semaphore is initialized with a count of 1 to act as a binary semaphore.

2. **Maintain Task Execution Behavior**
   - `GreenLEDTask` and `RedLEDTask` toggle their respective LEDs and access the shared resource (`AccessSharedData()`).
   - `OrangeLEDTask` operates independently and remains unaffected by the semaphore-protected resource.

---

### **Code Explanation**

#### **Semaphore Initialization**
```c
osSemaphoreId_t CriticalResourceSemaphoreHandle;
const osSemaphoreAttr_t CriticalResourceSemaphore_attributes = {
    .name = "CriticalResourceSemaphore"
};

void InitializeSemaphore() {
    CriticalResourceSemaphoreHandle = osSemaphoreNew(1, 1, &CriticalResourceSemaphore_attributes);
}
```
- Creates a binary semaphore named `CriticalResourceSemaphore`.

#### **Critical Section Protection**
Replace the previous interrupt disable/enable mechanism with semaphore acquisition and release:
```c
osSemaphoreAcquire(CriticalResourceSemaphoreHandle, osWaitForever);
AccessSharedData();
osSemaphoreRelease(CriticalResourceSemaphoreHandle);
```
- **Acquire**: Ensures mutual exclusion by blocking the task if the semaphore is unavailable.
- **Release**: Signals that the critical section is free for other tasks.

---

#### **Task Definitions**
Tasks are updated to use the semaphore for protecting the shared resource:
- **GreenLEDTask**: Flashes every 200 ms and accesses the critical resource.
- **RedLEDTask**: Flashes every 550 ms and accesses the critical resource.
- **OrangeLEDTask**: Toggles the orange LED every 50 ms without interacting with the resource.

**Example GreenLEDTask Implementation**:
```c
void StartGreenLEDTask(void *argument)
{
    for(;;)
    {
        HAL_GPIO_WritePin(LED_GREEN_GPIO_Port, LED_GREEN_Pin, GPIO_PIN_SET);

        osSemaphoreAcquire(CriticalResourceSemaphoreHandle, osWaitForever);
        AccessSharedData(); // Protected critical section
        osSemaphoreRelease(CriticalResourceSemaphoreHandle);

        osDelay(200);
        HAL_GPIO_WritePin(LED_GREEN_GPIO_Port, LED_GREEN_Pin, GPIO_PIN_RESET);
        osDelay(200);
    }
}
```

---

### **Expected Behavior**
1. **Smooth Task Execution**:
   - `GreenLEDTask` and `RedLEDTask` take turns accessing the shared resource using the semaphore.
   - Semaphore ensures tasks execute predictably and without interruption.

2. **No System-Wide Impact**:
   - Semaphore control is limited to the critical section (`AccessSharedData()`), unlike interrupt disabling, which affects the entire system.

3. **Uninterrupted OrangeLEDTask**:
   - Since `OrangeLEDTask` does not interact with the shared resource, it continues toggling its LED at 50 ms intervals without interference.

---

### **Advantages of Using Semaphores**
1. **Selective Control**:
   - Resource protection is limited to the critical section, leaving the rest of the system unaffected.

2. **Scalability**:
   - Semaphores are suitable for complex systems with multiple tasks and shared resources.

3. **Minimized Overhead**:
   - Unlike disabling interrupts, semaphores avoid halting unrelated system processes.

---

### **Demonstration Steps**

1. **Set Up**:
   - Use the STM32 microcontroller and configure GPIO pins for the LEDs.
   - Initialize the semaphore and ensure tasks are assigned their respective priorities.

2. **Run the System**:
   - Observe the LEDs:
     - Green and red LEDs flash at their expected intervals, sharing the critical resource seamlessly.
     - Orange LED toggles consistently without interruption.

3. **Compare Results**:
   - Verify that semaphore usage eliminates the timing disruptions seen in **Exercise 7**.


### **Demonstration Video**
https://github.com/user-attachments/assets/698c0213-68ed-424d-87d7-ec10f89a0724


---

### **Conclusion**
This exercise demonstrates how semaphores resolve resource contention effectively and with minimal disturbance to system performance. By using semaphores, multitasking systems can ensure smooth task execution, predictable timing, and reliable synchronization, even in resource-sharing scenarios.

---

### **Author**
- Dana Yoga Setya Ikhwandi (3222600016)
- Nio Perdana Azizudin (3222600022)
