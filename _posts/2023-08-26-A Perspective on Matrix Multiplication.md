---
title: A Perspective on Matrix Multiplication
auther: ckh
date: 2023-08-26 22:41:28 +0800
categories: [Math, LinearAlgebra]
tags: [LinearAlgebra]    
use_math: true
---

## Introduction
행렬곱을 basis의 공간변환으로 표현하는 경우가 많다.
행렬 곱셈의 직관적인 이해를 위해 ``column space``의 관점을 알아보자.


### Column space의 정의
<br/>  
>The column space of a matrix is the set of all possible linear combinations of its column vectors  
<br/>  
  
해석하면 열벡터(column vectors)의 모든 선형 조합이다.  
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/d7c0bd5d-5156-4fc2-8d4d-f1c3d992c106)  

이와 같은 행렬이 있다면, 동그라미 쳐진 4개의 열벡터의 모든 결합이 열벡터 공간이다.  
이제 행렬 곱이 어떻게 표현되는지 확인해보자.  
  
## column space 관점의 행렬 곱  
column space 관점으로 행렬 곱은 아래와 같다.    
<br/>  
$$\begin{pmatrix}a_{1} & b_{1}\\ a_{2} & b_{2} \end{pmatrix}\times\begin{pmatrix}x\\ y\end{pmatrix}=\begin{pmatrix}a_{1}\\ a_{2}\end{pmatrix}x+\begin{pmatrix}b_{1}\\ b_{2}\end{pmatrix}y$$ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (1)  
<br/>  
  
$$\begin{pmatrix}a_{1}\\ a_{2}\end{pmatrix}$$를 a basis로, $$\begin{pmatrix}b_{1}\\ b_{2}\end{pmatrix}$$를 b basis로 보자.   
이 관점에서 matrix와 vector의 행렬곱은 ``basis``의 ``선형 조합``이 된다. (basis는 여기서 column vector이다)  
이를 column space에서 직접 확인해보자.  
  
## 행렬곱의 공간변환 관점
$$\begin{pmatrix} x \\ y \end{pmatrix}$$ 벡터는 [1,0] basis와 [0,1]의 basis로 좌표를 표시했다고 볼 수 있다.    
<br/>  
$$\begin{pmatrix} x \\ y \end{pmatrix}=\begin{pmatrix} 1&0 \\ 0&1 \end{pmatrix}\begin{pmatrix} x \\ y \end{pmatrix}=\begin{pmatrix} 1 \\ 0 \end{pmatrix}x+\begin{pmatrix} 0 \\ 1 \end{pmatrix}y$$  
<br/>  
이 관점에서 ``식(1)``을 다시 보면,  
column vector a * x + column vector b * y 이다.   
행렬곱은  
basis a, basis b 의 공간에서 x,y 좌표가  
basis [1,0], basis [0,1] 좌표 공간에서 가지는 좌표를 보여준다.  

## 결과값
곱셈의 결과값의 의미를 알아보자.  
  
>모든 vector는 I matrix의 basis를 가진 공간에서 좌표를 표현 한 것이다.
>이 vector에 matrix를 좌곱하면 matrix의 각 column 을 basis로 하여 표현한 vector좌표 값을 I matrix를 basis로 가지는 좌표계에서 표현한다.  
  
행렬 곱 이후 나온 값은 matrix의 basis로 좌표 x,y를 나타낸 것을,  
[1,0], [0,1] (즉 I matrix) 를 basis로한 공간에서 표시한 것이라 볼 수 있다.    


