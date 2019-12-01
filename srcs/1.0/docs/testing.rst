.. _testing:

플라스크 응용 테스트
====================

   **테스트가 안 됐으면 안 도는 거다.**

이 말을 누가 했는지는 알려져 있지 않다. 전적으로 옳은 말은 아니지만
그렇다고 진실과 거리가 먼 것도 아니다. 테스트 안 된 응용에선 기존
코드를 개선하는 게 힘들고 테스트 안 된 응용의 개발자들은 상당히
편집증적으로 되는 경향이 있다. 응용에 자동화된 테스트가 있으면
안전하게 변경 작업을 할 수 있으며 뭔가 문제가 생겼을 때 바로
알 수 있다.

플라스크에서 응용 테스트가 가능하도록 Werkzeug의 테스트용
:class:`~werkzeug.test.Client`\를 제공하고 문맥 지역 데이터를
다뤄 준다. 그래서 좋아하는 테스팅 솔루션과 함께 이용하면 된다.

이 문서에서는 테스트 기반 프레임워크로 `pytest`_\를 사용한다.
다음처럼 ``pip``\로 설치할 수 있다. ::

    pip install pytest

.. _pytest: https://docs.pytest.org/

응용
----

일단 테스트 할 응용이 있어야 되니까 :ref:`tutorial`\의 응용을
이용할 것이다. 그 응용이 없으면 :gh:`예시 디렉터리
<examples/tutorial>`\에서 소스 코드를 얻을 수 있다.

테스트를 위한 골격
------------------

먼저 응용 루트 밑에 tests 디렉터리를 추가하자. 그 다음 테스트를
담은 파이썬 파일(:file:`test_flaskr.py`)을 만들자. 파일 이름을
``test_*.py`` 형식으로 하면 pytest가 자동으로 찾아내게 된다.

다음으로 :func:`client`\라는 `pytest 픽스처`_\를 만들자. 이
픽스처에서는 응용을 테스트에 맞게 구성하고 새 데이터베이스를
초기화 한다. ::

    import os
    import tempfile

    import pytest

    from flaskr import flaskr


    @pytest.fixture
    def client():
        db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        flaskr.app.config['TESTING'] = True
        client = flaskr.app.test_client()

        with flaskr.app.app_context():
            flaskr.init_db()

        yield client

        os.close(db_fd)
        os.unlink(flaskr.app.config['DATABASE'])

각 개별 테스트에서 이 client 픽스처를 호출하게 된다. 응용에 대한
간단한 인터페이스를 제공하는데, 그 인터페이스를 통해 응용으로
테스트 요청을 보낼 수 있다. client에서는 쿠키 처리도 해 준다.

구성 과정에서 설정 플래그 ``TESTING``\을 활성화한다. 그렇게 하면
요청 처리 중 오류를 잡지 않아서 응용에 대해 테스트 요청 수행을
할 때 오류 보고를 더 잘 받을 수 있게 된다.

SQLite3가 파일 시스템 기반이기 때문에 :mod:`tempfile` 모듈을
이용하면 간단히 임시 데이터베이스를 만들고 초기화 할 수 있다.
:func:`~tempfile.mkstemp` 함수는 해 주는 일이 두 가지라서
저수준 파일 핸들과 난수 파일 이름을 반환해 주며, 후자는
데이터베이스 이름으로 쓴다. `db_fd`\는 :func:`os.close` 함수로
파일을 닫기 위해 그대로 가지고 있어야 한다.

테스트 후 데이터베이스를 삭제하기 위해 픽스처에서는 파일을
닫고 파일 시스템에서 삭제한다.

이제 테스트 스위트를 돌리면 다음 출력을 보게 된다. ::

    $ pytest

    ================ test session starts ================
    rootdir: ./flask/examples/flaskr, inifile: setup.cfg
    collected 0 items

    =========== no tests ran in 0.07 seconds ============

실제로 돌린 테스트는 하나도 없지만 ``flaskr`` 응용이 문법적으로
유효하다는 건 알게 됐다. 안 그랬으면 임포트에서 예외로 죽었을
것이다.

.. _pytest 픽스처:
   https://docs.pytest.org/en/latest/fixture.html

첫 번째 테스트
--------------

