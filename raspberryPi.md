# 1. Cài đặt OS

## 1.1. Raspberry Pi Raspbian 64bit headless setup
Hướng dẫn từng bước cài Headless Raspbian 64bit OS trên Raspberry Pi. Đã thử nghiệm trên 3B, 3B+ và 4B.

### 1.1.1. Instructions
1) [Tải OS](https://www.raspberrypi.org/downloads/).

2) Ghi OS đã tải vào SD card (*gợi ý sử dụng Etcher*).

3) Thiết lập mạng WiFi

- Tạo tệp `wpa_supplicant.conf` trong thư mục `boot`:
    ```
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
        update_config=1

        network={
            ssid="<tên mạng>"
            psk="<mật khẩu>"
        }
    ```

- Ngoài ra, có thể dụng cáp Ethernet để kết nối mạng

4) Cho phép truy cập SSH:

Tạo tệp trống `ssh` trong thư mục `boot`.

5) Cho phép kernel 64 bit bằng cách thêm dòng bên dưới vào cuối tệp `config.txt`:
    ```
    arm_64bit=1
    ```
    
6) Rút thẻ từ máy tính và cắm thẻ SD vào Pi. Bật nguồn cho Pi.

7) Tìm địa chỉ IP của Raspberry Pi,sau đó truy cập SSH:
    ```
    ssh pi@<ip>
    ```
    
    - You will be asked to authenticate. By default, the password is `raspberry`.
    - Optionally, you can also [set a static IP address](https://www.pcmag.com/how-to/how-to-set-up-a-static-ip-address) for easier access in the future.

8) Thay đổi các cấu hình của Pi:
    ```
    sudo raspi-config
    ```
    
    1) Set up custom password in `Change User Password`
    2) Update timezone in `Localisation Options -> Change Timezone`
    3) Enable full capacity utilization of the SD card through `Advanced Options -> Expand Filesystem`
    
9) Cập nhập phần mềm:
    ```
    sudo apt-get update && sudo apt-get upgrade
    ```
    
10) Khởi động lại Pi:
    ```
    sudo reboot
    ```
    
### 1.1.2. Notes

- You can check the currently used kernel with:
    ```
    uname -a
    ```
- Some models might only support 2.4GHz wireless network connections
- A forum thread about 64bit Raspbian with troubleshooting tips can be found [here](https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=250730)

## 1.2. Ubuntu 18.04 (no Desktop) 64-bits

1) Tải tệp ảnh OS:
    Truy cập:
        https://ubuntu.com/download/raspberry-pi
    Lựa chọn tệp tải xuống tương ứng cho Pi. (Hướng dẫn này sử dụng: 18.04.3 LTS 64-bit)        

2) Cài đặt tệp ảnh vào Pi:
    
1. Sử dụng công cụ Etcher:
    
- Tải Etcher từ website: https://www.balena.io/etcher/
- Giải nén và chạy file vừa tải:
        Ví dụ dùng command-line:
            cd Downloads
            chmod a+x Etcher-linux-x64.AppImage
            ./Etcher-linux-x64.AppImage
    
2. Cắm thẻ SD của Pi vào máy tính

3. Trên giao diện của Etcher:
    
- Lựa chọn image sử dụng (ở đây là: ubuntu-18.04.4-preinstalled-server-arm64+raspi3.img.xz)
- Lựa chọn thẻ SD của Pi
- Nhấn OK để bắt đầu, đợi ghi ảnh OS vào thẻ
            
4. Cấu hình kết nối WiFi:

- Vào thư mục "writeable" tương ứng với thẻ SD trên máy tính
- Vào thư mục /etc/netplan
- Tạo 1 tệp tên là: "50-cloud-init.yaml". Ghi vào tệp đó nội dung sau (lưu ý viết đúng cú pháp của tệp yaml):
    ``` yml
    network:
      version: 2
      ethernets:
          eth0:
            optional: true
            dhcp4: true
      wifis:
          wlan0:
            optional: true
            access-points:
              "YOUR-SSID-NAME":
               password: "YOUR-NETWORK-PASSWORD"
            dhcp4: true
    ```
5. Lưu và thoát

6. Rút thẻ từ máy tính và cắm thẻ SD vào Pi. Bật nguồn cho Pi.


# 2. Truy cập SSH từ xa tới Raspberry Pi dùng ngrok
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

# 3. Tạo Public Server bằng Python trên Raspberry Pi
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

# 4. Đăng nhập mạng trường Bách Khoa trên terminal

`curl -d "username=<user>&password=<password>" https://bknet20.hust.edu.vn/login`
