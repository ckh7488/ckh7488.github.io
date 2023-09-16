---
title: Column Space
auther: ckh
date: 2023-08-26 22:41:28 +0800
categories: [Math, LinearAlgebra]
tags: [LinearAlgebra]    
---

## 개요
행렬 연산의 직관적인 이해를 위해, ``column space``의 관점에서 봐야 할 때가 있다.  
column space 관점을 간단히 이해해보자.  

## column space
이름의 뜻 그대로, 세로선을 하나의 벡터로 보는 관점이다.  
이 관점으로 볼 시, 곱셈의 접근이 달라지게 되는데,  
$$\begin{pmatrix}a_{1} & b_{1}\\ a_{2} & b_{2} \end{pmatrix}\times\begin{pmatrix}x\\ y\end{pmatrix}=\begin{pmatrix}a_{1}\\ a_{2}\end{pmatrix}x+\begin{pmatrix}b_{1}\\ b_{2}\end{pmatrix}y$$
이런 식으로 벡터의 곱이 column 벡터의 조합으로 표현 할 수 있다.  
이 방식으로 행렬 곱셈을 이해할 때 재밌는 관점이 생기는데,  
원래 벡터를 ``basis`의 조합으로 생각 할 수 있게 된다.  
위의 $$\begin{pmatrix} x \\ y \end{pmatrix}$$벡터를 다시 생각해보면  
$$\begin{pmatrix} x \\ y \end{pmatrix}=\begin{pmatrix} 1&0 \\ 0&1 \end{pmatrix}\begin{pmatrix} x \\ y \end{pmatrix}$$  
위처럼 생각 할 수 있으며, 이는 $$\begin{pmatrix} 1 \\ 0 \end{pmatrix}x+\begin{pmatrix} 0 \\ 1 \end{pmatrix}y$$  
처럼, 두 basis 1,0 과 0,1 의 선형 조합으로 이루어 져있다 볼 수 있다.  
이 관점에서 위의 식을 다시 보면,  
basis a * x 와 basis b * y 로 판단이 가능하고,   
이 표현은 결국 원래의 [1,0] 와 [0,1]의 basis에서 basis a, basis b 의 공간으로 변환이 이루어 졌다고 생각 할 수 있다.  
행렬의 곱셈연산은 basis의 변경으로 인한 공간변환으로 해석이 가능해지고,  
결과값은 공간에서 공간변환 이후의 좌표를 [1,0], [0,1] (I matrix) 를 basis로한 공간에서 표시한 것이라 볼 수 있다.  

![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/f867afc9-6faa-4e34-aefd-f2f56607e157)
위의 그림에서 원래 공간의 1,1좌표는 $$\begin{pmatrix}2 & 0\\ 0 & 2\end{pmatrix}$$ 공간변화를 거친 후의 공간변화 이후의 좌표계에서는  1,1이다.  
원래의 I matrix에서이 공간변환 된 좌표계의 값을 바라 볼 때, 값은 [2,2] 가 되고, 이는
$$\begin{pmatrix}
2 & 0\\ 
0 & 2
\end{pmatrix} \times \begin{pmatrix}
1 \\ 
1 
\end{pmatrix}=\begin{pmatrix}
2 \\ 
2 
\end{pmatrix}$$
로 표현된다.  
