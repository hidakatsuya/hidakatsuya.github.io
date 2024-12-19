---
title: Redmine の CI を安定化させるためにやったこと
---

これは [Redmine Advent Calendar 2024](https://adventar.org/calendars/10311) の19日目の記事です。

今年の 3月から Redmine の開発に携わるようになりましたが、 実は Redmine を触ったのは、バージョン 3.x 系以来9年ぶりでした。
そんな中、この9ヶ月での redmine.org へのチケットの投稿は次の通りでした。

* 投稿したチケットの数 = 20
* 取り込まれたチケットの数 = 19
* Patch = 11, Defect = 8, Feature = 1

このうち 8つのチケットが、CI またはテスト関連の改善です。
というわけで、今回は Redmine の CI を安定化させるために取り組んだことについて書きます。

## ひたすら落ちているテストを直す

暇さえあれば CI の結果を確認して、落ちているテストを直しました。

その中でもランダムで落ちるテストの原因の傾向としては、次の二点にまとめることができます。

* ソート順が保証されていない
* テストデータが不足している

### #41901 Fix random test failure in DestroyProjectsJobTest due to unsorted `@projects`

https://www.redmine.org/issues/41901

これは、ソート順が保証されていないケースとしてわかりやすいものです。

```diff
--- a/test/unit/jobs/destroy_projects_job_test.rb
+++ b/test/unit/jobs/destroy_projects_job_test.rb
@@ -23,7 +23,7 @@ class DestroyProjectsJobTest < ActiveJob::TestCase
   fixtures :users, :projects, :email_addresses

   setup do
-    @projects = Project.where(id: [1, 2]).to_a
+    @projects = Project.where(id: [1, 2]).order(:id).to_a
     @user = User.find_by_admin true
     ActionMailer::Base.deliveries.clear
   end
```

失敗するテストコードはこちら。

```ruby
test "schedule should enqueue job" do
  assert_enqueued_with(
    job: DestroyProjectsJob,
    args: [[1, 2], @user.id, '127.0.0.1']
  ) do
    @user.remote_ip = '127.0.0.1'
    DestroyProjectsJob.schedule @projects, user: @user
  end
end
```

`DestroyProjectsJob.schedule()` は、引数の `@projects` の id を取り出してジョブにエンキューするので、
`setup` で `@projects` のクエリ条件の `[1, 2]` を期待としています。
しかし、 `@projects` 自体のソート順が保証されていないため、ランダムで失敗してしまいます。

### #41931 Fix random failures in IssueRelationTest#test_create_with_initialized_journals due to ambiguous conditions for retrieving the expected detail

https://www.redmine.org/issues/41931

これも、ソート順が関係しているものですが、この原因を特定するのはとても苦労しました。
このチケットを投稿した時間が 2:41 であることからもわかります。

失敗するテストコードはこちら。

```ruby
assert_equal 'relation', to.journals.last.details.last.property
```

`to.journals.last.details` は、実際には3つのデータが含まれ、かつソート順が保証されていないために失敗します。
対象の fixtures を blame すると、当初のテストデータでは `details` は1つのレコードしか持たなかったようです。しかし、その後テストデータの変更によって、データが増えて、ランダムで失敗するようになったということだと思います。

### #41623 Fix tests that randomly failed due to required fixtures not being loaded

https://www.redmine.org/issues/41623

必要な fixtures が宣言されておらず、ランダムで失敗するテストをまとめて直したチケットです。
なぜ必ず失敗するのではなく、ランダムで失敗するのかについては後述します。

少し前までの Redmine のテストでは、こんな風に失敗するテストは、たいてい fixtures の不足によるものでした。
そして、それを直しても直しても、一向に減らない。そういう状況でした。

```
Failure:
ChangesetTest#test_ref_keywords_any [test/unit/changeset_test.rb:53]:
Expected: 3
  Actual: 2

bin/rails test test/unit/changeset_test.rb:39
```

このチケットでやっていることは単純ですが、中でも、テストデータが不足しているため、内部的にバリデーションに失敗し、結果テストが失敗するケースは厄介でした。しかも、バリデーションの失敗を握りつぶしているので、特定がとても難しい。

```
From: /redmine/app/models/changeset.rb @ line 260 :

    255: Redmine::Hook.call_hook(:model_changeset_scan_commit_for_issue_ids_pre_issue_update,
    256:                             {:changeset => self, :issue => issue, :action => action})
    257:
    258:     if issue.changes.any?
    259:       unless issue.save
 => 260:         binding.irb
    261:         logger.warn("Issue ##{issue.id} could not be saved by changeset #{id}: #{issue.errors.full_messages}") if logger
    262:       end
    263:     else
    264:       issue.clear_journal
    265:     end

irb(#<Changeset:0x000073145f1036a0>):001> issue.id
=> 2
irb(#<Changeset:0x000073145f1036a0>):002> issue.errors.full_messages
=> ["Target version is not included in the list"]
```

## 常に全ての fixture をロードするように変更

https://www.redmine.org/issues/41961

少し前までの Redmine のテストでは、テストファイルごとに使用するテストデータを宣言していました。こんな感じ。

```ruby
class UserTest < ActiveSupport::TestCase
  fixtures :users, :email_addresses, :members, :projects, :roles, :member_roles, :auth_sources, (snip)
```

この fixtures の宣言が不足していると、ランダムで失敗します。なぜランダムなのか。
詳細は別の記事に書いたので、 詳細は [Fixture はどのようにデータをロードしているか](https://hidakatsuya.dev/2024/12/09/how-the-fixtures-work.html) をご覧ください。

例えば、以下の２つのテストを用意します。

```ruby
# test/unit/a_test.rb
class ATest < ActiveSupport::TestCase
  fixtures :users

  test "A test" do
    puts "-- A test"
    assert User.exists?(1)
  end
end

# test/unit/b_test.rb
class BTest < ActiveSupport::TestCase
  test "B test" do
    puts "-- B test"
    assert User.exists?(1)
  end
end
```

このとき、`BTest` の結果は、テストの実行順序によって変わります。

* A test -> B test: BTest は成功する
* B test -> A test: BTest は失敗する

これが、ランダムで失敗する原因です。テストの数だけ、テストの順序の組み合わせがあるため、テストを直しても直しても減らないというわけです。

これを解決するために、テストファイルごとに必要な fixtures を宣言することをやめて、 全てのテストで `fixtures :all` を宣言し、
常に全てのテストデータをロードするように変更しました。

結果、今の所、fixtures の不足によるテストの失敗は観測していません。

また、これによって、テストに必要な fixtures を調べて宣言する必要もなくなりました。それ自体、困難な作業でした。

## 最後に

`fixtures :all` によって安定したおかげで、テストの並列化が導入しやすくなりました。
Rails 標準の `pararellaize` を使うことで、GitHub CI で 10分から5分程度に短縮できることも検証済みです。

システムテストの導入と合わせて、今後も CI の改善に取り組みたいと思います。

## CI・テスト関連のチケット（おまけ）

* [#41096](https://www.redmine.org/issues/41096) "##" syntax auto complete does not work
* [#41961](https://www.redmine.org/issues/41961) Use `fixtures :all` to ensure consistent test data and improve test reliability
* [#41969](https://www.redmine.org/issues/41969) Add SQLite3 tests to CI
* [#41931](https://www.redmine.org/issues/41931) Fix random failures in IssueRelationTest#test_create_with_initialized_journals due to ambiguous conditions for retrieving the expected detail
* [#41902](https://www.redmine.org/issues/41902) Fix class name to match file name in keyboard_shortcuts_test.rb
* [#41901](https://www.redmine.org/issues/41901) Fix random test failure in DestroyProjectsJobTest due to unsorted `@projects`
* [#41623](https://www.redmine.org/issues/41623) Fix tests that randomly failed due to required fixtures not being loaded
* [#41238](https://www.redmine.org/issues/41238) Fix test failure in IssuesSystemTest due to incorrect attachment count expectation
