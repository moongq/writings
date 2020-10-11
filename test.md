[29.sep.2020]

`JavaScript dependency injection in Node.js – friends or foes?`번역글입니다. 
출처: https://tsh.io/blog/dependency-injection-in-node-js/
<br>

# DI
__자바스크립트의 의존성 주입은 잘알려진 기술이다. 이 의존성 주입은 모듈의 확장성과 독립성에 큰 도움을 주고 있다. 여러 프로그래밍 언어, 프레임워크에서 의존성주입이 쓰이고 있는데 Node.js에서는 그리 유명하지 않다. 이유를 따져보면 사람들이 Node.js에서의 의존성 주입을 잘못생각하고 있어서 그런것 같다. 이제부터 이 글에서 의존성 주입이 어떻게 너의 삶을 편안하게 해줄수 있는지 파악해보려 한다.__

당신이 `Node.js`를 처음 배울 때 아마 `module pattern`도 배울 것이다. Node.js에선 이 module pattern 때문에 의존성 주입이 그리 필요하지 않아보인다. 왜나면`require`가 있기 때문에!. 하지만 난 그리 동의 하지 않는다. 왜냐고?
이제부터 진짜 __Node.js에서의 의존성 주입__이 뭔지 알아보자.

## What is dependency injection?
의존성 주입은 하나의 패턴이다. 만약에 의존성들을 인자들로 전달해준다면, 모듈안에서 의존성들을 불러오거나 새로 만드는 것을 피할수 있다.

어려울수 있지만 차근히 봐보자. 아래 코드는 간단한 service module이다.

```js
//users-service.js
const User = require('./User');
const UsersRepository = require('./users-repository');

async function getUsers() {
  return UsersRepository.findAll();
}

async function addUser(userData) {
  const user = new User(userData);
  
  return UsersRepository.addUser(user);
}

module.exports = {
  getUsers,
  addUser
}
```
괜찮아 보이지? 이 `users-service.js`는 비지니스 로직을 책임지고 있고, `user repository`는 데이터들에 대해 책임지고 있다. 그런데 위 코드에선 두가지 문제점이 있다.

첫번째는 `service`가 특정 `repository`와 연결돼있다는 것. 만약에 우리가 다른 repository로 바꾸고 싶다면, 위의 코드를 싹다 바꿔야할거다. 싹다 바꿔야한다는건 확장성이 떨어진단걸 의미한다.

두번째는 모듈의 테스트가 힘들어진다는 것. `getUser` 메소드가 잘작동하는지 확인하려면, usersRepository에 대한 가짜 객체를 만들어줘야하는데. Jest의 Mock를 이용해도 되고 Sinon을 이용해도 된다. 그런데 기왕이면 외부 라이브러리를 안쓰는게 훨씬 편한 테스트 방식이지 않을까? 의존성주입을 외부 라이브러리 이용을 최소화할 수 있게 된다.

```js
const UsersRepository = require('./users-repository');
const UsersService = require('./users');
const sinon = require('sinon') ;
const assert = require('assert');

describe('Users service', () => {
  it('gets users', async () => {
    const users = [{
      id: 1,
      firstname: 'Joe',
      lastname: 'Doe'
    }];

    sinon.stub(UsersRepository, 'findAll', () => {
      return Promise.resolve(users)
    });

    assert.deepEqual(await UsersService.getUsers(), users);
  });
});
```
코드가 더 복잡해보이지 않나? 의존성 주입을 써서 위 코드를 고쳐보자.
우리가 해야할 건 `usersRepository`를 직접 인자로 넘겨주는게 다다.
```js
const User = require('./User');

function UsersService(usersRepository) { // check here
  async function getUsers() {
    return usersRepository.findAll();
  }
  
  async function addUser(userData) {
    const user = new User(userData);
  
    return usersRepository.addUser(user);
  }
  
  return {
    getUsers,
    addUser
  };
}

module.exports = UsersService
```
이제 `UserService`는 `repository`와 엮여있지 않는 상태다. 다만 `usersRepository`를 전달받아야만 한다. 이렇게 변경된 게 테스트에 큰 영향을 미치게 된다. 아래는 새로운 Test Code.
```js
const UsersService = require('./users');
const assert = require('assert');

describe('Users service', () => {
  it('gets users', async () => {
    const users = [{
      id: 1,
      firstname: 'Joe',
      lastname: 'Doe'
    }];
    
    const usersRepository = {
      findAll: async () => {
        return users
      }
    };
    
    const usersService = new UsersService(usersRepository);
    
    assert.deepEqual(await usersService.getUsers(), users);
  });
});
```
위 코드를 보면 `sinon`대신 `usersRepository`를 이용했다. 굳이 어렵게 외부 라이브러리를 쓸 필요가 없어진거지.

