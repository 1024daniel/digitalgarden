---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Cpp/cpp-concurrency/dead lock/","noteIcon":"3"}
---

references:
https://medium.com/@back_to_basics/c-11-locking-strategy-adopt-lock-and-defer-lock-eeedf76a2689
https://stackoverflow.com/questions/76549242/why-use-defer-lock-with-unique-lock

#deadlock
对于死锁问题可以用一个toy来进行举例，一个toy包含两个组件drum and drumstick，两个小孩想要击鼓的话需要同事持有drum and drumstick才能正常跑起来，但是如果A拿到了drum，B拿到了drumstick，除非其中一个be nice and give the other piece, 否则就会出现两个小孩都没法玩.
在计算机世界，concurrency场景下的deadalock问题一般就是由以上现实例子一样存在多个线程相互等待的情况.

> [!NOTE] Guidance
> The guidelines for avoiding deadlock all boil down to one idea:  
don’t wait for another thread if there’s a chance it’s waiting for you. The individual  
guidelines provide ways of identifying and eliminating the possibility that the other  
thread is waiting for you.


避免死锁问题的一般解决方法
### 1. lock mutex in the same order
每个进程必须先hold mutex A then hold mutex B，然而如果A和B是protect同一个类的不同instances的话就没法区分
### 2. lock multiple mutex in one call provided by C++ Standard Library
C++ 标准库实现lock可以同时持有多个mutex，并且防止出现deadlock
deadlock avoidance algorithm to avoid deadlock.
```cpp
class some_big_object;
void swap(some_big_object& lhs, some_big_object& rhs);
class X {
 private:
  some_big_object some_detail;
  std::mutex m;

 public:
  X(some_big_object const& sd) : some_detail(sd) {}
  friend void swap(X& lhs, X& rhs) {
    if (&lhs == &rhs) return;
    std::lock(lhs.m, rhs.m);
    // adopt_lock used to assume the calling thread already has ownership of the mutex without having to relock
    // using lock_guard to make sure both already-locked mutexes are unlocked at the end of scope
    std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
    std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
    swap(lhs.some_detail, rhs.some_detail);
  }
};
```
### 3.  avoid nested lock

### 4.  AVOID CALLING USER- SUPPLIED CODE WHILE HOLDING A LOCK
### 5. USE A LOCK HIERARCHY