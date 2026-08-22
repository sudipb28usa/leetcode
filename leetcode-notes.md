**1. LeetCode — Longest Substring Without Repeating Characters**
**Technique:** Sliding Window · **Data Structure:** Array (fixed-size `int[128]` lookup table)

**⚡ Quick Recall (30 sec):** Need the longest substring with no repeated characters. `right` scans forward one char at a time; `left` only ever jumps forward (never resets) when the char at `right` was last seen inside the current window. Time: O(n) / Space: O(1).

**Approach:**
1. Keep `lastSeen[128]`, storing one-past the last index each char was seen (`0` = never seen).
2. For each `right`, if `lastSeen[c] > left`, that duplicate is inside the window — jump `left = lastSeen[c]`.
3. Record `maxLength = max(maxLength, right - left + 1)` every step.
4. Update `lastSeen[c] = right + 1`.

**ASCII Trace:**
```
 s = "ab", lastSeen[] size 128, all start at 0

 right=0 'a': lastSeen['a']=0, not > left(0) → no shrink
              maxLength = max(0, 0-0+1) = 1
              lastSeen['a'] = 0+1 = 1

 right=1 'b': lastSeen['b']=0, not > left(0) → no shrink
              maxLength = max(1, 1-0+1) = 2
              lastSeen['b'] = 1+1 = 2

 lastSeen[] after "ab":
   idx 97('a') 98('b') 99('c') ... 127
       ┌────┬────┬────┬─────┐
       │  1 │  2 │  0 │ ... │
       └────┴────┴────┴─────┘

 final maxLength = 2  (the whole string "ab" has no repeats)
```

**Complexity:**
- Time: O(n) — `right` makes one pass; `left` only ever moves forward, so together at most 2n steps.
- Space: O(1) — `lastSeen` is a fixed `int[128]`, not sized to input length.

**Tricky part of the code:**
```java
left = Math.max(left, lastSeen[s.charAt(right)]);
```
Easy to write `left = lastSeen[...]` directly instead of `Math.max` — without the max, `left` could jump backward if the duplicate was seen before the current window started.

**Key trick to remember:** `lastSeen[c] = right + 1`, not `right` — storing "one past" means default `0` doubles as "never seen" with no separate init step, and `Math.max(left, lastSeen[c])` just works.

**Pattern tag:** Sliding Window — shares shape with any "longest/shortest window satisfying a constraint" problem.

---

**2. LeetCode — Rotating the Box**
**Technique:** Simulation (gravity drop + 90° rotation in one pass) · **Data Structure:** 2D array (`char[][] outputGrid`)

**⚡ Quick Recall (30 sec):** Stones fall under gravity, then the box rotates 90° clockwise — do both in one pass. Scan each row counting consecutive `#`s (`stoneCount`); flush just before an obstacle `*` or at row's end, writing directly into rotated coordinates. Time: O(rows × cols) / Space: O(rows × cols).

**Approach:**
1. Pre-fill `outputGrid` entirely with `.` — empty cells never need an explicit write.
2. Scan each row left → right, incrementing `stoneCount` on `#`, skipping `.`.
3. On `*`: drop `stoneCount` `#`s just above it, write `*` itself, reset `stoneCount = 0`.
4. On end of row: drop `stoneCount` `#`s at the floor (bottom of that output column).
5. Every write lands at `outputGrid[currentColumn][totalRows - currentRow - 1]`.

**ASCII Trace:**
```
 inputGrid[currentRow]:   #    .    *    #    #    .
 currentColumn:           0    1    2    3    4    5

 currentColumn=0  '#'  → stoneCount = 1
 currentColumn=1  '.'  → skip
 currentColumn=2  '*'  → drop stoneCount(1) just above obstacle, reset to 0
 currentColumn=3  '#'  → stoneCount = 1
 currentColumn=4  '#'  → stoneCount = 2
 currentColumn=5  '.'  → end of row → drop stoneCount(2) at the floor

 outputGrid column (totalRows-currentRow-1), rows 0..totalColumns-1:
     row 0  ┌───┐
     row 1  │ # │   ← stone dropped just above obstacle (from currentColumn=0)
     row 2  │ * │   ← obstacle itself (was at currentColumn=2)
     row 3  │ · │
     row 4  │ # │   ← stones dropped at floor (from currentColumn=3,4)
     row 5  │ # │
            └───┘
```

**Complexity:**
- Time: O(totalRows × totalColumns) — single pass over every input cell; `fill` calls collectively touch each output cell once.
- Space: O(totalRows × totalColumns) — for `outputGrid`, required as output regardless.

