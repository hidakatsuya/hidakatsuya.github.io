---
title: Fixture はどのようにデータをロードしているか
---

`fixtures :user` がどのようにデータをロードしているのかを調べる。
なお、以下は Redmine の master の最新を使って検証を行った結果である。

## 準備

まず、検証用の ATest と BTest を追加する。
https://github.com/hidakatsuya/redmine/commit/36c68804179eea117ac6eb1ea57ac4d2eb898ca4

```ruby
# a_test.rb
require_relative '../test_helper'

class ATest < ActiveSupport::TestCase
  fixtures :users

  test "A test" do
    puts "-- A test"
    assert User.exists?(1)
  end
end

# b_test.rb
require_relative '../test_helper'

class BTest < ActiveSupport::TestCase
  fixtures :projects

  test "B test" do
    puts "-- B test"
    assert User.exists?(1)
  end
end
```

さらに、クエリの動きを観察するために、ActiveRecord の SQLイベントを購読する。

```ruby
# test_helper.rb
ActiveSupport::Notifications.subscribe('sql.active_record') do |name, start, finish, id, payload|
  puts "SQL: #{payload[:sql]}"
end
```

## 検証

テストを実行する。
```
$ bin/rails test test/unit/a_test.rb test/unit/b_test.rb
```

A -> B の順で実行されるとき、fixture はデータベースを次のように操作する。

### 1. a_test.rb

1. `fixtures :users` のロード
   1. `SQL: begin transaction`
   2. `SQL: DELETE FROM "users";`: users テーブルをクリア
   3. `INSERT INTO "userstes" ...`: users.yml のデータを全て追加
   4. `SQL: commit transaction`
2. ATest の実行
   1. `SQL: begin transaction`
   2. `test "A Test"` を実行
   3. `SQL: rollback transaction`

### 2. b_test.rb

1. `fixtures :projects` のロード
   1. `SQL: begin transaction`
   2. `SQL: DELETE FROM "projects"`: projects テーブルをクリア
   3. `INSERT INTO "projects" ...`: projects.yml のデータを全て追加
   4. `SQL: commit transaction`
2. BTest の実行
   1. `SQL: begin transaction`
   2. `test "B Test"` を実行
   3. `SQL: rollback transaction`

a_test.rb の実行後、users テーブルのデータは残っているので、b_test.rb は users.yml のデータ + projects.yml のデータの状態で実行されることになる。

## 付録

<details><summary>テスト実行時の全ての出力結果</summary>

