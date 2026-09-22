# Algorithm
个人模板
# 数据结构
#### 二维前缀和
```cpp
//二维前缀和
class NumMatrix {
    Matrix sum;
public:
    NumMatrix(Matrix& mat) {
        i32 m = mat.size(), n = mat[0].size();
        sum.resize(m + 1, vector<i32>(n + 1));
        for (i32 i = 0; i < m; i++) {
            for (i32 j = 0; j < n; j++) {
                sum[i + 1][j + 1] = sum[i + 1][j] + sum[i][j + 1] - sum[i][j] + mat[i][j];
            }
        }
    }

    // 返回左上角在 (r1, c1)，右下角在 (r2, c2) 的子矩阵元素和
    i32 sumRegion(i32 r1, i32 c1, i32 r2, i32 c2) {
        return sum[r2 + 1][c2 + 1] - sum[r2 + 1][c1] - sum[r1][c2 + 1] + sum[r1][c1];
    }
};
```

#### 并查集
```cpp
//并查集模板
class UnionFind {
    std::vector<int> fa;
    std::vector<int> sz;

public:
    int cc;

    UnionFind(int n) : fa(n), sz(n, 1), cc(n) {
        std::ranges::iota(fa, 0);
    }

    int find(int x) {
        if (fa[x] != x) fa[x] = find(fa[x]);
        return fa[x];
    }

    bool is_same(int x, int y) {
        return find(x) == find(y);
    }

    bool merge(int from, int to) {
        int x = find(from), y = find(to);
        if (x == y) return false;
        fa[x] = y;
        sz[y] += sz[x];
        cc--;
        return true;
    }

    int get_size(int x) {
        return sz[find(x)];
    }
};
```

#### 树状数组
``` cpp
//普通树状数组模板
template<typename T>
class FenwickTree {
    std::vector<T> tree;

public:
    FenwickTree(int n) : tree(n + 1) {}

    void update(int i, T val) {
        for (; i < tree.size(); i += i & -i)
            tree[i] += val;
    }

    T pre(int i) const {
        T res{};
        for (; i > 0; i &= i - 1)
            res += tree[i];
        return res;
    }

    T query(int l, int r) const {
        if (r < l) return 0;
        return pre(r) - pre(l - 1);
    }
};

//值域树状数组
class FenwickTree {
    const vector<int>& sorted;
    const int high_bit;
    vector<int> cnt;
    vector<long long> sum;

public:
    FenwickTree(const vector<int>& sorted) :
        sorted(sorted),
        high_bit(1 << (bit_width(sorted.size()) - 1)),
        cnt(sorted.size() + 1),
        sum(sorted.size() + 1) {}

    // 添加 num 个 val，其中 val 离散化后的值为 i（i 从 1 开始）
    // 如果 num < 0，表示减少 -num 个 val
    // 注意 val = sorted[i - 1]，无需手动传入
    void update(int i, int num) {
        auto val = sorted[i - 1];
        for (; i < cnt.size(); i += i & -i) {
            cnt[i] += num;
            sum[i] += 1LL * num * val;
        }
    }

    // 返回第 k 小的数（k 从 1 开始）
    int kth(int k) const {
        int i = 0;
        for (int b = high_bit; b > 0; b >>= 1) {
            int nxt = i | b;
            if (nxt < cnt.size() && cnt[nxt] < k) {
                k -= cnt[nxt];
                i = nxt;
            }
        }
        return sorted[i];
    }

    // 返回前 k 小的数之和（k 从 1 开始）
    long long pre_sum(int k) const {
        long long s = 0;
        int i = 0;
        for (int b = high_bit; b > 0; b >>= 1) {
            int nxt = i | b;
            if (nxt < cnt.size() && cnt[nxt] < k) {
                k -= cnt[nxt];
                s += sum[nxt];
                i = nxt;
            }
        }
        // 加上等于第 k 小的数
        return s + 1LL * sorted[i] * k;
    }
};
```

