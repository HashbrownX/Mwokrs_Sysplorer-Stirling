### 一、基础参数

#### 1. **`C[Medium.nC]`**  
- **默认值**：无显式默认（继承自 `Medium.C_default`，见初始化部分）  
- **解释**：  
  这是一个“示踪物质”（trace substance）数组，用于模拟非主要组分（如污染物、添加剂）。  
  - `nC` 是介质中定义的示踪物种类数（通常为 0）。  
  - 如果 `Medium` 不包含示踪物，则此数组为空，无需设置。  
- ✅ **一般用户可忽略**。

---

#### 2. **`initialize_p`**  
- **默认值**：`not Medium.singleState`  
- **解释**：  
  - `Medium.singleState` 是介质模型的一个属性：  
    - 对于**不可压缩流体**（如液体），`singleState = true` → 压力不独立，无需初始化。  
    - 对于**可压缩气体**（如空气、氦气），`singleState = false` → 压力是独立状态变量，**需要初始值**。  
  - 所以 `not Medium.singleState` 意味着：**如果是气体，就启用压力初始化**。  
- ✅ **对斯特林发动机（气体系统）自动为 `true`，合理且通常无需修改**。

---

#### 3. **`Medium`**  
- **默认值**：`replaceable package Medium = Modelica.Media.Interfaces.PartialMedium`  
- **解释**：  
  - `PartialMedium` 是一个**抽象基类**，不能直接使用。  
  - `replaceable` 表示你必须在实例化时**替换为具体介质**，例如：  
    ```modelica
    replaceable package Medium = Modelica.Media.IdealGases.SingleGases.Helium;
    ```  
- ❗ **必须显式指定**，否则模型无法编译。

---

#### 4. **`portInDensities[nPorts]`**  
- **默认值**：无（由仿真器根据端口状态自动计算）  
- **解释**：  
  此数组用于存储每个端口处流体的密度，供内部计算使用。  
  - 用户**不需要手动设置**，由连接的流体网络自动提供。  
- ✅ **保留默认即可**。

---

#### 5. **`s[nPorts]`**  
- **默认值**：无  
- **解释**：  
  这些是用于**理想开关器件**（如阀门）的流量-压差特性曲线参数（例如多项式系数）。  
  - `SweptVolume` 本身**不含阀门**，所以此参数通常未使用。  
  - 即使有端口，也多用于高级流动建模。  
- ✅ **初学者可忽略**。

---

#### 6. **`vessel_ps_static[nPorts]`**  
- **默认值**：无  
- **解释**：  
  表示在**零流速**条件下，容器内各端口高度处的静压（考虑重力影响）。  
  - 若 `nPorts = 0`（封闭系统），此数组为空。  
  - 若有端口且考虑液位差，才需计算。  
- ✅ **斯特林发动机通常设为 0 端口，无需关心**。

---

#### 7. **`fluidLevel_max`**  
- **默认值**：`1` （单位：m）  
- **解释**：  
  容器允许的最大液位高度，主要用于**两相流或液罐模型**。  
  - `SweptVolume` 通常用于**单相气体**，此参数无效。  
- ✅ **可保留默认，不影响气体仿真**。

---

#### 8. **`vesselArea`**  
- **默认值**：`Modelica.Constants.inf`（无穷大）  
- **解释**：  
  - 在普通容器中，`vesselArea` 用于由液位反推体积。  
  - 但在 `SweptVolume` 中，**体积由活塞位置直接决定**，不需要通过面积计算。  
  - 设为无穷大是为了**禁用基于液位的体积计算逻辑**。  
- ✅ **绝对不要修改**！这是关键设计。

---

#### 9. **`regularFlow[nPorts]`**  
- **默认值**：无（由仿真器内部处理）  
- **解释**：  
  布尔标志，用于区分“规则流动”与“奇异流动”（如零流量附近），辅助数值正则化。  
