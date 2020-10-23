# DI framework를 하나 파보자. - awilix
`typeDI`를 파고 싶었지만, 이번 프로젝트는 `awilix-koa`를 이용한다...
나는 주로 대세 프레임워크를 쓰는 편인데 이번 프로젝트의 모듈들은 대부분 주류가 아니어서 아쉽다. 그래도 비주류 모듈의 장점이 있다면 잘 정리된 코드 혹은 글이 많지 않아 코드 복붙을 할 확률이 현저히 낮아진다.!! 강제로 코드 복붙이 불가능해진다. 그리고 프레임워크 내부 코드를 더 많이 보게 된다. 설명도 부족하고 잘돌아가는 코드가 인터넷에 별로 없기 때문에 내부적으로 어떻게 짜여있는지, 어떤 모듈들을 쓰는지 하나하나 파악해봐야만 내 프로젝트에 적용시킬수 있다. 
> 주류 모듈보다 비주류 모듈들이 공부가 훨씬 많이 된다. <br>Awilix-koa를 쓰기 전에 Awilix부터 공부하자.

# Installation
npm 설치
```npm install awilix --save```

# Usage



























////
////
////
## Awilix-koa

# Installation
```npm install --save awilix-koa```
(Requires Node v6 or above)

# Basic Usage
Koa 앱에 아래 미들웨어를 추가해보자.
```
const { asClass, asValue, createContainer } = require('awilix');
const { scopePerRequest} = require('awilix-koa');

const container = createContainer();
container.register({
    // `.scoped()` = 매 request마다 새로운 객체가 생성된다는 것 의미한다.
    todosService: asClass(TodoService).scoped()
});

// Awilix container를 전달해주는 미들웨어.
// 'context'에 범위가 정해진 container를 결합시킨다.
app.use(scopedPerRequest(container));

// 이제부터 request마다 다른 데이터들을 붙여줄수 있게 됐다.
// 잘 이해안되지만 위의 `scopedPerRequest()`이후에는 개별 request마다 개발 데이터를 설정할수 있게 된다는 뜻으로 파악된다.
app.use((ctx, next) => {
    ctx.state.container.register({
        user: asValue(ctx.state.user) // 어딘가 만들어져있을 권한관리 미들웨어로부터 전해졌다.
    })
    return next()
});
```
그리고 route handler에서는...
```
const { makeInvoker } = require('awilix-koa');

function makeAPI({ todosService }) {
    return {
        find: ctx => {
            return todosService.find().then(result => {
                ctx.body = result
            })
        }
    }
}

const api = makeInvoker(makeAPI);

// 개개별의 request들에게 범위가 지정된 객체를 전달해주는 `makeAPI`를 호출할 미들웨어를 만들었다.
router.get('/todos', api('find'));
``` 

# Awesome Usage
