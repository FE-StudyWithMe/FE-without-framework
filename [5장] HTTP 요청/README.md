```
❗ 프레임워크 없는 방식으로 HTTP 클라이언트를 구축하는 방법을 알아보자.
```

## 1. 간단한 역사: AJAX의 탄생

- 1999년 이전에는 서버에서 데이터를 가져올 필요가 있는 경우 전체 페이지를 다시 로드해야 했다.
- 1999년에 아웃룩, 지메일과 구글 지도 같은 애플리케이션들은 페이지를 완전히 다시 로드하지 않고 최초 페이지 로드 후 필요한 데이터만 서버에서 로드하는 새로운 기술을 사용하기 시작했다.
    - 제시 제임스 가레트(Jesse James Garrett)가 이 기술을 ***비동기 자바스크립트 및 XML(Asynchronous JavaScript and XML)***의 약어인 AJAX로 명명했다.
- AJAX 애플리케이션의 핵심은 ***XMLHttpRequest*** 객체다.
    - 이 객체를 사용하면 HTTP 요청으로 서버에서 데이터를 가져올 수 있다.
    - W3C는 2006년 이 객체의 표준 규격 초안을 작성했다.
- AJAX가 등장했을 때 웹 애플리케이션은 서버 데이터를 XML 형식으로 수신했으나, 지금은 JSON(자바스크립트 애플리케이션용) 형식이 사용된다.
    
    > PDF이신 분들 여기 사진 첨부 하나 해주실 수 있을까요.. (아날로그는 웁니다. 🥲)
    > 


## 2. todo 리스트 REST 서버

```
❗ 개발할 클라이언트 테스트를 위해 데이터를 가져올 간단한 서버를 만들어본다.
아래는 Express로 Node.js용 REST 서버를 구축한 것으로, 실제 데이터베이스 대신 임시 배열을 사용해 todo 리스트와 관련 데이터를 저장한다.
```

[전체 코드 링크](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/server.js)

```js
// 가짜 ID를 생성하고자 사용
// 범용 고유 식별자(UUID, Universally Unique Identifiers)를 생성할 수 있는 라이브러리
const uuidv4 = require('uuid/v4')
```

```js
// 임시 배열에 todo 리스트와 관련된 데이터 저장
let todos = []

// todos를 사용해 CRUD 동작 수행
app.get('/api/todos', (req, res) => { ... })

app.post('/api/todos', (req, res) => { ... })

app.patch('/api/todos/:id', (req, res) => { ... })

app.delete('/api/todos/:id', (req, res) => { ... })
```

### 2-1. REST

- REST는 REpresentational State Transfer의 약자로 웹 서비스를 디자인하고 개발하는 방법이다.

- 모든 REST API의 추상화는 리소스에 있다.
    - 도메인을 리소스로 분할해야 하며 각 리소스는 특정 URI(Uniform Resource Identifier)로 접근해 읽거나 조작할 수 있어야 한다.
    
    > **💡 리소스 중심의 추상화
    :** 리소스는 일반적으로 도메인 모델에서 다루는 실체(entities)나 주요 데이터를 의미한다. 예를 들어, SNS 애플리케이션에서 사용자는 User, 게시물은 Post와 같이 도메인 내의 주요 개념을 리소스라고 할 수 있다.
    > 
    
    > **💡 리소스는 URI로 접근**
    : 각 리소스는 특정한 URI(Uniform Resource Identifier)를 통해 접근할 수 있어야 한다. URI는 쉽게 말해 리소스의 "주소" 역할을 하는데, 이 주소를 통해 API 사용자는 리소스에 접근하거나 조작할 수 있다.
    > 
    > - 사용자의 목록을 가져오기 위한 URI: `/users`
    > - 특정 사용자의 정보를 가져오기 위한 URI: `/users/{userId}`

- HTTP 메서드를 통해 리소스를 조작할 수 있다.

    | 동작 | URI | HTTP 메서드 |
    | --- | --- | --- |
    | 모든 사용자의 데이터 읽기 | `/users` | GET |
    | ID가 1인 사용자의 데이터 읽기 | `/users/1` | GET |
    | 새로운 사용자 생성 | `/users` | POST |
    | ID가 1인 사용자 데이터 교체 | `/users/1` | PUT |
    | ID가 1인 사용자 데이터 업데이트 | `/users/1` | PATCH |
    | ID가 1인 사용자 데이터 삭제 | `/users/1` | DELETE |


## 3. 세 가지 HTTP 클라이언트 비교

```
❗ XMLHttpRequest, Fetch, axios의 세 가지 기술을 사용해 세 가지 HTTP 클라이언트 버전을 작성해보고, 각 클라이언트의 강점과 약점을 분석해보자.
```

### 3-1. 기본 구조

[메인 컨트롤러 전체 코드](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/public/00/index.js)

[todos 모델 객체 코드](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/public/00/todos.js)

```js
import todos from './todos.js'

// ... 중략 ...

const onListClick = async () => {
  const result = await todos.list() // ✅ 중요 포인트
  printResult('list todos', result)
}

const onAddClick = async () => {
  const result = await todos.create('A simple todo Element') // ✅ 중요 포인트
  printResult('add todo', result)
}

// ... 생략 ...
```

