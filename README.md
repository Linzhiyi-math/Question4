# 第四题：多神经元群体编码的 MVUB 估计数值模拟

## 作业说明

本作业通过数值模拟，研究多神经元群体编码中**最小方差无偏估计量（MVUB）**的性能，重点比较两种常用估计量——**最大似然估计（MLE）**与**群体向量估计（Population Vector，PV）**——并以 **Cramér–Rao 下界（CRLB）** 作为理论基准。

---

## 环境要求

- **开发环境**：Python 3.14.4
- **运行环境**：Jupyter Notebook
- **依赖库**：numpy, matplotlib, scipy, tqdm, platform, warnings
- （版本见 requirements.txt）

安装第三方依赖：

```bash
pip install -r requirements.txt
```
---
## 运行方法

使用 Jupyter Notebook**

```bash
jupyter notebook 第四题.ipynb
```

打开后，点击菜单栏 **Cell → Run All** 运行全部代码。

---

## 模型说明

### 神经元调谐函数

每个神经元 $a$ 对刺激 $s$ 的响应用高斯调谐函数描述：

$$f_a(s) = \exp\left(-\frac{(s - s_a)^2}{2\sigma^2}\right)$$

其中 $s_a$ 为该神经元的偏好刺激方向（均匀分布于 $[0°, 180°]$）。

### 发放计数模型

神经元的发放计数服从泊松分布：

$$k_a \sim \text{Poisson}(T \cdot f_a(s))$$

其中 $T$ 为观测时间窗口长度。

---

## 参数配置

| 参数 | 符号 | 默认值 | 说明 |
|------|------|--------|------|
| 神经元数量 | $N$ | 100 | 群体编码神经元总数 |
| 调谐宽度 | $\sigma$ | 20° | 高斯调谐函数的标准差 |
| 观测时间 | $T$ | 1.0 s | 泊松计数的时间窗口 |
| 模拟次数 | $M$ | 5000 | 每个测试方向的 Monte Carlo 重复次数 |
| 测试方向数 | $N_\text{test}$ | 37 | 均匀采样自 $[10°, 170°]$ |
| 刺激范围 | $[S_\text{min}, S_\text{max}]$ | $[0°, 180°]$ | 神经元偏好方向的覆盖范围 |

---

## 估计量实现

### 1. 最大似然估计（MLE）

采用**两阶段优化**策略，兼顾全局搜索与精度：

1. **粗网格搜索**：在 500 点均匀网格上计算对数似然，定位峰值区间；
2. **Brent 法精细化**：在峰值附近 $\pm\max(3\sigma, 15°)$ 区间内，用 Brent 法精确求解 Score 方程 $= 0$；
3. **降级处理**：若 Brent 法因区间端点同号失败，退化为粗网格峰值。

### 2. 群体向量估计（PV）

以发放计数为权重，对神经元偏好方向取加权平均：

$$\hat{s}_\text{PV} = \frac{\sum_a k_a \cdot s_a}{\sum_a k_a}$$

### 3. Cramér–Rao 下界（CRLB）

基于 Fisher 信息量计算无偏估计量的方差下限：

$$\text{CRLB}(s) = \frac{1}{I(s)}, \quad I(s) = \sum_a \frac{[\lambda_a'(s)]^2}{\lambda_a(s)}$$

---

## 输出内容

### 图像文件：`simulation_q4.png`

包含三个子图：

| 子图 | 内容 |
|------|------|
| 上图（全幅） | MLE 与 PV 的 MSE 随刺激方向的变化，对比 CRLB 理论下界 |
| 左下图 | 效率比 $\text{MSE} / \text{CRLB}$，值越接近 1 表示估计越高效 |
| 右下图 | MLE 与 PV 的估计偏差随刺激方向的变化 |

### 终端数值摘要

程序结束后打印跨全部测试方向和中心区域（$s \in [30°, 150°]$）的统计汇总：

- CRLB 均值与标准差
- MSE（MLE 与 PV）均值与标准差
- 效率比 $\text{MSE} / \text{CRLB}$
- 估计偏差绝对均值

---
## 代码结构

```
第四题.ipynb
├── 参数设置          # N, σ, T, M, 测试方向等
├── 核心函数
│   ├── lambda_a_vec()         # 泊松均值向量
│   ├── log_likelihood_grid()  # 批量对数似然（粗网格）
│   ├── score()                # Score 函数（MLE 求根目标）
│   ├── mle_estimate()         # 两阶段 MLE 求解
│   ├── pv_estimate()          # 群体向量估计
│   ├── fisher_information()   # Fisher 信息量
│   └── crlb()                 # Cramér–Rao 下界
├── 主模拟循环        # Monte Carlo 估计 MSE、偏差
├── 绘图              # GridSpec 三子图输出
└── 数值摘要          # 全域与中心区域统计
```

---

## 注意事项

- 随机种子固定为 `seed=42`，结果可完全复现。
- 中文字体自动检测，支持 macOS、Windows 及 Linux（含 WenQuanYi / Noto CJK）。
- 边界附近（$s < 20°$ 或 $s > 160°$）的 PV 估计因偏好方向分布截断存在系统偏差，属正常现象。
- MLE 在中心区域应接近达到 CRLB（效率比约为 1），PV 在调谐宽度较大时偏离明显。