- ✅ **自动处理，无需设置**。

---

#### 10. **`inFlow[nPorts]`**  
- **默认值**：无  
- **解释**：  
  指示每个端口当前是流入（`true`）还是流出（`false`），用于方向敏感的压损计算。  
- ✅ **由仿真器自动判断，用户无需干预**。

---

#### 11. **`pistonCrossArea`**  
- **默认值**：**无**（必须由用户指定）  
- **解释**：  
  活塞横截面积 \( A \)，单位 m²。  
  - 用于计算：  
    - 容积：\( V = A \cdot s + V_0 \)  
    - 力：\( F = (p - p_{\text{amb}}) \cdot A \)  
- ❗ **必须设置**！否则模型无法运行。

---

#### 12. **`clearance`**  
- **默认值**：**无**（必须由用户指定）  
- **解释**：  
  余隙容积（dead volume），即活塞在上止点（`s = 0`）时腔室剩余体积。  
  - 防止容积为零（避免压力发散）。  
  - 典型值为总扫掠容积的 5%~20%。  
- ❗ **必须设置**！

---

### 二、自定义参数（Custom Parameters）

#### 13. **`heatTransfer`**  
- **默认值**：  
  ```modelica
  heatTransfer(
    surfaceAreas = {
      pistonCrossArea + 2*sqrt(pistonCrossArea/pi)*(flange.s + clearance/pistonCrossArea)
    }
  )
  ```
- **解释**：  
  动态计算换热面积：  
  - 底面积：`pistonCrossArea`  
  - 侧面积：周长 × 高度  
    - 周长 ≈ \( 2 \sqrt{\pi A} \)（假设圆形活塞）  
    - 高度 = 当前活塞位移 `flange.s` + 余隙对应高度 `clearance / A`  
  - 这是一个**合理的圆柱形腔室表面积近似**。  
- ✅ **建议保留**，除非你有特殊几何形状。

---

### 三、高级参数（Advanced）

#### 14. **`m_flow_nominal`**  
- **默认值**：  
  ```modelica
  if system.use_eps_Re then system.m_flow_nominal else 1e2*system.m_flow_small
  ```
- **解释**：  
  用于**数值正则化**的质量流量标称值。  
  - 如果系统启用了雷诺数判断（`use_eps_Re = true`），则取全局设定值。  
  - 否则取 `100 × m_flow_small` 作为“典型流量”。  
- ✅ **一般无需修改**，除非仿真不收敛。

---

#### 15. **`m_flow_small`**  
- **默认值**：  
  ```modelica
  if system.use_eps_Re then system.eps_m_flow*m_flow_nominal else system.m_flow_small
  ```
- **解释**：  
  定义“接近零流量”的阈值，用于平滑处理零流量奇点。  
- ✅ **保留默认即可**。

---

#### 16. **`use_Re`**  
- **默认值**：`system.use_eps_Re`  
- **解释**：  
  决定是否使用**雷诺数**（Re）来判断流态（层流/湍流）。  
  - 若为 `false`，则直接用 `m_flow_small` 判断。  
- ✅ **通常保持与系统一致**。

---

### 四、假设（Assumptions）

#### 17. **`energyDynamics`**  
- **默认值**：`system.energyDynamics`  
- **解释**：  
  控制能量方程是否包含时间导数：  
  - `SteadyState`：稳态（代数方程）  
  - `DynamicFree`：瞬态（微分方程）  
- ✅ **斯特林发动机必须为 `DynamicFree`**！检查你的 `System` 设置。

#### 18. **`massDynamics`**  
- **默认值**：`system.massDynamics`  
- **解释**：  
  同上，控制质量守恒是否动态。  
- ✅ **气体系统通常设为 `DynamicFree`**。

#### 19. **`use_HeatTransfer`**  
- **默认值**：`false`  
- **解释**：  
  是否启用壁面热传递模型。  
  - 默认关闭 → 腔室绝热。  
