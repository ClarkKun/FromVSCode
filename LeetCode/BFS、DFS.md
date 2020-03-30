# BFS与DFS

## B定义
### BFS
广度优先遍历类似于二叉树的层次遍历，使用队列来存储遍历过的值，再从队列中取值向下层遍历，即不断地向外层进行遍历

### DFS
>深度优先搜索算法（Depth First Search，简称DFS）：一种用于遍历或搜索树或图的算法。 沿着树的深度遍历树的节点，尽可能深的搜索树的分支。当节点v的所在边都己被探寻过或者在搜寻时结点不满足条件，搜索将回溯到发现节点v的那条边的起始节点。整个进程反复进行直到所有节点都被访问为止。属于盲目搜索,最糟糕的情况算法时间复杂度为O(!n)。
DFS借助栈来回溯状态

## 题目
### 1.腐烂的橘子
>链接：https://leetcode-cn.com/problems/rotting-oranges/  
在给定的网格中，每个单元格可以有以下三个值之一：  
值 0 代表空单元格；  
值 1 代表新鲜橘子；  
值 2 代表腐烂的橘子。  
每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。  
返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1。

题解：  
1. 使用队列来存储遍历过的坐标，值得注意是的是，队列通常只存储一个整型值，**故将数组中的位置坐标转换为一个整型值就特别巧妙**，使用 i * col+j 的形式，来表示坐标值，当需要使用坐标值时，横坐标可以通过value / col获得，纵坐标可以通过value % col获得  
2. 使用map来存储遍历节点的深度，即第几次被遍历到的，key为坐标，value为层级深度  
3. 在遍历上下左右坐标时，可以先设定两个数组，分别对应着上下左右时横纵坐标的偏移量
```java
class Solution {
    //上下左右
    //上：i-1,j;下：i+1,j;左：i,j-1;右：i,j+1
    int[] dr = new int[]{-1, 1, 0, 0};
    int[] dc = new int[]{0, 0, -1, 1};

    public int orangesRotting(int[][] grid) {
        //用于存储遍历过的坐标 坐标用i*col+j表示
        Queue<Integer> queue = new ArrayDeque<>(); 
        //用于存储查找层级 key:坐标、value：第几层展示
        HashMap<Integer, Integer> depth = new HashMap<>(); 

        int row = grid.length;
        int col = grid[0].length;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (grid[i][j] == 2) {
                    int rot = i * col + j;
                    queue.add(rot);
                    depth.put(rot, 0);
                }
            }
        }

        int ans = 0;
        while (!queue.isEmpty()) {
            int curOrange = queue.remove();
            int curRow = curOrange / col;
            int curCol = curOrange % col;
            for (int k=0;k<4;k++) {
                int i = curRow + dr[k];
                int j = curCol + dc[k];
                //遍历腐烂橘子上下左右的橘子，并加入队列
                if (i >= 0 && i < row && j >= 0 && j < col && grid[i][j] == 1) {
                    grid[i][j] = 2;
                    int rot = i * col + j;
                    queue.add(rot);
                    depth.put(rot, depth.get(curOrange) + 1);
                    ans = depth.get(rot);
                }
            }

        }
        //遍历数组，存在新鲜橘子（值为1），说明没有完全烂掉，返回-1
        for (int[] temp : grid){
            for (int num : temp) {
                if (num == 1) {
                    return -1;
                }
            }
        }

        return ans;
    }
}
```


### 2.岛屿的最大面积
>给定一个包含了一些 0 和 1的非空二维数组 grid , 一个 岛屿 是由四个方向 (水平或垂直) 的 1 (代表土地) 构成的组合。你可以假设二维矩阵的四个边缘都被水包围着。
找到给定的二维数组中最大的岛屿面积。(如果没有岛屿，则返回面积为0。)
链接：https://leetcode-cn.com/problems/max-area-of-island


### 3.水壶问题
>有两个容量分别为 x升 和 y升 的水壶以及无限多的水。请判断能否通过使用这两个水壶，从而可以得到恰好 z升 的水？  
如果可以，最后请用以上水壶中的一或两个来盛放取得的 z升水。  
你允许：  
装满任意一个水壶  
清空任意一个水壶  
从一个水壶向另外一个水壶倒水，直到装满或者倒空    
输入: x = 3, y = 5, z = 4  
输出: True  
链接：https://leetcode-cn.com/problems/water-and-jug-problem

题解：
https://leetcode-cn.com/problems/water-and-jug-problem/solution/tu-jie-bfs-c-jie-zhu-unordered_set-queue-shi-xian-/

```java
import java.util.*;

/**
 * 水量的最终状态由x和y两个参数决定，而x与y也有固定的规律
 * 根据条件：可以装满任意一个水壶；
 * 可以清空任意一个水壶；
 * 可以从一个水壶向另一个水壶倒水，直到倒满或倒空
 * 由此，比如容量为1，2的两个水壶，状态有：
 * (0,0),(1,0)
 * (0,1),(1,1)
 * (1,2),(0,2)
 * 使用BFS来完成这个操作
 * （BFS使用一个Queue和一个Set，Queue用来记录遍历的路径，Set用来消除重复遍历）
 * （DFS使用一个Stack和一个Set，与BFS类似，只是注重纵深，BFS更注重广度）
 */
class Solution {
    public boolean canMeasureWater(int x, int y, int z) {
        if (z == 0) {
            return true;
        }
        if (x + y < z) {
            return false;
        }
        //使用Map.Entry来存储每一个状态
        Queue<Map.Entry<Integer, Integer>> queue = new LinkedList<>();
        Set<Map.Entry<Integer, Integer>> set = new HashSet<>();
        //初始状态为0，0
        AbstractMap.SimpleEntry<Integer, Integer> start = new AbstractMap.SimpleEntry<Integer, Integer>(0, 0);
        ((LinkedList<Map.Entry<Integer, Integer>>) queue).add(start);

        while (!queue.isEmpty()) {
            Map.Entry<Integer, Integer> curEntry = queue.poll();
            int curX = curEntry.getKey();
            int curY = curEntry.getValue();
            if (curX + curY == z || curX == z || curY == z) {
                return true;
            }
            if (curX == 0) {
                addToQueue(queue, set, new AbstractMap.SimpleEntry<>(x, curY));
            }
            if (curY == 0) {
                addToQueue(queue, set, new AbstractMap.SimpleEntry<>(curX, y));
            }
            if (curX < x) {
                addToQueue(queue, set, new AbstractMap.SimpleEntry<>(curX, 0));
            }
            if (curY < y) {
                addToQueue(queue, set, new AbstractMap.SimpleEntry<>(0, curY));
            }
            int offset = Math.min(curX, y - curY);
            addToQueue(queue, set, new AbstractMap.SimpleEntry<>(curX - offset, curY + offset));
            offset = Math.min(curY, x - curX);
            addToQueue(queue, set, new AbstractMap.SimpleEntry<>(curX + offset, curY - offset));
        }
        return false;
    }

    public void addToQueue(Queue<Map.Entry<Integer, Integer>> queue, Set<Map.Entry<Integer, Integer>> set, Map.Entry<Integer, Integer> curEntry) {
        if (!set.contains(curEntry)) {
            set.add(curEntry);
            queue.add(curEntry);
        }
    }
}
```
