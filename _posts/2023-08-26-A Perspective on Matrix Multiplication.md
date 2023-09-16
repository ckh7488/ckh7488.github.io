---
title: A Perspective on Matrix Multiplication
auther: ckh
date: 2023-08-26 22:41:28 +0800
categories: [Math, LinearAlgebra]
tags: [LinearAlgebra]    
use_math: true
---

## Introduction
행렬 곱셈의 직관적인 이해를 위해 ``column space``의 관점에서 봐야 할 때가 있다.  
특히, 딥러닝쪽에서 행렬곱을 공간변환의 관점에서 말하는 경우가 많으므로 이 관점에 대한 직관적인 이해가 필요하다.  
Column space의 정의는  
>The column space of a matrix is the set of all possible linear combinations of its column vectors  
인데, 이에 대한 내용을 아래에서 설명하고, 공간변환 관점을 알아보자.  

## column space 관점의 행렬 곱
column vector는 이름의 뜻 그대로 각각의 column을 하나의 벡터로 보는 관점이다.  
column space 관점으로 행렬 곱을 보면  
<br/>
$$\begin{pmatrix}a_{1} & b_{1}\\ a_{2} & b_{2} \end{pmatrix}\times\begin{pmatrix}x\\ y\end{pmatrix}=\begin{pmatrix}a_{1}\\ a_{2}\end{pmatrix}x+\begin{pmatrix}b_{1}\\ b_{2}\end{pmatrix}y$$ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (1)  
<br/>
이 관점에서 matrix와 vector의 행렬곱은 ``basis``의 ``선형 조합``이 된다.  
$$\begin{pmatrix}a_{1}\\ a_{2}\end{pmatrix}$$를 a basis로, $$\begin{pmatrix}b_{1}\\ b_{2}\end{pmatrix}$$를 b basis로 보자.   
다음 섹션에서 원래의 공간에서의 좌표 x,y를 변환된 ``basis a,b``를 기반으로 x,y좌표를 표시하는 ``공간변환``으로 이해해보자.    

## 행렬곱의 공간변환 관점
위의 $$\begin{pmatrix} x \\ y \end{pmatrix}$$벡터는 [1,0] basis와 [0,1]의 basis로 좌표를 표시했다고 볼 수 있다.    
<br/>
$$\begin{pmatrix} x \\ y \end{pmatrix}=\begin{pmatrix} 1&0 \\ 0&1 \end{pmatrix}\begin{pmatrix} x \\ y \end{pmatrix}$$  
<br/>
수식으로 $$\begin{pmatrix} 1 \\ 0 \end{pmatrix}x+\begin{pmatrix} 0 \\ 1 \end{pmatrix}y$$ 표현하면 좀더 이해하기 쉬워진다.  
이 관점에서 ``식(1)``을 다시 보면,  
basis a * x 와 basis b * y 로 판단이 가능하고,   
이 표현은 결국 원래의 [1,0] 와 [0,1]의 basis로 표현된 x,y 좌표가
basis a, basis b 의 공간에서 x,y좌표 표현으로 ``변환``이 이루어 졌다고 생각 할 수 있다.  
행렬의 곱셈연산을 공간 변환 후 변환 된 basis로 같은 좌표 x,y를 표현하는 것을 수식적으로 확인이 가능하다.    

## 결과값
곱셈의 결과값의 의미를 알아보자.  
  
>모든 vector는 I matrix의 basis를 가진 공간에서 좌표를 표현 한 것이다.
>이 vector에 matrix를 좌곱하면 matrix의 각 column 을 basis로 하여 vector좌표를 표시하는 것으로 이해할 수 있다.
  
곱셈을 마친 후, 다시 vector형태로 돌아간다.  
이는 위의 ``모든 vector는 I matrix의 basis를 가진 공간에서 좌표를 표현 한 것이다.``의미로 볼 수 있다.  
  
따라서, 
공간변환(곱셈) 이후 나온 값은 matrix의 basis로 좌표 x,y를 나타낸 것을,  
[1,0], [0,1] (즉 I matrix) 를 basis로한 공간에서 표시한 것이라 볼 수 있다.    
<br/>
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/f867afc9-6faa-4e34-aefd-f2f56607e157)  
<br/>
위의 그림에서 원래 공간의 1,1좌표는 $$\begin{pmatrix}2 & 0\\ 0 & 2\end{pmatrix}$$ 공간변화를 거친 후의 공간변화 이후의 좌표계에서는  1,1이다.  
원래의 I matrix에서이 공간변환 된 좌표계의 값을 바라 볼 때, 값은 [2,2] 가 되고, 이는
$$\begin{pmatrix}
2 & 0 \\ 
0 & 2 
\end{pmatrix} \times \begin{pmatrix}
1 \\ 
1 
\end{pmatrix}=\begin{pmatrix}
2 \\ 
2 
\end{pmatrix}$$
로 표현된다.  
