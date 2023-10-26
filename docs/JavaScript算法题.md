# 移动零（美团）
原始值：[0, 1, 2, 0, 5, 8, 6, 0, 2, 7]   
结果：[0, 0, 0, 1, 2, 5, 8, 6, 2, 7]

```javascript
// 自己的解法：这个解法有待完善，如下
function sortArray(list) {
    // 边界条件判断
//     if(list.length === 0) {
//         return list;
//     }
    
    const length = list.length;
    // 倒序遍历
    for(let i = length - 1; i > 0; i--) {
        // 判断是否为零数字
        if(list[i] === 0) {
            console.log('123')
            list.splice(i, 1); // 删除0元素
            list.unshift(0); // 在数组前面添加0
            // i++; 这个注意
        }
    }
    return list;
}

sortArray([0, 1, 2, 0, 5, 8, 6, 0, 2, 7, 0, 0])

// 面试官建议的解法：双指针
var moveZeroes = function(nums) {
    let slowIndex = 0;
    for (let fastIndex = 0; fastIndex < nums.length; fastIndex++) {
        if (nums[fastIndex] !== 0) {
            [nums[slowIndex], nums[fastIndex]] = [nums[fastIndex], nums[slowIndex]];
            slowIndex++;
        }
    }
};
```

# 对象扁平化（字节）

原始值：`{ a: { b: { c: 1 } }, d: 2, e: [3, { f: 4, g: [5] }, [6, 7]], h: 8 }`   
要求输出：`{ "a.b.c": 1, d: 2, "e[0]": 3, "e[1].f": 4, "e[1].g[0]": 5, "e[2][0]": 6, "e[2][1]": 7, h: 8 }`

```javascript
function flatten(obj) {
    let res = {};
    function flat(cur, key = '', isArray = false) {
        for (let [k, v] of Object.entries(cur)) {
            if(Array.isArray(v)) {
                let temp = isArray ? key + "[" + k + "]" : key + k;
                flat(v, temp, true);
            } else if(typeof v === 'object') {
                let temp = isArray ? key + "[" + k + "]." : key + k + ".";
                flat(v, temp);
            } else {
                let temp = isArray ? key + "[" + k + "]" : key + k;
                res[temp] = v
            }
        }
    }
    flat(obj);
    return res;
}
```
# 无重复字符的最长子串（好未来）
题目描述：给定一个字符串 s ，请你找出其中不含有重复字符的` 最长连续子字符串 `的长度。
输入: s = "abcabcbb" 
输出: 3  
解释: 因为无重复字符的最长子字符串是 "abc"，所以其长度为 3。
```javascript
// 方法1：
var lengthOfLongestSubstring = function(s) {
    let count = 0, l = 0;
    const map = new Map();
    // 以[l, r]为一个窗口
    for(let r = 0; r < s.length; r++) {
        // 遇到重复字符的时候把左指针移到前面的重复字符的下一位。
        //（相当于把前面的重复字符删除）
        // map.get(s[r]) >= l ，因为有可能有多个重复的值
        if(map.has(s[r]) && map.get(s[r]) >= l) {
            l = map.get(s[r]) + 1;
        }
        // 不重复
        count = Math.max(count, r - l + 1);
        map.set(s[r], r);
    }
    return count;
};


// 方法2：
var lengthOfLongestSubstring = function(s) {
    let count = 0, left = 0;
    const map = new Map();
    // 以[left, right]为一个窗口
    for(let right = 0; right < s.length; right++) {
        // 如果重复
        if(map.has(s[right])) {
            left = Math.max(left, map.get(s[right]) + 1); // 注意这段代码
        }
        // 不重复
        count = Math.max(count, right - left + 1);
        map.set(s[right], right);
    }
    return count;
};
```

