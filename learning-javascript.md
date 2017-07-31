# Javascript

### 数组

#### 最大值 最小值
数字数组中，获取最大值和最小值。ES6 快捷方法：
```javascript
let arr = [1, 3, 4, 6, 7];
Math.max(...arr)  // 7
Math.min(...arr)  // 1
```

#### 数组去重
```javascript
let oldArray = [1, 2, 3, 3, 4, 5, 5, 5]
let newArray = [...new Set(oldArray)]
console.log(newArray) // [1, 2, 3, 4, 5]

// 举个例子：
// 处理显示的规格名称。
let skuNames = [...new Set(skuList.map(item => item.color.name))].join(' ');
let skuSizes = [...new Set(skuList.map(item => item.size1.name))].join(' ');
```

#### map() 和 reduce()
```javascript

```

#### filter()
```javascript

```


### 函数

#### bind()、apply() 和 call()
```javascript

```
