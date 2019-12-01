.. _application-errors:

응용 오류
=========

.. versionadded:: 0.3

응용에도 문제가 생기고 서버에도 문제가 생긴다. 늦든 빠르든 운용 중에
예외 상황을 보게 된다. 심지어 작성한 코드가 100% 올바르다고 해도
때로 예외 상황을 보게 된다. 왜냐고? 관련된 다른 모든 곳에서 문제가
생길 것이기 때문이다. 다음은 코드가 완벽해도 서버 오류로 이어질 수
있는 몇 가지 경우다.

-   클라이언트가 이르게 요청을 종료했고 응용에서 입력 데이터를
    계속 읽어 들이려 했다.
-   데이터베이스 서버가 과부하 때문에 질의를 처리할 수 없다.
-   파일 시스템이 가득 찼다.
-   하드 드라이브가 깨졌다.
-   백엔드 서버 과부하.
-   사용하는 라이브러리 내의 프로그래밍 오류.
-   서버와 다른 시스템 간의 네트워크 연결 장애.

그리고 이건 만날 수 있는 이슈들 중에 일부를 뽑아 본 것일 뿐이다.
그럼 이런 류의 문제들에 어떻게 대처해야 할까? 응용이 운용 모드로
돌고 있으면 플라스크는 기본적으로 아주 간단한 페이지를 표시해
주고 예외를 :attr:`~flask.Flask.logger`\로 기록한다.

하지만 할 수 있는 게 더 있다. 여기선 오류를 더 잘 다루기 위한
방법을 몇 가지 다룰 것이다.

오류 로깅 도구
--------------

오류 알림 이메일은 아주 심각한 것만 보낸다 해도 일정 이상의
사용자가 오류를 일으키면 감당이 어려워질 수 있고 로그 파일은 보통
절대 들여다보지 않는다. 그래서 우리는 응용 오류를 다루는 데
`Sentry <https://sentry.io/welcome/>`_\를 권장한다.
`GitHub에 <https://github.com/getsentry/sentry>`__ 오픈 소스
프로젝트로 있고 무료로 써 볼 수 있는 `호스팅 버전
<https://sentry.io/signup/>`_\도 있다. Sentry는 중복 오류를
모아 주고, 디버깅을 위해 전체 스택 트레이스와 지역 변수들을
수집해 주며, 신규 오류나 발생 빈도 기준에 따라 메일을 보내 준다.

Sentry를 쓰려면 추가로 `flask` 의존성을 줘서 `raven` 클라이언트를
설치해야 한다. ::

    $ pip install raven[flask]

그리고 플라스크 앱에 다음을 추가하면 된다. ::

    from raven.contrib.flask import Sentry
    sentry = Sentry(app, dsn='YOUR_DSN_HERE')

팩토리를 쓰고 있다면 좀 있다 초기화를 할 수도 있다. ::

    from raven.contrib.flask import Sentry
    sentry = Sentry(dsn='YOUR_DSN_HERE')

    def create_app():
        app = Flask(__name__)
        sentry.init_app(app)
        ...
        return app

`YOUR_DSN_HERE` 값을 Sentry를 설치해서 얻은 DSN 값으로 바꿔 줘야
한다.

이렇게 하고 나면 장애가 자동으로 Sentry로 보고되고 거기서 오류
알림을 받을 수 있다.

.. _error-handlers:

오류 핸들러
-----------

오류가 발생했을 때 사용자에게 따로 만든 오류 페이지를 보여 주고
싶을 수도 있다. 오류 핸들러를 등록해서 그렇게 할 수 있다.

오류 핸들러는 응답을 반환하는 평범한 뷰 함수이다. 하지만 라우트에
등록되는 게 아니라 요청 처리 시도 중 발생할 수 있을 예외나
HTTP 상태 코드에 등록된다.

등록
````

함수를 :meth:`~flask.Flask.errorhandler`\로 꾸며서 핸들러를
등록하면 된다. 아니면 편할 때
:meth:`~flask.Flask.register_error_handler`\를 써서 등록할
수도 있다. 응답 반환 시 오류 코드 설정하는 걸 잊지 말자. ::

    @app.errorhandler(werkzeug.exceptions.BadRequest)
    def handle_bad_request(e):
        return 'bad request!', 400

    # 데코레이터 안 쓰고 등록하기
    app.register_error_handler(400, handle_bad_request)

:exc:`werkzeug.exceptions.HTTPException`\의 서브클래스로
:exc:`~werkzeug.exceptions.BadRequest` 등이 있으며 핸들러 등록 시
HTTP 코드를 대신 쓸 수도 있다. (``BadRequest.code == 400``)

Werkzeug가 모르는 비표준 HTTP 코드는 코드 값으로 등록할 수
없다. 그럴 때는 적당한 코드로
:class:`~werkzeug.exceptions.HTTPException`\의 서브클래스를
정의해서 등록한 다음 그 예외 클래스를 던지면 된다. ::

    class InsufficientStorage(werkzeug.exceptions.HTTPException):
        code = 507
        description = 'Not enough storage space.'

    app.register_error_handler(InsuffcientStorage, handle_507)

    raise InsufficientStorage()

