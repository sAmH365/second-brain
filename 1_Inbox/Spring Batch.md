tag: #SpringBatch 
### JOB (작업)
* 정의: 배치 처리 과정 전체를 하나의 단위로 묶은 최상위 개념
* Job의 구성요소:
	* [[#JobInstance (작업 인스턴스)]]
	* [[#JobParameters (작업 매개변수)]]
	* [[#JobExecution / StepExecution (실행 상태)|JobExecution (실행 상태)]]
	* [[#Step (단계)]]
* Job을 빌드하는 방법:
	* **JobBuilder 사용**: `new JobBuilder("작업명", jobRepository)`를 통해 직접 객체 생성 (`JobBuilderFactory`는 폐지됨 *(SpringBatch5.0 이후)* )
    * **Step 연결**: 생성된 빌더에 `.start(stepA).next(stepB).build()` 형태로 단계를 순차적으로 연결하여 구성
### Step (단계)
* 정의: Job을 구성하는 독립적인 작업단위
* Job은 N개의 Step을 가질 수 있음
* Step의 흐름 제어 방식: ?
### Tasklet vs Chunk (작업 처리 방식)
* 정의: Step을 실행하는 두 가지 핵심 스타일
### JobInstance (작업 인스턴스)
* 정의: Job이 실제 실행될 때 논리적으로 생성되는 고유한 실행 객체
* 동일한 jobName + 동일한 JobParameter = **~={green}JobInstance=~**
### JobExecution / StepExecution (실행 상태)
* 정의: Job과 Step이 실제로 실행된 '시도(Attempt)와 결과 상태를 담은 객체'
### JobParameters (작업 매개변수)
* 정의: 배치 실행할 때 외부에서 주입하는 파라미터 (타입: String, Long, Double, Date)
* ~={yellow}**JobInstance를 구분하는 기준이 됨**=~
### JobRepository (저장소)
* 정의: 배치가 실행되는 모든 메타데이터 (Job 상태, 성공 여부, 실행 시간 등)를 DB에 저장하고 관리하는 관리자역할
* 키워드: `BATCH_JOB_INSTANCE`, `BATCH_JOB_EXECUTION`
### [[JobOperator|JobOperator]] & [[JobOperator|JobLauncher]](실행기)
* 정의: 배치를 구동하고 제어하는 실행 인터페이스
### ItemReader ➔ ItemProcessor ➔ ItemWriter (Chunk 지향)
* 정의: Chnk지향 Step에서 데이터를 가공하는 3단계 핵심 컴포넌트
### ExecutionContext(공유 저장소)
* 정의: 배치가 도는 동안 상태를 유지하고, 다른 Step이나 컴포넌트끼리 데이터를 공유하기 위해 사용하는 키-값 저장소