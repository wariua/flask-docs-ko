.. _templates:

템플릿
======

플라스크에서는 템플릿 엔진으로 Jinja2를 이용한다. 다른 템플릿 엔진을
자유롭게 사용할 수도 있지만 그래도 플라스크 자체를 돌리기 위해
Jinja2를 설치하긴 해야 한다. 이 요건은 풍부한 확장들을 위해
필요하다. 확장에서는 Jinja2가 늘 있다고 믿을 수 있다.

이 절에선 Jinja2가 플라스크에 어떻게 통합돼 있는지만 아주 짧게
소개한다. 템플릿 엔진의 문법에 대해 알고 싶으면 공식
`Jinja2 템플릿 문서 <http://jinja.pocoo.org/docs/templates>`_\를
들여다보면 된다.

Jinja 설정
----------

따로 변경하지 않으면 플라스크에서 Jinja2를 다음처럼 설정한다.

-   :func:`~flask.templating.render_template`\을 쓸 때
    ``.html``, ``.htm``, ``.xml``, ``.xhtml``\로 끝나는 모든
    템플릿에서 자동 이스케이핑이 켜진다.
-   :func:`~flask.templating.render_template_string`\을 쓸 때
    모든 문자열에서 자동 이스케이핑이 켜진다.
-   템플릿에서 ``{% autoescape %}`` 태그를 써서 자동
    이스케이핑을 켜거나 끌 수 있다.
-   플라스크에서 기본으로 있는 값들에 더해서 두 가지 전역
    함수와 헬퍼들을 Jinja2 문맥에 집어넣는다.

표준 문맥
---------

기본적으로 Jinja2 템플릿 내에서 다음 전역 변수들을 쓸 수
있다.

.. data:: config
   :noindex:

   현재 설정 객체 (:data:`flask.config`)

   .. versionadded:: 0.6

   .. versionchanged:: 0.10
      이제 임포트 한 템플릿을 포함해 언제나 사용 가능하다.

.. data:: request
   :noindex:

   현재 요청 객체 (:class:`flask.request`). 활성 요청 문맥 없이
   템플릿을 그리는 경우에는 이 변수가 사용 가능하지 않다.

.. data:: session
   :noindex:

   현재 세션 객체 (:class:`flask.session`). 활성 요청 문맥 없이
   템플릿을 그리는 경우에는 이 변수가 사용 가능하지 않다.

.. data:: g
   :noindex:

   요청에 결속된 전역 변수 저장용 객체 (:data:`flask.g`). 활성
   요청 문맥 없이 템플릿을 그리는 경우에는 이 변수가 사용
   가능하지 않다.

.. function:: url_for
   :noindex:

   :func:`flask.url_for` 함수.

.. function:: get_flashed_messages
   :noindex:

   :func:`flask.get_flashed_messages` 함수.

.. admonition:: Jinja 문맥 동작 방식

   이 변수들이 템플릿의 문맥에 추가되는 것이지 전역 변수인 건
   아니다. 차이가 뭐냐면 임포트 된 템플릿의 문맥에서 기본적으로는
   보이지 않게 된다는 점이다. 이렇게 하는 건 성능을 고려해서이기도
   하고 명시성을 유지하기 위해서이기도 하다.

   그래서 어떻게 된다는 걸까? 만약 임포트 하려는 매크로가 있는데
   거기서 요청 객체에 접근해야 한다면 가능한 방법이 두 가지 있다.

   1.   요청을, 또는 관심 있는 요청 객체의 속성을 명시적인
        매개변수로 매크로에게 전달하기
   2.   "with context"로 매크로 임포트 하기

   문맥과 함께 임포트 하는 건 다음과 같이 한다.

   .. sourcecode:: jinja

      {% from '_helpers.html' import my_macro with context %}

표준 필터
---------

Jinja2 자체에서 제공하는 필터들에 더해서 다음 필터들을 Jinja2
내에서 이용할 수 있다.

.. function:: tojson
   :noindex:

   이 함수는 주어진 객체를 JSON 표현으로 변환한다. 예를 들어
   동적으로 자바스크립트를 생성하려 할 때 아주 유용하다.

   ``script`` 태그 내에선 어떤 이스케이핑도 이뤄져선 안 된다.
   그래서 플라스크 0.10 전에선 ``script`` 태그 내에서
   쓰려는 경우에 꼭 ``|safe``\를 써서 이스케이핑을 꺼 줘야 한다.

   .. sourcecode:: html+jinja

       <script type=text/javascript>
           doSomethingWith({{ user.username|tojson|safe }});
       </script>

자동 이스케이핑 제어하기
------------------------

