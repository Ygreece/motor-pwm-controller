# Multisim 完整电路整合绘制指南

## 概述

本指南将帮助你在 `motor_pwm_controller.ms14` 中从零绘制完整的直流电机PWM调速器电路。

**布局规划：**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  VCC_5V 总线 (顶部)                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │  NE555 PWM   │    │  2N2222 隔离  │    │  IRL540N     │                   │
│  │  发生器       │───→│  + MOSFET    │───→│  电机驱动    │                   │
│  │  (左上)       │    │  (中上)       │    │  (右上)      │                   │
│  └──────────────┘    └──────────────┘    └──────────────┘                   │
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │  CD40106     │    │  CD4518×2    │    │  CD4511×2    │                   │
│  │  整形+振荡    │───→│  计数器       │───→│  译码器       │                   │
│  │  (左下)       │    │  (中下)       │    │  (右下)      │                   │
│  └──────────────┘    └──────────────┘    └──────────────┘                   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  GND 总线 (底部)                                                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 第一步：创建新文件并设置环境

### 1.1 打开 Multisim
1. 启动 Multisim
2. 文件 → 新建 → 空白
3. 保存为 `motor_pwm_controller.ms14`

### 1.2 设置图纸大小
1. 选项 → 图纸大小
2. 选择 A3 或自定义大小（足够容纳所有模块）

---

## 第二步：绘制电源总线

### 2.1 VCC_5V 总线
1. 在顶部绘制一条水平线
2. 添加标签：`VCC_5V`
3. 连接 +5V 电源

### 2.2 VCC_12V 总线
1. 在 VCC_5V 下方绘制另一条水平线
2. 添加标签：`VCC_12V`
3. 连接 +12V 电源

### 2.3 GND 总线
1. 在底部绘制一条水平线
2. 添加标签：`GND`
3. 连接地线

---

## 第三步：绘制 NE555 PWM 发生器（左上区域）

### 3.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| NE555 | Mixed → TIMER → LM555CM | 1 |
| 电阻 1kΩ | Basic → RESISTOR → 1k | 1 |
| 电位器 20kΩ | Basic → POTENTIOMETER → 20k | 1 |
| 电位器 50kΩ | Basic → POTENTIOMETER → 50k | 1 |
| 二极管 | Diodes → VIRTUAL_DIODE | 2 |
| 电容 10nF | Basic → CAPACITOR → 10n | 2 |
| 电源 +5V | Sources → DC_POWER → 5V | 1 |
| 接地 | Sources → GROUND | 1 |

### 3.2 连线步骤

**步骤1：放置 NE555**
1. Place → Component → Mixed → TIMER → LM555CM
2. 放置在左上区域

**步骤2：连接电源**
1. NE555 pin8(VCC) → VCC_5V 总线
2. NE555 pin1(GND) → GND 总线
3. NE555 pin4(RST) → VCC_5V 总线

**步骤3：连接定时电路**
```
VCC_5V ── R1(1kΩ) ── Node A ── NE555 pin7(DIS)
                       │
              ┌────────┴────────┐
              │                 │
         D1(阳→阴)         D2(阴←阳)
              │                 │
         R_freq(20kΩ)     R_duty(47kΩ)
              │                 │
              └────────┬────────┘
                       │
                  Node B ── NE555 pin2(TR) ── NE555 pin6(TH)
                       │
                  C(10nF) ── GND
```

**步骤4：连接旁路电容**
1. NE555 pin5(CTRL) → C_bypass(10nF) → GND

**步骤5：输出连接**
1. NE555 pin3(OUT) → 连接到 2N2222 隔离电路（下一步）

**步骤6：添加测试点**
1. 在 NE555 pin3 添加示波器通道 A
2. 在 Node B 添加示波器通道 B

---

## 第四步：绘制 TLP121 光耦隔离 + MOSFET 驱动（中上区域）

