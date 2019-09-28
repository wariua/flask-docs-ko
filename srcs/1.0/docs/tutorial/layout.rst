프로젝트 구조
=============

프로젝트 디렉터리를 만들고 다음과 같이 입력한다.

.. code-block:: none

    mkdir flask-tutorial
    cd flask-tutorial

그리고 :doc:`설치 절차 </installation>`\를 따라 파이썬 가상 환경을
구성하고 이 프로젝트를 위한 플라스크를 설치한다.

지금부터 따라하기에서는 ``flask-tutorial`` 디렉터리에서 작업이
이뤄진다고 가정한다. 코드 블록 상단의 파일 이름들이 이 디렉터리를
기준으로 한다.

----

플라스크 응용이 파일 하나로 돼 있을 수도 있다.

.. code-block:: python
    :caption: ``hello.py``

    from flask import Flask

    app = Flask(__name__)


    @app.route('/')
    def hello():
        return 'Hello, World!'

하지만 프로젝트가 커 가면서 한 파일에 모든 걸 집어넣는 건 감당할
수가 없게 된다. 파이썬 프로젝트들에서는 *패키지* 를 사용해
코드를 여러 모듈로 조직하고 필요한 곳에서 임포트 하는데,
이 따라하기에서도 그렇게 할 것이다.

프로젝트 디렉터리에 다음이 들어가게 된다.

* ``flaskr/``: 응용 코드와 파일들을 담은 파이썬 패키지.
* ``tests/``: 테스트 모듈을 담은 디렉터리.
* ``venv/``: 플라스크 및 기타 의존 패키지들이 설치되는
  파이썬 가상 환경.
* 프로젝트 설치 방법을 파이썬에게 알려 주는 설치 파일들.
* `git`_ 같은 버전 관리 도구의 설정. 크기가 어떻든 모든
  프로젝트에 어떤 종류의 버전 관리를 사용하는 습관을
  들이는 게 좋다.
* 향후 추가하게 될 수 있을 여타 프로젝트 파일들.

.. _git: https://git-scm.com/

그래서 프로젝트 구조가 다음처럼 된다.

.. code-block:: none

    /home/user/Projects/flask-tutorial
    ├── flaskr/
    │   ├── __init__.py
    │   ├── db.py
    │   ├── schema.sql
    │   ├── auth.py
    │   ├── blog.py
    │   ├── templates/
    │   │   ├── base.html
    │   │   ├── auth/
    │   │   │   ├── login.html
    │   │   │   └── register.html
    │   │   └── blog/
    │   │       ├── create.html
    │   │       ├── index.html
    │   │       └── update.html
    │   └── static/
    │       └── style.css
    ├── tests/
    │   ├── conftest.py
    │   ├── data.sql
    │   ├── test_factory.py
    │   ├── test_db.py
    │   ├── test_auth.py
    │   └── test_blog.py
    ├── venv/
    ├── setup.py
    └── MANIFEST.in

버전 관리를 쓰고 있다면 프로젝트 실행 때 생성되는 다음 파일들을
무시해야 한다. 그리고 사용 편집기에 따라 다른 파일들도 있을 수
있다. 일반적으로 작성하지 않은 파일들은 무시하면 된다. 예를 들어
git 사용 시:

.. code-block:: none
    :caption: ``.gitignore``

    venv/

    *.pyc
    __pycache__/

    instance/

    .pytest_cache/
    .coverage
    htmlcov/

    dist/
    build/
    *.egg-info/

:doc:`factory` 절로 이어진다.
