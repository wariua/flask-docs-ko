.. _config:

설정 다루기
===========

응용에는 어떤 종류든 설정이 필요하다. 응용 환경에 따라서 디버그
모드 켜고 끄기, 비밀 키 설정하기, 기타 환경별 사항 설정하기
같은 다양한 설정을 바꾸고 싶을 수 있다.

플라스크가 설계된 방식에서는 일반적으로 응용이 시작할 때
설정이 있어야 한다. 코드 내에 설정을 하드코딩 할 수도 있고,
그게 작은 응용들에서는 사실 그리 나쁘지 않은 방법이긴 하지만
더 나은 다른 방법들이 있다.

설정을 어떻게 적재하든 상관없이 적재된 설정 값들을 담고 있는
설정 객체가 있다. 바로 :class:`~flask.Flask`\의
:attr:`~flask.Flask.config` 속성이다. 플라스크 자체에서 여기에
특정 설정 값들을 집어넣으며 확장들에서도 각자의 설정 값을
집어넣을 수 있다. 그리고 응용 설정을 둘 수 있는 곳이기도 하다.


설정 기본 사용법
----------------

:attr:`~flask.Flask.config`\는 사실 딕셔너리의 서브클래스이기
때문에 여느 딕셔너리처럼 변경할 수 있다. ::

    app = Flask(__name__)
    app.config['TESTING'] = True

특정 설정 값들은 :attr:`~flask.Flask` 객체로 전달되고, 그래서
그 객체에서 읽고 쓸 수 있다. ::

    app.testing = True

여러 키를 한 번에 바꾸려면 :meth:`dict.update` 메소드를
이용할 수 있다. ::

    app.config.update(
        TESTING=True,
        SECRET_KEY=b'_5#y2L"F4Q8z\n\xec]/'
    )


환경과 디버그 기능
------------------

설정 값들 중 :data:`ENV`\와 :data:`DEBUG`\는 좀 특별한데, 응용
작동이 시작된 후에 값이 바뀌면 동작이 비일관적이 될 수도 있다.
환경과 디버그 모드를 안전하게 설정하기 위해 플라스크에서는
환경 변수를 이용한다.

환경 값은 플라스크가 어떤 맥락에서 돌고 있는지를 플라스크와
확장, Sentry 같은 여타 프로그램들에 알려 주기 위한 것이다.
환경 변수 :envvar:`FLASK_ENV`\로 제어하며 따로 지정하지
않으면 ``production``\이다.

:envvar:`FLASK_ENV`\를 ``development``\로 설정하면 디버그
모드가 켜진다. 디버그 모드에서 ``flask run`` 하면 기본적으로
대화형 디버거와 재적재 기능을 쓰게 된다. 환경과 별도로 이를
제어하려면 :envvar:`FLASK_DEBUG` 플래그를 쓰면 된다.

.. versionchanged:: 1.0
    디버그 모드와 별도로 환경을 제어하기 위한 :envvar:`FLASK_ENV`
    추가. 개발 환경에서는 디버그 모드가 켜진다.

플라스크를 개발 환경으로 전환하고 디버그 모드를 켜려면
:envvar:`FLASK_ENV`\를 다음처럼 설정하면 된다. ::

    $ export FLASK_ENV=development
    $ flask run

(윈도우에서는 ``export`` 대신 ``set`` 사용.)

위와 같이 환경 변수를 쓰는 방식을 권장한다. 설정이나 코드 안에서
:data:`ENV`\와 :data:`DEBUG`\를 설정하는 게 가능하긴 하지만
가급적 하지 않기를 권한다. 그렇게 하면 ``flask`` 명령에서 그
값을 일찍 읽을 수가 없는데, 그러면 일부 시스템 내지 확장에서
이미 이전 값에 따라 구성을 마쳐 버렸을 수가 있다.


내장 설정 값
------------

플라스크 내부에서 다음 설정 값들을 사용한다.

.. py:data:: ENV

    응용이 어떤 환경에서 돌고 있는가. 플라스크와 확장들에서
    환경에 따라서 디버그 모드 켜기 같은 동작을 활성화할 수
    있다. :attr:`~flask.Flask.env` 속성이 이 설정 키와
    연결된다. :envvar:`FLASK_ENV` 환경 변수에 따라 설정되며
    코드에서 설정 시 예상대로 동작하지 않을 수 있다.

    **운용 환경 도입 시 개발 환경을 활성화하지 않아야 한다.**

    기본값: ``'production'``

    .. versionadded:: 1.0

