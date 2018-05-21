---
layout: post
title:  "나의 첫 번째 풀 리퀘스트"
date:   2018-5-21
categories: [opensource]
---

<p class="intro"><span class="dropcap">패</span>스트캠퍼스 웹 프로그래밍 스쿨을 등록하게 되었는데 과정 중에서 git으로 오픈소스에 기여하는 방법을 배웠다.</p>

마침 마이크로소프트의 Azure 관련 문서 중에서 추가했으면 하는 내용이 있었는데 잘 되었다는 생각이 들었다. 마이크로소프트의 문서라면 github에 저장소가 있을 것이다. 그렇다면 내가 직접 풀 리퀘스트를 날릴 수는 없을까 알아보았는데 역시 마이크로소프트의  [azure-docs]라는 저장소에 수정하고 싶은 문서가 있었다. 수정하고 싶은 내용은 아래와 같다.

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

[Face API Java for Android tutorial]라고 하는 문서 중 일부이다. 마이크로소프트의 [Face API]를 안드로이드 네이티브 환경에서 사용하는 방법을 설명하고 있다. [Face API]는 마이크로소프트가 만든 인공지능 서비스이다. 튜토리얼 대로 진행하면 인물 사진 파일의 스트림을 마이크로소프트 Azure 서버로 전송해서 인물에 대한 정보(얼굴 인식, 성별, 나이 등)를 받을 수 있다. 위 코드는 API에게서 정보를 받아내는 부분이라고 할 수 있다.

내가 추가하고 싶었던 것은 faceServiceClient.detect 메소드에 대한 주석이었다. 튜토리얼이기 때문에 작성자는 가장 기본적인 기능(얼굴 인식)만 구현해놓았다. 그래서 얼굴 인식 외 다른 정보는 받을 수 없는데 그 이유는 4번째 인자를 null로 설정했기 때문이다. 아마 [Face API] 사용자라면 얼굴 인식 외에 여러 정보를 추가로 얻고 싶을 것이다. 그러기 위해서는 여기에 적절한 인자를 넣어야 한다.

FaceServiceClient 클래스를 조금 더 살펴보니 detect() 메소드의 4번째 인자는 FaceServiceClient.FaceAttributeType 열거형(enum)의 배열을 받는다. 이 내용은 처음 사용하는 사람들에게는 어려울 수 있는 부분이다. 그래서 아래와 같이 내가 주석을 달았다.

```java
...
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
...
```

내가 추가한 주석이 실제 문서에도 추가되도록 [azure-docs]를 포크하고 새로운 브랜치를 만든 후에 풀 리퀘스트를 보냈다. 초조하게 기다리고 있었는데 다행히도 저장소에서 병합을 해주었다! 증거(?)는 아래에서 볼 수 있다.

[Add a comment in detectAndFrame method #8847]

앞으로는 코드에 직접 기여할 수 있기를 바라본다! :)

[Face API]: https://azure.microsoft.com/ko-kr/services/cognitive-services/face/
[azure-docs]: https://github.com/MicrosoftDocs/azure-docs
[Face API Java for Android tutorial]: https://docs.microsoft.com/en-us/azure/cognitive-services/face/tutorials/faceapiinjavaforandroidtutorial
[Add a comment in detectAndFrame method #8847]: https://github.com/MicrosoftDocs/azure-docs/pull/8847
