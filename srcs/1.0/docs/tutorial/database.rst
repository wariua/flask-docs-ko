.. currentmodule:: flask

데이터베이스 정의와 접근
========================

응용에서는 `SQLite`_ 데이터베이스를 이용해 사용자와 게시 글을
저장한다. 파이썬에는 :mod:`sqlite3` 모듈로 SQLite 지원이 내장돼
있다.

SQLite가 편리한 건 데이터베이스 서버를 따로 구성할 필요 없이
파이썬에 내장돼 있기 때문이다. 하지만 동시에 여러 요청에서
데이터베이스에 쓰기를 하려고 하면 각 쓰기가 순차적으로 이뤄지면서
속도가 느려지게 된다. 작은 응용에서는 그게 눈에 띄지 않겠지만
응용이 커지고 나면 다른 데이터베이스로 바꾸고 싶어 질 수도 있다.

이 길라잡이에서는 SQL에 대해 자세히 들어가지 않는다. SQL에
익숙치 않다면 SQLite 문서에 있는 `언어`_ 설명을 볼 수 있다.

.. _SQLite: https://sqlite.org/about.html
.. _언어: https://sqlite.org/lang.html


데이터베이스에 연결하기
-----------------------

SQLite 데이터베이스를 (그리고 다른 대부분의 파이썬 데이터베이스
라이브러리를) 쓸 때 가장 먼저 하는 일은 연결 만들기이다. 모든
질의와 동작을 그 연결로 수행하다가 할 일이 끝나면 연결을 닫는다.

웹 응용에서 이 연결은 보통 어떤 요청에 연계돼 있다. 그래서 요청을
처리하는 어느 시점에 만들었다가 응답을 보내기 전에 닫는다.

.. code-block:: python
    :caption: ``flaskr/db.py``

    import sqlite3

    import click
    from flask import current_app, g
    from flask.cli import with_appcontext


    def get_db():
        if 'db' not in g:
            g.db = sqlite3.connect(
                current_app.config['DATABASE'],
                detect_types=sqlite3.PARSE_DECLTYPES
            )
            g.db.row_factory = sqlite3.Row

        return g.db


    def close_db(e=None):
        db = g.pop('db', None)

        if db is not None:
            db.close()

:data:`g`\는 각 요청에 고유한 특수 객체이다. 요청을 처리하는 동안
여러 함수에서 접근할 수도 있는 데이터를 저장하는 데 쓴다. 여기에
연결을 저장해 뒀다가 같은 요청에서 ``get_db``\를 두 번째 호출할 때
새 연결을 만들지 않고 기존 연결을 재사용한다.

:data:`current_app`\은 또 다른 특수 객체인데 요청을 처리하고 있는
플라스크 응용을 가리킨다. 응용 팩토리를 사용했기 때문에 코드
나머지 부분에서는 응용 객체가 없다. ``get_db``\가 호출될 때는
응용이 생성돼서 요청을 처리할 때이고, 따라서 :data:`current_app`\을
쓸 수 있다.

:func:`sqlite3.connect`\는 설정 키 ``DATABASE``\가 가리키는 파일로
연결을 만든다. 이 파일이 미리 존재해야 하는 건 아니며 잠시 후
데이터베이스를 초기화 할 때까지는 만들어지지 않는다.

:class:`sqlite3.Row`\는 연결에서 반환하는 행이 딕셔너리처럼 동작하게
한다. 그래서 이름으로 열에 접근할 수 있다.

``close_db``\에서는 ``g.db``\가 설정돼 있는지 확인해서 연결이
생성돼 있는지 살펴본다. 연결이 존재하면 닫는다. 잠시 후에 응용
팩토리에서 ``close_db`` 함수에 대해 응용에 알려 줘서 요청이 끝날
때마다 호출되게 할 것이다.


테이블 만들기
-------------

SQLite에서 데이터는 *테이블*\과 *칼럼*\에 저장된다. 따라서 데이터를
저장하고 꺼내려면 미리 만들어 둬야 한다. Flaskr에서는 ``user``
테이블에 사용자를 저장하고 ``post`` 테이블에 게시 글을 저장한다.
빈 테이블들을 만드는 데 필요한 SQL 명령을 담은 파일을 하나 만들자.

