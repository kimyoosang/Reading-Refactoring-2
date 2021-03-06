# Chapter11. API 리팩터링

- 모듈과 함수는 소프트웨어를 구성하는 빌딩 블록이며, API는 이 블록들을 끼워 맞추는 연결부다. 이런 API를 이해하기 쉽고 사용하기 쉽게 만드는 일은 중요한 동시에 어렵기도 하다
- 그래서 API를 개선하는 방법을 깨달을 떄마다 그에 맞게 리팩터링 해야 한다

## **11.1 질의 함수와 변경 함수 분리하기**

### **배경**

- 우리는 외부에서 관찰할 수 있는 겉보기 부수효과가 전혀 없이 값을 반환해주는 함수를 추구해야 한다. 이런 함수는 어느 때건 원하는 만큼 호출해도 아무 문제가 ㅇ벗다. 호출하는 문장의 위치를 호출하는 함수 안 어디로든 옮겨도 되며 테스트하기도 쉽다. 한마디로, 이용할 때 신경 쓸 거리가 매우 적다
- 겉보기 부수효과가 있는 함수와 없는 함수는 명확히 구분하는 것이 좋다. 이를 위한 한 강지 방법은 '질의 함수(읽기 함수) 는 모두 부수효과가 없어야 한다'는 규칙을 따르는 것이다. 이를 **명령-질의 분리**라 한다
- 흔히 쓰는 최적화 기법 중 요청된 값을 캐시해두고 다음번 호출 때 빠르게 응답하는 방법이 있는데, 이러한 캐싱도 객체의 상태를 변경하지만 객체 밖에서는 관찰할 수 없다. 즉, 겉보기 부수효과 없이 어떤 순서로 호출하든 모든 호출에 항상 똑같은 값을 반환할 뿐이다

### **절차**

1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다
2. 새 질의 함수에서 부수효과를 모두 제거한다
3. 정적 검사를 수행한다
4. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 질의에 새로 추가한다. 하나 수정할 때마다 테스트한다
5. 원래 함수에서 질의 관련 코드를 제거한다
6. 제거한다

### **예제**

- [질의 함수와 변경 함수 분리하기](./Example/SeparateQueryFromModifier.md)

## **11.2 함수 매개변수화하기**

### **배경**

- 두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다
- 이렇게 하면 매개변수 값만 바꿔서 여러곳에서 쓸 수 있으니 함수의 유용성이 커진다

### **절차**

1. 비슷한 함수 중 하나를 선택한다
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다
4. 테스트한다
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다
6. 비슷한 다른 함수를 호출하는 곳을 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다

### **예시**

- [함수 매개변수화하기](./Example/ParamererizeFunction.md)

## **11.3 플래그 인수 제거하기**

### **배경**

- 플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다
- 플래그 인수는 호출할 수 있는 함수들이 무엇이고 어떻게 호출해야 하는지를 이해하기 어렵게 만든다. 플래그 인수가 있으면 API를 익힐때 함수들의 기능 차이가 잘 드러나지 않는다
- 사용할 함수를 선택한 후에도 플래그 인수로 어떤 값을 넘겨야 하는지를 또 알아내야 한다. 불리언 플래그는 코드를 읽는 이에게 뜻을 온전히 전달하지 못하기 때무넹 더욱 좋지 못하다
- 플래그 인수가 되려면 호출하는 쪽에서 불리언 값으로 리터럴 값을 건넨야 한다. 또한, 호출되는 함수는 그 인수를 제어 흐름을 결정하는 데 사용해야 한다
- **플래그 인수를 제거하면 코드가 깔끔해짐은 물론 프로그래밍 도구에도 도움을 준다** 예컨데 코드 분석 도구는 프리미엄 로직 호출과 일반 로직 호출의 차이를 더 쉽게 파악할 수 있게 된다
- 함수 하나에서 플래그 인수를 두 개 이상 사용하면 플래그 인수를 써야 하는 합당한 그거가 될 수 있다. 플래그 인수 없이 구현하려면 플래그 인수들의 가능한 조합 수만큼의 함수를 만들어야 하기 때문이다. 그런데 다른 고나점에서 보자면, 플래그 인수가 둘 이상이면 함수 하나가 너무 많은 일을 처리하고 있다는 신호이기도 하다. 그러니 같은 로직을 조합해내는 더 간단한 함수를 만들 방법을 고민해봐야 한다

### **절차**

1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다
2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응하는 명시적 함수를 호출하도록 수정한다

### **예시**

- [플래그 인수 제거하기](./Example/RemoveFlagArgument.md)

## **11.4 객체 통째로 넘기기**

### **배경**

