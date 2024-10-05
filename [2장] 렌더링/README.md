## [2장] 랜더링

- ### 💡 들어가기에 앞서
	- 해당 챕터는 랜더링 엔진을 코드로 만드는 과정을 예시로 설명한 챕터입니다. 반드시 책과 예시코드를 하나하나 확인해 가며 읽기를 권장합니다 :)
- ### 💡 DOM?
  - DOM(문서 객체 모델) 은 HTML요소로 정의된 트리를 조작할 수 있는 API
- ### 💡 랜더링 엔진 성능 모니터링
  - POINT 🥸 : 렌더링 엔진 성능 모니터링시 FPS (frame per second)를 중요하게 보는 이유는 렌더링 파이프라인의 모든 단계 (JavaScript 실행, 스타일 계산, 레이아웃, 페인트 등)가 FPS에 영향을 주기 때문
  - 방법 1 : 크롬 개발자 도구 사용
  - <img width="598" alt="스크린샷 2024-10-05 오전 3 10 03" src="https://github.com/user-attachments/assets/0c193747-48d1-4dd4-96ab-95e9bed7956f">
  - <img width="382" alt="스크린샷 2024-10-05 오전 3 10 16" src="https://github.com/user-attachments/assets/1b013f94-c190-4cac-a7a6-31a9786bc110">
  - 방법 2 : stats.js 라이브러리 사용
  - 방법 3 : `requestAnimationFrame`을 사용한 사용자 정의 위젯 생성
   
   
- ## 💡 랜더링 함수 만들기 ([todoMVC](https://todomvc.com/))
- <img width="601" alt="스크린샷 2024-10-05 오전 2 26 02" src="https://github.com/user-attachments/assets/0eb9b9a9-49d7-477f-b690-aab723218a6c">
- ### 단계 1 : 순수함수를 사용해 요소를 DOM에 랜더링하기
  - ```js
    //index.html
    <body>
    <section class="todoapp">
        <header class="header">
            <h1>todos</h1>
            <input class="new-todo" placeholder="What needs to be done?" autofocus>
        </header>
        <section class="main">
            <input id="toggle-all" class="toggle-all" type="checkbox">
            <label for="toggle-all">Mark all as complete</label>
            <ul class="todo-list">
            </ul>
        </section>
        <footer class="footer">
            <span class="todo-count">1 Item Left</span>
            <ul class="filters">
                <li>
                    <a href="#/">All</a>
                </li>
                <li>
                    <a href="#/active">Active</a>
                </li>
                <li>
                    <a href="#/completed">Completed</a>
                </li>
            </ul>
            <button class="clear-completed">Clear completed</button>
        </footer>
    </section>
    <footer class="info">
        <p>Double-click to edit a todo</p>
        <p>Created by <a href="http://twitter.com/thestrazz86">Francesco Strazzullo</a></p>
        <p>Thanks to <a href="http://todomvc.com">TodoMVC</a></p>
    </footer>
    <script type="module" src="index.js"></script>
	</body>
    ```
