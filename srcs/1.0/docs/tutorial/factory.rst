.. currentmodule:: flask

응용 준비
=========

플라스크 응용은 :class:`Flask` 클래스의 인스턴스이다. 설정이나
URL 같은 응용에 대한 모든 것들이 이 클래스에 기록된다.

플라스크 응용을 만드는 가장 단순한 방법은 앞서 "Hello, World!"
예시에서 했던 것처럼 코드 상단에서 전역 :class:`Flask` 인스턴스를
만드는 것이다. 이 방식은 간편하고 일부 경우에 유용하기도 하지만
프로젝트가 커지면서 좀 까다로운 문제들을 일으킬 수 있다.

전역으로 :class:`Flask` 인스턴스를 만드는 대신 어떤 함수 안에서
만들게 된다. 그 함수를 *응용 팩토리* 라고 한다. 설정이나 등록,
기타 응용에서 필요한 준비 작업들이 그 함수 안에서 이뤄지고
마지막으로 응용이 반환된다.


응용 팩토리
-----------

코딩 할 시간이다! ``flaskr`` 디렉터리를 만들고 ``__init__.py``
파일을 추가하자. ``__init__.py`` 에는 두 가지 역할이 있는데,
응용 팩토리가 들어가게 되며 파이썬에서 ``flaskr`` 디렉터리를
패키지로 다루도록 한다.

.. code-block:: none

    mkdir flaskr

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    import os

    from flask import Flask


    def create_app(test_config=None):
        # 앱 생성 및 설정
        app = Flask(__name__, instance_relative_config=True)
        app.config.from_mapping(
            SECRET_KEY='dev',
            DATABASE=os.path.join(app.instance_path, 'flaskr.sqlite'),
        )

        if test_config is None:
            # 테스트가 아닐 때 인스턴스 설정이 있으면 적재
            app.config.from_pyfile('config.py', silent=True)
        else:
            # 테스트 설정을 받았으면 적재
            app.config.from_mapping(test_config)

        # 인스턴스 폴더가 존재해야 함
        try:
            os.makedirs(app.instance_path)
        except OSError:
            pass

        # 인사를 하는 간단한 페이지
        @app.route('/hello')
        def hello():
            return 'Hello, World!'

        return app

``create_app`` 이 응용 팩토리 함수이다. 길라잡이 이후 부분에서
여기 뭔가를 또 추가하겠지만 지금도 충분히 많은 일을 하고 있다.

#.  ``app = Flask(__name__, instance_relative_config=True)`` 로
    :class:`Flask` 인스턴스를 만든다.

    *   ``__name__`` 은 현재 파이썬 모듈의 이름이다. 일부
        경로들을 만들기 위해 앱에서 자기 위치를 알 필요가
        있는데 ``__name__`` 은 그걸 알려 주는 편리한 방법이다.

    *   ``instance_relative_config=True`` 는 설정 파일이
        :ref:`인스턴스 폴더 <instance-folders>` 기준 상대 경로임을
        앱에게 알려 준다. 인스턴스 폴더는 ``flaskr`` 패키지 밖에
        위치해 있으며 비밀값이나 데이터베이스 파일 설정처럼
        버전 관리로 들어가선 안 되는 로컬 데이터를 담을 수 있다.

#.  :meth:`app.config.from_mapping() <Config.from_mapping>` 으로
    앱에서 사용할 몇 가지 기본 설정을 준다.

    *   :data:`SECRET_KEY` 를 사용해 플라스크와 확장들에서 데이터를
        안전하게 보관한다. 개발 중에는 간편한 값인 ``'dev'`` 로
        설정하지만 도입 시에는 난수 값으로 바꿔야 한다.

    *   ``DATABASE`` 는 SQLite 데이터 파일이 저장될 경로이다.
        플라스크에서 인스턴스 폴더로 선택한 경로인
        :attr:`app.instance_path <Flask.instance_path>` 아래에
        있게 된다. 다음 절에서 데이터베이스에 대해 배우게 된다.

#.  :meth:`app.config.from_pyfile() <Config.from_pyfile>` 로
    인스턴스 폴더에 ``config.py`` 파일이 있으면 거기서 가져온
    값들로 기본 설정을 바꾼다. 예를 들어 도입 시에 이를 이용해
    진짜 ``SECRET_KEY`` 를 설정할 수 있다.

    *   팩토리로 ``test_config`` 를 줘서 인스턴스 설정 대신
        쓰이게 할 수 있다. 길라잡이 이후에서 작성하게 될
        테스트에 이미 설정돼 있는 개발용 값들과는 독립적인
        설정을 주기 위한 것이다.

#.  :func:`os.makedirs` 로 :attr:`app.instance_path
    <Flask.instance_path>` 가 꼭 존재하게 한다. 플라스크에서
    인스턴스 폴더를 자동으로 만들어 주지 않으므로 직접 만들어야
    한다. 거기에 SQLite 데이터베이스 파일을 만들게 된다.

#.  :meth:`@app.route() <Flask.route>` 로 길라잡이 나머지 부분으로
    들어가기 전에 응용이 동작하는지 확인하기 위한 간단한 루트를
    만든다. 이 경우는 URL ``/hello`` 와 응답으로 문자열
    ``'Hello, World!'`` 를 반환하는 함수를 연결한다.


응용 실행하기
-------------

이제 ``flask`` 명령으로 응용을 실행할 수 있다. 터미널에서 응용이
어디 있는지 플라스크에게 알려 주고서 개발 모드로 실행해 보자.

개발 모드에서는 페이지에서 예외를 던질 때마다 대화형 디버거가
나타나고 코드를 변경할 때마다 서버가 재시작된다. 계속 돌게 두고서
길라잡이를 따라가며 브라우저 페이지만 새로고침 하면 된다.

리눅스 및 맥:

.. code-block:: none

    export FLASK_APP=flaskr
    export FLASK_ENV=development
    flask run

윈도우 cmd, ``export`` 대신 ``set`` 사용:

.. code-block:: none

    set FLASK_APP=flaskr
    set FLASK_ENV=development
    flask run

윈도우 파워셸, ``export`` 대신 ``$env:`` 사용:

.. code-block:: none

    $env:FLASK_APP = "flaskr"
    $env:FLASK_ENV = "development"
    flask run

다음과 비슷한 출력을 보게 된다.

.. code-block:: none

     * Serving Flask app "flaskr"
     * Environment: development
     * Debug mode: on
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: 855-212-761

브라우저에서 http://127.0.0.1:5000/hello 를 열면 "Hello, World!"
메시지를 보게 된다. 축하한다! 작성한 플라스크 웹 응용이 지금 돌고
있는 거다.

:doc:`database` 절로 이어진다.