**Tricky part of the code:**
```java
outputGrid[currentColumn][totalRows - currentRow - 1] = '*';
fill(outputGrid, stoneCount, currentColumn - 1, totalRows - currentRow - 1);
```
The coordinate flip (`currentColumn` becomes the row, `totalRows - currentRow - 1` becomes the column) is easy to transpose backward by mistake.

**Key trick to remember:** Gravity drop and 90° rotation happen in the SAME pass — no separate rotation step, because `currentColumn` already increases in the same direction as the output's row index.

**Pattern tag:** Simulation — matrix/grid problems that fuse two transformations (physics + geometry) into one scan.

---

**3. LeetCode — Search a 2D Matrix**
**Technique:** Binary Search · **Data Structure:** 2D array treated as a flattened array

**⚡ Quick Recall (30 sec):** Matrix rows sorted, and each row's first value exceeds the previous row's last — the whole thing is one sorted array in disguise. Binary search `[0, n*m-1]`, translate `mid` back with `row = mid/m`, `col = mid%m`. Time: O(log(n×m)) / Space: O(1).

**Approach:**
1. Set `low = 0`, `high = n*m - 1` where `m` = column count.
2. Standard binary search loop: `mid = low + (high-low)/2`.
3. Translate: `row = mid / m`, `col = mid % m`.
4. Compare `mat[row][col]` to target, move `low`/`high` exactly like a normal binary search.

**ASCII Trace:**
```
 mat = [[ 1,  2,  3,  4],
        [ 5,  6,  7,  8],
        [ 9, 10, 11, 12]]     target = 8

 flat index:  0   1   2   3   4   5   6   7   8   9  10  11
 flat value:  1   2   3   4   5   6   7   8   9  10  11  12
                                     ↑ target=8 lives at flat index 7

 low=0, high=11
   mid=5  → row=5/4=1, col=5%4=1 → mat[1][1]=6 < 8 → low=mid+1=6
   mid=8  → row=8/4=2, col=8%4=0 → mat[2][0]=9 > 8 → high=mid-1=7
   mid=7  → row=7/4=1, col=7%4=3 → mat[1][3]=8 == 8 → found!
```

**Complexity:**
- Time: O(log(n × m)) — standard binary search over a virtual array of n*m elements.
- Space: O(1) — only `low`, `high`, `mid`, `row`, `col`.

**Tricky part of the code:**
```java
int row = mid / m;
int col = mid % m;
```
Using `n` instead of `m` here is a common slip — `m` (column count) is what determines how many flat indices make up one row.

**Key trick to remember:** `m` is the column count, and it's what you divide/mod by — not `n`. Mixing them up silently breaks the translation.

**Pattern tag:** Binary Search — any "sorted structure disguised as something else" problem (flattened matrix, rotated array, etc.).

---

**4. LeetCode 102 — Binary Tree Level Order Traversal**
**Technique:** Breadth-First Search (BFS) · **Data Structure:** Queue (`LinkedList<TreeNode>`)

**⚡ Quick Recall (30 sec):** Need node values grouped level by level. BFS with a queue, but freeze `size = queue.size()` before popping each level — that's what keeps levels from bleeding together. Time: O(n) / Space: O(n).

**Approach:**
1. Push `root` onto the queue (return empty list if `root == null`).
2. Each `while` iteration = one level: snapshot `size = queue.size()` before popping anything.
3. Pop exactly `size` nodes, record values into `level`, enqueue their children.
4. After the inner loop, push `level` into `result`.

**ASCII Trace:**
```
        3
       / \
      9   20
         /  \
        15   7

 queue=[3]        size=1
   pop 3  → level=[3]                enqueue 9, 20   → queue=[9,20]

 queue=[9,20]     size=2
   pop 9  → level=[9]                no children
   pop 20 → level=[9,20]             enqueue 15, 7   → queue=[15,7]

 queue=[15,7]     size=2
   pop 15 → level=[15]               no children
   pop 7  → level=[15,7]             no children     → queue=[]

 result = [[3], [9,20], [15,7]]
```

**Complexity:**
- Time: O(n) — every node enqueued and dequeued exactly once.
- Space: O(n) — queue holds at most one full level, up to O(n) for a complete tree's bottom level.

**Tricky part of the code:**
```java
int size = queue.size();          // freeze this level's width before popping
```
Skipping this line still visits every node correctly, but loses the level grouping entirely — children get enqueued mid-loop and would get scooped up in the same iteration's pop count if `size` isn't frozen first.

**Key trick to remember:** Snapshot `queue.size()` before the inner loop starts — that single freeze is the entire difference between a plain BFS traversal and a level-order one.

**Pattern tag:** Breadth-First Search — any "process level by level" tree/graph problem.

---

