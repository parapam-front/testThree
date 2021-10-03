<template>
  <div>
    <canvas id="container"></canvas>
  </div>
</template>

<script>
  import * as THREE from 'three'
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
  import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';

  export default {
    name: 'HelloWorld',
    props: {
      msg: String
    },
    methods: {
      gltfload() {
        const canvas = document.querySelector('#container');
        const renderer = new THREE.WebGLRenderer({canvas});
        renderer.setSize(window.innerWidth - 5, window.innerHeight - 5)

        const fov = 45;
        const aspect = window.innerWidth / window.innerHeight;  // the canvas default
        const near = 0.1;
        const far = 100;
        const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
        camera.position.z = 5

        // позволяет нам покрутиться вокруг полотна
        const controls = new OrbitControls(camera, canvas);
        controls.target.set(0, 5, 0);
        controls.update();

        const scene = new THREE.Scene();
        scene.background = new THREE.Color('#ffffff');

        function frameArea(sizeToFitOnScreen, boxSize, boxCenter, camera) {
          const halfSizeToFitOnScreen = sizeToFitOnScreen * 1;
          const halfFovY = THREE.MathUtils.degToRad(camera.fov * .5);
          const distance = halfSizeToFitOnScreen / Math.tan(halfFovY);
          // вычисляем единичный вектор, указывающий в направлении, в котором сейчас находится камера
          // в плоскости xz от центра прямоугольника
          const direction = (new THREE.Vector3())
            .subVectors(camera.position, boxCenter)
            .multiply(new THREE.Vector3(45, 0, 1))
            .normalize();

          camera.position.copy(direction.multiplyScalar(distance).add(boxCenter));

          // расстояние ближнего и дальшего конца бокса
          camera.near = boxSize / 100;
          camera.far = boxSize * 100;

          camera.updateProjectionMatrix();

          // камера направлена в центр бокса
          camera.lookAt(boxCenter.x, boxCenter.y, boxCenter.z);
        }

        // загружаем модель
        const gltfLoader = new GLTFLoader();
        gltfLoader.load('/assets/scene.gltf', (gltf) => {
          const root = gltf.scene;
          scene.add(root);

          // все файлы от корня и ниже подгружает вроде ????
          const box = new THREE.Box3().setFromObject(root);

          const boxSize = box.getSize(new THREE.Vector3()).length();
          const boxCenter = box.getCenter(new THREE.Vector3());

          // вставляем камеру на сцену
          frameArea(boxSize * 0.5, boxSize, boxCenter, camera);

          // тут обрабатываем новый размер
          controls.maxDistance = boxSize * 10;
          controls.target.copy(boxCenter);
          controls.update();
        });

        function resizeRendererToDisplaySize(renderer) {
          const canvas = renderer.domElement;
          const width = canvas.clientWidth;
          const height = canvas.clientHeight;
          const needResize = canvas.width !== width || canvas.height !== height;
          if (needResize) {
            renderer.setSize(width, height, false);
          }
          return needResize;
        }

        function render() {
          if (resizeRendererToDisplaySize(renderer)) {
            const canvas = renderer.domElement;
            camera.aspect = canvas.clientWidth / canvas.clientHeight;
            camera.updateProjectionMatrix();
          }

          renderer.render(scene, camera);

          requestAnimationFrame(render);
        }

        requestAnimationFrame(render);
      }
    },
    mounted() {
      this.gltfload()
    }
  }
</script>


<style>
  * {
    margin: 0;
  }
</style>
