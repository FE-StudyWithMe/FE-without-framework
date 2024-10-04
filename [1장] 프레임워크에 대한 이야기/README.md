## [1장] 프레임워크에 대한 이야기

- ### 💡 프레임 워크란?
	- 무언가를 만들 수 있는 지지 구조
- ### 💡 프레임 워크 vs 라이브러리
	- 프레임 워크는 코드를 호출한다. 코드는 라이브러리를 호출한다.
  <img width="805" alt="스크린샷 2024-10-04 오후 8 46 39" src="https://github.com/user-attachments/assets/200a6296-f44f-4986-8730-30da0c2d7583">
- ### 💡 🤔 이런 느낌?
	- 라이브러리 (예: React): "**여기 도구가 있습니다. 필요한 대로 사용하세요.**"
	- 프레임워크 (예: Angular): "**이렇게 사용하는 것이 가장 좋습니다. 우리가 제공하는 방식을 따라주세요.**"
- ### 💡 라이브러리인지 프레임워크인지 판별하는 기준은 제약조건이 있느냐에 달려있다.
	- point 🙋 : 예시가 앵귤러로 나온 이유는 프레임워크 중에서도 설명하고자 하는 제약조건이 모두 있는 프레임 워크이기 때문!
	- **언어**
		- 프레임 워크는 특정 언어를 강제하거나 선호한다.
		- 앵귤러의 경우 타입스크립트에 대해 강제성을 띄지는 않지만 공식문서 등 에서도 강력하게 권유한다.
		- 트랜스파일이 필요하게 됨.
	- **의존성 주입**
		- 프레임 워크 내부의 동작을 사용하게 의존성을 연결시켜둠
		- Angular에서는 서비스나 컴포넌트가 필요로 하는 의존성을 프레임워크가 자동으로 제공.
		- React는 Context API를 강제하지 않고 다른 라이브러리로 상태관리를 할 수 있지만 다른 프레임 워크의 경우에는 내장된 상태관리 로직을 사용해야만 한다!
	- **옵저버블**
		- 데이터의 변화를 감지하고 그에 반응하는 것
		- React의 경우 useState, useEffect, contextAPI로 옵저버블 라이크 사용
		- 프레임 워크 제약사항 : 특정 방식의 데이터 처리를 강제한다.
- ### 💡 리액트가 라이브러리 인가 프레임 워크 인가?
	- 공식문서에는 '사용자 인터페이스 구축을 위한 자바스크립트 라이브러리'
	- 리액트의 선언적 패턴<br/> : 주요 제약 사항
 ```
		  📍 명령형 프로그래밍: "어떻게(How)" 할 것인지를 상세히 기술합니다. 
		  📍 선언적 프로그래밍: "무엇을(What)" 원하는지를 기술합니다.
 ```
  - React에서의 선언적 패턴: React는 "어떻게 DOM을 변경할 것인가"가 아니라 "어떤 UI를 원하는가"를 기술
	 - JSX: **HTML-like 문법으로 UI를 선언적으로 표현합니다.**
	- 상태 관리: `useState`와 같은 훅을 통해 상태 변화를 선언적으로 관리합니다.
	- 생명주기 관리: `useEffect`를 통해 부수 효과를 선언적으로 정의합니다.
	- 책의 예시 : 리액트는 어떤 ui 변화시 직접 ui를 변화시키는것이 아닌 상태변경을 하게 한다. ex)isVisible
- ### 💡 저자가 React가 프레임워크라고 믿는 이유
    예시 1. 평소보는 코드
     ```js
	  import React, { useState } from 'react';

        function ToggleButton() {
      const [isOn, setIsOn] = useState(false);
    
      return (
        <button onClick={() => setIsOn(!isOn)}>
          {isOn ? 'ON' : 'OFF'}
        </button>
      );
      }
  ```

  ⚠️ 주의 : 이 아래 기묘함! <br/>
    예시 2. 동작은 같으나 다른 코드
    ```js
    import React, { useRef, useEffect } from 'react';
    
    function ToggleButton() {
      const buttonRef = useRef(null);
      
      useEffect(() => {
        const button = buttonRef.current;
        let isOn = false;
    
        const toggleState = () => {
          isOn = !isOn;
          button.textContent = isOn ? 'ON' : 'OFF';
        };
    
        button.addEventListener('click', toggleState);
    
        return () => button.removeEventListener('click', toggleState);
      }, []);
    
      return <button ref={buttonRef}>OFF</button>;
    }
    ```
**그니까 결국 저자는 사용자들이 기존 방식이 아닌 다른 방식을 사용했을때 "기묘함"을 느낀다면 이 자체가 프레임 워크적인 성격이 있지 않을까~ 생각한다는거다.**

- ### 💡 프레임 워크의 역사
	- 제이쿼리 -> 앵귤러JS ->리액트 -> 앵귤러 
- ### 💡 기술 부채
  - <img width="633" alt="스크린샷 2024-10-04 오후 8 54 17" src="https://github.com/user-attachments/assets/68fd4c17-f08b-44f0-b438-1a82ef4ecec7">
  - 디자인이 허술할수록 쌓이고 쌓여 더욱 커지는 부채
	- 프레임 워크의 기술 부채
		- 미래에 코드 변경이 어렵다는 점이 프레임워크에서의 기술 부채 비용
		- 그렇기 때문에 합당한 이유로 프레임 워크를 선정 해야한다! 필요할때만 사용할 것
