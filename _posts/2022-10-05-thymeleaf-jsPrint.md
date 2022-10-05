---
layout: post
title: Thymeleaf - 자바스크립트 인라인
---

<script> 태그에 다음 속성을 추가한다.
th:inline="javascript"

예시) <script th:inline="javascript">
  
  
타임리프 변수 출력 방법
[[${변수명}]]
  
값을 알맞은 자료형으로 자동 변환해준다.
객체는 JSON형식으로 자동 변환해준다.

예시)
  -변수
    인라인 사용 전 var username = userA;
    인라인 사용 후 var username = "userA";
  -객체
    인라인 사용 전 var user = BasicController.User(username=userA, age=10);
    인라인 사용 후 var var user = {"username":"userA","age":10};
  
  
참으로 편리하다
  
each
each문 또한 사용이 가능하다
  
예시)
  [# th:each="user, stat : ${users}"]
      var user[[${stat.count}]] = [[${user}]];
  [/]
  

결과)
  var user1 = {"username":"userA","age":10};
  var user2 = {"username":"userB","age":20};
  var user3 = {"username":"userC","age":30};

  
미쳤다. 개편하다
  
