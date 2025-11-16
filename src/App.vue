<template>
  <div class="min-h-screen bg-slate-900 text-slate-100 flex flex-col">
    <!-- 顶部导航栏 -->
    <header
      class="w-full border-b border-slate-800 bg-slate-900/80 backdrop-blur z-20"
    >
      <div
        class="max-w-6xl mx-auto flex flex-wrap items-center justify-between gap-4 px-6 py-4"
      >
        <div class="flex items-center gap-3">
          <!-- 简单 Logo 圆点 -->
          <div
            class="h-9 w-9 rounded-2xl bg-gradient-to-br from-blue-500 to-cyan-400 flex items-center justify-center shadow-lg shadow-blue-500/30"
          >
            <span class="text-xs font-bold tracking-tight">IFC</span>
          </div>
          <div>
            <h1 class="text-lg font-semibold tracking-tight">
              IFC Viewer
            </h1>
            <p class="text-xs text-slate-400">
              交互式三维模型浏览与结构检查
            </p>
          </div>
        </div>

        <!-- 右侧操作区 -->
        <div class="flex items-center gap-3">
          <!-- 文件信息 -->
          <div
            v-if="selectedFileName"
            class="hidden md:flex items-center gap-2 px-3 py-1.5 rounded-full bg-slate-800/80 border border-slate-700 text-xs"
          >
            <svg
              class="w-3.5 h-3.5 text-blue-400"
              viewBox="0 0 24 24"
              fill="none"
            >
              <path
                d="M14 3H6a2 2 0 0 0-2 2v14.5A1.5 1.5 0 0 0 5.5 21h13A1.5 1.5 0 0 0 20 19.5V9l-6-6Z"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linejoin="round"
              />
              <path
                d="M14 3v4a2 2 0 0 0 2 2h4"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <span class="max-w-[180px] truncate">
              {{ selectedFileName }}
            </span>
          </div>

          <!-- 顶部工具按钮：重置视角 -->
          <button
            type="button"
            @click="resetView"
            class="hidden sm:inline-flex items-center gap-1.5 text-xs px-3 py-1.5 rounded-full border border-slate-700 bg-slate-900/60 hover:border-blue-500 hover:bg-slate-900/90 transition-all"
          >
            <svg
              class="w-3.5 h-3.5"
              viewBox="0 0 24 24"
              fill="none"
            >
              <path
                d="M4 4v6h6"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path
                d="M20 20v-6h-6"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path
                d="M5 13a7 7 0 0 0 11.9 4.95L20 15"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path
                d="M19 11A7 7 0 0 0 7.1 6.07L4 9"
                stroke="currentColor"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <span>重置视角</span>
          </button>

          <!-- 选择 IFC 文件 -->
          <label
            class="inline-flex items-center gap-2 cursor-pointer px-4 py-2 rounded-full bg-gradient-to-r from-blue-500 to-cyan-400 text-xs font-medium shadow-lg shadow-blue-500/30 hover:from-blue-400 hover:to-cyan-300 transition-all"
          >
            <svg
              class="w-4 h-4"
              viewBox="0 0 24 24"
              fill="none"
            >
              <path
                d="M12 3v12m0 0 4-4m-4 4-4-4M5 17v1a3 3 0 0 0 3 3h8a3 3 0 0 0 3-3v-1"
                stroke="currentColor"
                stroke-width="1.6"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <span>选择 IFC 文件</span>
            <input
              type="file"
              accept=".ifc"
              @change="loadIfcFile"
              class="sr-only"
            />
          </label>
        </div>
      </div>
    </header>

    <!-- 主体区域 -->
    <main class="flex-1">
      <div
        class="max-w-6xl mx-auto px-6 py-6 lg:py-8 grid grid-cols-1 lg:grid-cols-[260px,minmax(0,1.4fr),260px] gap-5 lg:gap-6"
      >
        <!-- 左侧信息面板 -->
        <section
          class="hidden lg:flex flex-col gap-4 bg-slate-900/70 border border-slate-800 rounded-2xl p-4 shadow-xl shadow-black/40"
        >
          <h2 class="text-xs font-semibold text-slate-300 tracking-wide">
            模型信息
          </h2>
          <div class="space-y-3 text-xs text-slate-400">
            <p v-if="selectedFileName">
              <span class="text-slate-500">文件名：</span>
              <span class="break-all text-slate-200">
                {{ selectedFileName }}
              </span>
            </p>
            <p v-if="modelStats.elementCount != null">
              <span class="text-slate-500">节点数量：</span>
              <span class="text-slate-200">
                {{ modelStats.elementCount }}
              </span>
            </p>
            <p v-if="modelStats.levelCount != null">
              <span class="text-slate-500">楼层数量：</span>
              <span class="text-slate-200">
                {{ modelStats.levelCount }}
              </span>
            </p>

            <div class="mt-4 border-t border-slate-800 pt-3">
              <p class="text-[11px] text-slate-500 mb-1">
                操作提示
              </p>
              <ul class="space-y-1">
                <li>• 鼠标左键：旋转模型</li>
                <li>• 鼠标滚轮：缩放</li>
                <li>• 鼠标右键：平移视图</li>
                <li>• 单击构件：选中并高亮</li>
              </ul>
            </div>
          </div>

          <!-- 选中构件信息 -->
          <div class="mt-4 border-t border-slate-800 pt-3 space-y-2">
            <p class="text-[11px] text-slate-500">
              选中构件
            </p>
            <div
              v-if="selectedElementId != null"
              class="rounded-xl border border-slate-800 bg-slate-900/80 px-3 py-2.5 space-y-1 text-xs"
            >
              <p>
                <span class="text-slate-500">ID：</span>
                <span class="text-slate-200">{{ selectedElementId }}</span>
              </p>
              <p v-if="selectedElementType">
                <span class="text-slate-500">类型：</span>
                <span class="text-slate-200">{{ selectedElementType }}</span>
              </p>
              <p v-if="selectedElementName">
                <span class="text-slate-500">名称：</span>
                <span class="text-slate-200">{{ selectedElementName }}</span>
              </p>
            </div>
            <p
              v-else
              class="text-[11px] text-slate-500"
            >
              暂无选中构件，单击模型即可选中。
            </p>
          </div>
        </section>

        <!-- 中间 3D 视图 -->
        <section
          class="relative bg-gradient-to-br from-slate-900 to-slate-950 border border-slate-800 rounded-3xl shadow-2xl shadow-black/50 overflow-hidden"
        >
          <!-- 顶部小标签 -->
          <div
            class="absolute left-4 top-4 z-10 inline-flex items-center gap-2 px-3 py-1.5 rounded-full bg-slate-900/80 border border-slate-700 text-[11px] text-slate-300"
          >
            <span
              class="inline-flex h-1.5 w-1.5 rounded-full"
              :class="hasModelLoaded ? 'bg-emerald-400' : 'bg-slate-500'"
            ></span>
            <span>
              {{ hasModelLoaded ? '模型已加载' : '等待加载 IFC 模型' }}
            </span>
          </div>

          <!-- 右下角视图控制按钮 -->
          <div
            class="absolute right-4 bottom-4 z-10 flex flex-col gap-2"
          >
            <button
              type="button"
              @click="setView('top')"
              class="px-3 py-1.5 text-[11px] rounded-full border border-slate-700 bg-slate-900/80 hover:border-blue-400 hover:bg-slate-900 transition-all"
            >
              Top
            </button>
            <button
              type="button"
              @click="setView('front')"
              class="px-3 py-1.5 text-[11px] rounded-full border border-slate-700 bg-slate-900/80 hover:border-blue-400 hover:bg-slate-900 transition-all"
            >
              Front
            </button>
            <button
              type="button"
              @click="setView('iso')"
              class="px-3 py-1.5 text-[11px] rounded-full border border-slate-700 bg-slate-900/80 hover:border-blue-400 hover:bg-slate-900 transition-all"
            >
              ISO
            </button>
          </div>

          <!-- 3D Canvas 容器 -->
          <div
            ref="canvasContainer"
            class="w-full h-[60vh] md:h-[70vh] lg:h-[72vh]"
          ></div>

          <!-- 初始提示 / 加载遮罩 -->
          <div
            v-if="!hasModelLoaded || isLoading"
            class="absolute inset-0 flex flex-col items-center justify-center bg-slate-950/80 backdrop-blur-sm text-sm text-slate-300"
          >
            <svg
              class="w-7 h-7 mb-4 animate-spin text-blue-400"
              viewBox="0 0 24 24"
              fill="none"
            >
              <circle
                class="opacity-20"
                cx="12"
                cy="12"
                r="9"
                stroke="currentColor"
                stroke-width="2"
              />
              <path
                d="M21 12a9 9 0 0 0-9-9"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />
            </svg>
            <p class="tracking-wide mb-1">
              {{ isLoading ? '正在解析 IFC 模型…' : '请选择左上角的 IFC 文件进行加载' }}
            </p>
            <p class="text-xs text-slate-500">
              大文件加载时间可能稍长
            </p>
          </div>
        </section>

        <!-- 右侧控制面板 -->
        <section
          class="hidden lg:flex flex-col gap-4 bg-slate-900/70 border border-slate-800 rounded-2xl p-4 shadow-xl shadow-black/40"
        >
          <h2 class="text-xs font-semibold text-slate-300 tracking-wide">
            视图与显示
          </h2>

          <div class="space-y-3 text-xs text-slate-300">
            <label class="flex items-center justify-between gap-2">
              <span>显示线框</span>
              <input
                type="checkbox"
                v-model="options.showWireframe"
                class="h-3.5 w-3.5 rounded border-slate-600 bg-slate-900"
              />
            </label>
            <label class="flex items-center justify-between gap-2">
              <span>楼层剖切（水平）</span>
              <input
                type="checkbox"
                v-model="options.enableSection"
                class="h-3.5 w-3.5 rounded border-slate-600 bg-slate-900"
              />
            </label>
            <label class="flex items-center justify-between gap-2">
              <span>显示坐标轴</span>
              <input
                type="checkbox"
                v-model="options.showAxes"
                class="h-3.5 w-3.5 rounded border-slate-600 bg-slate-900"
              />
            </label>
          </div>

          <div class="mt-4 border-t border-slate-800 pt-3 space-y-2">
            <p class="text-[11px] text-slate-500">
              快捷操作
            </p>
            <div class="flex flex-wrap gap-2">
              <button
                type="button"
                @click="focusSelection"
                class="px-3 py-1.5 text-[11px] rounded-full border border-slate-700 bg-slate-900/60 hover:border-blue-500"
              >
                聚焦选中构件
              </button>
              <button
                type="button"
                @click="toggleXRay"
                class="px-3 py-1.5 text-[11px] rounded-full border border-slate-700 bg-slate-900/60 hover:border-blue-500"
              >
                X-Ray 模式
              </button>
            </div>
          </div>
        </section>
      </div>
    </main>

    <!-- 底部状态条 -->
    <footer
      class="border-t border-slate-800 bg-slate-900/90 backdrop-blur text-[11px] text-slate-400"
    >
      <div
        class="max-w-6xl mx-auto px-6 py-2 flex items-center justify-between gap-3"
      >
        <div class="flex items-center gap-2">
          <span class="text-slate-500">状态：</span>
          <span>
            {{
              isLoading
                ? '模型加载中…'
                : hasModelLoaded
                ? '就绪'
                : '等待选择 IFC 文件'
            }}
          </span>
        </div>
        <div class="hidden sm:flex items-center gap-2">
          <span class="text-slate-500">相机：</span>
          <span>{{ cameraInfo }}</span>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  reactive,
  onMounted,
  onBeforeUnmount,
  watch,
} from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { IFCLoader } from 'web-ifc-three/IFCLoader';