### 4.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| TLP121 光耦 | Optocoupler → TLP121 | 1 |
| IRL540N | Transistors → POWER_MOSFET → IRL540N | 1 |
| 电阻 330Ω | Basic → RESISTOR → 330 | 1 |
| 电阻 10kΩ | Basic → RESISTOR → 10k | 1 |
| 二极管 1N4007 | Diodes → 1N4007 | 1 |
| 电阻 24Ω | Basic → RESISTOR → 24 | 1 |
| 电感 10mH | Basic → INDUCTOR → 10m | 1 |
| 电源 +12V | Sources → DC_POWER → 12V | 1 |
| 接地 | Sources → GROUND | 1 |

### 4.2 连线步骤

**步骤1：放置光耦和 MOSFET**
1. Place → Component → Optocoupler → TLP121
2. Place → Component → Transistors → POWER_MOSFET → IRL540N
3. 放置在中上区域

**步骤2：连接光耦隔离电路**
```
NE555 pin3(OUT) ── R_led(330Ω) ── TLP121 pin1(阳极)
TLP121 pin2(阴极) ── GND

TLP121 pin4(集电极) ── IRL540N Gate
TLP121 pin3(发射极) ── GND

VCC(+12V) ── R_pull(10kΩ) ── IRL540N Gate
```

**步骤3：连接电机驱动电路**
```
VCC(+12V) ── 电机等效(24Ω + 10mH 串联) ── IRL540N Drain
                                          │
                                      IRL540N Source ── GND
```

**步骤4：连接续流二极管**
```
1N4007 阴极 ── VCC(+12V)
1N4007 阳极 ── IRL540N Drain
```

**步骤5：添加测试点**
1. 在 IRL540N Gate 添加示波器通道 C
2. 在 IRL540N Drain 添加示波器通道 D

---

## 第五步：绘制 CD40106 整形电路（左下区域）

### 5.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| CD40106 | CMOS → CMOS_5V → 40106BP_5V | 1 |
| 电阻 10kΩ | Basic → RESISTOR → 10k | 1 |
| 脉冲源 | Sources → CLOCK_VOLTAGE → 5V, 50Hz | 1 |
| 电源 +5V | Sources → DC_POWER → 5V | 1 |
| 接地 | Sources → GROUND | 1 |

### 5.2 连线步骤

**步骤1：放置 CD40106**
1. Place → Component → CMOS → CMOS_5V → 40106BP_5V
2. 放置在左下区域

**步骤2：连接电源**
1. CD40106 pin14(VDD) → VCC_5V 总线
2. CD40106 pin7(VSS) → GND 总线

**步骤3：连接转速信号源**
1. Place → Component → Sources → CLOCK_VOLTAGE
2. 设置：5V, 50Hz（模拟转速信号）
3. 脉冲源输出 → CD40106 pin1(1A 输入)

**步骤4：连接上拉电阻**
1. CD40106 pin1 → R_pull(10kΩ) → VCC_5V

**步骤5：连接整形输出**
1. CD40106 pin2(1Y 输出) → 连接到 CD4518 计数器（下一步）

**步骤6：添加测试点**
1. 在脉冲源输出添加示波器通道 E
2. 在 CD40106 pin2 添加示波器通道 F

---

## 第六步：绘制 CD40106 振荡 + 差分时序（左下区域，与整形共用）

### 6.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| 电位器 100kΩ | Basic → POTENTIOMETER → 100k | 1 |
| 电解电容 10µF | Basic → CAPACITOR → 10u | 2 |
| 瓷片电容 0.1µF | Basic → CAPACITOR → 100n | 1 |
| 电阻 10kΩ | Basic → RESISTOR → 10k | 3 |
| 二极管 1N4148 | Diodes → 1N4148 或 VIRTUAL_DIODE | 1 |
| 电源 +5V | Sources → DC_POWER → 5V | 1 |
| 接地 | Sources → GROUND | 1 |

### 6.2 连线步骤

**步骤1：连接振荡器（使用 CD40106 第2个门）**
```
VCC_5V ── R_osc(100kΩ 电位器 W3) ── CD40106 pin3(2A)
                                     │
                                C_osc(10µF 电解，正极接 pin3) ── GND
                                     │
                                R_fb(10kΩ) ── CD40106 pin4(2Y)
```

