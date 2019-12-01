.. rst-class:: hide-header

플라스크 열기
=============

.. image:: _static/flask-logo.png
    :alt: 플라스크: 웹 개발, 한 번에 한 방울씩
    :align: center
    :target: https://palletsprojects.com/p/flask/

플라스크(Flask) 문서를 읽게 된 걸 환영한다. :ref:`installation` 절로
시작한 다음 :ref:`quickstart` 절에서 전체적으로 둘러보면 된다. 더 자세한
:ref:`tutorial` 절에선 작지만 완전한 응용을 플라스크로 만드는 방법을
보게 된다. 그리고 :ref:`patterns` 절에는 많이 쓰는 패턴들이 설명돼 있다.
문서 나머지에선 플라스크의 구성 요소 각각을 자세히 설명하며 :ref:`api`
절에는 완전한 참조 문서가 있다.

플라스크를 쓰려면 `Jinja`_ 템플릿 엔진과 `Werkzeug`_ WSGI 툴킷이
필요하다. 다음 사이트에서 그 라이브러리들의 문서를 볼 수 있다.

- `Jinja 문서 <http://jinja.pocoo.org/docs>`_
- `Werkzeug 문서 <https://werkzeug.palletsprojects.com/>`_

.. _Jinja: https://www.palletsprojects.com/p/jinja/
.. _Werkzeug: https://www.palletsprojects.com/p/werkzeug/


사용자 안내서
-------------

주로 산문체인 이 부분에서는 플라스크에 대한 몇 가지 배경 정보로
시작해서 플라스크로 웹 개발을 하는 방법을 차례로 설명한다.

.. toctree::
   :maxdepth: 2

   foreword
   advanced_foreword
   installation
   quickstart
   tutorial/index
   templating
   testing
   errorhandling
   logging
   config
   signals
   views
   appcontext
   reqcontext
   blueprints
   extensions
   cli
   server
   shell
   patterns/index
   deploying/index
   becomingbig


API 참조 문서
-------------

특정 함수나 클래스, 메소드에 대한 내용을 찾는다면 여기를 보면 된다.

.. toctree::
   :maxdepth: 2

   api


추가 문서
---------

관심 있는 이들을 위한 설계 문서, 법률 정보, 변경 이력 등이 있다.

.. toctree::
   :maxdepth: 2

   design
   htmlfaq
   security
   unicode
   extensiondev
   styleguide
   upgrading
   changelog
   license
   contributing
