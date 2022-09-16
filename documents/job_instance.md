# Job Instance

## 기본 개념 

- Job 을 식별하는 단위. Job 이 실행될 때 생성됨. 

- Job 의 구성과 설정은 동일하지만 Job 이 실행될 때 처리하는 내용은 다를 수 있기 떄문에 잡의 실행을 구분해야함.

  - 예로 매일 실행되는 배치 Job 이 있다면 각각의 Job 들을 Job Instance 로 구별할 수 있다.

- Job Instance 의 생성 및 실행 

  - 처음 시작하는 Job + JobParameter 일 경우 새로운 JobInstance 를 생성한다. 
  - 이전과 동일한 Job + JobParameter 일 경우 이미 존재하는 JobInstance 를 리턴한다. 
  - Job 구별을 Job Parameter 와 Job 설정 정보로 하는듯.
  - Job 과는 1:N 관계로 알고 있으면 된다. 

- `BATCH_JOB_INSTANCE` 테이과의 매핑 
  - Job_name (job) 과 job_key (jobParameter 해쉬값) 과 동일한 데이터는 중복 저장이 안됨. 
  - 위의 예기와 똑같은 것. 

- `JobLauncher` 가 job 을 실행시키기 위해선 (= run()) job 과 JobParameter 가 필요하다.
  - Job 만 만드는게 아니라 JobParameter 도 만들어야한다. 
  - 이후에 `JobRepository` (이전에 배운 Job 의 메타데이터를 데이터베이스에 저장해주는 레파지토리.) 가 이 Job 이 처음 만들어지는 건지, 아니면 다시 실행하는건지 확인한다.
    - 이떄 job 과 jobParameter 를 이용한다. 
    - 이미 있는 것이라면 기존 JobInstance 를 가져온다. 
      - 이 이후는 Job 이 실행되지않고 예외를 던진다. (한번 실행되었으므로.)
    - 처음 실행되는거면 JobInstance 를 생성하고 저장한다.
      - `BATCH_JOB_INSTANCE` 와 매핑된다. 
      - JobParameter 정보는 `BATCH_JOB_EXECUTION_PARAMS` 에 저장된다. 

