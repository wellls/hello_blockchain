## 什么是区块链

区块链本质上是一个公开的数据库，而不是一个私人的数据库，每个使用它的人都有一个完整或者部分的副本。只有经过其他“数据库管理员”的同意，才能向数据库中添加新的记录。

---

### 区块

区块链中，真正存储有效信息的是区块(block)。除此之外，区块还包含了一些技术实现相关的信息比如版本，当前时间戳和前一个区块的哈希。

---

### Hash

保证区块链安全

``` golang
Hash = SHA256(PrevBlockHash + Timestamp + Data)
```

将 `Block` 中的 `Timestamp`, `Data`, `PrevBlockHash` 字段相互拼接起来，然后在拼接后的结果上计算一个 SHA-256 ，就得到了 Hash 。

``` golang
// SetHash
func (b *Block) SetHash() {
    timestamp := []byte(strconv.FormatInt(b.Timestamp, 10))
    headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
    hash := sha256.Sum256(headers)

    b.Hash = hash[:]
    }
```

``` golang
// NewBlock
func NewBlock(data string, prevBlockHash []byte) *Block{
    block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}}
    block.SetHash()
    return block
    }
```

---

### First blockchain

```golang
type Blockchain struct {
    blocks []*Block
}
```

### Add a blockchain

```golang
func (bc *Blockchain) AddBlock(data string) {
    prevBlock := bc.blocks[len(bc.blocks)-1]
    newBlock := NewBlock(data, prevBlock.Hash)
    bc.blocks = append(bc.blocks, newBlock)
    }
```

### genesis block

```golang
func NewGenesisBlock() *Block {
    return NewBlock("Genesis Block", []byte{})
    }
```

```golang
// new a genesis block
func NewBlockchain() *Blockchain {
    return &Blockchain{[]*Block{NewGenesisBlock()}}
    }
```


### Main function
```golang
func main() {
    bc := NewBlockchain()

    bc.AddBlock("Send 1 BTC to Ivan")
    bc.AddBlock("Send 2 more BTC to Ivan")

    for _, block := range bc.blocks {
        fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
        fmt.Printf("Data: %s\n", block.Data)
        fmt.Printf("Hash: %x\n", block.Hash)
        fmt.Println()
        }
    }
```

---
---

## 工作量证明

你必须要经过十分繁重的计算，来获得添加一个新块的权力。

##### Hashcash
1. 取一些公开的数据，如邮箱或者是区块头
2. 给公开数据加一个计数器。默认从0开始
3. 将 data(数据) 和 counter(计数器) 组合到一起，获得一个哈希。
4. 检查哈希是否符合一定的条件
   1. 符合，结束过程
   2. 不符合, 增加计数器，重复 3-4 步骤(递归)。

## 共识

要加入一个新块，必须要被网络的其他参与者确认和同意