.. py:data:: DEBUG

    디버그 모드가 켜져 있는지 여부. ``flask run``\으로 개발 서버를
    시작하면 처리 안 된 예외가 있을 때 대화형 디버거가 뜨고 코드가
    바뀔 때 서버가 다시 적재된다. :attr:`~flask.Flask.debug` 속성이
    이 설정 키와 연결된다. :data:`ENV`\가 ``'development'``\일 때
    켜지며 환경 변수 ``FLASK_DEBUG``\로 바꿀 수 있다. 코드 내에서
    설정 시 예상과 다르게 동작할 수 있다.

    **운용 환경 도입 시 디버그 모드를 켜지 않아야 한다.**

    기본값: :data:`ENV`\가 ``'development'``\면 ``True``, 아니면
    ``False``.

.. py:data:: TESTING

    테스트 모드 켜기. 예외가 응용의 오류 핸들러에서 처리되는 게
    아니라 계속 전파된다. 확장들에서도 테스트를 수월하게 만들기 위해
    동작 방식을 바꿀 수 있다. 작성하는 테스트 내에서 켜 줘야 한다.

    기본값: ``False``

.. py:data:: PROPAGATE_EXCEPTIONS

    예외들이 응용의 오류 핸들러에서 처리되는 게 아니라 다시 던져진다.
    설정돼 있지 않은 경우 ``TESTING``\이나 ``DEBUG``\가 켜져 있으면
    암묵적으로 참이다.

    기본값: ``None``

.. py:data:: PRESERVE_CONTEXT_ON_EXCEPTION

    예외 발생 시 요청 문맥을 꺼내지 않기. 설정돼 있지 않은 경우
    ``DEBUG``\가 참이면 참이다. 오류 발생 시 디버거에서 요청 데이터를
    들여다볼 수 있게 해 주며 보통 직접 설정해 줄 필요는 없다.

    기본값: ``None``

.. py:data:: TRAP_HTTP_EXCEPTIONS

    ``HTTPException`` 계열 예외에 대한 핸들러가 없는 경우에 간단한
    오류 응답을 반환하는 대신 다시 던져서 대화형 디버거에서 처리할 수
    있게 한다.

    기본값: ``False``

.. py:data:: TRAP_BAD_REQUEST_ERRORS

    ``args``\나 ``form`` 같은 요청 딕셔너리들의 존재하지 않는 키에
    접근하려 하면 400 Bad Request 오류 페이지가 반환된다. 이 설정을
    켜면 그 오류를 처리 안 된 예외로 다뤄서 대화형 디버거가 뜨게
    된다. ``TRAP_HTTP_EXCEPTIONS``\의 더 상세한 버전이다. 설정돼
    있지 않으면 디버그 모드에서 켜진다.

    기본값: ``None``

.. py:data:: SECRET_KEY

    세션 쿠키를 안전하게 서명하는 데 쓰이고 확장이나 응용의 기타
    보안 관련 용도에 쓰일 수 있는 비밀 키. 긴 난수 바이트 열이어야
    하되 유니코드도 허용한다. 예를 들어 다음 코드의 출력을 설정으로
    복사할 수 있다. ::

        python -c 'import os; print(os.urandom(16))'
        b'_5#y2L"F4Q8z\n\xec]/'

    **질문을 올리거나 코드를 커밋 할 때 비밀 키를 노출하지 않도록 해야 한다.**

    기본값: ``None``

.. py:data:: SESSION_COOKIE_NAME

    세션 쿠키의 이름. 같은 이름의 쿠키가 이미 있는 경우에 바꿀 수 있다.

    기본값: ``'session'``

.. py:data:: SESSION_COOKIE_DOMAIN

    세션 쿠키가 유효하게 되는 도메인 일치 규칙. 설정돼 있지 않으면
    :data:`SERVER_NAME`\의 하위 도메인 전체에서 쿠키가 유효하게 된다.
    ``False``\이면 쿠키의 도메인이 설정되지 않는다.

    기본값: ``None``

.. py:data:: SESSION_COOKIE_PATH

    세션 쿠키가 유효하게 되는 경로. 설정돼 있지 않으면 ``APPLICATION_ROOT``
    (그 값이 설정 안 돼 있으면 ``/``) 아래에서 쿠키가 유효하게 된다.

    기본값: ``None``

