# Sliding Window

**Data structure:** Array (fixed-size `int[128]` lookup table, no stack/queue needed)
**Technique:** Sliding Window

**5-second recall:**
- `right` scans forward one char at a time; `left` only ever moves forward too — never resets.
- `lastSeen[c]` = one past the index where `c` was last seen (`0` = never seen).
- If `lastSeen[c] > left`, that duplicate is *inside* the current window — jump `left` up to `lastSeen[c]`.
- Every step: record `maxLength = max(maxLength, right-left+1)`, then update `lastSeen[c] = right+1`.

```
   [ left ................................. right ]
        ←  window, guaranteed no duplicates inside  →

   char c at `right`:
     lastSeen[c] > left   →  left = lastSeen[c]     (dup is inside window — shrink)
     lastSeen[c] <= left  →  do nothing              (dup was outside window — ignore)
   then always:
     maxLength = max(maxLength, right - left + 1)
     lastSeen[c] = right + 1
```

```
 char:      a    b    c
 ascii idx: 97   98   99
                 
 lastSeen[] (size 128, all start at 0)
 
 idx 97 98 99 ... 127
     ┌──┬──┬──┬─────┐
     │ 1│ 2│ 0│ ... │   ← after seeing "ab"
     └──┴──┴──┴─────┘
       ↑last 'a' was      ↑'c' never
       at index 0,          seen → 0
       stored as 0+1=1
```

`lastSeen[c] = r + 1` → "one past where `c` last showed up." Default `0` means "never seen," so no separate init step is needed, and `left = Math.max(left, lastSeen[c])` just works.

```java
public int lengthOfLongestSubstring(String s) {
    int[] lastSeen = new int[128];  // ASCII index, default 0
    int left = 0, maxLength = 0;

for (int right = 0; right < s.length(); right++) {
   left = Math.max(left, lastSeen[s.charAt(right)]);       // shrink if duplicate is inside window
   maxLength = Math.max(maxLength, right - left + 1);      // record window size
   lastSeen[s.charAt(right)] = right + 1;                  // remember where this char was seen

   }

    return maxLength;
}
```

**Time complexity:** O(n) — the `right` pointer makes a single pass over the string, and `left` only ever moves forward (never resets), so together they do at most 2n steps total.

**Space complexity:** O(1) — `lastSeen` is a fixed `int[128]`, sized to the ASCII range, not to the input length.

# Rotating the Box

**Data structure:** 2D array (`char[][] outputGrid`)
**Technique:** Simulation (gravity drop + 90° matrix rotation in one pass)

**5-second recall:**
- Create `outputGrid` first, pre-filled entirely with `.` — that's the default for every cell, so empty space needs no explicit write.
- Scan each row of the OLD grid left → right, counting consecutive `#` stones as you go (`stoneCount`).
- Only `#` and `*` ever get written into `outputGrid` — `.` is already sitting there from the pre-fill, so it's never touched.
- Hit `*` → drop `stoneCount` `#`s just above it, write the `*` itself, reset `stoneCount = 0`.
- Hit end of row → drop `stoneCount` `#`s at the floor (bottom-most cell of that column).
- Wherever something gets written, its new location is `outputGrid[currentColumn][totalRows - currentRow - 1]` — the standard 90° clockwise rotation formula, applied in the same pass as the gravity drop.

```
   outputGrid starts as ALL '.'  (nothing to do for blank cells)

   scan row →:   #   .   *   #   #   .
                 └─┬─┘   ↑   └──┬──┘
              stoneCount   reset=0    stoneCount
              (before *)             (before row end)

   '#'        →  stoneCount++                        (no write yet)
   '.'        →  skip — already '.' in outputGrid     (no write)
   '*'        →  write stoneCount '#'s just above,    write '*' itself, stoneCount = 0
   end of row →  write stoneCount '#'s at the floor

   every write lands at: outputGrid[currentColumn][totalRows - currentRow - 1]
```