```
$ bin/rails test test/unit/a_test.rb test/unit/b_test.rb

Run options: --seed 50730

# Running:

SQL: PRAGMA table_xinfo("users")
SQL: SELECT sql FROM
  (SELECT * FROM sqlite_master UNION ALL
   SELECT * FROM sqlite_temp_master)
WHERE type = 'table' AND name = 'users'
SQL: PRAGMA table_xinfo("users")
SQL: SELECT sql FROM
  (SELECT * FROM sqlite_master UNION ALL
   SELECT * FROM sqlite_temp_master)
WHERE type = 'table' AND name = 'users'
SQL: begin transaction
SQL: PRAGMA foreign_keys
SQL: PRAGMA defer_foreign_keys
SQL: PRAGMA defer_foreign_keys = ON
SQL: PRAGMA foreign_keys = OFF
SQL: DELETE FROM "users";
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (1, 'admin', 'b5b6ff9543bf1387374cdfa27a54c96d236a7150', 'Redmine', 'Admin', 1, 1, '2006-07-19 20:57:52', 'en', NULL, '2006-07-19 17:12:21', '2006-07-19 20:57:52', 'User', 'all', '82090c953c4a0000a7db253b0691a6b4');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (2, 'jsmith', 'bfbe06043353a677d0215b26a5800d128d5413bc', 'John', 'Smith', 0, 1, '2006-07-19 20:42:15', 'en', NULL, '2006-07-19 17:32:09', '2006-07-19 20:42:15', 'User', 'all', '67eb4732624d5a7753dcea7ce0bb7d7d');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (3, 'dlopper', '8f659c8d7c072f189374edacfa90d6abbc26d8ed', 'Dave', 'Lopper', 0, 1, NULL, 'en', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'User', 'all', '7599f9963ec07b5a3b55b354407120c0');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (4, 'rhill', '9e4dd7eeb172c12a0691a6d9d3a269f7e9fe671b', 'Robert', 'Hill', 0, 1, NULL, 'en', NULL, '2006-07-19 17:34:07', '2006-07-19 17:34:07', 'User', 'all', '3126f764c3c5ac61cbfc103f25f934cf');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification") VALUES (5, 'dlopper2', '1', 'Dave2', 'Lopper2', 0, 3, NULL, 'en', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'User', 'all');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification") VALUES (6, '', '1', '', 'Anonymous', 0, 0, NULL, '', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'AnonymousUser', 'only_my_events');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (7, 'someone', '8f659c8d7c072f189374edacfa90d6abbc26d8ed', 'Some', 'One', 0, 1, NULL, 'en', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'User', 'only_my_events', '7599f9963ec07b5a3b55b354407120c0');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification", "salt") VALUES (8, 'miscuser8', '8f659c8d7c072f189374edacfa90d6abbc26d8ed', 'User', 'Misc', 0, 1, NULL, 'it', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'User', 'only_my_events', '7599f9963ec07b5a3b55b354407120c0');
INSERT INTO "users" ("id", "login", "hashed_password", "firstname", "lastname", "admin", "status", "last_login_on", "language", "auth_source_id", "created_on", "updated_on", "type", "mail_notification") VALUES (9, 'miscuser9', '1', 'User', 'Misc', 0, 1, NULL, 'it', NULL, '2006-07-19 17:33:19', '2006-07-19 17:33:19', 'User', 'only_my_events');
INSERT INTO "users" ("id", "lastname", "status", "created_on", "updated_on", "type") VALUES (10, 'A Team', 1, '2024-12-08 15:20:14.581047', '2024-12-08 15:20:14.581047', 'Group');
INSERT INTO "users" ("id", "lastname", "status", "created_on", "updated_on", "type") VALUES (11, 'B Team', 1, '2024-12-08 15:20:14.581047', '2024-12-08 15:20:14.581047', 'Group');
INSERT INTO "users" ("id", "lastname", "status", "created_on", "updated_on", "type") VALUES (12, 'Non member users', 1, '2024-12-08 15:20:14.581047', '2024-12-08 15:20:14.581047', 'GroupNonMember');
INSERT INTO "users" ("id", "lastname", "status", "created_on", "updated_on", "type") VALUES (13, 'Anonymous users', 1, '2024-12-08 15:20:14.581047', '2024-12-08 15:20:14.581047', 'GroupAnonymous')
SQL: PRAGMA defer_foreign_keys = 0
SQL: PRAGMA foreign_keys = 1
SQL: commit transaction
SQL: begin transaction
-- A test
SQL: SELECT 1 AS one FROM "users" WHERE "users"."type" IN (?, ?) AND "users"."id" = ? LIMIT ?
SQL: rollback transaction
.SQL: PRAGMA table_xinfo("projects")
SQL: SELECT sql FROM
  (SELECT * FROM sqlite_master UNION ALL
   SELECT * FROM sqlite_temp_master)
WHERE type = 'table' AND name = 'projects'
SQL: PRAGMA table_xinfo("projects")
SQL: SELECT sql FROM
  (SELECT * FROM sqlite_master UNION ALL
   SELECT * FROM sqlite_temp_master)
WHERE type = 'table' AND name = 'projects'
SQL: begin transaction
SQL: PRAGMA foreign_keys
SQL: PRAGMA defer_foreign_keys
SQL: PRAGMA defer_foreign_keys = ON
SQL: PRAGMA foreign_keys = OFF
SQL: DELETE FROM "projects";
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (1, 'eCookbook', 'Recipes management application', 'http://ecookbook.somenet.foo/', 1, NULL, '2006-07-19 17:13:59', '2006-07-19 20:53:01', 'ecookbook', 1, 10);
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (2, 'OnlineStore', 'E-commerce web site', '', 0, NULL, '2006-07-19 17:14:19', '2006-07-19 17:14:19', 'onlinestore', 11, 12);
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (3, 'eCookbook Subproject 1', 'eCookBook Subproject 1', '', 1, 1, '2006-07-19 17:15:21', '2006-07-19 17:18:12', 'subproject1', 6, 7);
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (4, 'eCookbook Subproject 2', 'eCookbook Subproject 2', '', 1, 1, '2006-07-19 17:15:51', '2006-07-19 17:17:07', 'subproject2', 8, 9);
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (5, 'Private child of eCookbook', 'This is a private subproject of a public project', '', 0, 1, '2006-07-19 17:15:51', '2006-07-19 17:17:07', 'private-child', 2, 5);
INSERT INTO "projects" ("id", "name", "description", "homepage", "is_public", "parent_id", "created_on", "updated_on", "identifier", "lft", "rgt") VALUES (6, 'Child of private child', 'This is a public subproject of a private project', '', 1, 5, '2006-07-19 17:15:51', '2006-07-19 17:17:07', 'project6', 3, 4)
SQL: PRAGMA defer_foreign_keys = 0
SQL: PRAGMA foreign_keys = 1
SQL: commit transaction
SQL: begin transaction
-- B test
SQL: SELECT 1 AS one FROM "users" WHERE "users"."type" IN (?, ?) AND "users"."id" = ? LIMIT ?
SQL: rollback transaction
.

Finished in 0.046865s, 42.6756 runs/s, 42.6756 assertions/s.
2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

</details>
