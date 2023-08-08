---
title: "[ThreeJS] Material,Texture, Geometry"
author: "ckh"
date: "2023-08-06 10:41:18 +0800"
categories: ["ThreeJS", "JS"]
tags: ["ThreeJS"]  
---

# Rendering Objects
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/b4c9231b-77f6-4f9b-a302-2b3c871f6eae)
그림을 보면, 랜더링 되는 물체는 geometry와 Material을 가지고있고, Material은 Texture를 가진다.  
즉, ThreeJS에서 물체가 카메라에 표현되기위해, Geometry, Material, Texture 세가지가 필수적으로 필요하다고 볼 수 있다.  
이 세가지에 대해 알아보자.  
  
## Geometry
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/40c2ffc9-c883-4201-aaab-466e3cdd841e)
``3D object``의 실제 형태를 가지고 있다. 핵심적인 component로
>1. vertices
>2. indices
>3. Normals
>4. UV coordinates  
가 있다.

### vertices
위의 ``wireframe`` 형태로 표현할 때, 각 ``점`` 부분.  
```
v1: [0, 0]
v2: [1, 0]
v3: [0, 1]
v4: [1, 1]
```
### indices
삼각형을 이루는 면을 형성하는 vertices를 모아둔 데이터.
```javscript
Indices1 = [0, 1, 2, 1, 3, 2] // 2 triange
Indices2 = [0, 1, 2, 1, 3, 2, 3, 4, 2]  // 3 triangle, (셋다 유효한 삼각형인지 체크안함)
```
이런식으로, 각각의 삼각형 하나당 3개의 element를 차지하여 계속 이어나가는 형식이다.

### Normals
각 ``vertices``마다 가지고 있는 normal vector이다. 표면에 직교하여 바깥쪽을 향한다.
```
Normals: 
v1: [0, 0, 1]
v2: [0, 0, 1]
v3: [0, 0, 1]
v4: [0, 0, 1]
```

### UVs
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/796a0ee6-5ffd-404f-8f2f-de08f7c39f4f)
3D로 구성된 오브젝트는 각 triangle들을 2차원 표면에 펼쳐서, 펼쳐진 2차원에 해당하는 좌표를 가지게 된다.  
이는 추후 texture를 매핑 할 때 사용된다.

이 외에도 다른 componenet들이 있으나, 최소한의 중요한 요소는 이 4가지정도로 봐도 무방하다.

## Material
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/e85a11bb-e702-4d7c-8574-c9ea6d375b8f)
```javsascript

let geometry = new THREE.BoxGeometry(1, 1, 1);

// 1. with texture
let textureLoader = new THREE.TextureLoader();
let cubeTexture = textureLoader.load('path_to_your_image.jpg');

let material = new THREE.MeshBasicMaterial({ map: cubeTexture });

//2. without texture
let material = new THREE.MeshBasicMaterial({ color: 0xff0000 }); // Red color

let cube = new THREE.Mesh(geometry, material);
```
Material은 geometry의 triangle들, 즉 면이 있는부분과 광원(빛)과의 상호작용을 표현한 오브젝트이다.
한 마디로 ``광택``을 표현한다.
다만, 색상에 대한 값도 존재한다. 
이는 기본색으로써, 나중에 ``texture``의 색과 겹칠 경우, ``texture``의 색깔이 ``Material``의 색깔을 덮어씌운다.
즉, 광택에 대한 부분은 texture가 간섭 못하고, 색상에 대한 부분은 texture가 스티커 붙이듯 덮어쓴다.

### geometry + Material 
3D 오브젝트는 여러가지 Material이 섞여있다. 가령 의자를 생각해보면, 나무재질의 다리와 쿠션역할의 천의 material이 섞여있는 것을 알 수 있다.  
이를 해결하기위해, 하나의 geometry에 있는 triangle들은 몇번쨰 material을 적용할지 정할 수있다(``Geometry segmenting``).  
예를들어, 의자의 경우 다른 재질없이 ``천``과 ``나무``의 재질만 있다면 triangle들은 0,1 두 가지중 하나의 material을 정하여 저장된다.  
나중에 0번에 천, 1번에 나무재질이 들어가면 이에 따라 triangle들은 0번에 assgin되었으면 천의 material을, 1번에 assign되어있으면 나무의 재질을 가지는 식으로 작동된다.  
  
## Texture
```javascript
//init
let textureLoader = new THREE.TextureLoader();
let texture = textureLoader.load('path_to_your_image.jpg');

//apply it to material
let material = new THREE.MeshBasicMaterial({ map: texture });
```
``geometry``의 실재로 보이는 ``이미지``를 표현하는 Object.


