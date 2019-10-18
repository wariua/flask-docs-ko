.. currentmodule:: flask

테스트 커버리지
===============

응용에 유닛 테스트를 만들면 작성한 코드가 의도한 대로 동작하는지
확인할 수 있다. 요청을 모의로 만들어서 응용으로 보내고 응답
데이터를 반환하는 테스트 클라이언트를 플라스크에서 제공한다.

가능하면 많은 코드를 테스트 해야 한다. 함수 내의 코드는 그
함수가 호출될 때만 실행되고 ``if`` 블록 같은 분기문 내의 코드는
그 조건이 맞을 때만 실행된다. 함수 각각을, 그리고 분기 각각으로
가는 데이터로 테스트를 해야 할 것이다.

커버리지가 100%에 가까울수록 어떤 변경 사항이 예상치 못하게
다른 동작 방식을 바꾸는 일이 없으리라 여기고 안심할 수 있게
된다. 하지만 커버리지가 100%라는 게 응용에 버그가 없음을
보장하는 건 아니다. 특히 사용자가 브라우저에서 어떻게 응용과
상호작용하는지는 테스트 하지 못한다. 하지만 그렇다고 해도
테스트 커버리지는 분명 개발 과정에서 이용할 수 있는 중요한
도구이다.

.. note::
    따라하기에서는 후반에 소개하고 있지만 향후
    프로젝트에서는 개발과 테스트를 함께 진행해야 할 것이다.

코드 테스트와 측정에 `pytest`_\와 `coverage`_\를 이용할
것이다. 두 패키지를 설치하자.

.. code-block:: none

    pip install pytest coverage

.. _pytest: https://pytest.readthedocs.io/
.. _coverage: https://coverage.readthedocs.io/


환경 구성과 픽스처
------------------

테스트 코드는 ``tests`` 디렉터리에 들어간다. 이 디렉터리는
``flaskr`` 패키지 안이 아니라 *옆에* 위치한다. ``tests/conftest.py``
파일에는 *픽스처(fixture)*\라는 환경 구성 함수가 있어서
각 테스트에서 이용하게 된다. 테스트들은 ``test_``\로 시작하는
파이썬 모듈 안에 있고 그 모듈 안의 테스트 함수들도 각각
``test_``\로 시작한다.

각 테스트마다 임시 데이터베이스를 새로 만들고 데이터를 좀
채워 넣어서 테스트에서 이용하게 된다. 그 데이터를 집어넣기
위한 SQL 파일을 작성해야 한다.

.. code-block:: sql
    :caption: ``tests/data.sql``

    INSERT INTO user (username, password)
    VALUES
      ('test', 'pbkdf2:sha256:50000$TCI4GzcX$0de171a4f4dac32e3364c7ddc7c14f3e2fa61f2d17574483f7ffbb431b4acb2f'),
      ('other', 'pbkdf2:sha256:50000$kJPKsz6N$d2d4784f1b030a9761f5ccaeeaca413f27f2ecb76d6168407af962ddce849f79');

    INSERT INTO post (title, body, author_id, created)
    VALUES
      ('test title', 'test' || x'0a' || 'body', 1, '2018-01-01 00:00:00');

픽스처 ``app``\에서는 팩토리를 부르면서 ``test_config``\를 주고,
그래서 로컬 개발 설정을 쓰는 대신 테스트에 쓰기 위한 응용 및
데이터베이스를 구성한다.

.. code-block:: python
    :caption: ``tests/conftest.py``

    import os
    import tempfile

    import pytest
    from flaskr import create_app
    from flaskr.db import get_db, init_db

    with open(os.path.join(os.path.dirname(__file__), 'data.sql'), 'rb') as f:
        _data_sql = f.read().decode('utf8')


    @pytest.fixture
    def app():
        db_fd, db_path = tempfile.mkstemp()

        app = create_app({
            'TESTING': True,
            'DATABASE': db_path,
        })

        with app.app_context():
            init_db()
            get_db().executescript(_data_sql)

        yield app

        os.close(db_fd)
        os.unlink(db_path)


    @pytest.fixture
    def client(app):
        return app.test_client()


    @pytest.fixture
    def runner(app):
        return app.test_cli_runner()