const canvasContainer = ref<HTMLDivElement | null>(null);

const hasModelLoaded = ref(false);
const isLoading = ref(false);
const selectedFileName = ref<string | null>(null);
const cameraInfo = ref('-');

const modelStats = reactive({
  elementCount: null as number | null,
  levelCount: null as number | null,
});

const options = reactive({
  showWireframe: false,
  enableSection: false,
  showAxes: true,
});

// 选中构件信息
const selectedElementId = ref<number | null>(null);
const selectedElementName = ref<string | null>(null);
const selectedElementType = ref<string | null>(null);

let scene: THREE.Scene | null = null;
let renderer: THREE.WebGLRenderer | null = null;
let camera: THREE.PerspectiveCamera | null = null;
let ifcLoader: IFCLoader | null = null;
let controls: OrbitControls | null = null;
let currentModel: THREE.Object3D | null = null;
let currentModelID: number | null = null;
let axesHelper: THREE.AxesHelper | null = null;
let axesOffset = new THREE.Vector3();

// --- 新增：坐标轴专用场景和相机 ---
let axesScene: THREE.Scene | null = null;
let axesCamera: THREE.OrthographicCamera | null = null;
// --- 结束新增 ---

let resizeHandler: () => void;
let pointerHandler: (e: MouseEvent) => void;

