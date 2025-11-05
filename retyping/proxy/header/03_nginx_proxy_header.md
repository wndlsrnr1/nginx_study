proxy_set_header X-Forwarded-Proto

X-Forwarded-Proto 헤더의 의미

X-Forwarded-Proto는 프록시를 통과한 요청의 원본 프로토콜 (HTTP, HTTPS)를 전달하는 비표준 HTTP 헤더입니다. 

헤더 형식

X-Forwarded-Proto: https
X-Forwarded-Proto: http

용도

SSL/TLS 종료 (SSL Termination) 환경에서 원본 프로토콜 정보 보존
백엔드가 HTTPS인지 HTTP인지 판단
리다이렉트 URL 생성 시 올바른 프로토콜 사용

HTTPS 종료 (SSL Termination) 환경

SSL Termination이란? 

프록시 서버에서 SSL/TLS를 종료하고, 백엔드 서버와는 일반 HTTP로 통신하는 방식입니다. 

아키텍처

클라이언트 -> HTTPS -> nginx 프록시 -> HTTP -> 백엔드 서버 -> SSL 종료

장점

성능 향상: SSL 암호화/복호화 부하를 프록시에서만 처리
인증서 관리: 프록시에서만 인증서 관리
백엔드 단순화: 백엔드는 HTTP만 처리

문제점

백엔드는 HTTP로 요청을 받지만, 실제로는 HTTPS 요청이었음
백엔드가 프로토콜로 구분할 수 없음

$scheme 변수의 의미

$scheme 변수

nginx 변수로, 요청이 들어온 프로토콜을 의미합니다. 

값

$scheme = "http"
$scheme = "https" 요청

동작 방식

클라이언트 -> [HTTPS] -> nginx

nginx 에서 $scheme = "https"

클라이언트 -> [HTTPS] -> nginx

nginx에서 $scheme = "http"

6.4 브라우저 요청과 프로토콜 정보

브라우저의 동작

브라우저는 URL의 프로토콜을 확인하여 요청을 보냅니다.

예시

URL: https://example.com/api
브라우저는 HTTPS 연결을 시도
HTTP 요청에는 프로토콜 정보가 헤더에 포함되지 않음. 

실제 요청

GET /api HTTP/1.1

Host: example.com

프로토콜 정보는 tcp 레벨에서만 확인 가능

문제

프록시를 통과하면: 

백엔드는 HTTP로 요청 받음

원본이 HTTPS였는 지 알 수 없음.

6.5 nginx에서의 처리

기본 설정

proxy_set_header X-Forwarded-Proto $scheme

동작 우너리

1. nginx가 클라이언트로부터 요청을 받음
2. $scheme에 프로토콜 정보 저장 (http 또는 https)
3. X-Forwarded-Proto 헤더에 이 값을 설정하여 백엔드로 전달

예시 설정

location / {
    proxy_pass http://backend_server;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
}

전달 과정

HTTPS 요청의 경우

X-Forwarded-Proto: https 헤더 추가

백엔드: HTTP로 요청을 받지만, X-Forwarded-Proto로 원본 프로토콜 확인

HTTP 요청의 경우

클라이언트 -> [HTTP] -> nginx

nginx: $scheme = 'http'

X-Forwarded-Proto: http 헤더 추가

백엔드: HTTP 요청임을 확인

SSL Termination 설정 예시

server {
    listen 443 ssl;
    server_name example.com

    ssl_certificate /path/to/cert.pem;
    ssl-certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://backend_server;
        proxy_setheader Host $host;
        proxy_set_hreader X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header_ X-Forwarded-Proto $scheme;
    }


}

백엔드에서의 프로토콜 인식 방법

백엔드가 받는 헤더

X-Forwarded-Proto: https
Host: example.com

활용 사례

리다이렉트 URL 생성

```py
def get_redirect_url(path):
    proto=request.headers.get("X-Forwarded_Proto", "http")
    host=request.headres.get("Host", "localhost")
    return .. 

proto =requets.headdres.get("X-Forwarded-Proto", "http")
if proto != "https":
    return "HTTPS required", 403

3. 쿠키 설정
proto = request.headers.get("X-Forwarded-Proto", "http")
secure = (proto=="https")
response.set_cookie("session", value, secure=secure)
```

주의 사항

신뢰할 수 있는 프록시: 클라이언트가 헤더를 조작할 수 있음
기본값 처리: 헤더가 없을 경우 HTTP로 가정
보안: 중요한 경우 추가 검증 필요

다중 프록시 환경

첫 번째 프록시에서 원본 프로토콜 설정
이후 프록시는 값을 그대로 전달
각 프록시의 프로토콜이 아닌 원본 프로토콜이 중요

실제 사용 예시

server {
    listen 80;
    listen 443 ssl;

    location / {
        proxy_pass http://backend;
        proxy_set_header X-Forwarded_Proto $scheme;
        # HTTP 요청: X-Forwarded-Proto: http
        # HTTPS 요청: X-Forwarded-Proto: https
    }
}

