.. currentmodule:: flask

템플릿
======

응용에 인증 뷰를 작성하기는 했지만 서버를 돌리고 아무 URL이나
들어가 보면 ``TemplateNotFound`` 오류가 나오게 된다. 뷰에서
:func:`render_template`\을 호출하고 있는데 템플릿을 아직
작성하지 않았기 때문이다. 템플릿 파일은 ``flaskr`` 패키지 내의
``templates`` 디렉터리에 저장하게 된다.

템플릿은 정적 데이터와 함께 동적 데이터 플레이스홀더를 담고
있는 파일이다. 구체적 데이터로 템플릿을 렌더링 해서 최종 문서를
만들어 낸다. 플라스크에서는 `Jinja`_ 템플릿 라이브러리를 써서
템플릿을 렌더링 한다.

응용에서 템플릿을 이용해 `HTML`_\을 만들면 그 결과가 사용자의
브라우저에 표시된다. 플라스크에서는 HTML 템플릿에 표시하는
모든 데이터를 *자동 이스케이프* 하도록 Jinja를 설정해서 쓴다.
즉 사용자 입력을 마음놓고 표시할 수 있다는 뜻이다. 입력한 문자
중에 ``<``\이나 ``>``\처럼 HTML을 꼬이게 만들 수 있는 게 있으면
브라우저에서 똑같이 보이지만 부작용을 일으키지 않는 *안전한*
값으로 *이스케이프* 한다.

Jinja는 생김새와 동작이 파이썬과 거의 비슷하다. 특수한 구분자를
써서 템플릿의 정적 데이터와 Jinja 문법을 구별한다. ``{{``\와
``}}`` 사이에 있는 식은 최종 문서로 출력된다. ``{%``\와 ``%}``\는
``if``\나 ``for`` 같은 흐름 제어문을 나타낸다. 그리고
파이썬과 달리 들여쓰기가 아니라 시작 태그와 끝 태그로 블록을
나타낸다. 블록 내의 정적 텍스트에서 들여쓰기가 바뀔 수도 있기
때문이다.

.. _Jinja: http://jinja.pocoo.org/docs/templates/
.. _HTML: https://developer.mozilla.org/docs/Web/HTML


기본 구조
---------

응용의 각 페이지에는 동일한 기본 구조가 있고 몸통 부분만 달라지게
된다. 따라서 템플릿마다 전체 HTML 구조를 작성하는 대신 각
템플릿이 기반 템플릿을 *확장해서* 특정 부분만 바꾸게 된다.

.. code-block:: html+jinja
    :caption: ``flaskr/templates/base.html``

    <!doctype html>
    <title>{% block title %}{% endblock %} - Flaskr</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <nav>
      <h1>Flaskr</h1>
      <ul>
        {% if g.user %}
          <li><span>{{ g.user['username'] }}</span>
          <li><a href="{{ url_for('auth.logout') }}">Log Out</a>
        {% else %}
          <li><a href="{{ url_for('auth.register') }}">Register</a>
          <li><a href="{{ url_for('auth.login') }}">Log In</a>
        {% endif %}
      </ul>
    </nav>
    <section class="content">
      <header>
        {% block header %}{% endblock %}
      </header>
      {% for message in get_flashed_messages() %}
        <div class="flash">{{ message }}</div>
      {% endfor %}
      {% block content %}{% endblock %}
    </section>

템플릿에서는 기본적으로 :data:`g`\를 쓸 수 있다. ``g.user``\가
(``load_logged_in_user``\에서) 설정됐는지에 따라서 사용자 이름과
로그아웃 링크가 표시되든지 등록 및 로그인 링크가 표시된다.
:func:`url_for` 역시 기본적으로 사용 가능하다. 뷰로 가는 URL을
직접 작성하지 않고 이 함수로 생성한다.