- ✅ **斯特林发动机必须设为 `true`**，并连接热源到 `heatPort`。

#### 20. **`HeatTransfer`**  
- **默认值**：`IdealHeatTransfer`（理想热传导，无热阻）  
- **解释**：  
  可替换的热传递模型，默认假设**壁面温度等于流体温度**（无限大传热系数）。  
- ✅ **教学模型可用；工程模型应替换为更真实的传热模型**。

#### 21. **`nPorts`**  
- **默认值**：`0`  
- **解释**：  
  流体端口数量。  
  - `SweptVolume` 常用于**封闭循环**（如斯特林），气体不进出，故为 0。  
  - 若连接回热器，可能需要设为 1 或 2。  
- ✅ **根据拓扑结构设置**。

#### 22. **`use_portsData`**  
- **默认值**：`true`  
- **解释**：  
  是否使用详细的端口数据（直径、高度、阻力系数）计算压损和动能。  
  - 若 `nPorts = 0`，此设置无效。  
- ✅ **可保留**。

#### 23. **`portsData[...]`**  
- **默认值**：动态数组（长度取决于 `nPorts` 和 `use_portsData`）  
- **解释**：  
  存储每个端口的几何和阻力数据。  
- ✅ **若启用 `use_portsData`，需提供数据；否则忽略**。

---

### 五、初始化参数（Initialization）

#### 24. **`p_start`**  
- **默认值**：`system.p_start`（通常 101325 Pa）  
- **解释**：  
  压力初始猜测值。  
- ✅ **若工作压力远离大气压，应修改**（如高压斯特林机）。

#### 25. **`use_T_start`**  
- **默认值**：`true`  
- **解释**：  
  优先使用温度而非比焓作为热力学初始条件。  
- ✅ **推荐保持 `true`，更直观**。

#### 26. **`T_start`**  
- **默认值**：`if use_T_start then system.T_start else ...`（通常 293.15 K）  
- **解释**：  
  温度初始值。  
- ✅ **斯特林热腔应设为高温（如 800 K），冷腔设为低温（如 300 K）**。

#### 27. **`h_start`**  
- **默认值**：由 `T_start` 和 `p_start` 计算得出  
- **解释**：  
  比焓初始值，仅在 `use_T_start = false` 时生效。  
- ✅ **一般不用管**。

#### 28. **`X_start[Medium.nX]`**  
- **默认值**：`Medium.X_default`（单组分气体为 `{1}`）  
- **解释**：  
  质量分数初始值。  
- ✅ **单组分工质无需修改**。

#### 29. **`C_start[Medium.nC]`**  
- **默认值**：`Medium.C_default`  
- **解释**：  
  示踪物质初始值。  
- ✅ **通常为空，可忽略**。

---

### 六、接口（Interfaces）

> 接口没有“默认值”，但说明其用途：

| 接口 | 说明 |
|------|------|
| `ports[nPorts]` | 连接流体网络（如回热器） |
| `heatPort` | 连接热源（如 heater/cooler） |
| `flange` | **最关键**！连接活塞机械运动（接收位移 `s`，输出力 `F`） |
| 其他 `RealInput` / `BooleanInput` | 用于高级端口配置（初学者通常不用） |

---

## ✅ 总结：哪些默认值你**必须关注并修改**？

| 参数 | 是否必须修改 | 建议 |
|------|------------|------|
| `Medium` | ✅ 是 | 指定工质（如 Helium） |
| `pistonCrossArea` | ✅ 是 | 根据缸径计算 |
| `clearance` | ✅ 是 | 设为扫掠容积的 5%~15% |
| `T_start` | ✅ 是 | 热腔/冷腔分别设高温/低温 |
| `use_HeatTransfer` | ✅ 是 | 设为 `true` |
| `energyDynamics` | ✅ 是 | 确保为 `DynamicFree` |

其余参数在大多数情况下可保留默认。
