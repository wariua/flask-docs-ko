.. currentmodule:: flask

청사진과 뷰
===========

뷰(view) 함수란 응용에 대한 요청에 응답하기 위해 작성하는 코드이다.
플라스크에서는 패턴을 써서 들어오는 요청 URL과 그걸 처리해야 하는
뷰를 연결시킨다. 그리고 뷰에서 데이터를 반환하면 플라스크에서
응답으로 바꿔서 내보낸다. 또한 플라스크에서는 방향을 거꾸로 해서
뷰의 이름과 인자를 가지고 URL을 만들 수도 있다.


청사진 만들기
-------------

:class:`Blueprint`\는 연관된 뷰와 기타 코드를 하나로 묶어서 관리할
수 있는 방법이다. 뷰와 기타 코드를 응용에 직접 등록하는 게 아니라
청사진에 등록해 둔다. 그 후 준비가 됐을 때 팩토리 함수에서 청사진을
응용에 등록한다.

Flaskr에서 청사진을 두 개 만들 것이다. 하나는 인증 함수들을 위한
것이고 다른 하나는 블로그 글 함수들을 위한 것이다. 각 청사진의
함수들은 별도의 모듈로 들어가게 된다. 블로그에서 인증에 대해 알아야
하니까 먼저 인증 쪽을 작성할 것이다.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    import functools

    from flask import (
        Blueprint, flash, g, redirect, render_template, request, session, url_for
    )
    from werkzeug.security import check_password_hash, generate_password_hash

    from flaskr.db import get_db

    bp = Blueprint('auth', __name__, url_prefix='/auth')

이렇게 하면 ``'auth'``\라는 :class:`Blueprint`\가 만들어진다.
응용 객체와 마찬가지로 청사진에서도 정의된 위치를 알고 있어야
하므로 두 번째 인자로 ``__name__``\을 준다. 그리고 그 청사진에
연계된 모든 URL 앞에 ``url_prefix``\가 덧붙게 된다.

팩토리에서 청사진을 임포트 해서 :meth:`app.register_blueprint()
<Flask.register_blueprint>`\로 등록하자. 팩토리 함수 끝의
app 반환 앞에 새 코드를 집어넣는다.

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    def create_app():
        app = ...
        # 기존 코드 생략

        from . import auth
        app.register_blueprint(auth.bp)

        return app

인증 청사진에는 새 사용자를 등록하는 뷰, 그리고 로그인과 로그아웃을
위한 뷰가 들어가게 된다.


첫 번째 뷰: 등록
----------------

사용자가 URL ``/auth/register``\에 방문하면 ``register`` 뷰에서
사용자가 채울 양식이 있는 `HTML`_\을 반환하게 된다. 사용자가
양식을 제출하면 뷰에서 입력을 검증해서 오류 메시지와 함께 다시
양식을 표시하거나 새 사용자를 만들고 로그인 페이지로 이동한다.

.. _HTML: https://developer.mozilla.org/docs/Web/HTML

일단은 뷰 코드만 작성할 것이다. 다음 페이지에서 HTML 양식을
생성하기 위한 템플릿을 작성하게 된다.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    @bp.route('/register', methods=('GET', 'POST'))
    def register():
        if request.method == 'POST':
            username = request.form['username']
            password = request.form['password']
            db = get_db()
            error = None

            if not username:
                error = 'Username is required.'
            elif not password:
                error = 'Password is required.'
            elif db.execute(
                'SELECT id FROM user WHERE username = ?', (username,)
            ).fetchone() is not None:
                error = 'User {} is already registered.'.format(username)

            if error is None:
                db.execute(
                    'INSERT INTO user (username, password) VALUES (?, ?)',
                    (username, generate_password_hash(password))
                )
                db.commit()
                return redirect(url_for('auth.login'))

            flash(error)

        return render_template('auth/register.html')

``register`` 뷰 함수에서 하는 일은 다음과 같다.

