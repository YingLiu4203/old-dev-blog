---
layout: post
title: Sequlize.js Note
categories:
- Notes
tags:
- Sequlize node.js
---
# Sequelize.js Note

Sequelize is a promise-based ORM for Node.js. It supports dialects PostgreSQL, MySQL, SQLite and MSSQL. It features transaction support, relations, read replication and more. This is a note based on [Sequlize Office Doc](http://docs.sequelizejs.com/).

## 1 Basic Usage

You first create an instance of Sequelize by `new Sequelize(otpionsObject)`. The options include:

* `database`, `username`, `password`: the database (can be a connection string) and user info.
* `dialect`: one of `mysql`, `sqlite`, `postgres`, `mssql`.
* `host`, `port`, `protocol`: connection data.
* `dialectOptions{ ... }`: any dialect-specific options.
* `storage`: path to sqlite db file.
* `logging`: default is false using `console.log`.
* `omitNull`: disable inserting undefined values as NULL. default: false.
* `native`: use a natvie lib or not. default: true.
* `define`: specify `sequelize.define` options such as `timestamps`, `underscored`, and etc.
* `sync`: force syn for models. default: `{force: true}`.
* `pool`: pool configurations include `max`, `idle`, and `acquire`.
* `isolationLevel`: specify transaction isolation level.

Use `sequlize.query` to run raw SQL queries. You can specify a model to map the query result. it can take `?` unnamed parameters and `:key` named parameters that are replacements in the query.

## 2 Model Definition

The `define` method defines a mapping between a model and a table. A column has some options like `type`, `allowNull`, `defaultValue`, `unique`, `autoIncrement`, `field` for custom field name, `references` for foreign keys. `indexes: [{unqiue: true|false, fields: ['field1', 'field2'],...]` is a special field that specifies indexes in the model.

By default, Sequelize adds the attributes `createdAt` and `updatedAt` to  a model. The two fields are also needed in Sequelize migrations. The fields can be disabled or customized.

Some common data types are below -- all in a syntax of `Sequelize.TYPE`:

* `STRING`, `STRING(1234)`, `STRING.BINARY`, `TEXT`, `TEXT('tiny')`
* `INTEGER`, `BIGINT`, `FLOAT`, `DOUBLE`, `DECIMAL`
* `DATE`, `DATEONLY` only date wihtout time
* `BOOLEAN`, `ENUM('val1', 'val2')`
* `JSON`, `JSONB` Postgresql only
* `BLOB`, `BLOB('tiny')`, `UUID`

You can define getters and setters for object properties or as part of the model options. `this.getDataValue()` and `this.setDataValue()` are used to retrieve and set an underlying property value.

A field can use predeined or custom validations including: `is`, `not`, `isEmail`, `isUrl`, `isIP`, `isInt`, `isLowercase`, `notNull`, `notEmpty`, `equals`, `isUUID`, `isDate`, `isBefore`, `isAfter`, `max`, `min`, `isCreditCard`, etc. It also allows model validation.

You can specify model options such as `timestamps`, `paranoid` to not delete database entries, `underscored` to use camelcase or understcored name, `tablename` for custom table name, `version: true` to turn on optimistic locking.

You can call the `sync()` to create a table, `drop()` to drop a table. To create or drop all tables, use `sequelize.sync()` or `sequelize.drop()`. For `sync()` method, use an option `{ force: true|false, match: /regex/ }` to specify whether to override an existing table.

You can define a class level method using `ClassName.classLevelMethod = function() {...}` or an instance level method using `ClassName.prototype.instanceLevelMethod = function () { ... }`.

## 3 Model Usage

### 3.1 CRUD

* `findById()` search by id.
* `findOne({ where: { column: 'value'}})` searches by attributes, returns sepecified attributes.
* `findOrCreate({where: {...}, defaults: {...}})` finds an array of matched records or create a new one.
* `create({...})` creates a new instance.
* `destroy()` deletes an instance.
* `update()` update an instance.
* `findAndCountAll({where: {...})` find and returns `count` and `rows`. Use `include` to specify the include criteria.
* `finaAll()` returns an array of matched instances.
* `count()` to count the matched rows.
* `max('column')` and `min('column')` to get the max/min value of a column.
* `sum('column')` to get the sum.

For each row you select, Sequelize creates an instance with functions for update, delete, get associations etc. For many read-only rows, use `raw: true`.

### 3.2 Attributes

Use `attributes: ['col1', 'col2']` to porject columns. use a nested array `attributes: ['col1', ['col2', 'col2Alias']]` to define an attribute alias.

Use `sequelize.fn('funName', sequelize.col('column'))` to call `SELECT funName(column)`. To include all atrributes with an aggregator, use `attributes: { include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']] }`. To exclude some attributes, use `attributes: { exclude: ['baz'] }`.

### 3.3 Where

`where` takes an object of `attribute: value` pairs. The value can be primitives for equality matches or keyed objects for other operators. The operators in `Op.operator` are: `and`, `or`, `gt`, `gte`, `lt`, `lte`, `ne`, `eq`, `not`, `between`, `notBetween`, `in`, `notIn`, `like`, `notLike`, `regexp`, `notRegexp`, `iRegExp`, `notIRegexp`, `like`, and etc. Operators can be combined as `[Op.or]: { [Op.lt]: 1000, [Op.eq]: null }`. The default combinator is `[Op.and]`.

Sequelize allows operator aliases but not recommended.

JSONB can be queried as nested objects, nested key or containment.

### 3.4 Assoications

Use `include` to eager load associated models. Use `include: [{ all: true }]` to include everything. Use nested `include` to eager load nested models. Use `include: [{ all: true, nested: true }]` to laod everything nested. Use `$nexted.colunm$` in top elvel `where` property.

### 3.5 Pagination / Limiting

You can use `limit`, `offset`, `order` and `group` in a query.

## 4 Instances

Use `build` method to create an unsaved object that can be saved by calling `save()`. Use `create` to create a persistent instance. use `update` and `destroy` to update and delete instances.

Use `Model.bulkCreate()`, `Model.update()` and `Model.destroy()` to perform bulk operations.

The returned instance has additional properties. To only get the column values, use `get({ plain: true})`.

Use `reload()` to get the current data. Postgresql supports `increment` and `decrement` methods that allow updating data withtout running into concurrency issues.

## 5 Assoications

In `User.relationMethod(Project)`, the `User` is the **source** and the `Project` is the **target**.

`Source.belongsTo(Target)` defines a one-to-one relations. It adds a camelCase (or underscored if `underscored: true` ) foreign key `targetIdColumn` in source table. If `{as: 'fkName'}` is provided, the foreign key is `fkNamId`. `{foreignKey: 'name'}` directly gives the foreign key name. You can specify a target column using `targetKey`.

`Source.hasOne(Target)` defines a one-to-one relation and adds a foreign key in target. It also adds accessor methods to the source as `getTarget` and `setTarget`.

`Source.hasMany(Target)` defines a one-to-many association. It adds a foreign key column in the target. It adds accessor methods to the source. Foreign key name and source column can be cusotmized.

`M1.belongsToMany(M2, {through: 'M1M2'}); M2.belongsToMany(M1, {through: 'M1M2'});` defines a many-to-many relationship between M1 and M2 through a new model `M1M2`. It adds accessor methods and foreign keys.

When you create associations, foreign key references with constratints will be created automatically. The constraint can be customized using `onUpdate` and `onDelete` options. Valid options are `RESTRICT`, `CASCADE`, `NO ACTION`, `SET DEFAULT`, and `SET NULL`. For one-to-one and one-to-many, the default is `SET NULL` for deletion and `CASCADE` for updates. For many-to-many, the default is `CASCADE` for both delte and update. use `constraints: false` to avoid cyclic dependency error.

An instance can be created with nested assoications in one step.

## 6 Transactions

Sequelize support automatical transaction management and explict transaction management. The automatic one uses callback and the manual one use promise `then()`.

Use `sequelize.transaction(function (t) {... })` to use the automatic transaction management. The sequelize methods use `{transaction: t}` as the 2nd argument. To automatically pass transaction context to all operations, use continuation local storage module and instantiate a namespace. Below is an example: 

```javascript
const cls = require('continuation-local-storage')
const namespace = cls.createNamespace('my-very-own-namespace')
const Sequelize = require('sequelize')
Sequelize.useCLS(namespace)

const sequelize = new Sequelize(....)
    // With CLS enabled, all operations are inside the transaction
})
```

Sequelize also support concurrent and partial transactions. SQLite supports only one transaction.

When creating transaction context, you can configure it with the following options: 

* `automcommit`: default is `true`
* `isolationLevel`: default is `'REPEATABLE_READ`.
* `deferrable`: `'NOT DEFERRABLE'` for Postgres.

## 7 Scope and Hooks

Scoping defines commonly used queries that can be reused by one or more models. You can define `socpes` in model creation or call `addScope` to add scopes to a model. A `defaultScope` is always applied. You can remove the default scope by calling `unscoped()` or invoking another scope as `scope(null)` or `scope(anouther)`.

Scopes apply to `find`, `findAll`, `count`, `update`, `increment` and `destroy`.

You can add scope to assoiations which will be applied automatically for the accessor methods.

Hooks are lifecycle events that allow functions to be called before or after sequelize create, destroy, update, and save operations. You use `hooks` property in the option object when you define a model. You can use `hook`, `addHook`, or `Model.hookName` to define hooks and use `removeHook` to remove hooks.

You can define global hooks that is executed in every model.

## 8 Migrations

Sequelize allows you to use migration files to transfer database schema into another state. The migration files describe the way how to get to the new state and how to revert the changes in order to get back to the old state. The `sequelize-cli` supports migrations and project bootstrapping.

### 8.1 Bootstrapping

Run the following two command to create an empty project.

```sh
npm install --save sequelize-cli
node_modules/.bin/sequelize init
```

The following folders will be created:

* `config`: contains the `config.json` file that tells CLI how to connect with database.
* `models`: contains all models.
* `migrations`: contains all migration files
* `seeders`: contains all seed files.

The following is a sample config file: 

```json
{
  development: {
    username: 'root',
    password: null,
    database: 'database_development',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'root',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.PROD_DB_USERNAME,
    password: process.env.PROD_DB_PASSWORD,
    database: process.env.PROD_DB_NAME,
    host: process.env.PROD_DB_HOSTNAME,
    dialect: 'mysql'
  }
}
```

If database doesn't exist, call `db:create` to create a new one.

### 8.2 Creating First Model and Migration

Use `model:generate` to create a model. `node_modules/.bin/sequelize model:generate --name User --attributes firstName:string,lastName:string,email:string` create a model file `user` in the `models` folder and a migration file like `xxxx-create-user.js` in `migrations` folder.

Then run `node_modules/.bin/sequelize db:migrate` to create a table in database. It executes the following steps:

* create a `SequelizeMeta` table to record the migration.
* run new migration files by checking the `SequelizeMeta` table.
* record the mirgration in the `SequelizeMeta` table.

Run `node_modules/.bin/sequelize db:migrate:undo` to revert the most recent migration. Use `db:migrate:undo:all` to revert back to initial state. Use `--to migration-file.js` to go back to a specific migration.

Run `node_modules/.bin/sequelize seed:generate --name demo-user` to create a seed file in `seeders` folder. The seed file has teh same `up` and `down` semantics like migration files. Run `node_modules/.bin/sequelize db:seed:all` to create seeds in a database or `node_modules/.bin/sequelize db:seed:undo` to revert seeds.

### 8.3 Configuration

Use `.sequelizerc` to configure Sequelize file paths. The configuration file can be replaced by JS code file. It's helpful to use environment variables like `process.env.PROD_DB_USERNAME`.

The migration and seed storage can be `sequelize` using the `SequelizeMeta` table, `json` using JSON files or `none` for no storage.