nginx proxy_set_header 설정 완전 이드

목차

http header와 proxy 기본 개념

proxy_set_header upgrate
proxy_set_header Host
proxy_set_header X-Real-IP
proxy_set_header X-Forwarded-For
proxy_set_header x-Forwarded-Proto
통합 예제 및 모범사례 

http header와 프록시의 기본 개념

http 헤더란 무엇인가

http 헤더(http header)는 http 요청과 응답에 포함되는 메타데이터이다. 
클라이언트와 서버 간의 통신에 필요한 추가 정보를 전달 하는 역할을 한다.

WHY 헤더는 응답에도 포함이 되는가? 

주요 특징

크-값 쌍 (key-value pari) 형식: header-name: header -value

요청 헤더(request header): 클라이언트가 서버로 보내는 정보
응답 헤더(response header): 서버가 클라이언트로 보내는 정보

예시

host: example.com
user-agent: mozailla/5.0
content-type: applicatoin/json

프록시의 역할과 동작 원리

프록시는 클라이언트와 서버 사이에서 중간자 역할을 하는 서버입니다. 

프록시의 주요 기능. 

요청 전달: 클라이언트의 요청을 백엔드로 전달
응답 중계: 백엔드 서버으 ㅣ응답을 클라이언트로 전달
로드 밸런싱: 여러 벡엔드 서버에 요청 분산
캐싱: 자주 요청되는 내용을 저장하여 성능 향상
보안: 클라이언트 IP를 숨기거나 필터링 수행

프록시 동작 흐름

클라이언트 -> 프록시 서버 -> 백엔드 서버

1.3 nginx proxy의 기본 동작 방식

nginx는 proxy_pass 지시어를 사용하여 프록시 기능을 구현합니다.

location / {
    proxy_pass http://backend_server;
}

nginx의 기본 헤더 전달 동작

nginx는 기본적으로 클라이언트가 보낸 모든 헤더를 그대로 벡엔드로 전달합니다. 
하지만 프록시 환경에서는 원본 정보가 손실도리 수 있습니다.
클라이언트의 실제 IP는 프록시의 IP로 대체된다! 


proxy_set_header의 필요성

proxy_set_header upgrade

upgrade 헤더는 http 프로토콜에서 다른 프로토콜로 전환하고자 할 때 사용된다. 

주요 용도

websocker 연결: http에서 websocker 프로토콜 전환
http/2 업그레이드: http/1.1 에서 http/2로 전환

기타 프로토콜 전환: 필요한 경우 다른 프로토콜로 전호나

헤더 형식

upgrade: websocket
connection: upgarde

upage 헤더는 반드시 connection: upgrade 와 함께 사용되어야한다. 

websocket 연결에서의 역할

websocker은 실시간 양방향 통신을 위한 프로토콜입니다. http 핸드 셰이크를 통해 websocker으로 전환됩니다. 

websocket 핸드 셰이크 과정

클라이언트가 upgrade: websocket 헤더오 ㅏ함께 요청

서버가 101 switching protocols 응답

httpㅇ녀결이 websocket 연결로 전환

브라우저에서의 동작

브라우저가 upgrade 헤더를 추가하는 경우

브라우저는 직접적으로 upgrade 헤더를 추가하지 않습니다. 대신

javascript webscoket api 사용시에

브라우저가 자동으로 upgrade: websocket 헤더 추가

connection: upgrade 헤더도 자동 추가한다.

브라우저의 동작: 

websocket 연결 시도 시 자동으로 필요한 헤더 생성
개발자가 직접 헤더를 설정할 수 없음.

실제 요청 예시 

get /socket http/1.1
host: example.com
upgrade: websocket
connection: upgrade
sec-websocket-key:
sec-websocket-version: 13

nginx에서의 처리 방식 

문제 상황

nginx는 기본적으로 upgrade와 connection 헤더를 제거할 수 있습니다. 이는 websocket 연결이 실패하는 역할이 됩니다. 

해결 방법

proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade"

설정 설명 

$http_upgrade: 클라이언트가 보낸 Upgrade 헤더의 값을 변수로 저장
proxy_set_header Upgrade $http_upgarde: 클라이언트의 upgrade 헤더 값을 그대로 백엔드로 전달
proxy_set_header connection "upgrade": connection 헤더를 명시적으로 설정

nginx 변수 설명

$http_header_name: 클라이언트가 보낸 특정 헤더 값을 ㅇ릭는 변수 
$http_upgrade: Upgrade 헤더의 값을 의미
값이 없으면 빈 문자열이 됨

완전한 WebSocket 프록시 설정

location /socket {
    proxy_pass http://backend_server
    proxy_http_version: 1.1;
    proxy_set_header Uprage $http_upgrade;
    proxy_setHeader Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}

 proxy_set_header Host

 Host 헤더의 의미와 중요성

 Host 헤더는 http/1.1 에서 필수 헤더이며, 요청이 전송되는 대상 서버의 호스트명과 포트를 지정합니다. 

 Host 헤더 형식

 Host: example.com
 Host: example.com:8080

 중요성

