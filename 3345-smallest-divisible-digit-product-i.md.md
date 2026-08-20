# 3345-最小可整除数位乘积 I [简单]

> <https://leetcode.cn/problems/smallest-divisible-digit-product-i>

---

## 常规解法

**直接从 $n$ 开始递增遍历测试就是最优解。**

因为题目给出的数据范围非常小（$n \le 100$，$t \le 10$），直接暴力遍历不仅写起来最简单，而且在时间复杂度上已经是 $O(1)$。

### 为什么直接遍历是最优方法？

1. **最多只需要尝试 10 次：**

   在任意连续的 10 个整数中，必然存在一个个位数是 `0` 的数字（例如 10, 20, 100 等）。

2. **数位乘积必定为 0：**

   末尾为 `0` 的数字，其各个数位相乘的结果一定是 `0`。

3. **0 能被任何 $t$ 整除：**

   对于任何 $1 \le t \le 10$，$0 \pmod t == 0$ 恒成立。

因此，循环**最多执行 10 次**就必定能找到答案，完全不需要复杂的数学公式或剪枝策略。

### 代码实现 (Java)

```Java
class Solution {
    public int smallestNumber(int n, int t) {
        while (true) {
            if (getDigitProduct(n) % t == 0) {
                return n;
            }
            n++;
        }
    }

    // 计算数字各数位的乘积
    private int getDigitProduct(int num) {
        int prod = 1;
        while (num > 0) {
            prod *= num % 10;
            num /= 10;
        }
        return prod;
    }
}
```

### 复杂度分析

- **时间复杂度：** $O(1)$。循环最多执行 10 次，每次计算数位乘积最多处理 3 位数字（$n \le 100$），耗时为常数级。
- **空间复杂度：** $O(1)$。仅使用常数个额外变量。

## 如果 n 的范围扩大到 10^9 时该怎么优化？

即使将 $n$ 扩大到 $10^9$，**直接递增遍历依然是最优解，不需要做任何复杂优化**。

### 情况一：规则不变（允许数位乘积为 0）

只要题目允许数位乘积为 $0$（且 $0 \pmod t == 0$），该逻辑对任意大小的 $n$ 均成立：

- 在 $[n, n + 9]$ 这连续 10 个整数中，**必然存在一个末尾为 `0` 的数字**（例如 $1,234,567,890$）。
- 该数字的各数位乘积必然为 `0`。
- `0` 可以被任意正整数 $t$ 整除。

因此，无论 $n$ 是 $100$ 还是 $10^9$，`while` 循环**最多只会执行 10 次**。计算 10 位数的乘积仅需 10 次基本运算，整体时间复杂度为 $O(\log_{10} n)$，实际耗时不足 1 微秒。

### 情况二：附加条件“数位乘积必须大于 0”且 $t$ 较大

如果题目增加限制，要求**数位乘积不能为 0**（即不能含有数字 `0`），此时不能靠 `0` 兜底，就需要采用 **质因数剪枝 + 数位回溯（DFS）**：

#### 1. 质因数分解剪枝

1 到 9 的数字分解质因数后，仅包含 **2、3、5、7** 四种质因子。

- 如果 $t$ 含有除 2、3、5、7 以外的质因子（如 11、13、17 等），则不可能凑出非零的乘积，直接无解。
- 否则，将 $t$ 化简为所需质因子的数量要求：$t = 2^{a} \times 3^{b} \times 5^{c} \times 7^{d}$。

#### 2. 数位回溯搜索（DFS）

从高位到低位逐位确定答案：

- **前缀匹配**：尽量与 $n$ 的高位保持一致；若某一位填了比 $n$ 对应的位更大的数字，后面的位就可以从 `1` 开始贪心选择。
- **状态记录**：递归时维护当前各数位乘积中 $2, 3, 5, 7$ 质因子的累积数量。
- **终止条件**：当填满位数且累积的质因子数量满足 $t$ 的要求时，返回该数值；利用 DFS 优先搜索较小数字的特性，第一个找到的合法解即为最小值。

```Java
// 伪代码思路（非零乘积 + 大 t 场景）
public int smallestNumberNoZero(int n, int t) {
    // 1. 检查 t 是否只含 2,3,5,7 质因子，若含其他质因子则直接无解
    // 2. 将 n 转为字符数组/数字数组
    // 3. 从高位到低位进行 DFS 贪心搜索
    // 4. 找到第一个满足乘积能被 t 整除且大于等于 n 的无零数字
}
```

#### 核心算法思路

1. **质因数可行性检查**：

   数字 `1~9` 的质因数分解只包含 **2, 3, 5, 7**。如果 $t$ 含有其他质因素（如 11、13 等），直接返回无解 (`-1`)。

2. **可行性剪枝函数 (`canFulfill`)**：

   在 DFS 过程中，计算剩余的位数 `remLen` 是否**足够**凑齐还需要的 $2, 3, 5, 7$ 因子数量。若位数不足直接剪枝（回溯），极大减少搜索空间。

3. **同长度 DFS 搜索**：

   优先尝试与 $n$ 同位数的解，从高位到低位依次枚举数字 `1~9`（若原位置是 `0`，则强制填 `1` 并标记已大于 $n$）。

4. **跨长度贪心构造**：

   若同长度无解，则增加位数。位数增加后，生成的数必然大于 $n$，只需按照“字典序最小”原则逐位贪心选择最小的有效数字即可。