자동 이스케이핑이란 특수 문자들을 자동으로 이스케이프 처리해
주는 것이다. HTML에서 (또는 XML, 그리고 XHTML에서) 특수 문자란
``&``, ``>``, ``<``, ``"``, ``'``\이다. 이 문자들이 문서 내에서
자체적으로 특별한 의미가 있기 때문에 텍스트로 사용하려면
그걸 "엔터티(entity)"라는 것으로 바꿔 줘야 한다. 그렇게 하지
않으면 텍스트에서 그 문자들을 쓸 수 없어서 사용자가 불편해질
뿐 아니라 보안 문제로 이어질 수도 있다. (:ref:`xss` 참고.)

하지만 때로 템플릿에서 자동 이스케이핑을 꺼야 할 때가 있을
것이다. 예를 들어 마크다운을 HTML로 변환하는 것처럼 안전한
HTML을 생성하는 시스템에서 온 거라면 페이지에 HTML을 집어넣고
싶은 경우가 있을 수 있다.

그렇게 할 수 있는 세 가지 방법이 있다.

-   파이썬 코드에서 HTML 문자열을 템플릿으로 전달하기 전에
    :class:`~flask.Markup` 객체로 감싸면 된다. 일반적으로
    권장하는 방식이다.
-   템플릿 내에서 문자열에 ``|safe`` 필터를 써서
    (``{{ myvariable|safe }}``) 안전한 HTML이라고 표시해
    주면 된다.
-   임시로 자동 이스케이프 시스템 자체를 끄면 된다.

템플릿 내에서 자동 이스케이프 시스템을 끄려면 ``{% autoescape
%}`` 블록을 쓰면 된다.

.. sourcecode:: html+jinja

    {% autoescape false %}
        <p>autoescaping is disabled here
        <p>{{ will_not_be_escaped }}
    {% endautoescape %}

이렇게 하는 경우에는 그 블록 내에서 쓰려는 변수들을 부디
아주 조심스럽게 다루길 바란다.

.. _registering-filters:

필터 등록하기
-------------

Jinja2에 자기만의 필터를 등록하고 싶다면 두 가지 방법이 있다.
응용의 :attr:`~flask.Flask.jinja_env`\에 직접 집어넣거나
:meth:`~flask.Flask.template_filter` 데코레이터를 쓰면 된다.

다음 두 방식은 동일하게 동작하며 둘 모두 객체를 뒤집는다. ::

    @app.template_filter('reverse')
    def reverse_filter(s):
        return s[::-1]

    def reverse_filter(s):
        return s[::-1]
    app.jinja_env.filters['reverse'] = reverse_filter

데코레이터를 쓰는 경우에 함수 이름을 필터 이름으로 할 거면
인자가 선택적이다. 등록을 하고 나면 그 필터를 템플릿 안에서
Jinja2의 내장 필터들처럼 쓸 수 있다. 예를 들어 문맥에
`mylist`\라는 파이썬 리스트가 있다고 하자. ::

    {% for x in mylist | reverse %}
    {% endfor %}


문맥 처리기
-----------

템플릿 문맥으로 새로운 변수들을 자동으로 집어넣기 위해
플라스크에는 문맥 처리기가 있다. 문맥 처리기는 템플릿을 채우기
전에 실행되며 템플릿 문맥으로 새 값들을 집어넣을 수 있다.
문맥 처리기는 딕셔너리를 반환하는 함수이며, 그 딕셔너리의 키와
값이 응용의 모든 템플릿에서 템플릿 문맥으로 합쳐진다. ::

    @app.context_processor
    def inject_user():
        return dict(user=g.user)

위 문맥 처리기는 `g.user`\의 값으로 `user`\라는 변수를 만들어서
템플릿 안에서 쓸 수 있게 해 준다. 템플릿 안에서 어차피 `g`\를
쓸 수 있으니까 그리 재밌는 예는 아니지만 어떻게 동작하는지를
보여 준다.

변수로 값만 가능한 건 아니다. (파이썬에서 함수를 전달하는 게
가능하므로) 문맥 처리기를 통해 템플릿에서 함수를 사용 가능하게
만들 수도 있다. ::

    @app.context_processor
    def utility_processor():
        def format_price(amount, currency=u'€'):
            return u'{0:.2f}{1}'.format(amount, currency)
        return dict(format_price=format_price)

위 문맥 처리기는 모든 템플릿에서 `format_price` 함수를 쓸 수
있게 해 준다. ::

    {{ format_price(0.33) }}

`format_price`\를 템플릿 필터 형태로 만들 수도 있을 것이다.
(:ref:`registering-filters` 참고.) 이 예는 문맥 처리기에서
어떻게 함수를 전달하는지 보여 주기 위한 것이다.
