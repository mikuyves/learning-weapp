# Learning Leancloud
微信小程序是一个全新的跨平台移动应用平台，LeanCloud 为小程序提供一站式后端云服务，为你免去服务器维护、证书配置等繁琐的工作，大幅降低你的开发和运维成本。

## 用户系统
小程序中提供了登录 API 来获取微信的用户登录状态，应用可以访问到用户的昵称、性别等基本信息，但是如果想要保存额外的用户信息，如用户的手机号码、收货地址等，则需要使用 LeanCloud 的用户系统。

### 一键登录
- 登录
```javascript
AV.User.loginWithWeapp().then(user => {
  this.globalData.user = user.toJSON();
}).catch(console.error);
```

- 登陆后可以获得当前用户。以后可以用这个用户进行请求，获取 leancloud 的数据。
```javascript
const user = AV.User.current();
```
这个 user 可以随时获取，不用再设置全局变量。

### 转化为 JSON
AV.Object 在模板上使用，最方便的方法是转成 JSON 格式。
例如，一个 AV.Object `prod` 有属性 `name` 为 "Sample name" ，要获取 `name` 的值，需要：
```javascript
console.log(prod.attributes.name)  // "Sample name"
console.log(prod.name)  // undefined
```
若为 name 建立索引，则可以用 `prod.name` 取值，但我们不会为每一个属性都建立索引。
为了方便，在获取 AV.Object 的数据后，需要：
```javascript
let prod = await new Query('Prod').first()
prod = prod.toJSON()
console.log(prod.name)  // "Sample name"
```

### JSON 与 AV.Object 数据转换
```javascript
let jsonObject = AV.Object.toFullJSON()  // 把 Object 的数据完整转化到 JSON。主要是 _className 属性。
let avObject = AV.ParseJSON(jsonObject)  //

AV.Object.toFullJSON 和 AV.ParseJSON 两个方法互逆。 

```

### 获取 Pointer 的属性值。
Query 不能获取 Pointer 的属性，只能获取其 objectId。要获取更多信息，需要用 `include`
```javascript
let skus = await new AV.Query('Sku')
.equalTo('prod', prod)
.include('color')
.include('size1')
.find()
```

### set 方法

每个 AV.Object 都有 get 和 set 的方法。善用 set 方法任意组合获取的数据成为 JSON 对象。因为 Leancloud 关联查找实在不太人性化。例如：一个 prod 对应数个 sku，用 Pointer 进行关联，若要查询 prod 及其对应的 sku，需要查询两次，而这两次的查询是独立的。数据需要自己进行优化。
```javascript
// 获取 Prod
let query = new AV.Query('Prod')
query.descending('createdAt')  // 创建时间倒排。
query.skip(this.start)  // 跳过的条目，用于翻页。
query.limit(this.count)  // 每次获取的条目数。
query.include('mainPic')  // 获取 Pointer 属性值。
query.include('brand')
query.include('cate')
query.include('supplier')
let prods = await query.find();

// 获取每个 prod 对应的 sku。
for (let prod of data) {
  let skus = await new AV.Query('Sku')
    .equalTo('prod', prod)
    .include('color')
    .include('size1')
    .find()

  // 注意这里要转 JSON，之后才能用 `.` 方法获取值。
  let skuList = skus.map(item => item.toJSON())
  let skuStock = skuList.map(item => item.stock)
  let skuStock = skuList.map(item => item.stock)
  let skuNames = skuList.map(item => item.color.name).join(' ') // 合并字符串。
  let sumStock = skuStock.reduce((total, num) => total + num) // 求和。

  // 关联数据。
  prod.set('skuList', skuList)
  prod.set('stock', sumStock)
  prod.set('skuNames', skuNames)
  prod = prod.toJSON()
}
```

### Pointer 反查

LeanCloud 已弃用 AV.Relation，多对一的关系就比较烦。只能在“多”的一方用 Pointer 关联起来，不能再“一”的一方建立 relation。那么用 Pointer 就涉及反查询的问题。对于每一个 Pointer 需要查询两次，产生两个问题：
1. 如果是一个 Pointer 列表，这要查询 n x 2 次？
2. 查询后，经常需要把 Pointer 和 Relation 合拼成一个对象，方便之后调用。

例如，一个商品（prod）有数个规格（SKU），每次查询商品的时候需要同时获得SKU的详情，处理数据（包括：商品总数、价格区间、规格名称等），并让数据组合成如下关系：
```javascript
prods: [
  prod: {
    name: 'xxx',
    // 其他属性...
    skuList: [
      {
        name: 'xxx-yyy',
        price: 300,
        stock: 20,
        // 其他属性...
      },
      {
        name: 'xxx-zzz',
        price: 330,
        stock: 15,
        // 其他属性...
      },
    ]
  }
]
```

