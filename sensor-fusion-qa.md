# 传感器融合 Q&A 补充笔记

跟单篇论文精读不同,这是一份跨论文的概念澄清笔记——由讨论 **EMMA(Waymo, arXiv 2410.23262)** 附录 A.5 的局限性段落引出,展开成"摄像头 vs LiDAR vs 雷达""为什么要融合""融合卡在哪""LiDAR 数据长什么样"这几个连续问题。EMMA 本身尚未按标准模板精读,这里只作为引子引用。

---

## 1. EMMA 自曝的三条局限,分别对应哪些论文的哪些设计

EMMA 原文(附录 A.5):

> "it faces challenges for real-world deployment due to: (1) limitations in 3D spatial reasoning due to its inability to fuse camera inputs with LiDAR or radar, (2) the need for realistic and computationally expensive sensor simulation to power its closed-loop evaluation, and (3) the increased computational requirements relative to conventional models."

**核对结论:** 这三条不是 EMMA 独有的弱点,而是"用大模型统一处理驾驶"这整条技术路线(DriveVLM、Alpamayo-R1、Qwen-Drive-1.0、DriveZero、WA-JEPA 都在内)共同要面对的代价。三条分别对应:

### (1) 3D 空间推理受限——没有 LiDAR/毫米波融合

纯视觉(camera-only)是这批 VLM 驱动模型的常态:**DriveVLM、Qwen-Drive-1.0、DriveZero、Drive-JEPA、WA-JEPA 全都是纯摄像头输入**,都会面对同样的问题。

- **反例(加了 LiDAR 但没用上)**:DriveMLM 融合了 LiDAR,但消融实验(表5)显示准确率几乎没差(74.99% vs 75.23%,反而略降)。论文自己承认可能是 LiDAR 编码器(SST)和语言模型 decoder 表征差距太大。
- **正例(LiDAR 真正有效融合)**:ARTEMIS——8 路相机图像 + 5 个 LiDAR 点云融合成 BEV 特征(TransFuser 式),是它感知模块的核心设计。

结论:这条限制更准确的说法是"纯视觉路线普遍缺精确深度信息,但加 LiDAR 不代表一定有效——融合方法本身的设计比'要不要加传感器'更关键"。

### (2) 闭环评测需要昂贵的传感器仿真

几乎每篇论文都在应对这个矛盾,程度不同:

| 论文 | 应对方式 |
|---|---|
| NAVSIM | 专门为此设计的折中方案:非反应式仿真(PDM Score),比开环可信、比闭环便宜 |
| Qwen-Drive-1.0 | 三档都报(开环<伪闭环 NAVSIM<闭环 AlpaSim),诚实标注闭环结果只是"promising potential" |
| WA-JEPA | 额外在 HUGSIM(436 个零样本场景)上做真闭环验证,SOTA(HD-Score 0.4462) |
| DriveZero | 把闭环仿真用在训练阶段(DriveRL 混合智能体闭环 RL),不只是评测阶段 |
| DriveMLM | 直接对接成熟的 CARLA 闭环仿真器 + 传统模块化系统 |
| Drive-JEPA / SparseDriveV2 | 老实承认只做了 open-loop/pseudo-closed-loop,真实闭环表现"未知" |

### (3) 计算需求比传统模型高

三种不同的工程哲学:

- **DriveVLM-Dual**:"绕开"——VLM 当慢系统(System 2)生成低频参考轨迹,传统快速规划器当快系统(System 1)高频微调,OrinX 上约 410ms。
- **Alpamayo-R1**:"改造推理方式"——训练用离散 token 自回归,推理时改用 flow matching 动作专家(5步 Euler 积分),99ms 端到端延迟,比 DriveVLM-Dual 快 4 倍多。
- **Qwen-Drive-1.0**:"暂不解决,先如实承认"——需要 24GB+ 显存,未做实时化工程优化,写进了自己的 Limitation。

---

## 2. "Fuse(融合)"到底想达到什么效果?——不是标准化

**常见误解**:融合 = 把不同传感器的数据格式统一/标准化。

**实际目的**:利用不同传感器"不一样"这件事本身——每种传感器的失效场景是互相错开的,融合让感知结果由多个独立证据源共同支撑,而不是让数据"整齐划一"。

| 传感器 | 强项 | 弱项 |
|---|---|---|
| 摄像头(Camera) | 语义信息丰富(颜色/纹理/类别/文字/灯光状态) | 深度是推断出来的,不是实测;逆光/夜晚/大雾容易失效 |
| LiDAR(激光雷达) | 直接测距,3D 点云精确,不受光照影响 | 没有颜色/纹理,形状相似物体难分类(石头 vs 塑料袋);暴雨大雾会被散射 |
| 毫米波雷达(Radar) | 直接测速度(多普勒),穿透雾霾沙尘能力强,探测距离远 | 空间分辨率很低,分不清形状/类别 |

融合的真正目的:摄像头说"可能是行人",LiDAR 给出精确距离和轮廓验证;LiDAR 说"前方20米有物体",摄像头判断是路障还是猫,决定要不要急刹。任一传感器失效时,系统还有别的证据源兜底——这是加传感器的根本动机。

**融合方式本身很关键,读过的论文里至少有三种不同做法:**

