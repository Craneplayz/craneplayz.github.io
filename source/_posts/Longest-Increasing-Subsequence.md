---
title: Longest Increasing Subsequence
date: 2023-05-21 02:13:45
tags: Competitive Programming
---

# Longest Increasing Subsequence
Longest Increasing Subsequence 问题指的是在一个给定的Array中寻找所有元素都递增的Subsequence中最长的

### Naive Approach

> **Time Complexity: $O(n^2)$**
> 

假设 dp[i] 是以 a_i 结尾的 LIS 长度

那样 ans = dp 中最大的那个

#### Transition

`**dp[i] =**` 

`**max(dp[j] + 1)**` where **`a[j] < a[i]`**

> 尝试把新的`**a[i]**`接在现有的LIS后面
> 

4 5 3 2 9 8 7

dp[0] = 1 

从 4 开始 如果今天4

---

#### Code

```cpp
vector<int> a, ans, parent;

void GetPath(int i){
   ans.push_back(a[i]);
   if (parent[i] == -1)
      return;
   return GetPath(parent[i]);
}

int main(){
   int n; 
   cin >> n;
   a.assign(n, -1);
   for (int i = 0; i < n; i++) cin >> a[i];

   vector<int> dp(n, 1);
   parent.assign(n, -1);

   for (int i = 0; i < n; i++)
      for (int j = 0; j < i; j++)
         if ( a[j] <= a[i] && dp[i] < dp[j]+1){
            dp[i] = dp[j]+1;
            parent[i] = j;
         }

   int ansIndex = max_element(dp.begin(), dp.end()) - dp.begin();
   GetPath(ansIndex);
   reverse(ans.begin(), ans.end());
   for (int i : ans)
      cout << i << " ";
   cout << "\n";
   
   return 0;
}
```

---

## Approach 2 - Binary Search

### Observation

LIS中的元素必定是**Sorted**了的，由小至大

### State

$$
dp[i]=len
$$

- `**a[1..n]**`:  原始序列
- `**dp[k]**`: **长度为k的LIS末尾元素的最小值**
- `**len**`: 当前已知的最长子序列的长度

### Transistion

- 每次操作时均替换掉最有可能被替换掉的元素，
也就是**第一个大于等于要放入的元素的元素
`lower_bound` ——  $O(\log{n})$**
- 如果这个元素大于LIS里的任一一个元素，那么替换相当于放入多一个元素
反之，替换之后不影响当前的长度

> **意义**
> 
> 
> 这样的操作降低了未来元素变成“Increasing”的难度，
> 新的元素更容易变成 Increasing，也更容易Append多一个元素在LIS末端
> 
> 替换的过程相当于开另一条subseqeunce
> 替换的元素之前的保持一样，之后的则放弃掉
> 之后的过程相当于接上新的Subsequence
> 
- ~~如此存取的`**dp**`并不是LIS本体，基本上也不太可能通过GetPath的方式回溯相关的LIS~~

### Time Complexity

$O(n\log{n})$

---

## Code

```cpp
void solve(){
    int n;
    vector<int> a(n);

    int lis = 0;
    vector<int> dp(n, INT_MAX);

    for (int i = 0; i < n; i++){
        auto k = lower_bound(dp.begin(), dp.end(), a[i]); // Increasing(<)
				// Use "upper_bound" for Non-Decreasing(<=)
        
        if (*k == INT_MAX)
            lis++;
        
        *k = min(*k, a[i]);
    }

    cout << lis << '\n';
}
```

## Problems

