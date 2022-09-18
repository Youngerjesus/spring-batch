# StepContribution

## 기본 개념 

- 청크 프로세스의 변경 사항을 버퍼링해서 StepExecution 의 상태를 변경하기 위한 객체. 
- 청크 커밋 직전에 StepExecution 의 apply 메소드를 호출해서 상태를 업데이트한다. 
- ExitStatus 의 기본 종료코드외 사용자 정의 종료코드를 지정하는 것도 가능.

## StepContribution 이 가지고 있는 데이터  

- readCount
- writeCount
- filterCount
- parentSkipCount: 부모 클래스인 StepExecution 에서 발생한 skip 횟수 
- readSkipCount: read 에 실패해서 스킵한 횟수 
- writeSkipCount
- processSkipCount
- ExitStatus
- stepExecution 

## 처리 진행과정 

1) TaskletStep 이 Step 마다 StepExecution 을 만든다. (create())

- Step 안에 TaskletStep 이 있다. 실행을 위해서 StepExecution 을 만드는 거임.

2) StepExecution 은 내부적으로 StepContribution 을 만든다. (create())
- TaskletStep.doInTransaction() 이라는 트랜잭션을 시작하는 코드를 보면 `createStepContribution` 를 통해서 만든다.
  - doInTransaction 은 청크 기반의 처리를 하는 시점을 말한다.
    - chunkTransactionCallback() 에서 호출함.
    - 내부에 tasklet.execute() 메소드가 있음. 
    - 그리고 stepExecution.apply(contribution) 메소드가 있음. 여기에서 실행하고 contribution 으로 업데이트함. 
- StepExecution 내부에 이 메소드가 이미 있다.

3) TaskletStep 은 tasklet 을 실행할 떄 contribution 과 chunkContext 를 전달해서 실행한다. (execute()) 

- 스프링에서 제공해주는 Tasklet 은 ChunkOrientedTasklet 이다.

4) 처리는 내부적으로 ItemReader, ItemProcessor, ItemWriter 에의해서 이뤄진다.

- Contribution 은 Reader, Processor, Write 에서 생긴 정보들 (ex. read, readSkipCount, filterCount, processSkipCount) 등을 기록한다 
  - 기록은 각각의 시점마다 기록함.
  - 이렇게 기록하는걸 버퍼링이라고한다.

## Review 

- StepContribution 이 어떤 역할인지, 어떤 과정으로 contribution 이 업데이트되는지 확인가능.
