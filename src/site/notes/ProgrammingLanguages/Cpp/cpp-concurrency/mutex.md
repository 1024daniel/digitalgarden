---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Cpp/cpp-concurrency/mutex/","noteIcon":"3"}
---

### summaries

### 1.使用RAII idiom lock method
针对mutex的lock和unlock操作，为了防止出现exception或者其他分支情况导致锁未释放，推荐使用具有RAII思想的`std::lock_guard`

```cpp
#include <list>
#include <mutex>
#include <algorithm>
std::list<int> some_list;
std::mutex some_mutex;
void add_to_list(int new_value)
{
std::lock_guard<std::mutex> guard(some_mutex);
some_list.push_back(new_value);
}
bool list_contains(int value_to_find)
{
std::lock_guard<std::mutex> guard(some_mutex);
return std::find(some_list.begin(),some_list.end(),value_to_find)
!= some_list.end();
}

```



### 2. perfer mutex and operation functions in a class
对于protected data，mutex可以在cpp文件中当做global variable，一些针对protected data的操作需要获取mutex，但是有限推荐使用面向对象的思想将protected data，mutex lock还有对应的operations封装到一个类中，针对protected data的操作为类中的私有函数
3.对于类中被mutex保护的protected data，如果存在一个member function能返回对应数据的reference or pointer，所有的lock都是徒劳，因为这会给对应的protected data留下一个backhole，其他threads即使没有获取到lock但是通过reference or pointer都可以modify protected data，另外do not pass protected data as arguments to user-supplied functions

### 3. pay attention to race condition inherent in some interfaces
如下所示， stack的pop接口是通过mutex锁互斥的，但是对于用户操作接口层面会出现两个线程读取了同一个值，但是将两个元素都pop了，导致一个元素从始至终都没有被read，这可能导致非常难以发现的bug
![Pasted image 20240504161306.png](/img/user/ProgrammingLanguages/Cpp/cpp-concurrency/attachments/Pasted%20image%2020240504161306.png)
对于这类问题，一般是通过这个接口内的操作都用mutex来进行线程间互斥，也就是粗粒度的锁(lock at large a granularity), 但是对于锁住过多的data会导致eliminate any performance benefits of concurrency,

这里我们自行实现一个stack的pop函数来解决这个问题，我们将上图的操作纳入到一个pop函数，这个函数内是mutex互斥的，
```cpp
T pop()

```
对于这个函数可能存在一个问题，比如以下操作会先将stack的top元素先删除之后再返回，如果内部元素T是一个比较大的vector类型，再删除栈上元素之后再将vector拷贝的时候会存在可能资源不足的情况导致失败，最后导致数据丢失
```cpp
T top = stack.pop() 

```
对于这个问题解决options有
	1. PASS IN A REFERENCE
	2. REQUIRE A NO-THROW COPY CONSTRUCTOR OR MOVE CONSTRUCTOR
	3. RETURN A POINTER TO THE POPPED ITEM
	4. PROVIDE BOTH OPTION 1 AND EITHER OPTION 2 OR 3

```cpp
#include <exception>
#include <memory>
#include <mutex>
#include <stack>
struct empty_stack : std::exception {
  const char* what() const throw();
};
template <typename T>
class threadsafe_stack {
private:
std::stack<T> data;
mutable std::mutex m;
public:
threadsafe_stack() {}
// do the copy in the constructor body rather than the member initializer list in order to ensure that the mutex is held across the copy.
threadsafe_stack(const threadsafe_stack& other) {
  std::lock_guard<std::mutex> lock(other.m);
  data = other.data;
}
threadsafe_stack& operator=(const threadsafe_stack&) = delete;
void push(T new_value) {
  std::lock_guard<std::mutex> lock(m);
  data.push(new_value);
}
// option3, return pointer
std::shared_ptr<T> pop() {
  std::lock_guard<std::mutex> lock(m);
  if (data.empty()) throw empty_stack();
  // store pointer for the item before popping
  std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
  data.pop();
  return res;
}
// option1, pass in the reference
void pop(T& value) {
  std::lock_guard<std::mutex> lock(m);
  if (data.empty()) throw empty_stack();
  value = data.top();
  data.pop();
}
bool empty() const {
  std::lock_guard<std::mutex> lock(m);
  return data.empty();
}
};

```

对于fine-grained locking schemes，我们有时会不可避免的需要同事持有多个mutex，这会导致另一个问题存在[[ProgrammingLanguages/Cpp/cpp-concurrency/dead lock\|dead lock]]