#.  :meth:`@bp.route <Blueprint.route>`\로 URL ``/register``\를
    뷰 함수 ``register``\로 연결시킨다. 플라스크가
    ``/auth/register``\로 요청을 받으면 ``register`` 뷰를
    호출해서 그 반환 값을 응답으로 쓰게 된다.

#.  사용자가 양식을 제출하면 :attr:`request.method <Request.method>`\가
    ``'POST'``\가 된다. 그 경우 입력 검증을 시작한다.

#.  :attr:`request.form <Request.form>`\은 특별한 :class:`dict`\로서,
    제출받은 양식 키와 값들을 매핑 해 준다. 사용자는 자기
    ``username``\과 ``password``\를 입력하게 된다.

#.  ``username``\과 ``password``\가 비어 있지 않은지 확인한다.

#.  ``username``\이 이미 등록돼 있지 않은지 확인한다.
    데이터베이스에 질의를 해서 결과가 반환되는지 본다.
    :meth:`db.execute <sqlite3.Connection.execute>`\는 사용자
    입력에 대한 플레이스홀더 ``?``\가 있는 SQL 질의와
    플레이스홀더를 채울 값들의 튜플을 받는다. 데이터베이스
    라이브러리에서 값들을 이스케이프 해 주므로 *SQL 인젝션
    공격*\에 취약하지 않다.

    :meth:`~sqlite3.Cursor.fetchone`\은 질의에서 행 하나를
    반환한다. 질의에서 반환한 결과가 없으면 ``None``\을 반환한다.
    나중에는 :meth:`~sqlite3.Cursor.fetchall`\도 쓰게 되는데
    이는 모든 결과를 담은 리스트를 반환한다.

#.  검사가 통과하면 새 사용자 데이터를 데이터베이스에 집어넣는다.
    보안상 패스워드는 절대 데이터베이스에 직접 저장하지 말아야
    한다. 대신 :func:`~werkzeug.security.generate_password_hash`\를
    써서 패스워드를 안전하게 해싱 하고 그 해시를 저장한다.
    이 질의로 데이터를 변경하기 때문에 변경 내용을 저장하기 위해
    이어서 :meth:`db.commit() <sqlite3.Connection.commits>`\을
    호출해야 한다.

#.  사용자를 저장한 다음에는 로그인 페이지로 돌린다.
    :func:`url_for`\가 이름을 가지고 로그인 뷰의 URL을 만들어
    준다. 이렇게 하면 이후 링크 하는 코드를 바꾸지 않고도 URL을
    바꿀 수 있기 때문에 URL을 직접 적어넣는 것보다 이 방식이
    낫다. :func:`redirect`\는 생성된 URL로 가는 재지향 응답을
    만들어 낸다.

#.  검사에 실패하면 사용자에게 오류를 보여 준다. :func:`flash`\로
    메시지를 저장하면 템플릿을 렌더링 할 때 꺼낼 쓸 수 있다.

#.  사용자가 ``auth/register``\를 처음으로 열거나 검사 오류가
    있었을 때는 등록 양식이 있는 HTML 페이지를 보여 줘야 한다.
    :func:`render_template`\이 그 HTML을 담은 템플릿을 렌더링
    하게 되는데 따라하기 다음 단계에서 그 템플릿을 작성한다.


로그인
------

이 뷰는 위의 ``register`` 뷰와 같은 패턴을 따른다.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    @bp.route('/login', methods=('GET', 'POST'))
    def login():
        if request.method == 'POST':
            username = request.form['username']
            password = request.form['password']
            db = get_db()
            error = None
            user = db.execute(
                'SELECT * FROM user WHERE username = ?', (username,)
            ).fetchone()

            if user is None:
                error = 'Incorrect username.'
            elif not check_password_hash(user['password'], password):
                error = 'Incorrect password.'

            if error is None:
                session.clear()
                session['user_id'] = user['id']
                return redirect(url_for('index'))

            flash(error)

        return render_template('auth/login.html')

``register`` 뷰와 몇 가지 다른 점이 있다.

#.  먼저 사용자를 질의해서 이후 쓸 수 있게 변수에 저장한다.