이제 응용의 기능성 테스트를 해 볼 차례다. 응용의 루트(``/``)에
접근하면 응용에서 "No entries here so far"라고 보이는지 확인해
보자. 그러기 위해 :file:`test_flaskr.py`\에 새 테스트 함수를
다음처럼 추가하자. ::

    def test_empty_db(client):
        """빈 데이터베이스로 시작."""

        rv = client.get('/')
        assert b'No entries here so far' in rv.data

테스트 함수 이름이 `test`\로 시작하는 데 유의하자. 그 이름을
통해 `pytest`_\가 테스트로 실행할 함수를 자동으로 알아낼 수 있다.

``client.get``\을 쓰면 어떤 경로로 응용에게 HTTP ``GET`` 요청을
보낼 수 있다. 반환 값은 :class:`~flask.Flask.response_class` 객체이다.
그럼 :attr:`~werkzeug.wrappers.BaseResponse.data` 속성을 통해
응용에서 온 (문자열) 반환 값을 살펴볼 수 있다. 이 경우엔
출력에 ``'No entries here so far'``\가 포함돼 있는지 확인한다.

다시 돌려 보면 테스트 통과 1건을 보게 된다. ::

    $ pytest -v

    ================ test session starts ================
    rootdir: ./flask/examples/flaskr, inifile: setup.cfg
    collected 1 items

    tests/test_flaskr.py::test_empty_db PASSED

    ============= 1 passed in 0.10 seconds ==============

로그인과 로그아웃
-----------------

우리 응용의 기능 대부분은 권한 있는 사용자만 쓸 수 있으므로 테스트
클라이언트를 로그인 및 로그아웃 시킬 방법이 필요하다. 이를 위해
필요한 양식 데이터(사용자 이름과 패스워드)로 로그인 및 로그아웃
페이지로 요청을 날린다. 그리고 로그인 및 로그아웃 페이지에서
재지향을 하므로 클라이언트가 재지향을 따라가도록 한다.

:file:`test_flaskr.py` 파일에 다음 두 함수를 추가하자. ::

    def login(client, username, password):
        return client.post('/login', data=dict(
            username=username,
            password=password
        ), follow_redirects=True)


    def logout(client):
        return client.get('/logout', follow_redirects=True)

이제 로그인과 로그아웃이 잘 동작하는지, 그리고 유효하지 않은
인증 정보로는 실패하는지 쉽게 테스트 할 수 있다. 새 테스트
함수를 추가하자. ::

    def test_login_logout(client):
        """로그인과 로그아웃이 잘 동작하는가."""

        rv = login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'])
        assert b'You were logged in' in rv.data

        rv = logout(client)
        assert b'You were logged out' in rv.data

        rv = login(client, flaskr.app.config['USERNAME'] + 'x', flaskr.app.config['PASSWORD'])
        assert b'Invalid username' in rv.data

        rv = login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'] + 'x')
        assert b'Invalid password' in rv.data

메시지 추가 테스트
------------------

메시지 추가 동작이 잘 동작하는지도 테스트 해야 한다. 새 테스트 함수를
다음처럼 추가하자. ::

    def test_messages(client):
        """메시지가 동작하는지 테스트."""

        login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'])
        rv = client.post('/add', data=dict(
            title='<Hello>',
            text='<strong>HTML</strong> allowed here'
        ), follow_redirects=True)
        assert b'No entries here so far' not in rv.data
        assert b'&lt;Hello&gt;' in rv.data
        assert b'<strong>HTML</strong> allowed here' in rv.data

HTML이 의도한 대로 본문에서는 허용되고 제목에서는 안 되는 걸 확인한다.

돌려 보면 이제 테스트 세 건이 통과된다. ::

    $ pytest -v

    ================ test session starts ================
    rootdir: ./flask/examples/flaskr, inifile: setup.cfg
    collected 3 items

    tests/test_flaskr.py::test_empty_db PASSED
    tests/test_flaskr.py::test_login_logout PASSED
    tests/test_flaskr.py::test_messages PASSED

    ============= 3 passed in 0.23 seconds ==============


기타 테스트 기법
----------------

위와 같이 테스트 클라이언트를 이용하는 방법 말고도
:meth:`~flask.Flask.test_request_context` 메소드를 ``with``
문과 같이 써서 요청 문맥을 한시적으로 활성화할 수도 있다.
그렇게 하면 뷰 함수 안에 있는 것처럼 :class:`~flask.request`,
:class:`~flask.g`, :class:`~flask.session` 객체에 접근할 수
있다. 다음이 그 방식을 보여 주는 예이다. ::

    import flask

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        assert flask.request.path == '/'
        assert flask.request.args['name'] == 'Peter'

