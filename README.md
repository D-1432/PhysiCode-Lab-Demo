PhysiCodeLab — 高性能物理求解器核心
PhysiCodeLab 是一个模块化、跨平台的物理仿真求解器，专注于刚体动力学与粒子系统。它提供 C++17 核心库，并通过 Emscripten 编译为 WebAssembly，可直接在浏览器中运行实时物理仿真。

当前版本：v0.6.0 — MVP（最小可行产品）已完整实现，包含刚体/粒子运动、碰撞检测与响应、RK4/Euler 积分器、WebAssembly 绑定及交互式前端演示。

https://screenshot.png (你可以后续添加一张实际运行的截图)

✨ 主要特性
双精度刚体与粒子：支持球体、立方体、粒子，质量、弹性系数、摩擦系数可配置。

碰撞检测与响应：基于 Bullet Physics 的精确碰撞检测，采用顺序冲量法求解接触约束。

时间积分器：Euler 和 Runge-Kutta 4 (RK4)，固定时间步长。

WebAssembly 原生绑定：通过 Emscripten + Embind 暴露 API，前端可无缝调用。

实时可视化：随附的 HTML 演示程序，利用 Canvas 绘制粒子，支持动画、单步、CSV 数据导出。

跨平台：可在 Windows / Linux / macOS 编译原生可执行文件，也可生成 WASM 用于浏览器。

🚀 快速体验（在线演示）
如果你只想直接运行演示，无需本地编译，可访问以下任意托管地址：

GitHub Pages：https://你的用户名.github.io/PhysiCodeLab-Demo/

Gitee Pages（国内推荐）：https://你的用户名.gitee.io/PhysiCodeLab-Demo/

注意：在线演示仅包含前端交互，无需任何后台服务。所有物理计算均在您的浏览器中完成。

📦 项目结构
text
PhysiCodeLab/
├── CMakeLists.txt                 # 主构建脚本
├── MathLibrary.h                  # Eigen 封装，定义 Vec3, Mat3, Quat
├── Body.h                         # 物理物体抽象基类
├── Particle.h / .cpp              # 粒子实现
├── RigidBody.h / .cpp             # 刚体实现（球/盒）
├── StaticPlane.h / .cpp           # 静态平面
├── World.h / .cpp                 # 场景管理、碰撞求解、积分器
├── CollisionDetector.h            # 碰撞检测抽象接口
├── BulletCollisionDetector.h/.cpp # Bullet 实现
├── Observer.h                     # 观察者模式，用于数据导出
├── wasm_bindings.cpp              # WebAssembly 绑定代码
├── test_*.cpp                     # Google Test 单元测试
└── web_demo/                      # 前端演示文件
    ├── index.html
    ├── PhysiCodeSolver_WASM.js
    └── PhysiCodeSolver_WASM.wasm
🔧 本地编译（原生 Windows/Linux/macOS）
依赖项
CMake 3.14+

C++17 编译器（GCC 9+ / Clang 12+ / MSVC 2019+）

Eigen 3.4（已包含在 D:/libs/eigen-3.4.0，请按需修改路径）

Bullet Physics 3.25（原生使用 find_package(Bullet)，需提前安装）

Google Test（仅单元测试需要）

编译步骤
bash
git clone https://github.com/yourname/PhysiCodeLab.git
cd PhysiCodeLab
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=ON
cmake --build . --config Release
运行单元测试：

bash
ctest --output-on-failure
🌐 编译 WebAssembly 模块
前置条件
安装 Emscripten SDK（版本 3.1.50+）

激活环境：emsdk activate latest && emsdk_env.bat（Windows）或 source ./emsdk_env.sh（Linux/macOS）

编译 Bullet 静态库（仅首次需要）
bash
cd /path/to/bullet3-3.25
mkdir build-wasm && cd build-wasm
emcmake cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DBUILD_BULLET2_DEMOS=OFF -DBUILD_CPU_DEMOS=OFF -DBUILD_EXTRAS=OFF -DUSE_DOUBLE_PRECISION=ON
ninja
编译 PhysiCodeLab 为 WASM
bash
cd PhysiCodeLab
rmdir /s /q build-wasm   # 清理旧构建
mkdir build-wasm && cd build-wasm
emcmake cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=OFF
ninja
生成的文件位于 build-wasm/：

PhysiCodeSolver_WASM.js

PhysiCodeSolver_WASM.wasm

运行前端演示
将 index.html 与上述两个文件放在同一目录，启动 HTTP 服务器：

bash
python -m http.server 8080
访问 http://localhost:8080。

🎮 前端 API 说明（JavaScript）
通过 window.PhysiCodeSolverModule 加载模块后，可获得以下接口：

javascript
const Module = await window.PhysiCodeSolverModule();
const world = new Module.World();

// 设置重力 (Vec3)
const gravity = new Module.Vec3(0, -9.8, 0);
world.setGravity(gravity);

// 添加平面 (法线, 偏移)
const normal = new Module.Vec3(0, 1, 0);
world.addPlane(normal, 0.0);

// 添加粒子 (x, y, z, 质量, 半径)
world.addParticle(0, 5, 0, 1.0, 0.3);

// 单步仿真
world.step(0.02);

// 获取所有粒子的位置和半径 (返回 JS 数组)
const data = world.getParticlesData();  // [x1,y1,z1,r1, x2,y2,...]

// 导出 CSV
const csv = world.exportData();

// 释放资源
world.delete();
详细示例请参考 web_demo/index.html。

📊 性能与精度
指标	目标（计划书）	当前实测
自由落体位移误差（RK4, dt=0.01）	< 0.1%	≈ 1e-5%
弹性碰撞动能相对误差	< 0.5%	≈ 1e-5
刚体数量 @ 60fps（WASM）	≥ 500	尚未严格测试（欢迎贡献）
粒子数量 @ 30fps（仅边界）	≥ 5000	未测试
性能测试需在目标设备上进行。当前实现已针对单线程做了基本优化。

📄 版权与许可
© 2026 PhysiCodeLab 作者。保留所有权利。

本仓库的代码仅供展示与学习用途。未经作者明确书面许可，禁止任何形式的复制、修改、重新分发或商业使用。

若您希望获得使用许可，请通过 GitHub Issues 联系作者。

为什么不选开源许可证？—— 我们希望在项目早期保护核心算法与实现细节，待 API 稳定、文档齐全后再考虑开源。当前保留所有权利是最严谨的保护方式。

🤝 贡献指南
目前项目处于个人开发阶段，暂不接受外部代码贡献。但非常欢迎您提交 Bug 报告 或 功能建议（通过 Issues）。我们会在后续版本中评估您的想法。

📞 联系方式
QQ:3440319476
作者邮箱：3440319476@qq.com

🗓️ 版本历史
v0.6.0 (2026-04-02)

首次公开演示版本

实现完整的 WebAssembly 绑定与前端交互

支持粒子添加、重置世界、数据导出

完成单元测试（自由落体、弹性碰撞、惯性张量等）

（后续版本记录会在此更新）

🙏 致谢
Eigen — 线性代数库

Bullet Physics — 碰撞检测引擎

Emscripten — WebAssembly 编译工具链

PhysiCodeLab — 让物理仿真变得简单、高效、可嵌入。
