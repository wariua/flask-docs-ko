설치 가능하게 만들기
====================

프로젝트를 설치 가능하게 만든다는 건 *배포* 파일을 빌드 해서
그걸 다른 환경에 설치할 수 있다는 뜻이다. 지금 환경에
플라스크를 설치했던 것처럼 말이다. 그러면 지금 만드는
프로젝트가 배치되는 과정이 여느 라이브러리 설치와 비슷하게
되고, 그래서 표준 파이썬 도구들로 모든 걸 관리할 수 있게 된다.

이 따라하기를 통해선, 또는 파이썬을 처음 써보는 경우라면
바로 생각하지 못할 수도 있겠지만 설치 방식에는 다른 이점들도
있다.

*   현재 파이썬과 플라스크에서 ``flaskr`` 패키지를 이용할 수
    있는 건 프로젝트의 디렉터리에서 실행하고 있기 때문이다.
    패키지를 설치하면 어디서 실행하든 임포트 할 수 있다.

*   다른 패키지들과 마찬가지로 프로젝트의 의존성을 관리할 수
    있다. 그래서 ``pip install yourproject.whl``\이라고 하면
    필요한 다른 패키지들이 설치된다.

*   테스트 도구들에서 테스트 환경을 개발 환경과 격리시킬
    수 있다.

.. note::
    따라하기 후반에서야 이 내용을 소개하고 있지만 향후
    프로젝트에서는 항상 이 작업부터 시작해야 한다.


프로젝트 기술하기
-----------------

``setup.py`` 파일이 프로젝트와 거기 속한 파일들을 기술한다.

.. code-block:: python
    :caption: ``setup.py``

    from setuptools import find_packages, setup

    setup(
        name='flaskr',
        version='1.0.0',
        packages=find_packages(),
        include_package_data=True,
        zip_safe=False,
        install_requires=[
            'flask',
        ],
    )


``packages`` 는 포함시킬 디렉터리를 (그리고 그 안의 파이썬
파일들을) 파이썬에게 알려 준다. ``find_packages()``\가 자동으로
디렉터리를 찾아 주므로 직접 입력해 줄 필요가 없다. static 및
templates 디렉터리 같은 기타 파일들을 포함시키기 위해
``include_package_data``\를 True로 설정한다. 그 기타 데이터가
뭔지 파이썬에서 알려면 ``MANIFEST.in``\이라는 또 다른 파일이
필요하다.

.. code-block:: none
    :caption: ``MANIFEST.in``

    include flaskr/schema.sql
    graft flaskr/static
    graft flaskr/templates
    global-exclude *.pyc

``static`` 및 ``templates`` 디렉터리 안의 모든 내용물과
``schema.sql`` 파일을 복사하되 바이트코드 파일들은 모두
제외하게 한다.

사용 파일 및 옵션에 대한 추가 설명은 `공식 패키징 안내서`_\를
보라.

.. _공식 패키징 안내서: https://packaging.python.org/tutorials/distributing-packages/


프로젝트 설치하기
-----------------

``pip``\를 써서 프로젝트를 가상 환경에 설치하자.

.. code-block:: none

    pip install -e .

이렇게 하면 pip가 현재 디렉터리에서 ``setup.py``\를 찾아서
*편집 가능* 모드로 (즉 *개발* 모드로) 설치한다. 편집 가능
모드에서는 로컬 코드에 변경 작업을 해 나가면서 의존성 같은
프로젝트 메타데이터를 바꾸는 경우에만 재설치를 해 주면 된다.

``pip list``\로 프로젝트가 설치돼 있는 걸 확인할 수 있다.

.. code-block:: none

    pip list

    Package        Version   Location
    -------------- --------- ----------------------------------
    click          6.7
    Flask          1.0
    flaskr         1.0.0     /home/user/Projects/flask-tutorial
    itsdangerous   0.24
    Jinja2         2.10
    MarkupSafe     1.0
    pip            9.0.3
    setuptools     39.0.1
    Werkzeug       0.14.1
    wheel          0.30.0

프로젝트를 실행하는 방식은 아무것도 달라지지 않는다. 마찬가지로
``FLASK_APP``\을 ``flaskr``\로 설정하고서 ``flask run``\으로
응용을 실행한다.

:doc:`tests` 절로 이어진다.
