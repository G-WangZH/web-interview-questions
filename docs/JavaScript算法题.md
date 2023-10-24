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