.. py:data:: SESSION_COOKIE_HTTPONLY

    보안을 위해 브라우저들에선 "HTTP only"로 표시된 쿠키에
    자바스크립트가 접근하는 걸 허용하지 않는다.

    기본값: ``True``

.. py:data:: SESSION_COOKIE_SECURE

    쿠키가 "secure"로 표시돼 있으면 브라우저들에선 HTTPS 상으로만
    그 쿠키를 보내게 된다. 이게 의미가 있으려면 응용이 HTTPS로
    서비스 돼야 한다.

    기본값: ``False``

.. py:data:: SESSION_COOKIE_SAMESITE

    외부 사이트의 요청으로 쿠키를 보낼 수 있는 방법을 제약한다.
    ``'Lax'``\(권장)나 ``'Strict'``\로 설정할 수 있다.
    :ref:`security-cookie` 참고.

    기본값: ``None``

    .. versionadded:: 1.0

.. py:data:: PERMANENT_SESSION_LIFETIME

    ``session.permanent``\가 참이면 쿠키 만료 시점을 이 값(초
    단위)만큼 후로 설정한다. :class:`datetime.timedelta` 또는
    ``int``\일 수 있다.

    플라스크의 기본 쿠키 구현에서는 암호학적 서명이 이 값보다
    오래되지 않았는지 검사한다.

    기본값: ``timedelta(days=31)`` (``2678400`` 초)

.. py:data:: SESSION_REFRESH_EACH_REQUEST

    ``session.permanent``\가 참일 때 응답마다 쿠키를 보낼지 여부를
    제어한다. 매번 쿠키를 보내면 (기본 방식) 세션이 만료되는 걸
    확실히 방지할 수 있지만 더 많은 대역폭을 쓰게 된다. 유지 방식이
    아닌 세션에는 영향을 주지 않는다.

    기본값: ``True``

.. py:data:: USE_X_SENDFILE

    파일을 보내야 할 때 플라스크에서 그 데이터를 보내는 대신
    ``X-Sendfile`` 헤더를 설정한다. 아파치 같은 일부 웹 서버에서
    그걸 인식해서 더 효율적인 방식으로 데이터를 보낸다. 그런 서버를
    쓰고 있을 때만 의미가 있다.

    기본값: ``False``

.. py:data:: SEND_FILE_MAX_AGE_DEFAULT

    파일을 보낼 때 캐시 컨트롤의 최대 수명을 이 값(초 단위)으로
    설정한다. :class:`datetime.timedelta` 또는 ``int``\일 수
    있다. 응용이나 청사진에서 :meth:`~flask.Flask.get_send_file_max_age`\를
    이용해 파일별로 이 값을 지정할 수 있다.

    기본값: ``timedelta(hours=12)`` (``43200`` 초)

.. py:data:: SERVER_NAME

    어느 호스트 및 포트에 결속돼 있는지 응용에게 알린다. 서브도메인
    라우트 지원을 위해선 필수다.

    설정 시 :data:`SESSION_COOKIE_DOMAIN`\이 설정돼 있지 않으면
    세션 쿠키 도메인으로 쓰인다. 요즘 웹 브라우저들에선 점 없는
    도메인에 대해선 쿠키 설정을 허용하지 않는다. 로컬에서
    도메인을 쓰려면 응용으로 라우팅 돼야 하는 아무 이름이나
    ``hosts`` 파일에 추가해 주면 된다. ::

        127.0.0.1 localhost.dev

    설정 시 ``url_for``\가 요청 문맥 아닌 응용 문맥으로만 외부
    URL을 만들어 낼 수 있다.

    기본값: ``None``

.. py:data:: APPLICATION_ROOT

    웹 서버에서 응용을 어느 경로에 마운트 하고 있는지 응용에게 알린다.

    ``SESSION_COOKIE_PATH``\가 설정돼 있지 않으면 세션 쿠키 경로로
    쓰인다.

    기본값: ``'/'``

.. py:data:: PREFERRED_URL_SCHEME

    요청 문맥 안이 아닐 때 외부 URL 생성에 이 연결 방식을 쓴다.

    기본값: ``'http'``

