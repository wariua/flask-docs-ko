현장에 배치하기
===============

이번 따라하기에서는 응용을 배치할 서버가 한 대 있다고 가정한다.
배포용 파일을 어떻게 만들어서 설치하는지 간단히 살펴보되, 어떤
서버나 소프트웨어를 쓸 것인지까지는 들어가지 않는다. 개발용
컴퓨터에 새로 환경을 구성해서 아래 과정을 따라해 볼 수도
있겠지만 정말 공개할 응용을 올리는 데 쓰지는 말아야 할 것이다.
응용을 올리는 여러 방식에 대해선 :doc:`/deploying/index` 절을
보라.


빌드 및 설치
------------

응용을 다른 어딘가에 배치하고 싶을 때는 배포 파일을 만든다.
현재 파이썬 배포 표준은 ``.whl`` 확장자를 쓰는 *wheel* 형식이다.
일단 wheel 라이브러리가 설치돼 있어야 한다.

.. code-block:: none

    pip install wheel

파이썬으로 ``setup.py``\를 실행하면 빌드 관련 명령을 내릴 수
있는 명령행 도구가 된다. ``bdist_wheel`` 명령을 주면 wheel
배포 파일을 빌드 한다.

.. code-block:: none

    python setup.py bdist_wheel

``dist/flaskr-1.0.0-py3-none-any.whl`` 경로에서 파일을 찾을 수
있다. 파일 이름은 프로젝트 이름, 버전, 그리고 파일에 대한 몇
가지 태그들로 돼 있다.

이 파일을 다른 머신으로 복사하고
:ref:`새로 virtualenv를 구성 <install-create-env>`\한 다음
``pip``\로 파일을 설치하면 된다.

.. code-block:: none

    pip install flaskr-1.0.0-py3-none-any.whl

pip가 프로젝트와 의존 패키지들을 함께 설치해 줄 것이다.

다른 머신이므로 인스턴스 폴더에서 다시 ``init-db``\를 실행해
데이터베이스를 만들어야 한다.

.. code-block:: none

    export FLASK_APP=flaskr
    flask init-db

설치돼 있다는 걸 (즉 편집 가능 모드가 아니라는 걸) 플라스크에서
감지하면 인스턴스 폴더로 다른 디렉터리를 쓴다.
``venv/var/flaskr-instance``\에 있다.


비밀 키 설정하기
----------------

따라하기 초반에서 :data:`SECRET_KEY`\에 기본값을 줬다.
운용 버전에선 그걸 어떤 난수 바이트로 바꿔 줘야 한다.
안 그러면 공개된 ``'dev'`` 키를 이용해 공격자가 세션 쿠키나
비밀 키를 쓰는 뭐든 조작할 수 있게 된다.

다음 명령으로 난수 비밀 키를 만들 수 있다.

.. code-block:: none

    python -c 'import os; print(os.urandom(16))'

    b'_5#y2L"F4Q8z\n\xec]/'

인스턴스 폴더에 ``config.py`` 파일을 만들어 두면 팩토리가 그걸
읽게 된다. 생성된 값을 그 파일로 복사하자.

.. code-block:: python
    :caption: ``venv/var/flaskr-instance/config.py``

    SECRET_KEY = b'_5#y2L"F4Q8z\n\xec]/'

필요한 다른 항목이 있으면 마찬가지로 이 파일에서 설정할 수
있다. 하지만 Flaskr에서는 ``SECRET_KEY``\만 설정하면 된다.


제품 수준 서버로 돌리기
-----------------------

개발 중 말고 공개해서 돌릴 때는 내장 개발용 서버(``flask run``)를
쓰지 말아야 한다. 개발용 서버는 Werkzeug에서 편의를 위해 제공하는
것일 뿐이며 특별히 효율적이거나 안정적이거나 안전하도록 설계돼
있지 않다.

대신 전용 WSGI 서버를 사용하자. 예를 들어 `Waitress`_\를 쓰려면
먼저 가상 환경에 설치하면 된다.

.. code-block:: none

    pip install waitress

Waitress에게 응용에 대해 알려 줘야 하는데 ``flask run``\에서처럼
``FLASK_APP``\을 쓰지 않는다. 응용 팩토리를 임포트 하고 호출해서
응용 객체를 얻도록 해야 한다.

.. code-block:: none

    waitress-serve --call 'flaskr:create_app'

    Serving on http://0.0.0.0:8080

응용을 올리는 다른 여러 방법들에 대해선 :doc:`/deploying/index`
절을 보라. Waitress는 예일 뿐이며, 윈도우와 리눅스 모두 지원하기
때문에 고른 것뿐이다. 프로젝트에 선택해 쓸 수 있는 WSGI 서버와
배치 방식들이 여러 가지 있다.

.. _Waitress: https://docs.pylonsproject.org/projects/waitress/en/stable/

:doc:`next` 절로 이어진다.
