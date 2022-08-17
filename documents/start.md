# 스프링 배치 시작 

- `@EnableBatchProcessing` 이라는 애노테이션을 통해서 스프링 배치가 작동한다.
  - 이 애노테이션은 4개의 설정 파일을 활성화 시킨다.
    - `BatchAutoConfiguration`
      - Job 을 수행하는 `JobLauncherApplicationRunner` 빈을 생성한다. 
      - JobLauncherApplicationRunner 는 `ApplicationRunner` 를 상속받았다. 
      - job 을 실행하기 때문에 가장 마지막에 실행된다.
    - `SimpleBatchConfiguration`
      - JobBuilderFactory 와 StepBuilderFactory 를 생성한다.
      - 스프링 배치의 주요 구성 요소를 생성한다.
      - 프록시 객체로 생성됨
      - 가장 먼저 실행되는 설정 파일이다. 배치 구성 요소를 생성해야하므로.
    - `BatchConfigurerConfiguration`
      - `BasicBatchConfigurer`
        - `SimpleBatchConfiguration` 에서 생성한 프록시 객체의 실제 대상 클래스
          - 빈으로 의존받아서 주요 객체를 참조하는게 가능함.
        - `JpaBatchConfiguer`
          - JPA 관련 객체를 생성하는 설정 클래스 
        - 그리고 사용자 정의 BatchConfigurer 클래스를 생성하는 것도 가능하다.