그리고 한가지 더 이점이 생겼다. service와 repository를 `decoupleing`하게 되면서 당신이 원하는 repository로 언제든지 편하게 적용할수 있게 됐다.

## Dependency injection in Node.js - classes vs functions

의존성 주입이 특히 Node.js 환경에서 유명하지 않은 또다른 이유가 있다. 사람들이 의존성 주입을 OOP(객체지향프로그래밍)만을 위한 컨셉이라고 생각하는 경향! 바로 그것이다. 이건 절대적으로 틀렸다고 본다. !

클래스 형태에서 의존성 주입을 진행하는 것은 아주 분명하고 쉽다. 클래스에는 생성자(constructor)라는게 있으니까. 
의존성을 주입할 때 개별 의존성을 하나 하나 인자로 전달하는 것보다 객체로 감싸서 한번에 주는게 더 좋다.(우리 회사/TSH는 이 방식을 좋아한다.)
```js
class UsersService {
  constructor({ usersRepository, mailer, logger }) {
    this.usersRepository = usersRepository;
    this.mailer = mailer;
    this.logger = logger;
  }

  async findAll() {
    return this.usersRepository.findAll();
  }

  async addUser(user) {
    await this.usersRepository.addUser(user);
    this.logger.info(`User created: ${user}`);
    await this.mailer.sendConfirmationLink(user);
    this.logger.info(`Confirmation link sent: ${user}`);
  }
}

module.exports = UsersService;

const usersService = new UsersService({
  usersRepository,
  mailer,
  logger
});
```
당신이 원하는 의존성을 선택, 주입하기 훨씬 쉬워졌다. 그리고 `TypeScript`를 쓴다면 더 편해진다. 아래 코드를 보자.
```js
type UsersDependencies = {			// Here is all dependencies.
  usersRepository: UserRepository
  mailer: Mailer
  logger: Logger
};

export class UserService {
  constructor(
  	private dependencies: UsersDependenceis			// looks better isnt it?
  ) {}
  
  async findAll() {
    return this.dependencies.usersRepository.findAll();
  }
  
  async addUser(user) {
    await this.dependencies.usersRepository.addUser(user);		// more easy to access dependencies
    this.dependencies.logger.info(`User created: ${user}`);
    await this.dependencies.mailer.sendConfirmationLink(user);	// more easy to access dependencies
    this.dependencies.logger.info(`Confirmation link sent: ${user}`);
  }
}

const usersService = new UserService({
  usersRepository,
  mailer,
  logger
});
```

지금까지는 class에 관해서만 살펴봤다. 이제 function에 관해서 파악해보자. 사실 function이라고 해서 특별한 것은 없다. `parameter`로 의존성을 주입한다. 그게 끝이다. JavaScript의 `closure`덕분에 function내에서 의존성에 편히 접근할 수 있다.

```js
type UsersDependencies = {
  usersRepository: UsersRepository
  mailer: Mailer
  logger: Logger
};

export const usersService = (dependencies: UsersDependencies) => {
  const findAll = () => dependencies.usersRepository.findAll();
  const addUser = user => {
    await dependencies.usersRepository.addUser(user)
    dependencies.logger.info(`User created: ${user}`)
    await dependencies.mailer.sendConfirmationLink(user)
    dependencies.logger.info(`Confirmation link sent: ${user}`)
  };
  
  return {
    findAll,
    addUser
  };
}

const service = usersService({
  usersRepository,
  mailer,
  logger
});
```
위 코드를 보면 알듯이. 의존성 주입은 Class만을 위한 것이 아니다. !