:func:`tempfile.mkstemp`\는 임시 파일을 생성해서 연 다음 파일
객체와 그 경로를 반환한다. 인스턴스 폴더 대신 그 임시 경로를
가리키도록 ``DATABASE`` 경로를 바꾼다. 경로를 설정한 다음
데이터베이스 테이블들을 생성하고 테이스 데이터를 삽입한다.
테스트가 끝나고 나면 임시 파일을 닫고서 제거한다.

:data:`TESTING`\은 앱이 테스트 상태라는 걸 플라스크에게 알려 준다.
플라스크에서는 몇 가지 내부 동작을 바꿔서 테스트를 더 쉽게 하는데,
다른 확장들에서도 테스트를 더 쉽게 하는 데 이 플래그를 쓸 수 있다.

``client`` 픽스처는 ``app`` 픽스처로 생성한 응용 객체로
:meth:`app.test_client() <Flask.test_client>`\를 호출한다.
테스트들에서 그 클라이언트를 써서 서버를 실행하지 않고
응용으로 요청을 보낸다.

``runner`` 픽스처는 ``client``\와 비슷하다.
:meth:`app.test_cli_runner() <Flask.test_cli_runner>`\가
생성하는 실행기가 응용에 등록된 클릭 명령을 호출할 수 있다.

Pytest에서는 테스트 함수의 인자 이름과 일치하는 함수 이름의
픽스처를 쓴다. 예를 들어 곧 작성하게 될 ``test_hello``
함수는 ``client`` 인자를 받는다. Pytest에서 그 이름과
일치하는 ``client`` 픽스처 함수를 찾아서 호출하고,
반환된 값을 테스트 함수에게 전달해 준다.


팩토리
------

팩토리 자체에는 테스트 할 게 그리 많지 않다. 대부분의 코드를
각 테스트에서 이미 실행했을 거라서 뭔가 잘못이 있다면 다른
테스트에서 표시가 나게 된다.

유일하게 바꿀 수 있는 동작은 테스트 설정을 주는지 여부이다.
설정을 주지 않은 경우에는 어떤 기본 설정이 있어야 할 것이고,
아닌 경우에는 설정이 오버라이드 돼야 한다.

.. code-block:: python
    :caption: ``tests/test_factory.py``

    from flaskr import create_app


    def test_config():
        assert not create_app().testing
        assert create_app({'TESTING': True}).testing


    def test_hello(client):
        response = client.get('/hello')
        assert response.data == b'Hello, World!'

따라하기 초반에 팩토리를 작성하면서 예시로 ``hello`` 라우트를
추가했다. "Hello, World!"를 반환하므로 응답 데이터가 일치하는지
테스트에서 확인한다.


데이터베이스
------------

응용 문맥 내에서 ``get_db``\는 호출될 때마다 같은 연결을
반환해야 한다. 그리고 문맥이 끝나면 연결이 닫혀야 한다.

.. code-block:: python
    :caption: ``tests/test_db.py``

    import sqlite3

    import pytest
    from flaskr.db import get_db


    def test_get_close_db(app):
        with app.app_context():
            db = get_db()
            assert db is get_db()

        with pytest.raises(sqlite3.ProgrammingError) as e:
            db.execute('SELECT 1')

        assert 'closed' in str(e)

``init-db`` 명령이 ``init_db`` 함수를 호출하고 메시지를
출력해야 한다.

.. code-block:: python
    :caption: ``tests/test_db.py``

    def test_init_db_command(runner, monkeypatch):
        class Recorder(object):
            called = False

        def fake_init_db():
            Recorder.called = True

        monkeypatch.setattr('flaskr.db.init_db', fake_init_db)
        result = runner.invoke(args=['init-db'])
        assert 'Initialized' in result.output
        assert Recorder.called

이 테스트에서는 Pytest의 ``monkeypatch`` 픽스처를 써서 ``init_db``
함수를 자기가 호출된 걸 기록하는 함수로 바꾼다. 그리고 위에서
작성했던 ``runner`` 픽스처를 써서 이름으로 ``init-db`` 명령을
호출한다.


인증
----