#### 线段树
``` cpp
//线段树
template<typename T>
class SegmentTree {
    int n;
    std::vector<T> tree;

    T merge_val(T a, T b) const { // 合并两个 val
        return std::max(a, b);
    }

    void maintain(int o) {  // 合并左右儿子的 val 到当前节点的 val
        tree[o] = merge_val(tree[o * 2], tree[o * 2 + 1]);
    }

    void build(const std::vector<T>& a, int o, int l, int r) { // 用 a 初始化线段树
        if (l == r) { // 叶子
            tree[o] = a[l]; // 初始化叶节点的值
            return;
        }
        int m = (l + r) >> 1;
        build(a, o * 2, l, m); // 初始化左子树
        build(a, o * 2 + 1, m + 1, r); // 初始化右子树
        maintain(o);
    }

    void update(int o, int l, int r, int i, T val) {
        if (l == r) { // 叶子（到达目标）
            tree[o] = merge_val(tree[o], val); // 如果想直接替换的话，可以写 tree[node] = val
            return;
        }
        int m = (l + r) >> 1;
        if (i <= m) update(o * 2, l, m, i, val);// i 在左子树
        else update(o * 2 + 1, m + 1, r, i, val);// i 在右子树
        maintain(o);
    }

    T query(int o, int l, int r, int L, int R) const {
        if (L <= l && r <= R) return tree[o]; // 当前子树完全在 [ql, qr] 内
        int m = (l + r) >> 1;
        if (R <= m) return query(o * 2, l, m, L, R); // [ql, qr] 与右子树无交集，仅需递归左子树
        if (L > m) return query(o * 2 + 1, m + 1, r, L, R);// [ql, qr] 与左子树无交集，仅需递归右子树
        // [ql, qr] 与左右子树均有交集，分别递归，然后合并结果
        T l_res = query(o * 2, l, m, L, R);
        T r_res = query(o * 2 + 1, m + 1, r, L, R);
        return merge_val(l_res, r_res);
    }
public:
    SegmentTree(int n, T init_val) : SegmentTree(vector<T>(n, init_val)) {}

    SegmentTree(const std::vector<T>& a) : n(a.size()), tree(2 << std::bit_width(a.size() - 1)) {
        build(a, 1, 0, n - 1);
    }

    void update(int i, T val) { // 更新 a[i] 为 merge_val(a[i], val)
        update(1, 0, n - 1, i, val);
    }

    T query(int L, int R) const { // 返回用 merge_val 合并所有 a[i] 的计算结果，其中 i 在闭区间 [ql, qr] 中
        return query(1, 0, n - 1, L, R);
    }

    T get(int i) const { // 获取 a[i] 的值
        return query(1, 0, n - 1, i, i);
    }
};

//lazy线段树
template<typename T, typename F>
class LazySegmentTree {
    static constexpr F TODO_INIT = 0; //默认值

    struct Node { //一个结构体集结lazy数组和普通数组
        T val;
        F todo;
    };
    int n;
    std::vector<Node> tree;

    T merge_val(const T& a, const T& b) const { //合并两个数
        return std::max(a, b);
    }

    F merge_todo(const F& a, const F& b) const { //合并懒加载
        return a + b;
    }

    void apply(int o, int l, int r, F todo) { //懒加载标记后续不再遍历
        Node& cur = tree[o];
        cur.val += todo;
        cur.todo = merge_todo(todo, cur.todo);
    }

    void spread(int o, int l, int r) { //下传懒标记
        Node& cur = tree[o];
        F todo = cur.todo;
        if (todo == TODO_INIT) return;
        int m = (l + r) >> 1;
        apply(o * 2, l, m, todo);
        apply(o * 2 + 1, m + 1, r, todo);
        cur.todo = TODO_INIT;
    }

    void maintain(int o) { //维护父亲节点
        tree[o].val = merge_val(tree[o * 2].val, tree[o * 2 + 1].val);
    }

    void build(const std::vector<T>& a, int o, int l, int r) {
        Node& cur = tree[o];
        cur.todo = TODO_INIT;
        if (l == r) {
            cur.val = a[l];
            return;
        }
        int m = (l + r) >> 1;
        build(a, o * 2, l, m);
        build(a, o * 2 + 1, m + 1, r);
        maintain(o);
    }

    void update(int o, int l, int r, int L, int R, F f) {
        if (L <= l && r <= R) { //如果区间已经在需要找的内部
            apply(o, l, r, f);
            return;
        }
        spread(o, l, r);
        int m = (l + r) >> 1;
        if (L <= m) update(o * 2, l, m, L, R, f);
        if (R > m) update(o * 2 + 1, m + 1, r, L, R, f);
        maintain(o);
    }

    T query(int o, int l, int r, int L, int R) {
        if (L <= l && r <= R) return tree[o].val;
        spread(o, l, r);
        int m = (l + r) >> 1;
        if (R <= m) return query(o * 2, l, m, L, R);
        if (L > m) return query(o * 2 + 1, m + 1, r, L, R);
        T l_res = query(o * 2, l, m, L, R);
        T r_res = query(o * 2 + 1, m + 1, r, L, R);
        return merge_val(l_res, r_res);
    }

public:
    LazySegmentTree(int n, T init_val = 0) : LazySegmentTree(std::vector<T>(n, init_val)) {}

    LazySegmentTree(const std::vector<T>& a) : n(a.size()), tree(2 << std::bit_width(a.size() - 1)) {
        build(a, 1, 0, n - 1);
    }

    void update(int L, int R, F f) {
        update(1, 0, n - 1, L, R, f);
    }

    T query(int L, int R) {
        return query(1, 0, n - 1, L, R);
    }
};
```

#### st表
```cpp
class SparseTable {
    vector<vector<int>> st_min;
    vector<vector<int>> st_max;

public:
    SparseTable(const vector<int>& nums) {
        int n = nums.size();
        int w = bit_width((uint32_t) n);
        st_min.resize(w, vector<int>(n));
        st_max.resize(w, vector<int>(n));

        for (int j = 0; j < n; j++) {
            st_min[0][j] = nums[j];
            st_max[0][j] = nums[j];
        }

        for (int i = 1; i < w; i++) {
            for (int j = 0; j <= n - (1 << i); j++) {
                st_min[i][j] = min(st_min[i - 1][j], st_min[i - 1][j + (1 << (i - 1))]);
                st_max[i][j] = max(st_max[i - 1][j], st_max[i - 1][j + (1 << (i - 1))]);
            }
        }
    }

    int query_min(int l, int r) const {
        int k = bit_width((uint32_t) r - l) - 1;
        return min(st_min[k][l], st_min[k][r - (1 << k)]);
    }
    
    int query_max(int l, int r) const {
        int k = bit_width((uint32_t) r - l) - 1;
        return max(st_max[k][l], st_max[k][r - (1 << k)]);
    }
};
```
