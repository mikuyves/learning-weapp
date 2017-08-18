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
// 重整数组内的元素

let skuStocks = skuList.map(item => item.stock);  // 得到所有 sku 的库存列表。

// 如果箭头函数内有多个 statements，需要用 return。
skuList = goods.goodsSkuInfo.goodsSkuDetails.map(item => {
const price = parseFloat(item.goodsSkuDetailBase.price).toFixed(2);
const sku = item.sku;
const stock = goods.goodsStocks.find(item => item.sku == sku).stock;
return {price, sku, stock};
});

// 计算 Number 数组的总和。
var sumStock = skuStocks.reduce((total, num) => total + num);

// 计算 Object 数组内 Object 的某个属性值的总和。
// bad 分两步，用两个方法。
let skuStocks = skuList.map(item => item.stock);
if (skuStocks.length > 0) {
  var sumStock = skuStocks.reduce((total, num) => total + num, 0);
}

// good 只需一步，一个方法。
let tp = this.skuList.reduce((total, item) => total + item.stock, 0)

// 还可以进行复杂的操作。 计算多个 sku 指定价格的总和。
let tp = this.markList.reduce((total, item) => total + item.sku[this.showPriceId] * item.qtt, 0)

```

#### filter()
```javascript
// 返回过滤后的新数组。
const stock = goods.goodsStocks.filter(item => item.sku.stock === 1)

// 使用箭头函数时，多个表达式，记得要 return。
let pointerColumnName = pointers[0].className.toLowerCase()
return pointers.map(p => {
  p.set(relationListName, relations.filter(
    r => r['attributes'][pointerColumnName]['id'] == p.id)
  );
  return p
})
```

#### find()
```javascript
// 返回找到的第一个元素。
const stock = goods.goodsStocks.find(item => item.sku === sku).stock;
```

#### every()
数组中，执行函数的结果全部为真，才返回真。
```javascript
isAllSoldOut() {
return this.skuList.map(item => item.isSoldOut).every((v) => v)
}
```

### 数据类型

#### 注意：返回值是 true/false
```javascript
{} ? true : false  // true
[] ? true : false // true
new Set() ?  true : false
new Map() ?  true : false
```


### 函数

#### bind()、apply() 和 call()
```javascript

```


### 拆解数组。
将一个含有子数组的数组，变成一个字含有基本元素的数组。
```javascript
static flatten(arr) {
return [].concat(...arr)
}
```