1. 가상 호스팅 (Virtual Hosting): 하나의 서버에서 여러 도메인을 구분
2. 라우팅: 서버가 요청을 어떤 애플리케이션으로 보낼지 결정
3. SSL/TLS 인증서: 인증서와 도메인 매칭 확인

가상 호스팅에서의 역할

가상 호스팅이란? 

하나의 서버에서 여러 도메인을 운영하는 기술.

동작 원리

서버는 host 헤더를 확인하여,
어떤 도메인으로 요청이 왔는지 판단
해당 도메인에 맞는 설정/애플리케이션으로 요청 전달

예시

 요청 1: Host: kotlip.kr -> 애플리케이션 A
 요청 2: Host: another.lcom -> 애플리케이션 B

3.3 브라우저 요청 시 Host 헤더 동작

브라우저는 URL 기반으로 자동으로 Host 헤더를 생성합니다. 

예시
URL: https://kotlip.kr/api/docs
브라우저가 생성하는 헤더: Host: kotlip.kr

브라우저가 host 헤더를 추가하는 과정

1. 사용자가 URL을 입력하거나 링클르 클릭
2. 브라우저가 URL에서 호스트명 추출
3. http 요청에 Host: <호스트명> 헤더 자동 추가
4. 개발자가 직접 설정할 필요 없음. 

실제 요청 예시 

GET /api/docs HTTP/1.1
Host: kotlip.kr
User-Agent: Mozilla/5.0 ...

nginx에서 Host 헤더 변경의 필요성

문제 상황

프록시 환경에서 클라이언트가 보낸 host 헤더를 그대로 전달하면 문제가 발생할 수 있습니다.

시나리오

클라이언트 요청: Host: kotlip.kr
nginx 프록시
백엔드 서버: proxy1.aiserv.ktcloud.com

백엔드 서버는 Host: proxy1.aiserv.ktcloud.com을 기대
하지만 클라이언트의 Host: kotlip.kr이 전달된다.
백엔드가 Host 헤더로 라우팅/검증을 하면 실패

3.5 $host vs 명시적 Host 값

$host 변수

nginx 변수로 클라이언트가 보낸 Host 헤더 값을 의미
예: 클라이언트가 Host: kotlip.kr을 보내면 $host = "kotlip.kr"

명시적 Host 값
백엔드가 기대하는 정확한 Host 값을 직접 지정
proxy_set_header Host proxy1.aiserv.ktcloud.com

비교

# 방법1: 클라이언트의 Host를 그대로 전달
proxy_set_header Host $host; # kotlip.kr이 그대로 전달됨 

# 방법2: 백엔드가 기대하는 host로 변경
proxy_set_header Host proxy1.aiserv.ktcloud.com # 백엔드가 기대하는 값으로 전달

3. 6 백엔드에서 host 헤더를 사용하는 방식

백엔드의 host 헤더 활용

1. 가상 호스팅 구분: 여러 도메인을 하나의 서버에서 처리
2. 라우팅: Host 값에 따라 다른 애플리케이션으로 라우팅
3. 검증: 허용된 Host만 처리하도록 검증
4. 인증서 매칭: SSL/TLS 인증서와 도메인 매칭 확인

백엔드가 받는 헤더 예시

# 올바른 경우
Host: proxy1.aiserv.ktcloud.com -> 백엔드가 정상적으로 처리

# 잘못된 경우
Host: kotlip.kr -> 백엔드가 인식하지 못하거나 거부

3.7 실제 문제 사례 분석

문제 상황

사용자가 제공한 실제 문제:  kotlip.kr에서 proxy1.aiserv.ktcloud.com으로 프록시

이전 설정 (문제)

location /api {
    proxy_pass https://api_backend/api;
    proxy_set_header Host $host; #kotlip.kr을 그대로 전달
}

문제점 분석
1. 클라이언트 요청: Host: kotlip.kr
2. nginx 전달: Host: kolip.ker (그대로 전달)
3. 백엔드 수신: Host: kotlip.kr
4. 백엔드가 proxy1.aiserv.ktcloud.com을 기대하므로 실패

현재 설정 (해결)

location /api {
    proxy_pass https://api_backend;
    proxy_set_header Host proxy1.aiserv.ktlcoud.com;
}

해결 과정 분석

클라이언트 요청: host: kotlip.kr
nginx 처리: Host 헤더를 proxy1.aiserv.ktcloud.com으로 변경
백엔드 수신: Host: proxy1.aiserv.ktcloud.com
백엔드가 올바른 Host를 받아 정상 처리

추가로 수정된 부분: proxy_pass 경로

이전 설정의 문제

proxy_pass https://api_backend/api;

경로가 중복되거나 잘못 전달 될 수 있음
/api/docs 요청이 /api/api/docs로 전달 될 수 있음

현재 설정

proxy_pass https://api_backend;

전체 경로(/api/docs)가 그대로 전달 된다
백엔드가 올바른 경로를 받는다.

결론

Host 헤더를 백엔드가 기대하는 값으로 설정해야 요청이 정상 처리된다
$host를 사용하면 클라이언트의 Host가 그대로 전달되어 문제가 발생할 수 있습니다.
백엔드의 가상 호스팅 설정에 맞춰 명시적으로 Host를 지정하는 것이 안전합니다. 