- 하나의 레코드에 값 두어 개를 가져와 인수로 넘기는 코드를 보면, 그 값들 대신 레코드를 통째로 넘기고 함수 본문에서 필요한 값들을 꺼내 쓰도록 수정하는 것이 좋다
- 레코드를 통째로 넘기면 변화에 대응하기 쉽다. 예컨대 그 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요가 ㅇ벗다. 그리고 매개변수 목록이 짧아져서 일반적으로는 함수 사용법을 이해하기 쉬워진다. 한편, 레코드에 담긴 데이터 중 일부를 받는 함수가 여러 개라면 그 함수들 끼리는 같은 데이터를 사용하는 부분이 있을 것이고, 그 부분의 로직이 중복될 가능성이 커니다. **레코드를 통째로 넘긴다면 이런 로직 중복도 없앨 수 있다**
- 하지만 함수가 레코드 자체에 의존하기를 원치 않을 떄는 이 리팩터링을 수행하지 않는데, 레코드와 함수가 서로 다른 모듈에 속한 상황이면 특히 더 그렇다
- 객체 통째로 넘기기는 특히 매개벼너수 만들기 후, 즉 산재한 수많은 데이터 더미를 새로운 객체로 묶은 후 적용하곤 한다
- 다른 객체의 메서드를 호출하면서 호출하는 객체 자신이 가지고 있는 데이터 여러 개를 건네느 경우에, 데이터 여러 개 대신 객체 자신의 참조만 건데도록 수정할 수 있다(자바스크립트라면 this를 건넬 것이다)

### **절차**

1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다
2. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다
3. 정적 검사를 수행한다
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트한다
5. 호출자를 모두 수정했다면 원래 함수를 인라인한다
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다

### **예시**

- [객체 통째로 넘기기](./Example/PreserveWholeObject.md)

## **11.5 매개변수를 질의 함수로 바꾸기**

### **배경**

- 매개변수 목록은 함수의 변동 요인을 모아놓은 곳이다. 즉, 함수의 동작에 변화를 줄 수 있는 일차적인 수단이다. 다른 코드와 마찬가지로 이 목록에서도 중복은 피하는게 좋으며 짧을 수록 이해하기 쉽다
- 피호출 함수가 스스로 '쉽게' 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복이다. 이런 함수를 호출할 때 매개변수의 값은 호출자가 정하게 되는데, 이 결정은 사실 하지 않아도 되었을 일이니 의미 없이 코드만 복잡해질 뿐이다
- 매개변수를 제거하면 값을 결정하는 책임 주체가 달라진다. 매개변수가 있다면 결정 주체가 호출자가 되고, 매개변수가 없다면 피호출 함수가 된다
- 배개변수를 질의 함수로 바꾸지 말아야 할 상황도 있다. 가장 흔한 예는 매개변수를 제거하면 피호출 함수에 원치 않는 의존성이 생길 때다. 즉, 해당 함수가 알지 못했으면 하는 프로그램 요소에 접근해야 하는 상황을 만들때다
- 제거하려는 매개변수의 값을 다른 매개변수에 질의해서 얻을 수 있다면 안심하고 질의 함수로 바꿀 수 있다. 다른 매개변수에서 얻을 수 있는 값을 별도 매개변수로 전달하는 것은 아무 의미가 없다
- 주의사항이 하나있다. 대상 함수가 '참조 투명'해야 한다는 것이다. 참조 투명이란 '함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작한다'는 뜻이다. 이런 함수는 동작을 예측하고 테스트하기가 훨씬 쉬우니 이 특성이 사라지지 않도록 주의하자. 따라서 매개변수를 없애는 대신 가변 전역 변수를 이용하는 일은 하면 안된다

### **절차**

1. 필요하다면 대상 매개변수의 값을 계산하는 코드를 변도 함수로 추출해놓는다
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다

### **예시**

- [매개변수를 질의 함수로 바꾸기](./Example/ReplaceParameterWithQuery.md)

## **11.6 질의 함수를 매개변수로 바꾸기**

### **배경**

