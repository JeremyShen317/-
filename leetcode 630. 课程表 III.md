# leetcode [630. 课程表 III](https://leetcode-cn.com/problems/course-schedule-iii/)

这里有 n 门不同的在线课程，按从 1 到 n 编号。给你一个数组 courses ，其中 courses[i] = [durationi, lastDayi] 表示第 i 门课将会 持续 上 durationi 天课，并且必须在不晚于 lastDayi 的时候完成。

你的学期从第 1 天开始。且不能同时修读两门及两门以上的课程。

返回你最多可以修读的课程数目。

 

示例 1：

输入：courses = [[100, 200], [200, 1300], [1000, 1250], [2000, 3200]]
输出：3
解释：
这里一共有 4 门课程，但是你最多可以修 3 门：
首先，修第 1 门课，耗费 100 天，在第 100 天完成，在第 101 天开始下门课。
第二，修第 3 门课，耗费 1000 天，在第 1100 天完成，在第 1101 天开始下门课程。
第三，修第 2 门课，耗时 200 天，在第 1300 天完成。
第 4 门课现在不能修，因为将会在第 3300 天完成它，这已经超出了关闭日期。
示例 2：

输入：courses = [[1,2]]
输出：1
示例 3：

输入：courses = [[3,2],[4,3]]
输出：0


提示:

1 <= courses.length <= 104
1 <= durationi, lastDayi <= 104

## 个人题解

拿到这道题是典型的贪心算法的题目，首先得按照结束时间进行升序排序，然后对贪心策略进行思考。

对排序后的数组进行遍历：

以day记录当前学习的天数，如果当前这门课程学习那么天数增加duration，只要day+duration<=lastday，那么该课程是可以学习的并学习它。

如果day+duration>lastday，表示该课程无法学习。此时又该怎么办？

直接不学习该课程？显然是不合理的，以示例1为例：学习第一门课程day=100,学习第二门课程day=1100,此时无法学习第三门课程，如果第四门课程的截止时间为3000，那么在学习完第二门课程后，无法学习后面两门，那么最终只能学习两门；相反如果我们学习第一门和第三门课程，day=300，那么第四门课程是可以学习的，day=2300。

所以在day+duration>lastday时，我们也许应该从已经学习的课程中找一门出来与当前要学习的课程进行替换，使我们能学习这门课程，那么准则应该是什么？其实不难看出，如果我们学相同的课程数，day最小的话，之后就能学习更多的课程，因此我们应该选出在已经学习的课程中，花费duration最多的课程进行替换。如果花费的duration最多的也没有当前这门课程多，说明我们不应该学习当前这门课程。

综上，贪心策略如下：按照lastDay进行升序排列，如果day+duration<=lastDay，那么就学习这门课程；如果day+duration>lastDay，说明要放弃一门课程，放弃之前到现在的课程中duration最大的那门课程。

```c++
class Solution {
public:
    int scheduleCourse(vector<vector<int>>& courses) {
        sort(courses.begin(),courses.end(),[](const vector<int>& a,const vector<int>& b){ return a[1]<b[1]; });
        priority_queue<int> q;
        int day=0;
        for (auto course : courses)
        {
            if (day+course[0]<=course[1])
            {
                day+=course[0];
                q.push(course[0]);
                continue;
            }
            if (!q.empty() && q.top()>course[0])
            {
                day-=q.top();
                day+=course[0];
                q.pop();
                q.push(course[0]);
            }
        }
        return q.size();
    }
};
```