.. py:data:: MAX_CONTENT_LENGTH

    들어오는 요청 데이터에서 이 바이트 수 넘게 읽어 들이지 않는다.
    설정돼 있지 않은 경우에 요청에 ``CONTENT_LENGTH``\가 지정돼 있지
    않으면 보안을 위해 데이터를 전혀 읽지 않는다.

    기본값: ``None``

.. py:data:: JSON_AS_ASCII

    객체들을 ASCII 인코딩 JSON으로 직렬화 한다. 이 설정이 꺼져
    있으면 JSON이 유니코드 문자열로 반환되거나, 아니면 ``jsonify``\에
    의해 ``UTF-8``\로 인코딩 된다. 그렇게 되면 템플릿에서 JSON을
    자바스크립트로 바꿀 때 보안성에 영향을 주게 되므로 보통은 켜진
    채로 둬야 한다.

    기본값: ``True``

.. py:data:: JSON_SORT_KEYS

    JSON 객체의 키들을 알파벳 순서로 정렬한다. 캐싱에 유용한데,
    파이썬 해시 시드가 어떤 값이든 데이터가 같은 방식으로 직렬화
    되도록 보장해 주기 때문이다. 권장하진 않지만 캐싱을 대가로
    성능을 개선해 보기 위해 이 설정을 끌 수 있다.

    기본값: ``True``

.. py:data:: JSONIFY_PRETTYPRINT_REGULAR

    ``jsonify`` 응답이 사람이 읽기 쉽도록 개행, 공백, 들여쓰기를
    포함해서 출력된다. 디버그 모드에선 항상 켜진다.

    기본값: ``False``

.. py:data:: JSONIFY_MIMETYPE

    ``jsonify`` 응답의 mimetype.

    기본값: ``'application/json'``

.. py:data:: TEMPLATES_AUTO_RELOAD

    템플릿이 바뀌면 재적재한다. 설정돼 있지 않은 경우 디버그 모드에서
    켜지게 된다.

    기본값: ``None``

.. py:data:: EXPLAIN_TEMPLATE_LOADING

    템플릿 파일이 어떻게 적재됐는지 따라가는 디버깅 정보를 로그로
    찍는다. 템플릿이 적재되지 않거나 잘못된 파일이 적재된 것 같을 때
    원인을 알아내는 데 쓸모가 있을 수 있다.

    기본값: ``False``

.. py:data:: MAX_COOKIE_SIZE

    쿠키 헤더가 이 바이트 수보다 크면 경고를 찍는다. 기본값은
    ``4093``\이다. 이보다 큰 쿠키는 브라우저에서 조용히 무시될 수도
    있다. ``0``\으로 설정하면 경고를 끈다.

.. versionadded:: 0.4
   ``LOGGER_NAME``

.. versionadded:: 0.5
   ``SERVER_NAME``

.. versionadded:: 0.6
   ``MAX_CONTENT_LENGTH``

.. versionadded:: 0.7
   ``PROPAGATE_EXCEPTIONS``, ``PRESERVE_CONTEXT_ON_EXCEPTION``

.. versionadded:: 0.8
   ``TRAP_BAD_REQUEST_ERRORS``, ``TRAP_HTTP_EXCEPTIONS``,
   ``APPLICATION_ROOT``, ``SESSION_COOKIE_DOMAIN``,
   ``SESSION_COOKIE_PATH``, ``SESSION_COOKIE_HTTPONLY``,
   ``SESSION_COOKIE_SECURE``

.. versionadded:: 0.9
   ``PREFERRED_URL_SCHEME``

.. versionadded:: 0.10
   ``JSON_AS_ASCII``, ``JSON_SORT_KEYS``, ``JSONIFY_PRETTYPRINT_REGULAR``

.. versionadded:: 0.11
   ``SESSION_REFRESH_EACH_REQUEST``, ``TEMPLATES_AUTO_RELOAD``,
   ``LOGGER_HANDLER_POLICY``, ``EXPLAIN_TEMPLATE_LOADING``

.. versionchanged:: 1.0
    ``LOGGER_NAME``\과 ``LOGGER_HANDLER_POLICY`` 제거함.
    설정 방법에 대한 정보는 :ref:`logging` 참고.

    :envvar:`FLASK_ENV` 환경 변수를 반영하는 :data:`ENV` 추가.

    세션 쿠키의 ``SameSite`` 옵션을 제어하는
    :data:`SESSION_COOKIE_SAMESITE` 추가.

    Werkzeug의 경고를 제어하는 :data:`MAX_COOKIE_SIZE` 추가.