# 树
## 列表转成树 && 树转化为列表（面试题）
题目1： 输入如下列表如下，转化成树形结构；
题目2： 将树形结构转化为列表；
如下：
```javascript
[
  { id: 1, title: "child1", parentId: 0 },
  { id: 2, title: "child2", parentId: 0 },
  { id: 3, title: "child1_1", parentId: 1 },
  { id: 4, title: "child1_2", parentId: 1 },
  { id: 5, title: "child2_1", parentId: 2 }
]

// 转化为：
tree = [
  {
    "id": 1,
    "title": "child1",
    "parentId": 0,
    "children": [
      {
        "id": 3,
        "title": "child1_1",
        "parentId": 1
      },
      {
        "id": 4,
        "title": "child1_2",
        "parentId": 1
      }
    ]
  },
  {
    "id": 2,
    "title": "child2",
    "parentId": 0,
    "children": [
      {
        "id": 5,
        "title": "child2_1",
        "parentId": 2
      }
    ]
  }
]
```

##### 题目1：输入上述列表，转化成树形结构；
**使用对象存储数据, 典型的空间换时间**，时间复杂度为O(n)、空间复杂度为O(n)，代码实现：
```javascript
// 列表转化为树
// 方法1：效率最高的一种方式
function listToTree(data) {
    // map来存储数据，res为最终结果
    const [map, res] = [{}, []];
    // 遍历原始数据data，存到map中，id为key，值为数据
    for (let item of data) {
        map[item.id] = item;
    }
    // 再次遍历data
    for (let i of data) {
        // 通过map获取每一项的parent的数据
        let parent = map[i.parentId];
        if(parent) {
            // 判断父节点的children是否存在，如果不存在设置初始值[], 存在直接push
            (parent.children || (parent.children = [])).push(i);
        } else {
            // parent找不到对应值，说明是根节点，直接插入到数组中
            res.push(i);
        }
    }
    return res;
}
```
##### 题目2：上述树转化为列表


