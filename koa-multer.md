# 해결해야할 것 !
> `Image, Audio 파일` 업로드와 읽어오기 with `AWS S3`.<br>

# 해결 과정 추측해보기
1. multer를 쓰면 되나? nodejs에서는 multer로 파일 업로드를 하던데?
2. 파일 업로드는 서버에서 하는건가? 프론트에선 못하나?
3. S3 업로드할 때, 읽어올 때 권한 관리는? 아무나 다 받을 수 있으면 요금 폭탄 맞는거 아닌가?

# 해결 과정
## 1. multer를 쓰면 되나? nodejs에서는 multer로 파일 업로드를 하던데?
구글을 다 찾아보니 `multer`와 `aws-sdk`를 이용해서 서버에서 업로드를 한다. 좀더 쉬운 방법은 `multers3`라는 npm module이 있다. 정~말 쉽게 S3에 업로드하게 해준다. AWS S3 권한 설정하는게 훨씬 어려울 정도로 쉽게 만들어져있다. 업로드는 뚝딱 할 수 있다. (multer에 대한 자료가 필요하다면 밑의 참고 자료를 참고!)

## 2. 파일 업로드는 서버에서 하는건가? 프론트에선 못하나?
음 서버에서 업로드하면 파일이 서버를 거쳐야 한다는 건데? 파일들은 용량이 꽤 큰데? 서버를 안거치는 방법이 있겠지?
**있다.** 그게 바로 `presignedURI`. 찾아보니 AWS 블로그에 좋은 글이 올라와 있다. [내 번역글로 이동 ! Uploading to Amazon S3 directly from a web or mobile application](https://aws.amazon.com/blogs/compute/uploading-to-amazon-s3-directly-from-a-web-or-mobile-application/)


<br>

## 참고 자료
- koa-multer:<br>
  - https://www.npmjs.com/package/koa-multer<br>
- 디테일한 multer 자료: <br>
  - https://github.com/expressjs/multer<br>
- AWS S3 for JavaScript:<br> 
  - https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/s3-example-creating-buckets.html
- s3/node file upload example:<br>
  - https://charlie-choi.tistory.com/248<br>
  - https://yangeok.github.io/node.js/2019/03/22/s3-upload.html<br>
  - https://medium.com/@varad911/aws-s3-file-upload-and-download-in-node-js-with-koa-729eb8ae280b
- presignedURI<br>
  - Upload a File to an S3 Pre-Signed URL with React Native<br>https://codedaily.io/tutorials/179/Upload-a-File-to-an-S3-Pre-Signed-URL-with-React-Native<br>
  - Uploading to Amazon S3 directly from a web or mobile application<br>https://aws.amazon.com/blogs/compute/uploading-to-amazon-s3-directly-from-a-web-or-mobile-application/