```
 inputGrid[currentRow]:   #    .    *    #    #    .
 currentColumn:           0    1    2    3    4    5

 scan left → right, counting '#' until '*' or end of row:
   currentColumn=0  '#'  → stoneCount = 1
   currentColumn=1  '.'  → skip
   currentColumn=2  '*'  → drop stoneCount(1) just above this obstacle, reset to 0
   currentColumn=3  '#'  → stoneCount = 1
   currentColumn=4  '#'  → stoneCount = 2
   currentColumn=5  '.'  → end of row → drop stoneCount(2) at the floor

 mapping:
 inputGrid[currentRow][currentColumn]  →  outputGrid[currentColumn][totalRows - currentRow - 1]

 outputGrid column (totalRows-currentRow-1), rows 0..totalColumns-1:
     row 0  ┌───┐
     row 1  │ # │   ← stone dropped just above the obstacle (from currentColumn=0)
     row 2  │ * │   ← obstacle itself (was at currentColumn=2)
     row 3  │ · │
     row 4  │ # │   ← stones dropped at the floor (from currentColumn=3,4)
     row 5  │ # │
            └───┘
```

The 90° clockwise rotation and the gravity simulation happen in one pass. `currentColumn` becomes the row index in `outputGrid`, and `totalRows - currentRow - 1` becomes the column index — that's the standard clockwise-rotation formula. Because increasing `currentColumn` (moving right in the original row) lines up with increasing row index in `outputGrid` (moving down), stones can be "dropped" directly while scanning, with no separate rotation step needed. `stoneCount` accumulates consecutive `'#'` and gets flushed the instant a `'*'` (drop just above it) or the row's end (drop at the floor) is hit.

```java
if (inputGrid[currentRow][currentColumn] == '#') {
stoneCount++;
        } else if (inputGrid[currentRow][currentColumn] == '*') {
outputGrid[currentColumn][totalRows - currentRow - 1] = '*';
fill(outputGrid, stoneCount, currentColumn - 1, totalRows - currentRow - 1); // drop above obstacle
stoneCount = 0;
        }
// after the row ends:
fill(outputGrid, stoneCount, totalColumns - 1, totalRows - currentRow - 1);       // drop at the floor
```

**Time complexity:** O(totalRows × totalColumns) — a single pass over every cell of `inputGrid`; the `fill` calls collectively touch each `outputGrid` cell at most once.

**Space complexity:** O(totalRows × totalColumns) — for `outputGrid`, which is required as the output regardless.

# Search a 2D Matrix (Binary Search)

**Data structure:** 2D array treated as a flattened array (no extra structure — just `low`/`high`/`mid` ints)
**Technique:** Binary Search

**5-second recall:**
- Rows sorted + each row's first value > previous row's last ⇒ the whole matrix reads as ONE sorted array, row-major.
- Binary search `[0, n*m - 1]` exactly like a normal array — just translate `mid` back to 2D.
- `row = mid / m`, `col = mid % m` — `m` (column count) is how many flat indices make up one row.
- Compare `mat[row][col]` to target, move `low`/`high` exactly like standard binary search.

```
   flat array:  [ 0  1  ...  m-1 | m  m+1  ...  2m-1 | ... ]
                 └── row 0 ──────┘ └──── row 1 ───────┘

   mid → row = mid / m     (which row)
   mid → col = mid % m     (position within that row)

   mat[row][col] == target  →  found
   mat[row][col]  < target  →  low  = mid + 1
   mat[row][col]  > target  →  high = mid - 1
```

```
 mat = [[ 1,  2,  3,  4],
        [ 5,  6,  7,  8],
        [ 9, 10, 11, 12]]     target = 8

 Rows are sorted left→right, and each row's first value is bigger
 than the previous row's last value — so read row-major and the
 whole matrix is just ONE sorted array in disguise:

 flat index:  0   1   2   3   4   5   6   7   8   9  10  11
 flat value:  1   2   3   4   5   6   7   8   9  10  11  12
                                      ↑
                                   target=8 lives at flat index 7

 low=0, high=11
   mid=5  → row=5/4=1, col=5%4=1 → mat[1][1]=6 < 8 → low=mid+1=6
   mid=8  → row=8/4=2, col=8%4=0 → mat[2][0]=9 > 8 → high=mid-1=7
   mid=7  → row=7/4=1, col=7%4=3 → mat[1][3]=8 == 8 → found!
```