**중요 포인트**

- 이 컨트롤러에서는 HTTP 클라이언트를 직접 사용하는 대신 HTTP 요청을 todos 모델 객체에 래핑했다.
- 이런 캡슐화가 유용한 이유
    1. 테스트 가능성: todos 객체를 정적 데이터 세트를 반환하는 모의(mock)로 바꿀 수 있다. → 컨트롤러 독립적 테스트 가능
    2. 가독성: 모델 객체는 코드는 좀 더 명확하게 만든다.

### 3-2. XMLHttpRequest

[XMLHttpRequest를 사용하는 HTTP 클라이언트 코드](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/public/00/http.js)

```js
const request = params => {
  return new Promise((resolve, reject) => { // ✅ 중요 포인트 1
    const xhr = new XMLHttpRequest()

    const {
      method = 'GET',
      url,
      headers = {},
      body
    } = params

    xhr.open(method, url) // ✅ 중요 포인트 2

    setHeaders(xhr, headers)

    xhr.send(JSON.stringify(body)) // ✅ 중요 포인트 2

    xhr.onerror = () => {
      reject(new Error('HTTP Error')) // ✅ 중요 포인트 2
    }

    xhr.ontimeout = () => {
      reject(new Error('Timeout Error')) // ✅ 중요 포인트 2
    }

    xhr.onload = () => resolve(parseResponse(xhr)) // ✅ 중요 포인트 2
  })
}
```

**중요 포인트**

1. HTTP 클라이언트의 공개 API는 프라미스(Promise)를 기반으로 한다.
    - 따라서 request 메서드는 표준 XMLHttpRequest 요청을 새로운 Promise 객체로 묶는다.
    - 공개 메서드 get, post, put, patch, delete는 코드를 더 읽기 쉽게 해주는 request 메서드의 래퍼다.
2. XMLHttpRequest는 콜백을 기반으로 한다.
    - 완료된 요청에 대한 처리, 오류로 끝나는 HTTP에 대한 처리, 타임아웃된 요청에 대한 처리는 모두 콜백으로 한다.
    - 디폴트로 타임아웃은 없지만 xhr 객체의 timeout 속성을 수정하면 타임아웃을 변경할 수 있다.

**XMLHttpRequest를 사용한 HTTP 요청의 흐름**

1. 새로운 XMLHttpRequest 객체 생성 (`new XMLHttpRequest()`)
2. 특정 URL로 요청 초기화 (`xhr.open(method, url)`)
3. 요청(헤더 설정, 타임아웃 등)을 구성
4. 요청 전송 (`xhr.send(JSON.stringify(body))`)
5. 요청이 끝날 때까지 대기
    1. 요청이 성공적으로 끝나면 `onload` 콜백 호출
    2. 요청이 오류로 끝나면 `onerror` 콜백 호출
    3. 요청이 타임아웃으로 끝나면 `ontimeout` 콜백 호출

### 3-3. Fetch

