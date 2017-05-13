<p class="title-main">RESTful API やってみよう</p>

Python ~ Flask ~ Django ~ Swagger

<img src="https://www.python.org/static/opengraph-icon-200x200.png" height="90px">
<img src="http://flask.pocoo.org/docs/0.12/_static/flask.png" height="80px">
<img src="https://www.fullstackpython.com/img/logos/django.png" height="80px">
<img src="https://avatars2.githubusercontent.com/u/7658037?v=3&s=400" height="80px" style="margin-left: 5px;">

---

<p class="title-sub">Index</p>

* RESTful APIについて
* 環境準備
* Pythonについて
* Flask
* Django

---

<p class="title-sub">RESTful API ？</p>

* 一意のURIとHTTPメソッドで情報が表現できる
* ステートレス
* リソース指向

曖昧..

参考になるいい記事↓

<a href="http://qiita.com/mserizawa/items/b833e407d89abd21ee72" target="_blank">翻訳: WebAPI 設計のベストプラクティス</a>

---

<p class="title-sub">どの言語がいいの？</p>

どの言語でも書ける

個人的には性能や言語特性からGoが向いてると思う  
日本語ドキュメントが充実してるのはRoRとかかな

今回はPython  
理由は僕が直近の案件で書いてたから..

---

<p class="title-sub">環境準備</p>

```sh
$ docker run -p 8000:8000 -v `pwd`:/home -it python /bin/bash
root@ContainerId:/ python -V
Python 3.6.1
```

---

<p class="title-sub">Pythonについて少し触れとく</p>

* 2系と3系がある(これからは3系)
* 学習コストが低い
* オブジェクト指向
* スクリプト言語(v3.5以上では静的型チェックがある)
* インデント構文
* 最近は機械学習やデータサイエンスで人気
* Webもいける
* pip：パッケージインストーラー
* pyenv：バージョンマネジメント

---

<p class="title-sub">PythonでHello Worldしとく</p>

```sh
$ python
>>> print('Hello World!')
>>> exit()
```

---

<p class="title-sub">PythonならWebサーバがワンライナー</p>

```sh
$ python3 -m http.server 8000 # default port is 8000
```

---

<p class="title-sub">軽量なフレームワークFlask</p>

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8000)
```
```sh
$ pip install Flask
$ python app.py # localhost:8000
```

---

<p class="title-sub">FlaskでREST API</p>

```python
# app2.py
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)
```
```sh
$ pip install flask-restful
$ python app2.py # curl localhost:8000 | jq
```

---

<p class="title-sub">FlaskでREST API with Swagger</p>

```python
# app3.py
from flask import Flask
from flask_restplus import Resource, Api

app = Flask(__name__)
api = Api(app)

@api.route('/hello')
class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)
```
```sh
$ pip install flask-restplus
$ python app3.py # localhost:8000
```

---

<p class="title-sub">Swagger ？</p>

* RESTful APIのインターフェイスを記述するための標準フォーマット
* JSONからドキュメントやコードの生成、コードからドキュメントの生成ができたりする
* Swagger UI：RESTクライアント
* Swagger Editor：ドキュメント生成ツール
* Swagger Codegen：SDK生成ツール

---

<p class="title-sub">フルスタックなフレームワークDjangoでREST API with Swagger</p>

① setup

```sh
$ pip install django djangorestframework django-rest-swagger
$ django-admin startproject firstProject
$ cd firstProject
```

↓

+++

② create resource and model

```sh
$ python manage.py startapp app
```

```python
# app/models.py
from django.db import models

class Member(models.Model):
    name = models.CharField(max_length=16)
    mail = models.EmailField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

↓

+++

③ serialize

```python
# app/serializer.py
from rest_framework import serializers
from .models import Member

class MemberSerializer(serializers.ModelSerializer):
    class Meta:
        model = Member
        fields = ('name', 'mail', 'created_at', 'updated_at')
```

↓

+++

④ viewset

```python
# app/views.py
from rest_framework import viewsets
from .models import Member
from .serializer import MemberSerializer

class MemberViewSet(viewsets.ModelViewSet):
    queryset = Member.objects.all()
    serializer_class = MemberSerializer
```

↓

+++

⑤ routing

```python
# app/urls.py
from rest_framework import routers
from .views import MemberViewSet

router = routers.DefaultRouter()
router.register(r'members', MemberViewSet)
```

```python
# firstProject/urls.py
from django.conf.urls import url, include
from django.contrib import admin
from app.urls import router
from rest_framework_swagger.views import get_swagger_view

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api/', include(router.urls)),
    url(r'^$', get_swagger_view(title='API Document')),
]
```

↓

+++

⑥ settings

```python
# firstProject/settings.py
INSTALLED_APPS = [
    # add
    'app',
    'rest_framework',
    'rest_framework_swagger',
]

REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 10
}
```

↓

+++

⑦ migrate and run

```sh
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py runserver 0:8000 # localhost:8000
```

---

<p class="title-sub">Fin</p>