There's no need to search row-by-row and then column-by-column. Since the matrix behaves like a single sorted array of `n * m` elements, a plain binary search works directly on the range `[0, n*m - 1]`. The only extra step is translating a 1-D `mid` back into 2-D coordinates: `row = mid / m` and `col = mid % m`, where `m` is the number of columns. That's the same row/col split you'd use to index into a flattened array — `m` tells you how many elements make up one "row's worth" before you wrap to the next.

```java
public boolean searchMatrix(int[][] mat, int target) {
    int n = mat.length;
    int m = mat[0].length;
    int low = 0, high = n * m - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        int row = mid / m;
        int col = mid % m;

        if (mat[row][col] == target) return true;
        else if (mat[row][col] < target) low = mid + 1;   // answer is to the right
        else high = mid - 1;                                // answer is to the left
    }
    return false;
}
```

**Time complexity:** O(log(n × m)) — standard binary search over a virtual array of `n * m` elements; the row/col translation is O(1) per step, so it doesn't change the order.

**Space complexity:** O(1) — only `low`, `high`, `mid`, `row`, `col` are used; no extra data structure.

# 102. Binary Tree Level Order Traversal (Medium) — Breadth-First Search

**Data structure:** Queue (`LinkedList` implementing `Queue<TreeNode>`)
**Technique:** Breadth-First Search (BFS)

Given the root of a binary tree, return the level order traversal of its nodes' values — left to right, level by level.

**5-second recall:**
- Queue starts with just `root`.
- Each `while` iteration = one tree level. Freeze `size = queue.size()` *before* popping — that's what stops levels from bleeding into each other.
- Pop exactly `size` nodes, record their values into `level`, enqueue their children (children always belong to the *next* level).
- After the inner loop finishes, push `level` into `result`.

```
   while queue not empty:
       size = queue.size()              ← freeze this level's width
       level = []
       repeat `size` times:
           node = queue.poll()
           level.add(node.val)
           enqueue node.left, node.right   (lands in queue for NEXT level)
       result.add(level)
```

```
        3
       / \
      9   20
         /  \
        15   7

 Process the queue one full level at a time: snapshot size = queue.size()
 BEFORE the inner loop, then pop exactly that many nodes. Anything enqueued
 during those pops belongs to the NEXT level, not this one.

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

The `size = queue.size()` line at the top of each `while` iteration is the whole trick — it's what turns a plain queue drain into a *level-by-level* traversal. Without it you'd still visit every node in the right order, but you'd lose the grouping, since nodes from the next level would already be sitting in the queue behind the current level's tail.

```java
import java.util.*;

public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();          // freeze this level's width before popping
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

**Time complexity:** O(n) — every node is enqueued and dequeued exactly once, where n is the number of nodes.

