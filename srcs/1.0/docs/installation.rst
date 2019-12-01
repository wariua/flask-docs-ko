.. _installation:

설치
====

파이썬 버전
-----------

파이썬 3 최신 버전 사용을 권한다. 플라스크는 파이썬 3.4 및 이후 버전,
파이썬 2.7, 그리고 PyPy를 지원한다.

의존하는 패키지
---------------

플라스크를 설치할 때 다음 패키지들이 자동으로 설치된다.

* `Werkzeug`_ 응용 서버 간 인터페이스 파이썬 표준인 WSGI를 구현.
* `Jinja`_ 응용에서 내놓는 페이지를 생성하는 템플릿 언어
* `MarkupSafe`_ Jinja에 딸려 있다.. 템플릿을 페이지로 옮길 때
  입력을 이스케이프 해서 인젝션 공격을 막는다.
* `ItsDangerous`_ 데이터에 안전한 서명을 해서 무결성을 보장한다.
  플라스크의 세션 쿠키를 보호하는 데 쓴다.
* `Click`_ 명령행 응용 작성을 위한 프레임워크. ``flask`` 명령이
  이걸로 만들어졌으며 다른 관리 명령들을 추가할 수 있다.

.. _Werkzeug: http://palletsprojects.com/p/werkzeug/
.. _Jinja: http://palletsprojects.com/p/jinja/
.. _MarkupSafe: https://palletsprojects.com/p/markupsafe/
.. _ItsDangerous: https://palletsprojects.com/p/itsdangerous/
.. _Click: http://palletsprojects.com/p/click/

기타 의존 패키지
~~~~~~~~~~~~~~~~

다음 패키지들은 자동으로 설치되지 않는다. 설치돼 있으면 플라스크에서
탐지해서 사용한다.

* `Blinker`_ :ref:`signals` 기능 제공.
* `SimpleJSON`_ 파이썬 ``json`` 모듈과 호환되는 빠른 JSON 구현체.
  설치돼 있으면 JSON 처리에 사용한다.
* `python-dotenv`_ ``flask`` 명령 실행 시 :ref:`dotenv` 지원을
  가능하게 해 줌.
* `Watchdog`_ 개발용 서버에 빠르고 효율적인 재적재 기능 제공.

.. _Blinker: https://pythonhosted.org/blinker/
.. _SimpleJSON: https://simplejson.readthedocs.io/
.. _python-dotenv: https://github.com/theskumar/python-dotenv#readme
.. _watchdog: https://pythonhosted.org/watchdog/

가상 환경
---------

개발 및 도입 시에 가상 환경을 사용해 프로젝트 의존성을 관리하자.

가상 환경이 해결해 주는 문제가 뭘까? 파이썬 프로젝트가 많아질수록
여러 버전의 파이썬 라이브러리들을, 또 때로는 여러 버전의 파이썬을
다뤄야 할 가능성이 높다. 그런데 어떤 프로젝트에 쓸 라이브러리
최신 버전 때문에 다른 프로젝트에서 호환성 문제가 생길 수 있다.

가상 환경은 각 프로젝트마다 하나씩 있는 독립적인 파이썬 라이브러리
모음이다. 어떤 프로젝트를 위해 패키지를 설치해도 다른 프로젝트나
운영 체제 패키지에 영향을 주지 않게 된다.

파이썬 3에는 가상 환경을 만들기 위한 :mod:`venv` 모듈이 기본으로
딸려 있다. 따라서 파이썬 요새 버전을 쓰고 있다면 다음 절을 계속
읽어 나가면 된다.

파이썬 2를 쓰고 있다면 먼저 :ref:`install-install-virtualenv` 절을
보자.

.. _install-create-env:

환경 만들기
~~~~~~~~~~~

프로젝트 폴더를 만들고 그 안에 :file:`venv` 폴더를 만든다.

.. code-block:: sh

    mkdir myproject
    cd myproject
    python3 -m venv venv

윈도우:

.. code-block:: bat

    py -3 -m venv venv

파이썬 2를 사용 중이라 virtualenv를 설치해야 했던 경우라면 대신
다음 명령을 써야 한다.

.. code-block:: sh

    python2 -m virtualenv venv

윈도우:

.. code-block:: bat

    \Python27\Scripts\virtualenv.exe venv

.. _install-activate-env:

환경 활성화하기
~~~~~~~~~~~~~~~

프로젝트에서 작업을 하기 전에 해당 환경을 활성화하자.

.. code-block:: sh

    . venv/bin/activate

윈도우:

.. code-block:: bat

    venv\Scripts\activate

셸 프롬프트에 활성화된 환경 이름이 나오게 된다.

플라스크 설치
-------------

활성 환경 내에서 다음 명령으로 플라스크를 설치하자.

.. code-block:: sh

    pip install Flask

이제 플라스크가 설치됐다. :doc:`/quickstart` 절을 읽어 보거나
:doc:`문서 개요 </index>`\로 가면 된다.

모험하며 살기
~~~~~~~~~~~~~

출시되기도 전의 최신 플라스크 코드를 쓰고 싶다면 마스터 브랜치 코드로
설치 및 업데이트 하면 된다.

.. code-block:: sh

    pip install -U https://github.com/pallets/flask/archive/master.tar.gz

.. _install-install-virtualenv:

virtualenv 설치
---------------

파이썬 2를 쓰고 있다면 venv 모듈이 없을 것이다. 대신 `virtualenv`_
모듈을 설치하자.

리눅스에서는 패키지 관리자를 통해 virtualenv를 설치할 수 있다.

.. code-block:: sh

    # Debian, Ubuntu
    sudo apt-get install python-virtualenv

    # CentOS, Fedora
    sudo yum install python-virtualenv

    # Arch
    sudo pacman -S python-virtualenv

맥 OS X나 윈도우라면 `get-pip.py`_ 파일을 내려받아서 다음을
실행하자.

.. code-block:: sh

    sudo python2 Downloads/get-pip.py
    sudo python2 -m pip install virtualenv

윈도우면 관리자 권한으로 다음을 실행하자.

.. code-block:: bat

    \Python27\python.exe Downloads\get-pip.py
    \Python27\python.exe -m pip install virtualenv

이제 위로 돌아가 :ref:`install-create-env` 절을 따라가면 된다.

.. _virtualenv: https://virtualenv.pypa.io/
.. _get-pip.py: https://bootstrap.pypa.io/get-pip.py