#### 完整 Java 代码实现

```Java
public class Solution {

    public String smallestNumberNoZero(long n, long t) {
        // 1. 分解 t 的质因数 (仅允许包含 2, 3, 5, 7)
        int[] req = new int[4]; // 分别代表 2, 3, 5, 7 的所需数量
        long tempT = t;
        int[] primes = {2, 3, 5, 7};
        for (int i = 0; i < 4; i++) {
            while (tempT % primes[i] == 0) {
                req[i]++;
                tempT /= primes[i];
            }
        }
        if (tempT > 1) {
            return "-1"; // 包含 2,3,5,7 以外的质因子，无解
        }

        String s = String.valueOf(n);
        int len = s.length();

        // 2. 优先尝试与 n 同长度的解
        String sameLenRes = dfs(0, false, req[0], req[1], req[2], req[3], s, new StringBuilder());
        if (sameLenRes != null) {
            return sameLenRes;
        }

        // 3. 同长度无解，增加位数构造最小解
        int minLen = req[2] + req[3] + getMinDigits23(req[0], req[1]);
        int targetLen = Math.max(len + 1, minLen);

        return buildSmallest(targetLen, req[0], req[1], req[2], req[3]);
    }

    // DFS 搜索同长度解
    private String dfs(int idx, boolean isGreater, int c2, int c3, int c5, int c7, String s, StringBuilder sb) {
        int remLen = s.length() - idx;
        // 剪枝：剩余位数不足以凑齐所需的质因数
        if (!canFulfill(remLen, c2, c3, c5, c7)) {
            return null;
        }
        if (idx == s.length()) {
            return sb.toString();
        }

        int startDigit = 1;
        if (!isGreater) {
            startDigit = s.charAt(idx) - '0';
            if (startDigit == 0) { // 原数为 0 时，不能填 0，填 1 必然大于原数
                startDigit = 1;
                isGreater = true;
            }
        }

        for (int d = startDigit; d <= 9; d++) {
            boolean nextIsGreater = isGreater || (d > s.charAt(idx) - '0');
            sb.append(d);

            int[] f = getDigitFactors(d);
            String res = dfs(idx + 1, nextIsGreater, c2 - f[0], c3 - f[1], c5 - f[2], c7 - f[3], s, sb);
            if (res != null) {
                return res;
            }

            sb.deleteCharAt(sb.length() - 1); // 回溯
        }
        return null;
    }

    // 贪心构造指定长度的最小合规数
    private String buildSmallest(int targetLen, int c2, int c3, int c5, int c7) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < targetLen; i++) {
            int remLen = targetLen - 1 - i;
            for (int d = 1; d <= 9; d++) {
                int[] f = getDigitFactors(d);
                if (canFulfill(remLen, c2 - f[0], c3 - f[1], c5 - f[2], c7 - f[3])) {
                    sb.append(d);
                    c2 -= f[0]; c3 -= f[1]; c5 -= f[2]; c7 -= f[3];
                    break;
                }
            }
        }
        return sb.toString();
    }

    // 判断剩余长度是否足够填充剩余质因数
    private boolean canFulfill(int remLen, int c2, int c3, int c5, int c7) {
        if (remLen < 0) return false;
        c5 = Math.max(0, c5);
        c7 = Math.max(0, c7);
        return (c5 + c7 + getMinDigits23(c2, c3)) <= remLen;
    }

    // 计算凑齐 2 和 3 所需的最少数字个数 (利用 9, 8, 6, 4 优化)
    private int getMinDigits23(int c2, int c3) {
        c2 = Math.max(0, c2);
        c3 = Math.max(0, c3);
        int minDigits = Integer.MAX_VALUE;
        // 枚举使用数字 '6' (同时消耗 1 个 2 和 1 个 3) 的数量
        for (int k6 = 0; k6 <= Math.min(c2, c3); k6++) {
            int r2 = c2 - k6;
            int r3 = c3 - k6;
            int d9 = (r3 + 1) / 2; // 使用 9 (两个 3)
            int d8 = (r2 + 2) / 3; // 使用 8 (三个 2)
            minDigits = Math.min(minDigits, k6 + d9 + d8);
        }
        return minDigits;
    }

    // 数字 1-9 对应的质因子 (2, 3, 5, 7) 分解表
    private int[] getDigitFactors(int d) {
        switch (d) {
            case 2: return new int[]{1, 0, 0, 0};
            case 3: return new int[]{0, 1, 0, 0};
            case 4: return new int[]{2, 0, 0, 0};
            case 5: return new int[]{0, 0, 1, 0};
            case 6: return new int[]{1, 1, 0, 0};
            case 7: return new int[]{0, 0, 0, 1};
            case 8: return new int[]{3, 0, 0, 0};
            case 9: return new int[]{0, 2, 0, 0};
            default: return new int[]{0, 0, 0, 0}; // d = 1
        }
    }
}
```

#### 复杂度分析

- **时间复杂度**：$O(\log_{10} n)$。DFS 的分支极度依赖 `canFulfill` 剪枝，由于质因子组合要求非常严格，实际搜索树节点数极少，可在数十微秒内返回结果。

- **空间复杂度**：$O(\log_{10} n)$。主要为递归栈和字符串构造所占用的空间。
