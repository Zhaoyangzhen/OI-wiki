在介绍 Hash 算法之前，首先你需要了解关于 [字符串匹配](/string/match) 的事情。

## Hash 的思想

Hash 的核心思想在于，暴力算法中，单次比较的时间太长了，应当如何才能缩短一些呢？

如果要求每次只能比较 $O(1)$ 个字符，应该怎样操作呢？

是比较第一个？最后一个？随便选一个？求所有字符的和？

我们定义一个把 string 映射成 int 的函数 $f$，这个 $f$ 称为是 Hash 函数。

我们希望这个函数 $f$ 可以方便地帮我们判断两个字符串是否相等。

具体来说，我们希望在 Hash 函数值不一样的时候，两个字符串一定不一样。

另外，反过来不需要成立。我们把这种条件称为是单侧错误。

我们需要关注的是什么？

时间复杂度和 Hash 的准确率。

通常我们采用的是多项式 Hash 的方法，即 $\operatorname{f}(s) = \sum s[i] \times b^i \pmod M$

这里面的 $b$ 和 $M$ 需要选取得足够合适才行，以使得 Hash 的冲突尽量均匀。

如果 $b$ 和 $M$ 互质，在输入随机的情况下，这个 Hash 函数在 $[0,M)$ 上每个值概率相等

此时错误率为 $\frac1M$ （单次比较）

在输入不是随机的情况下，效果也很好。

## Hash 的实现

伪代码：

```
match_pre(int n) {
    exp[0] = 1;
    for (i = 1; i < n; i++) {
        exp[i] = exp[i - 1] * b % M;
    }
}

match(char *a, char *b, int n, int m) {
    // match 函数返回：长度为 m 的串 b 在长度为 n 的串 a 中的匹配位置
    // hash(a, m) 函数用来获得某个字符串前 m 个字符的部分的 hash 值
    ans = new vector();
    int ha = hash(a, m);
    int hb = hash(b, m);
    for (i = 0; i < n - m + 1; i++) {
        if ((ha - hb * exp[i]) % M == 0) {
            ans.push_back(i);
        }
        ha = (ha - a[i] * exp[i] + a[i + m] * exp[i + m]) % M;
    }
    return ans;
}
```

通过上面这段代码，可以发现，每次直接计算 Hash 是 $O(串长)$ 的

## Hash 的分析与改进

改进时间复杂度或者错误率

在上述例子中，时间复杂度已经是 $O(n+m)$

我们来分析错误率

由于 $n >> m$，要进行约 $n$ 次比较，每次错误率 $\frac1{M}$ ，那么总错误率是？

先补充一些随机数学的知识（非严格地）

现在考虑这 $n$ 次比较，如果看成独立的，总错误率 $1-(1-1/M)^n$

当 $M >> n$ 时，总错误率接近于$\frac{n}{M}$

当 $M = n$ 时，接近于 $1-\frac{1}{e} (≈0.63)$

如果不是独立的，最坏情况也就是全部加起来，等于 $\frac{n}{M}$

要改进错误率，可以增加 $M$

选取一个大的 $M$，或者两个互质的小的 $M$

时间复杂度不变，单次错误率平方

## Hash 的应用

不只是字符串中，在其他情况也可以用。

假设有个程序要对 $10^{18}$ 大小的数组进行操作，保证只有 $10^6$ 个元素被访问到

由于存不下，我们在每次操作时，对下标取 Hash 值（比如，直接 $\bmod M$），然后在 $M$ 大小的数组内操作

如果冲突了怎么办？

方案1：尝试找下一个位置，下下个位置（优点：速度快；缺点：删除麻烦）

方案2：在数组的每个位置直接挂一个链表
