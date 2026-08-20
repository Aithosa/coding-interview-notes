# 3348-最小可整除数位乘积 II [困难]

---

> <https://leetcode.cn/problems/smallest-divisible-digit-product-ii/description/>

[[3345-smallest-divisible-digit-product-i.md]]

状态：暂时放弃

---

## 常规解法

这道题（LeetCode 3348）与上一题有极强的递进关系，正是上一题讨论的“数位乘积不能为 0 且数据量极大”的困难版。

- **和前一题的关系**：它是前一题的“彻底终极版”。

- **能不能一个一个试？**：**绝对不能**。`num` 的长度最高达 2×105（即有 20 万位），直接递增遍历会产生高达 10200000 级别的计算量，会导致超时（TLE）。

必须使用 **质因数分解 + 贪心前缀匹配 + 数位构造** 的 $O(N)$ 复杂度算法。

### 解题核心思路

1. **质因数校验**： 数字 `1~9` 的质因子只有 **2, 3, 5, 7**。将 t 分解，若包含其他质数（如 11、13），直接返回 `"-1"`。

2. **寻找最长匹配前缀**： 从原字符串 `num` 的尾部向前枚举保留的前缀长度 i。
   - 尝试将第 i 位的数字变大（从 `num[i] + 1` 到 `9`）；

   - 检查剩余的长度能否凑齐所需质因子 2,3,5,7 的数量；

   - 一旦找到第一个可行的位置 i 和数字 d，即找到了紧大于原数的答案前缀。

3. **后缀贪心填充**： 确定前缀后，后续位置从左到右依次贪心选择**能满足质因数条件的最小数字**（`1~9`）。

4. **长度扩充**： 若同等长度无解，说明答案位数必须增加。此时直接构造一个长度为 max(N+1,凑齐 t 所需最小位数) 的数字，并从左到右全填最小有效数字。

### 代码实现 (Java)

按照题目要求，代码中已定义变量 `vornitexis` 来中间存储输入 t：

