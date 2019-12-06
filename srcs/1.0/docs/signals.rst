.. _signals:

시그널
======

.. versionadded:: 0.6

플라스크 0.6부터 시그널 전송을 통합적으로 지원한다. 탁월한
`blinker`_ 라이브러리로 기능을 제공하는데 사용 가능하지
않을 때는 매끄럽게 기능이 꺼진다.

시그널이 뭘까? 시그널을 쓰면 코어 프레임워크나 다른 플라스크
확장 어딘가에서 동작이 일어날 때 알림을 보내기 때문에 응용을
분리하는 게 쉬워진다. 요컨데 시그널을 쓰면 송신자가 뭔가
있어났다는 걸 구독자에게 알릴 수 있다.

플라스크에는 두 가지 시그널이 있고 다른 확장들에서 더 많은
시그널을 제공할 수도 있다. 그리고 염두에 둘 건 시그널은
구독자에게 알림을 주기 위한 것이므로 구독자에서 데이터를
변경하도록 하지 말아야 한다는 점이다. 그리고 일부 내장
데코레이터와 같은 역할을 하는 것처럼 보이는 시그널이 있지만
(가령 :data:`~flask.request_started`\는
:meth:`~flask.Flask.before_request`\와 아주 비슷하다.)
동작 방식에 차이가 있다. 예를 들어 코어의
:meth:`~flask.Flask.before_request` 핸들러는 특정 순서로
실행되며 응답을 반환해서 요청을 이른 시점에 중단시킬 수
있다. 반면 모든 시그널 핸들러는 규정 안 된 순서로 실행되고
어떤 데이터도 변경하지 않는다.

핸들러에 대한 시그널의 큰 장점은 안전하게 짧은 시간 동안만
구독할 수 있다는 점이다. 이런 일시 구독이 예를 들어 유닛
테스트에 도움이 된다. 가령 요청 처리 과정에서 어떤 템플릿을
렌더링 했는지 알고 싶다면 시그널이 바로 그걸 할 수 있게
해 준다.

시그널 구독하기
---------------

시그널을 구독하려면 시그널의 :meth:`~blinker.base.Signal.connect`
메소드를 사용하면 된다. 첫 번째 인자는 시그널이 나왔을 때
호출돼야 할 함수고, 선택적인 두 번째 인자는 송신자를
나타낸다. 시그널 구독을 해제하려면
:meth:`~blinker.base.Signal.disconnect` 메소드를 쓰면 된다.

코어 플라스크의 시그널에서 송신자는 시그널을 날린 응용이다.
시그널에 구독할 때는 진짜로 모든 응용에서 온 시그널을 듣고
싶은 게 아니라면 꼭 송신자를 지정해 줘야 한다. 확장을
개발하려 할 때 특히 그렇다.

예를 들어 다음은 유닛 테스트에서 쓸 수 있는 헬퍼 문맥 관리자인데
어떤 템플릿이 렌더링 됐고 그 템플릿으로 어떤 변수들이 전달됐는지
알아낼 수 있다. ::

    from flask import template_rendered
    from contextlib import contextmanager

    @contextmanager
    def captured_templates(app):
        recorded = []
        def record(sender, template, context, **extra):
            recorded.append((template, context))
        template_rendered.connect(record, app)
        try:
            yield recorded
        finally:
            template_rendered.disconnect(record, app)

다음처럼 테스트 클라이언트와 쉽게 연계할 수 있다. ::

    with captured_templates(app) as templates:
        rv = app.test_client().get('/')
        assert rv.status_code == 200
        assert len(templates) == 1
        template, context = templates[0]
        assert template.name == 'index.html'
        assert len(context['items']) == 10

플라스크에서 시그널에 새 인자를 도입하더라도 호출이 실패하지
않도록 추가 인자 ``**extra``\를 꼭 받도록 해야 한다.

그러면 ``with`` 블록 내에서 응용 `app`\이 일으키는 코드 내
템플릿 렌더링이 모두 `templates` 변수에 기록된다. 템플릿이
렌더링 될 때마다 템플릿 객체와 문맥이 리스트에 덧붙는다.