**5. LeetCode — Bubble Sort**
**Technique:** Bubble Sort (adjacent swap, comparison-based) · **Data Structure:** Array (in-place)

**⚡ Quick Recall (30 sec):** Repeatedly swap adjacent out-of-order pairs; each pass "bubbles" the largest remaining value to the right edge of the unsorted zone. Early-exit with a `swap` flag if a full pass makes zero swaps. Time: O(n²) worst / O(n) best / Space: O(1).

**Approach:**
1. Outer loop `i`: one pass per iteration.
2. Inner loop `j` from `0` to `size-i-2`: compare `arr[j]` vs `arr[j+1]`.
3. If out of order, swap and set `swap = true`.
4. If a full pass makes no swaps, `break` early — array's already sorted.

**ASCII Trace:**
```
 arr = [5, 1, 4, 2, 8]     [x] = the pair that just swapped

 Start:            5   1   4   2   8

 Move 1 (5>1):    [1] [5]  4   2   8
 Move 2 (5>4):     1  [4] [5]  2   8
 Move 3 (5>2):     1   4  [2] [5]  8    ← pass 1 done (5<8, no swap)
 Move 4 (4>2):     1  [2] [4]  5   8    ← pass 2 done (4<5, no swap)

 Pass 3 — no swaps at all (1<2, 2<4) → swap=false → break early

 Final:             1   2   4   5   8
```

**Complexity:**
- Time: O(n²) worst case (reverse-sorted never triggers early exit) — O(n) best case (already sorted triggers `swap=false` on pass one).
- Space: O(1) — only `temp` and `swap` as extras.

**Tricky part of the code:**
```java
for (int j = 0; j < (size - i - 1); j++)
```
The `-i-1` bound is easy to get off-by-one on — it shrinks because the last `i` elements are already settled from prior passes and don't need rechecking.

**Key trick to remember:** The `swap` flag is the whole point of the early exit — without it, bubble sort always runs the full O(n²), even on an already-sorted array.

**Pattern tag:** Comparison Sort — baseline against which Insertion and Selection Sort's tradeoffs are usually compared.

---

**6. LeetCode — Insertion Sort**
**Technique:** Insertion Sort (shift-and-insert, comparison-based) · **Data Structure:** Array (in-place)

**⚡ Quick Recall (30 sec):** Everything left of `i` is always sorted. Pull `key = arr[i]`, slide bigger elements right one at a time (`arr[j+1] = arr[j]`), insert `key` where the sliding stops. Time: O(n²) worst / O(n) best / Space: O(1).

**Approach:**
1. Loop `i` from 1 to end; `key = arr[i]`, `j = i - 1`.
2. While `j >= 0 && arr[j] > key`: shift `arr[j]` right, decrement `j`.
3. Insert: `arr[j+1] = key`.

**ASCII Trace:**
```
 arr = [5, 1, 4, 2, 8]     [x] = the key value just inserted

 Start:                5   1   4   2   8

 i=1: key=1 — shift 5 right, insert at idx 0
                       [1]  5   4   2   8

 i=2: key=4 — shift 5 right, insert at idx 1
                        1  [4]  5   2   8

 i=3: key=2 — shift 5 then 4 right, insert at idx 1
                        1  [2]  4   5   8

 i=4: key=8 — nothing bigger to its left, no shifts
                        1   2   4   5  [8]
```

**Complexity:**
- Time: O(n²) worst case (reverse-sorted, every key shifts to the front) — O(n) best case (`arr[j] > key` fails immediately on sorted input).
- Space: O(1) — `key` and `j` are the only extras; mutates in place, returns `void`.

**Tricky part of the code:**
```java
while (j >= 0 && arr[j] > key) {
    arr[j + 1] = arr[j];
    j = j - 1;
}
arr[j + 1] = key;
```
The final insert (`arr[j+1] = key`) is correct whether the loop stopped from `j < 0` or from finding a smaller element — easy to think these need separate handling, but they don't.

**Key trick to remember:** Think of it as sorting playing cards in your hand — pull one card out, slide it left past bigger cards, drop it in the gap. That mental model prevents overcomplicating the loop exit condition.

**Pattern tag:** Comparison Sort — good for nearly-sorted input, unlike Selection Sort which costs the same regardless of input order.

---

**7. LeetCode — Selection Sort**
**Technique:** Selection Sort (find-min-then-swap, comparison-based) · **Data Structure:** Array (in-place)

**⚡ Quick Recall (30 sec):** Outer loop locks index `i`; inner loop scans the rest hunting for the smallest value's index only (no swaps during scan); exactly ONE swap closes out each `i`. Always exactly `n` swaps, even on sorted input. Time: O(n²) always / Space: O(1).

