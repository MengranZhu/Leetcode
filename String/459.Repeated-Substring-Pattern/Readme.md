### 459.Repeated-Substring-Pattern

#### 解法1
暴力尝试所有的循环节长度，查看是否是否每段循环节长度的区间都相等。

#### 解法2
我们计算字符串s的后缀数组suf[i]，得到长度len = suf[n-1]，这是s最长的前缀字符串使得期恰好也等于s的后缀字符串。

如果我们得到了这样的len，会有什么性质呢？如下图，我们画出最长前缀/后缀字符串的范围（星号）
```
s: [* * *] [* * *] [* * *] [_____]
      A           B           C
      
s: [_____] [* * *] [* * *] [* * *]
      D           E           F

```
由最长前缀/后缀字符串的关系，我们知道B的后缀等于F.同时由于B的后缀就是E的后缀，所以得到E的后缀等于F。我们可以类似地一路向前推进。所以只要A+B的长度是F的长度（也就是n-len）的整数倍，那么A+B（或者D+E）就有F作为循环节。因此判断存在循环节的充要条件是```len>0 && （n-len）%len==0```.

另外，n-len也一定是最小循环节长度。
