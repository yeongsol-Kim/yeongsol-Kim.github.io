---
layout: post
title: Data JPA - 테스트 케이스 작성
tag: [Spring]
---

진행 중이던 외주 프로젝트에 새로 API를 추가할 일이 생겼다.  
매우매우 간단한 내용이지만 블랙박스 테스트를 해보기가 번거로운 상황이다.  
카카오톡 로그인을 사용하여 테스트 해야 하고, 그러려면 휴대폰 인증을 해야 하지만 내가 지금 호주에 있기 떄문이다.  
그래서 테스트 코드를 작성해서 테스트 해보기로 하였다!  

사실 테스트 코드 작성은 이런 특정 상황이 아니더라도 항상 해야하는 선택이 아닌 "필수" 항목이지만 부끄럽게도 나는 여태까지 테스트코드 작성을 해본 적이 거의 없다.  
부끄럽다는 말로 퉁 치기엔 너무나도 부끄러운 일이다.  
항상 언젠가부터 습관을 들여야겠다는 생각은 항상 하고 있었지만, 테스트용 리포지토리와 저장소도 매번 따로 만들 생각에 그만큼 시간도 쏫아야 하고 일정도 빡빡해서(사실 추잡한 핑계의 불과하다) 부딪치기가 무서웠었다.
아무튼 지금부터라도 습관을 들이기 위해 시도를 해보았는데, 그 과정에서 겪은 문제와 해결한 방법들이 기록 한다!



### 개발
개발 사항은 다음과 같다.
- 사용자의 로그인 정보를 이용해 사용자가 작성한 글 목록을 반환하는 API

#### Repository
```java
@Repository
public interface GiLogRepository extends JpaRepository<GiLog, Long> {
    ...
    List<GiLog> findByUserId(Long userid);
    ...
}
```
데이터 JPA를 사용 중이기 때문에 다음과 같이 한 줄만 추가하였다

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
