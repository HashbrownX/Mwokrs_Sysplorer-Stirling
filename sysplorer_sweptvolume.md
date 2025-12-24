 **Modelica中 SweptVolume 组件的参数与接口表格** ：

---

## 📊 一、参数（Parameters）

| 类型 | 名称 | 默认值 | 单位 | 描述 |
|------|------|--------|------|------|
| `ExtraProperty` | `C[Medium.nC]` | — | — | 示踪物质混合物含量 |
| `Boolean` | `initialize_p` | `not Medium.singleState` | — | 若为 `true`，则为压力设置初始方程 |
| `Medium` | `Medium` | `replaceable package Medium = Modelica.Media.Interfaces.PartialMedium` | — | 组件中的工作介质（可替换） |
| `Density` | `portInDensities[nPorts]` | — | kg/m³ | 设备边界处流体的密度 |
| `Real` | `s[nPorts]` | — | — | 端口流量与端口压力关系的曲线参数；详见 Modelica 教程：理想开关器件 |
| `AbsolutePressure` | `vessel_ps_static[nPorts]` | — | Pa | 容器内对应端口高度处的静压（流速为零时） |
| `Height` | `fluidLevel_max` | `1` | m | 容器中流体的最大液位 |
| `Area` | `vesselArea` | `Modelica.Constants.inf` | m² | 容器横截面积，用于关联端口流通面积 |
| `BooleanInput` | `regularFlow[nPorts]` | — | — | （未明确说明，通常用于流态判断） |
| `BooleanInput` | `inFlow[nPorts]` | — | — | （通常表示流入/流出方向） |
| `Area` | `pistonCrossArea` | — | m² | 活塞横截面积 |
| `Volume` | `clearance` | — | m³ | 活塞行程为零时的剩余容积（余隙容积） |

### 🔧 自定义参数（Custom Parameters）
| 类型 | 名称 | 默认值 | 单位 | 描述 |
|------|------|--------|------|------|
| `HeatTransfer` | `heatTransfer` | `heatTransfer(surfaceAreas={pistonCrossArea + 2*sqrt(pistonCrossArea*pi)*(flange.s + clearance/pistonCrossArea)})` | — | 热传递模型，表面积随活塞位置动态计算 |

---

## ⚙️ 二、高级参数（Advanced）

### ▶ 端口属性（Port properties）
| 类型 | 名称 | 默认值 | 单位 | 描述 |
|------|------|--------|------|------|
| `MassFlowRate` | `m_flow_nominal` | `if system.use_eps_Re then system.m_flow_nominal else 1e2*system.m_flow_small` | kg/s | 端口质量流量的标称值 |
| `MassFlowRate` | `m_flow_small` | `if system.use_eps_Re then system.eps_m_flow*m_flow_nominal else system.m_flow_small` | kg/s | 零质量流量附近的正则化范围 |
| `Boolean` | `use_Re` | `system.use_eps_Re` | — | 若为 `true`，湍流区由雷诺数 Re 定义；否则由 `m_flow_small` 定义 |

### ▶ 假设（Assumptions）

#### 动力学（Dynamics）
| 类型 | 名称 | 默认值 | 描述 |
|------|------|--------|------|
| `Dynamics` | `energyDynamics` | `system.energyDynamics` | 能量平衡的建模形式（稳态/瞬态） |
| `Dynamics` | `massDynamics` | `system.massDynamics` | 质量平衡的建模形式 |

#### 热传递（Heat transfer）
| 类型 | 名称 | 默认值 | 描述 |
|------|------|--------|------|
| `Boolean` | `use_HeatTransfer` | `false` | 若为 `true`，启用热传递模型 |
| `HeatTransfer` | `HeatTransfer` | `replaceable model HeatTransfer = ... IdealHeatTransfer ...` | 壁面热传递模型（可替换） |

#### 通用（General）
| 类型 | 名称 | 默认值 | 描述 |
|------|------|--------|------|
| `Integer` | `nPorts` | `0` | 端口数量 |
| `Boolean` | `use_portsData` | `true` | 若为 `false`，忽略压损和动能项 |
| `VesselPortsData` | `portsData[if use_portsData then nPorts else 0]` | — | 进/出口端口数据（如直径、高度、阻力系数等） |

---

## 🌱 三、初始化参数（Initialization）

| 类型 | 名称 | 默认值 | 单位 | 描述 |
|------|------|--------|------|------|
| `AbsolutePressure` | `p_start` | `system.p_start` | Pa | 压力初始值 |
| `Boolean` | `use_T_start` | `true` | — | 若为 `true`，使用 `T_start`；否则使用 `h_start` |
| `Temperature` | `T_start` | `if use_T_start then system.T_start else Medium.temperature_phX(...)` | K | 温度初始值 |
| `SpecificEnthalpy` | `h_start` | `if use_T_start then Medium.specificEnthalpy_pTX(...) else Medium.h_default` | J/kg | 比焓初始值 |
| `MassFraction` | `X_start[Medium.nX]` | `Medium.X_default` | kg/kg | 质量分数初始值（各组分） |
| `ExtraProperty` | `C_start[Medium.nC]` | `Medium.C_default` | — | 示踪物质初始值 |

---

## 🔌 四、接口（Connectors / Interfaces）

| 类型 | 名称 | 描述 |
|------|------|------|
| `VesselFluidPorts_b` | `ports[nPorts]` | 流体进出口端口 |
| `Modelica.Thermal.HeatTransfer.Interfaces.HeatPort_a` | `heatPort` | 热端口（用于连接热源/散热器） |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_diameter_internal[nPorts]` | 内部端口直径输入（通常由模型自动处理） |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_height_internal[nPorts]` | 内部端口高度输入 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_zeta_in_internal[nPorts]` | 内部进口局部阻力系数 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_zeta_out_internal[nPorts]` | 内部出口局部阻力系数 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_diameter[nPorts]` | 用户可配置的端口直径 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_height[nPorts]` | 用户可配置的端口高度 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_zeta_in[nPorts]` | 用户可配置的进口阻力系数 |
| `Modelica.Blocks.Interfaces.RealInput` | `portsData_zeta_out[nPorts]` | 用户可配置的出口阻力系数 |
| `Modelica.Blocks.Interfaces.BooleanInput` | `regularFlow[nPorts]` | 流态标志（如层流/湍流） |
| `Modelica.Blocks.Interfaces.BooleanInput` | `inFlow[nPorts]` | 流向标志（流入/流出） |
| `Modelica.Mechanics.Translational.Interfaces.Flange_b` | `flange` | 活塞的平移法兰接口（接收位移，输出力） |

---

## ✅ 关键要点总结

- **核心几何参数**：`pistonCrossArea` 和 `clearance` 决定了容积变化规律。
- **热交换**：通过 `heatPort` 实现，是否启用由 `use_HeatTransfer` 控制。
- **机械接口**：`flange` 是关键，连接活塞运动机构。
- **流体端口**：支持多端口，但无阀门，适用于封闭循环（如斯特林发动机）。
- **初始化灵活**：可通过温度 `T_start` 或比焓 `h_start` 启动。

---