- 코드를 읽다 보면 함수 안에 두기엔 거북한 참조를 발견할 때가 있다. 전역 변수를 참조한다거나 제거하길 원하는 원소를 참조하는 경우가 여기 속한다
- 이 문제는 해당 참조를 매개변수로 바꿔 해결할 수 있다. 참조를 풀어내는 책임을 호출자로 옮기는 것이다
- 이런 상황 대부분은 코드의 의존관계를 바꾸려 할 때 벌어진다
- 똑같은 값을 건네면 매번 똑같은 결과를 내는 함수는 다루기 쉽다. 이런 성질을 '참조 투명성'이라 한다. 참조 투명하지 않은 함수에 접근하는 모든 함수는 참조 투명성을 읽게 되는데, 이 문제는 해당 원소를 매개변수로 바꾸면 해결된다
- 그래서 모듈을 개발할 때 순수 함수들을 따로 구분하고, 프로그램의 입출력과 기타 가변 원소들을 다루는 로직으로 순수함수들의 겉을 감싸는 패턴을 많이 활용한다
- 이 리팩터링의 **단점**은 질의 함수를 매개변수로 바꾸면 어떤 값을 제공할지를 호출자가 알아내야 한다. 결국 호출자가 복잡해지는데, 이왕이면 고객(호출자)의 삶이 단이 단순해지도록 설계하자는 의견과 배치된다. 이 문제는 결국 책임 소재를 프로그램의 어디에 배정하느냐의 문제로 귀결된다

### **절차**

1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다
3. 방금 만든 변수를 인라인하여 제거한다
4. 원래 함수도 인라인한다
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다

### **예시**

- [질의 함수를 매개변수로 바꾸기](./Example/ReplaceQueryWithParameter.md)

## **11.7 세터 제거하기**

### **배경**

- 세터 메서드가 있다고 함은 필드가 수정될 수 있다는 뜻이다. 객체 생성 후에는 수정되지 않길 원하는 필드라면 세터를 제공하지 않았을 것이다. 그러면 해당 필드는 오직 생성자에서만 설정되며, 수정하지 않겠다는 의도가 명명백백해지고 변경될 가능성이 봉쇄된다
- 세터 제거하기 리팩터링이 필요한 상황은 주로 두 가지다. 첫째, 사람들이 무조건 접근자 메서드를 통해서만 필드를 다루려 할 때다. 이러면 오직 생성자에서만 호출하는 세터가 생겨나곤 한다
- 두 번째 상황은 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때다. 생성 스크립트란 생성자를 호출한 후 일련의 세터를 호출하여 객체를 완성하는 형태의 코드를 말한다(별도의 스크립트 파일이 아니다). 그러면서 설계자는 스크립트가 완료된 뒤로는 그 객체의 필드 일부는 변경되지 않으리라 기대한다
- 즉, 해당 세터들은 처음 생성할 때만 호출되리라 가정한다. 이런 경우에도 세터들을 제거하여 의도를 더 정확하게 전달하는 게 좋다

### **절차**

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다(함수 선언 바꾸기)
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 떄마다 테스트한다
3. 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다
4. 테스트한다

### **예시**

- [세터 제거하기](./Example/RemoveSettingMethod.md)

## **11.8 생성자를 팩터리 함수로 바꾸기**

### **배경**

- 많은 객체 지향 언어에서 제공하는 생성자는 객체를 초기화하는 특별한 용도의 함수다
- 그러마 생성자에서는 일반 함수에는 없는 이상한 제약이 따라붙기도 한다. 가령 자바 생성자는 반드시 그 생성자를 정의한 클래스의 인스턴스를 반환해야 한다. 서브 클래스의 인스턴스나 프락시를 반환할 수는 없다. 생성자의 이름도 고정되어, 기본 이름보다 더 적절한 이름이 있어도 사용할 수 없다. 생성자를 호출하려면 특별한 연산자 (많은 언어에서 new를 쓴다)를 사용해야 해서 일반 함수가 오길 기대하는 자리에는 쓰기 어렵다
- 팩터리 함수에는 이런 제약이 없다. 팩터리 함수를 구현하는 과정에서 생성자를 호출할 수는 있지만, 원한다면 다른 무언가로 대체할 수 있다

### **절차**

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다
3. 하나씩 수정할 때마다 테스트한다
4. 생성자의 가시 범위가 최소가 되도록 제한한다

### **예시**

- [생성자를 팩터리 함수로 바꾸기](./Example/ReplaceContructorWithFactoryFunction.md)

## **11.9 함수를 명령으로 바꾸기**

### **배경**

- 함수(독립된 함수든 객체에 쏙된 메서드든)는 프로그래밍의 기본적인 빌딩 블록 중 하나다. 그런데 함수를 그 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황이 있다. 이런 객체를 가리켜 **_명령 객체_**혹은 단순히 **_명령_**이라 한다
- 명령 객체 대부분은 메서드 하나로 구성되며, 이 메서드를 요청해 실앻하는 것이 이 객체의 목적이다
- 명령은 평범한 함수 매커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다. 명려어은 되돌리기 갍은 보조 연산을 제공할 수 있으며, 생명주기를 더 정밀하게 제어하는 데 필요한 매개변수를 만들어주는 메서드도 제공할 수 있다. 상속과 훅을 이용해 사용자 맞춤형으로 만들 수도 있다. 객체는 지원하지만 일급 함수를 지원하지 않는 프로그래밍 언어를 사용할 때는 명령을 이용해 일급 함수의 기능 대부분을 흉내낼 수 있다
- 비슷하게, 중첩 함수를 지원하지 않는 언어에서도 메서드와 필드를 이용해 복잡한 함수를 잘게 쪼갤 수 있고, 이렇게 쪼갠 메서드들을 테스트와 디버깅에 직접 이용할 수 있다

