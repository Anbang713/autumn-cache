`autumn-cache` 定位于：一个更加贴近真实项目的缓存功能组件，其核心是提供更加友好的方法解决缓存相关场景的问题。比如：

（1）通过程序来规范缓存键的设计。

（2）提供更加灵活更加友好的缓存增删改查操作。

（3）提供本地缓存（基于caffeine）、分布式缓存（基于Redis）、二级缓存等方案的切换实现。

（4）提供缓存使用过程中场景的场景封装，比如：互斥锁、自动续约等。

# 1. 快速入门

需求：将根据用户id查找到的用户数据放入缓存，下次加载时，如果缓存中有则直接返回缓存中的数据。

## 1.1 创建缓存键

```java
CacheKey cacheKey = new CacheKeyBuilder()
  .module("user") // 模块，不允许为空
  .keyword("su") // 关键字，允许为空
  .bussiness("data") // 业务场景，不允许为空
  .build(); // 构造键对象
```

通常来说，不管是本地缓存还是分布式缓存，缓存的键都是字符串类型。但在真实项目中，对于缓存键通常有些约束和规范。比如：

（1）缓存键中不允许包含除：数据、字母、英文冒号、下划线外的特殊字符。

（2）缓存键的长度不允许超过xxx。

（3）缓存键中必须包含模块名、业务描述等信息。

基于以上原因，`autumn-cache` 默认提供 `CacheKey` 类来生成缓存键。实际上真正的缓存键为 `AbstractKey` ，使用者可以继承该类实现自己的缓存键，`CacheKey` 只是组件默认的实现。具体可见《如何定义自己的缓存键》。

## 1.2 写入缓存

将数据写入缓存，我们只需要调用 `CacheProvider` 类的 `add` 重载方法即可。

```java
CacheProvider cacheProvider = new CacheProvider();
// 写入缓存，使用系统默认的过期时长
cacheProvider.add(cacheKey, user);

// 写入缓存，自定义缓存的过期时长
cacheProvider.add(cacheKey, user, 5, TimeUnit.MINUTES);
```

**需要注意的是：** 所有被添加到缓存里的数据都应该具有一定的时效性，这样做的目的是为了降低缓存与数据库的一致性。

## 1.3 读取缓存

从缓存中读取数据，我们只需要调用 `CacheProvider` 类的 `get` 方法即可。通常来说：如果缓存存在，则直接返回；如果不存在，则从数据库中查询并将查询结果写入缓存。

```java
// 查询缓存
Optional<User> optional = cacheProvider.get(cacheKey, User.class);
if (optional.isPresent()) {
  return optional.get();
}

// 缓存为空，则从数据库查询
User user = userSerivce.get("su");
cacheProvider.add(cacheKey, user);
```

## 1.4 删除缓存

缓存有效期到了之后，会自动从缓存里删除。但是我们仍然需要提供手动删除缓存的方法来清除缓存。

```java
cacheProvider.remove(cacheKey);
```

以上就是缓存增删改查的基本用法，从入门案例中可以看出，缓存组件最核心的类 `CacheProvider` 承载着对缓存的所有操作。