:exc:`~werkzeug.exceptions.HTTPException` 서브클래스나 HTTP
상태 코드뿐 아니라 어떤 예외 클래스에도 핸들러를 등록할 수 있다.
특정 클래스에 대해서, 또는 어느 부모 클래스의 모든 서브클래스에
대해 핸들러를 등록할 수 있다.

처리
````

플라스크에서 요청 처리 중에 예외를 잡으면 먼저 코드로 검색을 한다.
그 코드에 핸들러가 등록돼 있지 않으면 클래스 계층 구조로 검색을
해서 가장 구체적인 클래스에 대한 핸들러를 선택한다. 아무 핸들러도
등록돼 있지 않은 경우 :class:`~werkzeug.exceptions.HTTPException`
서브클래스들은 코드 값에 대한 범용 메시지를 보여 주며 다른
예외들은 범용 500 Internal Server Error로 바뀐다.

예를 들어 :exc:`ConnectionRefusedError` 인스턴스를 잡았는데
:exc:`ConnectionError`\와 :exc:`ConnectionRefusedError`\에
각각 핸들러가 등록돼 있으면 더 구체적인
:exc:`ConnectionRefusedError`\의 핸들러를 예외 인스턴스로
호출해서 응답을 만들어 낸다.

예외를 일으킨 요청을 청사진에서 처리하고 있던 경우 청사진에
등록된 핸들러가 응용에 전역으로 등록된 핸들러보다 우선한다.
다만 청사진에서는 404 라우팅 오류를 처리할 수 없는데, 404는
청사진을 정하기 전 라우팅 단계에서 발생하기 때문이다.

.. versionchanged:: 0.11

   등록 순서가 아니라 등록된 예외 클래스의 구체성을 가지고
   핸들러 우선순위를 정한다.

로그
----

관리자에게 이메일을 보내는 등의 예외 기록 방법에 대해선
:ref:`logging` 절을 보라.


응용 오류 디버깅
================

실제 운용하는 응용에는 :ref:`application-errors` 절에서 설명한
것처럼 로그 및 알림 설정을 해야 한다. 이 절에는 배치된 구성을
디버깅 하고 제대로 된 파이썬 디버거로 문제를 파고들려 할 때를
위한 조언들이 있다.


미심쩍으면 직접 실행해 보자
---------------------------

응용을 실제로 돌리려는 중에 문제가 생겼는가? 호스트에 셸 접근이
가능하다면 운용 환경의 셸에서 응용을 직접 실행해서 확인해 보면
된다. 권한 문제를 피하기 위해 배치 구성과 같은 사용자 계정으로
실행하도록 하자. 운용 호스트에서 `debug=True`\로 해서 플라스크
내장 개발 서버를 쓸 수도 있는데, 설정 이슈를 잡는 데 유용하다.
단 **통제된 환경에서 일시적으로만** 그렇게 해야 한다.
`debug=True`\로 운용해선 안 된다.


.. _working-with-debuggers:

디버거 이용하기
---------------

플라스크에서는 깊이 파고들어서 코드 실행 추적 등을 할 수 있도록
기본으로 디버거를 제공한다. (:ref:`debug-mode` 참고.) 만약 다른
파이썬 디버거를 쓰고 싶다면 디버거들이 서로 간섭한다는 점에
유의해야 한다. 좋아하는 다른 디버거를 쓰려면 몇 가지 옵션을
설정해야 한다.

* ``debug``        - 디버그 모드를 켜고 예외를 잡을지 여부
* ``use_debugger`` - 플라스크 내장 디버거를 쓸지 여부
* ``use_reloader`` - 예외 시 새로고침 하고 프로세스를 포크 할지 여부

``debug``\가 True여야 (즉 예외를 잡도록 해야) 다른 두 옵션에
뭔가 값을 설정할 수 있다.

디버깅에 Aptana/Eclipse를 쓸 거라면 ``use_debugger``\와
``use_reloader`` 모두 False로 설정해 줘야 한다.

유용할 수도 있는 설정 패턴은 config.yaml에서 다음처럼 설정하는
것이다. (물론 블록은 응용에 맞게 바꿔 줘야 한다.)::

   FLASK:
       DEBUG: True
       DEBUG_WITH_APTANA: True

그러고서 응용 시작점(main.py)에서 다음처럼 할 수 있다. ::

   if __name__ == "__main__":
       # aptana가 오류를 받게 하려면 use_debugger=False 설정
       app = create_app(config="config.yaml")

       if app.debug: use_debugger = True
       try:
           # 외부 디버거 요청 시 플라스크의 디버거 끄기
           use_debugger = not(app.config.get('DEBUG_WITH_APTANA'))
       except:
           pass
       app.run(use_debugger=use_debugger, debug=app.debug,
               use_reloader=use_debugger, host='0.0.0.0')
