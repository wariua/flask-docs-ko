.. _tutorial:

따라하기
========

.. toctree::
    :caption: Contents:
    :maxdepth: 1

    layout
    factory
    database
    views
    templates
    static
    blog
    install
    tests
    deploy
    next

이 따라하기에서는 Flaskr라는 간단한 블로그 응용을 만드는 과정을
따라가 본다. 이 응용에서는 사용자가 회원 등록을 하고, 로그인 하고,
글을 쓰고, 자기 글을 편집하거나 삭제할 수 있다. 응용을 패키징 해서
다른 컴퓨터에 설치할 수도 있게 된다.

.. image:: flaskr_index.png
    :align: center
    :class: screenshot
    :alt: 인덱스 페이지 스크린샷

읽는 이가 파이썬에 이미 익숙하다고 가정한다. 배우거나 복습하려면
파이썬 문서의 `공식 자습서`_\를 읽는 게 좋은 방법이다.

.. _공식 자습서: https://docs.python.org/3/tutorial/

출발점으로 삼기에는 좋지만 따라하기에서 플라스크의 모든 기능을
다루지는 않는다. :ref:`quickstart` 절에서 플라스크가 뭘 할 수 있는지
둘러본 다음 문서를 더 들여다 보면 된다. 한편으로 따라하기에서는
플라스크와 파이썬에서 제공하는 것만 이용한다. 다른 프로젝트에서는
어떤 작업을 더 간편하게 하기 위해 :ref:`extensions`\이나 다른
라이브러리를 이용할 수도 있을 것이다.

.. image:: flaskr_login.png
    :align: center
    :class: screenshot
    :alt: 로그인 페이지 스크린샷

플라스크는 유연하다. 특정 프로젝트 배치나 코드 배치를 써야 한다고
요구하지 않는다. 하지만 처음에는 좀 구조적인 방식을 따르는 게
도움이 된다. 그래서 따라하기 앞부분에서 이것저것 갖다 붙여야 할 게
좀 있을 텐데, 신규 개발자가 마주치는 여러 흔한 문제들을 피하기
위한 일이기도 하고 확장하기 쉬운 프로젝트를 만드는 작업이기도
하다. 플라스크에 좀 더 익숙해지고 나면 그 구조에서 빠져나와서
플라스크의 유연성을 맘껏 즐길 수 있다.

.. image:: flaskr_edit.png
    :align: center
    :class: screenshot
    :alt: 편집 페이지 스크린샷

:gh:`따라하기의 프로젝트가 플라스크 저장소에 예시로 있다
<examples/tutorial>`. 따라하기를 따라가면서 작성한 프로젝트를
최종 결과물과 비교해 볼 수 있을 것이다.

:doc:`layout` 절로 이어진다.
