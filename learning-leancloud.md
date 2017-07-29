# Learning Leancloud
微信小程序是一个全新的跨平台移动应用平台，LeanCloud 为小程序提供一站式后端云服务，为你免去服务器维护、证书配置等繁琐的工作，大幅降低你的开发和运维成本。

## 用户系统
小程序中提供了登录 API 来获取微信的用户登录状态，应用可以访问到用户的昵称、性别等基本信息，但是如果想要保存额外的用户信息，如用户的手机号码、收货地址等，则需要使用 LeanCloud 的用户系统。

### 一键登录
- 登录
```
AV.User.loginWithWeapp().then(user => {
  this.globalData.user = user.toJSON();
}).catch(console.error);
```

- 登陆后可以获得当前用户。以后可以用这个用户进行请求，获取 leancloud 的数据。
```
const user = AV.User.current();
```
这个 user 可以随时获取，不用再设置全局变量。

### 转化为 JSON
AV.Object 在模板上使用，最方便的方法是转成 JSON 格式。
例如，一个 AV.Object `prod` 有属性 `name` 为 "Sample name" ，要获取 `name` 的值，需要：
```
console.log(prod.attributes.name)  // "Sample name"
console.log(prod.name)  // undefined
```
若为 name 建立索引，则可以用 `prod.name` 取值，但我们不会为每一个属性都建立索引。
为了方便，在获取 AV.Object 的数据后，需要：
```
let prod = await new Query('Prod').first()
prod = prod.toJSON()
console.log(prod.name)  // "Sample name"
```

### 获取 Pointer 的属性值。
Query 不能获取 Pointer 的属性，只能获取其 objectId。要获取更多信息，需要用 `include`
```
let skus = await new AV.Query('Sku')
.equalTo('prod', prod)
.include('color')
.include('size1')
.find()
```

### set 方法
每个 AV.Object 都有 get 和 set 的方法。善用 set 方法任意组合获取的数据成为 JSON 对象。因为 Leancloud 关联查找实在不太人性化。例如：一个 prod 对应数个 sku，用 Pointer 进行关联，若要查询 prod 及其对应的 sku，需要查询两次，而这两次的查询是独立的。数据需要自己进行优化。
```
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