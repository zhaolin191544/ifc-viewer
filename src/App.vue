<template>
  <div class="relative w-full h-screen flex justify-center items-center bg-gray-100">
    
    <input
      type="file"
      accept=".ifc"
      @change="loadIfcFile"
      class="absolute top-4 left-4 z-10 p-2 bg-white rounded shadow-md text-sm file:mr-2 file:py-1 file:px-2 file:rounded file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
    />

    <div 
      ref="canvasContainer" 
      class="w-4/5 h-4/5 rounded-lg shadow-xl"
    >
      </div>
  
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'; 
import { IFCLoader } from 'web-ifc-three/IFCLoader'; 

const canvasContainer = ref<HTMLDivElement | null>(null);

// 将核心 Three.js 对象提升到 setup 作用域
let scene: THREE.Scene;
let renderer: THREE.WebGLRenderer;
let camera: THREE.PerspectiveCamera;
let ifcLoader: IFCLoader;
let controls: OrbitControls; // 提升 controls

// 存储 resize-handler 以便后续移除
let resizeHandler: () => void;

// ****************************
// ***** 修复 2: 自动对焦功能 *****
// ****************************
/**
 * 计算模型的边界盒并自动将相机对准模型
 */
function frameModel(model: THREE.Object3D) {
  // 1. 计算模型的边界盒
  const bbox = new THREE.Box3().setFromObject(model);
  
  // 2. 获取边界盒的中心点
  const center = bbox.getCenter(new THREE.Vector3());
  
  // 3. 获取边界盒的尺寸
  const size = bbox.getSize(new THREE.Vector3());
  
  // 4. 计算到相机的合适距离
  const maxDim = Math.max(size.x, size.y, size.z);
  const fov = camera.fov * (Math.PI / 180);
  let cameraZ = Math.abs(maxDim / 2 / Math.tan(fov / 2));
  
  cameraZ *= 1.5; // 留出一些边距 (1.5x)
  
  // 5. 设置相机位置 (从中心点 + Z 轴偏移)
  // (我们假设模型是 Y 轴朝上, 所以我们在 Z 轴上后退)
  camera.position.set(center.x, center.y, center.z + cameraZ);
  
  // 6. 更新控制器（OrbitControls）的目标
  controls.target.copy(center);
  controls.update();
}

onMounted(() => {
  if (!canvasContainer.value) return;

  const container = canvasContainer.value;

  // 1. 场景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xeeeeee);

  // 2. 相机 (aspect 暂时给 1, resizeHandler 会修复它)
  camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000);
  camera.position.set(0, 0, 10); // 初始位置不重要了，frameModel 会覆盖

  // 3. 渲染器 (先添加，后设置大小)
  renderer = new THREE.WebGLRenderer({ antialias: true });
  container.appendChild(renderer.domElement); 

  // 4. 光源
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(1, 1, 1);
  scene.add(directionalLight);

  // 5. 控制器 (赋值给 'controls' 变量)
  controls = new OrbitControls(camera, renderer.domElement);

  // 6. IFC Loader
  ifcLoader = new IFCLoader();
  ifcLoader.ifcManager.setWasmPath('/'); 

  // 7. 渲染循环
  const animate = () => {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  };
  animate();
  
  // **********************************
  // ***** 修复 1: 窗口尺寸调整逻辑 *****
  // **********************************
  resizeHandler = () => {
    if (container) {
      const width = container.clientWidth;
      const height = container.clientHeight;
      
      // 检查宽高是否有效，防止为 0
      if (width > 0 && height > 0) {
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        renderer.setSize(width, height);
      }
    }
  };
  
  window.addEventListener('resize', resizeHandler);
  
  // 在 onMounted 末尾手动调用一次，以设置正确的初始尺寸
  setTimeout(resizeHandler, 0);
});

// 在组件卸载时移除监听器
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

// 9. 文件加载函数
const loadIfcFile = (event: Event) => {
  const input = event.target as HTMLInputElement;

  const file = input.files?.[0];
  if (!file) {
    return;
  }
  console.log("开始加载文件:", file.name);

  const url = URL.createObjectURL(file);

  try {
    ifcLoader.load(
      url,
      (model) => {
        console.log("模型加载成功!", model);
        const ifcModel = model as THREE.Object3D;
        scene.add(ifcModel);
        
        // ****************************
        // ***** 修复 2: 调用对焦 *****
        // ****************************
        frameModel(ifcModel);
        
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