# 400-第 N 位数字[中等]

---

> <https://leetcode.cn/problems/nth-digit/description>

#剑指Offer #数字相关

---

## 常规解法

这道题的核心核心解法就是**按数字位数分段处理**。

我们需要把序列按数字的位数划分成若干区间：

- **1 位数**：1 ~ 9，共 9 个数，占 $9 \times 1 = 9$ 个位数。

- **2 位数**：10 ~ 99，共 90 个数，占 $90 \times 2 = 180$ 个位数。

- **3 位数**：100 ~ 999，共 900 个数，占 $900 \times 3 = 2700$ 个位数。

- **$k$ 位数**：共 $9 \times 10^{k-1}$ 个数，占 $9 \times 10^{k-1} \times k$ 个位数。

### 解题步骤

1. **定位位数 ($len$)**：不断用 $n$ 减去当前位数所占的总位数，确定目标数字落在几位数区间。

2. **定位目标具体是哪个数 ($num$)**：求出它具体落在哪一个数字上。

3. **定位目标在数中的哪一位 ($digit$)**：求出目标字符是在该数字中的第几位（从左往右数）。

### Java 代码实现

```Java
class Solution {
    public int findNthDigit(int n) {
        int len = 1;          // 当前数字的位数（1位数、2位数……）
        long count = 9;       // 当前位数区间总共包含的“数字位数总和”
        long start = 1;       // 当前位数区间的起始数字（1, 10, 100, ...）

        // 1. 确定目标数字落在哪种位数的区间内
        while (n > count) {
            n -= count;
            len++;
            start *= 10;
            count = start * 9 * len; // 防止溢出，用 long 类型
        }

        // 2. 确定目标数字具体落在哪个数上
        // (n - 1) 是因为 start 本身就是第 1 个数
        long num = start + (n - 1) / len;

        // 3. 确定是 num 中的第几位数字
        int digitIndex = (n - 1) % len;

        // 提取该位置上的数字
        return String.valueOf(num).charAt(digitIndex) - '0';
    }
}
```

> `count = start * 9 * len`
> 当前区间数字的个数正好等于 start \* 9，也就是这一段的数有9、90、900...

### 关键注意事项

- **防溢出**：提示中 $n$ 最大为 $2^{31} - 1$（约 21 亿）。在计算 `count = start * 9 * len` 时可能会超出 `int` 的最大范围，因此 `count` 和 `start` 必须使用 **`long`** 类型。

- **索引对齐**：使用 `(n - 1) / len` 和 `(n - 1) % len` 可以极大地简化边界偏移（从 0-based 索引处理）。