// 选中高亮
let raycaster: THREE.Raycaster;
const mouse = new THREE.Vector2();
let highlightMaterial: THREE.MeshLambertMaterial;
let highlightSubset: THREE.Mesh | null = null;

// 剖切平面
let clippingPlane: THREE.Plane | null = null;

// X-Ray 状态
const xRayEnabled = ref(false);

// 将模型框进视野
function frameModel(model: THREE.Object3D) {
  if (!camera || !controls) return;

  const bbox = new THREE.Box3().setFromObject(model);
  const center = bbox.getCenter(new THREE.Vector3());
  const size = bbox.getSize(new THREE.Vector3());
  const maxDim = Math.max(size.x, size.y, size.z);

  const fov = (camera.fov * Math.PI) / 180;
  let cameraZ = Math.abs(maxDim / 2 / Math.tan(fov / 2));
  cameraZ *= 1.5;

  camera.position.set(center.x, center.y, center.z + cameraZ);
  controls.target.copy(center);
  controls.update();
}

// 重置视角
function resetView() {
  if (highlightSubset) {
    frameModel(highlightSubset);
    return;
  }
  if (currentModel) {
    frameModel(currentModel);
  } else if (camera && controls) {
    camera.position.set(0, 0, 10);
    controls.target.set(0, 0, 0);
    controls.update();
  }
}