### **절차**

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다
2. 방금 생성한 빈 클래스로 함수를 옮긴다
3. 함수의 인수들 각각은 명려어의 필드로 만들어 생성자를 통해 설정할지 고민해본다

### **예시**

- [함수를 명령으로 바꾸기](./Example/ReplaceFunctionWithCommand.md)

## **11.10 명령을 함수로 바꾸기**

### **배경**

- 명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다. 구체적으로는, 큰 연산 하나를 여러 개의 작은 메서드로 쪼개고 필드를 이용해 쪼개진 메서드들끼리 정보를 공유할 수 있다. 또한 어떤 메서드를 호출하냐에 따라 다른 효과를 줄 수 있고 각 단계를 거치며 데이터를 조금씩 완성해갈 수도 있다
- 명령은 그저 함수를 하나 호출해 정해진 일을 수행하는 용도로 주로 쓰인다. 이런 상황이고 로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는게 낫다

### **절차**

1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다
2. 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인한다
3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명려으이 실행 메서드로 옮긴다
4. 명려어의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다
5. 생성자 호출과 명려ㅓ으이 실행 메서드 호출을 호출자 안으로 인라인한다
6. 테스트한다
7. 죽은 코드 제거하기로 명령 클래스를 없앤다

### **예시**

- [명령을 함수로 바꾸기](./Example/ReplaceCommandWithFunction.md)

## **11.11 수정된 값 반환하기**

### **배경**

- 데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분중 하나다. 특히 같은 데이터 블록을 읽고 수정하는 코드가 여러 곳이라면 데이터가 수정되는 흐름과 코드의 흐름을 일치시키기다 상당히 어렵다. 그래서 데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다
- 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다. 이 방식으로 코딩하면 호출자 코드를 읽을 때 변수가 갱신될 것임을 분명히 인지하게 된다. 해당 변수의 값을 단 한 번만 정하면 될 때 특히 유용하다
- 이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 가장 효과적이고, 반대로 값 여러개를 갱신하는 함수에는 효과적이지 않다

### **절차**

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다
2. 테스트한다
3. 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다
4. 테스트한다
5. 계산이 선언과 동시에 이뤄지도록 통합한다
6. 테스트한다
7. 피호출 함수의 변수 이름을 새 역할에 어올리도록 바꿔준다
8. 테스트한다

## **예시**

- [수정된 값 반환하기](./Example/ReturnModifiedValue.md)

## **11.12 오류 코드를 예외로 바꾸기**

### **배경**

- 예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘이다. 오류가 발견되면 예외를 던진다. 그러면 적절한 예외 핸들러를 찾을 때까지 콜스택을 타고 위로 전파된다
- 예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 된다. 예외에는 독자적인 흐름이 있어서 프로그램의 나머지에서는 오류 발생에 따른 복잡한 상황에 대처하는 코드를 작성하거나 읽을 일이 없게 해준다
- 예외는 정교한 메커니즘이지만 대다수의 다른 정교한 메커니즘과 같이 정확하게 사용할 때만 최고의 효과를 낸다. 예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다. 달리 말하면 프로그램의 정상 범주에 들지 않는 오류를 나타낼 때만 쓰여야 한다
- 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것이다. 정상 동작하지 않을 것 같다면 예외를 사용하지 말라는 신호다. 예외 대신 오류를 검출하여 프로그램을 정상 프름으로 되돌리게끔 처리해야 한다

### **절차**

1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다
2. 테스트한다
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다
4. 정적 검사를 수행한다
5. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다
6. 테스트한다
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다
8. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다

### **예시**

- [오류 코드를 예외로 바꾸기](./Example/ReplaceErrorCodeWithException.md)

## **11.13 예외를 사전확인으로 바꾸기**

### **배경**

- 예외는 '뜻밖의 오류'라는, 말 그대로 예외적으로 동작할 때만 쓰여야 한다. 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지를 대신 호출하는 곳에서 조건을 검사하도록 해야 한다

### **절차**

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다
2. catch 블록에 어서션을 추가하고 테스트한다
3. try문과 catch 블록을 제거한다
4. 테스트한다

### **예시**

- [예외를 사전학인으로 바꾸기](./Example/ReplaceExceptionWithPrecheck.md)