문맥에 결속된 다른 객체들도 모두 같은 식으로 이용할 수 있다.

다른 설정들로 응용을 테스트 하고 싶은데 그렇게 할 손쉬운 방법이
없는 것 같다면 응용 팩토리로 바꾸는 걸 생각해 볼 수 있다.
(:ref:`app-factories` 참고.)

그런데 테스트 요청 문맥을 쓰는 경우에는
:meth:`~flask.Flask.before_request` 및 :meth:`~flask.Flask.after_request`
함수가 자동으로 호출되지 않는 점에 유의해야 한다. 단
:meth:`~flask.Flask.teardown_request` 함수는 테스트 요청 문맥이
``with`` 블록을 나갈 때 잘 실행된다. :meth:`~flask.Flask.before_request`
함수도 호출되게 하고 싶다면 :meth:`~flask.Flask.preprocess_request`\를
직접 호출해 줘야 한다. ::

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        app.preprocess_request()
        ...

응용이 어떻게 설계됐느냐에 따라 다르겠지만 데이터베이스 연결을
열거나 비슷한 뭔가를 하기 위해 이렇게 해 줘야 할 수 있다.

:meth:`~flask.Flask.after_request` 함수를 호출하고 싶다면
:meth:`~flask.Flask.process_response`\를 호출하면 되기는 한데
응답 객체를 줘야 한다. ::

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        resp = Response('...')
        resp = app.process_response(resp)
        ...

이 정도가 되면 차라리 테스트 클라이언트를 직접 쓰기 시작할
거라서 일반적으로 쓸모가 많지는 않다.

.. _faking-resources:

자원과 문맥 흉내내기
--------------------

.. versionadded:: 0.10

많이 쓰는 패턴으로 사용자 권한 정보와 데이터베이스 연결을 응용
문맥이나 :attr:`flask.g` 객체에 저장하는 방식이 있다. 일반적인
사용 패턴은 처음 사용 때 객체를 거기 넣었다가 파기 때 제거하는
것이다. 예를 들어 현재 사용자를 얻는 다음과 같은 코드를
생각해 보자. ::

    def get_user():
        user = getattr(g, 'user', None)
        if user is None:
            user = fetch_current_user_from_database()
            g.user = user
        return user

테스트 할 때 코드를 바꾸지 않고 외부에서 이 사용자를
바꿔치기할 수 있다면 편리할 것이다.
:data:`flask.appcontext_pushed` 시그널을 후킹해서
그렇게 할 수 있다. ::

    from contextlib import contextmanager
    from flask import appcontext_pushed, g

    @contextmanager
    def user_set(app, user):
        def handler(sender, **kwargs):
            g.user = user
        with appcontext_pushed.connected_to(handler, app):
            yield

다음과 같이 쓰면 된다. ::

    from flask import json, jsonify

    @app.route('/users/me')
    def users_me():
        return jsonify(username=g.user.username)

    with user_set(app, my_user):
        with app.test_client() as c:
            resp = c.get('/users/me')
            data = json.loads(resp.data)
            self.assert_equal(data['username'], my_user.username)


문맥 가지고 있기
----------------

.. versionadded:: 0.4

일반적인 요청을 보내면서도 문맥을 좀 더 오래 유지해서 추가로
내용을 살펴볼 수 있으면 좋을 때가 있다. 플라스크 0.4에선
:meth:`~flask.Flask.test_client`\를 ``with`` 블록과 함께
쓰면 가능하다. ::

    app = flask.Flask(__name__)

    with app.test_client() as c:
        rv = c.get('/?tequila=42')
        assert request.args['tequila'] == '42'

:meth:`~flask.Flask.test_client`\를 ``with`` 블록 없이 썼다면
(실제 요청 바깥에서 사용을 시도하는 것이므로) `request`\가
사용 가능하지 않아서 ``assert``\가 오류로 실패했을 것이다.


세션 접근 및 변경
-----------------

.. versionadded:: 0.8

테스트 클라이언트에서 세션에 접근하거나 그 내용을 변경하는 게
아주 유용한 때가 있다. 일반적으로 두 가지 방법이 있다. 단순히
세션에서 특정 키에 특정 값이 설정돼 있는지 확인하고 싶은 거라면
문맥을 갖고 있으면서 :data:`flask.session`\에 접근하면 된다. ::

    with app.test_client() as c:
        rv = c.get('/')
        assert flask.session['foo'] == 42

