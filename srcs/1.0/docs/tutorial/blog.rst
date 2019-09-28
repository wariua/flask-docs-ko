.. currentmodule:: flask

블로그 청사진
=============

인증 청사진을 작성할 때 배운 기법들을 블로그 청사진을 작성하는
데 그대로 쓰게 된다. 블로그에서는 전체 글 목록을 보여 줘야 하고,
사용자가 로그인 해서 글을 쓸 수 있어야 하고, 글 작성자가
편집이나 삭제를 할 수 있어야 한다.

뷰를 차례로 작성하는 동안 개발 서버를 계속 돌리고 있자. 그리고
변경 사항을 저장할 때마다 브라우저에서 그 URL로 가서 확인을
해 보자.

청사진
------

청사진을 정의하고 응용 팩토리에서 등록하자.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    from flask import (
        Blueprint, flash, g, redirect, render_template, request, url_for
    )
    from werkzeug.exceptions import abort

    from flaskr.auth import login_required
    from flaskr.db import get_db

    bp = Blueprint('blog', __name__)

팩토리에서 청사진을 임포트 해서 :meth:`app.register_blueprint()
<Flask.register_blueprint>`\로 등록하자. 팩토리 함수 끝의
app 반환 앞에 새 코드를 집어넣는다.

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    def create_app():
        app = ...
        # 기존 코드 생략

        from . import blog
        app.register_blueprint(blog.bp)
        app.add_url_rule('/', endpoint='index')

        return app


인증 청사진과 달리 블로그 청사진에는 ``url_prefix``\가 없다.
그래서 ``index`` 뷰가 ``/``\에 있게 되고 ``create`` 뷰가
``/create``\에 있게 된다. 블로그가 Flaskr의 핵심 기능이므로
블로그 인덱스가 최상위 인덱스가 되는 게 맞다.

아래 정의할 ``index`` 뷰의 종점은 ``blog.index``\가 된다.
그런데 인증 뷰 일부에서는 그냥 ``index`` 종점을 참조했다.
:meth:`app.add_url_rule() <Flask.add_url_rule>`\은 종점
이름 ``'index'``\를 URL ``/``\에 연계해서
``url_for('index')``\와 ``url_for('blog.index')`` 어느 쪽도
쓸 수 있게 한다. 어떻게 해도 같은 URL ``/``\를 만들어 낸다.

다은 응용에서는 응용 팩토리에서 블로그 청사진에
``url_prefix``\를 줘서 ``hello`` 뷰와 비슷하게 따로 ``index``
뷰를 정의할 수도 있을 것이다. 그러면 ``index``\와
``blog.index``\의 종점과 URL이 서로 다르게 될 것이다.


인덱스
------

인덱스에서는 모든 글을 최근 것부터 보여 주게 된다. ``JOIN``\을
사용해서 결과에 ``user`` 테이블의 작성자 정보를 넣는다.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/')
    def index():
        db = get_db()
        posts = db.execute(
            'SELECT p.id, title, body, created, author_id, username'
            ' FROM post p JOIN user u ON p.author_id = u.id'
            ' ORDER BY created DESC'
        ).fetchall()
        return render_template('blog/index.html', posts=posts)

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/index.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Posts{% endblock %}</h1>
      {% if g.user %}
        <a class="action" href="{{ url_for('blog.create') }}">New</a>
      {% endif %}
    {% endblock %}

    {% block content %}
      {% for post in posts %}
        <article class="post">
          <header>
            <div>
              <h1>{{ post['title'] }}</h1>
              <div class="about">by {{ post['username'] }} on {{ post['created'].strftime('%Y-%m-%d') }}</div>
            </div>
            {% if g.user['id'] == post['author_id'] %}
              <a class="action" href="{{ url_for('blog.update', id=post['id']) }}">Edit</a>
            {% endif %}
          </header>
          <p class="body">{{ post['body'] }}</p>
        </article>
        {% if not loop.last %}
          <hr>
        {% endif %}
      {% endfor %}
    {% endblock %}

사용자가 로그인 돼 있으면 ``header`` 블록에 ``create`` 뷰로 가는
링크가 추가된다. 그 사용자가 글의 작성자이면 그 글에 대한
``update`` 뷰로 가는 "Edit" 링크를 보게 된다. ``loop.last``\는
`Jinja for 루프`_ 내에서 쓸 수 있는 특수 변수이다. 이를 이용해
마지막을 빼고 게시 글 아래마다 선을 넣어서 글들을 시각적으로
분리한다.

.. _Jinja for 루프: http://jinja.pocoo.org/docs/templates/#for


생성
----

``create`` 뷰는 인증의 ``register`` 뷰와 동일하게 동작한다. 즉
양식을 표시하거나, 아니면 제출된 데이터를 검증하고 데이터베이스에
글을 추가하든지 오류를 보이든지 한다.

