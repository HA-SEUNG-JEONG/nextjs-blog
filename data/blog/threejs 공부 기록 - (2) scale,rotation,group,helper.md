---
title: threejs 공부 기록 (2)
date: '2023-08-09 01:00:00'
lastmod: ''
tags: ['threejs']
draft: false
summary: ''
---

### Rotation,Scale,Position

![xyz](/static/images/xC6Yy48.png)

```js
mesh.position.z = 1
mesh.scale.x = 2
mesh.rotation.x = Math.PI * 0.25
mesh.rotation.y = Math.PI * 1.2
```

위 그림에서 보듯이,

`position`

x값이 감소하면 왼쪽, 증가하면 오른쪽으로 이동하며,
y값이 감소하면 밑으로 증가하면 위로 올라간다.
z값이 감소하면 점점 멀어지는 것처럼 느껴지고 증가하면 다가오는 것처럼 느껴진다.

`scale`은 말 그대로 크기를 조정하는 것이다.

`rotation`은 어느 특정한 축을 기준으로 움직일 때 사용한다.

![rotate](/static/images/rotate.png)

### 객체 더 추가하기

예를 들어 다른 색상의 객체를 더 추가하고 싶을 경우에

```js
const geometry2 = new THREE.BoxGeometry(1, 1, 1)
const material2 = new THREE.MeshBasicMaterial({ color: 'green' })
const mesh2 = new THREE.Mesh(geometryGroup, materialGroup)
meshGroup.position.y = 2

scene.add(mesh2)
```

이렇게 추가할수도 있지만

`Group`을 사용하는 방법도 있다. 실제로 문서상에서도 더 효율적이라고 한다.

```js
const geometryGroup = new THREE.BoxGeometry(1, 1, 1)
const materialGroup = new THREE.MeshBasicMaterial({ color: 'green' })
const meshGroup = new THREE.Mesh(geometryGroup, materialGroup)
meshGroup.position.y = 2

group.add(mesh, meshGroup)
group.position.x = 3
scene.add(group)
```

### AxesHelper

축을 시각화해주는 Help function이다.

`X - red`, `Y - green`, `Z - blue`로 표시된다.

```js
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)
```

### Reference

https://discourse.threejs.org/t/orbit-controls-and-2d-rotation/6609
