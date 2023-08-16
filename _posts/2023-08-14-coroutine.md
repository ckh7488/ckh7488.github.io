---
title: "[system] coroutine "
author: "ckh"
date: "2023-08-14 15:31:08 +0800"
categories: ["system", "multiThreads"]
tags: ["system"]  
---

# Threads 분류  
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/d2ae8b4b-bbf7-4a4a-a601-c914e0ca5878)
Threads 는 크게 2종류(HW, SW)가 있다. 이 분류에 대한 내용은 [참조](https://ckh7488.github.io/posts/Threads/)  
일반적으로 Thread 라 하면 이 중 SW Thread를 의미하며(이하 Thread 혹은 쓰레드), 이 쓰레드는 OS Thread와 User Thread로 나뉜다.  
이 두 쓰레드의 간단한 설명, 사용법을 알아보고, 코루틴 및 goroutine에 대해 알아보자.  
  
# OS Thread  
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/15e8d113-944b-4148-9f26-6d169f442711)  
OS에서 직접 쓰레드를 생성하여, 유저에게 리턴해주는 방식으로 사용된다.  
이 쓰레드들은 state를 가지며 이를 통해 OS의 scheduler가 관리한다.  
OS는 HW쓰레드에 이 OS Thread를 분배하는 식으로 사용한다.  
HW Thread가 2개이고, OS가 만든 Thread가 1000개이면, 각 state와 우선순위등을 적용하여 이 2개의 HW 쓰레드에 OS Thread를 시간별로 스케쥴링하여 실행한다.  
## 성능
성능의 이슈가 되는 부분은 
1. HW쓰레드에 실행되는 OS쓰레드가 변경될 때, Context switching이 일어나는데 이때 overhead  
2. OS에서 User의 요청으로 Thread가 직접 생성되고 이를 User에 반환할 때
크게 이 두가지가 있다.
1. 특정 조건(IO bound work)을 만족 할 경우, Coroutine 혹은 유저 레벨 쓰레드를 이용하여 성능을 개선한다.
2. Thread pool이라는 구조를 이용하는데, 프로그램 실행 시, 미리 Theads를 정해진 숫자 만큼 만들어놓고, 반환하지 않고 계속 만들어둔 쓰레드에 작업을 분배하는 방식이다.
  
아래는 OS thread(Pthread)를 이용하여 두 가지(keboard, network)IO작업을 동시에 실행시키는 코드이다.  
```C++
#include <iostream>
#include <pthread.h>
#include <unistd.h>  // for sleep function

// Function to get keyboard input
void* get_keyboard_input(void* arg) {
    std::string text;
    while (true) {
        std::cout << "Enter something (or 'exit' to stop): ";
        std::cin >> text;
        if (text == "exit") {
            break;
        }
        std::cout << "You entered: " << text << std::endl;
    }
    return nullptr;
}

// Function to simulate network IO
void* simulate_network_io(void* arg) {
    while (true) {
        // Simulating network fetch (by simply sleeping for 5 seconds)
        sleep(5);
        std::cout << "Fetched data from the network!" << std::endl;
    }
    return nullptr;
}

int main() {
    pthread_t keyboard_thread, network_thread;

    // Create threads
    pthread_create(&keyboard_thread, nullptr, get_keyboard_input, nullptr);
    pthread_create(&network_thread, nullptr, simulate_network_io, nullptr);

    // Wait for the keyboard thread to finish
    pthread_join(keyboard_thread, nullptr);
    
    // Terminate the network thread since we're done
    pthread_cancel(network_thread);

    return 0;
}
```
  
# User Thread(Coroutine)
Coroutine등에서 주로 이용되며, OS의 context switching overhead를 줄이기 위해 생겨났다.  
OS 쓰레드를 사용하여 IO작업을 할 경우, IO마다 OS Thread를 생성하여 block될 때마다 다른 작업이 가능한 Thread로 실행을 변경하는 방식으로 작동된다.  
이는 많은 context switching과 유휴시간이 많은 OS Thread들이 다량 만들어지게 되는 문제점이 생긴다.  
또한, 프로그래밍 방식도 복잡해지는 부과적인 문제점 역시 생긴다.  
이를 해결하기 위해, Coroutine은 user level에서 blocking 되는 쓰레드를 쉬지 않고 다른 일을 하도록 스케쥴링 하였다.  
또한, 프로그래밍 방식 역시 기존의 방식을 그대로 쓰도록 설계하여 분석 및 작성도 편하도록 설계하였다.  

즉, 코루틴의 설계 핵심은
* IO bound task의 불필요한 context switching과 OS thread 생성을 막고,
* 프로그래밍 방식을 기존의 방식과 최대한 동일하도록 하는 것이다.

