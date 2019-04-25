# Django Tutorial

https://docs.djangoproject.com/ja/1.11/intro/


## プロジェクトを作成する

``` shell
$ mkdir django_tutorial
$ cd django_tutorial
$ python3 -m venv myvenv	# myvenv という名前で仮想環境を作成
$ source myvenv/bin/activate	# 仮想環境に入る
(myvenv) $ pip install django==1.11	# 仮想環境内で Django インストール
(myvenv) $ django-admin startproject project .
```

仮想環境をつくり、その中で Django をインストールし、プロジェクトをセットアップすることで、
システム全体で複数のバージョンの Django 環境を作成することが可能になる。
ここまでで `project` というディレクトリが作成される。カレントディレクトリの構成は以下のようになる。

``` shell
(myvenv) $ tree
.
├── README.md
├── manage.py
├── myvenv
│   ├── bin
│   │   ├── __pycache__
│   │   │   └── django-admin.cpython-36.pyc
│   │   ├── activate
........
└── project
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

`manage.py` はプロジェクトに対する様々な操作をするためのユーティリティ。`project` パッケージに
Django プロジェクトの設定ファイルである `settings.py` モジュール、プロジェクトの URL 設定の起点となる `urls.py` モジュール、そして WSGI 互換 Web サーバのエントリポイントとなる `wsgi.py` モジュールがある。


## Django の起動

ここまでで Web アプリケーションとしての Django を起動できる。

``` shell
(myvenv) $ python manage.py runserver
```

こうすると、localhost の 8000 番ポートを listen する 簡易 Web サーバがスタートし、その中で Django を初期化し、後述の URLConf や View の定義に
したがい、リクエストの URL パターンに合う適切な View 関数を呼び出して、レスポンスを返す。ここではまだ URLConf の定義をしていないので
単に挨拶が返されるだけだ。ローカルからのアクセスだけでなく、他のホストからのアクセスも受け付けたいなら、

```shell
(myvenv) $ python manage.py runserver 0.0.0.0:8080
```

のようにする。上ではデフォルトの 8000 ポートではなく 8080 を指定した。

Django に付属の簡易 Web サーバ以外でも Django アプリを実行するためには、それら Web サーバが Django に対応していなくてはならないが、それだと
使える Web サーバが限られてくるので、Web サーバと、Django のような Web アプリケーションとの共通インターフェースを Python で定義したのが
`WSGI(Web Server Gateway Interface)` である。Django は WSGI をサポートしているので、WSGI をサポートする Web サーバ（WSGI サーバ or
WSGI アプリケーションコンテナ）なら Django を使うことができる。Python 製の WSGI サーバが `gunicorn` であり、これを使うなら、

```shell
(myvenv) $ pip install gunicorn
(myvenv) $ gunicorn project.wsgi
```

とすればいい。デフォルトで 8000 番ポートが使われる。まず `gnicorn` をインストールし、`project.wsgi` モジュールをエントリポイントとして指定している。WSGI サーバである gunicorn は
`project.wsgi` モジュールを使って Django アプリケーションを WSGI アプリケーションに変換し、それにより、WSGI に則って Django と通信する。

gunicorn 以外にも `uWSGI` 等がある。

`Apache` や `Nginx` 等の WSGI サーバでない Web サーバと Django アプリケーションを連携させる場合、`gunicorn` や `uWSGI` と連携させることになる。

[Django をデプロイする](https://docs.djangoproject.com/ja/1.11/howto/deployment/)


Web サーバを使わずに Django アプリケーションを実行することもできる。次のようにする。

```shell
(myvenv) $ python manage.py shell
```

[API で遊んでみる](https://docs.djangoproject.com/ja/1.11/intro/tutorial02/#playing-with-the-api)


こうすると、シェルが起動し、その中で Django アプリケーションを試すことができる。

どういう形であれ、Django アプリケーションを実行するには、設定モジュールを読み込んで初期化しなければならない。この初期化モジュールはデフォルトでは、`project.settings` モジュールである。これは `manage.py` や `wsgi.py` の中身を見ればわかる。設定モジュールを切り替えるには、`DJANGO_SETTINGS_MODULE` 環境変数に切り替え先の設定モジュールをセットする。または、`manage.py` の場合なら、`--settings` オプションで次のように指定する。

```shell
(myvenv) $ python manage.py shell --settings=project.debug_settings
```

[Django のエントリポイントとアプリケーションの仕組み](https://www.slideshare.net/tokibito/django-52696489)


## アプリケーションの作成

次のコマンドで polls というアプリケーションを作る。

```shell
(myvenv) $ python manage.py startapp polls
(myvenv) $ tree
.
├── README.md
├── manage.py
├── myvenv
│   ├── bin
│   │   ├── __pycache__
│   │   │   └── django-admin.cpython-36.pyc
│   │   ├── activate
.....
.....
├── polls
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
└── project
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

