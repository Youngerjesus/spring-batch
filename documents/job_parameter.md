# JobParameter 

## 기본 개념 

- Job 을 실행할 때 함꼐 포함되어서 사용하는 객체 
  - JobInstance 들을 구별하기 위해서 존재. 
  - JobParameter 와 JobInstance 1:1 
- JobParameter 는 key-value 로 저장한다.

- 생성 및 바인딩 
  - 어플리케이션 실행 시 주입 가능 
    - `java -jar LogBatch.jar requestDate=20210101`
    - 실행할 때 타입을 명시할 수 있다.
      - `age(double)=16.5` 이렇게. 기본은 String 
      - `date(date)=2022/06/12` 이렇게. 
  - 코드로 생성 가능 
    - JobParameterBuilder, DefaultJobParameterConverter 
  - SpEL 이용 
    - @Value("#(jobParameter[requestDate])"), @JobScope, @StepScope 선언 필수
      - `@Value` 로 시작하는 방식은 어플리케이션 실행할 때 전달받은 인자를 이용하는 방식이다.
  - **그래서 각 방식의 장단점이 모임써야하는거임?**
  
- `BATCH_JOB_EXECUTION_PARAM` 테이블과 매핑된다. 
  - `JOB_EXECUTION` 1:N 관계를 이룬다. 

- `JobParameters` 라는 클래스는 내부적으로 파라미터를 가지고 있기 위해서 LinkedHashMap 을 이용한다.
  - LinkedHashMap<String, JobParameter> 식으로 저장한다. 
  - JobParameters 가 Job 실행할 때 필요한거임.
  - JobParameter 내부는 다음과 같다.
    - Object parameter
    - ParameterType parameterType 
    - boolean identifying
      - ParameterType 에는 STRING, DATE, LONG, DOUBLE 을 지원한다. 
  - 각각의 JobParameter 들이 `BATCH_JOB_EXECUTION_PARAM` 에 저장됨. 

- JobParameters 는 Step 에서 참조가 가능하다. step 에서 tasklet 을 만들 떄 받는 파라미터 정보인 `StepContribution` 과 `ChunkContext` 에서. 
  - `StepContribution` -> `StepExecution` -> `JobExecution` -> `JobParameters` 가 있다. 
  - `ChunkContext` -> `StepContext` -> `StepExecution` 이 있다. 