통합 예제 및 모범 사례

기본 프록시 설정

```conf

location / {
    proxy_pass http://backend_server;

    #기본헤더 설정
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-forwarded_proto $scheme;

    # 타임아웃 설정

    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}

WebSocket 지원 프록시 설정

location /socket {
    proxy_pass http://backend_server;

    #WebSocket을 위한 HTTP 버전 업그레이드
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    #기본 헤더 설정
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-forwarded_proto $scheme;.

    # Websocket 타임아웃 (연결유지)

    proxy_read_timeout 3600s;
}

# 모든 헤더를 함께 사용하는 경우

# 완전한 프록시 설정

location /api {
    proxy_pass https://backend_server;

    #HTTP 버전 (WebSocket 지원)
    proxy_http_version 1.1

    #WebSocket 헤더
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    #Host 헤더 (백엔드가 기대하는 값으로 설정)
    proxy_set_header Host proxy1.aiserv.ktcloud.com

    #클라이언트 IP 정보
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    #프로토콜 정보
    proxy_set_header X-Forwarded-Proto $scheme;

    #추가 헤더(선택사항)
    proxy_set_header X-forwarded-Host $host;
    proxy_set_header X-Forwared_Port $server_port;
}

# Connection 헤더 변수 설정

WebSocket을 사용할 때는 $connection_upgrade 변수를 사용하는 것이 좋습니다.

map $http_ugparde $connecioUpgrade {
    dfeaultupgae
    "close"
}

server {
    location / {
        proxy_set_header Connection $connection_upgarde
    }
}

```

문제 해결 시나리오

시나리오 1: Host 헤더 문제

증상

백엔드에서 404 또는 라우팅 실패
로그에서 잘못된 Host 값 확인

원인

proxy_set_header Host $host; # 클라이언트의 Host를 그대로 전달

해결

proxy_set_header Host proxy1.aiserv.ktcloud.com; # 백엔드가 기대하는 host로 설정

WebSocket 연결 실패

증상

Websocket 연결이 즉시 끊김

HTTP 요청으로 처리됨

원인

# Upgrade 헤더가 전달 되지 암ㅎ음

location /socket {
    pa
}

# 시나리오 3: 클라이언트 IP가 프록시 IP로 표시 됨

증상

로그에 프록시 IP만 기록됨
실제 클라이언트 IP를 알 수 없음

원인

# X-Real-IP 또는 X-Forwarded-For 헤더가 설정되지 암ㅎ음

location / {
    proxy_pass http://backend;
}

해결

location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwared_for $proxy_add_x_forwarded_for;
}

시나리오4 : HTTPS 리다이렉트가 HTTP로 생성됨

증상

HTTPS로 접속했는데 리다이렉트 URL이 HTTP로 생성됨
보안 경고 발생

원인 

# X-Forwarded-Proto 헤더가 설정되지 암ㅎ음

location / {
    proxy_pass http://backend;
}

location / {
    proxy_pass http://backend;
    proxy_set_header X-forwarded-Proto $scheme;
}

# 주의 사항 및 베스트 프랙티스

Host 헤더 설정

구너징: 백엔은ㄷ가 기대하는 host 값을 명시적으로 설정
비구너장: $host를 무조건 사용 (백엔드 설정과맞지 않을 수 잇음)

클라이언트 IP 전달

권장: X-Real-IP와 X-Forwarded-For를 함께ㅐ 사용 

권장: $proxy_add_x_forwarded_for 사용(기존 값 보존)

비권장: X-Forwarded-for를 수도ㅗㅇ으로 설정 (기존 값 손실)

프로토콜 정보

구너장: SSL Termination 환경에서 반드시 X-Forwarded-Proto 설정
권장 $scheme 변수 사용

Websocket 지원

권장: WebSocket 경로에 Upgrade와 Connefction 헤더 설정
권장: proxy_http_version 1.1 사용
권장: WebSocket 경로에 더 긴 타임아웃 설정

보안 고려사항

주의: 클라이언트가 헤더 조작가능
권장: 신뢰할 수 잇는 프록시에서만 헤더 설정
권장: IP 화이트 리스트 사용 고려
권장: 백엔드에서 헤더 검증 로직 구현

선능 최적화: 

권장: 적절한 버퍼 크기 설정
권장: 타임아웃 값 조정
권장: 불필요한 헤더 제거

디버깅 팁

로그에 받은 헤더 값 기록
curl-H
백엔드에서 받은 헤더 값 확인
ngixn 변수 값을 로그로 출력

# 설정 검증
# 디버깅용 로그 설정

요약

핵심 포인트

Host: 백엔드가 기댛나느 값으로 명시적으로 ㅅ저러
X-Real-IP: 단일 프록시에서 클라이언트 IP 전달
X-Forwared-For: 다중 프록시에서 IP 체인 추적
X-Forwarded-Proto: SSL Termination 환경에서 프로토콜 정보 전달
Upgrade: Websocket 연결 지우너

설정 체크리트스트

host 헫헫