대부분의 뷰에서 사용자는 로그인을 해야 한다. 테스트 내에서 그렇게
하는 가장 쉬운 방법은 클라이언트로 ``POST`` 요청을 ``login`` 뷰로
보내는 것이다. 그런데 그걸 매번 작성하는 대신 그 동작을 하는
메소드가 있는 클래스를 작성한 다음 픽스처를 이용해 각 테스트에서
클라이언트를 받도록 할 수 있다.

.. code-block:: python
    :caption: ``tests/conftest.py``

    class AuthActions(object):
        def __init__(self, client):
            self._client = client

        def login(self, username='test', password='test'):
            return self._client.post(
                '/auth/login',
                data={'username': username, 'password': password}
            )

        def logout(self):
            return self._client.get('/auth/logout')


    @pytest.fixture
    def auth(client):
        return AuthActions(client)

``auth`` 픽스처를 쓰면 테스트 내에서 ``auth.login()``\을 호출해
로그인을 할 수 있다. ``app`` 픽스처에서 테스트용 데이터로 넣어 뒀던
``test`` 사용자로 로그인 한다.

``register`` 뷰는 ``GET``\에 정상적으로 결과가 나와야 한다. 유효한
양식 데이터의 ``POST``\에는 로그인 URL로 재지향을 해야 하며
사용자 데이터가 데이터베이스에 들어가 있어야 한다. 유효하지 않은
데이터에는 오류 메시지를 표시해야 한다.

.. code-block:: python
    :caption: ``tests/test_auth.py``

    import pytest
    from flask import g, session
    from flaskr.db import get_db


    def test_register(client, app):
        assert client.get('/auth/register').status_code == 200
        response = client.post(
            '/auth/register', data={'username': 'a', 'password': 'a'}
        )
        assert 'http://localhost/auth/login' == response.headers['Location']

        with app.app_context():
            assert get_db().execute(
                "select * from user where username = 'a'",
            ).fetchone() is not None


    @pytest.mark.parametrize(('username', 'password', 'message'), (
        ('', '', b'Username is required.'),
        ('a', '', b'Password is required.'),
        ('test', 'test', b'already registered'),
    ))
    def test_register_validate_input(client, username, password, message):
        response = client.post(
            '/auth/register',
            data={'username': username, 'password': password}
        )
        assert message in response.data

:meth:`client.get() <werkzeug.test.Client.get>`\은 ``GET`` 요청을
하고 플라스크가 돌려준 :class:`Response` 객체를 반환한다. 비슷하게
:meth:`client.post() <werkzeug.test.Client.post>`\는 ``POST``
요청을 하며, 딕셔너리 ``data``\를 양식 데이터로 변환해 준다.

페이지가 잘 표시되는지 테스트 하려고 간단한 요청을 하고서
:attr:`~Response.status_code`\가 ``200 OK``\인지 확인한다. 표시가
실패하면 플라스크가 ``500 Internal Server Error`` 코드를
반환할 것이다.

등록 뷰에서 로그인 뷰로 재지향 할 때는 :attr:`~Response.headers`\에
``Location`` 헤더가 로그인 URL로 들어가 있게 된다.

:attr:`~Response.data`\에 응답 바디가 bytes로 들어 있다. 페이지
내용에 특정 값이 있기를 기대한다면 ``data`` 내에 있는지 확인하면
된다. bytes는 bytes로 비교해야 한다. 유니코드 텍스트로 비교하고
싶다면 :meth:`get_data(as_text=True)
<werkzeug.wrappers.BaseResponse.get_data>`\를 쓰면 된다.

``pytest.mark.parametrize``\는 Pytest가 같은 테스트 함수를
여러 인자들로 돌리게 한다. 같은 코드를 세 번 작성할 필요 없이
다양한 비유효 입력과 오류 메시지를 테스트 하는 데 쓴다.

``login`` 뷰 테스트는 ``register``\와 아주 비슷하다.
데이터베이스의 데이터를 확인하는 게 아니라 로그인 후에
:data:`session`\에 ``user_id``\가 설정돼 있어야 한다.

