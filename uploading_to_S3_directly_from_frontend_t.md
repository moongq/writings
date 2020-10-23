# 웹 혹은 모바일에서 Amazon S3에 직접 파일 업로드하기.
원본: https://aws.amazon.com/blogs/compute/uploading-to-amazon-s3-directly-from-a-web-or-mobile-application/ by [[James Beswick]](https://aws.amazon.com/blogs/compute/author/jbeswick/)

웹과 모바일 어플리케이션에서, 유저에게 데이터를 업로드할수 있게 해주는 것은 흔한 일입니다. 당신의 어플리케이션은 PDF와 같은 여러 문서들, 사진이나 비디오와 같은 미디어를 업로드할 수 있도록 허용합니다. 모든 현대 웹 **서버** 기술은 이러한 작업들이 가능합니다. 아래 사진의 흐름대로 작동합니다.

![](https://images.velog.io/images/moongq/post/8fb7e920-e785-4ac7-82a2-38f059fe0053/uploadingToS3_1.PNG)
1. 유저는 어플리케이션 서버에 파일을 업로드합니다.
2. 어플리케이션 서버는 업로드할 파일을 임시적인 공간에 저장해둡니다. 
3. 어플리케이션은 2번에서 저장한 파일을 영구적인 저장을 위해 데이터베이스, 파일 서버 혹은 객체 저장소에 저장합니다.

과정 자체는 간단합니다. 그런데 간혹 웹 서버 성능에 아주 큰 부작용이 발생할 수도 있습니다. 미디어는 보통 크기가 큽니다. 그래서 이러한 미디어 전송 작업은 네트워크 I/O(input and output)과 CPU time에 큰 부하를 줄 수 있습니다. 그리고 파일을 업로드하는 당사자는 파일 전송이 제대로/정확히 되고 있는지 확실히 관리해야 합니다.

이것은 매우 도전적인 과제입니다. 예를 들자면 새해 기념 메시지가 있습니다. 12월 31일 엄청난 인원이 기념 메시지를 같은 시간에 보낸다면, 게다가 메시지에 미디어 혹은 다른 파일을 함께 보낸다면, 이런 상황에서는 서버의 크기를 늘리는 작업이 필요하게 됩니다. 그리고 충분한 네트워크 대역폭 또한 필요합니다.

> 이러한 상황에 Client단에서 Amazon S3에 파일을 직접 업로드한다면, 수많은 파일들이 웹서버를 거치는 과정을 피할 수 있게 됩니다.

파일들이 서버를 거치치않고 업로드 된다면 엄청난 양의 네트워크 트레픽, 서버 CPU 사용량이 줄어듭니다. 그리고 당신의 서버는 그 시간에 다른 작업들을 처리할 수 있게 되죠. 또한 S3는 가용성과 내구성이 뛰어나 사용자 업로드를위한 이상적인 영구 저장소랍니다.

이번 블로그 글에서 저는 serverless uploads를 어떻게 구현하는지, 그리고 이 방식의 장점들을 보여드릴 것입니다. 

## serverless uploading to S3 훑어보기.
당신이 S3 bucket에 직접 업로드할 때, 당신은 첫째로 아마존 서비스에게 `signed URL`을 요청해야 합니다. `signed URL`을 통해 파일을 직접 업로드할 수 있게 됩니다. 프론트엔드에서 이것은 투-스텝 방식입니다.

![](https://images.velog.io/images/moongq/post/7c5d614c-d75d-41cf-82e1-7687c43f7224/uploadingToS3_2.PNG)
1. `signed URL`을 받을 수 있는 API를 요청합니다. (이 글에서는 서버레스 방식이라 Amazon Lambda를 통해 signedURL을 받습니다. 보통(서버레스가 아닌 웹)에서는 API 서버를 통해 signedURL을 전달받습니다.)
2. signedURL을 이용해서 어플리케이션에서 S3 bucket으로 직접 업로드합니다.


## 자! 이제 백엔드에서는 무엇을 해주면 되느냐.
> signedURL을 발급해주는 API를 만들면 됩니다. !

## S3 업로드 방식 이해하기
S3를 통해 객체를 업로드할 때면 당신은 CORS(Cross-Origin Resource Sharing)을 위한 세팅을 해줘야합니다. CORS 규칙은 bucket안에 XML 문서로 정의되어야 합니다. 
![](https://images.velog.io/images/moongq/post/b9b42b47-981b-45ec-bc1b-5917fb40c976/uploadingToS3_3.PNG)

`S3 버킷 -> Permissions -> CORS configuration`에 들어가면 설정할 수 있습니다.
제 버킷은 아무것도 설정하지 않아 텅 비어있습니다. 아래 화면은 CORS configuration 실예입니다.

```XML
S3UploadBucket:
    Type: AWS::S3::Bucket
    Properties:
      CorsConfiguration:
        CorsRules:
        - AllowedHeaders:
            - "*"
          AllowedMethods:
            - GET
            - PUT
            - HEAD
          AllowedOrigins:
            - "*"
```
이 예는 모든 headers와 origin을 허용합니다. 실제 프로덕션에서는 더욱 강력한 정책을 사용하셔야합니다 !

다음은 signedURL을 받는 API 코드입니다.
```js
const AWS = require('aws-sdk')
AWS.config.update({ region: process.env.AWS_REGION })
const s3 = new AWS.S3()
const URL_EXPIRATION_SECONDS = 300

// Main Lambda entry point
exports.handler = async (event) => {
    return await getUploadURL(event)
}

const getUploadURL = async function(event) {
    const randomID = parseInt(Math.random() * 100000000)
    const Key = `${randomID}.jpg`

    // Get signed URL from S3
    const s3Parmas = {
        Bucket: process.env.UploadBucket,
        Key,
        Expires: URL_EXPIRATION_SECONDS,
        ContentType: 'image/jpeg'
    }
    const uploadURL = await s3.getSignedUrlPromise('putObject', s3Params)
    return JSON.stringify({
        uploadURL: uploadURL,
        Key
    })
}
```
위 함수는 업로드된 객체의 이름, key를 랜덤 숫자를 이용해서 정의합니다. `s3Params` 객체는 받아 들일수 있는 컨텐츠의 타입을 정의할뿐만 아니라 key의 만료까지 정의합니다. 위 예에서는 300초 동안만 key가 유효합니다.
함수의 return 값으로 signedURL과 key가 JSON 객체로 전달됩니다.

`signed URL`은 파일을 업로드할 수있게 해주는 보안 토큰을 가지고 있습니다. 이 보안 토큰을 성공적으로 만들기 위해 위 함수의 `getSignedUrlPromise` 코드는 `s3:putObject` 권한을 가지고 있어야 합니다.

업로드된 객체는 파라미터에 정의된 `content-type`과 `file name`과 반드시 같아야합니다.
만약 expires 시간을 설정하지 않으면 디폴트 15분으로 설정됩니다.

프론트엔드에서 `signedURL`을 API로부터 받게 되면 프론트엔드에서는 PUT method를 이용해서 binary data를 업로드할수 있게 됩니다.
```js
let blobData = new Blob([new Unit8Array(array)], { type: 'image/jpeg'})
const result = await fetch(signedURL, {
    method: 'PUT',
    body: blobData
})
```
이 시점에 어플리케이션은 S3 Service와 직접적으로 연결됩니다.**( 서버와 연결되는 것이 아닙니다 !)** 그리고 S3는 200 상태코드를 리턴하고 업로드가 성공적으로 끝나게 됩니다.

## ACL(Access control list) 수정하기, 객체를 퍼블릭 읽기 상태로 바꾸기
지금까지의 진행 상황에서 업로드된 객체들은 공개적으로 접근할 수 없습니다. url을 이용해서 모든 사람볼 수 있게 하기 위해서 ACL을 수정해야 합니다. `s3.getSignedUrl` 함수의 파라미터를 설정하면 됩니다. 간단합니다.

```js
const s3Params = {
    Bucket: process.env.UploadBucket,
    Key,
    Expires: URL_EXPIRATION_SECONDS,
    ContentType: 'image/jpeg',
    ACL: 'public-read' // Check here !!
}
```
> 파일이 업로드된 이후의 접근 권한은 ACL에 작성
signedURL의 권한 관리는 BucketPolicy에 작성하세요 !

## 결론
Client단에서 파일을 직접 S3에 업로드할수 있게 되면 서버가 정말 가벼워집니다. 가벼워진 서버는 정말 필요한 다른 작업을 먼저 수행할 수 있어 어플리케이션 전체적으로 수용할 수 있는 작업의 양이 증가합니다. 왠만하면 파일 업로드는 프론트단에서 하는걸로 !