作成したアプリケーションは、`polls` という Python パッケージになる。ひとつのプロジェクト内に複数のアプリケーションを作ることもできるし、別のプロジェクトへアプリケーションをもっていくこともできる。

以下、`polls` パッケージ中にモジュールを作成することにより、polls アプリケーションを作成していく。


## Database の設定と Model の作成

`project.settings` モジュールには、データベースの設定が `DATABASES = {...}` の項目名で記述されている。デフォルトでは SQLite を使用するようになっているが、他のデータベースを使うなら、それらをセットアップし、それを利用するための設定を記述する。

Django プロジェクトには、自分で作成した `polls` アプリケーションの他にも、管理サイトや認証システム、セッションフレームワーク等、共通して利用されるアプリケーションがデフォルトで含まれている。これら、プロジェクトで利用されるアプリケーションは、`project.settings` モジュールの `INSTALLED_APPS = {...}` に記述されている。

これら共通して利用されるアプリケーションはデータベースを使うので、テーブル等のスキーマ設定をデータベースに反映させる必要がある。

```shell
(myvenv) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
```

こうすると、`INSTALLED_APPS` に記述されているアプリケーションのスキーマ定義のみがデータベースに反映される。

ここで `project.settings` モジュールの `LANGUAGE_CODE` と `TIME_ZONE` を自分の言語とタイムゾーンに設定する。Django は日時情報をデータベースに保存するとき、UTC を使う。template や form、サードパーティの Django Rest Framework の serializer 等において、`TIME_ZONE` に指定したタイムゾーンに変換する。


`polls` アプリケーションのためのデータベース定義は、`polls.models` モジュール内に記述する。

