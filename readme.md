![GLTFSJSX](https://i.imgur.com/ZB4uUaz.png)

[![Discord Shield](https://discordapp.com/api/guilds/740090768164651008/widget.png?style=shield)](https://discord.gg/ZZjjNvJ)

Turns GLTF assets into dynamic, re-usable [react-three-fiber](https://github.com/pmndrs/react-three-fiber) JSX components. See it in action here: https://twitter.com/0xca0a/status/1224335000755146753

The usual GLTF workflow is cumbersome: objects can only be found by traversal, changes are made by mutation, making contents conditional is hard. 

With gltfjsx the full graph is declarative and immutable. It creates look-up tables of all the objects and materials inside your asset, it will not touch or modify your files in any way.

## Usage

```bash
$ npx @react-three/gltfjsx model.gltf [Model.js] [options]
```

### Options
```bash
  --types, -t         Adds Typescript definitions                [boolean]
  --verbose, -v       Verbose output w/ names and empty groups  [boolean] [default: false]
  --precision, -p     Number of fractional digits               [number ] [default: 2]
  --draco, -d         Draco binaries                            [string ] [default from Google CDN]
  --root, -r          Sets directory from which .gltf is served [string ]
  --help              Show help                                 [boolean]
  --version           Show version number                       [boolean]
```

### Requirements

- The GLTF file has to be present in your projects `/public` folder
- [react-three-fiber](https://github.com/pmndrs/react-three-fiber) version 5 or later
- [@react-three/drei](https://github.com/pmndrs/drei) version 2 or later

### A typical result will look like this

```jsx
/*
auto-generated by: https://github.com/react-spring/gltfjsx
author: abcdef (https://sketchfab.com/abcdef)
license: CC-BY-4.0 (http://creativecommons.org/licenses/by/4.0/)
source: https://sketchfab.com/models/...
title: Model
*/

import React from 'react'
import { useLoader } from 'react-three-fiber'
import { useGLTF } from '@react-three/drei/useGLTF'

function Model(props) {
  const { nodes, materials } = useGLTF('/model.gltf')
  return (
    <group {...props} dispose={null}>
      <group name="Camera" position={[10, 0, 50]} rotation={[Math.PI / 2, 0, 0]}>
        <primitive object={nodes.Camera_Orientation} />
      </group>
      <group name="Sun" position={[100, 50, 100]} rotation={[-Math.PI / 2, 0, 0]}>
        <primitive object={nodes.Sun_Orientation} />
      </group>
      <group name="Cube">
        <mesh material={materials.base} geometry={nodes.Cube_003_0.geometry} />
        <mesh material={materials.inner} geometry={nodes.Cube_003_1.geometry} />
      </group>
    </group>
  )
}
```

This component is async and must be wrapped into `<Suspense>` for fallbacks:

```jsx
import React, { Suspense } from 'react'

function App() {
  return (
    <Suspense fallback={null}>
      <Model />
    </Suspense>
```

### Draco compression

You don't need to do anything if your models are draco compressed, since `useGLTF` defaults to a draco CDN (`https://www.gstatic.com/draco/v1/decoders/`). By adding the `--draco` flag you can refer to [local binaries](https://github.com/mrdoob/three.js/tree/dev/examples/js/libs/draco/gltf) which must reside in your /public folder.

### Animation

If your GLTF contains animations it will add a THREE.AnimationMixer to your component and extract the clips:

```jsx
const actions = useRef()
const [mixer] = useState(() => new THREE.AnimationMixer())
useFrame((state, delta) => mixer.update(delta))
useEffect(() => {
  actions.current = { storkFly_B_: mixer.clipAction(gltf.animations[0]) }
  return () => gltf.animations.forEach((clip) => mixer.uncacheClip(clip))
}, [])
```

If you want to play an animation you can do so at any time:

```jsx
<mesh onClick={(e) => actions.current.storkFly_B_.play()} />
```

### Preload

The asset will be preloaded by default, this makes it quicker to load and reduces time-to-paint. Remove the preloader if you don't need it.

```jsx
function Model(props) {
  const { nodes, materials } = useGLTF('/model.gltf')
  ...
}

useGLTF.preload('/model.gltf')
```

### Types

Add the `--types` flag and your GLTF will be typesafe.

```tsx
type GLTFResult = GLTF & {
  nodes: {
    cube1: THREE.Mesh
    cube2: THREE.Mesh
  }
  materials: {
    base: THREE.MeshStandardMaterial
    inner: THREE.MeshStandardMaterial
  }
}

function Model(props: JSX.IntrinsicElements['group']) {
  const { nodes, materials } = useGLTF<GLTFResult>('/model.gltf')
```