广度优先遍历
![Alt](../image/广度优先遍历.png#pic_center)
```javascript
/**
 * 广度优先遍历
 * 队列  先进先出
 */
function wideTraversal(node) {
    let [stack, data] = [node, []];
    while (stack.length > 0) {
        let shift = stack.shift();
        data.push({
            id: shift.id,
            title: shift.title,
            parentId: shift.parentId
        })
        let children = shift.children;
        if(children) {
            for(let i = 0; i < children.length; i++){
                stack.push(children[i])
            }
        }
    }
    return data;
}
```
深度优先遍历
![Alt](../image/深度优先遍历.png#pic_center)
```javascript

/**
 * 树转数组扁平化结构   
 * 深度优先遍历  栈  后进先出
 */
function deepTraversal(node) {
	let [stack, res] = [node,[]];
	while(stack.length !== 0){
		let pop = stack.pop();
		res.push({
			id: pop.id,
			name: pop.name,
			parentId: pop.parentId
		})
		let children = pop.children;
		if(children){
			for(let i = children.length - 1; i >= 0; i--){
				stack.push(children[i]);
			}
		}
	}
	return res;
}
```
备注：另外还有递归方式实现，此处暂时省略,[JS算法之深度优先遍历(DFS)和广度优先遍历(BFS)](https://segmentfault.com/a/1190000018706578)
[JS 之深度优先遍历和广度优先遍历](https://workstudy.top/2020/11/17/JS-%E4%B9%8B%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E5%92%8C%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86/#%E6%A6%82%E5%BF%B5)

## 二叉树前、中、后遍历套路详解
```javascript
// 二叉树的后序遍历
var postorderTraversal = function(root) {
    const res = [];
    function traversal(root) {
        if(root !== null) {
            traversal(root.left);
            traversal(root.right);
            res.push(root.val); // 三者的区别在于res.push(root.val)的位置
        }
    }
    traversal(root);
    return res;
};

// 二叉树的前序遍历
// 二叉树的中序遍历
// 区别在于res.push(root.val)的位置
```

## 二叉树的最大深度
这个题在面试滴滴的时候遇到过，主要是掌握二叉树遍历的套路
- 只要遍历到这个节点既没有左子树，又没有右子树的时候，说明就到底部了，这个时候如果之前记录了深度，就可以比较是否比之前记录的深度大，大就更新深度
- 然后以此类推，一直比较到深度最大的
```javascript
var maxDepth = function(root) {
    if(!root) return root;
    let ret = 1;
    function dfs(root, depth){
        if(!root.left && !root.right) ret = Math.max(ret, depth);
        if(root.left) dfs(root.left, depth+1);
        if(root.right) dfs(root.right, depth+1);
    }
    dfs(root, ret);
    return ret
};

```

## 反转二叉树（陌陌）
给定一棵二叉树的根节点 root，请左右翻转这棵二叉树，并返回其根节点。
![反转二叉树](../image/反转二叉树.png#pic_center)
```javascript
/**
 * @param {TreeNode} root
 * @return {TreeNode}
 */
var mirrorTree = function(root) {
    // 边界判断
    if(!root) {
        return root;
    }
    // 交换
    [root.left, root.right] = [root.right, root.left];
    mirrorTree(root.left);
    mirrorTree(root.right);
    return root;
};
```

## 数组转tree
输入：'[abc[bcd[def]]]'
结果：{"value":"abc","children":{"value":"bcd","children":{"value":"def"}}}
```javascript

// 先进行分割，然后使用reduce遍历实现
function normalize1(str) {
    var arr = str.split(/[\[\]]/g).filter(v => v);
    const res = {};
    arr.reduce((total, cur, index, self) => {
        total.value = cur;
        if(index !== self.length - 1) {
            total.children = {};
            return total.children;
        }
    }, res)
    return res;
}
console.log(normalize1('[abc[bcd[def]]]') )
```


# 第一个不重复字符
题目： 输入一个字符串，找到第一个不重复字符的下标
输入： 'abcabcde'
输出： 6
```javascript

```
# 版本号排序（汽车之家）
题目： 输入一组版本号，输出从大到小的排序

输入： ['2.1.0.1', '0.402.1', '10.2.1', '5.1.2', '1.0.4.5']

输出： ['10.2.1', '5.1.2', '2.1.0.1', '1.0.4.5', '0.402.1']


```javascript
// 思路：利用数组的sort方法，在sort中进行判断实现
function sortVersion(list) {
    return list.sort((a, b) => {
    	// 转化数组
        let arr1 = a.split('.').map(i => i * 1);
        let arr2 = b.split('.').map(i => i * 1);
        
        const maxLength = Math.max(arr1.length, arr2.length);
        // 以数组最长长度为遍历的基准
        for(let i = 0; i < maxLength; i++) {
            if((arr1[i] || 0) > (arr2[i] || 0)) {
                return 1;
            }
            if((arr1[i] || 0) < (arr2[i] || 0)) {
                return -1;
            }
        }
        return 0;
    })
}
```
# 三数之和
题目： 给定一个数组nums，判断 nums 中是否存在三个元素a，b，c，使得 a + b + c = target，找出所有满足条件且不重复的三元组合

输入： nums: [5, 2, 1, 1, 3, 4, 6] ；target:8

输出：[ [1, 1, 6], [1, 2, 5], [1, 3, 4] ]

```javascript
// 用双指针算法
// 注意：下面计算的是target为0的情况，如果算其他值，只需传入target即可
var threeSum = function(nums) {
    // 先将数组从小到大排序
    nums.sort((a, b) => a - b);
    const n = nums.length;
    let ans = [];

    // 循环到n-2，剩下两个留给另两个指针
    for (let i = 0; i < n - 2; i++) {
        // 跳过重复的arr[i]值, 比如[2, 1, 1],跳过第二个1
        if (i > 0 && nums[i] === nums[i - 1]) continue;
        // 如果target为其他值，下面的两行代码可不考虑
        if (nums[i] + nums[i + 1] + nums[i + 2] > 0) break; // 优化一
        if (nums[i] + nums[n - 2] + nums[n - 1] < 0) continue; // 优化二

        // 定义两个指针，初始位置：左指针为第二位，右指针为最后一位
        let j = i + 1, k = n - 1;
        
        while (j < k) {
            let sum = nums[i] + nums[j] + nums[k];
            if (sum > 0) k--;
            else if (sum < 0) j++;
            else {
                ans.push([nums[i], nums[j], nums[k]]);

                // 继续移动指针，还有其他元素满足条件
                j += 1;
                // 因为可能有多个重复的值，所有使用while遍历
                while(j < k && nums[j] === nums[j-1]) { // 跳过重复数字
                    j += 1;
                }

                k -= 1;
                while(k > j && nums[k+1] === nums[k]) { // 跳过重复数字
                    k -= 1;
                }
            }
        }
    }
    return ans;
};
```

# 数组对象根据另外一个数组排序（知乎）
数组对象根据另外一个数组排序
const array1 = [{value: 1}, {value: 2}, {value: 3}, {value: 1}];
const order = [2, 3, 1];
输出：
```javascript
// 输出结果
const b = [
  {
  "value": 2
  },
  {
  "value": 3
  },
  {
  "value": 1
  },
  {
  "value": 1
  }
]
```
思路：遍历order，过程中，根据value和每一项的值比较将array1数据进行分组，并将其存在map中，利用map的有序性进行排序。

# 
```javascript
pending...
```

# 手写LRU算法（蔚来）

```javascript
// 手写LRU算法
class LRUCache {
  constructor(length) {
    this.length = length; // 存储长度
    this.data = new Map(); // 存储数据
  }
  // 存储数据，通过键值对的方式
  set(key, value) {
    const data = this.data;
    if (data.has(key)) {
      data.delete(key)
    }
    data.set(key, value);


    // 如果超出了容量，则需要删除最久的数据
    if (data.size > this.length) {
      const delKey = data.keys().next().value;
      data.delete(delKey);
    }
  }
  // 获取数据
  get(key) {
    const data = this.data;
    // 未找到
    if (!data.has(key)) {
      return null;
    }
    const value = data.get(key); // 获取元素
    data.delete(key); // 删除元素
    data.set(key, value); // 重新插入元素
  }
}
const lruCache = new LRUCache(5);

```
[面试官：请使用JS完成一个LRU缓存？](https://juejin.cn/post/7105654083347808263)


# 合并两个有序数组
思路：while两个数组长度，双指针，最后把剩下的数组进行合并。
```javascript
略
```

# 斐波那契数列
题目： 从第3项开始，当前项等于前两项之和： 1 1 2 3 5 8 13 21 ……，计算第n项的值
输入： 10
输出： 89
```javascript
// 动态规划
// 使用动态规划，将复杂的问题拆分，也就是：`F(N) = F(N - 1) + F(N - 2)`，用数组将已经计算过的值存起来
// **`BigInt`** 是一种内置对象，它提供了一种方法来表示大于 `2^53 - 1` 的整数
function fib(n) {
  // 使用dp数组，将之前计算的结果存起来，防止栈溢出
  if (n < 2) return 1;
  let dp = [1n, 1n];
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
}
```


# 回文的最大长度
在一个字符串中切除一部分，使得剩下的部分是回文，求这个回文的最大长度
思路：只能切除连续的一段，例如‘abbcfa’中切除从第四个到第五个字符，剩下‘abba’是回文，最大长度4。
因为截掉的字符串只有两种情况：
● 是原字符串中间的一部分
● 是原字符串左边或者右边的一部分
我的解法是，先把左右两边对称的部分排除，这一部分一定是回文串的一部分，长度记为res1，然后剩下的没排除的部分就一定是第二种情况了，这时候只要依次遍历每个分割点，找到最大长度的回文res2，res1与res2的和就是答案
链接：https://juejin.cn/post/7130274948001562661
```javascript

```


# 数组排序
## 时间复杂度和空间复杂度
![Alt](../image/时间复杂度和空间复杂度.png#pic_center)
## 排序算法
![Alt](../image/排序算法.png#pic_center)

```javascript

```