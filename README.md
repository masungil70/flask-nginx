# flask-nginx

# ✅ 1. 환경 구성 (예: Ubuntu)
```
$sudo apt update
$sudo apt install python3 python3-pip python3-venv vim nginx -y
```

# ✅ 2. Flask 앱 준비

```
$mkdir ~/myflaskapp
$cd ~/myflaskapp
$python3 -m venv venv
$source venv/bin/activate
$pip install flask gunicorn
```
$vi app.py
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Flask + Gunicorn + Nginx!"
```
# ✅ 3. Gunicorn으로 실행 테스트
```
$gunicorn -w 4 -b 127.0.0.1:8000 app:app

```
```
-w 4: 워커 4개 사용
-b: 바인딩 주소
```
웹 브라우저에서 http://127.0.0.1:8000 로 접속하여 Flask 앱이 동작하는지 확인
```
콘솔에 확인 하는 방법 
$curl http://127.0.0.1:8000
```

# ✅ 4. Nginx 설정 (리버스 프록시)
```
$sudo vi /etc/nginx/sites-available/myflaskapp

server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 심볼릭 링크 생성
```
$sudo ln -s /etc/nginx/sites-available/myflaskapp /etc/nginx/sites-enabled/
$sudo nginx -t
$sudo systemctl restart nginx
```

# ✅ 5. Gunicorn을 systemd 서비스로 등록
```
sudo nano /etc/systemd/system/myflaskapp.service
[Unit]
Description=Gunicorn instance to serve myflaskapp
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/myflaskapp
Environment="PATH=/home/ubuntu/myflaskapp/venv/bin"
ExecStart=/home/ubuntu/myflaskapp/venv/bin/gunicorn -w 4 -b 127.0.0.1:8000 app:app

[Install]
WantedBy=multi-user.target
```

```
$sudo systemctl daemon-reexec
$sudo systemctl start myflaskapp
$sudo systemctl enable myflaskapp
```



# ✅ 마무리 확인 체크리스트
```
 $curl http://YOUR_SERVER_IP
      접속 시 Flask 앱이 나오면 성공

 $sudo systemctl status myflaskapp
      확인 시 상태가 active (running)

 방화벽에서 80 포트 열려 있어야 함
```
