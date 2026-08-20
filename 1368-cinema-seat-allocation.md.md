# 1386-安排电影院座位[中等]

---

> <https://leetcode.cn/problems/cinema-seat-allocation>

---

## Java

**按行独立计算是最优解**，因为各行之间的座位安排完全互不干扰，不存在跨行的约束。但**绝对不能从 $1$ 到 $n$ 逐行遍历**，因为 $n$ 最大可达 $10^9$，直接遍历会导致超时。

由于预定数组 `reservedSeats` 的长度最多只有 $10^4$，这意味绝大多数行是没有被预定的。我们可以采用**位运算 + 哈希表**的“稀疏处理”策略：仅处理有被预定的行，其余完全空闲的行直接通过公式计算。

### 解题核心逻辑

1. **座位块划分**
   - 第 1 列和第 10 列座位不参与任何 4 人小组的安排，完全可以忽略。

   - 剩余 2~9 号座位分为三个候选块：
     - **左侧块**：2, 3, 4, 5

     - **中间块**：4, 5, 6, 7

     - **右侧块**：6, 7, 8, 9

2. **单行判定规则**
   - **安排 2 组**：只有当左侧块和右侧块都完全空闲时，才能安排 2 组。

   - **安排 1 组**：如果无法安排 2 组，但左侧块、中间块、右侧块中任意**有一个**完全空闲，就能安排 1 组。

   - **安排 0 组**：上述条件均不满足。

3. **算法步骤**
   - 使用 `HashMap<Integer, Integer>` 记录有预定的行编号及其座位占用状态（用二进制掩码表示）。

   - 遍历哈希表，统计有预定行的最大可安排数量。

   - 未在哈希表中出现的空行，每行固定可以安排 **2** 个小组，数量为 `(n - map.size()) * 2`。

> - 这里`1<<3`实际是第4位是1，不过所有处理逻辑都是如此，所以不受影响。
> - 题目示例是每行从左到右1-10，但是代码在处理时是从右到左。
> - 1和10不能坐是题目的要求

### Java 实现代码

```Java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int maxNumberOfFamilies(int n, int[][] reservedSeats) {
        // 记录有被预定座位的行号及其掩码 (Bitmask)
        Map<Integer, Integer> rowMasks = new HashMap<>();

        for (int[] seat : reservedSeats) {
            int row = seat[0];
            int col = seat[1];
            // 只有 2~9 号座位会影响 4 人小组的安排
            if (col >= 2 && col <= 9) {
                rowMasks.put(row, rowMasks.getOrDefault(row, 0) | (1 << col));
            }
        }

        // 构造三组座位的掩码
        // 左侧块：座位 2, 3, 4, 5
        int leftMask  = (1 << 2) | (1 << 3) | (1 << 4) | (1 << 5);
        // 中间块：座位 4, 5, 6, 7
        int midMask   = (1 << 4) | (1 << 5) | (1 << 6) | (1 << 7);
        // 右侧块：座位 6, 7, 8, 9
        int rightMask = (1 << 6) | (1 << 7) | (1 << 8) | (1 << 9);

        // 完全没有被预定的行，每行可以直接安排 2 个小组
        int totalGroups = (n - rowMasks.size()) * 2;

        // 处理存在预定座位的行
        for (int mask : rowMasks.values()) {
            boolean leftFree  = (mask & leftMask) == 0;
            boolean rightFree = (mask & rightMask) == 0;
            boolean midFree   = (mask & midMask) == 0;

            if (leftFree && rightFree) {
                // 左右两侧同时满足，最多安排 2 组
                totalGroups += 2;
            } else if (leftFree || rightFree || midFree) {
                // 只要左、中、右任意一处满足，就可以安排 1 组
                totalGroups += 1;
            }
        }

        return totalGroups;
    }
}
```

### 复杂度分析

- **时间复杂度**：$\mathcal{O}(M)$，其中 $M$ 为 `reservedSeats` 的长度。遍历预定数组和哈希表的时间仅取决于预定座位的数量，与 $n$ 无关。

- **空间复杂度**：$\mathcal{O}(M)$，哈希表最多存储 $M$ 行的掩码信息。

## Python

逐行处理的结果就是全局最优解，因为不同行之间的座位安排相互独立，不存在跨行的组合限制。不过，直接对 $1$ 到 $n$ 行进行全量循环扫描会超时（因为 $n \le 10^9$），正确的策略是**只对有预约记录的行进行哈希表统计**，未被预约的行直接按每行 2 个小组计算。

**单行决策逻辑**

每行关注的有效座位仅为 2 到 9 号（1 号和 10 号不影响 4 人连续座位的安排）：

- **能放 2 组**：【2,3,4,5】和【6,7,8,9】均无人预订。

- **能放 1 组**：若无法放 2 组，但【2,3,4,5】或【6,7,8,9】或中间的【4,5,6,7】中有任意一个连续 4 座块未被预订，即可安排 1 组。

- **能放 0 组**：上述三种情况均不满足。

**优化思路与位运算**

由于 $1 \le reservedSeats.length \le 10^4$，有预订记录的行数最多只有 $10^4$ 行。通过哈希表结合位掩码（Bitmask），可以将空间与计算复杂度降至最简：

```Python
from collections import defaultdict

def maxNumberOfFamilies(n: int, reservedSeats: list[list[int]]) -> int:
    # 哈希表存储：row -> 座位占用状态的位掩码 (仅映射 2~9 号座位)
    occupied = defaultdict(int)
    for r, c in reservedSeats:
        if 2 <= c <= 9:
            # 座位 2~9 映射到 bit 0~7
            occupied[r] |= (1 << (c - 2))

    # 定义掩码常数：
    # 座位 2,3,4,5 (bit 0..3) -> 0b00001111 (15)
    # 座位 6,7,8,9 (bit 4..7) -> 0b11110000 (240)
    # 座位 4,5,6,7 (bit 2..5) -> 0b00111100 (60)
    LEFT_MASK = 0b00001111
    RIGHT_MASK = 0b11110000
    MID_MASK = 0b00111100

    # 没有任何预订信息的行，每行必然能放置 2 个小组
    ans = (n - len(occupied)) * 2

    # 仅遍历有预订记录的行
    for mask in occupied.values():
        if (mask & LEFT_MASK) == 0 and (mask & RIGHT_MASK) == 0:
            ans += 2
        elif (mask & LEFT_MASK) == 0 or (mask & RIGHT_MASK) == 0 or (mask & MID_MASK) == 0:
            ans += 1

    return ans
```

**复杂度分析**

- **时间复杂度**：$O(K)$，其中 $K$ 为 `reservedSeats` 的长度（最高 $10^4$）。只需遍历一次输入数组构建哈希表，再遍历一次哈希表即可。

- **空间复杂度**：$O(K)$，哈希表最多保存 $K$ 个键值对。