하지만 이 방법으로는 세션을 변경하거나 요청을 날리기 전에 세션에
접근하는 게 불가능하다. 플라스크 0.8부터는 "세션 트랜잭션"이라는
게 있어서 테스트 클라이언트 문맥에서 세션을 열고 그 내용을 변경하는
적절한 호출들을 흉내낼 수 있다. 트랜잭션 마지막에 세션이 저장돼서
테스트 클라이언트에서 사용할 수 있게 된다. 사용하는 세션 백엔드와는
독립적으로 동작한다. ::

    with app.test_client() as c:
        with c.session_transaction() as sess:
            sess['a_key'] = 'a value'

        # 여기까지 왔으면 세션이 저장돼 있어서 클라이언트에서 사용 가능
        c.get(...)

이 경우에 :data:`flask.session` 프록시가 아니라 ``sess`` 객체를
써야 한다는 점에 유의하자. 하지만 객체가 제공하는 인터페이스는
동일할 것이다.


JSON API 테스트
---------------

.. versionadded:: 1.0

플라스크는 JSON을 아주 잘 지원하고, 그래서 JSON API를 만들 때
인기 있는 선택지다. JSON 데이터로 요청을 만들고 응답의 JSON
데이터를 확인하는 걸 아주 편리하게 할 수 있다. ::

    from flask import request, jsonify

    @app.route('/api/auth')
    def auth():
        json_data = request.get_json()
        email = json_data['email']
        password = json_data['password']
        return jsonify(token=generate_token(email, password))

    with app.test_client() as c:
        rv = c.post('/api/auth', json={
            'username': 'flask', 'password': 'secret'
        })
        json_data = rv.get_json()
        assert verify_token(email, json_data['token'])

테스트 클라이언트 메소드에 ``json`` 인자를 주면 요청 데이터를
JSON 직렬화 객체로 설정하고 컨텐츠 타입을 ``application/json``\으로
설정한다. 그리고 ``get_json``\으로 요청이나 응답에서 JSON
데이터를 얻을 수 있다.


.. _testing-cli:

CLI 명령 테스트
---------------

`클릭`_\에는 CLI 명령 `테스트를 위한 유틸리티들`_\이 딸려 있다.
:class:`~click.testing.CliRunner`\는 격리 환경에서 명령을
돌리고 그 출력을 :class:`~click.testing.Result` 객체에 담아 준다.

플라스크에서 제공하는 :meth:`~flask.FLask.test_cli_runner`\는
:class:`~flask.testing.FlaskCliRunner`\를 생성하는데, 자동으로
플라스크 앱을 CLI에 전달해 준다.
:meth:`~flask.testing.FlaskCliRunner.invoke` 메소드를 쓰면
명령행에서 호출하는 것과 같은 식으로 명령을 작동시킬 수 있다. ::

    import click

    @app.cli.command('hello')
    @click.option('--name', default='World')
    def hello_command(name)
        click.echo(f'Hello, {name}!')

    def test_hello():
        runner = app.test_cli_runner()

        # 명령 직접 작동시키기
        result = runner.invoke(hello_command, ['--name', 'Flask'])
        assert 'Hello, Flask' in result.output

        # 이름으로 작동시키기
        result = runner.invoke(args=['hello'])
        assert 'World' in result.output

위 예에서 명령을 이름으로 작동시키는 게 쓸모가 있는 이유는 앱에
명령이 올바로 등록됐는지 검증해 주기 때문이다.

명령을 실행하지는 않으면서 명령에서 매개변수를 어떻게 파싱 하는지
테스트 하고 싶다면 :meth:`~click.BaseCommand.make_context`
메소드를 쓰면 된다. 복잡한 검증 규칙이나 새로 만든 타입을 테스트
하는 데 유용하다. ::

    def upper(ctx, param, value):
        if value is not None:
            return value.upper()

    @app.cli.command('hello')
    @click.option('--name', default='World', callback=upper)
    def hello_command(name)
        click.echo(f'Hello, {name}!')

    def test_hello_params():
        context = hello_command.make_context('hello', ['--name', 'flask'])
        assert context.params['name'] == 'FLASK'

.. _클릭: http://click.palletsprojects.com/
.. _테스트를 위한 유틸리티들: http://click.palletsprojects.com/testing/
