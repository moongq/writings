## [끝내지 못한 글]

# 음원/오디오 Streaming in Node.js

## 해결해야할 일
> AWS S3에 업로드되어 있는 음원 파일을 웹에서 실행/재생시킨다.<br>

<br>

## 탐색 과정  
### 1. "stream이 떠오랐다. 오디오나 동영상이나 비슷하겠지?"<br>
__탐색 결과__: <br>비슷하긴 하다. 그러나 다르다. 일단 둘은 **데이터 크기**에서부터 큰 차이가 있다. 동영상은 크기가 매우 크다보니 stream을 이용해서 전달하는 경우가 많다. + Stream과 Buffer를 동시에 사용하기도 한다. 성능이 좋다고 한다. (참고: [Buffering improves Streaming](https://itstillworks.com/difference-between-buffering-streaming-27518.html))<br><br>
### 2. 점점 오디오는 Streaming을 안하는 느낌의 글이 많이 보인다.
Audio media를 `Streming`할 수 있다. 하지만 Streaming하려면. mp3 파일을 .m3u8로 변환해야 한다. 주로 `Video Streming`에서 하는 여러 과정이 Audio에도 추가적으로 생긴다는 말이다. 그렇게 해도 된다. 하지만 Audio는 대부분이 데이터가 (비디오에 비해) 그리 크지 않다. 각 자료/데이터마다의 특성이 있으면 마땅히 이용해야 하는 것이 미덕이다. 그러니 Audio에서는 100% 순수한 Streaming 방식이 아닌 `Progressive Download`을 사용해도 된다.<br><br>
### 3. **progressive download** ??  

<br>

#
#

## 키워드
- HLS (HTTP Live Streaming)
- RTMP
- MediaConvert
- m3u8
- progressive download
- presigned URL
<br>

## 더 디테일한 정보
- VOD 자동화 파트 1<br>https://cloud.hosting.kr/techblog_190122_aws-elemental-mediaconvert/
- HTTP Live Streaming<br>https://ko.wikipedia.org/wiki/HTTP_%EB%9D%BC%EC%9D%B4%EB%B8%8C_%EC%8A%A4%ED%8A%B8%EB%A6%AC%EB%B0%8D
- MediaConvert<br>https://www.amazonaws.cn/en/mediaconvert/features/
- s3mediavault<br>https://s3mediavault.com/demo/s3-streaming-audio-player/
- 스트리밍이란?:<br>https://typeof-bong.tistory.com/2
- streaming 과 progress download를 헷갈려하는 당신에게.<br>https://stackoverflow.com/questions/50776300/how-can-i-stream-audio-files-mp3-from-aws-s3<br>
- Media buffering, seeking, and time ranges [Mozilia]:<br>https://developer.mozilla.org/en-US/docs/Web/Guide/Audio_and_video_delivery/buffering_seeking_time_ranges