---
title: Interconnect Matrix
auther: ckh
date: 2023-08-22 19:45:10 +0800
categories: [HW, embedded]
tags: [embedded]    
---

# 개요
필자는 STM32MP157 MPU 보드를 개발하고 있다.  
개발 중에 peripheral을 HAL Library를 거치지 않고 baremetal 처럼 사용해야 하는 경우가 종종 발생한다.  
이때 마주하게 되는 것이 Interconnect Matrix이다.  
하드웨어가 칩에서 직접적으로 연결되어 있는 것을 표현한 것이며, ARM cpu의 경우 AMBA라는 버스 시스템(하드웨어,프로토콜)을 주로 사용한다.  
이 MPU에서 AXI와 AHB를 A7과 M4를 연결하며 서로간의 통신을 위해 둘을 이어주는 부분도 존재한다.  
Interconnect Matrix에 대해 알아보자.  

# 설명
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/0282787f-a5cc-4e58-87b4-00c6fe921ab1)
위 그림은 하나의 칩에서 각 코어(A7 * 2, M4 * 1)가 어떻게 연결되어 있는지 알 수 있다.  
연결된 상태를 매트릭스화 하여 어떤 인터페이스로 어떻게 연결되어있는지 쉽게 도식화 된 것이 아래의 Interconnect Matrix 이다.  
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/64a9b002-0b14-475a-97f5-0fe1a47b6b5a)
마스터(Mx)에서 슬레이브(Sx)와 통신이 가능하다. 
* 흰색 접점은 Slave 디바이스가 enabled상태일 때 통신 가능하며,
* 검은 접점은 하나의 마스터가 두 포트를 한번에 접근하진 못한다는 의미이다.
 * 위의 그림에서 DDRCTRL peripheral 만 해당되는데, ETH가 접근 할 때, S0 또는 S1 둘중 하나만 접근 가능하다는 의미이다.
 * 또한, port가 두개이므로, master1가 S0을 사용하고, 다른 master2가 S1을 접근하는 식으로는 동작이 가능 한 것으로 알고있다(확실치는않음)  

## 예제 
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/5520e009-712d-45cd-8bae-e0a2bff38d11)
M4의 AHB를 보면 M4의 S,D,I선이 각각 M7,8,9에 연결되어 있다.  
M4가 SRAM1에서 Instruction(I)를 가져와 실행시키려면, M9와 SRAM1이 통신을 하게되며, 통신중에는 다른 하드웨어가 접근이 불가능하다.  
