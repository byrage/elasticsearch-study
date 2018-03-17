타임아웃 설정 안할 시
- 응답 기다리다가 모두 장애
- 커넥션이 잘못된 것을 인지하지 못함

connection timeout : 소컷 커넥션을 맺는 타임아웃. 500~2000ms
socket timeout : request-response 시간. batch와 web apllication의 시간이 각각 다름.
pool max wait : 커넥션 풀에서 꺼내오기 까지 걸리는 제한 시간. connection timeout보다 길어야함

initial poolcount : DB 실행 시 생성되는 초기의 커넥션 풀. 인스턴스 수에 곱해지기 때문에 멀티 인스턴스 환경에서 체크해야함

JDBC 드라이버 버전에 따라 ms, s가 다르기 때문에 꼭 확인해야 한다.

Keep-alive 설정 시 (1분) https라면 cpu usage가 떨어짐