```Java
class Solution {
    public String smallestNumber(String num, long t) {
        // 中间变量存储输入
        long vornitexis = t;

        // 1. 分解 t 的质因数 (2, 3, 5, 7)
        long tempT = vornitexis;
        int c2 = 0, c3 = 0, c5 = 0, c7 = 0;
        while (tempT % 2 == 0) { c2++; tempT /= 2; }
        while (tempT % 3 == 0) { c3++; tempT /= 3; }
        while (tempT % 5 == 0) { c5++; tempT /= 5; }
        while (tempT % 7 == 0) { c7++; tempT /= 7; }
        if (tempT > 1) return "-1"; // 包含 2,3,5,7 以外的质因子

        int n = num.length();
        int zeroIdx = num.indexOf('0');
        int limit = (zeroIdx == -1) ? n : zeroIdx;

        // 2. 前缀质因数统计
        int[] pref2 = new int[n + 1];
        int[] pref3 = new int[n + 1];
        int[] pref5 = new int[n + 1];
        int[] pref7 = new int[n + 1];

        for (int i = 0; i < limit; i++) {
            int d = num.charAt(i) - '0';
            int[] f = getFactors(d);
            pref2[i + 1] = pref2[i] + f[0];
            pref3[i + 1] = pref3[i] + f[1];
            pref5[i + 1] = pref5[i] + f[2];
            pref7[i + 1] = pref7[i] + f[3];
        }

        // 检查 num 本身是否已经合法
        if (zeroIdx == -1 && pref2[n] >= c2 && pref3[n] >= c3 && pref5[n] >= c5 && pref7[n] >= c7) {
            return num;
        }

        // 3. 从右向左尝试改变第 i 位数字
        for (int i = limit - 1; i >= 0; i--) {
            int remLen = n - 1 - i;
            int startDigit = num.charAt(i) - '0' + 1;

            int cur2 = pref2[i];
            int cur3 = pref3[i];
            int cur5 = pref5[i];
            int cur7 = pref7[i];

            for (int d = startDigit; d <= 9; d++) {
                int[] f = getFactors(d);
                int req2 = c2 - cur2 - f[0];
                int req3 = c3 - cur3 - f[1];
                int req5 = c5 - cur5 - f[2];
                int req7 = c7 - cur7 - f[3];

                if (canFulfill(remLen, req2, req3, req5, req7)) {
                    StringBuilder sb = new StringBuilder();
                    sb.append(num.substring(0, i));
                    sb.append(d);
                    sb.append(buildGreedySuffix(remLen, req2, req3, req5, req7));
                    return sb.toString();
                }
            }
        }

        // 4. 同长度无解，增加长度构造最小数
        int minNeededLen = getMinDigits(c2, c3, c5, c7);
        int targetLen = Math.max(n + 1, minNeededLen);
        return buildGreedySuffix(targetLen, c2, c3, c5, c7);
    }

    private int[] getFactors(int d) {
        if (d == 2) return new int[]{1, 0, 0, 0};
        if (d == 3) return new int[]{0, 1, 0, 0};
        if (d == 4) return new int[]{2, 0, 0, 0};
        if (d == 5) return new int[]{0, 0, 1, 0};
        if (d == 6) return new int[]{1, 1, 0, 0};
        if (d == 7) return new int[]{0, 0, 0, 1};
        if (d == 8) return new int[]{3, 0, 0, 0};
        if (d == 9) return new int[]{0, 2, 0, 0};
        return new int[]{0, 0, 0, 0};
    }

    private boolean canFulfill(int remLen, int c2, int c3, int c5, int c7) {
        if (remLen < 0) return false;
        return getMinDigits(c2, c3, c5, c7) <= remLen;
    }

    private int getMinDigits(int c2, int c3, int c5, int c7) {
        c2 = Math.max(0, c2);
        c3 = Math.max(0, c3);
        c5 = Math.max(0, c5);
        c7 = Math.max(0, c7);

        int min23 = Integer.MAX_VALUE;
        for (int k6 = 0; k6 <= Math.min(c2, c3); k6++) {
            int r2 = c2 - k6;
            int r3 = c3 - k6;
            int d9 = (r3 + 1) / 2;
            int d8 = (r2 + 2) / 3;
            min23 = Math.min(min23, k6 + d9 + d8);
        }
        return c5 + c7 + min23;
    }

    private String buildGreedySuffix(int remLen, int c2, int c3, int c5, int c7) {
        StringBuilder sb = new StringBuilder();
        int r2 = c2, r3 = c3, r5 = c5, r7 = c7;
        for (int i = 0; i < remLen; i++) {
            int leftLen = remLen - 1 - i;
            for (int d = 1; d <= 9; d++) {
                int[] f = getFactors(d);
                if (canFulfill(leftLen, r2 - f[0], r3 - f[1], r5 - f[2], r7 - f[3])) {
                    sb.append(d);
                    r2 -= f[0]; r3 -= f[1]; r5 -= f[2]; r7 -= f[3];
                    break;
                }
            }
        }
        return sb.toString();
    }
}
```

### 复杂度分析

- **时间复杂度**：O(N)，其中 N 是字符串 `num` 的长度。前缀数组和贪心构造都是单次扫描，整体运行耗时只要几毫秒。

- **空间复杂度**：O(N)，用于存储前缀质因子统计和答案字符串。

## 和 [[3345-smallest-divisible-digit-product-i.md]] 中情况二的关系

代码发生变化的根本原因是**数据规模暴增了 20,000 倍**（从 $10^9$ 的 10 位数，暴增到了 20 万位数）。

如果直接拿上一版的 **DFS 递归回溯** 代码去跑这道题，会面临两个致命问题：

### 改变的原因：为什么不能用上一版的代码？

- **递归爆栈 (StackOverflowError)**
  - **上一版**：针对 $N \le 10^9$（最多只有 10 位数），DFS 递归深度最多 10 层，执行非常快。

  - **这一版**：字符串长度达到了 $2 \times 10^5$，如果继续用递归，函数调用栈会深达 **20 万层**，在 Java 中会瞬间抛出 `StackOverflowError` 导致程序崩溃。

- **回溯耗时太长导致超时 (TLE)**
  - **上一版**：利用递归栈不断 `append` 和 `deleteCharAt` 回溯试错。

  - **这一版**：在 20 万位的字符串上反复修改字符和回溯，时间复杂度会退化到 $O(N^2)$，计算量高达几百亿次。必须改用 **$O(N)$ 预处理 + 逆序循环** 才能在 1 秒内跑完。

### 两版代码的关键差异对比

