---
layout: post
title: Spring 고급편 - Proxy factory
tag: [Spring, JPA]
---


### 프록시 팩토리란?
스프링이 제공하는 프록시 라이브러리이다.  

지난 예제들을 통해, 프록시를 점점 발전시켜왔다.  
프록시를 편리하게 사용하기 위해서 인터페이스가 존재할 떄는 Jdk 동적 프록시(InvocationHandler), 구체클래스만 존재할 때는 CGLIB(MethodInterceptor)를 사용하여 보다 편리하게 관리하였다.

프록시 팩토리는 이 둘을 통합하여 인터페이스 유무에 따라 자동으로 프록시 기술을 선택하여 실행시킨다.  
Advice라는 새로운 개념을 도입하여 InvocationHandler나 MethodInterceptor가 Advice를 호출하게 한다.
결과적으로 InvocationHandler나 MethodInterceptor를 만들 필요 없이 Advice만 만들면 된다.  

프록시 팩토리는 Advice를 호출하는 전용 InvocationHandler와 MethodInterceptor를 내부에서 사용한다.

#### 로직 Client -> 프록시팩토리 -> JDK 동적 프록시 or CGLIB -> Advice -> Target


### 사용법
테스트 코드를 통해 사용법을 알아보겠다.
다음은 타겟 클래스의 실행시간을 측정하는 프록시이다. 

#### TestAdvice.class
```java
@Slf4j
public class TestAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        
        // 시간 측정 시작
        log.info("timeProxy 실행");
        long startTime = System.currentTimeMillis();
        
        // 타겟 클래스 실행
        Object result = invocation.proceed();
        
        //시간 측정 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}ms", resultTime);
        
        return result;
    } 
}
```
MethodInterceptor를 상속받아 invoke 메소드를 구현하면 된다.

#### Service
```java
    ...

    // 기록 목록 가져오기 (로그인 유저)
    public List<GiLogDto> getMyGiLogList(Long userId) {
        List<GiLog> giLogList = giLogRepository.findByUserId(userId);
    
        // 리스트 DTO 변환
        List<GiLogDto> giLogDtoList = gilogListToDto(giLogList);
    
        return giLogDtoList;
    }
    
    ...
```
이제 리포지토리에서 giLogList가 제대로 불러와지는지 테스트 코드를 작성 해보겠다.

### 테스트 코드 작성
테스트 코드 작성을 위해 리포지토리와 스토어(저장소)를 새로 만들어야하나 했지만  
Data Jpa를 손쉽게 테스트 할 수 있게 해주는 @DataJpaTest 어노테이션이 있음을 알게 되었다!  
@DataJpaTest는 JPA관련 빈들만 로드해 @SpringBootTest보다 빠르고 @Transactional을 포함하고 있다.
또 무려 내장형 DB를 가지고 있어 @Entity가 붙은 클래스들을 읽어 스프링 데이터 JPA 저장소를 자동으로 구성해준다는 개쩌는 능력을 갖고있다!!...만  
무슨 이유에서인지 계속해서 ApplicationContextError가 발생하였다 ㅠㅠㅠ,,  
나는 이 오류가 DB 저장소를 구축하는데서 발생하는 거라고 추측하고 이것저것 시도를 해보았지만 결국 실제 DB를 사용하는 방법 이외에는 알아내지 못했다,,    
내장형 DB를 꼭 써보고 싶었지만 일단은 테스트가 우선이니 나중에 다시 꼭 시도 해보아야겠다!  
  
내장형 db를 사용하지 않고 실제 db를 사용하는 방법은 다음과 같다.
1. @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) 추가 또는
2. application.yml파일에 test.database.replace: none 추가  

나는 1번 방법을 채택하였다.
#### ServiceTest
```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class GiLogServiceTest {

    @Autowired
    GiLogRepository giLogRepository;
    
    @Test
    void getMyGiLogList() {

        // 더미데이터 저장 (given)
        GiLog giLog1 = new GiLog();
        giLog1.setQuestion("question1?");
        giLog1.setRequest("request1");
        giLog1.setUserId(1L);
        giLog1.setWriteDate(LocalDate.parse("2022-10-01"));

        GiLog giLog2 = new GiLog();
        giLog2.setQuestion("question2?");
        giLog2.setRequest("request2");
        giLog2.setUserId(1L);
        giLog2.setWriteDate(LocalDate.parse("2022-10-02"));

        GiLog giLog3 = new GiLog();
        giLog3.setQuestion("question3?");
        giLog3.setRequest("request3");
        giLog3.setUserId(2L);
        giLog3.setWriteDate(LocalDate.parse("2022-10-01"));

        giLogRepository.save(giLog1);
        giLogRepository.save(giLog2);
        giLogRepository.save(giLog3);
        

        // 메소드 실행 (when)
        List<GiLog> giLogList = giLogRepository.findByUserId(1L);

        
        // 확인 (then)
        Assertions.assertThat(giLogList).contains(giLog1);
        Assertions.assertThat(giLogList).contains(giLog2);
        Assertions.assertThat(giLogList).doesNotContain(giLog3);
        
    }
}
```
ApplicationCotextError없이 정상적으로 실행 되고 테스트 또한 모두 성공하였다!  
  
이로써 스프링 데이터 JPA를 사용하는 테스트코드를 작성 해보았다.  
앞서 말했듯 테스트코드를 작성하는 습관을 정~말 열심히 들여야겠다.  
또 다음에 applicationContextError도 제대로 해결해보아야겠다!