파일로 설정하기
---------------

설정을 별도 파일에, 가능하다면 응용 패키지 외부에 위치한 파일에
저장할 수 있으면 더 유용해진다. 그렇게 되면 다양한 패키지 관리
도구(:ref:`distribute-deployment`)를 통해 응용을 패키징 및
배포하고서 마지막에 설정 파일을 변경하는 방식이 가능해진다.

그래서 다음 패턴을 많이 쓰게 된다. ::

    app = Flask(__name__)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_envvar('YOURAPPLICATION_SETTINGS')

먼저 `yourapplication.default_settings` 모듈에서 설정을 가져온
다음 환경 변수 :envvar:`YOURAPPLICATION_SETTINGS`\가 가리키는
파일의 내용물로 값들을 바꾼다. 리눅스 및 OS X에서는 셸에서
서버 시작 전에 export 명령을 써서 환경 변수를 설정할 수 있다. ::

    $ export YOURAPPLICATION_SETTINGS=/path/to/settings.cfg
    $ python run-app.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader...

윈도우 시스템에선 내장 명령 `set`\을 쓰면 된다. ::

    >set YOURAPPLICATION_SETTINGS=\path\to\settings.cfg

설정 파일 자체는 그냥 파이썬 파일이다. 그런데 대문자로 된 값들만
설정 객체에 실제 저장된다. 따라서 설정 키에 꼭 대문자를 쓰도록
해야 한다.

다음은 설정 파일 예시이다. ::

    # 예시 설정
    DEBUG = False
    SECRET_KEY = b'_5#y2L"F4Q8z\n\xec]/'

설정을 아주 일찍 적재해야 한다. 그래야 확장들이 시작할 때 설정에
접근할 수 있게 된다. 설정 객체에는 개별 파일에서 읽어 오는 것
말고 다른 방법들도 있다. 전체 내용은 :class:`~flask.Config`
객체 문서를 읽어 보면 된다.


환경 변수로 설정하기
--------------------

환경 변수로 설정 파일을 가리키는 것에 더해서 환경에서 직접 설정
값을 가져오는 게 유용할 (또는 필요할) 수도 있다.

리눅스 및 OS X에서는 셸에서 서버 시작 전에 export 명령을 써서
환경 변수를 설정할 수 있다. ::

    $ export SECRET_KEY='5f352379324c22463451387a0aec5d2f'
    $ export MAIL_ENABLED=false
    $ python run-app.py
     * Running on http://127.0.0.1:5000/

윈도우 시스템에선 내장 명령 `set`\을 쓰면 된다. ::

    >set SECRET_KEY='5f352379324c22463451387a0aec5d2f'

사용 방법은 단순명료하지만 한 가지 기억할 건 환경 변수가
문자열이라는 점이다. 자동으로 파이썬 타입으로 역직렬화 되지
않는다.

다음은 환경 변수를 이용하는 설정 예시이다. ::

    import os

    _mail_enabled = os.environ.get("MAIL_ENABLED", default="true")
    MAIL_ENABLED = _mail_enabled.lower() in {"1", "t", "true"}

    SECRET_KEY = os.environ.get("SECRET_KEY")

    if not SECRET_KEY:
        raise ValueError("No SECRET_KEY key set for Flask application")


알다시피 파이썬에서는 빈 문자열을 제외한 모든 값이 불리언 ``True``
값으로 해석되므로 환경 변수에서 ``False``\를 나타내기 위해 명시적으로
값을 설정하는 경우엔 주의가 필요하다.

설정을 아주 일찍 적재해야 한다. 그래야 확장들이 시작할 때 설정에
접근할 수 있게 된다. 설정 객체에는 개별 파일에서 읽어 오는 것
말고 다른 방법들도 있다. 전체 내용은 :class:`~flask.Config`
객체 문서를 읽어 보면 된다.


모범적인 설정 방식
------------------

앞서 언급한 방식의 단점은 테스트가 조금 어려워진다는 점이다. 일반적으로
그 문제에 대한 100% 해법은 없지만 작업 경험 개선을 위해 기억해 둘 만한
것 두 가지는 있다.

