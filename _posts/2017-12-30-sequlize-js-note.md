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
