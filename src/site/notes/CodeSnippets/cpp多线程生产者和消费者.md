---
{"dg-publish":true,"permalink":"/CodeSnippets/cpp多线程生产者和消费者/","noteIcon":"3"}
---


```cpp
#include <iostream>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>

int counter = 0;
int maxSize = 30;
std::mutex mtx;
std::queue<int> dataQuene; // 被生产者和消费者共享

std::condition_variable producer, consumer; // 条件变量是一种同步机制，要和mutex以及lock一起使用

void func_consumer()
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(1000)); // 消费者比生产者慢

        std::unique_lock<std::mutex> lck(mtx);
        // void wait (unique_lock<mutex>& lck, Predicate pred) = while (!pred()) wait(lck);
        consumer.wait(lck, []
                      { return dataQuene.size() != 0; }); // 消费者阻塞等待， 直到队列中元素个数大于0
        int num = dataQuene.front();
        dataQuene.pop();
        std::cout << "consumer " << std::this_thread::get_id() << ": " << num << std::endl;

        producer.notify_all(); // 通知生产者当队列中元素个数小于maxSize
    }
}

void func_producer()
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(900)); // 生产者稍微比消费者快
        std::unique_lock<std::mutex> lck(mtx);
        producer.wait(lck, []
                      { return dataQuene.size() != maxSize; }); // 生产者阻塞等待， 直到队列中元素个数小于maxSize

        ++counter;
        dataQuene.push(counter);
        std::cout << "producer " << std::this_thread::get_id() << ": " << counter << std::endl;

        consumer.notify_all(); //通知消费者当队列中的元素个数大于0
    }
}

int main()
{
    std::thread consumers[2], producers[2];

    // 两个生产者和消费者
    for (int i = 0; i < 2; ++i)
    {
        consumers[i] = std::thread(func_consumer);
        producers[i] = std::thread(func_producer);
    }

    // for (int i = 0; i < 2; ++i)
    // {
    //     producers[i].join();
    //     std::cout<<"join middle end"<<std::endl;
    //     consumers[i].join();
    //     std::cout<<"join middle end"<<std::endl;
    // }
    producers[0].join();

    system("pause");
    return 0;
}

```