// 视角切换
function setView(mode: 'top' | 'front' | 'iso') {
  if (!camera || !controls) return;
  const target = controls.target.clone();
  const distance = camera.position.distanceTo(target) || 10;

  switch (mode) {
    case 'top':
      camera.position.set(target.x, target.y + distance, target.z);
      break;
    case 'front':
      camera.position.set(target.x, target.y, target.z + distance);
      break;
    case 'iso': {
      const d = distance / Math.sqrt(3);
      camera.position.set(target.x + d, target.y + d, target.z + d);
      break;
    }
  }
  camera.lookAt(target);
  controls.update();
}

// 线框模式
function applyWireframe(model: THREE.Object3D, enabled: boolean) {
  model.traverse((obj) => {
    const mesh = obj as THREE.Mesh;
    const material = (mesh as any).material;
    if (!material) return;

    const apply = (m: any) => {
      if ('wireframe' in m) m.wireframe = enabled;
    };

    if (Array.isArray(material)) {
      material.forEach(apply);
    } else {
      apply(material);
    }
  });
}

// X-Ray 模式
function applyXRay(enabled: boolean) {
  if (!scene) return;

  scene.traverse((obj) => {
    const mesh = obj as THREE.Mesh;
    const material = (mesh as any).material;
    if (!material) return;

    const apply = (m: any) => {
      if (!m) return;
      const alpha = enabled ? 0.2 : 1.0;
      m.transparent = alpha < 1;
      m.opacity = alpha;
    };

    // 高亮子集保持相对清晰
    if (highlightSubset && obj === highlightSubset) return;

    if (Array.isArray(material)) {
      material.forEach(apply);
    } else {
      apply(material);
    }
  });

  // 保证高亮子集比较清晰
  if (highlightSubset) {
    const mat = (highlightSubset as any).material;
    const setMat = (m: any) => {
      if (!m) return;
      m.transparent = true;
      m.opacity = enabled ? 0.8 : 0.6;
    };
    if (Array.isArray(mat)) mat.forEach(setMat);
    else setMat(mat);
  }
}

