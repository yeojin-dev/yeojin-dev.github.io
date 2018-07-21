---
layout: post
title:  "파이썬과 아두이노 시리얼 통신"
date:   2018-7-21
categories: [python]
---

<p class="intro"><span class="dropcap">p</span>ySerial 패키지를 이용하면 파이썬과 아두이노가 시리얼 통신을 할 수 있다.</p>

어제 패스트캠퍼스 해커톤에 참여하게 되었는데 기획 중에 파이선과 아두이노 사이에 통신을 하는 기술이 필요했다. 아두이노로 온도, 습도를 측정하고 이 데이터를 파이썬을 이용해서 AWS Django 애플리케이션로 request를 보내야 했다. 그래서 파이썬에 관련 패키지가 있는지 알아보았고 검색 결과 [pySerial]이라는 패키지를 이용하면 아두이노의 시리얼 입력을 파이썬이 받을 수 있다는 것을 알았다.

## Arduino 회로 구성

먼저 회로 구성은 아래와 같다. Arduino Uno, DHT11, LCD 모듈을 이용했고 [메카솔루션]에서 구입했다.

<div style="text-align: center;"><img src="{{ '/assets/img/arduino_circuit.png' | prepend: site.baseurl }}" alt=""></div>

매뉴얼에 있는 회로인데 실제로는 LCD 모듈에 백라이트 부분을 추가로 연결해주어야 한다. 실제 연결해보니 이런 모양새가 되었다.

<div style="text-align: center;"><img src="{{ '/assets/img/arduino_photo.jpg' | prepend: site.baseurl }}" alt=""></div>

아두이노에 들어간 코드는 아래와 같다.

```cpp
#include <LiquidCrystal.h>  // 액정 디스플레이 라이브러리(아두이노 내장)

// 온습도 센서(DHT11) 라이브러리
// https://github.com/adafruit/DHT-sensor-library/blob/master/DHT.h
#include "DHT.h"

// DHT11 초기화
DHT dht(8, DHT11);

// 액정 디스플레이 초기화
// LiquidCrystal(rs, enable, d4, d5, d6, d7)
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);  

int h;  // 습도
int t;  // 온도

void setup() {
  lcd.begin(16, 2);  // 16칸 2줄 LCD 디스플레이 사용
  Serial.begin(9600);  // 시리얼 통신 시작
}

void loop() {
  delay(2000);

  // 온습도 값 읽어들이기
  h = dht.readHumidity();
  t = dht.readTemperature();

  // 액정화면 표시
  lcd.setCursor(0, 0);
  lcd.print("Humi: "); lcd.print(h); lcd.print(" %");
  lcd.setCursor(0, 1);
  lcd.print("Temp: "); lcd.print(t); lcd.print(" C");

  // 시리얼 통신 표시
  Serial.print("{\"Humidity\": \""); Serial.print(h); Serial.print("\", ");
  Serial.print("\"Temperature\": \""); Serial.print(t); Serial.println("\"}");
}
```

## pySerial 활용

이제 아두이노는 2초마다 온도와 습도를 JSON 형식에 맞는 문자열로 시리얼 통신을 하고 있다. 이것을 파이썬으로 잡아낼 때 pySerial이 필요하다. 우선 pipenv를 통해 pySerial을 설치했다. 그리고 이 정보를 AWS Django 애플리케이션에 request를 보내야 하기 때문에 requests 패키지 또한 설치했다.

```shell
$ pipenv install pyserial requests
```

파이썬 코드는 아래와 같다.

```python
"""
request.py
    1. 아두이노의 시리얼 출력을 JSON으로 변환(현재시간 데이터 추가)
    2. JSON 데이터를 http://127.0.0.1:8000/ URL에 post 메소드로 request
    3. request를 보낼 때 csrf_token 추가
"""
from datetime import datetime
import json

import serial
import requests

URL = 'http://127.0.0.1:8000/'
arduino_port = '/dev/cu.usbmodemFD141'

ser = serial.Serial(
    port=arduino_port,  # 시리얼 포트
    baudrate=9600,
)

print('아두이노와 통신 시작')
print(f'아두이노의 접속 포트는 {arduino_port} 입니다.\n')

while True:

    try:
        if ser.readable():
            res = ser.readline()
            json_data = json.loads(res.decode()[:-1])
            json_data['time'] = datetime.now().strftime('%Y-%m-%dT%H:%M:%S')
            print(json_data)
            client = requests.session()

            client.get(URL)  # sets cookie

            if 'csrftoken' in client.cookies:
                csrf_token = client.cookies['csrftoken']
            else:
                csrf_token = client.cookies['csrf']

            sensor_data = dict(
                **json_data,
                csrfmiddlewaretoken=csrf_token
            )
            response = client.post(URL, data=sensor_data, headers=dict(Referer=URL))
            print(f'아두이노 데이터 {URL} 전송: {response}\n')

    except serial.serialutil.SerialException:
        print('아두이노가 연결되어 있지 않습니다.\n')
        break

    except requests.exceptions.ConnectionError:
        print('인터넷 전송이 불가능합니다.\n')
        continue

    except KeyboardInterrupt:
        print('아두이노 시리얼 통신을 중단합니다.\n')
        break
```

이 코드에서 신경을 쓴 부분을 정리해보았다.

1. 아두이노에서 JSON 형식의 문자열을 보내기 때문에 json 내장 라이브러리로 바로 파싱할 수 있었다. 다만 마지막 문자는 줄바꿈 문자이기 때문에 마지막 문자는 제외하고 파싱했다.
2. 아두이노 데이터를 받은 시각을 함께 전송하기 위해 datetime 내장 라이브러리를 사용했다.
3. Django 애플리케이션에 POST 방식으로 보내야 하기 때문에 csrf 토큰을 받아오는 코드를 추가했다. 먼저 GET 방식으로 요청을 보내서 토큰을 받고 토큰을 data에 추가해서 POST 방식의 요청을 보냈다.
4. 성공적으로 보냈는지 알기 위해서 respone의 HTTP 상태 코드를 볼 수 있도록 코드를 추가했다.

## DHT11의 아쉬움

안타깝게도 DHT11의 정확도로는 소수점 자리까지의 측정은 불가능했다. 그래서 계속 같은 값이 전송되었고 소수점 이하 자릿수의 온도 변화를 잡아낼 수 없었다. 더 정확도가 높은 부품은 DHT22가 있는데 다음에 구해서 써보면 좋을 것 같다.

[pySerial]: https://pythonhosted.org/pyserial/
[메카솔루션]: http://mechasolution.com/shop/goods/goods_view.php?goodsno=540978
