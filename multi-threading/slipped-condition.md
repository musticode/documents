# Slipped Condition

## What is Slipped Condition in Multi-threading

### Problem
Slipped Condition is a special type of race condition that can occur in a multithreaded application. In this, a thread is suspended after reading a condition and before performing the activities related to it. It rarely occurs, however, one must look for it if the outcome is not as expected.

- Example: Suppose there are two thread A and thread B which want to process a string S. Firstly, thread A is started, it checks if there are any more characters left to process, initially the whole string is available for processing, so the condition is true. Now, thread A is suspended and thread B starts. It again checks the condition, which evaluates to true and then processes the whole string S. Now, when thread A again starts execution, the string S is completely processed by this time and hence an error occurs. This is known as a slipped condition.

### Solution
- The solution to the problem of Slipped Conditions is fairly simple and straightforward. Any resources that a thread is going to access after checking the condition, must be locked by the thread and should only be released after the work is performed by the thread. All the access must be synchronized.

With respect to the problem above, the slipped conditions can be eliminated, by locking the String object of the CommonResource class. In this scenario, the thread first gains the access and locks the String and then tries to process the String.