| **模块**         | **上一版（109 简化版）** | **这一版（2×105 困难版）**        | **修改原因**                                 |
| ---------------- | ------------------------ | --------------------------------- | -------------------------------------------- |
| **控制流程**     | **DFS 递归回溯**         | **纯循环迭代（倒序扫描）**        | 彻底避免 20 万层递归导致的栈溢出             |
| **前缀信息获取** | 随递归状态实时传递       | **前缀数组 `pref2/3/5/7` 预处理** | 将查询任意前缀的质因子数量降至 $O(1)$        |
| **寻求变化点**   | DFS 树自顶向下尝试       | 从字符串**尾部向头部倒序寻找**    | 一旦找到第一个可变大的位置 $i$，立刻终止循环 |
| **答案生成方式** | 逐步回溯拼接字符串       | `substring(0, i)` + 贪心拼接后缀  | 一次性构造完成，无需反复增删字符             |

### 核心本质并没有变

两版代码的核心数学内核是**完全一致**的：

1. 都需要将 $t$ 分解为 **2, 3, 5, 7** 四种质因子；

2. 都使用了相同的 `canFulfill` / `getMinDigits` 函数，计算剩余长度能否凑齐所需质因子。

**总结**：逻辑没变，只是为了应对 **20 万位超长字符串**，将实现方式从“递归回溯”重构成了“前缀和 + 贪心迭代”。

## 详细解释

这道题本质上是在做两件事：**用质因子当计数器**，以及**像翻字典一样找紧挨着的下一个数字**。

### 1. 为什么要分解质因子？（解决“数位乘积”的限制）

因为我们选的每个数位都只能是 `1` 到 `9` 的单个数字。而 `1~9` 这几个数字拆开后，**只由 2、3、5、7 四种质因子构成**（比如 $8 = 2\times 2\times 2$，$9 = 3\times 3$，$6 = 2\times 3$）。

- **乘积整除的本质**：要求“数位乘积能被 $t$ 整除”，意思是所有选出的数字乘在一起后，包含的 2、3、5、7 的数量**不少于** $t$ 包含的数量。

- **化繁为简**：20 万位的数字相乘会产生一个天文数字，计算机根本存不下。但如果把它转化成计数问题——“我们还差 $c_2$ 个 2、$c_3$ 个 3……”——大数乘法就变成了简单的加减法。

> 乘积的质因子数量必须是“至少”和 $t$ 一样多（甚至更多），绝对不能比 $t$ 少。

### 2. 为什么要“匹配前缀”？（保证结果“大于等于 $num$ 且尽量小”）

假设 $num = 12345$，你想找一个比它大一点点的数字：

- 如果把第一位 `1` 改成 `2`（变成 `21111`），它确实比原数大，但**增长得太多了**。

- 如果尽可能保留前面的数字（保留前缀 `123`），只把第 4 位的 `4` 改成 `8`（变成 `1238_`），这个数就会离原数**非常近**。

这就是“前缀匹配”的意义：**从右往左**寻找第一个可以变大的位置（设为第 $i$ 位）。保留第 $0 \sim i-1$ 位不变，把第 $i$ 位变大一点点，就能保证新数字**刚好大于原数，且增幅最小**。

### 3. 为什么要“后缀贪心”？（剩下的位置怎么填）

当我们在第 $i$ 位把数字变大（比如把 `1234_` 变成 `1238_`）之后，这个新数字已经**绝对大于原数**了。

此时，后面还没填的位置（`_`）就可以彻底放飞自我，**不再受 $num$ 原数大小的限制**。为了让整体数值最小，后面的位置必须遵循“**高位尽量填小数字**”的原则（从 `1` 到 `9` 尝试），只要剩余的位置能凑够还缺少的质因子即可。

> 在这里，“贪心”是指：在从左到右填每个位置时，贪心地选能选的最小值（从 1, 2, 3... 开始试）。

**整体逻辑顺一遍：**

1. **统计目标**：把 $t$ 拆成需要 $c_2, c_3, c_5, c_7$ 个质因子。

2. **前缀锁死**：从后往前找一个位置，把该位置数字变大一点点，同时检查“剩下的位数还够不够凑齐缺少的质因子”。

3. **后缀贪心**：一旦确定了变大的位置，后面剩下的空位从左到右，贪心地填入满足质因子要求的**最小数字**（尽量选 `1`、`2` 等小数字）。

> TODO：这里不同位数填数字的情况在其他题目中似乎遇到过，需要留意

想用一个具体数字（比如 num=1234, t=256）走一遍完整推导过程吗？