1.  응용을 함수로 만들고 청사진으로 등록하자. 그렇게 하면 설정을
    다르게 붙여서 응용 인스턴스를 여럿 만들 수 있고, 그래서 유닛
    테스트가 훨씬 쉬워진다. 이를 이용해 필요한 대로 설정을 전달할
    수 있다.

2.  임포트 시점에 설정이 필요한 코드를 작성하지 말자. 요청에서만
    설정에 접근하도록 제한해 놓으면 나중에 필요할 때 객체를 재설정할
    수 있게 된다.

.. _config-dev-prod:

개발 / 운용
-----------

대다수 응용에서는 한 가지 이상의 설정이 필요하다. 적어도 운용 서버와
개발 중 쓰는 서버에는 별도 설정이 있어야 한다. 이를 처리하는 가장
쉬운 방법은 항상 적재되고 버전 컨트롤에도 들어가는 기본 설정을 두고서
별도 설정으로 위 예에서 언급한 것처럼 필요에 따라 값들을 바꾸는
것이다. ::

    app = Flask(__name__)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_envvar('YOURAPPLICATION_SETTINGS')

그러면 :file:`config.py` 파일을 추가하고
``YOURAPPLICATION_SETTINGS=/path/to/config.py``\를 export 해 주기만
하면 된다. 하지만 또 다른 방법들도 있다. 예를 들어 임포트나
서브클래스를 쓸 수도 있다.

Django 쪽에서 인기 있는 방식은 설정 파일에서 상단에
``from yourapplication.default_settings import *``\를 추가해서
임포트를 명확히 한 다음 직접 변경 사항들을 작성하는 것이다.
그리고 ``YOURAPPLICATION_MODE`` 같은 환경 변수를 확인해서
`production`, `development` 등으로 설정돼 있으면 그에 따라
하드코딩 된 다른 파일을 임포트 할 수도 있을 것이다.

흥미로운 또 다른 패턴은 설정에 클래스와 상속을 이용하는 것이다. ::

    class Config(object):
        DEBUG = False
        TESTING = False
        DATABASE_URI = 'sqlite:///:memory:'

    class ProductionConfig(Config):
        DATABASE_URI = 'mysql://user@localhost/foo'

    class DevelopmentConfig(Config):
        DEBUG = True

    class TestingConfig(Config):
        TESTING = True

이런 설정을 활성화 하려면 :meth:`~flask.Config.from_object`\를
호출하기만 하면 된다. ::

    app.config.from_object('configmodule.ProductionConfig')

참고로 :meth:`~flask.Config.from_object`\에서 클래스 오브젝트의
인스턴스를 생성하지는 않는다. 속성 접근 등을 위해 클래스
인스턴스를 만들어야 하는 경우라면 생성하고서
:meth:`~flask.Config.from_object` 호출을 해야 한다. ::

    from configmodule import ProductionConfig
    app.config.from_object(ProductionConfig())

    # 또는 문자열로 임포트 하기:
    from werkzeug.utils import import_string
    cfg = import_string('configmodule.ProductionConfig')()
    app.config.from_object(cfg)

설정 오브젝트 인스턴스를 만들면 설정 클래스에서 ``@property``\를
쓸 수 있게 된다. ::

    class Config(object):
        """기본 설정, 작업용 데이터베이스 서버 사용"""
        DEBUG = False
        TESTING = False
        DB_SERVER = '192.168.1.56'

        @property
        def DATABASE_URI(self):         # 주의: 모두 대문자
            return 'mysql://user@{}/foo'.format(self.DB_SERVER)

    class ProductionConfig(Config):
        """운영용 데이터베이스 서버 사용"""
        DB_SERVER = '192.168.19.32'

    class DevelopmentConfig(Config):
        DB_SERVER = 'localhost'
        DEBUG = True

    class TestingConfig(Config):
        DB_SERVER = 'localhost'
        DEBUG = True
        DATABASE_URI = 'sqlite:///:memory:'

여러 방법들이 다양하게 있고 설정 파일을 어떻게 관리할지는
각자 결정할 일이다. 다만 권장할 만한 사항들이 몇 가지 있다.

-   버전 컨트롤에 기본 설정 두기. 그 기본 설정으로 설정을 만들거나
    별도 설정 파일에서 기본 설정을 임포트 한 다음 값들을 바꾸면 된다.