**步骤2：连接差分电路（产生锁存脉冲 LE）**
```
CD40106 pin4(振荡输出) ── C_diff(0.1µF) ── 节点X
                                             │
                                        R_diff(10kΩ) ── GND
                                             │
                                        D_clamp(1N4148 阳极→阴极) ── GND
                                             │
                                        节点X ──→ 锁存脉冲(LE) → CD4511 ~EL
```

**步骤3：连接 RC 延时 + 整形（产生清零脉冲 RST）**
```
CD40106 pin4 ── R_delay(10kΩ) ── 节点Y
                                 │
                            C_delay(10µF 电解，正极接节点Y) ── GND
                                 │
                            CD40106 pin5(3A 输入)
                                 │
                            CD40106 pin6(3Y 输出) ──→ 清零脉冲(RST) → CD4518 RST
```

**步骤4：添加测试点**
1. 在 CD40106 pin4 添加示波器通道 G
2. 在节点X 添加示波器通道 H
3. 在 CD40106 pin6 添加示波器通道 I

---

## 第七步：绘制 CD4518 计数器（中下区域）

### 7.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| CD4518 | CMOS → CMOS_5V → 4518BP_5V | 1 |
| 电源 +5V | Sources → DC_POWER → 5V | 1 |
| 接地 | Sources → GROUND | 1 |

### 7.2 连线步骤

**步骤1：放置 CD4518**
1. Place → Component → CMOS → CMOS_5V → 4518BP_5V
2. 放置在中下区域

**步骤2：连接电源**
1. CD4518 VDD → VCC_5V 总线
2. CD4518 VSS → GND 总线

**步骤3：连接个位计数器**
```
CD4518 1CLK ── CD40106 pin2(整形后的转速脉冲)
CD4518 ~1CLK ── GND（低电平使能）
CD4518 1RST ── CD40106 pin6(清零脉冲 RST)
CD4518 1A~1D ──→ CD4511(个位) DA~DD
```

**步骤4：连接十位计数器**
```
CD4518 2CLK ── GND
CD4518 ~2CLK ── CD4518 1D（下降沿触发，实现进位）
CD4518 2RST ── CD40106 pin6(清零脉冲，与个位并联)
CD4518 2A~2D ──→ CD4511(十位) DA~DD
```

---

## 第八步：绘制 CD4511 译码器 + 数码管（右下区域）

### 8.1 元件清单
| 元件 | Multisim 路径 | 数量 |
|------|---------------|------|
| CD4511 | CMOS → CMOS_5V → 4511BD_5V | 2 |
| 七段数码管(共阴) | Indicators → HEX_DISPLAY → SEVEN_SEG_COM_K | 2 |
| 电阻 330Ω | Basic → RESISTOR → 330 | 14 |
| 电源 +5V | Sources → DC_POWER → 5V | 1 |
| 接地 | Sources → GROUND | 1 |

### 8.2 连线步骤

**步骤1：放置 CD4511 和数码管**
1. Place → Component → CMOS → CMOS_5V → 4511BD_5V × 2
2. Place → Component → Indicators → HEX_DISPLAY → SEVEN_SEG_COM_K × 2
3. 放置在右下区域

**步骤2：连接个位 CD4511**
```
CD4511(个位) DA~DD ── CD4518 1A~1D
CD4511(个位) ~LT ── VCC_5V
CD4511(个位) ~BI ── VCC_5V
CD4511(个位) ~EL ── 节点X(锁存脉冲 LE)
CD4511(个位) QA~QG ── 各经 330Ω ── 个位数码管 a~g
```

**步骤3：连接十位 CD4511**
```
CD4511(十位) DA~DD ── CD4518 2A~2D
CD4511(十位) ~LT ── VCC_5V
CD4511(十位) ~BI ── VCC_5V
CD4511(十位) ~EL ── 节点X(锁存脉冲，与个位并联)
CD4511(十位) QA~QG ── 各经 330Ω ── 十位数码管 a~g
```

