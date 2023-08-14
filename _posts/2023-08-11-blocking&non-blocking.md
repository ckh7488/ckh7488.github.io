---
title: blocking non-blocking
auther: ckh
date: 2023-08-11 06:55:00 +0800
categories: [Embedded]
tags: [Embedded]    
---

# 개요
임베디드 시스템을 프로그래밍 하는 경우, ``user-level``코드의 밑단에 있는 OS, hardware에 대해 알아야 하는 경우가 많다.  
개념을 간단히 정리하고, 어떤 역할이 어떤 layer에 있는지 간단히 확인해보자.  
  
## blocking vs non-blocking ? synchronous vs asynchronous?  

### blocking vs non-blocking
이 부분은 ``OS``레벨에서 ``system call``의 특성을 나타낸다.  
OS에서 blocking과 non-blocking을 관리한다.  
SW Thread(이하 thread)마다 state를 가지고, 이를 scheduler를 통해 우선순위를 정하여 실행시킨다.  
  
#### non-blocking  
> ''non-blocking`` syscall 의 경우, 호출 시 해당 함수가 바로 return한다. 결과에 대한 상태를 return 해준다.

``` C++
int senderSock = socket(AF_INET, SOCK_DGRAM, 0); // 아래에 오류처리 필요
fcntl(senderSock, F_SETFL, O_NONBLOCK) // ''
int sentLen = sendto(snderSocket, ... );
if(sentLen < 0)
        {
             if (errno == EWOULDBLOCK || errno == EAGAIN) 
             {
                std::cout << "kernel buffer full" <<std::endl;
                continue;
             }
             else
             {
                perror("sendto error");
                break;
             }
        }
        else
        {
            std::cout << sentLen << std::endl;
        }
```
위와 같이 소켓통신을 할 경우, 위의 sendto 함수는 바로 return하며, 결과값에 따라 성공/실패 여부를 알 수 있다.  
반대의 경우는 일반적인 ``file read`` 함수를 볼 수 있는데, 읽을 값이 있을 때 까지 ``blocking`` 한다.  
  
#### blocking  
> ``blocking`` syscall 의 경우, 호출 시 해당 함수가 return할 때 까지 호출한 thread를 ``block`` 한다.  

함수가 blocking 일 경우, 해당 thread를 blocking 상태로 변경 후 현재 실행 가능한 다른 쓰레드로 스위칭하는 context switching을 한다.  
blocking상태의 thread가 기다리는 값이 interrrupt를 통하여 확인되면, ISP을 통해 해당 thread가 ready 상태로 변경된다.
스케쥴러가 ready된 상태의 thread를 스케쥴러 알고리즘에 따라 실행시킨다(바로실행 안될수도 있다)
  
  
### synchronous vs asynchronous  
> Synchronous: Doing one thing at a time, waiting for it to finish before moving on to the next task.  
> Asynchronous: Starting a task, moving on to another before it completes, then dealing with the task's outcome (if needed) once it's done. It does not imply concurrency, although they often go together.  
이 두개념은 ``User level``의 코드에서 주로 쓰인다.  
  
위의 blocking, non-blocking과 거의 비슷해 보인다.  
동기/비동기는 system call 사용에 달려있는 경우가 많기 때문이다.   
  
#### 이해를 돕기위한 예  
``python``에서 ``asyncio``모듈의 ``run_in_executor``함수를 생각해보자.  
이 함수는 blocking 모델로 만들어진 task들을 non-blocking 한 것처럼 사용 할 수 있게 해준다.   
즉, ``synchronus한 코드를 asynchronous하게`` 실행시켜준다.  
```python
# jupyter note
import asyncio
import time

def sync_fun():
    time.sleep(1)  # simulate the synchronous IO.
    print("hello")

loop = asyncio.get_event_loop()

# # this takes 10 sec, it's synchronous way.
# for _ in range(10):
#  sync_fun()

# this takes around 1 sec
for _ in range(10):
    loop.run_in_executor(None, sync_fun)
```
위의 코드는 SW thread를 이용하여 sync_fun을 실행한다.   
for 루프 마다, SW thread가 하나씩 생성된다고 볼 수있다.
time.sleep(1)을 실행 시, OS에서 blocking으로 state를 변경하며 해당 함수를 실행하던 쓰레드는 다른 작업을 진행한다.


물론, 위의 방법 말고 퓨어한 asynchronus 함수는 아래와 같다.
```python
async def async_fun():
    await asyncio.sleep(1) # simulate the asynchronous IO
    print("hello")

tasks = [async_fun() for _ in range(10)]
asyncio.gather(*tasks)
```
위의 코드는 yeild 하면서 실행되므로, 하나의 쓰레드에서 실행된다.  
  
  
위의 두 코드 모두 asynchronous한 실행을 구현한 예이다.  
* 첫번째 코드는 각각의 함수를 실행하는 쓰레드들은 동기적이나, 전체 코드는 쓰레드풀을 이용하여 비동기적으로 실행된다.  
* 두번째 코드는 각각의 함수 자체부터 비동기적으로 구현되어있다. 따라서, 하나의 쓰레드로 작동하며, 이 역시 비동기적인 실행이다.  

#### 코루틴
유저레벨의 프로그래밍을 할 경우, OS 단의 블로킹/논블로킹 부분이 안보이도록 래핑되어 있는 경우가 많다. 유저레벨에서 대표적으로 많이 쓰이는 코루틴을 확인해보자.  
코루틴은 asynchronous 프로그램의 프로그래밍 방식을 일반적인 프로그래밍 방식과 같게 하기 위해 만들어 졌다. 위의 코드만 봐도, 쓰레드를 쓰던 안쓰던 실행 방식이 원래 코딩하던 방식과 비슷한 것을 알 수 있다.  
해당 패턴을 유지하기 위해, 코루틴은 OS의 blocking/non-blocking 방식 모두 코루틴 방식으로 끌어 쓸 수 있다.   
즉, asynchronus한 유저레벨 코딩패턴을 위해 아랫 단을 다 코루틴 으로 래핑한 구조이다.    
>파이썬은 위의 예처럼, blocking 한 코드도 코루틴 패턴으로 사용 가능하도록 설계되었으며, 새로 만드는 코드(두번째)는 코루틴에 맞게 코딩하면 된다.   
>자바스크립트의 경우, 언어자체에서 비동기 논블로킹을 강제하므로, 블로킹한 코드 자체가 존재하지 않는다. 따라서, 모든 프로그래밍 방식이 두번째 형태 즉, 코루틴 형태에 맞도록 코딩되어있다.  

