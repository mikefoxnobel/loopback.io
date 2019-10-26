---
lang: zh
title: '存储库（Repositories）'
keywords: LoopBack 4.0, LoopBack 4
sidebar: zh_lb4_sidebar
permalink: /doc/zh/lb4/Repositories.html
layout: translation
---

存储库（`Repository`）代表了一个特定的服务（ `Service`）接口，为基于数据库或服务的域模型（Domain Model）提供强类型的数据访问操作（例如CRUD）。

![Repository diagram](imgs/repository.png)

{% include note.html content="存储库（Repositories）为数据模型提供行为（behavior）。数据模型（Model）描述了数据的形状，存储库提供了诸如CRUD操作之类的行为。这与LoopBack 3.x中，数据模型（Model）中实现行为不同。" %}

{% include tip.html content="一个数据模型（Model）可以被多个不同的存储库（Repository）使用。" %}

存储库（`Repository`）可以由应用程序开发者定义和实现。
LoopBack为典型的CRUD和KV操作提供了一些预定义的存储库接口（`Repository` interfaces）。
这些存储库`Repository`的实现基于数据模型（`Model`）的定义和数据源（`DataSource`）的配置，以完成数据访问逻辑。

```js
interface Repository<T extends Model> {}

interface CustomerRepository extends Repository<Customer> {
  find(filter?: Filter<Customer>, options?: Options): Promise<Customer[]>;
  findByEmail(email: string): Promise<Customer>;
  // ...
}
```

访问以下链接查看更多示例：

