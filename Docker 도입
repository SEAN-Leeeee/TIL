
Spring Boot 프로젝트에 Docker를 도입했다.
나중에 어떻게 할지 모르지만 다른 개발자와 협업할 수도 있으니 환경을 고정해두고싶었다.

서버는 Docker로 띄우고 DB는 로컬에서 돌리면 환경이 달라지는 문제가 발생하니까 DB를 붙이는 지금 도입하기로 했다.

도입 과정에서 다음과 같은 두 파일을 작성했다:
	•	Dockerfile: 애플리케이션을 빌드하여 .jar 파일을 만들고, 해당 파일로 서버를 실행하는 명령어를 정의한 파일
	•	docker-compose.yml: Spring Boot와 MySQL 등 필요한 컨테이너의 설정을 정의한 파일
이와 함께, application.yml 파일에도 MySQL 연동을 위한 설정을 추가했다.

---- 

! 문제 발생: 컨테이너 실행 중 Spring Boot가 즉시 종료됨
  - 컨테이너를 실행하고 빌드하는 과정에서 다음과 같은 오류가 발생했다 -> "Public Key Retrieval is not allowed"

이 오류의 원인은, MySQL 서버의 인증 방식이 공개키 기반인데, Spring Boot 측에서 allowPublicKeyRetrieval=true 옵션을 지정하지 않아 공개키를 받아오지 못하고 접속에 실패한 것이었다.
즉, 애플리케이션이 DB에 연결하지 못해 실행 중단된 것이다. 
당황한 점은 MySQL은 살아있는데 Spring Boot는 꺼져 있었다는 것이다.
docker ps 명령어로 실행 중인 컨테이너를 확인했을 때, MySQL은 살아 있었고 Spring Boot 컨테이너는 보이지 않았다. 이후 docker ps -a로 확인하니, Spring Boot는 실행 직후 종료되어 있었음을 확인할 수 있었다.

? 컨테이너 내부 동작 원리 분석
  문제의 본질은 Docker의 내부 동작을 정확히 이해하지 못한 데 있었다.
	•	Spring Boot 컨테이너는 실행 중 오류가 발생하면 즉시 종료된다. 단일 애플리케이션을 실행하는 짧은 생명 주기의 프로세스형 컨테이너다.
	•	반면 MySQL 컨테이너는 데몬 프로세스로 동작하며, 외부 접속이 실패해도 계속 살아 있는 구조다.
	•	또한 docker-compose는 컨테이너들을 동시에 실행시키지만, Spring Boot는 내부적으로 MySQL 접속이 실패하면 애플리케이션 자체가 터지기 때문에 종료된다. 
    MySQL 컨테이너에는 아무 문제가 없기 때문에 계속 정상 작동 상태로 유지된다.


^__^ 얻은 점 
	•	컨테이너 실행과 내부 애플리케이션 성공 여부는 다르다.
	•	Spring Boot가 터졌다고 해서 MySQL이 죽는 것은 아니다.
	•	애플리케이션 연결 옵션(allowPublicKeyRetrieval=true)은 MySQL의 기본 인증 방식을 이해하고 정확히 맞춰야 한다.