[モデル](https://docs.djangoproject.com/ja/1.11/ref/models/)

[モデルフィールドリファレンス](https://docs.djangoproject.com/ja/1.11/ref/models/fields/)

Model の Field に指定するオプションは、データベースレベルで機能するものもあれば、Model レベル、あるいは、フォームや Django Rest Framework の serializer の Validation レベルで機能するものもある。

モデルを定義したら、それをデータベースへマイグレーションする必要がある。そのためには、`polls` アプリケーションをプロジェクトに登録しなければならない。そのために `project.settings` で

```python
INSTALLED_APPS = [
	'polls.apps.PollsConfig',
	......
]
```

のように `polls` アプリケーションを登録する。その後で

```shell
(myvenv) $ python manage.py makemigrations polls
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Choice
    - Create model Question
    - Add field question to choice
```

とする。`polls.migrations` パッケージに `0001_initial` というモジュールができ、そこにマイグレーション情報が出力される。admin 等の共通アプリケーションも同様に `admin.migrations` パッケージ内にマイグレーション情報が定義されたモジュールがあり、以前の `migrate` コマンドではそれらがデータベースに反映されたわけだ。ここでは、`polls` アプリケーションのマイグレーション情報が更新されたので、再度次のように `migrate` コマンドを発行する。

```shell
(myvenv) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

アプリケーションのモデルを更新する度に、`makemigrations` コマンドと `migrate` コマンドを実行する。


## Django Admin の利用

共通アプリケーションの admin は、データベース管理用のサイトを提供する。まず、admin サイトにログインできる管理者ユーザを作成する。

```shell
(myvenv) $ python manage.py createsuperuser
```

その後、サーバを起動してブラウザで `/admin` にアクセスすれば管理サイトに行ける。この URL は `project.urls` モジュールで定義されているものだ。

`polls` アプリで作成した `Question`, `Choice` モジュールに admin サイトでアクセスできるようにするには、`polls.admin` モジュールへ次のようにそれらを登録しなければいけない。

```python
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```


管理者のパスワードを忘れてしまった場合、次のようにして管理者パスワードを変更してやればいい。
管理者のユーザ名を admin とする。

```shell
(myvenv) $ python manage.py changepassword admin
```


## View の定義

View は、`HttpRequest` オブジェクトを処理して、`HttpResponse` オブジェクトを返すか、`Http404` のような例外をスローする。これらを `polls.views` モジュール中に定義する。

View には２種類ある。Function ベースの View と Class ベースの View だ。
Function ベースの View は Python の関数として定義される。Class ベースの
View は Python のクラスであり、これは主に汎用 View として使われる。DRF でも全く同様なので、より深く突っ込むのはそっちでやろう。

[汎用ビューを使う](https://docs.djangoproject.com/ja/1.11/intro/tutorial04/#use-generic-views-less-code-is-better)


View を定義するとき、サポートする URL パターンと一緒にどういう View を作成するかを決め、実装する。その後で、View と URL の対応を URLconf に定義するだろう。

View から例外がスローされたとき、`project.settings` の `DEBUG` が `True` なら URLConf 情報等のデバッグ情報がクライアントに返される。`False` ならば、例外に応じた Django 標準の簡単なエラーページが返されるが、`templates` の直下に `404.html` や `500.html` 等のテンプレートファイルを置いておくと、それが使用される。

https://docs.djangoproject.com/ja/1.11/topics/http/views/#returning-errors
https://docs.djangoproject.com/ja/1.11/topics/http/views/#the-http404-exception
https://docs.djangoproject.com/ja/1.11/topics/http/views/#customizing-error-views
https://docs.djangoproject.com/ja/1.11/ref/views/#error-views



## Template の定義

`project.settings` モジュールの `TEMPLATES` 項目にテンプレートのロード、レンダリングの方法が定義されている。デフォルトでは、`INSTALLED_APPS` に定義されているアプリケーションの `templates` ディレクトリを順に検索していき、最初に見つかったテンプレートを使用する（**'APP_DIRS': True**）。

そこで、`polls` パッケージ配下に次のように３つの template を作成する。

```shell
(myvenv) $ tree polls
polls
├── __init__.py
.......
├── templates
│   └── polls
│       ├── detail.html
│       ├── index.html
│       └── results.html
.....
```

そして、View の中ではたとえば次のようにテンプレートを指定する。

```python: polls/views.py
from django.template import loader

template = loader.get_template('polls/index.html')
```

`templates` ディレクトリの下に直接テンプレートファイルを置かずに名前空間を polls というアプリケーションと同じ名前で与えている。


`project.settings` モジュールの `TEMPLATES` 項目の `DIRS` にテンプレートを検索するディレクトリのリストを指定できる。たとえば、次のようにすると、

```python
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```

プロジェクトルート直下の `templates` ディレクトリを検索対象にする。複数のアプリケーションのテンプレートから継承されたりインクルードされるテンプレートはここに置いておくとよい。



## URLconf の定義

View と URL パターンの対応を URLConf で定義する。`project.urls` モジュールがトップレベルの定義であり、`polls.urls` モジュールに `polls` アプリケーション内での定義を記述する。

```python: project/urls.py
# project/urls.py
urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
```

`polls` 以外にもアプリケーションがあれば、それらの URLconf の定義もここに include する。

```python: polls/urls.py
# polls/urls.py
app_name = 'polls'
urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),

    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$',
        views.results, name='results'),

    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```
ここでは、`app_name = polls` で URLconf に名前を与え、各々の URL パターンにも `name=` で名前を与えている。これにより、View や template 内で URL パターンを参照するときに URL をハードコーディングしなくて済むようになる。

```python: polls/views.py
# polls/views.py
reverse('polls:results', args=(question.id,))
```

```html: detail.html
<!-- detail.html -->
<form action="{% url 'polls:vote' question.id %}" method="post">
```



## Form の処理

HTML の Form を直接、自前で処理することもできるが面倒である。そのため Django には
`django.forms.Form` クラスが用意されている。これは、Model における `django.db.models.Model` クラスに相当する。
Model と同じように `django.forms.Form` クラスを継承し、`django.forms.fields.Field` クラスのサブクラスとして提供されている
`CharField` や `IntegerField` 型のクラス変数を定義する。これらのクラス変数が HTML FORM 中の各フィールドに相当し、
変数名がフィールドの name 属性と対応する。

HTML フォームを `django.forms.Form` のオブジェクトとして処理することにより、型変換や妥当性検証やエラーメッセージの
保存などを Form オブエジェクト内にカプセル化することができる。

[フォームを使う](https://docs.djangoproject.com/ja/1.11/topics/forms/)


POST Form を使う場合、`Cross Site Request Forgery` を防ぐために、自サイト内を URL に指定した Post フォームには
`{%csrf_token %}` テンプレートタグを使う。POST 以外でも PUT や DELETE メソッドなど、unsafe な HTTP リクエストには
CSRF トークンがついてくることが必要であり、そうでないと 403 Forbidden エラーとなる。

[クロスサイトリクエストフォージェリ（CSRF）対策](https://docs.djangoproject.com/ja/1.11/ref/csrf/)



## CSRF の実験

`{%csrf_token %}` テンプレートタグを指定したフォームのページを `curl` コマンドでリクエストすると次のようになる。

```shell
$ curl -v "http://localhost:8000/polls/1/"
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 8000 failed: Connection refused
*   Trying fe80::1...
* TCP_NODELAY set
* Connection failed
* connect to fe80::1 port 8000 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET /polls/1/ HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.54.0
> Accept: */*
> 
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Date: Fri, 12 Oct 2018 08:59:11 GMT
< Server: WSGIServer/0.2 CPython/3.6.1
< Content-Type: text/html; charset=utf-8
< X-Frame-Options: SAMEORIGIN
< Vary: Cookie
< Content-Length: 486
< Set-Cookie:  csrftoken=MKUH7A4hVbB7623WptyZtrgT6NaldvVP7WsNApRr5joTfgbHOETyVauD04jwZrtK; expires=Fri, 11-Oct-2019 08:59:11 GMT; Max-Age=31449600; Path=/
< 
<h1>東京オリンピックに賛成？</h1>

<form action="/polls/1/vote/" method="post">
<input type='hidden' name='csrfmiddlewaretoken' value='3asxNHGbsUsMbGQq5BZiU8zaEHlmCntdom0DgwtlC2fykUYbuMkRmRNUyYuxoj18' />

    <input type="radio" name="choice" id="choice1" value="1" />
    <label for="choice1">賛成</label><br />

    <input type="radio" name="choice" id="choice2" value="2" />
    <label for="choice2">反対</label><br />

<input type="submit" value="Vote" />
</form>
* Closing connection 0
```

レスポンスの `Set-Cookie` ヘッダで `csrftoken` の値を指定し、FORM には `csrfmiddlewaretoken` の値が hidden フィールドとして
セットされている。これにより、FORM をサブミットすると、次の `curl` コマンドで実行した場合と同様のリクエストが送信される。

```shell
$ curl -X POST "http://localhost:8000/polls/1/vote/" \
>     -H "Content-Type: application/x-www-form-urlencoded" \
>     -H "Cookie: csrftoken=MKUH7A4hVbB7623WptyZtrgT6NaldvVP7WsNApRr5joTfgbHOETyVauD04jwZrtK" \
>     -d "csrfmiddlewaretoken=3asxNHGbsUsMbGQq5BZiU8zaEHlmCntdom0DgwtlC2fykUYbuMkRmRNUyYuxoj18&choice=2"
```

`csrftoken` クッキー、または `csrfmiddlewaretoken` フィールドがなかったり、間違っている場合には 403 Forbidden エラーが返る。


CSRF チェックを無効にしたい場合、CSRF チェックを必要とする unsafe な HTTP メソッドを処理する View 関数に
`django.views.decorators.csrf.csrf_exempt` をデコレータとしてつけてやればよい。そうすると、CSRF トークンをリクエストに
つけなくても成功するようになる。



## Django Rest Framework での CSRF

DRF の管理下にある HTTP メソッドの場合、通常の Django の CSRF チェックとは少々異なっている。DRF 管理下では
CSRF チェックは次のようにおこなわれる。

- GET 等の safe メソッドならば、通常の Django 同様に CSRF チェックはおこなわれない。
- unsafe メソッドでも、認証をかけていないリクエストであるなら、CSRF チェックはなされない。
- unsafe メソッド、かつ認証をかけていないリクエストでも、ログイン済みのリクエストである場合、すなわち、Cokkie ヘッダに有効な `sessionid` が入ってしまっていると
  CSRF チェックが働く。
- unsafe メソッドで認証をかけているリクエストの場合、`SessionAuthentication` ならば CSRF チェックが働くが、`TokenAuthentication` ならば CSRF チェックは
 働かない。


[Authentication and CSRF Protection in Django Rest Framework](http://kylebebak.github.io/post/django-rest-framework-auth-csrf)




## 自動テストのやり方

```shell
(myvenv) $ python manage.py shell
```

とシェル上で Django を起動して手動でテストすることもできるが、効率が悪いので、普通は自動テストを使う。Python には標準で `unittest` という自動テスト用のモジュールが含まれているが、Django で自動テストする場合、`unittest.TestCase` のサブクラスをもつ `django.test` モジュールを使用する。

https://docs.djangoproject.com/ja/1.11/intro/tutorial05/
[Django におけるテスト](https://docs.djangoproject.com/ja/1.11/topics/testing/)

自動テストは、`polls.tests` モジュール内の `django.test.TestCase` のサブクラスに `test` ではじまるメソッドとして記述する。各 test メソッドで共通の初期化処理は、

```python
def setUp(self):
	.....
```

に定義し、`TestCase` 全体で１回だけ実行する初期化処理は、

```python
@classmethod
def setUpClass(cls):
	super().setUpClass()
	.....
```

に定義する。`polls` アプリケーションをテストするには次のようにする。

```shell
(myvenv) $ python manage.py test polls
(myvenv) $ python manage.py test polls.tests.QuestionDetailViewTests
(myvenv) $ python manage.py test polls.tests.QuestionDetailViewTests.test_past_question
```



## 静的ファイルのデプロイ

静的ファイル、すなわち、CSS や JavaScript、画像ファイルの管理方法は開発時と運用時とでは異なる。

### 開発時

アプリケーション、ここでは `polls` パッケージ下に `static` ディレクトリを作成し、その下にさらに `polls` ディレクトリを作って、そこに `polls` アプリケーションで使用する静的ファイルを配置する。たとえば次のようになる。

```shell
(myvenv) $ tree polls
polls/
├── __init__.py
.........
├── static
│   └── polls
│       ├── images
│       │   └── background.png
│       └── style.css
```

さらに必要であれば、`project.settings` モジュールの `STATICFILES_DIRS` に、たとえば、次のようにフルパスのディレクトリを登録する。

```python
STATICFILES_DIRS = [
	'/opt/webfiles/common',
	'/var/www',
]
```

このようにした上で、サーバへのリクエストのパスが `project.settings` モジュールの `STATIC_URL` に指定されたディレクトリで始まっていた場合、各アプリケーションの `static` ディレククトリ、`STATICFILES_DIRS` に指定されたディレクトリを検索していき、最初に見つかったファイルがクライアントへ返される。たとえば、

```python
STATIC_URL = '/foo/'
```

で、URL が `http://localhost/foo/polls/style.css` だった場合、

- admin/static/polls/style.css
- polls/static/polls/style.css
- /opt/webfiles/common/polls/style.css
- /var/www/polls/style.css

が検索される。admin は、Admin サイトのアプリケーションであり、この他にも `INSTALLED_APPS` に登録されているアプリケーションについても検索される。ここでは、`polls/static/polls/syle.css` がヒットするだろう。

`polls/static/` 下に直接ファイルを配置せず、`polls/static/polls/` 下に配置するのは他のアプリケーションと名前空間を分けるためだ。


この静的ファイルの検索は、`project.settings` モジュールの `INSTALLED_APPS` に登録されている `django.contrib.staticfiles` アプリケーションにより実現されている。これが有効なのは、`project.settings` モジュールで `DEBUG = True` かつ、サーバを manage.py の runserver コマンドで実行している場合だけ。したがって、gunicorn で起動している場合には効かない。

なお、template 内に静的ファイルへのパスを書くとき、`/foo/polls/style.css` とハードコーディングするのではなく、`{% static %}` テンプレートタグを使って次のように書く。

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />
```

このテンプレートタグの機能は、常に有効であり、次の運用時にも使える。


### 運用時

`django.contrib.staticfiles` アプリケーションを使った静的ファイルの検索は非効率であり、セキュリティ的にも問題なので、運用時には使えない。運用時には、静的ファイルに変化があるたびに次のコマンドを実行する。

```shell
(myvenv) $ python manage.py collectstatic
```

こうすると、各アプリケーションの `static` ディレクトリ下のファイル、および、`STATICFILES_DIRS` に登録されているディレクトリ下のファイルが `project.settings` モジュールの `STATIC_ROOT` に指定されたディレクトリの下にコピーされる。後は、このディレクトリを運用時にアクセスできる場所へデプロイし、`{% static %}` がそこを指し示すように `STATIC_URL` を適切に設定してやればいい。




































