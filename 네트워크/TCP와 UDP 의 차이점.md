## TCP
<img width="683" height="281" alt="image" src="https://github.com/user-attachments/assets/7214d9d0-1d62-4243-8c90-c0ae3032a5ea" />

TCP는 UDP와 달리 신뢰성을 보장하기 위해서 사용하는 프로토콜이다.

세션 연결을 보장하기 위해서 3way handshake 연결을 진행하고, 순서를 보장하기 위한 정보를 패킷 헤더에 담아 전달할 뿐만 아니라, 흐름제어와 혼잡제어까지 진행한다.
하지만 이로 인해 지연 시간이 발생하는 문제가 있다.

따라서 TCP는 파일 전송, DB 연결 등의 신뢰성이 중요한 서비스에서 사용한다.

## UDP
<img width="820" height="312" alt="image" src="https://github.com/user-attachments/assets/747d363c-f90a-435c-b275-a73444013e4f" />

반대로 UDP는 헤더가 단순하고, 세션 연결 없이 바로 전송하여, 속도가 빠르다. 하지만 순서나 무결성을 보장하지는 못한다.
따라서 UDP는 영상 스트리밍, 방송, 게임 서비스 등에서 많이 사용한다.

(`checksum`이 존재하지만, 해당 깨졌는지 확인만 할 뿐, 재전송을 요청하거나 순서를 제어하지는 못한다.)
