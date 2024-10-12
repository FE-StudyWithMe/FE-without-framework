# [3장] DOM 이벤트 관리

## 💡 YAGNI 원칙
- You Aren't Gonna Need It; 정말 필요하다고 간주할 때 까지 기능을 추가하지 마라
- 익스트림 프로그래밍 원칙 중 하나
  - [익스트림 프로그래밍](https://itwiki.kr/w/%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%A6%BC_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D) : 애자일 방법론중 하나로 빠른 개발 속도를 유지하며 고객이 원하는 요구들을 지속적으로 피드백하는 방법
- 가장 중요한 기능에 초점을 맞춰 개발 → 새로운 요구가 생기면 아키텍처를 지속적으로 발전

## 💡 DOM 이벤트
- 웹 애플리케이션에서 발생하는 동작, 이에 따라 사용자는 반응할 수 있음
<img width="600" alt="image" src="https://github.com/user-attachments/assets/8d8a8157-f1ba-4c5c-9713-c568fb607506">
- [MDN에 정의된 이벤트](https://developer.mozilla.org/ko/docs/Web/API/Event) 외에 시스템 자체에서도 이벤트 생성 가능

## 💡이벤트 처리
### 1. 속성에 핸들러 연결
- `on` 속성을 사용하여 핸들러를 이벤트에 연결
- ex) `onclick`, `ondblclick`, `onmouseover`, `onblur`, `onfocus`
  ```js
  const button = document.querySelector('button');
  button.onclick = () => {
  console.log('Click managed using onclick property');
  };
  ```
- 빠르고 지저분한 해결책
  - 속성을 사용하면 한 번에 하나의 핸들러에만 연결 → 동일한 요소에 여러 개의 이벤트 등록 불가능
  - 위의 예시에서 코드가 onclick 핸들러를 덮어 쓰면 원래 핸들러는 영원이 손실

### 2. addEventListener로 핸들러 연결
- EventTarget 인터페이스
  - 이벤트를 수신하고 처리하는 모든 DOM 객체에 공통적으로 적용
  - 객체가 이벤트를 수신하고 지정된 이벤트 핸들러를 실행
- addEventListener
  - EventTarget 인터페이스에 정의된 메서드
  - 특정 이벤트가 발생했을 때 실행될 함수를 DOM 노드에 등록
  - 동일한 이벤트에 대해 여러개의 핸들러 등록 가능
  - 이벤트 전파 단계 (캡쳐링/버블링) 제어 가능
- removeEventListener
  - DOM에 요소가 더 이상 존재하지 않으면 메모리 누수를 방지하고자 이벤트 리스너를 삭제
  - 이벤트 핸들러에 대한 참조를 유지해야 함
  ```html
  <button>button</button>
  <script>
    const button = document.querySelector('button');
    const firstHandler = () => {
      console.log('First handler');
    };
    const secondHandler = () => {
      console.log('Second handler');
    };
    button.addEventListener('click', firstHandler);
    button.addEventListener('click', secondHandler);
    
    window.setTimeout(() => {
      button.removeEventListener('click', secondHandler);
      console.log('Remove event handlers')
    }, 1000);
  </script>
  ```
  <img width="280" alt="image" src="https://github.com/user-attachments/assets/2e1849a6-cfcc-4d5b-855b-48d19cc62fd8">

### 3. 이벤트 객체
- 이벤트에는 포인터 좌표, 이벤트 타입, 이벤트를 트리거 한 요소 등 다양한 정보가 들어있음 → Event 인터페이스
- Event 인터페이스
  - 모든 웹 이벤트의 기본 인터페이스
- 구체적인 Event 인터페이스
  - Event 인터페이스를 확장하여 특정 이벤트 유형에 추가적인 기능을 제공
  - ex) MouseEvent, KeyboardEvent, TouchEvent
  ```html
  <button>button</button>
  <script>
    const button = document.querySelector('button');
    button.addEventListener('click', (e) => {
      console.log('event', e)
    });
  </script>  
  ```
  <img width="600" alt="image" src="https://github.com/user-attachments/assets/405ff0e8-346d-4b41-a607-8389f61f2056"> <br/>
  <img width="500" alt="image" src="https://github.com/user-attachments/assets/fb8ab4b3-27bb-4f29-9bed-ec055fea2a26"> <br/>
  
  - 위 예시의 경우, 기본 Event 인터페이스를 확장하여 마우스 이벤트에 특화된 추가적인 속성과 메서드를 포함 (ex. clientX, clientY, screenX, screenY, button 등)