// 聚焦选中构件
function focusSelection() {
  if (highlightSubset) {
    frameModel(highlightSubset);
  } else if (currentModel) {
    frameModel(currentModel);
  }
}

// 切换 X-Ray 模式
function toggleXRay() {
  xRayEnabled.value = !xRayEnabled.value;
  applyXRay(xRayEnabled.value);
}

// 清除选中
function clearSelection() {
  selectedElementId.value = null;
  selectedElementName.value = null;
  selectedElementType.value = null;

  if (ifcLoader && currentModelID != null && highlightMaterial && scene) {
    try {
      ifcLoader.ifcManager.removeSubset(currentModelID, highlightMaterial, scene);
    } catch (e) {
      console.warn('移除高亮子集失败:', e);
    }
  }
  highlightSubset = null;
}

// 选中构件
async function selectElement(modelID: number, expressID: number) {
  if (!ifcLoader || !scene) return;

  selectedElementId.value = expressID;

  // 高亮子集
  try {
    highlightSubset = ifcLoader.ifcManager.createSubset({
      modelID,
      ids: [expressID],
      material: highlightMaterial,
      scene,
      removePrevious: true,
    }) as any;
  } catch (e) {
    console.warn('创建高亮子集失败:', e);
  }

  // 构件属性
  try {
    const props: any = await ifcLoader.ifcManager.getItemProperties(
      modelID,
      expressID,
      true,
    );
    selectedElementName.value = props?.Name?.value || null;
    selectedElementType.value = props?.type || props?.GlobalId?.value || null;
  } catch (e) {
    console.warn('获取构件属性失败:', e);
  }
}

// 统计模型信息（简单版：遍历空间树）
async function updateModelStats(modelID: number) {
  if (!ifcLoader) return;
  try {
    const tree: any = await ifcLoader.ifcManager.getSpatialStructure(
      modelID,
      true,
    );

    let elementCount = 0;
    let levelCount = 0;

    const traverse = (node: any) => {
      if (!node) return;
      if (node.expressID) {
        elementCount++;
      }
      if (node.type === 'IFCBUILDINGSTOREY') {
        levelCount++;
      }
      if (Array.isArray(node.children)) {
        node.children.forEach(traverse);
      }
    };

    traverse(tree);

    modelStats.elementCount = elementCount || null;
    modelStats.levelCount = levelCount || null;
  } catch (e) {
    console.warn('统计模型信息失败:', e);
  }
}