#.  :func:`~werkzeug.security.check_password_hash`\는 제출받은
    패스워드를 저장된 해시와 같은 방식으로 해싱 해서 안전하게
    비교한다. 일치하면 패스워드가 유효한 것이다.

#.  :data:`session`\은 여러 요청에 걸쳐 사용할 데이터를 저장하는
    :class:`dict`\이다. 검사가 성공하면 사용자의 ``id``\를 세
    세션에 저장한다. 그 데이터는 *쿠키*\에 저장돼서 브라우저로
    가고, 브라우저는 후속 요청들에서 그 데이터를 함께 보낸다.
    플라스크에서 데이터를 안전하게 *서명*\하므로 그 데이터를
    조작하는 게 불가능하다.

이제 사용자의 ``id``\가 :data:`session`\에 저장됐으므로 후속
요청에서 쓸 수 있다. 모든 요청 시작에서 사용자가 로그인 돼
있으면 그 정보를 읽어 들어서 다른 뷰에서 쓸 수 있게 하자.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    @bp.before_app_request
    def load_logged_in_user():
        user_id = session.get('user_id')

        if user_id is None:
            g.user = None
        else:
            g.user = get_db().execute(
                'SELECT * FROM user WHERE id = ?', (user_id,)
            ).fetchone()

:meth:`bp.before_app_request() <Blueprint.before_app_request>`\는
요청 URL과 상관없이 모든 뷰 함수 앞에 실행되는 함수를 등록한다.
``load_logged_in_user``\에서는 사용자 ID가 :data:`session`\에
저장돼 있는지 확인하고 사용자 데이터를 데이터베이스에서 읽어 온다.
그리고 요청 동안 유지되는 :data:`g.user <g>`\에 그 데이터를 저장한다.
사용자 ID를 받지 못했거나 존재하지 않는 ID이면 ``g.user``\가
``None``\이 된다.


로그아웃
--------

로그아웃 하려면 :data:`session`\에서 사용자 ID를 없애면 된다.
그러면 이후 요청에서 ``load_logged_in_user``\가 사용자를 읽어
들이지 않게 된다.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    @bp.route('/logout')
    def logout():
        session.clear()
        return redirect(url_for('index'))


뷰에 인증 강제하기
------------------

블로그 글을 만들거나 편집하거나 삭제하려면 사용자가 로그인 돼
있어야 한다. *데코레이터*\를 써서 대상 뷰들을 대신해 그걸
확인할 수 있다.

.. code-block:: python
    :caption: ``flaskr/auth.py``

    def login_required(view):
        @functools.wraps(view)
        def wrapped_view(**kwargs):
            if g.user is None:
                return redirect(url_for('auth.login'))

            return view(**kwargs)

        return wrapped_view

이 데코레이터는 적용 대상인 원래 뷰를 감싸는 새로운 뷰 함수를
반환한다. 새 함수에서는 사용자를 읽어 들였는지 확인해서 안 돼
있으면 로그인 페이지로 재지향 한다. 사용자가 적재돼 있으면
원래 뷰를 호출하고 원래대로 실행이 이어진다. 블로그 뷰를
작성할 때 이 데코레이터를 쓰게 된다.

종점과 URL
----------

:func:`url_for` 함수는 이름과 인자를 가지고 뷰로 가는 URL을
생성한다. 뷰에 연계돼 있는 이름을 *종점(endpoint)*\이라고도
하는데 기본적으로 뷰 함수 이름과 동일하다.

예를 들어 따라하기 초반에 앱 팩토리에 추가했던 ``hello()``
뷰의 이름은 ``'hello'``\이고 ``url_for('hello')``\로 링크를
만들 수 있다. 있다가 보겠지만 인자를 받는 경우에는
``url_for('hello', who='World')`` 같은 식으로 링크를 만들 수
있다.

청사진을 쓸 때는 청사진 이름이 함수 이름 앞에 덧붙는다. 그래서
앞서 작성한 ``login`` 함수는 ``'auth'`` 청사진에 추가한 것이므로
종점이 ``'auth.login'``\이 된다.

:doc:`templates` 절로 이어진다.
