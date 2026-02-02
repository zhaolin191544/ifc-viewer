# IFC 文件读取与渲染流程说明

本文档概述项目中 IFC 文件从选择、解析到渲染呈现的主要流程。

## 1. 关键依赖与加载器

- 项目使用 `web-ifc-three` 提供的 `IFCLoader` 作为 IFC 文件解析器，并与 `three` 的渲染管线结合，形成 IFC → Three.js 网格的加载通道。【F:package.json†L12-L22】【F:src/App.vue†L343-L348】
- 在组件初始化阶段创建 `IFCLoader`，并通过 `setWasmPath('/')` 指定 WebAssembly 文件的加载根路径，确保解析器可以加载 IFC 的 WASM 模块。【F:src/App.vue†L660-L664】

## 2. Three.js 场景与渲染循环初始化

- `onMounted` 中创建 `THREE.Scene`、`PerspectiveCamera`、`WebGLRenderer`，并将渲染画布挂载到 `canvasContainer` 节点上。【F:src/App.vue†L612-L651】
- 设置环境光与方向光，保证 IFC 模型有基础照明效果。【F:src/App.vue†L666-L671】
- 配置 `OrbitControls` 让用户可以旋转/缩放/平移视角。【F:src/App.vue†L684-L686】
- 自定义渲染循环 `animate`：持续更新控制器、渲染主场景，同时在右下角的子视口渲染坐标轴辅助场景。【F:src/App.vue†L747-L828】

## 3. IFC 文件读取流程

1. 用户在 UI 中选择 `.ifc` 文件后触发 `loadIfcFile` 事件处理函数。【F:src/App.vue†L92-L111】【F:src/App.vue†L835-L844】
2. 通过 `URL.createObjectURL` 生成本地临时 URL，并设置加载状态（`isLoading`、`hasModelLoaded` 等）。【F:src/App.vue†L846-L856】
3. 调用 `ifcLoader.load(url, onLoad, onProgress, onError)` 开始解析 IFC 文件：
   - 解析成功后，返回的模型对象（`model`）被添加到 `scene` 中，并保存为 `currentModel` 以供后续交互使用。【F:src/App.vue†L858-L881】
   - 调整相机视角使模型完整进入视野范围（`frameModel`）。【F:src/App.vue†L369-L386】【F:src/App.vue†L874-L875】
   - 根据当前 UI 选项（线框、X-Ray）应用渲染效果，同时触发模型统计更新逻辑。【F:src/App.vue†L880-L889】
4. 若加载失败，记录错误并退出加载状态。【F:src/App.vue†L892-L903】

## 4. 渲染显示与交互能力

- **线框显示**：遍历模型节点，将材质 `wireframe` 设为 `true/false`。【F:src/App.vue†L426-L447】
- **X-Ray 透明模式**：遍历场景内网格，动态设置材质的透明度与透明属性。【F:src/App.vue†L450-L497】
- **剖切平面**：通过 `renderer.localClippingEnabled` 与 `renderer.clippingPlanes` 控制水平剖切效果。【F:src/App.vue†L824-L833】
- **视角切换 / 重置**：通过相机位置与 `OrbitControls` 目标点更新实现视角切换与复位。【F:src/App.vue†L369-L424】

## 5. 构件选中与属性读取

- 鼠标点击画布时触发 `pointerdown`，使用 `Raycaster` 计算与模型交点。【F:src/App.vue†L707-L730】
- 通过 `ifcManager.getExpressId` 获取选中面对应的 IFC Express ID。【F:src/App.vue†L732-L741】
- 调用 `ifcManager.createSubset` 创建高亮子集并应用高亮材质；同时使用 `getItemProperties` 读取构件属性用于 UI 展示。【F:src/App.vue†L547-L575】

## 6. 模型统计信息

- 使用 `ifcManager.getSpatialStructure` 获取 IFC 空间树结构并递归统计：
  - 节点数量（含空间节点）
  - 楼层数量（`IFCBUILDINGSTOREY`）
- 统计结果展示在左侧“模型信息”面板中。【F:src/App.vue†L577-L610】【F:src/App.vue†L131-L164】

---

以上流程集中在 `src/App.vue` 的逻辑中，核心由 `IFCLoader` 解析 IFC → Three.js 网格，再由 `Three.js` 渲染管线实时绘制，并通过光照、控制器与交互逻辑实现可视化体验。【F:src/App.vue†L343-L903】
