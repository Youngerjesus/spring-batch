# Job Execution 

## 기본 개념 

- JobInstance 의 잡의 시도를 의미하는 객체. Job 실행 중에 발생한 정보들을 저장하고 있는 객체다. 

  - 시작시간, 종료시간, 상태(시작됨, 완료, 실패), 종료 상태의 속성을 가진다. 

- JobInstance 와의 관계
  - JobExecution 은 `FAILED` 또는 `COMPLETED` 등의 Job 실행 결과 상태를 가지고 있다.
  - 실행 상태 결과가 `COMPLETED` 면 완료되었으니 재실행이 불가능함. 
    - `JobInstanceAlreadyCompletedException` 이 발생한다. 
  - 반대로 `FAILED` 면 JobInstance 가 실행이 완료되지 않은 것으로 간주해서 재실행이 가능함.
    - JobParameter 가 동일한 값이더라도 재실행이 가능함. 
    - 이떄 또 JobExecution 은 생성된다. 다만 JobInstance 는 기존에 쓰던거 사용함. 
  - Completed 가 아니라면 JobInstance 는 여러 번의 시도를 하는게 가능함.
    - **다른 상태일때도 시도하는게 가능한가?** 

- BATCH_JOB_EXECUTION 테이블과의 매핑 
  - JobInstance 와 JobExecution 은 1:N 매핑 구조를 이루고있음. 

- JobExecution 이 가지는 데이터를 보자.
  - JobParameters 
  - JobInstance
  - ExecutionContext 
    - 실행하는 동안 유지해야하는 데이터 
  - BatchStatus
    - 실행 상태를 나타내는 Enum 클래스
    - `COMPLETED`, `STARTING`, `STARTED`, `STOPPING`, `STOPPED`, `FAILED`, `ABANDONED`, `UNKNOWN`
  - ExitStatus
    - 실행 결과를 나타내는 클래스
    - `UNKNOWN`, `EXECUTING`, `COMPLITED`, `NOOP`, `FAILED`, `STOPPED` 
  - startTime
    - Job 을 실행할 때 시스템 시간 
  - createTime 
    - JobExecution 이 처음 저장될 때 시스템 시간 
  - endTime
    - 성공 여부와 상관없이 실행이 종료되는 시간 
  - lastUpdated
    - JobExecution 이 마지막 저장될 때 시스템 시간