| 论文 | 融合方式 | 效果 |
|---|---|---|
| DriveMLM | 训练 LiDAR 编码器(SST)通过余弦相似度去逼近对应图像的 CLIP 特征——本质是让 LiDAR 表征"模仿"摄像头表征 | 消融实验几乎无提升(74.99% vs 75.23%),这个设计思路可能正好把 LiDAR 最独特的优势(精确几何)磨平了(**这是我的推断,非论文原文结论**) |
| ARTEMIS | 直接把两种模态投影进同一个 BEV 特征空间共同处理(TransFuser 式),不强迫一种模态模仿另一种 | 是感知模块的核心设计,效果良好,是更常见也通常更有效的融合思路 |
| Autoware(2015) | **外参反投影(reprojection)**:用相机-LiDAR 标定好的外参矩阵,把 3D 点云直接投影到相机图像上(给图像加深度信息、缩小检测搜索范围),反过来把图像检测结果投影回 3D 点云坐标 | 最经典、最"硬编码几何"的融合方式,不靠神经网络学习对齐,而是用物理安装位置关系做数学投影——最可靠、最好理解,但也最不"智能"。原文:"We can then project the 3D point-cloud information obtained by the 3D Lidar sensor onto the image captured by the camera...The result of object detection on the image can also be reprojected onto the 3D point-cloud coordinates using the same extrinsic parameters." |

---

## 3. 为什么这批 VLM 驱动的模型都倾向纯摄像头?——不只是"图像处理先进了"

**更准确的原因**:摄像头能蹭上互联网规模的图文预训练红利,LiDAR 蹭不上。

CLIP、Gemini、Qwen-VL 这类基础模型的强大能力,来自海量"图片+文字"网络爬取数据的预训练。**根本不存在互联网规模的"LiDAR 点云+文字描述"配对数据集**——没人会在网上传"这是一份激光雷达点云,配文字说明"。所以摄像头能白嫖大模型的常识推理能力,LiDAR 没有对应的预训练生态可蹭。

这正好呼应 Qwen-Drive-1.0 的核心论点:通用 VLM 表征已隐含了足够的 3D 场景理解,不需要专门的感知头去"学"——前提正是这个表征来自海量图文预训练,这条捷径 LiDAR 走不了。

**结论**:不是摄像头测距能力真的追上了 LiDAR(单目深度在物理上依然是推断,没有变),而是"camera-only"能省事接入现成大模型生态,LiDAR 想接入就得额外造桥梁(比如 DriveMLM 那种硬训对齐层)。

---

## 4. "传感器融合到底解决没解决"——要分两个场景看,原因是可以说清楚的

不是"融合整体解决不了、原因不明",要分场景:

**场景一(相对成熟)**:传统"摄像头+LiDAR 直接融合做 3D 检测"。工业界很多实际部署系统(包括 Waymo 自己真正上路的车,不是 EMMA 这个研究模型)依然全传感器融合使用,效果比纯摄像头好。ARTEMIS 的 TransFuser 式融合就是这类成熟方案的代表。

**场景二(真正不成熟)**:把 LiDAR 融合进这些新的 VLM/LLM 驾驶大模型里。原因明确——这些大模型骨干天生是为"图像+文字"设计和预训练的,LiDAR 点云是完全陌生的模态,没有现成桥梁能接进去。DriveMLM 的尝试(LiDAR 编码器模仿 CLIP 特征)效果不好,但失败原因可以具体诊断(见上面第2节),不是无解的黑箱。

---

## 5. LiDAR 数据长什么样

**原始格式**:不是图,是"点云(point cloud)"——一个列表,每行是空间中一个点:

```
x,     y,     z,     intensity
12.34, 5.67,  0.89,  0.42
12.31, 5.70,  0.91,  0.38
...(一帧可能几万到几十万行)
```

- **x, y, z**:相对车辆/传感器的三维坐标(米),实测距离。
- **intensity(反射强度)**:激光反射信号强度,和材质有关,常当伪灰度值辅助区分材质。
- 部分格式还有 ring(第几线激光束)、timestamp。

常见存储格式:`.bin`(KITTI,每 4 个 float32 一组)、`.pcd`、`.las`。

**和图像最本质的区别**:摄像头是"稠密规则网格"(每个像素格必有 RGB 值);LiDAR 点云是"稀疏不规则散点"(3D 空间绝大部分位置没有点,只有激光真正打到物体表面才产生点),而且**近处密、远处稀**(激光束放射状发出,越远相邻光束间隔越大)。

这正好解释了第 4 节"为什么接入 VLM 这么难"——图像那种稠密规则网格天生适合 CNN/ViT 直接处理;点云稀疏无序、点数还不固定,没法直接套用,得先体素化(voxelization,见 glossary)或用专门点云网络(PointNet、DriveMLM 用的 SST)转成规则表示,这个转换步骤本身就是一层额外复杂度和信息损失点。

**视觉直观**:从正上方看一帧(鸟瞰),车辆自身是空的(传感器装车顶,车身挡住的地方没反射点);周围车辆呈矩形点云"壳"(激光打不穿进去);树木是杂乱的垂直点团;地面是密集点铺成的平面;远处物体经常只有稀稀拉拉几个点,轮廓模糊——这也是"远处小物体 LiDAR 容易漏检"的直观原因。

实际样例图参考:
- [3D LiDAR Visualization: Case Study on 2D KITTI Depth Frames](https://learnopencv.com/3d-lidar-visualization/)
- [Example of 3D LIDAR point clouds from the KITTI benchmark dataset](https://www.researchgate.net/figure/Example-of-3D-LIDAR-point-clouds-from-the-KITTI-benchmark-dataset-followed-by-the_fig3_338591246)(点云图和摄像头图像并排对比)
