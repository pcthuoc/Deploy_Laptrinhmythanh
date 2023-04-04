Live site: https://laptrinhmythanh.com/


### Cài đặt trên máy Linux

1. Cài đặt môi trường

    ```bash
    sudo apt-get update && sudo apt-get install -y vim python-pip curl git
    pip install docker-compose
    ```

2. Cài Docker 

   ```bash
   sudo curl -sSL get.docker.com | sh
   ```


3. Clone repo

    ```bash
    git clone -b 2.0 git@github.com:pcthuoc/Deploy_Laptrinhmythanh.git && cd Deploy_Laptrinhmythanh
    ```

4. Khởi động

    ```bash
    docker-compose up -d
    ```

    > Nếu bạn không dùng user `root`，hãy sử dụng `sudo -E docker-compose up -d`.

Quá trình cài đặt có thể tốn từ 5 đến 30 phút phụ thuộc vào tốc độ mạng!

Sau đó, hãy kiểm tra bằng lệnh `docker ps -a`，nếu không có container nào ở trạng thái `unhealthy` hoặc `Exited (x) xxx` thì là ok rồi đó.

## Sử dụng

## Cấu hình Revese Proxy và Load Balancing

Setup Nginx:

```sh
sudo apt update
sudo apt install nginx
```

Check status: 

```sh
sudo serivce nginx status
```

Gỡ file `default`:

```sh
sudo ln -s /etc/nginx/sites-enabled/default
```

Tạo file `re-proxy.conf`

```sh
sudo vi /etc/nginx/sites-enabled/re-proxy.conf
```

Nội dung

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;
  	root /var/www/html;
	index index.html index.htm index.nginx-debian.html;
	client_max_body_size 125M; # Cấu hình to to 1 tí để đề phòng upload file testcase lớn :v
	server_name contest.vcspassport.com;
	location / {
		proxy_pass http://127.0.0.1:5000;
		proxy_set_header X-Forwarded-Proto $scheme;
		try_files $uri $uri/ /index.html =404;
	}
	location /api/(.*) {
                proxy_pass http://127.0.0.1:5000/api;
		proxy_set_header X-Forwarded-Proto $scheme;
                try_files $uri $uri/ /index.html =404;
        }
        # Caching and filter XSS
        gzip             on;
        gzip_comp_level  3;
        gzip_types       text/html text/plain text/css image/*;
        add_header    X-Content-Type-Options nosniff;
        add_header    X-Frame-Options SAMEORIGIN;
        add_header    X-XSS-Protection "1; mode=block";
}
```

Reload config:

```sh
sudo service nginx restart
```
