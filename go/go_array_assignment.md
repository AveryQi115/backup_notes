# Go array 和 slice 的复制和遍历

### 赋值

- array的复制

  ```go
  x := [3]int{2,4,5}
  y := x
  
  // x[0]不变，依然等于2
  y[0] = -10
  
  // not the same
  fmt.Printf("%v %v",&x,&y)
  
  // not the same
  fmt.Printf("%v %v",&x[0],&y[0])
  ```

- slice的复制

  ```go
  // x是一个切片！
  x := []int{2,4,5}
  // y相当于开了一个新空间，复制了x指针
  y := x
  
  // x[0] == -10 因为它们都指向一个切片空间
  y[0] = -10
  
  // not the same 因为是两个切片变量
  fmt.Printf("%v %v",&x,&y)
  
  // same 指向同一个切片空间
  fmt.Printf("%v %v",&x[0],&y[0])
  ```

### 遍历

- array的遍历

  ```go
  x := [3]int{2,4,5}
  
  for key,val := range x{
    // val相当于是把x中每个元素都复制了一遍
    // not the same
    // &val一直不变，始终分配的是那个堆栈空间局部变量
    fmt.Printf("%v %v",&val,&x[key])
  }
  ```

- slice的遍历

  ```go
  x := []int{2,4,5}
  
  for key,val := range x{
    // val相当于是把x切片空间中每个元素都复制了一遍
    // not the same
    // &val一直不变，始终分配的是那个堆栈空间局部变量
    fmt.Printf("%v %v",&val,&x[key])
  }
  ```



碎碎念：要一直没发现就相当于每次遍历都用了一个新的令牌桶

垃圾代码生产者...我

