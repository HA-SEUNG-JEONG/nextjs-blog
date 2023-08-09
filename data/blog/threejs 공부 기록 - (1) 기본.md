---
title: threejs 공부 기록 (1)
date: '2023-08-09 00:00:00'
lastmod: ''
tags: ['threejs']
draft: false
summary: ''
---

Three.js에서 가장 기본이 되는 객체는 scene, mesh, camera, renderer이다.

### Scene

```js
const scene = new THREE.Scene()
```

`scene`을 사용하여 렌더링할 대상과 위치를 설정할 수 있고, 또한 객체 및 조명, 카메라를 배치하는 곳을 설정할 수 있다.

### BoxGeometry

```js
const geometry = new THREE.BoxGeometry(1, 1, 1)
```

`BoxGeometry`는 클래스로서, 3개의 parameter를 갖는데 각각 `'폭', '높이', '깊이'`다.

### MeshBasicMaterial

```js
const material = new THREE.MeshBasicMaterial({ color: 'purple' })
```

가장 기본적인 material로, 프로퍼티는 선택사항이지만 대표적으로 `color`가 들어갈 수 있다.

단, `color`는 16진수로 들어갈 수 있으며, `0xffffff`가 기본값이다.

### PerspectiveCamera

```js
const camera = new THREE.PerspectiveCamera(75, aspect.width / aspect.height, 1, 2000)
```

`PerspectiveCamera( fov : Number, aspect : Number, near : Number, far : Number )`

![perspective](/static/images/perspective_view_frustum.png)

`fov`: 사람의 눈에서 수직으로 향할 때 그 시작점

`aspect`: 종횡비

`near`: 근평면

`far`: 원평면

### Camera Setting

```js
camera.position.z = 3
camera.position.x = 1
camera.position.y = 1
scene.add(camera)
```

camera에 대한 좌표를 맞추고 scene에 추가한다.

### Renderer

```js
const canvas = document.querySelector('.draw') // select the canvas element
const renderer = new THREE.WebGLRenderer({ canvas }) // add the WebGLRenderer
renderer.setSize(aspect.width, aspect.height) // Renderer Size
renderer.render(scene, camera) // display what the camera in the scecne captured
```

화면에 그리기 위해 `WebGLRenderer`를 추가하고

`setSize` 메소드를 이용하여 width,height를 설정한 후 scene과 camera를 표시한다.

### Reference

https://sbcode.net/threejs/scene-camera-renderer/
