# Step

## 기본 개념 

- job 에서 처리하는 하나의 독립적인 단계를 말함. 실제 배치 처리를 정의하는 부분

  - 입력과 처리 출력에 관한 비즈니스 로직 설정을 모두 포함해야함.
  - Job 의 세부 설정을 하는 부분
  - 모든 Job 은 하나 이상의 Step 을 포함하고 있음. 
  - step 간에는 간섭이 없다.

- step 은 `StepExecution` 을 인자로 받아서 실행한다. `execute()` 메소드를 통해서.

- AbstractStep 에서 포함하는 정보는 다음과 같다.
  - name: Step 의 이름  
  - startLimit: Step 실행 횟수 제한 
  - allowStartIfComplete: Step 이 완료된 후에도 재실행 여부 
  - stepExecutionListener: Step 이벤트 리스터 
  - JobRepository: Step 메타 저장

## 기본 구현체 

- TaskletStep 
  - 가장 기본이 되는 클래스로서 Tasklet 의 구현체들을 제어한다.
    - TaskletStep 이 실행의 시작점이 된다. (Step 들을 하나하나씩.)
  - steps 와 tasklet 의 정보를 가지고 있다.
    - tasklet 에 ItemReader, ItemProcessor, ItemWriter 가 있다.
- PartitionStep
  - 멀티스레드 방식으로 각 Step 을 여러 개로 분리해서 처리하는게 가능하다.
  - StepExecutionSplitter 와 PartitionHandler 라는 정보를 가지고 있다. 
- JobStep 
  - Step 내에서 Job 을 실행하도록 한다.
  - Job 과 JobLauncher 를 정보로 가지고 있다. 
- FlowStep 
  - Step 내에서 Flow 를 실행하도록 한다. 
  - Flow 정보를 가지고 있다.

## Step 설정법 

### 직접 생성한 Tasklet 이용 

```java
StepBuilderFactory.get("step1")
        .tasklet(myTasklet)
        .build()
```

### Chunk Oriented Tasklet 이용 

```java
StepBuilderFactory.get("step1")
        .chunk(100)
        .reader(myReader)
        .writer(myWriter)
        .build()
```

- 이건 직접 생성한 Tasklet 을 이용하는 건 아님. 스프링 배치가 만든 `ChunkOrientedTasklet` 을 실행하는 것.
  - 여기에 reader, writer, processor 를 등롣시키는 것.

### JobStep 이용 

```java
StepBuilderFactory.get("step1")
        .job(myJob)
        .launcher(myLauncher)
        .parameterExtractor(myJobParameterExtractor)
        .build()
```

- step 에서 job 을 실행하는 방법의 step 이다.

### FlowStep 이용 

```java
StepBuilderFactory.get("step1")
        .flow(myFlow)
        .build()
```

- step 에서 flow 를 실행하는 방법의 step 이다.

## Step 이 만들어지는 과정 

1) stepBuilderFactory.tasklet(myTasklet)

- 내부를 보면 TaskletStepBuilder 에 tasklet 을 추가한다. 

2) stepBuilderFactory.build() 를 통해서 taskletStep 을 만든다.

- taskletStep 이 step 의 구현체. tasklet 을 내부적으로 가지고 있고 tasklet 을 실행함. 

3) Job 을 만들 때 Job 안에 이렇게 만들어진 step 들을 등록한다. 

- jobBuilderFactory.build() 에서 steps 를 job 안에 등록함. (여기 들어가는 steps 가 taskletStep 임.)

4) Spring boot 가 Job 을 실행할 때 JobLauncher 에서 실행한다.

- 이게 SimpleJobLauncher.run() 메소드.
  - 내부를 보면 JobRepository.getLastJobExecution() 을 통해서 JobExecution() 을 가져와서 시작함. 
  - job.execute(jobExecution) 을 통해서 잡을 시작.
  - job 이 실행되는 걸 보면 내부에 있는 step 들을 가져와서 하나씩 실행한다. step 의 실행은 stepHandler 에 의해서 이뤄지고.