### 4. DOM 이벤트 라이프 사이클
- 캡쳐 → 목표 → 버블 단계로 구성 <br/>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/d9ce370e-ab68-4e96-8af6-eda2623b253e">

  - 캡쳐 단계
    - 이벤트가 최상위 요소(window 또는 document)에서 자식 요소(목표 요소)로 이동하며 전파되는 과정
    - 부모 요소들이 이벤트 처리 가능
    - 이벤트가 최상위 요소에서 먼저 처리되도록 하는 경우 사용
  - 목표 단계
    - 이벤트가 실제로 발생한 요소(타겟 요소)에서 이벤트 핸들러 실행
  - 버블 단계
    - 이벤트가 발생한 요소에서 다시 최상위 요소로 올라가며 전파되는 과정
    - 부모 요소들이 이벤트 처리 가능
    - 이벤트가 상위 요소들에게도 영향을 미치도록 하려는 경우 사용
    - Event 인터페이스의 stopPropagation 메서드를 통해 버블 체인 중지 가능
- addEventListener의 세 번째 매개변수인 useCapture 값으로 Capturing/Bubbling 설정 가능
- 캡쳐, 버블 단계 매커니즘 예시
    ```html
    <body>
      <div>
        This is a container
        <button>click</div>
      </div>
    </body>
    ```
    <img width="188" alt="image" src="https://github.com/user-attachments/assets/00f8d98f-e404-4778-9de3-015cf5ba62ac">

  - 캡쳐
    ```html
    <script>
      const div = document.querySelector('div');
      const button = document.querySelector('button');
      
      div.addEventListener('click', () => {
        console.log('div click')
      }, true)
      button.addEventListener('click', () => {
        console.log('button click')
      }, true)
    </script>
    ```
    <img width="136" alt="image" src="https://github.com/user-attachments/assets/e45a2ffe-d620-4ff9-a27b-ef7d7f495212">

  - 버블
    ```html
    <script>
      const div = document.querySelector('div');
      const button = document.querySelector('button');
      
      div.addEventListener('click', () => {
        console.log('div click')
      }, false)
      button.addEventListener('click', () => {
        console.log('button click')
      }, false)
    </script>
    ```
    <img width="143" alt="image" src="https://github.com/user-attachments/assets/b7354767-73e9-4f7d-afd7-fdbd4dc6e2cf">

### 5. 사용자 정의 이벤트 사용
- CustomEvent 생성자 함수 사용하여 사용자 정의 이벤트 생성
    ```js
    const EVENT_NAME = 'FiveCharInputValue';
    const input = document.querySelector('input');
    
    input.addEventListener('input', () => {
        const { length } = input.value;
        console.log('input length:', length);
        if (length === 5) {
            const time = new Date().getTime();
            const event = new CustomEvent(EVENT_NAME, { detail: { time } });
            input.dispatchEvent(event);
        }
    });
    
    input.addEventListener(EVENT_NAME, (e) => {
        console.log('handling custom event...', e.detail);
    });
    ```
    <img width="163" alt="image" src="https://github.com/user-attachments/assets/9a468266-9815-482f-a509-e4404808193a"> <br/>
    <img width="350" alt="image" src="https://github.com/user-attachments/assets/d1e529af-80bc-46db-8e46-f215577ce7d3">

## 💡TodoMVC에 이벤트 추가
### 1. 렌더링 엔진 리뷰
- 기존 코드는 일부가 DOM 요소 대신 문자열로 동작 → 이벤트 핸들러 추가 불가능 (∵ addEventListener를 호출하려면 DOM 노드 필요)
- DOM 요소 생성
  - document.createElement API
    - 비어있는 새 DOM 노드 생성
    - 코드를 읽고 유지하기 어려움
  - index,html 파일의 `<template>` 태그 안에 todo 요소의 마크업 유지
    - [template 요소](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template)
      - 재사용 가능한 HTML 구조를 정의하고 JavaScript를 통해 동적으로 DOM에 추가할 수 있도록 하는 HTML5의 요소
      - `<template>` 내부에 원하는 HTML 구조를 정의하고 JavaScript에서 이를 클론하여 사용
      <img width="700" alt="image" src="https://github.com/user-attachments/assets/e5205524-4f27-4eb4-b17d-207566aa7d33">

### 2. 기본 이벤트 처리 아키텍처
- 이벤트 핸들러를 애플리케이션에 연결
- 새로운 상태마다 새로운 DOM 트리를 생성해 가상 DOM 알고리즘 적용
  - 예시에서는 '루프'에 이벤트 핸들러 삽입
  - 모든 이벤트 다음에 상태 조작 후 새로운 상태로 메인 랜더링 함수 호출

    <img width="200" alt="image" src="https://github.com/user-attachments/assets/2fe16464-c31a-4c69-be6e-95298b7ec014">

  - 유스케이스 단계
    - 초기 상태: 비어있는 todo 리스트
    - 렌더링: 사용자에게 비어있는 리스트를 표시
    - 이벤트: 사용자가 '더미 항목'이라는 새 항목을 생성
    - 새로운 상태: 하나의 항목을 가진 todo 리스트
    - 렌더링: 사용자에게 하나의 항목을 가진 리스트를 표시
    - 이벤트: 사용자가 항목을 삭제
    - 새로운 상태: 비어있는 todo 리스트
    - 렌더링: 사용자에게 비어있는 리스트를 표시
