정적 파일
=========

인증 뷰와 템플릿이 이제 동작하긴 하지만 아주 밋밋해 보인다.
작성한 HTML 구조에 `CSS`_\를 좀 더해서 스타일을 줄 수 있다.
스타일은 내용이 바뀌지 않으며, 그래서 템플릿이 아니라
*정적* 파일이다.

플라스크에는 자동으로 추가되는 ``static``\이라는 뷰가 있어서
``flaskr/static`` 디렉터리를 기준으로 한 경로를 받아서 파일을
내놓는다. ``base.html`` 템플릿에는 이미 ``style.css`` 파일
링크가 있다.

.. code-block:: html+jinja

    {{ url_for('static', filename='style.css') }}

CSS 말고 다른 종류의 정적 파일로 자바스크립트 함수, 로고
이미지 등이 있을 수 있다. 모두 ``flaskr/static`` 안에
집어넣고 ``url_for('static', filename='...')``\이라고
참조한다.

따라하기에서 CSS 작성 방법을 다룰 건 아니므로 그냥 다음
내용을 ``flaskr/static/style.css`` 파일로 복사하면 된다.

.. code-block:: css
    :caption: ``flaskr/static/style.css``

    html { font-family: sans-serif; background: #eee; padding: 1rem; }
    body { max-width: 960px; margin: 0 auto; background: white; }
    h1 { font-family: serif; color: #377ba8; margin: 1rem 0; }
    a { color: #377ba8; }
    hr { border: none; border-top: 1px solid lightgray; }
    nav { background: lightgray; display: flex; align-items: center; padding: 0 0.5rem; }
    nav h1 { flex: auto; margin: 0; }
    nav h1 a { text-decoration: none; padding: 0.25rem 0.5rem; }
    nav ul  { display: flex; list-style: none; margin: 0; padding: 0; }
    nav ul li a, nav ul li span, header .action { display: block; padding: 0.5rem; }
    .content { padding: 0 1rem 1rem; }
    .content > header { border-bottom: 1px solid lightgray; display: flex; align-items: flex-end; }
    .content > header h1 { flex: auto; margin: 1rem 0 0.25rem 0; }
    .flash { margin: 1em 0; padding: 1em; background: #cae6f6; border: 1px solid #377ba8; }
    .post > header { display: flex; align-items: flex-end; font-size: 0.85em; }
    .post > header > div:first-of-type { flex: auto; }
    .post > header h1 { font-size: 1.5em; margin-bottom: 0; }
    .post .about { color: slategray; font-style: italic; }
    .post .body { white-space: pre-line; }
    .content:last-child { margin-bottom: 0; }
    .content form { margin: 1em 0; display: flex; flex-direction: column; }
    .content label { font-weight: bold; margin-bottom: 0.5em; }
    .content input, .content textarea { margin-bottom: 1em; }
    .content textarea { min-height: 12em; resize: vertical; }
    input.danger { color: #cc2f2e; }
    input[type=submit] { align-self: start; min-width: 10em; }

:gh:`예시 코드 <examples/tutorial/flaskr/static/style.css>`\에서
여백이 좀 있는 버전을 볼 수 있다.

http://127.0.0.1:5000/auth/login\으로 가 보면 페이지가 아래
스크린샷처럼 보일 것이다.

.. image:: flaskr_login.png
    :align: center
    :class: screenshot
    :alt: screenshot of login page

CSS에 대해선 `모질라의 문서 <CSS_>`_\에서 더 자세한 내용을
볼 수 있다. 정적 파일을 바꾸면 브라우저 페이지를 새로 고쳐야
한다. 바뀐 내용이 나오지 않으면 브라우저의 캐시를 비워 보자.

.. _CSS: https://developer.mozilla.org/docs/Web/CSS

:doc:`blog` 절로 이어진다.