.. code-block:: python
    :caption: ``tests/test_auth.py``

    def test_login(client, auth):
        assert client.get('/auth/login').status_code == 200
        response = auth.login()
        assert response.headers['Location'] == 'http://localhost/'

        with client:
            client.get('/')
            assert session['user_id'] == 1
            assert g.user['username'] == 'test'


    @pytest.mark.parametrize(('username', 'password', 'message'), (
        ('a', 'test', b'Incorrect username.'),
        ('test', 'a', b'Incorrect password.'),
    ))
    def test_login_validate_input(auth, username, password, message):
        response = auth.login(username, password)
        assert message in response.data

``client``\를 ``with`` 블록에 쓰면 응답 반환 후에도 :data:`session`
같은 문맥 변수에 접근할 수 있다. 원래 요청 밖에서 ``session``\에
접근하면 오류가 발생하게 된다.

``logout`` 테스트는 ``login``\의 반대다. 로그아웃 후에
:data:`session`\에 ``user_id``\가 없어야 한다.

.. code-block:: python
    :caption: ``tests/test_auth.py``

    def test_logout(client, auth):
        auth.login()

        with client:
            auth.logout()
            assert 'user_id' not in session


블로그
------

모든 블로그 뷰에 앞서 작성한 ``auth`` 픽스처를 이용한다.
``auth.login()``\을 호출하면 이어지는 클라이언트 요청들은
``test`` 사용자로 로그인 한 상태로 이뤄진다.

``index`` 뷰는 테스트 데이터로 추가했던 글에 대한 정보를
표시해야 한다. 그리고 작성자로 로그인 했을 때는 글을
편집하기 위한 링크가 있어야 한다.

``index`` 뷰를 테스트 하면서 몇 가지 다른 인증 동작까지
테스트 해 볼 수 있다. 로그인 돼 있지 않은 동안은 각
페이지에서 로그인 또는 등록을 위한 링크를 보여 준다.
로그인 돼 있을 때는 로그아웃을 위한 링크가 있다.

.. code-block:: python
    :caption: ``tests/test_blog.py``

    import pytest
    from flaskr.db import get_db


    def test_index(client, auth):
        response = client.get('/')
        assert b"Log In" in response.data
        assert b"Register" in response.data

        auth.login()
        response = client.get('/')
        assert b'Log Out' in response.data
        assert b'test title' in response.data
        assert b'by test on 2018-01-01' in response.data
        assert b'test\nbody' in response.data
        assert b'href="/1/update"' in response.data

``create``, ``update``, ``delete`` 뷰에 접근하려면 사용자가
로그인 돼 있어야 한다. 로그인 된 사용자가 ``update``\와
``delete``\에 접근하려면 글의 저자여야 하며 아니면 ``403
Forbidden`` 상태가 반환된다. 주어진 ``id``\의 ``post``\가
존재하지 않으면 ``update``\와 ``delete``\가 ``404 Not Found``\를
반환해야 한다.

.. code-block:: python
    :caption: ``tests/test_blog.py``

    @pytest.mark.parametrize('path', (
        '/create',
        '/1/update',
        '/1/delete',
    ))
    def test_login_required(client, path):
        response = client.post(path)
        assert response.headers['Location'] == 'http://localhost/auth/login'


    def test_author_required(app, client, auth):
        # 글 작성자를 다른 사용자로 바꾸기
        with app.app_context():
            db = get_db()
            db.execute('UPDATE post SET author_id = 2 WHERE id = 1')
            db.commit()

        auth.login()
        # 현 사용자가 다른 사용자의 글을 변경할 수 없다
        assert client.post('/1/update').status_code == 403
        assert client.post('/1/delete').status_code == 403
        # 현 사용자에게 편집 링크가 보이지 않는다
        assert b'href="/1/update"' not in client.get('/').data


    @pytest.mark.parametrize('path', (
        '/2/update',
        '/2/delete',
    ))
    def test_exists_required(client, auth, path):
        auth.login()
        assert client.post(path).status_code == 404

