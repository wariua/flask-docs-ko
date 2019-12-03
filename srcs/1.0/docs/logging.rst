.. _logging:

로그
====

플라스크에서는 표준적인 파이썬 :mod:`logging` 모듈을 쓴다. 모든
플라스크 관련 메시지들은 로그 네임스페이스 ``'flask'``\에 기록된다.
:meth:`Flask.logger <flask.Flask.logger>`\는 ``'flask.app'``\이라는
이름의 로거를 반환하는데, 이를 이용해 응용에서 메시지를 남길 수 있다. ::

    @app.route('/login', methods=['POST'])
    def login():
        user = get_user(request.form['username'])

        if user.check_password(request.form['password']):
            login_user(user)
            app.logger.info('%s logged in successfully', user.username)
            return redirect(url_for('index'))
        else:
            app.logger.info('%s failed to log in', user.username)
            abort(401)


기본적인 설정
-------------

프로젝트에 로그 설정을 하고 싶다면 프로그램 시작 후 가급적
빨리 하는 게 좋다. 로그 설정을 하기 전에 :meth:`app.logger
<flask.Flask.logger>`\에 접근하면 거기서 기본 핸들러를 추가하기
때문이다. 가능하다면 응용 객체 생성 전에 로그를 설정하자.

다음 예는 :func:`~logging.config.dictConfig`\를 사용해서
플라스크 기본 설정과 비슷한 로그 설정을 만든다. ::

    from logging.config import dictConfig

    dictConfig({
        'version': 1,
        'formatters': {'default': {
            'format': '[%(asctime)s] %(levelname)s in %(module)s: %(message)s',
        }},
        'handlers': {'wsgi': {
            'class': 'logging.StreamHandler',
            'stream': 'ext://flask.logging.wsgi_errors_stream',
            'formatter': 'default'
        }},
        'root': {
            'level': 'INFO',
            'handlers': ['wsgi']
        }
    })

    app = Flask(__name__)


기본 설정
`````````

로그 설정을 직접 하지 않으면 플라스크에서 :meth:`app.logger
<flask.Flask.logger>`\에 :class:`~logging.StreamHandler` 하나를
자동으로 추가해 준다. 요청 처리 동안은 ``environ['wsgi.errors']``\의
WSGI 지정 스트림(일반적으로 :data:`sys.stderr`)으로 기록을 하게
된다. 요청 밖에선 :data:`sys.stderr`\로 로그를 남긴다.


기본 핸들러 제거하기
````````````````````

:meth:`app.logger <flask.Flask.logger>`\에 접근한 다음에 로그를
설정했으며 기본 핸들러를 제거해야 한다면 임포트 해서 제거할
수 있다. ::

    from flask.logging import default_handler

    app.logger.removeHandler(default_handler)


관리자에게 오류 이메일 보내기
-----------------------------

실제 서비스용 원격 서버에서 응용을 돌리고 있다면 아마 로그 메시지를
자주 살펴보지는 않을 것이다. WSGI 서버에선 로그 메시지를 파일로
보낼 테고, 사용자가 뭔가 이상하다고 해야만 그 파일을 확인하게 된다.

버그 발견 및 수정을 위한 사전 대비를 하고 싶다면
:class:`logging.handlers.SMTPHandler`\를 설정해서 오류 및 그 이상의
로그가 발생했을 때 이메일을 보내도록 할 수 있다. ::

    import logging
    from logging.handlers import SMTPHandler

    mail_handler = SMTPHandler(
        mailhost='127.0.0.1',
        fromaddr='server-error@example.com',
        toaddrs=['admin@example.com'],
        subject='Application Error'
    )
    mail_handler.setLevel(logging.ERROR)
    mail_handler.setFormatter(logging.Formatter(
        '[%(asctime)s] %(levelname)s in %(module)s: %(message)s'
    ))

    if not app.debug:
        app.logger.addHandler(mail_handler)

이렇게 하려면 같은 서버에 SMTP 서버가 구성돼 있어야 한다. 핸들러
설정에 대한 자세한 내용은 파이썬 문서를 보라.


요청 정보 집어넣기
------------------

요청에 대한 (IP 주소 같은) 추가 정보를 보는 게 일부 오류를 디버깅
하는 데 도움이 될 수도 있다. :class:`logging.Formatter`\의
서브클래스를 만들면 원하는 필드를 집어넣어서 메시지에 쓸 수 있다.
플라스크 기본 핸들러나 위에서 정의했던 메일 핸들러, 그 외 어느
핸들러도 서식 클래스를 바꿀 수 있다. ::

    from flask import has_request_context, request
    from flask.logging import default_handler

    class RequestFormatter(logging.Formatter):
        def format(self, record):
            if has_request_context():
                record.url = request.url
                record.remote_addr = request.remote_addr
            else:
                record.url = None
                record.remote_addr = None

            return super().format(record)

    formatter = RequestFormatter(
        '[%(asctime)s] %(remote_addr)s requested %(url)s\n'
        '%(levelname)s in %(module)s: %(message)s'
    )
    default_handler.setFormatter(formatter)
    mail_handler.setFormatter(formatter)


다른 라이브러리
---------------

다른 라이브러리에서도 logging을 폭넓게 이용하고 있을 수 있는데
그 로그에 있는 유의미한 메시지들도 보고 싶다고 하자. 그렇게 하는
가장 간단한 방법은 앱 로거 대신 루트 로거에 핸들러를 추가하는
것이다. ::

    from flask.logging import default_handler

    root = logging.getLogger()
    root.addHandler(default_handler)
    root.addHandler(mail_handler)

프로젝트에 따라선 루트 로거만 설정하는 것보다 관심 있는 각 로거를
따로 설정하는 게 더 나을 수도 있다. ::

    for logger in (
        app.logger,
        logging.getLogger('sqlalchemy'),
        logging.getLogger('other_package'),
    ):
        logger.addHandler(default_handler)
        logger.addHandler(mail_handler)


Werkzeug
````````

Werkzeug에서는 ``'werkzeug'`` 로거로 기본적인 요청/응답 정보를
기록한다. 루트 로거에 핸들러가 설정돼 있지 않으면 Werkzeug에서는
자기 로거에 :class:`~logging.StreamHandler`\를 추가한다.


Flask 확장
``````````

상황에 따라 확장에서 :meth:`app.logger <flask.Flask.logger>`\로
로그를 남기기로 했을 수도 있고 자체 로거를 쓰기로 했을 수도 있다.
자세한 건 각 확장의 문서를 확인해 보자.