**Approach:**
1. Outer loop `i` from 0 to end.
2. Inner loop `p` from `i+1` to end: track `minIndex` if `arr[p] < arr[minIndex]`.
3. After the scan, swap `arr[i]` and `arr[minIndex]` — one swap per outer iteration, even if it's a no-op.

**ASCII Trace:**
```
 arr = [5, 1, 4, 2, 8]     [x] = the swap that closed out this i

 Start:                    5   1   4   2   8

 i=0: min=1 at idx 1 → swap idx 0,1
                          [1] [5]  4   2   8

 i=1: min=2 at idx 3 → swap idx 1,3
                           1  [2]  4  [5]  8

 i=2: min=4 at idx 2 (itself) → swap idx 2,2 — no-op
 i=3: min=5 at idx 3 (itself) → swap idx 3,3 — no-op
 i=4: min=8 at idx 4 (itself) → swap idx 4,4 — no-op

 Final:                     1   2   4   5   8
```

**Complexity:**
- Time: O(n²) always — inner loop scans the full remaining range every pass regardless of input order; no early-exit like Bubble Sort's `swap` flag.
- Space: O(1) — only `minIndex` and `temp`.

**Tricky part of the code:**
```java
int temp = arr[i];
arr[i] = arr[minIndex];
arr[minIndex] = temp;
```
This runs every single outer iteration, even when `minIndex == i` — it's easy to assume that case should be skipped, but the no-op swap is harmless and skipping it adds unneeded branching.

**Key trick to remember:** Selection Sort's cost lives entirely in comparisons, not writes — the exact opposite tradeoff from Insertion Sort, which does few comparisons on sorted input but many shifts on reversed input.

**Pattern tag:** Comparison Sort — always O(n²), no best-case speedup, unlike the other two.

---

**8. LeetCode 84 — Largest Rectangle in Histogram**
**Technique:** Monotonic Stack · **Data Structure:** Stack (`Stack<int[]>` holding `[start, height]` pairs)

**⚡ Quick Recall (30 sec):** Find the largest rectangle in a histogram. Stack holds `[start, height]` pairs in non-decreasing height order — when a shorter bar shows up, pop and close out every taller bar, inheriting its `start` so the current bar can reach back through it. Time: O(n) / Space: O(n).

**Approach:**
1. For each `i`, while `stack.peek()[1] > h`: pop `[previousIndex, previousHeight]`.
2. Close it out: `area = max(area, (i - previousIndex) * previousHeight)`.
3. Inherit: `start = previousIndex` (current bar can extend back through what it just outlasted).
4. Push `[start, h]` after the while loop.
5. After the main loop, drain the stack using `n` as the right wall for everything left.

**ASCII Trace:**
```
 heights = [2, 1, 5, 6, 2, 3]      index:  0  1  2  3  4  5

 i=0  h=2            stack empty → push [0,2]
      stack: [0,2]

 i=1  h=1            peek=[0,2], 2>1 → pop[0,2] (start=0)
                        area = max(0, (1-0)*2) = 2
                      stack empty → push [0,1]
      stack: [0,1]

 i=2  h=5            peek=[0,1], 1>5? no → push [2,5]
      stack: [0,1] [2,5]

 i=3  h=6            peek=[2,5], 5>6? no → push [3,6]
      stack: [0,1] [2,5] [3,6]

 i=4  h=2            peek=[3,6], 6>2 → pop[3,6] (start=3)
                        area = max(2, (4-3)*6) = 6
                      peek=[2,5], 5>2 → pop[2,5] (start=2)
                        area = max(6, (4-2)*5) = 10
                      peek=[0,1], 1>2? no → push [2,2]
      stack: [0,1] [2,2]

 i=5  h=3            peek=[2,2], 2>3? no → push [5,3]
      stack: [0,1] [2,2] [5,3]

 end of array, n=6 — drain stack, right wall = n for everyone left:
      pop[5,3] → area = max(10, (6-5)*3) = 10
      pop[2,2] → area = max(10, (6-2)*2) = 10
      pop[0,1] → area = max(10, (6-0)*1) = 10

 Final area = 10
```

**Why it's O(n) and not O(n²):** Each index is pushed exactly once and popped at most once across the entire run — even though there's a while loop inside a for loop, the total pop count across all iterations can never exceed n, so the work amortizes to linear.

**Complexity:**
- Time: O(n) — each index pushed once, popped at most once.
- Space: O(n) — worst case (strictly increasing heights), every bar sits in the stack at once.

**Tricky part of the code:**
```java
start = previousIndex;   // inherit how far left this run reaches
```
Easy to miss — when a bar gets popped, the new bar being pushed inherits its `start` index too, not just its height comparison outcome. Without this line, the algorithm can't correctly track how far left a merged rectangle could extend.

