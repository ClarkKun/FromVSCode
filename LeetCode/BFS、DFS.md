# BFS
广度优先遍历类似于二叉树的层次遍历，使用队列来存储遍历过的值，再从队列中取值向下层遍历，即不断地向外层进行遍历
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