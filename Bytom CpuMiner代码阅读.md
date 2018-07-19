# Bytom Miner代码阅读
  先了解一下Bytom的mine流程，这个是一个理解Bytom挖矿流程的很好参考例子。这个因为是cpu跑的，速度不是很好，里面应该只是为了调试使用，这些系列说写一个
  miner就是使用更高效的GPU方式实现。相对SOLO挖矿，GPU方式实现的是对接矿池挖矿，最大保证了矿工收益。SOLO和矿池挖矿的差别可自行网上了解。
  Bytom的miner代码在 [cmd/miner/main.go](https://github.com/Bytom/bytom/blob/master/cmd/miner/main.go),我们看下main函数，代码不多，如下：
  ```golang
  func main() {
	for true {
		data, _ := util.ClientCall("/get-work", nil) //SOLO 方式从本地node获取work
		if data == nil {
			os.Exit(1)
		}
		rawData, err := json.Marshal(data)
		if err != nil {
			log.Fatalln(err)
		}
		resp := &api.GetWorkResp{}
		if err = json.Unmarshal(rawData, resp); err != nil {
			log.Fatalln(err)
		}

		log.Println("Mining at height:", resp.BlockHeader.Height)
		if lastHeight != resp.BlockHeader.Height {
			lastNonce = ^uint64(0)
		}
		if doWork(resp.BlockHeader, resp.Seed) {  //使用work获取到的2参数调用doWork计算挖矿，如果找到返回了ture
			//提交计算结果获得奖励
			util.ClientCall("/submit-work", &api.SubmitWorkReq{BlockHeader: resp.BlockHeader})
			getBlockHeaderByHeight(resp.BlockHeader.Height)
		}

		lastHeight = resp.BlockHeader.Height
		if !isCrazy {
			return
		}
	}
}
```
main函数很简单，就3部，获取work,计算，找到结果提交。重点在doWrok的计算，这里的运算决定的挖矿性能。
```golang
// do proof of work
func doWork(bh *types.BlockHeader, seed *bc.Hash) bool {
	log.Println("Start from nonce:", lastNonce+1)
	for i := uint64(lastNonce + 1); i <= uint64(lastNonce+consensus.TargetSecondsPerBlock*esHR) && i <= maxNonce; i++ {
		bh.Nonce = i //每一次计算需要需要改变Nonce的值，如果对接矿池这个起步Nonce有矿池提供
		// log.Printf("nonce = %v\n", i)
		headerHash := bh.Hash()//计算出bh的Hash值
		if difficulty.CheckProofOfWork(&headerHash, seed, bh.Bits) {//使用Hash和seed Bits 三个参数调用计算，里面会调用到一次
		//Tensority算法，Tensority算法是矩阵计算，比较复杂，后面专门讲。
			log.Printf("Mining succeed! Proof hash: %v\n", headerHash.String())
			return true
		}
	}
	log.Println("Stop at nonce:", bh.Nonce)
	lastNonce = bh.Nonce
	return false
}
```
difficulty.CheckProofOfWork在 [consensus/difficulty/difficulty.go](https://github.com/Bytom/bytom/blob/8ae1695ca9807ef802a6ecbec63893c78c85188e/consensus/difficulty/difficulty.go)
其中
```golang
// CheckProofOfWork checks whether the hash is valid for a given difficulty.
func CheckProofOfWork(hash, seed *bc.Hash, bits uint64) bool {
	//调用tensority算法计算一次，获取到Hash
	compareHash := tensority.AIHash.Hash(hash, seed)
	//Hash和目标比较需要小于才算找到结果，不对就继续计算其他Nonce
	return HashToBig(compareHash).Cmp(CompactToBig(bits)) <= 0 
}
```

很简单的一个miner，流程就大概如此。但这个只是实验室的调试使用，目前实际上想用它去挖矿获取btm是不可能的。基于这个流程的了解，如果开发一个GPU miner
需要怎么做？
大概需要做到实现：
- 对接矿池，使用矿池的通信协议，获取work,就是这里的getWrok。
- GPU实现tensority算法，计算结果，对应dowok中的tensority.AIHash.Hash(hash, seed)。
- 提交找到符合条件的Nonce。

本篇完。

