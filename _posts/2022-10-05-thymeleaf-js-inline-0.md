---
layout: post
title: 자바스크립트 인라인
tag: 
---
### 자바스크립트 인라인이란?
타임리프에서 가져온 변수 또는 값을 자바스크립트에서 사용할 수 있게 하는 것이다
### 자바스크립트 인라인 사용법
1. script 태그에 다음 속성을 추가한다. th:inline="javascript"
2. 사용하고자 하는 변수를 다음과 같이 감싸준다 [[${변수명}]]

예시)
```html
<script th:inline="javascript">
    var myName = [[${name}]];
</script>
```

### 적용
  
자바스크립트 인라인을 사용하면 타임리프에서 값을 알맞은 자료형으로 자동 변환해준다.
객체는 JSON형식으로 자동 변환해준다.

예시)
```
-변수
    인라인 사용 전 var username = userA;
    인라인 사용 후 var username = "userA";
    
-객체
    인라인 사용 전 var user = BasicController.User(username=userA, age=10);
    인라인 사용 후 var var user = {"username":"userA","age":10};
```
  
참으로 편리하다
  
### 반복문 each
each문 또한 사용이 가능하다
  
예시)
```javascript
[# th:each="user, stat : ${users}"]
  var user[[${stat.count}]] = [[${user}]];
[/]
```
  
  

결과)
```javascript
  var user1 = {"username":"userA","age":10};
  var user2 = {"username":"userB","age":20};
  var user3 = {"username":"userC","age":30};
```
  
미쳤다. 개편하다