``create`` 및 ``update`` 뷰가 ``GET`` 요청에 대해 페이지를
표시하고 ``200 OK`` 응답을 반환해야 한다. ``POST`` 요청으로
유효한 데이터를 보냈을 때 ``create``\는 새 글 데이터를
데이터베이스에 집어넣어야 하고 ``update``\는 기존 데이터를
변경해야 한다. 유효하지 않은 데이터에 대해 두 페이지 모두
오류 메시지를 보여야 한다.

.. code-block:: python
    :caption: ``tests/test_blog.py``

    def test_create(client, auth, app):
        auth.login()
        assert client.get('/create').status_code == 200
        client.post('/create', data={'title': 'created', 'body': ''})

        with app.app_context():
            db = get_db()
            count = db.execute('SELECT COUNT(id) FROM post').fetchone()[0]
            assert count == 2


    def test_update(client, auth, app):
        auth.login()
        assert client.get('/1/update').status_code == 200
        client.post('/1/update', data={'title': 'updated', 'body': ''})

        with app.app_context():
            db = get_db()
            post = db.execute('SELECT * FROM post WHERE id = 1').fetchone()
            assert post['title'] == 'updated'


    @pytest.mark.parametrize('path', (
        '/create',
        '/1/update',
    ))
    def test_create_update_validate(client, auth, path):
        auth.login()
        response = client.post(path, data={'title': '', 'body': ''})
        assert b'Title is required.' in response.data

``delete`` 뷰는 인덱스 URL로 재지향 해야 하며 그 글이 더 이상
데이터베이스에 존재하지 않아야 한다.

.. code-block:: python
    :caption: ``tests/test_blog.py``

    def test_delete(client, auth, app):
        auth.login()
        response = client.post('/1/delete')
        assert response.headers['Location'] == 'http://localhost/'

        with app.app_context():
            db = get_db()
            post = db.execute('SELECT * FROM post WHERE id = 1').fetchone()
            assert post is None


테스트 돌리기
-------------

필수는 아니지만 coverage로 테스트를 돌릴 때 출력 양을
좀 줄여 주는 몇 가지 추가 설정을 프로젝트의 ``setup.cfg``
파일에 추가할 수 있다.

.. code-block:: none
    :caption: ``setup.cfg``

    [tool:pytest]
    testpaths = tests

    [coverage:run]
    branch = True
    source =
        flaskr

테스트를 돌리려면 ``pytest`` 명령을 쓰면 된다. 작성한 테스트
함수들을 모두 찾아서 실행해 준다.

.. code-block:: none

    pytest

    ========================= test session starts ==========================
    platform linux -- Python 3.6.4, pytest-3.5.0, py-1.5.3, pluggy-0.6.0
    rootdir: /home/user/Projects/flask-tutorial, inifile: setup.cfg
    collected 23 items

    tests/test_auth.py ........                                      [ 34%]
    tests/test_blog.py ............                                  [ 86%]
    tests/test_db.py ..                                              [ 95%]
    tests/test_factory.py ..                                         [100%]

    ====================== 24 passed in 0.64 seconds =======================

테스트가 하나라도 실패하면 발생한 오류를 보여 준다. ``pytest -v``\로
실행하면 점 대신 각 테스트 함수들의 목록을 볼 수 있다.

테스트의 코드 커버리지를 측정하려면 pytest를 직접 실행하는 대신
``coverage`` 명령을 이용해 실행하면 된다.

.. code-block:: none

    coverage run -m pytest

터미널에서 간단한 커버리지 분석 결과를 볼 수 있다.

.. code-block:: none

    coverage report

    Name                 Stmts   Miss Branch BrPart  Cover
    ------------------------------------------------------
    flaskr/__init__.py      21      0      2      0   100%
    flaskr/auth.py          54      0     22      0   100%
    flaskr/blog.py          54      0     16      0   100%
    flaskr/db.py            24      0      4      0   100%
    ------------------------------------------------------
    TOTAL                  153      0     44      0   100%

또는 HTML 보고서를 통해 각 파일의 어느 행들이 커버되고 있는지 볼 수도 있다.

.. code-block:: none

    coverage html

그러면 ``htmlcov`` 디렉터리에 파일들이 만들어진다. 브라우저에서
``htmlcov/index.html``\을 열어서 보고서를 볼 수 있다.

:doc:`deploy` 절로 이어진다.