[Increasing Subsequence](https://www.notion.so/Increasing-Subsequence-8d447be1681748c68fd570504b223a5a) 

---

# Approach 3 - Segment Tree

### Intuition

计算 max 的时间花太多了，可以改用 **Segment Tree** 来节省

空间也不需要到 2D，可以直接压缩在1D

### Normalization

Map the values in array into range [1-N] by increasing sequence

[Coordinate Compression](https://www.notion.so/Coordinate-Compression-19a67d31dd0e466994c2d575d1716edc) 

### State

`**dp[i]**` = 
Length of LIS ending with element `**j**` (after Normalization)

### Transistion

Manage `**dp**` array with **Segment Tree** (as **underlying array**)

`**dp[a[i]] = query(1, a[i] - 1) + 1**`

Update with the maximum of prefix **`dp[1..a[j-1]] + 1`**
(Adding 1 element to the end of LIS)

### Answer

`**query(1, n)**`
The Maximum length of LIS considering from which ending with element `1` to `N` respectively

### Time Complexity

$\text{O}(n\log{n})$

### Code

```cpp
class SegmentTree {
public:
    /// @brief Constructor without providing underlying array
    /// @param n Length of underlying array
    /// @param dummy Length of dummy value for underlying array and invalid segments
    SegmentTree(const int n, const int dummy = -1)
        : _n(n), _dummy(dummy) {
        _tree.assign(4 * _n + 1, _dummy);
    }

    /// @brief Constructor
    /// @param a Underlying array
    /// @param dummy Length of dummy value for underlying array and invalid segments
    SegmentTree(const vector<int> &a, const int dummy = -1)
        : _n(a.size()), _dummy(dummy), _tree(4 * _n + 1, dummy) {
        for (int i = 0; i < _n; i++)
            update(i, a[i]);
    }

    /// @brief Point update
    /// @param i Index of underlying array to be updated - (1-th based)
    /// @param val Value to be updated
    void update(const int i, const int val) {
        update(1, 1, _n, i, val);
    }

    /// @brief Query in range [i, j), public wrapper
    /// @param i Lower bound of query segment, inclusive, 1-th based index
    /// @param j Upper bound of query segment, inclusive, 1-th based index
    /// @return Range query
    int query(const int i, const int j) {
        return query(1, 1, _n, i, j);
    }

private:
    const int _n, _dummy;
    vector<int> _tree; // Array constituting the tree

    /// @brief Traverse to left children
    /// @param p Parent's position
    /// @return Left child's position - 2p 
    inline int left(const int p) {
        return (p << 1);
    }

    /// @brief Traverse to right child
    /// @param p Parent's position
    /// @return Right child's position - 2p + 1
    inline int right(const int p) {
        return (p << 1) + 1;
    }

    /// @brief Denoting what operation should be used to merge two children
    /// @param x Node 1 
    /// @param y Node 2
    /// @return Result of merging
    inline int join(const int x, const int y) {
        return max(x, y);
    }

    /// @brief Point update
    /// @param p Position of current node
    /// @param L Lower bound reprsented by Segment Tree, inclusive, 1-th based index
    /// @param R Upper bound represented by Segment Tree, exclusive, 1-th based index
    /// @param i Index to be updated, 1-th based index
    /// @param val Value to be updated
    void update(int p, int L, int R, const int i, const int val) {
        // Return if L and R only differs by one - 
        // segment contains single element
        if (L >= R - 1) {
            _tree[p] = val;
            return;
        }

        int mid = L + ((R - L) >> 1);

        if (i < mid) // Traversing left
            update(left(p), L, mid, i, val);
        else // Traversing right
            update(right(p), mid, R, i, val);

        // Merging result from children
        _tree[p] = join(_tree[left(p)], _tree[right(p)]);
    }

    /// @brief Query in range [i, j)
    /// @param p Position of current node
    /// @param L Lower bound reprsented by Segment Tree, inclusive, 1-th based index
    /// @param R Upper bound represented by Segment Tree, exclusive, 1-th based index
    /// @param i Lower bound of query segment, inclusive, 1-th based index
    /// @param j Upper bound of query segment, inclusive, 1-th based index
    /// @return Range query
    int query(int p, int L, int R, const int i, const int j) {
        // Return default value if segment is entirely out of query range
        if (j <= L || R < i)
            return _dummy;
        
        // Return value of node if segment is entirely in query range
        if (i <= L && R <= j)
            return _tree[p];

        // Traverse to node's children if segment is partially in query range
        int mid = L + ((R - L) >> 1);
        int node1 = query(left(p), L, mid, i, j);
        int node2 = query(right(p), mid, R, i, j);

        // Merging results from children
        return join(node1, node2);
    }
};

/// @brief   Coordinate Compression to range [1, N]
///          where N is the number of distinct element in the array
/// @param v Original array
void Normalize(vector<int> &v) {
    vector<int> aux = v;
    sort(aux.begin(), aux.end());

    // Removing duplicates
    auto itUnique = unique(aux.begin(), aux.end());
    aux.erase(itUnique, aux.end());

    // Updating elements with their positions in sorted
    // auxiliary array, 1-th based index 
    for (int i = 0; i < (int)v.size(); i++)
        v[i] = lower_bound(aux.begin(), aux.end(), v[i]) - aux.begin() + 1;
}

int LongestIncreasingSubsequence() {
    vector<int> a;
    const int n = a.size();

    // Normalize a to [1, N]
    Normalize(a);

    SegmentTree segmentTree(n, 0);
    for (int i = 0; i < n; i++) {
        // Querying segment [1, a[i])
        // Note that the upper bound is exclusive
        // This retrieves the maximum in range [1, a[i] - 1] 
        int result = segmentTree.query(1, a[i]); 

        // Update element a[i] of dp array to be RMQ(1, a[i]) + 1
        segmentTree.update(a[i], result + 1);
    }

    // This retrieves the maximum in range [1, N] - length of LIS
    return segmentTree.query(1, n + 1);
}
```
