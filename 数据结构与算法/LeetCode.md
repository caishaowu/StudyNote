[TOC]



## 1、双指针

### 167.Two Sum II - Input array is sorted（easy）

题目描述：在一个有序整型数组中找出两个数，让它们的和为`target`

**Example:**

```js
Input: numbers = [2,7,11,15], target = 9
Output: [1,2]
Explanation: The sum of 2 and 7 is 9. Therefore index1 = 1, index2 = 2.
```

解题思路：创建两个指针，一个指向数组中的最小值，另个一个指向最大值

- 如果两个指针指向元素相加之和 `sum = target`，返回对应下标值 + 1

- 如果 `sum > target`，指向较小值的指针向后移动一步，即`index1++`

- 如果 `sum < target`，指向较大值得指针向前移动一步，即 `index2--`

  

数组中的元素最多遍历一次，时间复杂度为 O(N)。只使用了两个额外变量，空间复杂度为 O(1)。

```java
public int[] twoSum(int[] numbers, int target) {
       
        int index1 = 0;
        int index2 = numbers.length -1;
        while(index1 < index2) {
            int sum = numbers[index1] + numbers[index2];
            if(sum == target){
                return new int[]{index1 + 1,index2 + 1};
                
            }else if(sum > target){
                index2--;
            }else if(sum < target){
                index1++;
            }
        }
        return null;
    }
```

### 633. Sum of Square Numbers（easy）

题目描述：给一个正整数`c`，判断是否有整数 `a` 和 `b` 的平方之和等于 `c`

**Example 1:**

```
Input: 5
Output: True
Explanation: 1 * 1 + 2 * 2 = 5
```

**Example 2:**

```
Input: 3
Output: False
```

解题思路：

```java
public boolean judgeSquareSum(int c) {
        if(c < 0) return false; 
        int a = 0;
        int b = (int)Math.sqrt(c);
        while(a <= b) {
            int sum = a * a + b * b;
            if(sum == c){
                return true;
            }else if (sum < c) {
                a++;
            }else if (sum > c) {
                b--;
            }
        }
        return false;
    }
```

## 2、贪心算法

### 455. Assign Cookies（easy）

题目描述：给孩子分配饼干。数组`int[] g`表示孩子的个数及需要的饼干大小，数组`int[] s`表示你拥有的饼干个数及相应大小，找出能满足孩子需要的饼干个数。

**Example 1:**

```
Input: [1,2,3], [1,1]

Output: 1

Explanation:你有3个孩子，分别需要size为1、2、3大小的饼干，而你只有两个size为1的饼干，故只能满足一个孩子的需要，输出1
```

**Example 2:**

```
Input: [1,2], [1,2,3]

Output: 2
```

解题思路：

```java
 public int findContentChildren(int[] g, int[] s) {
        if(g == null || s == null || g.length == 0 || s.length == 0) return 0;
        Arrays.sort(g);
        Arrays.sort(s);
       // int count = 0;
        int index1 = 0;
        int index2 = 0;
        
        while(index1 < g.length && index2 < s.length) {
            if(g[index1] <= s[index2] ) {
                index1++;
             //   count++;
            }
            index2++;
        }
        return /*count*/ index1;
    }
```

## 3、

### 144.Binary Tree Preorder Traversal（Medium）

题目描述：先序遍历二叉树

**Example:**

```
Input: [1,null,2,3]
 1
    \
     2
    /
   3

Output: [1,2,3]
```

解题思路：

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<Guide> path = new ArrayDeque<>();
        path.addFirst(new Guide(0,root));
        while(!path.isEmpty()){
            Guide current = path.removeFirst(); 
            if(current.node == null) continue;
            if(current.ope == 1) {
                result.add(current.node.val);
            }else{
                //对以下三行代码进行简单的修改，即可实现先序、中序、后序遍历
                path.addFirst(new Guide(0,current.node.right));
                path.addFirst(new Guide(0,current.node.left));
                path.addFirst(new Guide(1,current.node));
            }
        }
        return result;
    }
    
    private class Guide{
        private int ope; //0 visit,1 println
        private TreeNode node;
        Guide(int ope,TreeNode node){
            this.ope =ope;
            this.node = node;
        }
    }
}
```

