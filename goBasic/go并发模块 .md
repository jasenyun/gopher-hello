# Go 中的 select 语句

## 用法

多个通道 Channel 中信息的发送和接受处理的专用的语句—select 语句。select  语句会阻塞，直到其中的一个发送/接收操作准备好。select 语句和 switch 语句有点相似，但 select 语句在被执行时会选择执行其中的一个分支，且选择分支的方法完全是不相同的。

```go
ch1 = make(chan string)
ch2 = make(chan string)

ch1 <- "server1"
ch2 <- "server1"

select {
    case i := <- ch1:
      fmt.Printf("从ch1读取了数据%d", i)
    case j := <- ch2:
      fmt.Printf("从ch2读取了数据%d", i)
    default:
      fmt.Printf("no action...", i)
}
```

以上代码中，每个 case 后都只针对某个通道的接收语句，这个和 switch 不同，也没有 break。switch 语句右边是一个switch 表达式，但 select 右边是接大括号。

开始执行 select 语句时，所有跟在 case 关键字右边的表达式都会被求值，求值的顺序是自上而下，从左到右的。

## 使用场景

### 实现收发功能

select 是控制 channel 必不可少的部分，channel 的主要功能就是收发信息，基于此可以设计一个生产者消费者功能。生产者发送消息，消费者接受消息

```go
func main(){
// 生产数据，将数据写入 channel 
  n1 := make(chan int)
    go func() {
        i := 0
        for {
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            n1 <- i
            i++
        }
    }()
    n2 := make(chan int)
    go func() {
        i := 0
        for {
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            n2 <- i
            i++
        }
    }()

// 从 channel 中读取到数据就输出
    for {
        select {
        case n := <-n1:
            fmt.Printf("从ch1读取了数据%d", n)
        case n := <-n2:
            fmt.Printf("从ch1读取了数据%d", n)
        }
    }
}
```

## 注意事项

- select 只能用于 chan 的 IO 操作

- select 的 case 都是并行的，case 读取到数据就执行，但是如果没有读取到且未设置 default 将导致阻塞

- 尽量设置 default 避免没有 IO 操作发生时，select 语句一直阻塞，直到某个 case 分支命中

- 如果是空的 select 有可能会引起死锁,所以在 select 执行过程中，必须命中某一 case 分支

```go
select {}
```

- 防止阻塞还有一个方法：设置超时