**Space complexity:** O(n) — the queue holds at most one full level at a time, which is O(n) in the worst case (e.g., a complete binary tree's bottom level).

# Bubble Sort

**Data structure:** Array (in-place, no auxiliary structure)
**Technique:** Bubble Sort (adjacent swap, comparison-based)

**5-second recall:**
- Outer loop `i`: one pass per iteration, unsorted zone shrinks by one each time.
- Inner loop `j`: walk `0 .. size-i-2`, comparing neighbors `arr[j]` vs `arr[j+1]`.
- Out of order (`arr[j] > arr[j+1]`) → swap them, set `swap = true`.
- End of pass: `swap == false` → array's already sorted, `break` early. `swap == true` → next pass, inner bound shrinks (`size-i-1`).

```
   pass i:  compare arr[j] vs arr[j+1]  for j = 0 .. size-i-2

   arr[j] > arr[j+1]   →  swap them, swap = true
   arr[j] <= arr[j+1]  →  leave alone

   end of pass:
     swap == false  →  break (already sorted)
     swap == true   →  next pass, bound shrinks by 1
```

```
 arr = [5, 1, 4, 2, 8]     [x] = the pair that just swapped

 Start
 ┌───┬───┬───┬───┬───┐
 │ 5 │ 1 │ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 Move 1 — swap idx 0,1  (5 > 1)
 ┌───┬───┬───┬───┬───┐
 │[1]│[5]│ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 Move 2 — swap idx 1,2  (5 > 4)
 ┌───┬───┬───┬───┬───┐
 │ 1 │[4]│[5]│ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 Move 3 — swap idx 2,3  (5 > 2)   ← pass 1 done, no swap on idx 3,4 (5 < 8)
 ┌───┬───┬───┬───┬───┐
 │ 1 │ 4 │[2]│[5]│ 8 │
 └───┴───┴───┴───┴───┘

 Move 4 — swap idx 1,2  (4 > 2)   ← pass 2 done, no swap on idx 2,3 (4 < 5)
 ┌───┬───┬───┬───┬───┐
 │ 1 │[2]│[4]│ 5 │ 8 │
 └───┴───┴───┴───┴───┘

 Pass 3 — no swaps at all (1<2, 2<4) → swap=false → break early

 Final
 ┌───┬───┬───┬───┬───┐
 │ 1 │ 2 │ 4 │ 5 │ 8 │
 └───┴───┴───┴───┴───┘
```

Each pass walks the unsorted portion left to right, swapping any pair that's out of order — the biggest value in that pass always ends up "bubbling" to the right edge of the unsorted zone, which is exactly why the inner loop bound shrinks by one each pass (`size - i - 1`): the last `i` elements are already settled and don't need rechecking. The `swap` flag is the early-exit optimization — if a full pass makes zero swaps, the array is already sorted and there's no point running the remaining passes, so it breaks out instead of grinding through them.

```java
public static int[] bubbleSort(int[] arr) {
    int size = arr.length;
    for (int i = 0; i < size; i++) {
        boolean swap = false;
        for (int j = 0; j < (size - i - 1); j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swap = true;
            }
        }
        if (!swap) {
            break;
        }
    }
    return arr;
}
```

**Time complexity:** O(n²) worst case (reverse-sorted input never triggers the early exit) — but O(n) best case on an already-sorted array, since the very first pass sets `swap=false` and breaks immediately.

**Space complexity:** O(1) — sorts in place, only `temp` and `swap` as extra variables.

# Insertion Sort

**Data structure:** Array (in-place, no auxiliary structure)
**Technique:** Insertion Sort (shift-and-insert, comparison-based)

**5-second recall:**
- Left of `i` is always sorted — that invariant is the whole idea.
- `key = arr[i]` — the card you just picked up.
- `j = i - 1` — last card in the sorted pile.
- While `j >= 0 && arr[j] > key`: slide `arr[j]` right (`arr[j+1] = arr[j]`), step `j` left.
- Loop stops two ways — found a smaller element, or ran off the front (`j < 0`). Either way: insert KEY on the right of the compared element `j` (`arr[j+1] = key`).

```
   [ ... sorted so far ..., arr[j] ]   KEY   [ arr[i+1] ... unsorted ]
                              ↑
                    compare arr[j] vs KEY

   arr[j] > KEY  →  slide arr[j] right, j--      arr[j+1] = arr[j]
   arr[j] < KEY  →  stop here                    insert KEY right of j
   j < 0         →  stop, hit the front          insert KEY right of j
                                                  (arr[j+1] = KEY either way)
```

```
 arr = [5, 1, 4, 2, 8]     [x] = the key value just inserted into place

 Start
 ┌───┬───┬───┬───┬───┐
 │ 5 │ 1 │ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 i=1: key=1 — shift 5 right, insert 1 at idx 0
 ┌───┬───┬───┬───┬───┐
 │[1]│ 5 │ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 i=2: key=4 — shift 5 right, insert 4 at idx 1
 ┌───┬───┬───┬───┬───┐
 │ 1 │[4]│ 5 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 i=3: key=2 — shift 5 then 4 right, insert 2 at idx 1
 ┌───┬───┬───┬───┬───┐
 │ 1 │[2]│ 4 │ 5 │ 8 │
 └───┴───┴───┴───┴───┘

 i=4: key=8 — nothing bigger to its left, no shifts, insert 8 at idx 4
 ┌───┬───┬───┬───┬───┐
 │ 1 │ 2 │ 4 │ 5 │[8]│
 └───┴───┴───┴───┴───┘
```

Bubble sort compares and swaps *neighbors* over and over. Insertion sort works differently — it's how you'd sort playing cards in your hand: pull the next card out (`key`), then slide it left past every card bigger than it, one shift at a time, until it lands in the gap where it belongs. The `while` loop is doing the sliding — `arr[j+1] = arr[j]` shifts a bigger element one slot right instead of swapping — and the loop exits the instant it hits something smaller or runs off the front of the array (`j >= 0`). That's why `i=4` above costs nothing: 8 is bigger than everything to its left, so the `while` condition fails immediately and the "shift" is really just dropping the key back where it started.

```java
public static void insertionSort(int[] arr) {
    int n = arr.length;
    // Loop from the second element up to the last element
    for (int i = 1; i < n; i++) {
        int key = arr[i];           // The element to be positioned
        int j = i - 1;

        // Move elements of arr[0..i-1] that are greater than key
        // to one position ahead of their current position
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j = j - 1;
        }

        // Insert the key into its correct sorted location
        arr[j + 1] = key;
    }
}
```

**Time complexity:** O(n²) worst case (reverse-sorted input — every key shifts all the way to the front) — but O(n) best case on an already-sorted array, since `arr[j] > key` fails on the very first check for every `i`.

**Space complexity:** O(1) — sorts in place; `key` and `j` are the only extra variables. Note this method returns `void` — unlike the bubble sort above, it mutates `arr` directly instead of returning it.

# Selection Sort

**Data structure:** Array (in-place, no auxiliary structure)
**Technique:** Selection Sort (find-min-then-swap, comparison-based)

**5-second recall:**
- Outer loop `i`: locks down index `i` as the slot getting filled this round.
- Inner loop `p`: scans the rest of the array (`i+1 .. end`) hunting for the smallest value's *index* — no swapping during the scan, just tracking `minIndex`.
- Scan finishes → exactly ONE swap: `arr[i]` ↔ `arr[minIndex]`.
- That means selection sort always does exactly `n` swaps — even when `i` already holds the minimum, that "swap" is just a no-op (`arr[i]` swapped with itself).

```
   i fixed  →  scan p = i+1 .. end, track minIndex   (no swapping yet)

   arr[p] < arr[minIndex]   →  minIndex = p
   arr[p] >= arr[minIndex]  →  keep looking

   scan done  →  ONE swap: arr[i] ↔ arr[minIndex]
```

```
 arr = [5, 1, 4, 2, 8]     [x] = the swap that closed out this i

 Start
 ┌───┬───┬───┬───┬───┐
 │ 5 │ 1 │ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 i=0: scan finds min=1 at idx 1 → swap idx 0,1
 ┌───┬───┬───┬───┬───┐
 │[1]│[5]│ 4 │ 2 │ 8 │
 └───┴───┴───┴───┴───┘

 i=1: scan finds min=2 at idx 3 → swap idx 1,3
 ┌───┬───┬───┬───┬───┐
 │ 1 │[2]│ 4 │[5]│ 8 │
 └───┴───┴───┴───┴───┘

 i=2: scan finds min=4 at idx 2 (itself)  → swap idx 2,2 — no-op
 i=3: scan finds min=5 at idx 3 (itself)  → swap idx 3,3 — no-op
 i=4: scan finds min=8 at idx 4 (itself)  → swap idx 4,4 — no-op

 Final
 ┌───┬───┬───┬───┬───┐
 │ 1 │ 2 │ 4 │ 5 │ 8 │
 └───┴───┴───┴───┴───┘
```

This flips the other two algorithms around. Bubble sort and insertion sort both move elements *toward* where they belong, one step or one swap at a time. Selection sort instead spends its whole inner loop just looking — `minIndex` is the only thing that changes while scanning, the array itself is untouched — and only writes to the array once per outer-loop pass, right at the end. That's why it always costs exactly `n` swaps no matter what the input looks like: even a fully-sorted array still runs every comparison and still "swaps" each element with itself. The cost lives entirely in the comparisons, not the writes — the opposite trade-off from insertion sort, which skips comparisons on a sorted array but can do many shifts on a reversed one.

```java
public static int[] selectionSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int minIndex = i;
        for (int p = (i + 1); p < arr.length; p++) {
            if (arr[p] < arr[minIndex]) {
                minIndex = p;
            }
        }
        int temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
    return arr;
}
```

**Time complexity:** O(n²) — always, best case included. The inner loop scans the full remaining range every single pass regardless of how sorted the array already is; there's no early-exit condition like bubble sort's `swap` flag.

**Space complexity:** O(1) — sorts in place, only `minIndex` and `temp` as extra variables.
# 84. Largest Rectangle in Histogram (Hard) — Monotonic Stack

**Data structure:** Stack (`Stack<int[]>` holding `[start, height]` pairs)
**Technique:** Monotonic Stack

Given an array of bar heights (each bar width 1), return the area of the largest rectangle that fits in the histogram.

**5-second recall:**
- Stack holds `[start, h]` pairs — every bar still in the stack is part of a non-decreasing run, so its rectangle could still grow rightward.
- At each `i`, while `stack.peek()[1] > h`: that taller bar can't extend any further right — pop it, close it out.
- Closing a bar: `area = max(area, (i - previousIndex) * previousElement[1])` — width is "how far right it reached" minus "where it started."
- Popping also drags `start` back to `previousIndex` — the current bar inherits the popped bar's start, because it can extend left through anything shorter than itself was.
- Push `[start, h]` (not `[i, h]`) — that inherited `start` is what lets the current bar's future rectangle reach back through the bars it just outlasted.
- Leftover stack after the loop → same closing formula, just with `n` standing in for `i` (everything left can extend all the way to the end).

```
   stack: [start, h] pairs, heights strictly increasing bottom→top

   at index i, height h:
     while stack.peek().h > h:
         pop [previousIndex, previousHeight]
         area = max(area, (i - previousIndex) * previousHeight)   // right wall = i
         start = previousIndex                                     // inherit left reach
     push [start, h]

   after loop (i == n, the imaginary right wall):
     while stack not empty:
         pop [idx, h]
         area = max(area, (n - idx) * h)
```

```
 heights = [2, 1, 5, 6, 2, 3]      index:  0  1  2  3  4  5
 [x] = bar just popped and closed out

 i=0  h=2            stack empty → push [0,2]
      stack: [0,2]

 i=1  h=1            peek=[0,2], 2>1 → pop[0,2] (start=0)
                        area = max(0, (1-0)*2) = 2
                      stack empty → push [0,1]        (start inherited = 0)
      stack: [0,1]

 i=2  h=5            peek=[0,1], 1>5? no → push [2,5]
      stack: [0,1] [2,5]

 i=3  h=6            peek=[2,5], 5>6? no → push [3,6]
      stack: [0,1] [2,5] [3,6]

 i=4  h=2            peek=[3,6], 6>2 → pop[3,6] (start=3)
                        area = max(2,  (4-3)*6) = 6
                      peek=[2,5], 5>2 → pop[2,5] (start=2)
                        area = max(6,  (4-2)*5) = 10
                      peek=[0,1], 1>2? no → push [2,2]   (start inherited = 2)
      stack: [0,1] [2,2]

 i=5  h=3            peek=[2,2], 2>3? no → push [5,3]
      stack: [0,1] [2,2] [5,3]

 end of array, n=6 — drain the stack, right wall = n for everyone left:
      pop[5,3] → area = max(10, (6-5)*3) = max(10,3)  = 10
      pop[2,2] → area = max(10, (6-2)*2) = max(10,8)  = 10
      pop[0,1] → area = max(10, (6-0)*1) = max(10,6)  = 10

 Final area = 10
```

The stack only ever holds bars in non-decreasing height order, and that's the whole invariant: as long as a bar is sitting in the stack, every bar to its right seen so far has been tall enough not to disturb it. The moment a shorter bar shows up at `i`, every taller bar sitting above it in the stack has just met its right wall — it can't stretch past `i`, so its rectangle gets finalized right there with `area = (i - previousIndex) * previousElement[1]`. The `start` inheritance is the part that's easy to miss: when bar B (height 5) gets popped by a shorter bar, the shorter bar doesn't just take height 2 — it also takes B's `start`, because the new shorter bar was standing right where B used to reach back to. That's how one `push` per index still ends up encoding "how far back could this bar's rectangle have started," without ever explicitly scanning backward. The final drain loop is just the same closing step applied to whatever's left once there's no more shorter bar to trigger it — the end of the array acts as an infinitely short bar closing everything out at once, with `n` standing in for `i`.

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        // All heights left in the stack are non-decreasing, so each could
        // still extend right — pop only when a shorter bar shows up.
        Stack<int[]> stack = new Stack<>();
        int n = heights.length;
        int area = 0;

        for (int i = 0; i < n; i++) {
            int h = heights[i];
            int start = i;
            // Pop and close out every bar taller than the current one —
            // it can't extend past index i.
            while (!stack.isEmpty() && stack.peek()[1] > h) {
                int[] previousElement = stack.pop();
                int previousIndex = previousElement[0];
                area = Math.max(area, (i - previousIndex) * previousElement[1]);
                start = previousIndex;   // inherit how far left this run reaches
            }
            stack.push(new int[]{start, h});
        }

        // Nothing left to trigger a pop — close out the rest against n.
        while (!stack.isEmpty()) {
            int[] element = stack.pop();
            area = Math.max(area, element[1] * (n - element[0]));
        }

        return area;
    }
}
```

**Time complexity:** O(n) — each index is pushed onto the stack exactly once and popped at most once, so the total work across the whole loop (not per-index) is linear.

**Space complexity:** O(n) — worst case (strictly increasing heights) every bar sits in the stack at once before the final drain.

# 42. Trapping Rain Water (Hard) — Two Pointer

**Data structure:** Array (no stack — just `left`/`right`/`leftMax`/`rightMax` ints)
**Technique:** Two Pointer

Given an array of bar heights, find how much water gets trapped between them after rain.

**5-second recall:**
- No stack. Two pointers `left`, `right` close in from both ends; track `leftMax`, `rightMax` as running maxes seen so far on each side.
- Compare `heights[left]` vs `heights[right]` — whichever is *smaller* decides which side to process (the shorter bar's own running max is guaranteed to be the binding constraint, since the other side's max is at least as big as the taller bar).
- Process the smaller side: update its running max, add `runningMax - heights[pointer]` to water, move that pointer inward.
- Loop while `left < right`.

```
   left=0, right=n-1, leftMax=0, rightMax=0, water=0

   while left < right:
     if heights[left] < heights[right]:
         leftMax = max(leftMax, heights[left])
         water += leftMax - heights[left]
         left++
     else:
         rightMax = max(rightMax, heights[right])
         water += rightMax - heights[right]
         right--
```

```java
class Solution {
    public int trap(int[] heights) {
        int left = 0, right = heights.length - 1;
        int leftMax = 0, rightMax = 0;
        int water = 0;

        while (left < right) {
            if (heights[left] < heights[right]) {
                leftMax = Math.max(leftMax, heights[left]);
                water += leftMax - heights[left];
                left++;
            } else {
                rightMax = Math.max(rightMax, heights[right]);
                water += rightMax - heights[right];
                right--;
            }
        }

        return water;
    }
}
```

Why it's safe to trust `leftMax` alone when `heights[left] < heights[right]`: whatever `rightMax` turns out to be, it's at least `heights[right]`, which is already bigger than `heights[left]`. So `leftMax` is the real bound at `left` regardless of what's further right — no need to know `rightMax`'s exact value.

Dry run on `[0,1,0,2,1,0,1,3,2,1,2,1]`: final `water = 6`, driven mainly by the dips at indices 2, 5, and 6 (added 1, 2, 1 respectively) plus indices 4 and 9 (added 1 each).

**Time complexity:** O(n) — each pointer moves inward once per step, together covering the array in a single pass.

**Space complexity:** O(1) — only `left`, `right`, `leftMax`, `rightMax`, `water` as extra variables.
