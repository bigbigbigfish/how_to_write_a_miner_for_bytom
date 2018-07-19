# Bytom Miner代码阅读
  先了解一下Bytom的mine流程，这个是一个理解Bytom挖矿流程的很好参考例子。这个因为是cpu跑的，速度不是很好，里面应该只是为了调试使用，这些系列说写一个miner就是使用更高效的GPU方式实现。相对SOLO挖矿，GPU方式实现的是对接矿池挖矿，最大保证了矿工收益。SOLO和矿池挖矿的差别可自行网上了解。
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
			util.ClientCall("/submit-work", &api.SubmitWorkReq{BlockHeader: resp.BlockHeader})//提交计算结果获得奖励
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
		bh.Nonce = i
		// log.Printf("nonce = %v\n", i)
		headerHash := bh.Hash()
		if difficulty.CheckProofOfWork(&headerHash, seed, bh.Bits) {
			log.Printf("Mining succeed! Proof hash: %v\n", headerHash.String())
			return true
		}
	}
	log.Println("Stop at nonce:", bh.Nonce)
	lastNonce = bh.Nonce
	return false
}
```