이를 가능하게 하는 핵심 아이디어는 OS Thread가 느린 IO에 의해 block 될 때,  
OS 스케쥴러 스케쥴링을 통해 멀티쓰레드 프로그래밍을 하지 않고(block된 쓰레드를 context switching 하여 실행이 필요한 쓰레드 실행하는 방식)
실행되는 쓰레드가 block될 때 내부적으로 실행이 필요한 User thread(task)에게 실행을 양보하는 것 이다.  

이를 통해 
1. 불필요한 context switching과 Thread 생성을 막고, 스케쥴링할 OS쓰레드 개수를 줄여 프로그램이 좀더 효율적으로 실행되고
2. 유저레벨에서 여러 tasks를 관리하여 직관적인 프로그래밍이 가능해진다.
  
물론 CPU 작업이 필요한 일은 UserThread가 해결해주지 못한다.  
CPU작업이 중요한 프로그램이 single threa에서 IO가 자연스럽게 되려면  
* timeslicing을 직접 구현하여 cpu작업하다 IO를 확인하는 while루프식으로 작성
* OS thread를 사용하여 OS 스케쥴러가 알아서 동시실행되는 것 처럼 작동하도록 작성
멀티 cpu라면 그냥 OS Thread를 추가적으로 사용하여 직접 많은 HW Thread가 동시 실행되도록 해야한다.  
  
아래는 코루틴으로 비동기 IO작업들을(키보드, network IO) 동시에 실행시키는 예시이다.    
가독성이 더좋은 python코드
```python
import asyncio
import aiohttp

async def get_keyboard_input():
    while True:
        text = input("Enter something (or 'exit' to stop): ")
        if text == "exit":
            break
        print(f"You entered: {text}")

async def fetch_data():
    async with aiohttp.ClientSession() as session:
        while True:
            async with session.get('https://api.example.com/data') as response:
                print("Fetched data from the network!")
                await asyncio.sleep(5)  # Simulating repeated network fetch every 5 seconds

loop = asyncio.get_event_loop()

# Create the tasks
keyboard_task = loop.create_task(get_keyboard_input())
network_task = loop.create_task(fetch_data())

# Run until the keyboard task is done (when user enters 'exit')
loop.run_until_complete(keyboard_task)
```  
같은 역할을 boost lib을 이용한 c++ 코드.  
```C++
#include <iostream>
#include <boost/asio.hpp>
#include <boost/asio/spawn.hpp>

boost::asio::io_context io;

void get_keyboard_input(boost::asio::yield_context yield) {
    char data[128];
    while (true) {
        boost::asio::async_read_until(boost::asio::posix::stream_descriptor(io, STDIN_FILENO), boost::asio::buffer(data), '\n', yield);
        std::string text(data);
        if (text == "exit\n") {
            io.stop();
            break;
        }
        std::cout << "You entered: " << text;
    }
} 

void simulate_network_io(boost::asio::yield_context yield) {
    while (true) {
        boost::asio::steady_timer timer(io, boost::asio::chrono::seconds(5));
        timer.async_wait(yield);
        std::cout << "Fetched data from the network!" << std::endl;
    }
}

int main() {
    boost::asio::spawn(io, get_keyboard_input);
    boost::asio::spawn(io, simulate_network_io);
    io.run();
    return 0;
}
```
  
# Goroutine  
``Go``언어에서는 이 coroutine을 한 단계 더 발전시켜 사용한다.  
위의 OS Thread나 User Thread 둘을 같은 방식으로 프로그래밍 할 수있도록 런타임 스케쥴러가 내부적으로 존재한다.  
또한, channel을 이용한 동기화를 지원한다. 이 channel은 동기적으로 작동되며, 채널을 기다리는 코드는 return될 때까지 대기상태가 된다.  
이 두 특성을 사용하여, GO 언어의 goroutine은 IO작업이던, CPU작업이던 상관 없이 Goroutine을 통해 프로그래밍을 할 수 있다.  
파이썬이나 자바스크립트에서 CPU bound 작업은 멀티프로세싱이나 worker를 사용하여 따로 구현했다면, goroutine에서는 두 작업 모두 goroutine을 통해 프로그래밍한다.  
~(물론, cpu bound 한 작업을 할때 go runtime에 대한 이해가 있어야 한다)~  
  
# 결론
coroutine은 IO bound한 작업을 Single Thread에서 효율적/직관적으로 돌아가도록 만들어주는 프레임 워크이다.  
cpu bound한 작업은 python에선 multiprocess, js에서 worker, c/c++ 에서 Thread를 사용하는게 편하다.   
 * 물론, single Thread환경에서 time slicing(polling)이나 간단한 스케쥴러를 구현해서 사용해도 된다.


      