#### a. 이벤트와 관련 된 상태 수정 정의
- renderRoot 함수의 역할과 이벤트 전달
  ```js
  const newMain = registry.renderRoot(
    main,
    state,
    events)
  ```
  - 렌더링 엔진의 진입점
  - 현재 상태와 이벤트 핸들러를 받아 새로운 DOM 구조를 생성
  - 컴포넌트 레지스트리에 등록된 모든 컴포넌트를 조합하여 애플리케이션의 전체 UI를 구성
  - events 객체를 세 번째 매개변수로 전달
    - renderRoot 함수는 모든 컴포넌트가 중앙에서 관리되는 이벤트 핸들러에 접근 가능
    - 모든 구성 요소가 동일한 이벤트 핸들러에 접근 가능
    - 일관된 이벤트 처리 가능
- 이벤트 핸들러의 역할과 이벤트 레지스트리
  ```js
  const events = {
    deleteItem: (index) => {
      state.todos.splice(index, 1)
      render()
    },
    addItem: text => {
      state.todos.push({
        text,
        completed: false
      })
      render()
    }
  }
  ```
  - events 객체 내의 각 함수는 사용자 인터랙션에 따라 상태 수정
  - 수정 후 render() 함수를 호출하여 UI를 갱신
  - addItem : 새로운 Todo를 추가
  - deleteItem : 특정 Todo를 삭제
#### b. input 핸들러에 이벤트 핸들러 연결
  ```js
  const addEvents = (targetElement, events) => {
    targetElement
      .querySelector('.new-todo')
      .addEventListener('keypress', e => {
    if (e.key === 'Enter') {
        events.addItem(e.target.value)
        e.target.value = ''
      }
    })
  }
  ```
  - 새로 생성된 DOM 요소 내의 .new-todo 입력 필드에 keypress 이벤트 리스너 추가
  - 사용자가 엔터 키를 누르면 addItem 함수를 호출하여 새로운 Todo 항목을 추가하고 입력 필드를 초기화
#### c. 개별 핸들러 작성
  ```js
  todos
    .map((todo, index) => getTodoElement(todo, index, events))
    .forEach(element => {
      newTodoList.appendChild(element)
    })
  ```
  - getTodoElement 함수에 events 객체를 개별적으로 전달하여 각 컴포넌트가 개별 이벤트 핸들러에 접근하고 사용

### 3. 이벤트 위임
- 부모 요소에 이벤트 리스너를 등록하고, 자식 요소들에서 발생하는 이벤트를 부모 요소에서 처리하는 패턴
  - 책에서의 이벤트 위임 과정
    - 부모 요소에 단일 이벤트 리스너 등록
      ```js
      newTodoList.addEventListener('click', e => {
        if (e.target.matches('button.destroy')) {
          deleteItem(e.target.dataset.index)
        }
      })
      ```
      - newTodoList
        - targetElement가 복제된 리스트 요소
        - 이는 전체 Todo 목록을 감싸는 부모 요소로 가정
      - addEventListener
        - newTodoList에 클릭 이벤트 리스너를 등록
        - 모든 자식 요소(개별 Todo 항목들과 그 안의 버튼들)에서 발생하는 클릭 이벤트는 이 부모 리스너에서 포착
    - 이벤트 타겟 검증 및 처리
      ```js
      if (e.target.matches('button.destroy')) {
        deleteItem(e.target.dataset.index)
      }
      ```
      - e.target
        - 실제로 클릭된 요소
      - e.target.matches('button.destroy')
        - 클릭된 요소가 button 태그이며, 클래스가 destroy인지를 확인 (삭제 버튼을 식별하기 위해 사용)
      - deleteItem(e.target.dataset.index)
        - 삭제 버튼의 data-index 속성에 저장된 인덱스를 통해 해당 Todo 항목을 삭제
    - Todo 항목 생성 시 data-index 설정
      ```js
      element
        .querySelector('button.destroy')
        .dataset
        .index = index
      ```
      - data-index
        - 각 삭제 버튼에 고유한 인덱스를 할당
        - 어떤 Todo 항목이 삭제될지를 식별
        - 클릭된 버튼이 어느 Todo 항목을 참조하는지 쉽게 알 수 있어, 이벤트 핸들러에서 해당 항목을 정확히 삭제 가능
- [matches API](https://developer.mozilla.org/en-US/docs/Web/API/Element/matches)
  - 요소가 **실제** 이벤트 대상인지 확인
  - 대규모 프로젝트에서 해당 방식을 사용하면 웹 페이지 본문에서 하나의 이벤트 핸들러만 사용 가능
  - 주의 🚨 실제로 필요할 때 까지는 코드에 이벤트 위임 최적화 추가하지 말 것! YAGNI 원칙을 기억하고 반드시 필요한 경우에만 [gator.js](https://craig.is/riding/gators)같은 이벤트 위임 라이브러리를 기존 프로젝트에 추가하기
