# 해결해야할 것 !
> `Audio 파일` CRUD with AWS `S3`.<br>
Buffer와 Streams에 대해 한번 더 정리하고 넘어간다.

<br>

# 해결 과정 추측해보기
1. `koa-multer`를 이용해서 서버상의 폴더에 업로드되게 만든다.
2. AWS S3 테스트 계정을 만들고 필요한 설정들을 한다.
3. `1번`의 업로드 장소를 `2번`에서 만든 S3로 변경한다.<br>
AWS S3의 CRUD는 multer와 많이 다를 듯하다. 일단 부딛쳐봐야함.
업로드는 어떻게 쉽게 될듯함. __but__ streaming은 난관이 있을듯함.
4. 다른 API에서 이용할 수 있도록 모듈화한다.
5. Test Code를 작성한다. __!! MUST !!__ <br>
6. Documentation을 작성한다.

<br>

# 문제 해결하기

## 이용할 자료
- koa-multer:<br>https://www.npmjs.com/package/koa-multer<br>
디테일한 multer 자료: https://github.com/expressjs/multer<br>
AWS S3 for JavaScript: https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/s3-example-creating-buckets.html
- s3/node file upload example:<br>
https://charlie-choi.tistory.com/248<br>
https://yangeok.github.io/node.js/2019/03/22/s3-upload.html<br>
- https://medium.com/@varad911/aws-s3-file-upload-and-download-in-node-js-with-koa-729eb8ae280b
-

## 설치
```npm i koa-multer```



---
---
---
# 최고의 Media Streaming 방법은 무엇인가.
---
# 키워드
- `progressive download` vs `streming`