.. code-block:: sql
    :caption: ``flaskr/schema.sql``

    DROP TABLE IF EXISTS user;
    DROP TABLE IF EXISTS post;

    CREATE TABLE user (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      username TEXT UNIQUE NOT NULL,
      password TEXT NOT NULL
    );

    CREATE TABLE post (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      author_id INTEGER NOT NULL,
      created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      title TEXT NOT NULL,
      body TEXT NOT NULL,
      FOREIGN KEY (author_id) REFERENCES user (id)
    );

``db.py`` 파일에 이 SQL 명령들을 실행할 파이썬 함수를 추가하자.

.. code-block:: python
    :caption: ``flaskr/db.py``

    def init_db():
        db = get_db()

        with current_app.open_resource('schema.sql') as f:
            db.executescript(f.read().decode('utf8'))


    @click.command('init-db')
    @with_appcontext
    def init_db_command():
        """기존 데이터 비우고 새 테이블 만들기."""
        init_db()
        click.echo('Initialized the database.')

:meth:`open_resource() <Flask.open_resource>`\는 ``flaskr`` 패키지
기준 상대 경로로 파일을 연다. 따라서 이후 응용을 배포할 때 위치가
어딘지 알고 있을 필요가 없게 된다. ``get_db``\가 데이터베이스 연결을
반환하면 그걸 이용해 파일에서 읽어 온 명령들을 실행한다.

:func:`click.command`\는 ``init-db``\라는 명령행 명령을 정의해서
``init_db`` 함수를 호출하고 사용자에게 성공했다는 메시지를 표시한다.
명령 작성에 대해선 :ref:`cli` 절에서 더 배울 수 있다.


응용에 등록해 두기
------------------

``close_db``\와 ``init_db_command`` 함수를 응용에서 쓰려면 그
함수들을 응용 인스턴스에 등록해 두어야 한다. 하지만 팩토리 함수를
쓰고 있기 때문에 그 함수들을 작성할 때는 인스턴스를 쓸 수 없다.
이럴 때는 응용을 받아서 등록을 해 주는 함수를 작성하면 된다.

.. code-block:: python
    :caption: ``flaskr/db.py``

    def init_app(app):
        app.teardown_appcontext(close_db)
        app.cli.add_command(init_db_command)

:meth:`app.teardown_appcontext() <Flask.teardown_appcontext>`\는
응답을 보낸 다음 정리를 할 때 플라스크가 그 함수를 호출하도록
한다.

:meth:`app.cli.add_command() <click.Group.add_command>`\는
``flask`` 명령으로 호출할 수 있는 새 명령을 추가한다.

이 함수를 팩토리에서 임포트 해서 호출한다. 팩토리 함수 끝의
app 반환 앞에 새 코드를 집어넣는다.

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    def create_app():
        app = ...
        # existing code omitted

        from . import db
        db.init_app(app)

        return app


데이터베이스 파일 초기화 하기
-----------------------------

``init-db``\를 응용에 등록했기 때문에 이제 ``flask`` 명령으로
부를 수 있다. 앞 페이지의 ``run`` 명령처럼 하면 된다.

.. note::

    앞 페이지의 그 서버를 아직 돌리고 있는 경우에는 서버를 멈추거나
    아니면 새 터미널에서 명령을 실행하면 된다. 새 터미널을 쓰는 경우
    프로젝트 디렉터리로 이동해서 :ref:`install-activate-env` 절의
    설명대로 env 활성화를 해 줘야 한다. 또 앞 페이지에 나와 있는
    것처럼 ``FLASK_APP``\와 ``FLASK_ENV``\도 설정해 줘야 한다.

``init-db`` 명령 실행:

.. code-block:: none

    flask init-db
    Initialized the database.

그러면 프로젝트의 ``instance`` 폴더에 ``flaskr.sqlite`` 파일이
생기게 된다.

:doc:`views` 절로 이어진다.