**Key trick to remember:** Push `[start, h]`, not `[i, h]` — the inherited `start` is what lets a rectangle's width reach back through every taller bar it just outlasted, without ever scanning backward.

**Pattern tag:** Monotonic Stack — histogram/skyline problems where "what's the closest smaller/larger element" needs answering in one pass.

---

**9. LeetCode 42 — Trapping Rain Water**
**Technique:** Two Pointer · **Data Structure:** Array (no stack — `left`/`right`/`leftMax`/`rightMax` ints)

**⚡ Quick Recall (30 sec):** Find trapped water between bars after rain. Two pointers close in from both ends; whichever side is shorter is safe to process using only that side's running max — the other side's max is always at least as big. Time: O(n) / Space: O(1).

**Approach:**
1. `left=0`, `right=n-1`, `leftMax=0`, `rightMax=0`.
2. While `left < right`: compare `heights[left]` vs `heights[right]`.
3. If `heights[left]` is smaller: update `leftMax`, add `leftMax - heights[left]` to water, `left++`.
4. Else: update `rightMax`, add `rightMax - heights[right]` to water, `right--`.

**ASCII Trace:**
```
 heights = [0,1,0,2,1,0,1,3,2,1,2,1]

 left=0,right=11, leftMax=0,rightMax=0, water=0
   h[0]=0 < h[11]=1 → leftMax=0, water+=0, left=1
   h[1]=1, h[11]=1 → not < → rightMax=1, water+=0, right=10
   h[1]=1 < h[10]=2 → leftMax=1, water+=0, left=2
   h[2]=0 < h[10]=2 → leftMax=1, water+=(1-0)=1, left=3   → water=1
   ...
   (continues closing in)

 Final water = 6
```

**Why it's O(n) and not needing precomputed max arrays:** The classic approach precomputes `leftMax[]`/`rightMax[]` arrays in O(n) space. Two pointers track the same running maxes on the fly in O(1) space — whichever side is currently shorter has its water level fully determined by its own side's max, since the other side is guaranteed to be at least as tall as the taller bar.

**Complexity:**
- Time: O(n) — each pointer moves inward once per step, together covering the array in one pass.
- Space: O(1) — only `left`, `right`, `leftMax`, `rightMax`, `water`.

**Tricky part of the code:**
```java
if (heights[left] < heights[right]) {
    leftMax = Math.max(leftMax, heights[left]);
    water += leftMax - heights[left];
    left++;
}
```
Trusting `leftMax` alone here feels incomplete — it's easy to think you need `rightMax`'s exact value too, but you don't: since `heights[right] > heights[left]`, `rightMax` is guaranteed at least that big, so it can never be the binding constraint.

**Key trick to remember:** Process the shorter side — its own running max is always the true bound, because the taller side's max is guaranteed to be at least as big as the current shorter bar.

**Pattern tag:** Two Pointer — converging-pointer problems where a running max/min on each side substitutes for a precomputed array.

---

**10. LeetCode 1248 — Count Number of Nice Subarrays**
**Technique:** Sliding Window, "exactly K = atMost(K) − atMost(K−1)" transformation · **Data Structure:** Array (`start`/`end`/`totalOdd` ints)

**⚡ Quick Recall (30 sec):** Count subarrays with exactly `k` odd numbers. Counting "exactly k" directly with a sliding window doesn't work cleanly — count "at most k" instead, then `exactly k = atMost(k) - atMost(k-1)`. Time: O(n) / Space: O(1).

**Approach:**
1. Write a helper `totalSubArray(nums, K)` counting subarrays with at most `K` odd numbers.
2. Standard window: increment `totalOdd` on odd numbers at `end`.
3. Shrink while `totalOdd > K`: decrement if `nums[start]` is odd, then `start++`.
4. Add `(end - start + 1)` each step — every subarray ending at `end` starting from `start` onward is valid.
5. Call the helper twice: `totalSubArray(nums, k) - totalSubArray(nums, k-1)`.

**ASCII Trace:**
```
 nums = [1, 1, 2, 1, 1]   k = 3

 atMost(3):
   end=0 [1]        totalOdd=1  start=0 → count += 1   → total=1
   end=1 [1,1]      totalOdd=2  start=0 → count += 2   → total=3
   end=2 [1,1,2]    totalOdd=2  start=0 → count += 3   → total=6
   end=3 [1,1,2,1]  totalOdd=3  start=0 → count += 4   → total=10
   end=4 [...1,1]   totalOdd=4 >3 → shrink: nums[0]=1 odd→totalOdd=3, start=1
                     count += (4-1+1)=4   → total=14
   atMost(3) = 14

 atMost(2):
   end=0  totalOdd=1              count += 1               → total=1
   end=1  totalOdd=2              count += 2               → total=3
   end=2  totalOdd=2              count += 3               → total=6
   end=3  totalOdd=3 >2 → shrink: start=1, count += 3       → total=9
   end=4  totalOdd=3 >2 → shrink: start=2, count += 3       → total=12
   atMost(2) = 12

 answer = 14 - 12 = 2   ✓ matches expected output
```