-   설정 전환에 환경 변수 이용하기. 파이썬 인터프리터 밖에서
    이뤄질 수 있어서 개발과 배치를 훨씬 쉽게 만들어 준다. 코드를
    전혀 건드리지 않고도 설정을 빠르게 쉽게 전환할 수 있기
    때문이다. 여러 프로젝트에서 작업하는 경우가 많다면 virtualenv를
    source 해서 활성화하고 개발 설정을 export 해 주는 스크립트를
    따로 만들 수도 있다.
-   운용 시 `fabric`_ 같은 도구를 써서 운용 서버(들)로 코드와 설정을
    밀어 넣기. 그렇게 하는 방법에 대한 몇 가지 자세한 내용은
    :ref:`fabric-deployment` 패턴을 보면 된다.

.. _fabric: http://www.fabfile.org/


.. _instance-folders:

인스턴스 폴더
-------------

.. versionadded:: 0.8

플라스크 0.8에서 인스턴스 폴더가 도입됐다. 플라스크에서는 오랫동안
(:attr:`Flask.root_path`\를 통해) 응용 폴더를 기준으로 해서 상대
경로를 가리키는 게 가능했다. 그리고 이건 여러 개발자들이 응용
바로 옆에 저장된 설정을 읽어 오는 방법이기도 했다. 하지만 아쉽게도
이 방식은 응용이 패키지가 아닐 때만 잘 동작한다. 패키지인 경우엔
루트 경로가 패키지 내용을 가리킨다.

플라스크 0.8에서 새 속성 :attr:`Flask.instance_path`\가 도입됐다.
이 속성이 나타내는 건 "인스턴스 폴더"라는 새로운 개념이다.
인스턴스 폴더는 버전 컨트롤 하에 들어가지 않으면서 특정 배치
방식에 한정되지 않도록 설계됐다. 런타임에 바뀌는 것들이나 설정
파일들을 집어넣기에 딱 맞는 곳이다.

플라스크 응용 생성 시에 인스턴스 폴더 경로를 명시적으로 줄 수도
있고, 아니면 플라스크에서 인스턴스 폴더를 자동으로 찾게 할 수도
있다. 명시적으로 설정하려면 `instance_path` 매개변수를 쓰면
된다. ::

    app = Flask(__name__, instance_path='/path/to/instance/folder')

경로를 줄 때는 *반드시* 절대 경로여야 한다는 걸 명심하자.

`instance_path` 매개변수를 주지 않으면 다음 기본 위치를 쓴다.

-   설치 안 된 모듈::

        /myapp.py
        /instance

-   설치 안 된 패키지::

        /myapp
            /__init__.py
        /instance

-   설치된 모듈 또는 패키지::

        $PREFIX/lib/python2.X/site-packages/myapp
        $PREFIX/var/myapp-instance

    ``$PREFIX``\는 파이썬 설치 기준 경로다. ``/usr``\일 수도 있고
    virtualenv 경로일 수도 있다. ``sys.prefix`` 값을 찍어 보면
    기준 경로가 어떻게 설정돼 있는지 알 수 있다.

설정 객체에서 상대 경로명으로 설정 파일을 읽어 들이는 동작을
제공했기 때문에 원한다면 인스턴스 경로를 기준으로 한 파일명으로
읽어 들이도록 바꾸는 게 가능하게 만들었다. 응용 생성자에 주는
`instance_relative_config` 스위치를 통해 설정 파일 상대 경로의
동작 방식을 "응용 루트 기준"(기본 방식)이나 "인스턴스 폴더
기준"으로 바꿀 수 있다. ::

    app = Flask(__name__, instance_relative_config=True)

다음은 모듈에서 설정을 미리 읽어 들인 다음 인스턴스 폴더의 파일이
존재하면 그 내용으로 설정 값들을 바꾸도록 플라스크를 구성하는 방법을
보여 준다. ::

    app = Flask(__name__, instance_relative_config=True)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_pyfile('application.cfg', silent=True)

:attr:`Flask.instance_path`\로 인스턴스 폴더의 경로를 알아낼 수
있다. 또한 플라스크에서는 간편하게 인스턴스 폴더의 파일을 열 수
있도록 :meth:`Flask.open_instance_resource`\를 제공한다.

둘 모두의 사용례::

    filename = os.path.join(app.instance_path, 'application.cfg')
    with open(filename) as f:
        config = f.read()

    # 또는 open_instance_resource로:
    with app.open_instance_resource('application.cfg') as f:
        config = f.read()
