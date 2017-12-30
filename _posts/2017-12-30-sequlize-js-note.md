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

Use `sequlize.query` to run raw SQL queries.