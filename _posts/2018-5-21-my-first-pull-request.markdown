---
layout: post
title:  "나의 첫 번째 풀 리퀘스트"
date:   2018-5-21
categories: [opensource]
---

<p class="intro"><span class="dropcap">패</span>스트캠퍼스 웹 프로그래밍 스쿨을 등록하게 되어서 git으로 오픈소스에 기여하는 방법을 배웠다.</p>

마침 마이크로소프트의 Azure 관련 문서 중에서 추가했으면 하는 내용이 있었는데 내가 직접 풀 리퀘스트를 날릴 수는 없을까 알아보았다. 찾아보니 마이크로소프트에 [azure-docs]라는 저장소가 있었고 내가 수정하고 싶은 문서도 있었다. 수정하고 싶은 내용은 아래와 같다.

```java
...
protected Face[] doInBackground(InputStream... params) {
    try {
        publishProgress("Detecting...");
        Face[] result = faceServiceClient.detect(
                params[0],
                true,         // returnFaceId
                false,        // returnFaceLandmarks
                null           // returnFaceAttributes: a string like "age, gender"
        );
...
```

[Face API Java for Android tutorial]라고 하는 문서 중 일부이다. 마이크로소프트의 Face API를 안드로이드 네이티브 환경에서 사용하는 방법을 설명하고 있다. Face API는 마이크로소프트가 만든 인공지능 서비스로서 인물 사진의 URL 주소를 마이크로소프트 Azure 서버로 전송하면 인물에 대한 정보(얼굴 인식, 성별, 나이 등)를 받을 수 있다. 위 코드는 바로 API에게서 정보를 받아내는 함수라고 할 수 있다.

내가 추가하고 싶었던 부분은 faceServiceClient.detect 메소드였는데 위 메소드대로 보내면 얼굴 인식만 가능하다. 왜냐하면 4번째 패러미터를 null로 설정했기 때문이다. Face API 사용자라면 얼굴 인식 외에 여러 정보를 추가로 얻고 싶을 것이다. 그러기 위해서는 여기에 적절한 패러미터를 넣어야 한다.

그런데 이 패러미터는 FaceServiceClient.FaceAttributeType 열거형(enum)의 배열을 받는다. 처음 사용하는 사람들에게는 어려울 수 있는 부분이다. 그래서 아래와 같이 내가 주석을 달았다.

```java
Face[] result = faceServiceClient.detect(
        params[0],
        true,         // returnFaceId
        false,        // returnFaceLandmarks
        null           // returnFaceAttributes: a string like "age, gender"
        /* If you want value of FaceAttributes, try adding 4th argument like below.
        new FaceServiceClient.FaceAttributeType[] {
          FaceServiceClient.FaceAttributeType.Age,
          FaceServiceClient.FaceAttributeType.Gender }
        */				
);
```

주석을 달기 위해서 [azure-docs]를 포크하고 새로운 브랜치를 만든 후에 풀 리퀘스트를 보냈다. 초조하게 기다리고 있었는데 다행히도 저장소에서 병합을 해주었다! 증거(?)는 아래에서 볼 수 있다.

[Add a comment in detectAndFrame method #8847]

앞으로는 코드에 직접 기여할 수 있기를 바라본다! :)

[azure-docs]: https://github.com/MicrosoftDocs/azure-docs
[Face API Java for Android tutorial]: https://docs.microsoft.com/en-us/azure/cognitive-services/face/tutorials/faceapiinjavaforandroidtutorial
[Add a comment in detectAndFrame method #8847]: https://github.com/MicrosoftDocs/azure-docs/pull/8847