- Fetch는 원격 리소스에 접근하고자 만들어진 새로운 API로 Request나 Response 같은 많은 네트워크 객체에 대한 표준 정의를 제공하는 것이 목적이다.
- 이런 방식으로 이 객체는 [ServiceWorker](https://developer.mozilla.org/ko/docs/Web/API/Service_Worker_API)와 [Cache](https://developer.mozilla.org/ko/docs/Web/API/Cache) 같은 다른 API와 상호 운용할 수 있다.
- 요청을 생성하기 위해서는 `window.fetch` 메서드를 사용해야 한다.

[Fetch API를 기반으로 하는 HTTP 클라이언트 코드](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/public/01/http.js)

```jsx
const parseResponse = async response => {
  const { status } = response
  let data
  if (status !== 204) {
    data = await response.json() // ✅ 중요 포인트
  }

  return {
    status,
    data
  }
}

const request = async params => {
  const {
    method = 'GET',
    url,
    headers = {},
    body
  } = params

  const config = {
    method,
    headers: new window.Headers(headers)
  }

  if (body) {
    config.body = JSON.stringify(body)
  }

  const response = await window.fetch(url, config) // ✅ 중요 포인트

  return parseResponse(response)
}
```

**중요 포인트**

- `window.fetch`가 Promise 객체를 반환하기 때문에 훨씬 더 읽기 쉽다.
    - 따라서 전통적인 콜백 기반의 XMLHttpRequest 접근 방식을 최신의 프라미스 기반으로 변환하기 위한 보일러플레이트(boilerplate) 코드가 필요하지 않다.
- `window.fetch`에서 반환된 프라미스는 Response 객체를 해결(resolve)한다.
    - 이 객체를 사용해 서버가 보낸 응답 본문을 추출할 수 있다.
    - 수신된 데이터 형식에 따라 `text()`, `blob()`, `json()` 같은 메서드를 사용할 수 있고, `Content-Type` 헤더와 함께 적절한 메서드를 사용해야 한다.

### 3-4. Axios

- axios와 다른 방식과의 가장 큰 차이는 브라우저와 Node.js에서 바로 사용할 수 있다는 것이다.
- axios의 API는 프라미스를 기반으로 하고 있어 Fetch API와 매우 유사하다.

[axios 기반의 HTTP 클라이언트 코드](https://github.com/Apress/frameworkless-front-end-development/blob/master/Chapter05/public/02/http.js)

```jsx
const request = async params => {
  const {
    method = 'GET',
    url,
    headers = {},
    body
  } = params

  const config = {
    url,
    method,
    headers,
    data: body
  }

  return axios(config) // ✅ 중요 포인트
}
```

**중요 포인트**

- request 메서드는 axios 서명을 공공 계약과 일치하도록 매개변수를 재배열한다.
    
    > **💡 이해를 위한 설명**
    > 
    > - `request`의 매개변수 형식을 `axios`의 호출 방식에 맞추도록 정리했다는 의미로 이해할 수 있다.
    
    > **💡 request 메서드 코드 분석**
    > 
    > - `axios`의 기본적인 호출 방식은 `axios(config)` 형태로, `config` 객체에 요청의 메서드, URL, 헤더, 데이터 등을 담아서 전달한다.
    > - `request` 함수에서는 `params`라는 객체를 받아 `method`, `url`, `headers`, `body`를 분리한 다음, 이들을 `config` 객체로 구성하여 `axios(config)` 호출을 수행한다.
    
    > **💡 매개변수 재배열의 목적**
    > 
    > - `request` 함수가 `axios`의 `config` 객체 구조와 **일치하도록 매개변수들을 재구성**해 사용자가 쉽게 요청을 보낼 수 있다.
    > - **유연한 사용성**: `request` 함수가 `axios`의 호출 방식과 유사한 구조를 가져서, `axios` 호출 시 사용하는 옵션들을 그대로 활용할 수 있다.
    > - **코드 일관성**: `config` 객체가 `axios`의 기대 구조에 맞춰 정리되어 있어, `axios`의 업데이트나 다른 HTTP 클라이언트로 전환할 때도 큰 수정 없이 호환이 가능하다.
    > - **캡슐화**: `request` 함수는 직접 `axios`를 호출하는 대신 공통적인 형식을 제공하여, `axios`의 서명이 외부 API와의 “공공 계약”처럼 작용하게 만들었다.

### 3-5. 아키텍처 검토

- 세 버전의 클라이언트는 동일한 공용 API를 가진다.
- 이런 아키텍처 특성으로 인해 최소한의 노력으로 HTTP 요청에 사용되는 라이브러리를 변경할 수 있다.
    - 즉, XMLHttpRequest에서 Fetch나 axios로 변경하는 데에 큰 노력이 필요하지 않다.
    - (UML 다이어그램 사진.. 첨부 부탁합니다..)
- 만약 HTTP 클라이언트를 사용하지 않고 직접 axios를 사용하는 경우, Fetch API로 구현을 변경한다면 매우 비싸고 지루한 작업이 될 것이다.
- 모델 객체에서 axios를 사용한다는 것은 인터페이스(HTTP 클라이언트)가 아닌 구현(라이브러리)을 프로그래밍하는 것을 의미한다.

## 4. 적합한 HTTP API를 선택하는 방법

```
❗ XMLHttpRequest, Fetch API, axios의 특성을 다른 관점에서 알아보자.
```

### 4-1. 호환성

- 비즈니스 상 인터넷 익스플로러의 지원이 중요하다면 Fetch API는 최신 브라우저에서만 동작하기 때문에 axios나 XMLHttpRequest를 사용해야 한다.
    - Fetch API의 폴리필을 사용할 수도 있지만 인터넷 익스플로러의 지원을 조만간 중단할 계획을 가지고 있는 경우에만 이 방법을 사용하는 것이 좋다.
- axios는 인터넷 익스플로러 11을 지원하지만 이전 버전은 지원하지 않아 XMLHttpRequest를 사용해야 한다.

### 4-2. 휴대성

- Fetch API, XMLHttpRequest는 모두 브라우저에서만 동작한다.
- Node.js나 리액트 네이티브 같은 다른 자바스크립트 환경에서 코드를 실행해야 한다면 axios가 좋은 솔루션이다.

### 4-3. 발전성

- Fetch API는 Request나 Response 같은 네트워크 관련 객체의 표준 정의를 제공한다.
    - 따라서 Serviceworker API나 Cache API와 잘맞는다.

### 4-4. 보안

- axios에는 교차 사이트 요청(cross-site request) 위조나 XSRF에 대한 보호 시스템이 내장돼 있다.
- XMLHttpRequest나 Fetch API를 사용하면서 이런 종류의 보안 시스템을 구현해야 한다면 이 주제에 대한 [axios 단위 테스트](https://github.com/axios/axios/blob/main/test/specs/xsrf.spec.js)를 살펴보는 것을 추천한다.

### 4-5. 학습 곡선

- 콜백 작업에 익숙하지 않은 초보자에게는 axios나 Fetch API가 좀 더 이해하기 쉬울 것이다.