解决方案如下：
```javascript
// 查询 Pointers
/**
 *  分页处理。获取获取商品列表，包括 Pointer 的数据。
 */
async function getPointers(start, count) {
  let query = new AV.Query('Prod')
  query.descending('createdAt')
  query.skip(start)
  query.limit(count)
  query.include('mainPic')
  query.include('brand')
  query.include('cate')
  query.include('supplier')
  return await query.find();
}

// 查询 Relations
/**
 *  分页处理。获取获取指定商品对应的 SKU 列表，包括 Pointer 的数据。
 */
async function getRelations(prods) {
  let skus = await new AV.Query('Sku')
    .containedIn('prod', prods)  // 重点：这里只需做一次请求，连上述1次，共2次。
    .include('color')
    .include('size1')
    .find()
  return skus
}

// 合拼数据
/**
 *  Leancloud Pointer 反查方法，翻查后把 relation 组合到 Pointer 的数据中。
 */
function mapPointerRelation(pointers, relations) {
  // 注意在设定 Pointer 的 Column Name 时，必须是 Pointer 同名的小写。
  // 返回： 一个 Pointer 的数组，此数组以包含一个以 ‘<relationName>List' 命名的 relation 对象的数组。

  // 无 relations，直接返回。
  if (relations.length < 1) {
    return pointers
  }
  // 有 relations，关联起来。
  let relationListName = relations[0].className.toLowerCase() + 'List'
  let pointerColumnName = pointers[0].className.toLowerCase()
  return pointers.map(p => {
    p.set(relationListName, relations.filter(
      r => r['attributes'][pointerColumnName]['id'] == p.id)
    );
    return p
  })
}
```

### 查询后，数据重新组合。
```javascript
  /**
   * Leancloud AV.Object 通用方法。以每一行数据的某个 pointer 为中心把数据重新组合。
   * 可用于 pointer 数据反查，中间表查询后排列。
   * @param  {[Array]} list          进行重组的数据。
   * @param  {[String]} key          pointer 在数据表中的名字。
   * @param  {[String]} subListName  组合后的子数组的名字。
   * @return {[Array]}               返回的数组格式为: [
   *                                   key: pointer <AV.Object>
   *                                   subListName: subList <Array>
   *                                 ]
   */
  static groupByPointer(list, key, subListName) {
    let ids = list.map(item => {
      if (item[key].objectId) {
        return item[key].objectId;
      };
    });
    ids = [...new Set(ids)];
    let newList = [];
    for (let id of ids) {
      let pointer = list.find(item => item[key].objectId == id)[key];
      let subList = list.filter(item => item[key].objectId === id);
      let obj = {};
      obj[key] = pointer;
      obj[subListName] = subList;
      newList = [...newList, obj];

    }
    return newList;

  }
```


### 查询 - 判断 pointer 相等
注意，必须要用 createWithoutData, 尤其是 _User 类别，会找不到，因为 session 会不同。
使用了 createWithoutData，Leancloud 只需判断 objectId 是否相等，不会理会其他属性。
```javascript
let sku = AV.Object.createWithoutData('Sku', m.sku.objectId);
let res = await new AV.Query('SCart')
  .equalTo('handler', new AV.Object.createWithoutData('_User', handler.toJSON().objectId))
  .equalTo('customer', new AV.Object.createWithoutData('_User', customer.objectId))
  .equalTo('sku', sku)  
  .first()
```

### 内嵌查询
[https://leancloud.cn/docs/leanstorage_guide-js.html#内嵌查询]
```
  // 构建内嵌查询
  var innerQuery = new AV.Query('TodoFolder');
  innerQuery.greaterThan('likes', 20);

  // 将内嵌查询赋予目标查询
  var query = new AV.Query('Comment');

  // 执行内嵌操作
  query.matchesQuery('targetTodoFolder', innerQuery);
  query.find().then(function (results) {
     // results 就是符合超过 20 个赞的 TodoFolder 这一条件的 Comment 对象集合
  }, function (error) {
  });

  query.doesNotMatchQuery('targetTodoFolder', innerQuery);
  // 如此做将查询出 likes 小于或者等于 20 的 TodoFolder 的 Comment 对象
```

### AV.Promise.all
* 批量删除  `AV.Object.detroyAll(objectList)`
* 批量保存  `AV.Object.saveAll(objectList)`
* 批量获取  `AV.Object.fetchAll(objectList)`
* 多个 promise 一起请求 `AV.Promise.all(promiseList)`
```javascript
AV.Promise.all([
  AV.Object.destroyAll(pics),
  prod.destroy(),
  AV.Object.destroyAll(skus)
])
```


## 源码
@since 2017-08-13
