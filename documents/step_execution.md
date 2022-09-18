# StepExecution 

## 기본 개념 

- Step 에 대한 처리 시도를 하는 객체. Step 실행에 필요한 정보들을 가지고 있다. 
  - Step 처리가 실행될 때 마다 생성되는 객체다.
  - Job 이 재실행되더라도 이미 성공한 Step 은 실행하지 않음. 즉 실패한 Step 만 재실행 됨.
- 이전 단꼐 Step 이 실패하면 현재 단계의 Step 은 실행하지 않음. 그러므로 StepExecution 은 생성되지 않음.
- Step 들이 모두 성공해야지 JobExecution 도 성공된다. 
  - 즉 StepExecution 이 모두 성공해야함. 하나라도 실패하면 안됨.

## BATCH_STEP_EXECUTION 테이블과의 매핑 

- JobExecution 과 StepExecution 는 1:M 매핑이다. 

- 하나의 Job 에 여러개의 Step 으로 구성했을 경우 각 StepExecution 의 부모가 JobExecution 이다.

## StepExecution 이 가지는 데이터

- jobExecution 
- stepName
- BatchStatus: 실행 상태를 나타냄. enum 형식. (COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN)
- readCount: 성공적으로 read 한 아이템 수 
- writeCount: 성공적으로 write 한 아이템 수 
- commitCount: 실행 중에 커밋된 트랜잭션 수 
  - 트랜잭션이 뭔지 궁금해서 보니 `Chunk-oriented Processing` 에서 본 내용에 따르면 reader 에서 read 한 걸로 chunk 를 만들고, 이 chunk 들이 writer 에서 쓰여지는 것을 트랜잭션이라고 하는듯.
  - 청크단위의 처리를 트랜잭션이라고 한다.
- rollbackCount: 트랜잭션 중 롤백된 카운트 수 
- readSkipCount: read 에 실패해서 스킵된 횟수 
- processSkipCount: process 에 실패해서 스킵된 횟수 
- writeSkipCount: write 에 실패해서 스킵된 횟수
- filterCount: ItemProcessor 에서 필터링된 아이템 수 
- startTime: Job 을 실행할 때의 시스템 시간 
- endTime: 성공 여부와 상관없이 실행이 종료되는 시간 
- lastUpdated: JobExecution 이 마지막으로 저장될 떄의 시스템 시간
- ExecutionContext: 실행동안 유지해야하는 데이터 
- ExitStatus: 실행 결과를 나타내는 상태. 종료코드 포함 (UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED)
- failureExceptions: Job 실행 중 발생한 예외 리스트  