onMounted(() => {
  if (!canvasContainer.value) return;
  const container = canvasContainer.value;

  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x020617); // slate-950

  camera = new THREE.PerspectiveCamera(75, 1, 0.1, 2000);
  camera.position.set(0, 0, 10);

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setClearColor(0x020617);
  renderer.setPixelRatio(window.devicePixelRatio);
  
  // --- 修改：关闭自动清除 ---
  renderer.autoClear = false;
  // --- 结束修改 ---
  
  container.appendChild(renderer.domElement);

  // 剖切平面（默认水平，Y 向上）
  clippingPlane = new THREE.Plane(new THREE.Vector3(0, -1, 0), 0); // y = 0 以下裁剪

  // 灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(3, 5, 2);
  scene.add(directionalLight);

  // --- 修改：坐标轴设置 ---
  // 原来的代码 (已删除):
  // axesHelper = new THREE.AxesHelper(5);
  // axesHelper.visible = options.showAxes;
  // scene.add(axesHelper);

  // --- 新增：坐标轴专用场景和相机 ---
  axesScene = new THREE.Scene();
  const orthoSize = 8; // 正交相机的视口大小
  axesCamera = new THREE.OrthographicCamera(-orthoSize, orthoSize, orthoSize, -orthoSize, 0.1, 1000);
  axesScene.add(new THREE.AmbientLight(0xffffff, 1.0)); // 给坐标轴场景加个环境光
  
  axesHelper = new THREE.AxesHelper(3); // 辅助器大小
  axesHelper.visible = options.showAxes; // 初始可见性
  axesScene.add(axesHelper);
  // --- 结束新增 ---

  // 控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  // IFCLoader
  ifcLoader = new IFCLoader();
  ifcLoader.ifcManager.setWasmPath('/');

  // Raycaster & 高亮材质
  raycaster = new THREE.Raycaster();
  highlightMaterial = new THREE.MeshLambertMaterial({
    color: 0x22e1ff,
    transparent: true,
    opacity: 0.6,
    depthTest: true,
  });

  // 自适应尺寸
  resizeHandler = () => {
    if (!container || !camera || !renderer) return;
    const width = container.clientWidth;
    const height = container.clientHeight;
    if (width > 0 && height > 0) {
      camera.aspect = width / height;
      camera.updateProjectionMatrix();
      renderer.setSize(width, height, false);
      
      // 注意：axesCamera 是 1:1 的正交相机，
      // 我们渲染的视口也是 1:1 (120x120)，所以不需要更新 axesCamera 的 aspect
    }
  };

  window.addEventListener('resize', resizeHandler);
  setTimeout(resizeHandler, 0);

  // 鼠标拾取
  pointerHandler = (event: MouseEvent) => {
    if (!canvasContainer.value || !camera || !currentModel || !ifcLoader) return;

    const rect = canvasContainer.value.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObject(currentModel, true);
    if (!intersects.length) {
      clearSelection();
      return;
    }

    const intersect = intersects[0];
    if (intersect.faceIndex == null) return;

    const mesh = intersect.object as any;
    const geometry = mesh.geometry as THREE.BufferGeometry;

    try {
      const expressID = ifcLoader!.ifcManager.getExpressId(
        geometry,
        intersect.faceIndex,
      );
      const modelID = mesh.modelID ?? currentModelID;
      if (modelID == null) return;
      selectElement(modelID, expressID);
    } catch (e) {
      console.warn('拾取构件 ID 失败:', e);
    }
  };

  container.addEventListener('pointerdown', pointerHandler);

  // --- 修改：动画循环 (animate) ---
  const animate = () => {
    requestAnimationFrame(animate);
    if (!renderer || !scene || !camera || !axesScene || !axesCamera || !canvasContainer.value) {
      return;
    }

    controls?.update();

    const mainWidth = canvasContainer.value.clientWidth;
    const mainHeight = canvasContainer.value.clientHeight;

    // 0. 手动清除画布
    renderer.clear();

    // 1. 渲染主场景
    renderer.setViewport(0, 0, mainWidth, mainHeight);
    renderer.setScissor(0, 0, mainWidth, mainHeight);
    renderer.setScissorTest(true);
    renderer.render(scene, camera);

    // 2. 渲染坐标轴场景
    if (options.showAxes && controls) {
      // 同步相机旋转

    axesOffset.subVectors(camera.position, controls.target);
  // 2. 归一化（得到纯方向），并设置一个固定距离（例如 10）
    axesOffset.normalize().multiplyScalar(10);
  // 3. 将这个偏移量设置为坐标轴相机的位置
    axesCamera.position.copy(axesOffset);
  // 4. 确保坐标轴相机始终看向 (0,0,0)
    axesCamera.lookAt(0, 0, 0);

      const axesViewportSize = 120; // 120x120 像素
      const margin = 16; // 距离边缘 16px

      // 计算视口位置（右上角）
      const viewportX = mainWidth - axesViewportSize - margin;
      const viewportY = mainHeight - axesViewportSize - margin;

      // 清除深度缓冲区，确保坐标轴在最前面
      renderer.clearDepth(); 

      renderer.setViewport(viewportX, viewportY, axesViewportSize, axesViewportSize);
      renderer.setScissor(viewportX, viewportY, axesViewportSize, axesViewportSize);
      renderer.setScissorTest(true);
      
      renderer.render(axesScene, axesCamera);
    }
    
    // 3. 渲染状态
    renderer.setScissorTest(false); // 重置
    const pos = camera.position;
    cameraInfo.value = `X: ${pos.x.toFixed(1)}  Y: ${pos.y.toFixed(1)}  Z: ${pos.z.toFixed(1)}`;
  };
  animate();
  // --- 结束修改：animate ---
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
  if (pointerHandler && canvasContainer.value) {
    canvasContainer.value.removeEventListener('pointerdown', pointerHandler);
  }
});

