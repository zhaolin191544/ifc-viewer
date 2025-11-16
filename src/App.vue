<template>
  <div class="min-h-screen flex flex-col bg-slate-100">
    <header class="w-full bg-white shadow-sm">
      <div class="max-w-6xl mx-auto flex flex-wrap items-center justify-between gap-4 px-6 py-5">
        <h1 class="text-xl font-semibold text-slate-700">IFC Viewer</h1>
        <label class="inline-flex items-center gap-3 cursor-pointer">
          <span
            class="px-4 py-2 rounded-lg bg-blue-600 text-white text-sm font-medium shadow-sm transition-colors hover:bg-blue-500"
          >
            选择 IFC 文件
          </span>
          <input
            type="file"
            accept=".ifc"
            @change="loadIfcFile"
            class="sr-only"
          />
        </label>
      </div>
    </header>

    <main class="flex-1 flex items-center justify-center px-6 py-10">
      <div
        ref="canvasContainer"
        class="relative w-[80vw] h-[80vh] max-w-full max-h-full bg-slate-300 border border-slate-200 rounded-2xl shadow-2xl overflow-hidden"
      >
        <p
          v-if="!hasModelLoaded"
          class="absolute inset-0 flex items-center justify-center text-sm text-slate-500 tracking-wide"
        >
          等待加载 IFC 模型…
        </p>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'; 
import { IFCLoader } from 'web-ifc-three/IFCLoader'; 

const canvasContainer = ref<HTMLDivElement | null>(null);
const hasModelLoaded = ref(false);

let scene: THREE.Scene;
let renderer: THREE.WebGLRenderer;
let camera: THREE.PerspectiveCamera;
let ifcLoader: IFCLoader;
let controls: OrbitControls; 
let currentModel: THREE.Object3D | null = null;

let resizeHandler: () => void;

function frameModel(model: THREE.Object3D) {
  const bbox = new THREE.Box3().setFromObject(model);
  
  const center = bbox.getCenter(new THREE.Vector3());
  
  const size = bbox.getSize(new THREE.Vector3());
  
  const maxDim = Math.max(size.x, size.y, size.z);
  const fov = camera.fov * (Math.PI / 180);
  let cameraZ = Math.abs(maxDim / 2 / Math.tan(fov / 2));
  
  cameraZ *= 1.5;
  
  camera.position.set(center.x, center.y, center.z + cameraZ);
  
  controls.target.copy(center);
  controls.update();
}

onMounted(() => {
  if (!canvasContainer.value) return;

  const container = canvasContainer.value;

  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xd1d5db);

  camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000);
  camera.position.set(0, 0, 10); 

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setClearColor(0xd1d5db);
  container.appendChild(renderer.domElement); 

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(1, 1, 1);
  scene.add(directionalLight);

  controls = new OrbitControls(camera, renderer.domElement);

  ifcLoader = new IFCLoader();
  ifcLoader.ifcManager.setWasmPath('/'); 

  const animate = () => {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  };
  animate();
  
  resizeHandler = () => {
    if (container) {
      const width = container.clientWidth;
      const height = container.clientHeight;
      
      if (width > 0 && height > 0) {
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        renderer.setSize(width, height);
      }
    }
  };
  
  window.addEventListener('resize', resizeHandler);
  
  setTimeout(resizeHandler, 0);
});

onBeforeUnmount(() => {
  if (renderer) {
    renderer.dispose();
  }
  if (ifcLoader) {
    ifcLoader.ifcManager.dispose();
  }
  if (resizeHandler) {
    window.removeEventListener('resize', resizeHandler);
  }
});

const loadIfcFile = (event: Event) => {
  const input = event.target as HTMLInputElement;

  const file = input.files?.[0];
  if (!file) {
    return;
  }
  console.log("开始加载文件:", file.name);

  const url = URL.createObjectURL(file);

  hasModelLoaded.value = false;

  try {
    ifcLoader.load(
      url,
      (model) => {
        console.log("模型加载成功!", model);
        if (currentModel) {
          scene.remove(currentModel);
        }

        currentModel = model as THREE.Object3D;
        scene.add(currentModel);

        frameModel(currentModel);

        hasModelLoaded.value = true;

        URL.revokeObjectURL(url);
      },
      undefined,
      (error) => {
        console.error('加载 IFC 文件时出错:', error);
        URL.revokeObjectURL(url);
      }
    );
  } catch (error) {
    console.error('调用 .load() 时出错:', error);
    URL.revokeObjectURL(url);
  }
};
</script>