**Why it's O(n) and not requiring a harder "exactly k" window:** A window with exactly `k` odds can't grow/shrink by one simple rule — adding an odd number can jump the count from `k` straight past it with no valid intermediate state. Reframing as two "at most" windows sidesteps this entirely, and each "at most" call is a standard O(n) shrinking window, so two calls stay O(n) overall.

**Complexity:**
- Time: O(n) — each `totalSubArray` call is O(n) (start only moves forward); called twice.
- Space: O(1) — only `start`, `totalOdd`, `totalSubArray`.

**Tricky part of the code:**
```java
return totalSubArray(nums, k) - totalSubArray(nums, k - 1);
```
It's tempting to try to solve "exactly k" directly in one pass — this subtraction trick is the whole unlock, and it's easy to forget that `atMost(k)` and `atMost(k-1)` are two separate full passes, not one shared pass with two counters.

**Key trick to remember:** `exactly k = atMost(k) - atMost(k-1)` — this reframing pattern also applies to LC 992 (Subarrays with K Different Integers) and LC 930 (Binary Subarrays with Sum); only what's being counted changes.

**Pattern tag:** Sliding Window (At-Most Trick) — any "count subarrays with exactly K of some property" problem.

---

**11. LeetCode 128 — Longest Consecutive Sequence**
**Technique:** Hash Set (Sequence-Start Detection) · **Data Structure:** HashSet<Integer>

**⚡ Quick Recall (30 sec):** Unsorted array, need the longest run of consecutive integers without sorting (O(n) required). Dump everything into a HashSet, then only start counting from a number where `num - 1` is NOT in the set — a guaranteed sequence start. Walk forward while present. Time: O(n) / Space: O(n).

**Approach:**
1. Add all elements into a `HashSet<Integer>` for O(1) lookups.
2. For each number, check if it's a sequence start: `!set.contains(num - 1)`.
3. If it's a start, walk forward counting consecutive values (`num+1`, `num+2`, ...) while they exist in the set.
4. Track the max streak length seen across all starts.
5. Return the max (0 if input is empty).

**ASCII Trace:**
```
 set = {100, 4, 200, 1, 3, 2}

 check each number — is it a sequence START? (num-1 not in set)
   100 → 99 not in set → START → count: 100 → 200 not in set → length 1
   4   → 3 in set → NOT a start → skip
   200 → 199 not in set → START → count: 200 → 201 not in set → length 1
   1   → 0 not in set → START → count: 1,2,3,4 → 5 not in set → length 4
   3   → 2 in set → NOT a start → skip
   2   → 1 in set → NOT a start → skip

 longest = 4  (the sequence 1,2,3,4)
```

**Why it's O(n) and not O(n log n):** Sorting first gets you there in O(n log n). The hash-set version avoids sorting entirely — the "only expand from true starts" check guarantees each number is visited by the inner `while` loop at most once across the entire run, because every number belongs to exactly one sequence and only that sequence's start ever walks through it.

**Complexity:**
- Time: O(n) — HashSet build is O(n); start-check is O(1) per number; inner while loop's total iterations across all starts sum to n, not n².
- Space: O(n) — HashSet holds up to n elements.

**Tricky part of the code:**
```java
if (!set.contains(num - 1)) {
    int length = 1;
    while (set.contains(num + length)) length++;
    longest = Math.max(longest, length);
}
```
Skip this check and the algorithm still gives the right answer, but every number re-walks its own subsequence from scratch — silently degrading to O(n²) without throwing any error.

**Key trick to remember:** Only count forward from a number where `num - 1` is missing from the set. That single check is what keeps the total work linear instead of quadratic.

**Pattern tag:** Hash Set — same family as other "skip sorting, use O(1) lookups + a smart start condition" problems.

---

**12. LeetCode 54 — Spiral Matrix**
**Technique:** Boundary Shrinking (matrix traversal) · **Data Structure:** 2D array (no extra structure — `top`/`bottom`/`left`/`right` ints)

**⚡ Quick Recall (30 sec):** Return all elements of an m×n matrix in spiral order. Track four shrinking boundaries (`top`, `bottom`, `left`, `right`); walk each edge in order (top row L→R, right col T→B, bottom row R→L, left col B→T), shrinking the boundary just walked. Guard the last two walks with `if` checks to avoid double-counting. Time: O(m×n) / Space: O(1) extra.

