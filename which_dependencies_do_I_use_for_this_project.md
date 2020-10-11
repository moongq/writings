> 만들면서 새로 접한 library들을 파악하자. 회사에서 원하는 기한까지 mvp를 만들려면 일단 정해진 도구목록으로 진행해야 한다. 맘에 안드는 dependencies는 mvp이후에 논의하는 걸로.

<br>

# 처음써보는 도구들
- awilix
- apidoc
- winston
- yenv
- jest
- pm2
- shx
- webpack

<br>

## Awilix
의존성 주입 라이브러리다. 한글자료는 기대도 안했지만, 영어자료도 많지 않다. ㅎㅎ... 볼만한 자료는 공식 awilix github밖에 없다. 공식 설명이 헷갈릴수 있는데.. 구글에 검색해도 깊은 설명이 딱히 없다. 어쩔 수 없이 공식문서만 계속 읽으면서 머리속에 집어넣고 있다. __읽다보면 글쓴이의 의도를 이해하겠지 뭐..__<br> DI를 접하고 재밌다고 느꼇다. `"내 코드가 고급져질 수 있겠구나.. !"` 와 같은 기대감이랄까. 그래서 깊이 공부하고 사용하고 싶지만. 그럴 시간이 없다. 3달도 안남은 시간에 완성을 해야하니 CTO가 적용한 정도만 익히고 넘어가려 한다.
awilix가 적용된 코드는 몇 곳없고 정말 간단하게만 이용하고 있다. mongoose, redis 같은 외부 도구들 세팅할때만 쓰고 다른 곳에서는 딱히 쓰지 않는다. (쓸거면 다쓰지.. 참 애매하게 쓰시네.. mvp끝나고 건의하자 !)<br>
<br>
## Apidoc
> Inline Documentation for RESTful web APIs<br>
apidoc creates a documentation from API annotations in your source code.

```
/**
 * @api {get} /user/:id Request User information
 * @apiName GetUser
 * @apiGroup User
 *
 * @apiParam {Number} id Users unique ID.
 *
 * @apiSuccess {String} firstname Firstname of the User.
 * @apiSuccess {String} lastname  Lastname of the User.
 */
```
apiDoc gives you the ablity to attach a version number to an API so you can easily track changes between versions.
<br><br>
ApiDoc 설명이다. __이렇게만 하면 documentation html파일까지 만들어준다.__ 정말 정말 정말 쉽고 편하다. 게다가 소스코드위에 바로 붙여버리니 소스코드가 바뀌면 위에 붙어있는 어노테이션만 같이 바꿔주면 된다. 기존에는 documentation 파일을 따로 모아뒀었다. 따로 두다보니 소스코드는 바뀌었는데 documentaion은 까먹고 수정안해놓을 때가 엄~청 많았다. apidoc을 쓰면 덜 까먹게 해줄거라고 본다. 더이상의 설명은 필요 없다. !
<br><br>

## winston
