## 转化 PDF

使用 Markdown PDF 插件，设置 Header Template 为：

```plain
<div style="font-size: 9px; margin-left: auto; margin-right: 1cm; ">%%ISO-DATE%%</div>
```



## Markdown 文本

!!! notes "板子"

    ````markdown
    ## 字符串

    ### KMP

    ```cpp
    for (int i = 1; i < n; ++i) {
        nxt[i] = nxt[i - 1];
        while (nxt[i] && s[i] != s[nxt[i]])
            nxt[i] = nxt[nxt[i] - 1];
        if (s[i] == s[nxt[i]])
            ++nxt[i];
    }
    ```

    ```cpp
    for (int i = 1; i < m; ++i) {
        nxt[i] = nxt[i - 1];
        while (nxt[i] && b[i] != b[nxt[i]])
            nxt[i] = nxt[nxt[i] - 1];
        if (b[i] == b[nxt[i]])
            ++nxt[i];
    }
    for (int i = 0, j = 0; i < n; ++i) {
        while (j && a[i] != b[j])
            j = nxt[j - 1];
        if (a[i] == b[j])
            ++j;
        if (j == m)
            // do something
    }
    ```

    ### Z 函数

    ```cpp
    for (int i = 1, l = 0, r = 0; i < n; ++i) {
        if (i <= r && z[i - l] < r - i + 1)
            z[i] = z[i - l];
        else {
            z[i] = max(0, r - i + 1);
            while (i + z[i] < n && s[z[i]] == s[i + z[i]])
                ++z[i];
            l = i, r = i + z[i] - 1;
        }
    }
    z[0] = n;
    ```

    ### AC 自动机

    ```cpp
    class AC {
    private:
        int tot, deg[N], fail[N], son[N][C];

    public:
        void clear() {
            for (int i = 0; i <= tot; ++i) {
                fail[i] = 0;
                for (int j = 0; j < C; ++j)
                    son[i][j] = 0;
            }
            tot = 0;
        }
        void insert(const string &s) {
            int u = 0;
            for (int i = 0; i < s.length(); u = son[u][s[i] - 'a'], ++i)
                if (!son[u][s[i] - 'a'])
                    son[u][s[i] - 'a'] = ++tot;
        }
        void build() {
            queue<int> q;
            for (int i = 0; i < C; ++i)
                if (son[0][i])
                    q.push(son[0][i]);
            for (; !q.empty(); q.pop()) {
                int u = q.front();
                for (int i = 0; i < C; ++i)
                    if (son[u][i])
                        fail[son[u][i]] = son[fail[u]][i], ++deg[son[fail[u]][i]], q.push(son[u][i]);
                    else
                        son[u][i] = son[fail[u]][i];
            }
        }
        void query(const string &s) {
            for (int i = 0, u = 0; i < s.length(); ++i)
                u = son[u][s[i] - 'a'], ++cnt[u];
            queue<int> q;
            for (int i = 0; i <= tot; ++i)
                if (!deg[i])
                    q.push(i);
            for (; !q.empty(); q.pop()) {
                int u = q.front();
                // do something
                if (!--deg[fail[u]])
                    q.push(fail[u]);
            }
        }
    };
    ```

    ### 后缀数组

    ```cpp
    int n, ht[N], sa[N], cnt[N];
    vector<int> rk(N, 0), tp(N, 0);
    string s;
    void init() {
        int m = 0;
        for (int i = 1; i <= n; ++i)
            ++cnt[rk[i] = s[i - 1]], m = max(m, rk[i]);
        for (int i = 1; i <= m; ++i)
            cnt[i] += cnt[i - 1];
        for (int i = n; i; --i)
            sa[cnt[rk[i]]--] = i;
        for (int i = 1; i <= m; ++i)
            cnt[i] = 0;
        for (int w = 1, p = 0; p < n; m = p, w <<= 1) {
            p = 0;
            for (int i = n - w + 1; i <= n; ++i)
                tp[++p] = i;
            for (int i = 1; i <= n; ++i)
                if (sa[i] > w)
                    tp[++p] = sa[i] - w;
            for (int i = 1; i <= n; ++i)
                ++cnt[rk[i]];
            for (int i = 1; i <= m; ++i)
                cnt[i] += cnt[i - 1];
            for (int i = n; i; --i)
                sa[cnt[rk[tp[i]]]--] = tp[i];
            for (int i = 1; i <= m; ++i)
                cnt[i] = 0;
            p = 0, swap(rk, tp);
            for (int i = 1; i <= n; ++i)
                rk[sa[i]] = tp[sa[i]] == tp[sa[i - 1]] && (min(sa[i], sa[i - 1]) + w > n || (max(sa[i], sa[i - 1]) + w <= n && tp[sa[i] + w] == tp[sa[i - 1] + w])) ? p : ++p;
        }
        for (int i = 1, j = 0; i <= n; ++i) {
            if (rk[i] == 1)
                continue;
            if (j)
                --j;
            while (max(i, sa[rk[i] - 1]) + j <= n && s[i + j - 1] == s[sa[rk[i] - 1] + j - 1])
                ++j;
            ht[rk[i]] = j;
        }
    }
    ```

    $$
    lcp(sa_i, sa_j) = \min_{t = i + 1} ^ j\{ht_t\} \\
    $$

    ### 后缀自动机

    ```cpp
    class SAM {
    private:
        static const int M = N * 2;
        int tot, lst, fa[M], cnt[M], len[M], son[M][26];
        vector<int> edg[M];
        void dfs(int u) {
            for (int v : edg[u])
                dfs(v), cnt[u] += cnt[v];
            if (cnt[u] > 1)
                ans = max(ans, 1ll * len[u] * cnt[u]);
        }

    public:
        SAM() : tot(1), lst(1) {}
        void insert(int c) {
            c -= 'a';
            int u = lst, v = lst = ++tot;
            for (cnt[v] = 1, len[v] = len[u] + 1; u && !son[u][c]; u = fa[u])
                son[u][c] = v;
            if (!u)
                fa[v] = 1;
            else {
                int x = son[u][c];
                if (len[x] == len[u] + 1)
                    fa[v] = x;
                else {
                    int y = ++tot;
                    fa[y] = fa[x], len[y] = len[u] + 1;
                    for (int i = 0; i < 26; ++i)
                        son[y][i] = son[x][i];
                    for (fa[x] = fa[v] = y; u && son[u][c] == x; u = fa[u])
                        son[u][c] = y;
                }
            }
        }
        void init() {
            for (int i = 2; i <= tot; ++i)
                edg[fa[i]].push_back(i);
            dfs(1);
        }
    };
    ```

    ### Manacher

    ```cpp
    for (int i = 0, l = 0, r = -1; i < n; ++i) {
        d1[i] = i > r ? 1 : min(d1[l + r - i], r - i + 1);
        while (i - d1[i] >= 0 && i + d1[i] < n && s[i - d1[i]] == s[i + d1[i]])
            ++d1[i];
        if (i + d1[i] - 1 > r)
            l = i - d1[i] + 1, r = i + d1[i] - 1;
    }
    for (int i = 1, l = 0, r = -1; i < n; ++i) {
        d2[i] = i > r ? 0 : min(d2[l + r - i + 1], r - i + 1);
        while (i - d2[i] > 0 && i + d2[i] < n && s[i - d2[i] - 1] == s[i + d2[i]])
            ++d2[i];
        if (i + d2[i] - 1 > r)
            l = i - d2[i], r = i + d2[i] - 1;
    }
    ```

    ## 数据结构

    ### ST 表

    ```cpp
    class SparseTable {
    private:
        int lg[N], num[N][LG];

    public:
        void init() {
            lg[0] = -1;
            for (int i = 1; i <= n; ++i)
                lg[i] = lg[i >> 1] + 1, num[i][0] = a[i];
            for (int i = 1; i <= lg[n]; ++i)
                for (int j = 1; j <= n - (1 << i) + 1; ++j)
                    num[j][i] = max(num[j][i - 1], num[j + (1 << (i - 1))][i - 1]);
        }
        int query(int l, int r) {
            int g = lg[r - l + 1];
            return max(num[l][g], num[r - (1 << g) + 1][g]);
        }
    };
    ```

    ### 01Trie

    ```cpp
    class Trie {
    private:
        int tot, son[2][DIG * N];

    public:
        void insert(int x) {
            for (int i = DIG, j = 0; i >= 0; j = son[(x >> i) & 1][j], --i)
                if (!son[(x >> i) & 1][j])
                    son[(x >> i) & 1][j] = ++tot;
        }
        int query(int x) {
            int ans = 0;
            for (int i = DIG, j = 0; i >= 0; --i)
                if (son[!((x >> i) & 1)][j])
                    j = son[!((x >> i) & 1)][j], ans |= 1 << i;
                else
                    j = son[(x >> i) & 1][j];
            return ans;
        }
    } trie;
    ```

    ### LCT

    #### 维护链

    ```cpp
    class LinkCutTree {
    private:
        int fa[N], num[N], sum[N], son[2][N], *ls = son[0], *rs = son[1];
        bool tag[N];
        void addTag(int x) { tag[x] ^= 1, swap(ls[x], rs[x]); }
        void pushUp(int x) { sum[x] = num[x] + sum[ls[x]] + sum[rs[x]]; }
        void pushDown(int x) {
            if (tag[x])
                tag[x] = 0, addTag(ls[x]), addTag(rs[x]);
        }
        bool pos(int x) { return x == rs[fa[x]]; }
        bool isRoot(int x) { return x != ls[fa[x]] && x != rs[fa[x]]; }
        void push(int x) {
            if (!isRoot(x))
                push(fa[x]);
            pushDown(x);
        }
        void rotate(int x) {
            int y = fa[x], z = fa[y];
            bool p = pos(x);
            if (!isRoot(y))
                son[pos(y)][z] = x;
            son[p][y] = son[!p][x], son[!p][x] = y;
            fa[x] = z, fa[y] = x, fa[son[p][y]] = y;
            pushUp(y), pushUp(x);
        }
        void splay(int x) {
            push(x);
            for (int y = fa[x]; !isRoot(x); rotate(x), y = fa[x])
                if (!isRoot(y))
                    rotate((pos(x) == pos(y)) ? y : x);
        }
        void access(int x) {
            for (int y = 0; x; y = x, x = fa[x])
                splay(x), rs[x] = y, pushUp(x);
        }
        void makeRoot(int x) { access(x), splay(x), addTag(x); }
        int findRoot(int x) {
            access(x), splay(x);
            while (ls[x])
                x = ls[x];
            splay(x);
            return x;
        }

    public:
        void link(int x, int y) {
            makeRoot(x);
            if (findRoot(y) != x)
                fa[x] = y;
        }
        void cut(int x, int y) {
            makeRoot(x);
            if (findRoot(y) == x && fa[y] == x && ls[y] == 0)
                fa[y] = rs[x] = 0, pushUp(x);
        }
        void update(int x, int val) { splay(x), num[x] = val, pushUp(x); }
        int query(int x, int y) { return makeRoot(x), access(y), splay(x), sum[x]; }
    };
    ```

    #### 维护子树

    ```cpp
    class LinkCutTree {
    private:
        int fa[N], num[N], sum1[N], sum2[N], son[2][N], *ls = son[0], *rs = son[1];
        bool tag[N];
        void addTag(int x) { tag[x] ^= 1, swap(ls[x], rs[x]); }
        void pushUp(int x) { sum1[x] = num[x] + sum2[x] + sum1[ls[x]] + sum1[rs[x]]; }
        void pushDown(int x) {
            if (tag[x])
                tag[x] = 0, addTag(ls[x]), addTag(rs[x]);
        }
        bool pos(int x) { return x == rs[fa[x]]; }
        bool isRoot(int x) { return x != ls[fa[x]] && x != rs[fa[x]]; }
        void push(int x) {
            if (!isRoot(x))
                push(fa[x]);
            pushDown(x);
        }
        void rotate(int x) {
            int y = fa[x], z = fa[y];
            bool p = pos(x);
            if (!isRoot(y))
                son[pos(y)][z] = x;
            son[p][y] = son[!p][x], son[!p][x] = y;
            fa[x] = z, fa[y] = x, fa[son[p][y]] = y;
            pushUp(y), pushUp(x);
        }
        void splay(int x) {
            push(x);
            for (int y = fa[x]; !isRoot(x); rotate(x), y = fa[x])
                if (!isRoot(y))
                    rotate((pos(x) == pos(y)) ? y : x);
        }
        void access(int x) {
            for (int y = 0; x; y = x, x = fa[x])
                splay(x), sum2[x] += sum1[rs[x]] - sum1[y], rs[x] = y, pushUp(x);
        }
        void makeRoot(int x) { access(x), splay(x), addTag(x); }
        int findRoot(int x) {
            access(x), splay(x);
            while (ls[x])
                x = ls[x];
            splay(x);
            return x;
        }

    public:
        void link(int x, int y) {
            makeRoot(x);
            if (findRoot(y) != x)
                makeRoot(y), fa[x] = y, sum2[y] += sum1[x], pushUp(y);
        }
        void cut(int x, int y) {
            makeRoot(x);
            if (findRoot(y) == x && fa[y] == x && ls[y] == 0)
                fa[y] = rs[x] = 0, pushUp(x);
        }
        void update(int x, int val) { splay(x), num[x] = val, pushUp(x); }
        int query(int x) { return makeRoot(x), splay(x), sum1[x]; }
    };
    ```

    ## 图论

    ### 树剖

    ```cpp
    int n, tot, fa[N], id[N], dep[N], dfn[N], siz[N], son[N], top[N];
    vector<int> edg[N];
    void dfs1(int u, int f) {
        fa[u] = f, siz[u] = 1, dep[u] = dep[f] + 1;
        for (int v : edg[u])
            if (v != f)
                dfs1(v, u), siz[u] += siz[v], siz[v] > siz[son[u]] && (son[u] = v);
    }
    void dfs2(int u, int t) {
        if (!u)
            return;
        top[u] = t, dfn[u] = ++tot, id[tot] = u, dfs2(son[u], t);
        for (int v : edg[u])
            if (v != fa[u] && v != son[u])
                dfs2(v, v);
    }
    int lca(int u, int v) {
        for (; top[u] != top[v]; u = fa[top[u]])
            if (dep[top[u]] < dep[top[v]])
                swap(u, v);
        return dep[u] < dep[v] ? u : v;
    }
    void update(int u, int v) { // or query
        for (; top[u] != top[v]; u = fa[top[u]]) {
            if (dep[top[u]] < dep[top[v]])
                swap(u, v);
            // update l = dfn[top[u]], r = dfn[u]
        }
        if (dep[u] > dep[v])
            swap(u, v);
        // update l = dfn[u], r = dfn[v]
    }
    int jump(int x, int d) { // 跳到深度为 d 的祖先
        while (dep[top[x]] > d)
            x = fa[top[x]];
        return id[dfn[x] - dep[x] + d];
    }
    ```

    如果要用树剖+数据结构维护链，需要注意初始化数据结构时，下标应该是 `id[i]`。

    ### 缩点

    ```cpp
    void tarjan(int x) {
        dfn[x] = low[x] = ++tot, stk.push(x);
        for (int e : edge1[x]) {
            if (!dfn[e])
                tarjan(e), low[x] = min(low[x], low[e]);
            else if (!bel[e])
                low[x] = min(low[x], dfn[e]);
        }
        if (dfn[x] == low[x])
            for (++scc; !stk.empty() && dfn[stk.top()] >= dfn[x]; stk.pop())
                bel[stk.top()] = scc;
    }
    int main() {
        for (int i = 1; i <= n; ++i)
            if (!dfn[i])
                tarjan(i);
    }
    ```

    ### 点双连通分量

    ```cpp
    void tarjan(int x, int pre) {
        dfn[x] = low[x] = ++tot, stk.emplace(x);
        if (edge[x].empty())
            return s[++bcc].emplace_back(x), stk.pop();
        for (int e : edge[x])
            if (e != pre) {
                if (!dfn[e]) {
                    tarjan(e, x), low[x] = min(low[x], low[e]);
                    if (low[e] >= dfn[x])
                        for (s[++bcc].emplace_back(x); !stk.empty() && dfn[stk.top()] >= dfn[e]; stk.pop())
                            s[bcc].emplace_back(stk.top());
                } else
                    low[x] = min(low[x], dfn[e]);
            }
    }
    ```

    ### 边双连通分量

    ```cpp
    void tanjar(int x, int f) {
        dfn[x] = low[x] = ++tot, stk.push(x);
        for (Edge e : edge[x])
            if (e.second != f) {
                if (!dfn[e.first])
                    tanjar(e.first, e.second), low[x] = min(low[x], low[e.first]);
                else
                    low[x] = min(low[x], dfn[e.first]);
            }
        if (dfn[x] == low[x])
            for (++bcc; !stk.empty() && dfn[stk.top()] >= dfn[x]; stk.pop())
                s[bcc].emplace_back(stk.top());
    }
    ```

    ### 欧拉路径/欧拉回路

    #### 判定

    - **有向图欧拉路径**：图中恰好存在 $1$ 个点出度比入度多 $1$（这个点即为起点 $S$），$1$ 个点入度比出度多 $1$（这个点即为终点 $T$），其余节点出度=入度。
    - **有向图欧拉回路**：所有点的入度=出度（起点 $S$ 和终点 $T$ 可以为任意点）。
    - **无向图欧拉路径**：图中恰好存在 $2$ 个点的度数是奇数，其余节点的度数为偶数，这两个度数为奇数的点即为欧拉路径的起点 $S$ 和终点 $T$。
    - **无向图欧拉回路**：所有点的度数都是偶数（起点 $S$ 和终点 $T$ 可以为任意点）。

    #### 有向图欧拉路径

    ```cpp
    // 倒序输出点的编号
    void dfs(int u) {
        while (cur[u] < edg[u].size())
            dfs(edg[u][cur[u]++]);
        stk.push_back(u);
    }

    // 倒序输出边的编号（栈顶元素不取）
    void dfs(pii u) {
        while (cur[u.first] < edg[u.first].size())
            dfs(edg[u.first][cur[u.first]++]);
        stk.push_back(u.second);
    }
    ```

    #### 无向图欧拉路径

    ```cpp
    // 倒序输出点的编号
    void dfs(int u) {
        while (cur[u] < edg[u].size()) {
            while (cur[u] < edg[u].size() && vis[edg[u][cur[u]].second])
                ++cur[u];
            if (cur[u] == edg[u].size())
                break;
            vis[edg[u][cur[u]].second] = true, dfs(edg[u][cur[u]++].first);
        }
        stk.push_back(u);
    }

    // 倒序输出边的编号（栈顶元素不取）
    void dfs(pii u) {
        while (cur[u.first] < edg[u.first].size()) {
            while (cur[u.first] < edg[u.first].size() && vis[edg[u.first][cur[u.first]].second])
                ++cur[u.first];
            if (cur[u.first] == edg[u.first].size())
                break;
            vis[edg[u.first][cur[u.first]].second] = true, dfs(edg[u.first][cur[u.first]++]);
        }
        stk.push_back(u.second);
    }
    ```

    ### 最大流

    ```cpp
    namespace Graph {
    int s, t, tot = 1;
    int cur[N], dep[N], head[N];
    int w[M * 2], to[M * 2], nxt[M * 2];
    bool bfs() {
        memset(dep, 0, sizeof(dep)), memcpy(cur, head, sizeof(cur));
        queue<int> q;
        for (dep[s] = 1, q.push(s); !q.empty(); q.pop())
            for (int e = head[q.front()]; e; e = nxt[e])
                if (w[e] && !dep[to[e]]) {
                    dep[to[e]] = dep[q.front()] + 1, q.push(to[e]);
                    if (to[e] == t)
                        return true;
                }
        return false;
    }
    ll dfs(int x, ll flow) {
        if (x == t)
            return flow;
        ll now = flow;
        for (int &e = cur[x]; e; e = nxt[e])
            if (w[e] && dep[to[e]] == dep[x] + 1) {
                ll f = dfs(to[e], min(now, 1ll * w[e]));
                w[e] -= f, w[e ^ 1] += f, now -= f;
                if (!now)
                    break;
            }
        return flow - now;
    }
    void addEdge(int u, int v, int x) {
        ++tot, w[tot] = x, to[tot] = v, nxt[tot] = head[u], head[u] = tot;
        ++tot, w[tot] = 0, to[tot] = u, nxt[tot] = head[v], head[v] = tot;
    }
    ll solve() {
        ll ans = 0;
        while (bfs())
            ans += dfs(s, oo);
        return ans;
    }
    } // namespace Graph
    ```

    ### 费用流

    ```cpp
    namespace Graph {
    int s, t, cst, tot = 1;
    int dis[N], cur[N], head[N];
    int c[M * 2], w[M * 2], to[M * 2], nxt[M * 2];
    bool vis[N];
    queue<int> q;
    bool spfa() {
        memcpy(cur, head, sizeof(cur));
        for (int i = 1; i <= n; ++i)
            dis[i] = oo;
        for (dis[s] = 0, q.emplace(s); !q.empty(); vis[q.front()] = false, q.pop())
            for (int e = head[q.front()]; e; e = nxt[e])
                if (w[e] && dis[to[e]] > dis[q.front()] + c[e]) {
                    dis[to[e]] = dis[q.front()] + c[e];
                    if (!vis[to[e]])
                        q.emplace(to[e]), vis[to[e]] = true;
                }
        return dis[t] < oo;
    }
    int dfs(int x, int flow) {
        if (x == t)
            return flow;
        vis[x] = true;
        int now = flow;
        for (int &e = cur[x]; e; e = nxt[e])
            if (w[e] && !vis[to[e]] && dis[to[e]] == dis[x] + c[e]) {
                int f = dfs(to[e], min(w[e], now));
                w[e] -= f, w[e ^ 1] += f, now -= f, cst += f * c[e];
                if (!now)
                    break;
            }
        return vis[x] = false, flow - now;
    }
    void addEdge(int u, int v, int x, int y) {
        ++tot, c[tot] = y, w[tot] = x, to[tot] = v, nxt[tot] = head[u], head[u] = tot;
        ++tot, c[tot] = -y, w[tot] = 0, to[tot] = u, nxt[tot] = head[v], head[v] = tot;
    }
    pii solve() {
        cst = 0;
        int ans = 0;
        while (spfa())
            ans += dfs(s, oo);
        return pii(ans, cst);
    }
    } // namespace Graph
    ```

    ## 数学

    ### 线性筛

    ```cpp
    void sieve() {
        phi[1] = 1;
        for (int i = 2; i <= N; ++i) {
            if (!notPrm[i])
                prm[++cnt] = i, phi[i] = i - 1;
            for (int j = 1; j <= cnt && i * prm[j] <= N; ++j) {
                notPrm[i * prm[j]] = true, phi[i * prm[j]] = phi[i] * phi[prm[j]];
                if (i % prm[j] == 0) {
                    phi[i * prm[j]] = phi[i] * prm[j];
                    break;
                }
            }
        }
    }
    ```

    ### $O(n^2)$ gcd

    ```cpp
    for (int i = 1; i <= N - 5; ++i)
        gcd[i][0] = gcd[0][i] = i;
    for (int i = 1; i <= N - 5; ++i)
        for (int j = i; j <= N - 5; ++j)
            gcd[i][j] = gcd[j][i] = gcd[i][j % i];
    ```

    ### exgcd

    ```cpp
    void exgcd(int a, int b, int &x, int &y) {
        if (!b)
            return x = 1, y = 0, void();
        exgcd(b, a % b, y, x), y = dec(y, mul(x, a / b));
    }
    int inv(int a, int mod) {
        int x, y;
        return exgcd(a, mod, x, y), x;
    }
    ```

    ### 莫比乌斯反演

    $$
    \mu * 1 = \epsilon
    $$

    $$
    \varphi * 1 = Id
    $$

    $$
    \mu * Id = \varphi
    $$

    ### 二项式反演

    $$
    f(x) = \sum_{i = 0}^x (-1)^i \binom{x}{i} g(i) \iff g(x) = \sum_{i = 0}^x (-1)^i \binom{x}{i} f(i) \\
    $$

    $$
    f(x) = \sum_{i = 0}^x \binom{x}{i} g(i) \iff g(x) = \sum_{i = 0}^x (-1)^{x - i} \binom{x}{i} f(i) \\
    $$

    $$
    f(x) = \sum_{i = x}^n (-1)^i \binom{i}{x} g(i) \iff g(x) = \sum_{i = x}^n (-1)^i \binom{i}{x} f(i) \\
    $$

    $$
    f(x) = \sum_{i = x}^n \binom{i}{x} g(i) \iff g(x) = \sum_{i = x}^n (-1)^{i - x} \binom{i}{x} f(i) \\
    $$

    ## 多项式

    ### FFT

    ```cpp
    class Poly : public vector<int> {
    private:
        typedef complex<double> cp;
        static const int LG = 21, N = 1 << LG;
        static inline const double PI = acos(-1);
        struct Init {
        public:
            int rev[N];
            cp w[N];
            Init() {
                for (int i = (N >> 1); i < N; ++i)
                    w[i] = cp(cos(PI * 2 * (i - (N >> 1)) / N), sin(PI * 2 * (i - (N >> 1)) / N));
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
            }
        };
        static inline const Init init;
        static int len(int n) { return 32 - __builtin_clz(n - 1); }
        static void fft(vector<cp> &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (init.rev[i] >> (LG - lg)))
                    std::swap(a[i], a[init.rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        cp tmp1 = a[j], tmp2 = init.w[l + j - i] * a[j + l];
                        a[j] = tmp1 + tmp2, a[j + l] = tmp1 - tmp2;
                    }
            if (inv) {
                reverse(a.begin() + 1, a.end());
                cp v = 1.0 / n;
                for (int i = 0; i < n; ++i)
                    a[i] *= v;
            }
        }

    public:
        Poly(int x = 0) { assign(x, 0); }
        Poly(const vector<int> &x) { assign(x.begin(), x.end()); }
        friend istream &operator>>(istream &in, Poly &a) {
            for (int &i : a)
                in >> i;
            return in;
        }
        friend ostream &operator<<(ostream &out, const Poly &a) {
            for (int i : a)
                out << i << ' ';
            return out;
        }

        Poly &operator*=(Poly a) {
            int n = size(), m = a.size(), lg = len(n + m - 1);
            vector<cp> x(1 << lg, 0), y(1 << lg, 0);
            for (int i = 0; i < n; ++i)
                x[i] = at(i);
            for (int i = 0; i < m; ++i)
                y[i] = a[i];
            fft(x, lg, false), fft(y, lg, false);
            for (int i = 0; i < (1 << lg); ++i)
                x[i] *= y[i];
            fft(x, lg, true);
            resize(n + m - 1);
            for (int i = 0; i < size(); ++i)
                at(i) = (int)round(x[i].real());
            return *this;
        }
        friend Poly operator*(Poly a, const Poly &b) { return a *= b; }
    };
    ```

    ### NTT

    ```cpp
    class Poly : public vector<int> {
    private:
        static const int LG = 21, N = 1 << LG, G = 3, MOD = 998244353;
        struct Init {
        public:
            int w[N], rev[N];
            Init() {
                int u = power(G, (MOD - 1) >> LG);
                w[N >> 1] = 1;
                for (int i = (N >> 1) + 1; i < N; ++i)
                    w[i] = mul(w[i - 1], u);
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
            }
        };
        static inline const Init init;
        static int inc(int a, int b) { return a + b >= MOD ? a + b - MOD : a + b; }
        static int dec(int a, int b) { return a < b ? a - b + MOD : a - b; }
        static int mul(int a, int b) { return 1ll * a * b % MOD; }
        static int power(int a, int b) {
            int ans = 1;
            for (; b; b >>= 1, a = mul(a, a))
                if (b & 1)
                    ans = mul(ans, a);
            return ans;
        }
        static int len(int n) { return 32 - __builtin_clz(n - 1); }
        static void ntt(Poly &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (init.rev[i] >> (LG - lg)))
                    std::swap(a[i], a[init.rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        int tmp1 = a[j], tmp2 = mul(init.w[l + j - i], a[j + l]);
                        a[j] = inc(tmp1, tmp2), a[j + l] = dec(tmp1, tmp2);
                    }
            if (inv) {
                reverse(a.begin() + 1, a.end());
                for (int i = 0, v = power(n, MOD - 2); i < n; ++i)
                    a[i] = mul(a[i], v);
            }
        }

    public:
        Poly(int x = 0) { assign(x, 0); }
        Poly(const vector<int> &x) { assign(x.begin(), x.end()); }
        friend istream &operator>>(istream &in, Poly &a) {
            for (int &i : a)
                in >> i;
            return in;
        }
        friend ostream &operator<<(ostream &out, const Poly &a) {
            for (int i : a)
                out << i << ' ';
            return out;
        }

        Poly &operator*=(Poly a) {
            int n = size(), m = a.size(), lg = len(n + m - 1);
            resize(1 << lg, 0), a.resize(1 << lg, 0);
            ntt(*this, lg, false), ntt(a, lg, false);
            for (int i = 0; i < (1 << lg); ++i)
                at(i) = mul(at(i), a[i]);
            ntt(*this, lg, true);
            resize(n + m - 1);
            return *this;
        }
        friend Poly operator*(Poly a, const Poly &b) { return a *= b; }
    };
    ```

    ### 多模 NTT

    ```cpp
    typedef long long ll;
    typedef __int128 lll;
    const int N = 3;
    constexpr int MOD[N] = {998244353, 1004535809, 167772161};
    lll MOD0 = 1;
    template <int MOD>
    class Poly : public vector<int> {
    private:
        static const int LG = 21, N = 1 << LG, G = 3;
        struct Init {
        public:
            int w[N], rev[N];
            Init() {
                int u = power(G, (MOD - 1) >> LG);
                w[N >> 1] = 1;
                for (int i = (N >> 1) + 1; i < N; ++i)
                    w[i] = mul(w[i - 1], u);
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
            }
        };
        static inline const Init init;
        static int inc(int a, int b) { return a + b >= MOD ? a + b - MOD : a + b; }
        static int dec(int a, int b) { return a < b ? a - b + MOD : a - b; }
        static int mul(int a, int b) { return 1ll * a * b % MOD; }
        static int power(int a, int b) {
            int ans = 1;
            for (; b; b >>= 1, a = mul(a, a))
                if (b & 1)
                    ans = mul(ans, a);
            return ans;
        }
        static int len(int n) { return 32 - __builtin_clz(n - 1); }
        static void ntt(Poly &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (init.rev[i] >> (LG - lg)))
                    std::swap(a[i], a[init.rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        int tmp1 = a[j], tmp2 = mul(init.w[l + j - i], a[j + l]);
                        a[j] = inc(tmp1, tmp2), a[j + l] = dec(tmp1, tmp2);
                    }
            if (inv) {
                reverse(a.begin() + 1, a.end());
                for (int i = 0, v = power(n, MOD - 2); i < n; ++i)
                    a[i] = mul(a[i], v);
            }
        }

    public:
        Poly(int x = 0) { assign(x, 0); }
        Poly(const vector<int> &x) { assign(x.begin(), x.end()); }
        friend istream &operator>>(istream &in, Poly &a) {
            for (int &i : a)
                in >> i;
            return in;
        }
        friend ostream &operator<<(ostream &out, const Poly &a) {
            for (int i : a)
                out << i << ' ';
            return out;
        }

        Poly &operator*=(Poly a) {
            int n = size(), m = a.size(), lg = len(n + m - 1);
            resize(1 << lg, 0), a.resize(1 << lg, 0);
            ntt(*this, lg, false), ntt(a, lg, false);
            for (int i = 0; i < (1 << lg); ++i)
                at(i) = mul(at(i), a[i]);
            ntt(*this, lg, true);
            resize(n + m - 1);
            return *this;
        }
        friend Poly operator*(Poly a, const Poly &b) { return a *= b; }
        void getMod() {
            cout << MOD << "\n";
        }
    };
    void exgcd(int a, int b, int &x, int &y) {
        if (!b)
            return x = 1, y = 0, void();
        exgcd(b, a % b, y, x), y -= x * (a / b);
    }
    int inv(int a, int mod) {
        int x, y;
        return exgcd(a, mod, x, y), (x % mod + mod) % mod;
    }
    template <int n>
    void cal(const vector<int> &a, const vector<int> &b, vector<lll> &ans) {
        Poly<MOD[n - 1]> x(a), y(b);
        x *= y;
        if constexpr (n == N)
            ans.assign(x.size(), 0);
        lll c = (lll)MOD0 / MOD[n - 1] * inv(MOD0 / MOD[n - 1] % MOD[n - 1], MOD[n - 1]);
        for (int i = 0; i < x.size(); ++i)
            (ans[i] += c * x[i]) %= MOD0;
        if constexpr (n > 1)
            cal<n - 1>(a, b, ans);
    }
    int main() {
        for (int i = 0; i < N; ++i)
            MOD0 *= MOD[i];
    }
    ```

    ### 拆系数 FFT

    长度为 $10^5$ 可用 `double`，$10^6$ 必须用 `long double`。

    ```cpp
    int inc(int a, int b) { return a + b >= p ? a + b - p : a + b; }
    int mul(int a, int b) { return 1ll * a * b % p; }
    class Poly : public vector<int> {
    private:
        typedef long long ll;
        typedef long double db;
        typedef complex<db> cp;
        static const int LG = 21, N = 1 << LG, M = 31622;
        static inline const db PI = acosl(-1);
        struct Init {
        public:
            int rev[N];
            cp w[N];
            Init() {
                for (int i = (N >> 1); i < N; ++i)
                    w[i] = cp(cosl(PI * 2 * (i - (N >> 1)) / N), sinl(PI * 2 * (i - (N >> 1)) / N));
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
            }
        };
        static inline const Init init;
        static int len(int n) { return 32 - __builtin_clz(n - 1); }
        static void fft(vector<cp> &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (init.rev[i] >> (LG - lg)))
                    std::swap(a[i], a[init.rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        cp tmp1 = a[j], tmp2 = init.w[l + j - i] * a[j + l];
                        a[j] = tmp1 + tmp2, a[j + l] = tmp1 - tmp2;
                    }
            if (inv) {
                reverse(a.begin() + 1, a.end());
                for (int i = 0; i < n; ++i)
                    a[i] /= (db)n;
            }
        }
        static int get(db x) { return (ll)round(x) % p; }

    public:
        Poly(int x = 0) { assign(x, 0); }
        Poly(const vector<int> &x) { assign(x.begin(), x.end()); }
        friend istream &operator>>(istream &in, Poly &a) {
            for (int &i : a)
                in >> i;
            return in;
        }
        friend ostream &operator<<(ostream &out, const Poly &a) {
            for (int i : a)
                out << i << ' ';
            return out;
        }

        Poly &operator*=(Poly a) {
            int n = size(), m = a.size(), lg = len(n + m - 1);
            vector<cp> x(1 << lg, 0), y(1 << lg, 0), z(1 << lg, 0);
            for (int i = 0; i < n; ++i)
                x[i] = at(i) / M, y[i] = at(i) % M;
            for (int i = 0; i < m; ++i)
                z[i] = cp(a[i] / M, a[i] % M);
            fft(x, lg, false), fft(y, lg, false), fft(z, lg, false);
            for (int i = 0; i < (1 << lg); ++i)
                x[i] *= z[i], y[i] *= z[i];
            fft(x, lg, true), fft(y, lg, true);
            resize(n + m - 1);
            for (int i = 0; i < size(); ++i)
                at(i) = inc(inc(mul(mul(M, M), get(x[i].real())), mul(M, get(x[i].imag()))), inc(mul(M, get(y[i].real())), get(y[i].imag())));
            return *this;
        }
        friend Poly operator*(Poly a, const Poly &b) { return a *= b; }
    };
    ```

    ### 多项式全家桶

    ```cpp
    class Poly : public vector<int> {
    private:
        static const int LG = 18, N = 1 << LG, G = 3, MOD = 998244353;
        struct Init {
        public:
            int w[N], rev[N], fac[N], iFac[N];
            Init() {
                int u = power(G, (MOD - 1) >> LG);
                w[N >> 1] = 1;
                for (int i = (N >> 1) + 1; i < N; ++i)
                    w[i] = mul(w[i - 1], u);
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
                fac[0] = 1;
                for (int i = 1; i < N; ++i)
                    fac[i] = mul(fac[i - 1], i);
                iFac[N - 1] = inv(fac[N - 1]);
                for (int i = N - 1; i; --i)
                    iFac[i - 1] = mul(iFac[i], i);
            }
        };
        static inline const Init init;
        static int inc(int a, int b) { return a + b >= MOD ? a + b - MOD : a + b; }
        static int dec(int a, int b) { return a < b ? a - b + MOD : a - b; }
        static int mul(int a, int b) { return 1ll * a * b % MOD; }
        static int power(int a, int b) {
            int ans = 1;
            for (; b; b >>= 1, a = mul(a, a))
                if (b & 1)
                    ans = mul(ans, a);
            return ans;
        }
        static int inv(int a) { return power(a, MOD - 2); }
        static int len(int n) { return 32 - __builtin_clz(n - 1); }
        static void ntt(Poly &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (init.rev[i] >> (LG - lg)))
                    std::swap(a[i], a[init.rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        int tmp1 = a[j], tmp2 = mul(init.w[l + j - i], a[j + l]);
                        a[j] = inc(tmp1, tmp2), a[j + l] = dec(tmp1, tmp2);
                    }
            if (inv) {
                reverse(a.begin() + 1, a.end());
                for (int i = 0, v = Poly::inv(n); i < n; ++i)
                    a[i] = mul(a[i], v);
            }
        }

    public:
        Poly(int x = 0, int l = 1) { assign(l, x); }
        Poly(const initializer_list<int> &x) { assign(x); }
        template <class iterator>
        Poly(const iterator &begin, const iterator &end) { assign(begin, end); }
        friend istream &operator>>(istream &in, Poly &a) {
            for (int &i : a)
                in >> i;
            return in;
        }
        friend ostream &operator<<(ostream &out, const Poly &a) {
            for (int i : a)
                out << i << ' ';
            return out;
        }

        Poly &operator<<=(int a) {
            resize(size() + a);
            for (int i = (int)size() - 1; i >= a; --i)
                at(i) = at(i - a);
            for (int i = 0; i < a; ++i)
                at(i) = 0;
            return *this;
        }
        friend Poly operator<<(Poly a, int b) { return a <<= b; }

        Poly &operator>>=(int a) {
            for (int i = a; i < (int)size(); ++i)
                at(i - a) = at(i);
            resize(size() - a);
            return *this;
        }
        friend Poly operator>>(Poly a, int b) { return a >>= b; }

        Poly &operator%=(const int &a) { return resize(a, 0), *this; }
        friend Poly operator%(const Poly &a, const int &b) {
            Poly ans(a.begin(), min(a.end(), a.begin() + b));
            ans.resize(b, 0);
            return ans;
        }

        Poly &operator+=(const Poly &a) {
            if (a.size() > size())
                resize(a.size(), 0);
            for (int i = 0; i < (int)size(); ++i)
                at(i) = inc(at(i), a[i]);
            return *this;
        }
        friend Poly operator+(Poly a, const Poly &b) { return a += b; }

        Poly &operator-=(const Poly &a) {
            if (a.size() > size())
                resize(a.size(), 0);
            for (int i = 0; i < (int)size(); ++i)
                at(i) = dec(at(i), a[i]);
            return *this;
        }
        friend Poly operator-(Poly a, const Poly &b) { return a -= b; }

        Poly &operator*=(const int &a) {
            for (int &i : *this)
                i = mul(i, a);
            return *this;
        }
        friend Poly operator*(Poly a, const int &b) { return a *= b; }

        Poly &operator*=(Poly a) {
            int n = size(), m = a.size(), lg = len(n + m - 1);
            resize(1 << lg, 0), a.resize(1 << lg, 0);
            ntt(*this, lg, false), ntt(a, lg, false);
            for (int i = 0; i < (1 << lg); ++i)
                at(i) = mul(at(i), a[i]);
            ntt(*this, lg, true);
            resize(n + m - 1);
            return *this;
        }
        friend Poly operator*(Poly a, const Poly &b) { return a *= b; }

        static Poly diff(Poly a) {
            for (int i = 0; i < (int)a.size() - 1; ++i)
                a[i] = mul(i + 1, a[i + 1]);
            a.pop_back();
            return a;
        }
        static Poly inte(Poly a) {
            a.push_back(0);
            for (int i = (int)a.size() - 1; i; --i)
                a[i] = mul(a[i - 1], mul(init.iFac[i], init.fac[i - 1]));
            a[0] = 0;
            return a;
        }
        static Poly inv(const Poly &a, int n) {
            Poly ans(inv(a[0]));
            for (int lg = 2, len = 1; len < n; ++lg, len <<= 1) {
                Poly tmp = a % min(len << 1, (int)a.size());
                ans %= (len << 2), tmp %= (len << 2);
                ntt(ans, lg, false), ntt(tmp, lg, false);
                for (int i = 0; i < (len << 2); ++i)
                    ans[i] = mul(ans[i], dec(2, mul(ans[i], tmp[i])));
                ntt(ans, lg, true);
                ans %= (len << 1);
            }
            return ans % n;
        }
        static Poly ln(const Poly &a, int n) { return inte(diff(a) * inv(a, n - 1) % (n - 1)); }
        static Poly exp(const Poly &a, int n) {
            Poly ans({1});
            for (int len = 1; len < n; len <<= 1)
                ans = ans * (1 + a % (len << 1) - ln(ans, len << 1)) % (len << 1);
            return ans % n;
        }
        static Poly pow(Poly a, const string &b, int n) {
            int k1 = 0, k2 = 0, i = 0, c;
            for (char c : b)
                k1 = (10ll * k1 + c - '0') % MOD, k2 = (10ll * k2 + c - '0') % (MOD - 1);
            while (i < (int)a.size() && !a[i])
                ++i;
            if (i == (int)a.size() || (i && (b.length() >= 8 || 1ll * i * k2 >= n)))
                return Poly(0, n);
            a >>= i, c = a[0];
            return exp(ln(a * inv(c), n) * k1, n - i * k2) * power(c, k2) << i * k2;
        }
    };
    ```

    ## 杂项

    ### 快速乘

    ```cpp
    typedef long long ll;
    typedef unsigned long long ull;
    typedef long double ld;
    ll mul(ll a, ll b, ll mod) {
        ull c = (ld)a / mod * b + 0.5L, ans = (ull)a * b - c * mod;
        return ans < mod ? ans : ans + mod;
    }
    ```

    ### 离散化

    ```cpp
    sort(num.begin(), num.end());
    num.erase(unique(num.begin(), num.end()), num.end());
    int find(int x) { return lower_bound(num.begin(), num.end(), x) - num.begin() + 1; }
    ```

    ### 可变参数

    ```cpp
    template <class... Args>
    int inc(int a, Args... args) { return inc(a, inc(args...)); }
    ```

    ### bash

    #### 判断上条返回值

    ```bash
    if [ $? -ne 0 ]; then
    fi

    if [ $? -eq 0 ]; then
    fi
    ```

    #### while 循环

    ```bash
    while true; do
    done
    ```

    #### for 循环

    ```bash
    for i in $(seq 1 10); do
    done
    ```
    ````