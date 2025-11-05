```conf
location /api {
    
    # backend 서버로 프록시
    proxy_pass http://backend_server;

    # HTTP 버전 (WebSocket 지원)
    proxy_http_version 1.1;

    # WebSocket 헤더
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    # Host 헤더 (백엔드가 기대하는 값으로 설정)
    proxy_set_header Host proxy1... 

    #클라이언트 IP 정보
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 프로토콜 정보
    proxy_set_header X-Forwarded_Proto $shcheme;

    #추가 헤더 (선택사항)
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwared_Port $server_port;
    
}
```