앞서 작성했던 ``login_required`` 데코레이터를 블로그 뷰에
사용한다. 사용자가 이 뷰들에 방문하려면 로그인 상태여야 하며
아니면 로그인 페이지로 재지향 된다.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/create', methods=('GET', 'POST'))
    @login_required
    def create():
        if request.method == 'POST':
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if error is not None:
                flash(error)
            else:
                db = get_db()
                db.execute(
                    'INSERT INTO post (title, body, author_id)'
                    ' VALUES (?, ?, ?)',
                    (title, body, g.user['id'])
                )
                db.commit()
                return redirect(url_for('blog.index'))

        return render_template('blog/create.html')

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/create.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}New Post{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="title">Title</label>
        <input name="title" id="title" value="{{ request.form['title'] }}" required>
        <label for="body">Body</label>
        <textarea name="body" id="body">{{ request.form['body'] }}</textarea>
        <input type="submit" value="Save">
      </form>
    {% endblock %}


갱신
----

``update`` 뷰와 ``delete`` 뷰 모두 ``id``\로 ``post``\를 가져와서
작성자가 로그인 한 사용자와 일치하는지 확인해야 한다. 중복 코드를
피하기 위해 ``post``\를 가져오는 함수를 작성하고 각 뷰에서 그 함수를
호출할 수 있다.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    def get_post(id, check_author=True):
        post = get_db().execute(
            'SELECT p.id, title, body, created, author_id, username'
            ' FROM post p JOIN user u ON p.author_id = u.id'
            ' WHERE p.id = ?',
            (id,)
        ).fetchone()

        if post is None:
            abort(404, "Post id {0} doesn't exist.".format(id))

        if check_author and post['author_id'] != g.user['id']:
            abort(403)

        return post

:func:`abort`\는 HTTP 상태 코드를 반환하는 특수한 예외를 던진다.
선택적으로 메시지를 받아서 오류와 함께 보여 주며, 없으면 기본
메시지를 쓴다. ``404``\는 "Not Found"를 뜻하고 ``403``\은
"Forbidden"을 뜻한다. (``401``\이 "Unauthorized"를 뜻하는데,
우리는 그 상태를 반환하지 않고 로그인 페이지로 재지향 한다.)

``check_author`` 인자는 이 함수를 이용해 작성자 확인 없이
``post``\를 가져올 수 있도록 하기 위한 것이다. 글을 변경하지
않아서 사용자가 누구든 상관없는 페이지에 개별 글을 보여 주는
뷰를 작성하는 경우 유용할 것이다.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/<int:id>/update', methods=('GET', 'POST'))
    @login_required
    def update(id):
        post = get_post(id)

        if request.method == 'POST':
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if error is not None:
                flash(error)
            else:
                db = get_db()
                db.execute(
                    'UPDATE post SET title = ?, body = ?'
                    ' WHERE id = ?',
                    (title, body, id)
                )
                db.commit()
                return redirect(url_for('blog.index'))

        return render_template('blog/update.html', post=post)

지금까지 작성한 뷰들과 달리 ``update`` 함수는 ``id``\라는 인자를
받는다. 라우트의 ``<int:id>``\가 그 인자에 대응한다. 실제 URL은
``/1/update`` 같은 형태가 된다. 플라스크에서 ``1``\을 잡아내서
:class:`int`\인지 확인한 다음 ``id`` 인자로 전달해 준다.
``int:``\를 지정하지 않고 ``<id>``\라고 하면 문자열이 된다.
갱신 페이지에서 URL을 만들려면 :func:`url_for`\에 ``id``\를 줘서
뭘로 채울지 가르쳐 줘야 한다. 즉
``url_for('blog.update', id=post['id'])``\처럼 하면 된다.
위의 ``index.html`` 파일에서도 마찬가지다.

``create`` 뷰와 ``update`` 뷰는 아주 비슷하게 생겼다. 주된 차이는
``update`` 뷰에선 ``post`` 객체를 사용하고 ``INSERT`` 대신
``UPDATE`` 질의를 한다는 점이다. 리팩토링을 잘 하면 두 동작에
같은 뷰와 템플릿을 쓸 수도 있겠지만 길라잡이에 쓰기에는
둘을 따로 두는 게 이해하기에 좋다.

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/update.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Edit "{{ post['title'] }}"{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="title">Title</label>
        <input name="title" id="title"
          value="{{ request.form['title'] or post['title'] }}" required>
        <label for="body">Body</label>
        <textarea name="body" id="body">{{ request.form['body'] or post['body'] }}</textarea>
        <input type="submit" value="Save">
      </form>
      <hr>
      <form action="{{ url_for('blog.delete', id=post['id']) }}" method="post">
        <input class="danger" type="submit" value="Delete" onclick="return confirm('Are you sure?');">
      </form>
    {% endblock %}

이 템플릿에는 양식이 두 개 있다. 첫 번째 양식은 편집한 데이터를
현재 페이지(``/<id>/update``)로 POST 한다. 다른 양식에는 버튼이
있고 삭제 뷰로 POST 하도록 ``action`` 속성이 지정돼 있다. 그
버튼에서는 자바스크립트를 좀 써서 제출 전에 확인 대화창을
표시한다.

``{{ request.form['title'] or post['title'] }}`` 패턴을 써서
양식에 표시할 데이터를 선택한다. 양식을 제출하지 않았을 때는
원래 ``post``\의 데이터를 표시한다. 하지만 유효하지 않은 양식
데이터가 와서 사용자가 오류를 고칠 수 있게 표시하고 싶을 때는
``request.form``\을 쓴다. :data:`request`\는 템플릿 안에서
기본으로 사용 가능한 또 다른 변수이다.


삭제
----

삭제 뷰에는 따로 템플릿이 없으며 ``update.html``\에 있는 삭제
버튼이 ``/<id>/delete`` URL로 POST 한다. 템플릿이 없기 때문에
``POST`` 메소드만 처리한 다음 ``index`` 뷰로 재지향 한다.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/<int:id>/delete', methods=('POST',))
    @login_required
    def delete(id):
        get_post(id)
        db = get_db()
        db.execute('DELETE FROM post WHERE id = ?', (id,))
        db.commit()
        return redirect(url_for('blog.index'))

축하한다. 이제 응용 작성이 다 끝났다. 시간을 좀 내서
브라우저에서 이런저런 것들을 시도해 보자. 그런데 프로젝트가
완료되려면 할 일이 아직 남아 있다.

:doc:`install` 절로 이어진다.