**Approach:**
1. Set `top=0`, `bottom=rows-1`, `left=0`, `right=cols-1`.
2. While `top <= bottom && left <= right`: walk top row left→right, then `top++`.
3. Walk right column top→bottom, then `right--`.
4. If `top <= bottom`: walk bottom row right→left, then `bottom--`.
5. If `left <= right`: walk left column bottom→top, then `left++`.

**ASCII Trace:**
```
 matrix = [[1,2,3],
           [4,5,6],
           [7,8,9]]

 top=0, bottom=2, left=0, right=2, result=[]

 Round 1:
   walk row top=0, cols 0→2:      1, 2, 3          result=[1,2,3]      top=1
   walk col right=2, rows 1→2:    6, 9             result=[1,2,3,6,9]  right=1
   top(1)<=bottom(2)? yes
     walk row bottom=2, cols 1→0: 8, 7             result+=[8,7]       bottom=1
   left(0)<=right(1)? yes
     walk col left=0, rows 1→1:   4                result+=[4]         left=1

 State now: top=1, bottom=1, left=1, right=1 — only cell [1][1]=5 unvisited

 Round 2: top(1)<=bottom(1) && left(1)<=right(1) → loop runs again
   walk row top=1, cols 1→1:      5                result+=[5]         top=2
   walk col right=1, rows 2→1:    (empty range, top=2 > bottom=1)      right=0
   top(2)<=bottom(1)? no → skip bottom row
   left(1)<=right(0)? no → skip left col

 top=2, bottom=1 → top>bottom → loop ends

 Final result = [1, 2, 3, 6, 9, 8, 7, 4, 5]
```

**Complexity:**
- Time: O(m × n) — every cell is visited and added to the result exactly once.
- Space: O(1) extra — only `top`, `bottom`, `left`, `right` (not counting the output list, required regardless).

**Tricky part of the code:**
```java
if (top <= bottom) {
    for (int col = right; col >= left; col--) {
        result.add(matrix[bottom][col]);
    }
    bottom--;
}

if (left <= right) {
    for (int row = bottom; row >= top; row--) {
        result.add(matrix[row][left]);
    }
    left++;
}
```
Without these two guards, a matrix that collapses to a single leftover row or column gets double-counted — the bottom-row and left-column walks would re-walk cells the top-row/right-column walks just visited, once `top` and `bottom` (or `left` and `right`) meet at the same index.

**Key trick to remember:** The two `if` guards are what separates a correct spiral from one that only works on "nice" matrices. Once the boundaries collapse to a single remaining row or column, walking it once (via the top-row or right-column pass) is enough — walking it again unconditionally as a "bottom row" or "left column" revisits the same cells.

**Pattern tag:** Boundary Shrinking — same mental model as LC 48 (Rotate Image, layer by layer) and LC 59 (Spiral Matrix II, filling instead of reading).

# 13. 48 -Rotate Image (Medium) — Layer-by-Layer Cyclic Swap

**Data structure:** 2D array (in-place, no auxiliary matrix — just `l`/`r`/`t`/`b` boundary ints)
**Technique:** Matrix Traversal (4-way cyclic rotation, ring by ring)

Given an n x n matrix, rotate it 90° clockwise in place.

**5-second recall:**
- Four boundaries `l, r, t, b` define the current ring — start at the outermost, shrink inward each pass.
- Inner loop `i` walks `0 .. (r - l - 1)` along the top edge of the current ring — stops one short of the corner, since that corner gets covered by the wraparound from `i=0` of the next side.
- One `top` temp holds the top value; the swap chain runs counter-clockwise: **top ← left, left ← bottom, bottom ← right, right ← top(saved)**.
- After a full ring: `l++, r--, t++, b--` — shrink to the next ring in.
- Loop condition `l < r` — a single center cell (odd n) or empty middle never needs swapping.

```
   for each ring (l,r,t,b), for i = 0 .. (r-l-1):

     top = matrix[t][l+i]
     matrix[t][l+i]   = matrix[b-i][l]      // left  -> top
     matrix[b-i][l]   = matrix[b][r-i]      // bottom-> left
     matrix[b][r-i]   = matrix[t+i][r]      // right -> bottom
     matrix[t+i][r]   = top                 // top(saved) -> right

   then: l++, r--, t++, b--
```