**步骤4：连接数码管公共端**
```
个位数码管 COM(阴极) ── GND
十位数码管 COM(阴极) ── GND
```

---

## 第九步：添加示波器和测试点

### 9.1 示波器连接
1. Place → Instrument → Oscilloscope
2. 连接通道：
   - 通道 A：NE555 pin3(OUT) — PWM 输出
   - 通道 B：Node B — 锯齿波
   - 通道 C：IRL540N Gate — MOSFET 驱动
   - 通道 D：IRL540N Drain — 电机电压
   - 通道 E：转速脉冲源
   - 通道 F：CD40106 pin2 — 整形后转速
   - 通道 G：CD40106 pin4 — 振荡输出
   - 通道 H：节点X — 锁存脉冲
   - 通道 I：CD40106 pin6 — 清零脉冲

### 9.2 测试点标签
在关键节点添加网络标签：
- `PWM_OUT` — NE555 输出
- `GATE` — MOSFET 栅极
- `DRAIN` — MOSFET 漏极
- `SPEED_PULSE` — 转速脉冲
- `SPEED_SHAPED` — 整形后转速
- `OSC_OUT` — 振荡输出
- `LE` — 锁存脉冲
- `RST` — 清零脉冲

---

## 第十步：仿真验证

### 10.1 仿真设置
1. Simulate → Analyses and simulation
2. 选择 Transient Analysis
3. 设置：
   - End time: 10ms（先测 PWM）
   - Maximum time step: 1µs

### 10.2 分步验证

**验证1：NE555 PWM 输出**
1. 运行仿真
2. 观察示波器通道 A 和 B
3. 预期：3~5kHz 方波，占空比可调

**验证2：MOSFET 驱动**
1. 观察示波器通道 C 和 D
2. 预期：Gate 电压跟随 PWM，Drain 电压在 0V 和 12V 之间切换

**验证3：转速检测**
1. 观察示波器通道 E 和 F
2. 预期：50Hz 脉冲被 CD40106 整形为干净方波

**验证4：时序控制**
1. 延长仿真时间到 10s
2. 观察示波器通道 G、H、I
3. 预期：先锁存(LE)，后清零(RST)

**验证5：计数显示**
1. 观察数码管显示
2. 预期：显示转速值（50Hz → 显示 50）

---

## 常见问题排查

### NE555 不振荡
- 检查 RST(pin4) 是否接 VCC
- 检查 D1/D2 方向
- 检查电源负极是否接 GND

### MOSFET 不导通
- 检查 Gate 上拉电阻是否接好
- 检查 2N2222 是否正确连接
- 用示波器观察 Gate 电压变化

### CD40106 不振荡
- 检查 VCC/GND 是否正确
- 检查正反馈电阻 R_fb 是否接好
- 检查定时电容极性（电解电容正极接输入端）

### CD4518 不计数
- 检查 ~1CLK 是否接 GND
- 检查 1CLK 是否接到脉冲信号
- 检查 VDD/VSS

### CD4511 不译码
- 检查 ~LT/~BI 是否接 VCC
- 检查 ~EL 是否接到锁存脉冲
- 检查 QA~QG 是否正确连接到数码管

### 仿真极慢
- Simulate → Analyses and simulation → Interactive Simulation
- Maximum time step → User-defined → 输入 `1e-04`

---

## 完整接线清单（按元件顺序）

### NE555 (U1)
1. Pin1(GND) → GND
2. Pin2(TR) → Node B
3. Pin3(OUT) → 2N2222 Base (通过 R_base 1kΩ)
4. Pin4(RST) → VCC_5V
5. Pin5(CTRL) → C_bypass(10nF) → GND
6. Pin6(TH) → Node B
7. Pin7(DIS) → Node A
8. Pin8(VCC) → VCC_5V

### TLP121 光耦 (U_opto)
1. Pin1(阳极) → R_led(330Ω) → NE555 pin3
2. Pin2(阴极) → GND
3. Pin4(集电极) → IRL540N Gate + R_pull(10kΩ) → VCC_12V
4. Pin3(发射极) → GND

