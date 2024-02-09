---
layout: post
title: Spring 고급편 - Proxy factory
tag: [Spring, Proxy]
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
다음은 타겟 클래스의 실행시간을 측정하는 어드바이스다. 

#### TimeAdvice.class
```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {
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
Advice는 MethodInterceotor를 상속받아 구현한다.(패키지 이름에 주의)
어드바이스를 구현할 때 타겟 클래스를 따로 지정해주지 않는다. 타겟 클래스 정보는 MethodInvocation에 지정되어있기 떄문이다.

#### ProxyFactoryTest
```java
    ...

    @Slf4j
    public class ProxyFactoryTest {
        @Test
        @DisplayName("인터페이스가 있으면 JDK 동적 프록시 사용")
        void interfaceProxy() {
            // 인터페이스가 있는 클래스
            ServiceInterface target = new ServiceImpl();
            ProxyFactory proxyFactory = new ProxyFactory(target);
            proxyFactory.addAdvice(new TimeAdvice());

            ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
            
            log.info("targetClass={}", target.getClass());
            log.info("proxyClass={}", proxy.getClass());
            proxy.save();

            // 인터페이스가 있는 클래스기 떄문에 프록시 팩초리가 자동으로 Jdk 동적 프록시방식 사용
            assertThat(AopUtils.isAopProxy(proxy)).isTrue();
            assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue(); 
            assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
        }

        @Test
        @DisplayName("구체 클래스만 있으면 CGLIB 사용") void concreteProxy() {
            // 구체클래스만 있는 클래스
            ConcreteService target = new ConcreteService();
            ProxyFactory proxyFactory = new ProxyFactory(target);
            proxyFactory.addAdvice(new TimeAdvice());
            
            ConcreteService proxy = (ConcreteService) proxyFactory.getProxy();
            
            log.info("targetClass={}", target.getClass());
            log.info("proxyClass={}", proxy.getClass());
            proxy.call();
            
            // 구체클래스만 있기 떄문에 프록시 팩토리가 자동으로 CGLIB 사용
            assertThat(AopUtils.isAopProxy(proxy)).isTrue();
            assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
            assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
        }
    }
    
    ...
```
ProxyFactory를 생성하면서 target클래스를 지정해준 후 addAdvice 메소드를 이용해 Advice를 지정해준다. Advice는 여러개 지정 가능하다.  
그 후에 getProxy를 이용하여 프록시를 가져오고 call메소드를 실행하면 된다.
