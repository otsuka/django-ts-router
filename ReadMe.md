django-ts-router
=====================

## 概要

Django の URLconf モジュール(urls.py)に定義した URL パスを名前から逆引きできる TypeScript コードを生成します。

```py
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
]
```

URLconf モジュールに上記の URL パターンが定義されていた場合に、django-ts-router が生成した TypeScript コードを使うと
index ビューと detail ビューを示す URL を次のように取得できます。([[生成されたコードの例|Example of Generated Code]])<br>
このように Django の [reverse()](https://docs.djangoproject.com/en/1.8/ref/urlresolvers/#reverse) 関数と似た機能を TypeScript で提供します。

```ts
var indexUrl: stirng = com.example.router.reverse("index");
console.log(indexUrl); // "/"

var detailUrl: stirng = com.example.router.reverse("detail", {question_id: "123"});
console.log(detailUrl); // "/123/"
```

しかし、これでは TypeScript の型チェックを有効に活用できていないので、django-ts-router ではそれぞれの URL を取得する専用の関数も生成します。<br>
これらの関数を使うと上に書いた URL 取得コードは以下のように書き換えることができます。

```ts
var indexUrl: stirng = com.example.router.getIndexURL();
console.log(indexUrl); // "/"

var detailUrl: stirng = com.example.router.getDetailURL("123");
console.log(detailUrl); // "/123/"
```

tsc([TypeScript Compiler](http://www.typescriptlang.org/)) がインストールされている環境では、TypeScript をトランスパイルする形で直接 JavaScript のコードを生成することも可能です。


## システム要件

* Python (3.4)
* Django (1.8)

## インストール

`pip` を用いてインストールします。

    pip install django-ts-router

`INSTALLED_APPS` 設定に `'django_ts_router'` を追加します。

    INSTALLED_APPS = (
        ...
        'django_ts_router',
    )

## 設定

settings.py に `TS_ROUTER` 設定を追加します。

    TS_ROUTER = {
        'NAMES' = [
            'index',
            'detail',
        ],
        'MODULE' = 'com.example.router',
        'TSC': '/usr/local/bin/tsc',
    }

##### NAMES (必須)

出力する URL パターンの名前を指定したリスト。一つ以上の名前を指定する必要があります。

###### ネームスペース
ネームスペースを指定する場合は `api:get_entry` というように `namespace:` を名前の頭に付けます。<br>
`api:*` のようにネームスペースの後にアスタリスクを指定すると、そのネームスペースに属するすべての名前の URL が出力対象となります。

##### MODULE (オプション)

生成される TypeScript コードが属するモジュール名を指定します。<br>
指定がない場合、`django.tsrouter` が使用されます。

##### TSC (オプション)

直接 JavaScript コードを生成する場合に tsc のパスを指定します。


## TypeScript コードの生成

manage.py を通して利用できる `export_ts_router` コマンドで TypeScript コードを生成、出力します。

```sh
./manage.py export_ts_router --out router.ts
```

`--out` オプションで指定したファイルに出力します。`--out` オプションを指定しない場合は標準出力に出力されます。

#### JavaScript コードを生成する

```sh
./manage.py export_ts_router --javascript --out router.js
```

`--javascript` オプションを付けると、生成された TypeScript コードを tsc を通して JavaScript コードに変換して出力します。


## 制限

##### 名前付きグループ


名前付きじゃないグループ(positional group)を使った URL パターンは出力対象として利用できません。

```py
# サポートされていません!!
urlpatterns = [
    url(r'^articles/([0-9]{4})/$', views.year_archive, name='news-year-archive'),
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive, name='news-month-archive'),
    ...
```

以下のように名前付きの正規表現グループのみが出力対象として利用できます。

```py
urlpatterns = [
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive, name='news-year-archive'),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive, name='news-month-archive'),
    ...
```

##### ネストされたグループ

正規表現のグループがネストされた URL パターンは利用できません(これは Django の `reverse()` 関数でもサポートされていません)。

```py
# サポートされていません!!
urlpatterns = [
    url(r'^articles/(?P<year>[0-9]{4})(/(?P<month>[0-9]{2}))?$', views.archive, name='archive')
    ...
```