- [Repository/CrudRepository/EntityRepository](https://github.com/strongloop/loopback-next/blob/master/packages/repository/src/repositories/repository.ts)
- [KeyValueRepository](https://github.com/strongloop/loopback-next/blob/master/packages/repository/src/repositories/kv.repository.ts)

## 安装

对传统映射（Legacy juggler）的支持已经在`loopback-next`启用，可以由`@loopback/repository`包导入。
将`@loopback/repository`配置为你应用程序的依赖（Dependency）以实现此支持。

你可以将你喜欢的连接器（Connector）配置为你应用程序的依赖，以安装它们。

## 存储库混入（Repository Mixin）



`@loopback/repository`为你的应用程序提供了“混入（mixin）”，能够以方便的方法自动为你绑定存储库（Repository）类。
由组件声明的存储库（Repositories）同样会被自动绑定。

存储库（Repositoriy）被绑定到`repositories.${ClassName}`类。使用方法如下例：

```ts
import {Application} from '@loopback/core';
import {RepositoryMixin} from '@loopback/repository';
import {AccountRepository, CategoryRepository} from './repositories';

// 使用混入（Mixin）
class MyApplication extends RepositoryMixin(Application) {}

const app = new MyApplication();
// AccountRepository会被绑定到键`repositories.AccountRepository`
app.repository(AccountRepository);
// CategoryRepository会被绑定到键`repositories.CategoryRepository`
app.repository(CategoryRepository);
```

## 配置数据源

数据源（`DataSource`）是一个连接器（Connector）的命名配置。
配置的属性依据连接器不同而有差异。
例如，`MySQL`的数据源需要设置的连接器（`connector`）`loopback-connector-mysql`属性如下：

```json
{
  "host": "localhost",
  "port": 3306,
  "user": "my-user",
  "password": "my-password",
  "database": "demo"
}
```

连接器（`Connector`）为特定的后台系统提供了数据访问或API调用，这些后台系统包括数据库（database），REST服务，SOAP服务，gRPC微服务等。它将这些互动以Node.js方法的形式，抽象为一系列操作。

通常，一个连接器（Connector），将LoopBack查询及其变种，转换为本地的api调用，这种调用由特定后台基于Node.js的驱动支持。
例如，一个`MySQL`的连接器（Connector）会映射`create`方法到SQL
INSERT语句，能够被Node.js上的MySQL驱动所执行。

当一个数据源（`DataSource`）被实例化之后，配置属性会被用于初始化连接器（Connector）以连接到后台系统。你可以在你的Loopback 4应用程序中，用传统映射（legacy juggler）定义一个数据源（DataSource）如下：

{% include code-caption.html content="src/datsources/db.datasource.ts" %}

```ts
import {juggler} from '@loopback/repository';

// 这只是一个示例，“test”数据库并不真实存在。
export const db = new juggler.DataSource({
  connector: 'mysql',
  host: 'localhost',
  port: 3306,
  database: 'test',
  password: 'pass',
  user: 'root',
});
```

## 定义数据模型（Model）



数据模型（Model）像普通的JavaScript类一样定义。如果你希望你的数据模型（Model） 能够存储在数据库中，你的模型必须有一个`id`顺序ing，并且继承自`Entity`基类。

TypeScript示例：

```ts
import {Entity, model, property} from '@loopback/repository';

@model()
export class Account extends Entity {
  @property({id: true})
  id: number;

  @property({required: true})
  name: string;
}
```

JavaScript示例：

```js
import {Entity, ModelDefinition} from '@loopback/repository';

export class Account extends Entity {}

Account.definition = new ModelDefinition({
  name: 'Account',
  properties: {
    id: {type: 'number', id: true},
    name: {type: 'string', required: true},
  },
});
```

## 定义存储库（Repository）

继承`DefaultCrudRepository`类，创建一个存储库（Repository）类，提供传统映射桥（legacy
juggler bridge）并绑定到你的基于`Entity`的类，同时使用你先前配置的数据源（Datasource）。

建议使用[依赖注入（Dependency Injection）](Dependency-injection.md)来获取你的数据源。

TypeScript示例：

```ts
import {DefaultCrudRepository, juggler} from '@loopback/repository';
import {Account, AccountRelations} from '../models';
import {DbDataSource} from '../datasources';
import {inject} from '@loopback/context';

export class AccountRepository extends DefaultCrudRepository<
  Account,
  typeof Account.prototype.id,
  AccountRelations
> {
  constructor(@inject('datasources.db') dataSource: DbDataSource) {
    super(Account, dataSource);
  }
}
```

JavaScript示例：

```js
import {DefaultCrudRepository} from '@loopback/repository';
import {Account} from '../models/account.model';
import {db} from '../datasources/db.datasource';

export class AccountRepository extends DefaultCrudRepository {
  constructor() {
    super(Account, db);
  }
}
```

### 控制器（Controller）配置

当你为你的存储库（Respository）定义了数据源（DataSource）之后，你调用的这个存储库（Respository）的所有的CRUD方法都会使用映射（juggler）和你的连接器（Connector）的方法，除非你覆写了它们。
在你的控制器（Controller）中，你需要定义一个存储库（Respository）属性，并为其创建一个新的实例。在你的控制器的构造函数中，你需要如下实现：

```ts
export class AccountController {
  constructor(
    @repository(AccountRepository) public repository: AccountRepository,
  ) {}
}
```

### 为你的应用程序定义CRUD方法

When you want to define new CRUD methods for your application, you will need to
modify the API Definitions and their corresponding methods in your controller.
Here are examples of some basic CRUD methods:

1.  Create API Definition:

```json
{
  "/accounts/create": {
    "post": {
      "x-operation-name": "createAccount",
      "requestBody": {
        "description": "The account instance to create.",
        "required": true,
        "content": {
          "application/json": {
            "schema": {
              "type": "object"
            }
          }
        }
      },
      "responses": {
        "200": {
          "description": "Account instance created",
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/Account"
              }
            }
          }
        }
      }
    }
  }
}
```

Create Controller method:

```ts
async createAccount(accountInstance: Account) {
  return this.repository.create(accountInstance);
}
```

2.  Find API Definition:

```json
{
  "/accounts": {
    "get": {
      "x-operation-name": "getAccount",
      "responses": {
        "200": {
          "description": "List of accounts",
          "content": {
            "application/json": {
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/Account"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Find Controller method:

```ts
async getAccount() {
  return this.repository.find();
}
```

Don't forget to register the complete version of your OpenAPI spec through
`app.api()`.

Please See [Testing Your Application](Testing-Your-Application.md) section in
order to set up and write unit, acceptance, and integration tests for your
application.

## Access KeyValue Stores

We can now access key-value stores such as [Redis](https://redis.io/) using the
[KeyValueRepository](https://github.com/strongloop/loopback-next/blob/master/packages/repository/src/repositories/kv.repository.ts).

### Define a KeyValue Datasource

We first need to define a datasource to configure the key-value store. For
better flexibility, we split the datasource definition into two files. The json
file captures the configuration properties and it can be possibly overridden by
dependency injection.

1. redis.datasource.json

```json
{
  "name": "redis",
  "connector": "kv-redis",
  "host": "127.0.0.1",
  "port": 6379,
  "password": "",
  "db": 0
}
```

2. redis.datasource.ts

The class uses a configuration object to set up a datasource for the Redis
instance. By default, the configuration is loaded from `redis.datasource.json`.
We can override it by binding a new object to `datasources.config.redis` for a
context.

```ts
import {inject} from '@loopback/core';
import {juggler, AnyObject} from '@loopback/repository';
import * as config from './redis.datasource.json';

export class RedisDataSource extends juggler.DataSource {
  static dataSourceName = 'redis';

  constructor(
    @inject('datasources.config.redis', {optional: true})
    dsConfig: AnyObject = config,
  ) {
    super(dsConfig);
  }
}
```

To generate the datasource automatically, use `lb4 datasource` command and
select `Redis key-value connector (supported by StrongLoop)`.

### Define a KeyValueRepository

The KeyValueRepository binds a model such as `ShoppingCart` to the
`RedisDataSource`. The base `DefaultKeyValueRepository` class provides an
implementation based on `loopback-datasource-juggler`.

```ts
import {DefaultKeyValueRepository} from '@loopback/repository';
import {ShoppingCart} from '../models/shopping-cart.model';
import {RedisDataSource} from '../datasources/redis.datasource';
import {inject} from '@loopback/context';

export class ShoppingCartRepository extends DefaultKeyValueRepository<
  ShoppingCart
> {
  constructor(@inject('datasources.redis') ds: RedisDataSource) {
    super(ShoppingCart, ds);
  }
}
```

### Perform Key Value Operations

The KeyValueRepository provides a set of key based operations, such as `set`,
`get`, `delete`, `expire`, `ttl`, and `keys`. See
[KeyValueRepository](https://github.com/strongloop/loopback-next/blob/master/packages/repository/src/repositories/kv.repository.ts)
for a complete list.

```ts
// Please note the ShoppingCartRepository can be instantiated using Dependency
// Injection
const repo: ShoppingCartRepository =
  new ShoppingCartRepository(new RedisDataSource());
const cart1: ShoppingCart = givenShoppingCart1();
const cart2: ShoppingCart = givenShoppingCart2();

async function testKV() {
  // Store carts using userId as the key
  await repo.set(cart1.userId, cart1);
  await repo.set(cart2.userId, cart2);

  // Retrieve a cart by its key
  const result = await repo.get(cart1.userId);
  console.log(result);
});

testKV();
```

## Persist Data without Juggler [Using MySQL database]

{% include important.html content="This section has not been updated and code
examples may not work out of the box.
" %}

LoopBack 4 gives you the flexibility to create your own custom Datasources which
utilize your own custom connector for your favorite back end database. You can
then fine tune your CRUD methods to your liking.

### Example Application

You can look at
[the account-without-juggler application as an example.](https://github.com/strongloop/loopback-next-example/tree/master/services/account-without-juggler)

<!--lint enable no-duplicate-headings -->

1.  Implement the `CrudConnector` interface from `@loopback/repository` package.
    [Here is one way to do it](https://github.com/strongloop/loopback-next-example/blob/master/services/account-without-juggler/repositories/account/datasources/mysqlconn.ts)

2.  Implement the `DataSource` interface from `@loopback/repository`. To
    implement the `DataSource` interface, you must give it a name, supply your
    custom connector class created in the previous step, and instantiate it:

    ```ts
    export class MySQLDs implements DataSource {
      name: 'mysqlDs';
      connector: MySqlConn;
      settings: Object;

      constructor() {
        this.settings = require('./mysql.json'); // connection configuration
        this.connector = new MySqlConn(this.settings);
      }
    }
    ```

3.  Extend `CrudRepositoryImpl` class from `@loopback/repository` and supply
    your custom DataSource and model to it:

    ```ts
    import {CrudRepositoryImpl} from '@loopback/repository';
    import {MySQLDs} from './datasources/mysqlds.datasource';
    import {Account} from './models/account.model';

    export class NewRepository extends CrudRepositoryImpl<Account, string> {
      constructor() {
        const ds = new MySQLDs();
        super(ds, Account);
      }
    }
    ```

You can override the functions it provides, which ultimately call on your
connector's implementation of them, or write new ones.

### Configure Controller

The next step is to wire your new DataSource to your controller. This step is
essentially the same as above, but can also be done as follows using Dependency
Injection:

1.  Bind instance of your repository to a certain key in your application class

    ```ts
    class AccountMicroservice extends Application {
      private _startTime: Date;

      constructor() {
        super();
        const app = this;
        app.controller(AccountController);
        app.bind('repositories.NewRepository').toClass(NewRepository);
      }
    ```

2.  Inject the bound instance into the repository property of your controller.
    `inject` can be imported from `@loopback/context`.

    ```ts
    export class AccountController {
      @repository(NewRepository)
      private repository: NewRepository;
    }
    ```

### Example custom connector CRUD methods

Here is an example of a `find` function which uses the node-js `mysql` driver to
retrieve all the rows that match a particular filter for a model instance.

```ts
public find(
  modelClass: Class<Account>,
  filter: Filter<Account>,
  options: Options
): Promise<Account[]> {
  let self = this;
  let sqlStmt = "SELECT * FROM " + modelClass.name;
  if (filter.where) {
    let sql = "?? = ?";
    let formattedSql = "";
    for (var key in filter.where) {
      formattedSql = mysql.format(sql, [key, filter.where[key]]);
    }
    sqlStmt += " WHERE " + formattedSql;
  }
  debug("Find ", sqlStmt);
  return new Promise<Account[]>(function(resolve, reject) {
    self.connection.query(sqlStmt, function(err: any, results: Account[]) {
      if (err !== null) return reject(err);
      resolve(results);
    });
  });
}
```

## Example Application

You can look at
[the account application as an example.](https://github.com/strongloop/loopback4-example-microservices/tree/master/services/account)
