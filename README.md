# how_to_write_a_miner_for_bytom
### 怎样写一个Miner挖Bytom
大概从4月底开始，就着手打算写一个Bytom的GPU miner。一路开始阅读Bytom miner\tensority算法\矿池通信协议等代码。一边学习golang，虽然16年就开始学了golang,但一直是在入门的边缘。miner的GPU版本目前离不开2大卡，Nidia、AMD 这2家，这个过程也算了解了一个各家的编程方式OpenCL和Cuda，但是我的开发环境只有一个Nidia的1060显卡，刚开始Opencl的版本，为了2家显卡都兼容，但是在Nidia环境不理想没有经济效益，后来又去研究Cuda版本。前后到了7月份都是业余时间折腾。
### 目录
- Bytom cpuminer代码笔记
- Tensority算法代码阅读
- Miner和矿池的通信协议
- Miner的参考实现
  1. OpenCL版本
  2. Cuda版本
