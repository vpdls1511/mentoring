소켓은 실시간 통신의 기술이다.

우리는 그저 Socket은 실시간 통신을 위해서 사용하는 기술이다! 라고만 알고 있지만, 이번 기회에 더욱 정확하게 공부를 해 보자 !

# Socket 이전 실시간 통신 기술들

Socket이라는 개념이 나오기 전 어떠한 통신 기술이 존재했을까?

바로, http를 이용한 실시간 통신 방식 COMET 을 이용했는데 이는 무엇일까?

## COMET

COMET은 Client로 유의미한 메시지를 전달할 때 까지 HTTP응답을 지연시키는 기술이다.

서버가 클라이언트의 요청에 응답할 때 응답을 “늘어뜨리는" 방법을 이용해긴 시간동안 브라우저가 접속을 끊지 않고 서버의 응답을 대기하도록 만드는것이다.

Long polling과 유사한 기능을 가진다.

>Long Polling
서버측에서 단순히 클라이언트 측에 대한 연결을 길게 유지함으로, 지정 기간 내 정보가 있으면 응답을 전달하는 방식이다. 메시지 양이 많으면 Polling과 차이가 많이 나지 않으며 지정한 시간이 끊기면 다시 요청하는 말 그대로  Long한Polling이다.

즉, HTTP의 Stateless를 보안하기위해 만들어진 방법이라고 보면 좀 편할것같다.

통신을 하는동안 서버와 연결을 끊지 않고, 계속해서 데이터를 주고받는것이다.

# 클라이언트 소켓과 서버 소켓

소켓은 떨어져 있는 호스트를 이어주는 도구로써 인터페이스의 역하을 하는데, 주고받을 수 있는 구조체로 소켓을 통해 통로가 만들어진다. 이러한 소켓은 역할에 따라 서버 소켓, 클라이언트 소켓으로 구분이 된다.

두 개의 시스템 혹은 프로세스가 소켓을 통해 네트워크 연결을 만들기 위해서는 어느 한곳에서 연결을 요청해야 한다. 이 때 IP주소와 포트 번호로 식별할 대상에게 네트워크 연결을 하고싶다는 요청을 하는 것이다.

하지만, 무작적 요청을 한다고 해서 연결이 채결되는것은 아니다. 수신측에서 어떤 연결요청을 받아들일지 미리 시스템에 등록을 하고, 요청이 수신되었을 때 요청을 처리할 수 있도록 해야한다.

메세지를 예시로 들어보자.
우리가 채팅방을 하나 만들려고 한다. 그저 클라이언트가 “서버야 소켓연결해줘"라고 했을때 서버는 “응 그래" 라고 답이 올 수 있을까? 서버에 소켓통신과 관련된 기능이 구현이 되어있다면 가능할것이다. 하지만, 구현이 되어있지 않는다면 안된다고 할 것이다.

# 소켓 API 실행흐름

![](https://velog.velcdn.com/images/vpdls1511/post/b091da74-aa95-46a4-960c-1da093dfb96d/image.png)


소켓의 흐름에 대해 정리가 너무 잘 된 이미지이다.

## Client

- socket()
    - 소켓을 생성한다
    - TCP소켓일 경우 스트림 타입, UDP의 경우 데이터그램 타입 지정
    - 연결 대상 IP와 Port지정 후 connect() 호출
- connect()
    - socket()에서 받은 IP와 port를 토대로 연결 요청
    - 연결 요청에대한 결과가 정해지기 전까지 대기
    - 연결이 되면 send()/recv() 호출
- send()/recv()
    - 소켓으로 데이터를 보낼 때 사용한다.
    - connect()와 동일하게 결과가 정해지기 전 까지 대기한다.
    - recv()의 경우 한번 실행되며, 언제까지 대기할 지 쓰레드를 실행해 수신을 기다린다.
- close()
    - 데이터의 송수신이 필요 없게되면 소켓 연결을 종료하기위해 호출한다.

## Server

- socket()
    - socket을 생성한다.
- bind()
    - 포트번호 혹은 IP+ 포트번호를 파라미터로 받는다.
    - 만약, 이미 사용중인 포트라면 에러를 반환한다.
- listen()
    - Client의 접속 요청을 받을 수 있는 상태이다.
    - 연결이 될 때 까지 계속해서 기다린다.
        - 요청이 수신되지 않거나, 에러 발생 시 연결이 끊어진다.
    - 연결 요청은 Queue형태로 쌓이게 되며, 이 요청의 연결을 수립하기 위해서는 accpet()를 호출한다.
- accept()
    - accept()가 호출이 되면 연결된 Client와 통신을 할 수 있는것이다.
    - accept()가 호출이 될 때 마다 새로운 소켓이 만들어져 연결이 된다.
- send() / recv()
    - 클라이언트와 동일하다
- close()
    - 클라이언트와 동일하다
    - 하지만, 서버 소켓은 close() 호출 시 연결된 Client만 연결을 끊는것이 아닌 Socket을 닫아버려야 하니 유의해야한다.