-  위 앱을 동적으로 만들기 위해서는 todo 리스트의 데이터를 가져와 해당 앱에 업데이트 시켜줘야한다.
  - 필터링된 todo리스트를 가진 ul
    `<ul class="todo-list">`
  - 완료되지않은 todo수를 가진 span
    `<span class="todo-count">1 Item Left</span>`
  - selected 클래스를 오른쪽에 추가한 필터 유형을 가진 링크
    `<ul class="filters">`
    ```js
    //view.js
	const getTodoElement = todo => {
	  const {
	    text,
	    completed
	  } = todo
	
	  return `
	  <li ${completed ? 'class="completed"' : ''}>
	    <div class="view">
	      <input 
	        ${completed ? 'checked' : ''}
	        class="toggle" 
	        type="checkbox">
	      <label>${text}</label>
	      <button class="destroy"></button>
	    </div>
	    <input class="edit" value="${text}">
	  </li>`
	}

	const getTodoCount = todos => {
	  const notCompleted = todos
	    .filter(todo => !todo.completed)
	
	  const { length } = notCompleted
	  if (length === 1) {
	    return '1 Item left'
	  }
	
	  return `${length} Items left`
	}
	
	export default (targetElement, state) => {
	  const {
	    currentFilter,
	    todos
	  } = state
	
	  const element = targetElement.cloneNode(true)
	
	  const list = element.querySelector('.todo-list')
	  const counter = element.querySelector('.todo-count')
	  const filters = element.querySelector('.filters')
	
	  list.innerHTML = todos.map(getTodoElement).join('')
	  counter.textContent = getTodoCount(todos)
	
	  Array
	    .from(filters.querySelectorAll('li a'))
	    .forEach(a => {
	      if (a.textContent === currentFilter) {
	        a.classList.add('selected')
	      } else {
	        a.classList.remove('selected')
	      }
	    })
	
	  return element
	}
	//index.js
	import getTodos from './getTodos.js'
	import view from './view.js'
	
	const state = {
	  todos: getTodos(),
	  currentFilter: 'All'
	}
	
	const main = document.querySelector('.todoapp')
	
	window.requestAnimationFrame(() => {
	  const newMain = view(main, state)
	  main.replaceWith(newMain)
	})
    ```
  - 가상 돔을 만들어 해당 가상 돔을 requestAnimationFrame으로 실제 돔에 반영하는 방법<br/>
  - `requestAnimationFrame`은 모니터의 프레임의 주기(FPS)에 맞추어 일정한 시간 간격으로 랜더링 되게 해준다.
    - [requestAnimationFrame 참고 자료](https://inpa.tistory.com/entry/%F0%9F%8C%90-requestAnimationFrame-%EA%B0%80%EC%9D%B4%EB%93%9C#requestanimationframe)
    - [MDN 공식 문서](https://developer.mozilla.org/ko/docs/Web/API/Window/requestAnimationFrame)
  - 💣 여러가지 돔을 조작할 수 있는 함수가 단 하나뿐임, 한가지 작업을 여러가지 다른 방법으로 수행<br/>
  
  - ### 단계 2 : View함수의 분리 및 일관성 향상
<img width="845" alt="스크린샷 2024-10-05 오전 2 43 43" src="https://github.com/user-attachments/assets/f2ed36af-c618-415b-91f7-555f9586d1a4"><br/>
    - 💣 모든 뷰 함수를 수동으로 호출

  - ### 단계 3 : 구성 요소 함수
    - 수동으로 함수 호출하는 부분을 자동으로 변환시키기
    - ```js
       export default (targetElement, state) => {  
        const element = targetElement.cloneNode(true)  
          
        const list = element  
        .querySelector('.todo-list')  
        const counter = element  
        .querySelector('.todo-count')  
        const filters = element  
        .querySelector('.filters')  
          
        list.replaceWith(todosView(list, state))  
        counter.replaceWith(counterView(counter, state))  
        filters.replaceWith(filtersView(filters, state))  
          
        return element  
        }
      ```
    - step 1: index.html의 해당 쿼리에 구성요소의 이름을 data-component속성에 넣기<br/>
      ex) `<ul class="todo-list" data-component="todos">`
    - step 2: 구성요소 이름과 뷰를 매칭한 구성 요소 레지스트리 생성 (핵심 메커니즘)
      ```
      const registry = {
       'todos': todosView,
       'counter': counterView,
       'filters':filterView
      }
      ```
    - step 3: 구성 요소 래핑을 위한 고차함수 생성(고차함수 : 하나이상의 함수를 인자로 받는 함수)
      - 모든 구성요소가 data-component속성의 값을 읽고(`const childComponents = element
            .querySelectorAll('[data-component]')`) 올바른 함수를 자동으로 호출(`const child = registry[name]`)하는 기본 구성 요소에서 상속되어야한다.
      ```js
      const registry={}
      const renderWrapper = component => {
        return (targetElement, state) => {
          const element = component(targetElement, state)
      
          const childComponents = element
            .querySelectorAll('[data-component]')
      
          Array
            .from(childComponents)
            .forEach(target => {
              const name = target
                .dataset
                .component
      
              const child = registry[name]
              if (!child) {
                return
              }
      
              target.replaceWith(child(target, state))
            })
      
          return element
        }
      }
      ```
    - step 4: 레지스트리 추가 함수
      ```js
      const add = (name, component) => {
        registry[name] = renderWrapper(component)
      }
      ```
    - step 5: 부팅 함수
      ```js
      const renderRoot = (root, state) => {
        const cloneComponent = root => {
          return root.cloneNode(true)
        }
      
        return renderWrapper(cloneComponent)(root, state)
      }
      ```
    - step 6:위 함수들을 이용한 컨트롤러 생성
      ```js
      import getTodos from './getTodos.js'
      import todosView from './view/todos.js'
      import counterView from './view/counter.js'
      import filtersView from './view/filters.js'
      
      import registry from './registry.js'
      
      registry.add('todos', todosView)
      registry.add('counter', counterView)
      registry.add('filters', filtersView)
      
      const state = {
        todos: getTodos(),
        currentFilter: 'All'
      }
      
      window.requestAnimationFrame(() => {
        const main = document.querySelector('.todoapp')
        const newMain = registry.renderRoot(main, state)
        main.replaceWith(newMain)
      })
      ```
- ## 💡 동적 데이터 랜더링
  - ⭐️⭐️⭐️⭐️⭐️ 이 섹션은 항플3주차 과제와 아주 유사합니다! 해당 주차 과제 다시보고오기! ⭐️⭐️⭐️⭐️⭐️
  - [해당 섹션 관련 준일님 블로그](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/#_1-%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3-%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC)
  - ### 가상 DOM
    - 
    - 동적으로 데이터 변동시 변경된 부분은 변경 될 때마다 바로 실제 DOM에 반영되는 것이 아니라 batching과정을 거쳐 전체 리랜더링이 아닌 변경 부분만 리랜더링 될 수 있게 한다. 그러니까 인형이 팔이 하자가 있다고 해서 인형전체를 새걸로 바꾸는게 아니라 팔만 바꾸는거랑 비슷하게 생각하면 된다..(?)
    - 이러한 과정 (reconcilation)을 통해 랜더링 엔진의 성능이 향상된다!!!
    - ![스크린샷 2024-10-05 오전 3 49 42](https://github.com/user-attachments/assets/f5cdeb8a-446b-414a-a4d8-929407d2f663)
  - ### diff 알고리즘
    - 속성 수가 다르다.
    - 하나 이상의 속성이 변경 됐다.
    - 노드에는 자식이 없으며 textContext가 다르다.
    - 위 세가지 조건을 거쳐 가상DOM과 실제 DOM이 다른지 체크하여 프레임 마다 업데이트 시켜준다!

    


  
