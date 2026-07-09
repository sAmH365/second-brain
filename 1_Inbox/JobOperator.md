tag: #SpringBatch #JobOperator

* JobLauncher -> JobOperator 전환 배경 
	* **JobLauncher Deprecated:** Spring Batch 6.0부터 `JobLauncher` 인터페이스는 공식적으로 `@Deprecated` 되었으며, 향후 버전(6.2 이상)에서 삭제(Removal)될 예정임
	* **대체재:** 공식적으로 `JobOperator`를 대신 사용 권장
---
## 1. JobLauncher vs JobOperator 핵심 차이

### ~~JobLauncher (Deprecated 버전)~~
* **목적:** 배치의 단순 실행을 위한 저수준(Low-level) 인터페이스.
* **메서드:** `run(Job job, JobParameters jobParameters)`
* **한계:** * `Job` 객체와 `JobParameters` 객체를 직접 생성해서 넘겨야 하므로, 외부 API(Controller)나 HTTP 요청에서 파라미터를 받아 호출하기 까다로움.
  * 실행 중인 작업을 중지하거나 재시작하는 등의 '제어' 기능이 없음.
  * 동기(Synchronous)로 돌지, 비동기(Asynchronous)로 돌지 인터페이스 레벨에서 보장하지 않음.
### JobOperator (6.0+ 최신 버전)
* **목적:** 배치의 실행, 중지, 재시작, 모니터링을 총괄하는 Wrapper 인터페이스.
* **핵심 메서드:** 
  * `start(Job job, JobParameters jobParameters)`: 실행 (6.0부터 새 표준)
  * `stop(long executionId)`: 중지
  * `restart(JobExecution jobExecution)`: 재시작
* **특징:** 과거 5.x 버전까지 지원하던 문자열 기반(`String jobName`) 및 Properties 실행 방식은 파싱 에러 유발 문제로 인해 6.0부터 Deprecated 되었음.