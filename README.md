# 🧬 SM-ISA — Stream-Matrix Instruction Set Architecture

**新一代指令集架构**：矩阵计算是一等公民，数据搬运用流指令。

```
核心思想: 传统 ISA 为"取指-执行"设计, 每数据 1 条 load/store
         SM-ISA 把计算粒度提升到 tile, 数据用流搬运

传统 ISA: 512 次乘加 = ~1050 条指令
SM-ISA:   512 次乘加 = 9 条指令 (116× 压缩)
```

---

## 🎯 设计哲学

AI 时代计算瓶颈不是"算"，是**喂数据**。传统 ISA 每笔数据搬运都是一条指令、一个周期、一次功耗（数据通路功耗可占 70%）。

SM-ISA 的三个设计支柱：

| 支柱 | 指令 | 意义 |
|------|------|------|
| **矩阵原生** | `MTILE Td, Ts1, Ts2` | 1 条指令 = 8×8×8 矩阵乘 (512 MAC) |
| **流式搬运** | `SLOAD/SSTORE` | 地址自动递增, 数据"流"过计算单元, 零显式寻址 |
| **极简标量** | `LOADI/ADD/SUB/BZ` | 只管控制和地址, 不碰数据通路 |

---

## 📐 版本历史

| 版本 | 指令宽度 | tile | 特性 | 验证 |
|------|---------|------|------|------|
| **v0.1** | 16 位 | 4×4 | 最小可行: 矩阵乘是语言一等公民 | 4×4 矩阵乘 ✅ (14× 压缩) |
| **v0.2** | 32 位 | 8×8 | 流指令 SLOAD/SSTORE + 对齐 BF16 阵列 | 8×8 矩阵乘 + 流式 ✅ (116× 压缩) |

---

## 🚀 快速开始

```bash
# v0.1 模拟器 (16 位 ISA, 4×4 tile)
python3 sim/sm_sim.py

# v0.2 模拟器 (32 位 ISA, 8×8 tile + 流指令)
python3 sim/sm_sim_v2.py
```

自检输出（v0.2）：
```
8×8 矩阵乘 trial 0: 相对误差 0.284% ✅   (BF16 7 位尾数精度极限)
流式连续乘: 相对误差 0.212% ✅
流指针: R0 0→128→256→384→512 自动递增
验证: 全部通过 ✅
```

---

## 📖 指令集速览 (v0.2, 32 位定长)

```
[31:24] opcode | [23:16] 寄存器 | [15:0] 立即数

0x01 LOADI Rd, imm16      0x02 ADD Rd, Rs1, Rs2
0x03 SUB Rd, Rs1, Rs2     0x04 MZERO Td
0x05 MTILE Td, Ts1, Ts2   0x06 MLOAD Td, [Rs]
0x07 MSTORE [Rs], Ts      0x08 MADD Td, Ts
0x09 SLOAD Td, [Rs]       0x0A SSTORE [Rs], Ts
0x0B JMP imm16            0x0C BZ Rs, imm16
0x0D HLT
```

寄存器：8 标量 (R0-R7) + 4 个 8×8 tile (T0-T3, 各 64×BF16 = 128 字节)

---

## 📁 目录结构

```
docs/
  sm-isa-v01-spec.md     v0.1 规范 (16 位最小版)
  sm-isa-v02-spec.md     v0.2 规范 (流指令 + 8×8 tile)
sim/
  sm_sim.py              v0.1 汇编器+模拟器+自检
  sm_sim_v2.py           v0.2 汇编器+模拟器+自检
examples/
  matmul.hex             v0.1 4×4 矩阵乘程序
```

---

## 🗺️ 路线图

```
P1 ✅ v0.1 最小验证 (4×4 tile, 14× 压缩)
P2 ✅ v0.2 流指令 + 8×8 tile (116× 压缩)
P3 ⏳ Verilog 硬件实现 (解码器 + tile 执行单元 + 流引擎)
P4 ⏳ 编译器 (C 子集 → SM-ISA)
```

---

## 🔗 相关

- [AI-Accel](https://github.com/ljsysfurryACE/AI-Accel) — 加速器硬件 (BF16 阵列, 8×8 tile 对齐本 ISA)
- 全链路自研: CPU (PicoRV32) + OS (纸鸢微内核) + ISA (SM-ISA) + 游戏 (贪吃蛇)