```
 matrix = [[1,2,3],
           [4,5,6],
           [7,8,9]]      n=3, l=0,r=2,t=0,b=2

 Ring 0, i=0..0 (r-l-1 = 1, so only i=0):

 i=0:  top = matrix[0][0] = 1
       matrix[0][0] = matrix[2][0] = 7     -> top-left  = 7
       matrix[2][0] = matrix[2][2] = 9     -> bot-left  = 9
       matrix[2][2] = matrix[0][2] = 3     -> bot-right = 3
       matrix[0][2] = top = 1              -> top-right = 1

 After ring 0:
       [[7,2,1],
        [4,5,6],
        [9,8,3]]

 l++,r--,t++,b--  -> l=1,r=1 -> loop ends (l < r fails, single center cell 5 needs no swap)

 Final:
       [[7,4,1],
        [8,5,2],
        [9,6,3]]
```

The whole matrix is a stack of concentric square "rings," and each ring is 4 edges that need to rotate into each other simultaneously. Doing this in place is only safe because of the order: save the top value first (`top`), then overwrite top from left, left from bottom, bottom from right, and finally right from the saved `top` — a 4-way cycle closed by holding exactly one value outside the array at a time. The inner loop only needs to go one short of the ring's width (`i < r-l`, i.e. `l+i < r`) because each iteration handles one full 4-cell group (one from each edge), and the last position of each edge is already covered by the group that starts at that edge's own `i=0` — going further would double-swap the corners.

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        int l = 0, r = n - 1;
        int t = l, b = r;
        while (l < r) {
            for (int i = 0; (l + i) < r; i++) {
                int top = matrix[t][l + i];
                matrix[t][l + i] = matrix[b - i][l];
                matrix[b - i][l] = matrix[b][r - i];
                matrix[b][r - i] = matrix[t + i][r];
                matrix[t + i][r] = top;
            }
            l++;
            r--;
            t++;
            b--;
        }
    }
}
```

**Time complexity:** O(n²) — every cell is touched exactly once across all rings combined.

**Space complexity:** O(1) — in-place; `top` is the only extra variable, no auxiliary matrix.


#14. LeetCode 200 — Number of Islands

**Technique:** DFS (grid flood-fill, in-place mutation) · **Data Structure:** None — no visited array, no queue; uses the call stack only

**⚡ Quick Recall (30 sec):** Same island-counting problem, but drop the `boolean[][] visited` array entirely by sinking each explored land cell directly in the input grid (`grid[r][c] = '0'`) — a `'0'` cell fails the `!= '1'` check, so it can never be re-explored. Swap BFS/queue for recursive DFS, so the call stack does the "remember where to go" job instead of an explicit queue. Time: O(m×n) / Space: O(1) auxiliary (see caveat below).

**Approach:**
1. Scan every cell `(i, j)`; if `grid[i][j] == '1'`, call `dfs(grid, i, j)` and increment `totalIsland`.
2. `dfs` base case: out of bounds or `grid[r][c] != '1'` → return immediately.
3. Otherwise, sink the cell: `grid[r][c] = '0'` — this IS the visited mark.
4. Recurse into all 4 directions.

**Why this drops space from O(m×n) to O(1):**
The BFS version needed a `boolean[][] visited` array (O(m×n)) plus a `Queue<int[]>` that could hold up to O(m×n) cells at once in the worst case. The DFS-in-place version needs neither — visited-tracking is folded into the grid mutation itself (no extra array), and the "next cells to explore" bookkeeping is handled implicitly by the recursive call stack instead of an explicit queue object.

**Tricky part of the code:**
```java
grid[r][c] = '0';   // mark visited by sinking the land — no separate visited array
```
Easy to forget this line entirely and just recurse — without it, every cell gets revisited infinitely and the recursion never terminates (stack overflow). This single mutation is doing double duty: it's both the "flood fill" and the "visited" mechanism at once.

**Key trick to remember:** Mutating the input to encode "visited" state (sinking `'1'` → `'0'`) is a standard space-saving trick whenever the input is disposable — it replaces an O(m×n) auxiliary structure with zero extra memory, at the cost of destroying the original grid.

**Honest caveat on "O(1)":** The recursive call stack itself can go O(m×n) deep in the worst case (e.g., one long snake-shaped island spanning the whole grid) — so this is O(1) *extra data-structure* space, not O(1) counting the call stack. Worth stating explicitly if an interviewer pushes on "is this truly O(1)?" — the fully rigorous answer is O(1) auxiliary space, O(m×n) implicit recursion-stack space in the worst case.

**Trade-off to flag out loud in an interview:** this destroys the input grid (`'1'`s become `'0'`s). Acceptable here since the grid isn't needed afterward, but worth naming as a conscious choice rather than a side effect.

**Pattern tag:** Grid DFS / In-Place Mutation — same space-saving trick applies to LC 130 (Surrounded Regions) and LC 733 (Flood Fill) whenever the input grid is safe to overwrite.