## Orchestration and tooling
의존성 주입(DI)의 눈에 띄는 단점은 이용하려는 의존성들을 모두 미리 세팅해야 한다는 것이다. 아래 코드의 예시를 봐보자. 만약에 `users` __service__를 만들고 싶다면, 미리 `repository`도 만들어놔야 하고, `mailer`도 디테일하게 세팅해둬야하고 `logger`도 가져오든 세팅하든 다 해놔야한다. 이용할 모든 의존성을 구조화해둬야 한다는 말이다.
```js
const UsersRepository = require('./users-repository');
const Mailer = require('./mailer');
const Logger = require('./logger');
const UsersService = require('./users-service');
const InMemoryDataSource = require('./users-repository/data-source/in-memory');

const logger = new Logger({
  level: process.env || 'dev'
});
const dataSource = new InMemoryDataSource();
const mailer = new Mailer({
  templates: '/emails',
  logger
});
const usersRepository = new UsersRepository({
  logger,
  dataSource
});

const usersService = new UsersService({
  usersRepository,
  mailer,
  logger
});

module.exports = {
  usersService
}
```

usersService를 생성하기전에 미리 의존성들을 준비해둬야 한다. 의존성들을 모아두는 곳을 흔히 `container`라고 부른다. 그 container를 세팅하는게 여간 귀찮은게 아니다. 허나 신경쓰지 않아도 된다. 알아서 의존성을 찾고 가져와주는 라이브러리들이 여럿 존재한다.
대표적으로 `Awilix`와 `Inversify` 그리고 `TypeDI`.

`Awilix`와 `TypeDI`는 다소 비슷하고 JavaScript와 TypeScript에서 모두 작동한다. 
반면에 `Inversify`는 TypeScript에서만 작동한다.

(참고: TSH에서는 `Awilix`를 선호하지만 `TypeDI`도 함께 사용하고 있다.)

`Awilix`를 사용하면 자체적으로 종속성을 해결하는 특수 컨테이너를 만들 수 있다.
우리가 해야할 일은 Type 설정만 제공해주면 된다.(예를 들면 UsersService가 class이다 라는 것만 전달하면 됨.)
```js
const UsersRepository = require('./users-repository');
const Mailer = require('./mailer');
const Logger = require('./logger');
const UsersService = require('./users-service');
const InMemoryDataSource = require('./users-repository/data-source/in-memory');
const { createContainer, asClass } = require('awilix');

const createAppContainer = async () => {
  const container = createContainer();
  
  container.register({
    logger: asClass(Logger).inject(
      () => ({ level: process.env || 'dev'})
    ),
    dataSource: asClass(InMemoryDataSource),
    mailer: asClass(Mailer).inject(() => ({ templates: '/emails'})),
    usersRepository: asClass(UsersRepository),
    usersService: asClass(UsersService)
  });
  
  return container
);
 
(async () => {
  const container = await createAppContaier();
  
  const usersService = container.resolve('usersService');
})()
```
__Awilix container에서 `resolve` method를 실행시키면 모든 생성자/함수의 인자들을 거쳐가면서 container에 설정되어있는 의존성들을 찾아낸다.__

이렇게 하면 우리는 일일이 의존성을 만들어놓을 필요가 없어진다. 알아서 다 찾아서 설정해주니깐. ! 우리가 할일은 `resolve` 메서드만 실행시키면 된다. 개인적으로 당신이`Awilix`와 `TypeDI`를 모두 확인해보길 권장한다.

## Friends or Foes?
의존성 주입은 너무나 많은 유연성을 제공해준다. 테스트를 쉽게해주는 것뿐만아니라 여러가지 부분에서말이다. 또한 모듈을 완전히 독립적으로 만들수 있게 해준다.

여러 의존성 컨테이너 도구들을 사용함으로써 여러 모듈들을 쉽게 만들 수 있다. TSH에서는 모든 Node.js프로젝트에 의존성 주입을 무조건 사용하고 있다. 의존성주입이 없는 프로젝트는 상상도 할수 없다.!

출처: https://tsh.io/blog/dependency-injection-in-node-js/