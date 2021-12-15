# leetcode [851. 喧闹和富有](https://leetcode-cn.com/problems/loud-and-rich/)

有一组 n 个人作为实验对象，从 0 到 n - 1 编号，其中每个人都有不同数目的钱，以及不同程度的安静值（quietness）。为了方便起见，我们将编号为 x 的人简称为 "person x "。

给你一个数组 richer ，其中 richer[i] = [ai, bi] 表示 person ai 比 person bi 更有钱。另给你一个整数数组 quiet ，其中 quiet[i] 是 person i 的安静值。richer 中所给出的数据 逻辑自恰（也就是说，在 person x 比 person y 更有钱的同时，不会出现 person y 比 person x 更有钱的情况 ）。

现在，返回一个整数数组 answer 作为答案，其中 answer[x] = y 的前提是，在所有拥有的钱肯定不少于 person x 的人中，person y 是最安静的人（也就是安静值 quiet[y] 最小的人）。

 

示例 1：

输入：richer = [[1,0],[2,1],[3,1],[3,7],[4,3],[5,3],[6,3]], quiet = [3,2,5,4,6,1,7,0]
输出：[5,5,2,5,4,5,6,7]
解释： 
answer[0] = 5，
person 5 比 person 3 有更多的钱，person 3 比 person 1 有更多的钱，person 1 比 person 0 有更多的钱。
唯一较为安静（有较低的安静值 quiet[x]）的人是 person 7，
但是目前还不清楚他是否比 person 0 更有钱。
answer[7] = 7，
在所有拥有的钱肯定不少于 person 7 的人中（这可能包括 person 3，4，5，6 以及 7），
最安静（有较低安静值 quiet[x]）的人是 person 7。
其他的答案也可以用类似的推理来解释。
示例 2：

输入：richer = [], quiet = [0]
输出：[0]

提示：

n == quiet.length
1 <= n <= 500
0 <= quiet[i] < n
quiet 的所有值 互不相同
0 <= richer.length <= n * (n - 1) / 2
0 <= ai, bi < n
ai != bi
richer 中的所有数对 互不相同
对 richer 的观察在逻辑上是一致的

## 个人题解

首先吐槽一下，这个题目写的有点绕，richer里面还加ai,bi，还有person x之类的，给的符号实在太多。。。

先理清一下需要解决的问题：有n个人，richer数组记录了若干组富有关系，其中richer[i]表示richer[i] [0] > richer[i] [1] 即前者比后者富有。quiet数组记录这n个人的吵闹程度。需要返回长度为n的res数组，表示比i富有的人中（包括i）最安静的人。

### 思路1

拿到这个题，最直接的思路就是利用richer关系建立g[n] [n]的领接图，然后利用dfs搜索所有比你富有的人，返回quiet最小的index即可。PS：这里可以加个记忆化的优化，如果i已经被搜索过了，直接返回res[i]。

```c++
class Solution {
public:
    vector<int> ans;
    vector<vector<int>> g;
    vector<int> q;

    vector<int> loudAndRich(vector<vector<int>>& richer, vector<int>& quiet) {
        int n = quiet.size();

        q = quiet;
        ans = vector<int>(n, -1);
        g = vector<vector<int>>(n);

        for (auto e: richer) {
            // e[0] richer than e[1]
            g[e[1]].push_back(e[0]);
        }

        for (int i = 0; i < n; i++) {
            dfs(i);
        }

        return ans;
    }

    int dfs(int i) {
        if (ans[i] != -1) return ans[i];
        else ans[i] = i;
        for (auto n: g[i]) {
            if (q[dfs(n)] < q[ans[i]]) ans[i] = dfs(n);
        }
        return ans[i];
    };
};
```

### 思路2

又深入思考了一下，我们一开始要建立一个领接图，对于那些“最富”的人，他们的res为他们本身，而对于那些“次富“的人，他们的res为之前比他穷的人中最安静的那个人，依次类推，所以肯定存在一种方式能按照顺序遍历即可获得所有的res。

这里就用到了拓扑排序：利用richer[i]建立单向领结图，并记录richer[i] [1]的入度（后来的遍历顺序是从富到穷，所以应该由富指向穷，即richer[i] [0]指向richer[i] [1]），将入度为0的节点加入队列，遍历该节点所能到达的节点，更新所能到达节点的res，并将其入度减一，如果入度为零，则将这个节点加入遍历队列。

```c++
class Solution {
public:
    vector<int> loudAndRich(vector<vector<int>>& richer, vector<int>& quiet) {
        int n=quiet.size();
        vector<vector<int>> g(n);
        vector<int> inDep(n,0);
        for (auto bord : richer)
        {
            g[bord[0]].push_back(bord[1]);//单向图
            inDep[bord[1]]++;//入度记录
        }

        queue<int> q;
        vector<int> res(n);
        for (int i=0; i<n; i++) 
        {
            if (inDep[i]==0) q.push(i);
            res[i]=i;
        }

        while (!q.empty())
        {
            int index=q.front();
            q.pop();
            for (int j=0; j<g[index].size(); j++)
            {
                inDep[g[index][j]]--;
                if (quiet[ res[g[index][j]] ] > quiet[ res[index] ]) 
                    res[g[index][j]]=res[index];
                if (inDep[g[index][j]]==0) q.push(g[index][j]);
            }
        }
        return res;
    }
};
```