추가로 편리한 헬퍼 메소드(:meth:`~blinker.base.Signal.connected_to`)가
있어서 자체 문맥 관리자로 일시적으로 시그널을 구독할 수 있다.
문맥 관리자의 반환 값을 앞서처럼 지정할 수 없기 때문에 인자로
리스트를 줘야 한다. ::

    from flask import template_rendered

    def captured_templates(app, recorded, **extra):
        def record(sender, template, context):
            recorded.append((template, context))
        return template_rendered.connected_to(record, app)

그러면 위의 예가 다음처럼 된다. ::

    templates = []
    with captured_templates(app, templates, **extra):
        ...
        template, context = templates[0]

.. admonition:: Blinker API 변경

   :meth:`~blinker.base.Signal.connected_to` 메소드는 Blinker 버전
   1.1에서 등장했다.

시그널 만들기
-------------

응용에서 시그널을 쓰고 싶은 경우 blinker 라이브러리를 직접 사용할
수 있다. 일반적인 사용 방식은 자체 :class:`~blinker.base.Namespace`
안에 이름 있는 시그널을 만드는 것이다. 대부분의 경우 이 방식을
권장한다. ::

    from blinker import Namespace
    my_signals = Namespace()

그러면 다음처럼 새 시그널을 만들 수 있다. ::

    model_saved = my_signals.signal('model-saved')

시그널 이름은 유일성을 제공할 뿐 아니라 디버깅을 쉽게 만들어
준다. :attr:`~blinker.base.NamedSignal.name` 속성으로 시그널
이름에 접근할 수 있다.

.. admonition:: 확장 개발자에게

   플라스크 확장을 작성하는데 blinker가 설치돼 있지 않은 경우를
   매끄럽게 처리하고 싶다면 :class:`flask.signals.Namespace`
   클래스를 쓰면 된다.

.. _signals-sending:

시그널 보내기
-------------

시그널을 내보내고 싶으면 :meth:`~blinker.base.Signal.send` 메소드를
호출하면 된다. 첫 번째 인자로 송신자를 받고 선택적으로 몇 가지
키워드 인자를 받는데 그 인자들이 시그널 구독자에게 전달된다. ::

    class Model(object):
        ...

        def save(self):
            model_saved.send(self)

송신자를 잘 골라야 한다. 시그널을 내보내는 클래스가 있다면 송신자로
``self``\를 주면 된다. 임의 함수에서 시그널을 내보이는 경우라면
``current_app._get_current_object()``\를 송신자로 줄 수 있다.

.. admonition:: 프록시를 송신자로 주기

   시그널 송신자로 절대 :data:`~flask.current_app`\을 주지 말자.
   대신 ``current_app._get_current_object()``\를 써야 한다.
   :data:`~flask.current_app`\이 프록시일 뿐 진짜 응용 객체가
   아니기 때문이다.


시그널과 플라스크 요청 문맥
---------------------------

시그널에서는 수신 시 :ref:`request-context`\을 완전히 지원한다.
:data:`~flask.request_started`\와 :data:`~flask.request_finished`
사이에 문맥 로컬인 변수들이 계속 사용 가능하므로 :class:`flask.g`
몇 기타 변수들을 필요한 대로 쓸 수 있다. :ref:`signals-sending`\에서
설명하는 제약들과 :data:`~flask.request_tearing_down` 시그널에
유의하자.


데코레이터 기반 시그널 구독
---------------------------

Blinker 1.1 사용시 :meth:`~blinker.base.NamedSignal.connect_via`
데코레이터를 써서 쉽게 시그널을 구독할 수도 있다. ::

    from flask import template_rendered

    @template_rendered.connect_via(app)
    def when_template_rendered(sender, template, context, **extra):
        print 'Template %s is rendered with %s' % (template.name, context)

핵심 시그널
-----------

내장 시그널 전체 목록은 :ref:`core-signals-list` 참고.


.. _blinker: https://pypi.org/project/blinker/
