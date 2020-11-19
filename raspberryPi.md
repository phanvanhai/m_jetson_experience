# 1. Truy cập SSH từ xa tới Raspberry Pi dùng ngrok
1. Tải `ngrok`:

- `sudo wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-arm.zip`
- `unzip ngrok-stable-linux-arm.zip`
- `rm ngrok-stable-linux-arm.zip`

2. Đăng ký tài khoản (miễn phí) trên `https://ngrok.com`
3. Lấy mã xác thực tại `https://dashboard.ngrok.com/get-started/setup`:
`$./ngrok authtoken 1crSACIZhxxxxxxxxxxxxx8gy84fNx`
4. Xác thực: `./ngrok authtoken 1crSACIZhxxxxxxxxxxxxx8gy84fNx`
5. Chạy `ngrok` trên Pi:
```
$ ngrok tcp --region=ap 22
 Forwarding tcp://0.tcp.ap.ngrok.io:12681-> localhost:22
```
Địa chỉ trên có thể lấy từ: `https://dashboard.ngrok.com/status/tunnels`

6. Truy cập từ xa:
```$ ssh <user>@0.tcp.ap.ngrok.io -p 12681```

# 2. Tạo Public Server bằng Python trên Raspberry Pi
1. Chuẩn bị:

- Screen: `sudo apt install screen`

- Server: Ví dụ Server lắng nghe ở cổng 5000:
    - `mkdir /home/pi/server`
    - `sudo nano /home/pi/server/server.py`
    - Sao chép nội dung bên dưới và lưu lại:
    ``` python
    # pip3 install Flask
    # pip3 install requests

    from flask import Flask, json, request

    api = Flask(__name__)

    @api.route('/hello')
    def index():
      return "Hello world!" 

    @api.route('/post', methods=['POST'])
    def post_data():
        body = request.get_json()
        return json.dumps(body)

    if __name__ == '__main__':
        api.run(host="0.0.0.0",port=5000)
    ```
    - Mở 1 `screen` để chạy Server: `screen -S server`
    - `python3 /home/pi/server/server.py`
    - Detach screen: `ctrl+a` `ctrl+d`

2. Cài đặt: `sudo npm install -g localtunnel`
3. Tunnel ở cổng 5000:

- Mở 1 `screen` để chạy tunnel: `screen -S tunnel`
- Chạy Tunnel ở cổng 500: `lt -h "https://serverless.social" -s unique-your-domain -p 5000`
    Kết quả thu được: `your url is: https://unique-your-domain.serverless.social`
- Detach screen: `ctrl+a` `ctrl+d`

4. Kiểm tra kết quả:

- `curl --request GET 'https://unique-your-domain.serverless.social/hello'` --> Kết quả: `Hello world!`
- `curl --request POST 'https://unique-your-domain.serverless.social/post' --header 'Content-Type: application/json' --data-raw '{"hello":"world"}'` --> Kết quả: `{"hello": "world"}`

5. Dừng:

- Lấy danh sách các `screen` đang chạy: `screen -ls`
```
There are screens on:
        2895.server     (11/19/2020 01:16:12 PM)        (Detached)
        2825.tunnel     (11/19/2020 01:09:14 PM)        (Detached)
2 Sockets in /run/screen/S-pi.
```

- Hủy `screen` đang chạy Server:
    + Mở lại (`attach`) `screen` tên `Server`: `screen -r 2895`
    + `ctrl+C`
    + Xóa `screen` hiện tại: `ctrl+a` `k` `y`
- Hủy `screen` đang chạy Tunnel:
    + Mở lại (`attach`) `screen` tên `tunnel`: `screen -r 2825`
    + `ctrl+C`
    + Xóa `screen` hiện tại: `ctrl+a` `k` `y`
