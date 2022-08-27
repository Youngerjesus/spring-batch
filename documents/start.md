# 스프링 배치 시작 

## 초기화 

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

## 예시로 알아보는 스프링 배치 잡 

```java
@Configuration
public class HelloJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    public HelloJobConfiguration(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
        this.jobBuilderFactory = jobBuilderFactory;
        this.stepBuilderFactory = stepBuilderFactory;
    }

    @Bean
    public Job helloJob() {
        return jobBuilderFactory.get("helloJob")
                .start(helloStep())
                .build();
    }

    private Step helloStep() {
        return stepBuilderFactory.get("helloStep")
                .tasklet(((contribution, chunkContext) -> {
                    System.out.println("Hello Spring Batch");
                    return RepeatStatus.FINISHED;
                }))
                .build();
    }
}
```

- `@Configuration` 을 통해서 하나의 배치 Job 을 정의하고 빈을 설정한다. 
  - **(Job 을 만들거면 Configuration 파일을 만들면 되는건가)**
- `JobBuilderFactory` 를 통해서 Job 을 생성한다.
- `StepBuilderFactory` 를 통해서 Step 을 생성한다.
- `Job` 안에 `Step` 이 있다. `Step` 안에 `tasklet` 이 있다.
  - `tasklet` 안에 비즈니스 로직을 넣으면 된다.
  - 개념적으로 Job 은 일이고, Step 은 일을 하기 위한 단계, tasklet 은 그 구현에 대한 내용이다.
- `Step` 안에서 단일 태스크로 동작하는 로직을 구현할려면 `tasklet` 을 이용하면 된다.
  - tasklet 은 기본적으로 반복적으로 실행되는데 한번 실행되고 종료를 할려면 `RepeatStatus.FINISHED` 를 하면 된다.
  - `RepeatStatus.CONTINUABLE` 을 하면 무한으로 반복된다. 다음 단계로 못가고.

## 스프링 배치 메타데이터 

- 스프링 배치의 실행 및 관리를 목적으로 Job, Step, JobParameter 등의 정보를 저장, 업데이트, 조회할 수 있는 스키마가 필요하다. 
  - 이를 Spring Batch Core 에서 제공해줌. 스키마 파일들을.
  - 제공해주는 이유는 실행 및 관릴 때문인데 Job 의 성공과 실패등을 관리해주는 거니까 필요함.
  - DB 와 연동할 경우에는 필수적으로 이런 메타 테이블을 생성해야한다.
- DB 스키마는 /org/springframework/batch/core/schema 에 있다.
- 스키마 생성은 수동 생성과 자동 생성이 있다.
  - 수동 생성은 스키마 파일에 있는 모든 sql 문을 복사해서 실행해야한다.
  - 자동 생성은 `spring.batch.jdbc.initialize-schema` 설정에 따라서 다르다.
    - ALWAYS
      - 스크립트를 항상 실행한다.
      - RDB 설정을 한 경우 내장 DB 보다 먼저 실행한다. 
    - EMBEDDED
      - 내장 DB 일때만 실행한다. 내장 DB 보다 우선적으로 실행한다.
    - NEVER
      - 스크립트를 항상 실행 안한다. 
      - 운영환경일 경우에는 수동으로 스크립트 생성 후 실행하는 걸 권장한다. 

## DB 스키마 생성

### Job 관련 테이블 

- `BATCH_JOB_INSTANCE`
  - Job 이 실행될 때 JobInstance 정보가 저장되고 job_name 과 job_key 를 키로하여 데이터를 생성한다.
  - 동일한 job_name 과 job_key 로 중복 저장될 순 없다. (둘이 합쳐서 unique_key 로 지정)
  - 칼럼을 살펴보면 다음과 같다.
    - job_instance_id: 기본키 
    - version: 업데이트 될 때마다 1씩 증가 
    - job_name: job 을 구성하는 job 의 이름
    - job_key: jobParameter 와 job_name 을 합쳐서 해싱한 값이다
- `BATCH_JOB_EXECUTION`
  - job 의 실행정보가 저장되고 job 생성, 시작, 종료 시간, 실행 상태, 메시지 등을 관리한다.
  - 주요 칼럼을 보면 다음과 같다. 
    - job_execution_id: job_execution 을 식별하는 고유키
    - version: 업데이트 될 때마다 1씩 증가 
    - job_instance_id: job_instance 의 값이 저장
    - create_time: 실행이 생성된 시간 
    - start_time: 실행이 시작된 시간 
    - end_time: 실행이 종료된 시간. 오류로 인해서 중단되는 경우에는 값이 없을 수 있다.
    - status: 실행의 상태 (COMPLETED, FAILED, STOPPED)
    - exit_code: 실행 종료 코드 (COMPLETED, FAILED)
    - exit_message: status 가 실패일 경우 원인이 여기에 들어감
    - last_updated: 마지막 실행 시점을 timestamp 형식으로 기록 
- `BATCH_JOB_EXECUTION_PARAMS`
  - job 과 함께 실행되는 JobParameter 정보를 저장한다.
  - 주요 칼럼을 보면 다음과 같다.
    - job_execution_id: jobExecution 의 식별키. JobExecution 과 일대다 관 
    - TYPE_CD: STRING, LONG, DATE, DOUBLE 4 가지 타입 정보 
    - KEY_NAME: 파라미터 키 값 
    - DATE_VAL: 파라미터 날짜 값 
    - STRING_VAL: 파라미터 문자열 값
    - DOUBLE_VAL: 파라미터 DOUBLE 값
    - LONG_VAL: 파라미터 LONG 값 
    - IDENTIFYING: 식별 여부 (true or false)
- `BATCH_JOB_EXECUTION_CONTEXT`
  - job 의 실행동안 여러가지 상태 정보들, 공유 데이터를 json 화 해서 저장한다. 
  - Step 간에 공유도 가능하다. 
  - 주요 칼럼은 다음과 같다. 
    - JOB_EXECUTION_ID: jobExecution 의 식별키 job_execution 마다 생성한다. 
    - SHORT_CONTEXT: JOB 의 실행 상태 정보, 공유 데이터 등의 정보를 문자열로 저장 
    - SERIALIZED_CONTEXT: 직렬화 된 컨택스트 정보  

### Step 과 관련된 정보들 

- `BATCH_STEP_EXECUTION`
  - Step 의 실행 정보가 저장되고 생성, 시작, 종료, 실행상태, 메시지 등을 관리한다.
- `BATCH_STEP_EXECUTION_CONTEXT`
  - Step 의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화해서 저장한다. 
  - Step 별로 저장되며 Step 간 공유하는게 안됨