템플릿의 페이지 제목과 본문 사이에서 :func:`get_flashed_messages`\가
반환하는 메시지들에 루프를 돈다. 뷰에서 오류 메시지를 보여 주기
위해 :func:`flash`\를 사용했는데 바로 이 코드에서 그 메시지를
표시한다.

여기 정의된 세 가지 블록을 다른 템플릿에서 각기 바꾸게 된다.

#.  ``{% block title %}``: 브라우저 탭과 창 제목에 표시되는
    제목을 바꾼다.

#.  ``{% block header %}``: ``title``\과 비슷하되 페이지에서
    표시되는 제목을 바꾼다.

#.  ``{% block content %}``: 여기에 로그인 양식이나 블로그 글
    같은 각 페이지의 내용물이 들어간다.

기반 템플릿은 ``templates`` 디렉터리 바로 안에 있다. 파일들을
깔끔하게 유지하기 위해 청사진을 위한 템플릿들은 청사진 이름의
디렉터리로 들어가게 된다.


등록
----

.. code-block:: html+jinja
    :caption: ``flaskr/templates/auth/register.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Register{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="username">Username</label>
        <input name="username" id="username" required>
        <label for="password">Password</label>
        <input type="password" name="password" id="password" required>
        <input type="submit" value="Register">
      </form>
    {% endblock %}

``{% extends 'base.html' %}``\라고 하면 Jinja에서 기반 템플릿의
블록들을 이 템플릿을 가지고 바꿔 준다. 표시할 내용이 모두
``{% block %}`` 태그 안에 있어야 하며, 그 태그가 기반 템플릿의
블록들을 대체한다.

여기서 유용한 패턴 하나가 ``{% block title %}``\을
``{% block header %}`` 안에 넣는 것이다. 이렇게 하면 제목 블록을
설정하고서 그 값을 헤더 블록으로 출력하는데, 그러면 두 번 써 줄
필요 없이 창과 페이지에 같은 제목을 쓸 수 있다.

여기 ``input`` 태그에서 ``required`` 속성을 쓰고 있다. 이렇게 하면
그 필드가 채워지지 않으면 브라우저가 양식을 제출하지 않는다.
하지만 사용자가 이 속성을 지원하지 않는 구식 브라우저를 쓰고
있거나 브라우저 아닌 뭔가로 요청을 하고 있을 수 있으므로
플라스크 뷰에서는 여전히 데이터 검사를 해야 한다. 클라이언트에서
일부 검사를 해 주는 경우에도 서버에서는 항상 데이터를 완전히
검사해야 한다.


로그인
------

제목과 제출 버튼만 빼면 등록 템플릿과 동일하다.

.. code-block:: html+jinja
    :caption: ``flaskr/templates/auth/login.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Log In{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="username">Username</label>
        <input name="username" id="username" required>
        <label for="password">Password</label>
        <input type="password" name="password" id="password" required>
        <input type="submit" value="Log In">
      </form>
    {% endblock %}


사용자 등록
-----------

인증 템플릿들을 작성했으니 이제 사용자 등록을 할 수 있다. 서버가
잘 돌고 있는지 확인하고서 (아니라면 ``flask run``)
http://127.0.0.1:5000/auth/register\로 가 보자.

양식을 채우지 않고 "Register" 버튼을 누르면 브라우저의 오류 메시지를
보게 된다. ``register.html`` 템플릿에서 ``required`` 속성을 없애고
다시 "Register"를 눌러 보자. 그러면 브라우저에서 오류를 표시하는 게
아니라 페이지 새로고침이 일어나고서 뷰의 :func:`flash`\에서 온 오류가
보이게 된다.

사용자 이름과 패스워드를 채우고 나면 로그인 페이지로 재지향 된다.
사용자 이름을 틀리게 넣거나 사용자 이름은 맞고 패스워드는 틀리게
넣어 보자. 로그인을 하고 나면 오류를 보게 될 텐데, 아직 ``index``
뷰가 없기 때문이다.

:doc:`static` 절로 이어진다.