### IRL540N (Q2)
1. Gate → TLP121 Pin4 + R_pull(10kΩ) → VCC_12V
2. Drain → 电机等效(24Ω+10mH) + 1N4007 阳极
3. Source → GND

### 电机等效
1. 24Ω + 10mH 串联
2. 一端接 VCC_12V
3. 另一端接 IRL540N Drain

### 续流二极管 1N4007
1. 阴极 → VCC_12V
2. 阳极 → IRL540N Drain

### CD40106 (U2) — 六施密特触发反相器（14脚DIP）
1. Pin1(1A输入) → 转速脉冲源(50Hz) + R_pull(10kΩ) → VCC_5V
2. Pin2(1Y输出) → CD4518 1CLK
3. Pin3(2A输入) → R_osc(100kΩ) + C_osc(10µF) + R_fb(10kΩ) → Pin4
4. Pin4(2Y输出) → C_diff(0.1µF) + R_delay(10kΩ)
5. Pin5(3A输入) → R_delay(10kΩ) + C_delay(10µF)
6. Pin6(3Y输出) → CD4518 1RST + CD4518 2RST
7. Pin7(VSS) → GND
8. Pin8(4Y输出) → 未使用
9. Pin9(4A输入) → 未使用
10. Pin10(5Y输出) → 未使用
11. Pin11(5A输入) → 未使用
12. Pin12(6Y输出) → 未使用
13. Pin13(6A输入) → 未使用
14. Pin14(VDD) → VCC_5V

### CD4518 (U3)
1. VDD → VCC_5V
2. VSS → GND
3. 1CLK → CD40106 pin2
4. ~1CLK → GND
5. 1RST → CD40106 pin6
6. 1A~1D → CD4511(个位) DA~DD
7. 2CLK → GND
8. ~2CLK → 1D
9. 2RST → CD40106 pin6
10. 2A~2D → CD4511(十位) DA~DD

### CD4511 个位 (U4)
1. VDD → VCC_5V
2. VSS → GND
3. DA~DD → CD4518 1A~1D
4. ~LT → VCC_5V
5. ~BI → VCC_5V
6. ~EL → 节点X(锁存脉冲 LE)
7. QA~QG → 各经 330Ω → 个位数码管 a~g

### CD4511 十位 (U5)
1. VDD → VCC_5V
2. VSS → GND
3. DA~DD → CD4518 2A~2D
4. ~LT → VCC_5V
5. ~BI → VCC_5V
6. ~EL → 节点X(锁存脉冲，与个位并联)
7. QA~QG → 各经 330Ω → 十位数码管 a~g

### 个位数码管
1. a~g → CD4511(个位) QA~QG (各经 330Ω)
2. COM(阴极) → GND

### 十位数码管
1. a~g → CD4511(十位) QA~QG (各经 330Ω)
2. COM(阴极) → GND

---

## 绘制顺序建议

1. ✅ 先画电源总线（VCC_5V, VCC_12V, GND）
2. ✅ 画 NE555 PWM 发生器
3. ✅ 画 TLP121 光耦隔离 + MOSFET 驱动
4. ✅ 画 CD40106 整形电路
5. ✅ 画 CD40106 振荡 + 差分时序
6. ✅ 画 CD4518 计数器
7. ✅ 画 CD4511 译码器 + 数码管
8. ✅ 添加示波器和测试点
9. ✅ 运行仿真验证

---

## 完成检查清单

- [ ] 所有电源连接正确（VCC_5V, VCC_12V, GND）
- [ ] NE555 振荡正常（3~5kHz）
- [ ] MOSFET 驱动正常（Gate 电压变化）
- [ ] 电机等效电路工作正常
- [ ] 转速信号整形正常
- [ ] 时序控制正确（先锁存，后清零）
- [ ] 计数器正常计数
- [ ] 译码器正常译码
- [ ] 数码管显示正确
- [ ] 所有测试点可测量

---

**祝你绘制顺利！** 🎯