// 选项联动：线框
watch(
  () => options.showWireframe,
  (val) => {
    if (currentModel) {
      applyWireframe(currentModel, val);
    }
  },
);

// 选项联动：坐标轴
watch(
  () => options.showAxes,
  (val) => {
    if (axesHelper) {
      axesHelper.visible = val;
    }
    // 注意：我们还在 animate 循环中加了 if (options.showAxes)
    // 这样在隐藏时可以跳过坐标轴的渲染步骤
  },
);

// 选项联动：剖切平面
watch(
  () => options.enableSection,
  (val) => {
    if (!renderer) return;
    renderer.localClippingEnabled = val;
    if (val && clippingPlane) {
      renderer.clippingPlanes = [clippingPlane];
    } else {
      renderer.clippingPlanes = [];
    }
  },
);

const loadIfcFile = (event: Event) => {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];
  if (!file) return;

  console.log('开始加载文件:', file.name);
  selectedFileName.value = file.name;

  const url = URL.createObjectURL(file);
  hasModelLoaded.value = false;
  isLoading.value = true;
  clearSelection();
  modelStats.elementCount = null;
  modelStats.levelCount = null;

  if (!ifcLoader || !scene) {
    console.error('IFCLoader 或 scene 未初始化');
    URL.revokeObjectURL(url);
    isLoading.value = false;
    return;
  }

  try {
    ifcLoader.load(
      url,
      async (model) => {
        console.log('模型加载成功!', model);

        if (currentModel) {
          scene!.remove(currentModel);
        }

        currentModel = model as THREE.Object3D;
        currentModelID = (currentModel as any).modelID ?? 0;
        scene!.add(currentModel);

        frameModel(currentModel);
        hasModelLoaded.value = true;
        isLoading.value = false;

        // 线框 & X-Ray 状态同步
        if (options.showWireframe) {
          applyWireframe(currentModel, true);
        }
        if (xRayEnabled.value) {
          applyXRay(true);
        }

        // 更新模型统计信息
        if (currentModelID != null) {
          updateModelStats(currentModelID);
        }

        URL.revokeObjectURL(url);
      },
      undefined,
      (error) => {
        console.error('加载 IFC 文件时出错:', error);
        URL.revokeObjectURL(url);
        isLoading.value = false;
      },
    );
  } catch (error) {
    console.error('调用 .load() 时出错:', error);
    URL.revokeObjectURL(url);
    isLoading.value = false;
  }
};
</script>
