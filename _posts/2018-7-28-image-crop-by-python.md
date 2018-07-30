---
layout: post
title:  "python pillow 패키지를 이용해 이미지 자르기"
date:   2018-7-28
categories: [python]
---

<p class="intro"><span class="dropcap">p</span>ython의 pillow 패키지를 이용하면 이미지를 쉽게 조작할 수 있다. 이 패키지를 이용해서 큰 이미지를 격자 모양으로 조각조각내는 작업을 진행해보자.</p>

인터넷에서 구글로 서버 개발자를 위한 이모티콘을 찾아보니 이런 이모티콘을 찾을 수 있었다.

![폭스 로퍼씨의 개발일지](https://t1.daumcdn.net/cfile/tistory/9985B7395A9E50C833)

이렇게 하나의 이미지 파일에 여러 이모티콘이 있으니 파이썬으로 이미지를 하나의 이모티콘씩 잘라서 쓸 수는 없을까? 구글로 검색해봤더니 pillow 패키지를 이용하면 가능했다. 이 패키지는 Django에서도 사용하는 패키지이기 때문에 쓰기에 어렵지 않았다.

## Image 모듈의 메소드

pillow 패키지는 `pipenv install pillow` 명령어로 사용할 수 있지만 파이썬 코드에서는 `PIL'이라는 패키지명을 사용해야 한다. 이 포스트에서는 Image 모듈을 사용하려고 한다. Image 모듈의 메소드들은 아래와 같은 것들이 있다.

1. Image.open(path) : 해당 path의 이미지 파일을 읽어들여서 Image 인스턴스를 반환한다.
2. Image.crop(bbox) : bbox는 (왼쪽 시작지점, 위쪽 시작지점, 가로폭, 세로폭)의 의미를 갖고 있는 정수형 튜플이다. bbox의 정보대로 Image 인스턴스의 이미지를 잘라내고 잘라낸 이미지에 대한 Image 인스턴스를 반환한다.
3. Image.save(path) : 해당 path 경로로 Image 인스턴스를 파일로 저장한다.

간단하게 사용할 수 있어서 jupyter notebook을 통해 바로 구현해보았다.

## Implementation

```python
from PIL import Image
import math
import os

def vertical_slice(image_path, out_name, outdir, slice_size):
    img = Image.open(image_path)
    width, height = img.size
    upper = 0
    left = 0
    slices = int(math.ceil(height/slice_size))
    count = 1
    for slice in range(slices):
        if count == slices:
            lower = height
        else:
            lower = int(count * slice_size)

        bbox = (left, upper, width, lower)
        working_slice = img.crop(bbox)
        upper += slice_size
        working_slice.save(os.path.join(outdir, "slice_" + out_name + "_" + str(count)+".png"))
        count += 1

def horizontal_slice(image_path, out_name, outdir, slice_size):
    img = Image.open(image_path)
    width, height = img.size
    upper = 0
    left = 0
    slices = int(math.ceil(width/slice_size))
    count = 1
    for slice in range(slices):
        if count == slices:
            right = width
        else:
            right = int(count * slice_size)  
        bbox = (left, upper, right, height)
        working_slice = img.crop(bbox)
        left += slice_size
        working_slice.save(os.path.join(outdir, "slice_image_" + str(i) + '_' + out_name + "_" + str(count)+".png"))
        count += 1
```

두 메소드는 각각 세로로, 가로로 자르는 메소드이다. 먼저 세로로 자르고 자른 파일들을 다시 가로로 자르면 각각의 이모티콘을 잘라낼 수 있을 것이다.

## 주의!

위 이모티콘 이미지는 구글에서 검색할 수 있기에 링크로 가져왔지만 엄연히 저작권이 있는 자료이다. 때문에 메소드만 구현하고 실제로 자르는 코드는 구현하지 않았다. (구현하지 않았다고 했지만 사실 위 코드를 이해할 수 있다면 식은 죽 먹기일 것이다.)
