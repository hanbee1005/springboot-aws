# AWS 서버 환경 구축

## 사전 작업
- AWS 가입하기

## EC2 설정
1. EC2 인스턴스 생성 (리전 - 서울)
    - Amazon Linux AMI 2
    - t2.micro 유형
    - 기본 vpc, 서브넷 사용
    - 스토리지 크기 30GB
    - 태그 Name 추가
    - 보안 그룹 생성 (SSH 내 ip, 8080 포트 추가, HTTPS 추가)
    - 새 키 페어 생성
    
2. Elastic IP (탄력적 IP)
    - 새 주소 할당
    - 위에서 생성한 인스턴스와 주소 연결
    - 생성 후 인스턴스를 연결하지 않으면 과금
    
3. EC2 접속하기 (mac 기준)
    - 다운받은 .pem 파일을 ~/.ssh/로 복사
        + 만약 .ssh 디렉토리가 없다면,
            * cd ~
            * mkdir .ssh; chmod 700 ~/.ssh
        + cp pem_파일을_내려받은_위치.pem ~/.ssh 명령어로 복사
    - 복사된 파일 확인
        + cd ~/.ssh
        + ll | grep freelec
    - pem 키 권한 변경
        + chmod 600 ~/.ssh/pem키이름
    - config 파일 생성
        + vim ~/.ssh/config
          ```
          # 주석
          Host 본인이 원라는 서비스명
              HostName ec2의 탄력적 ip 주소
              User ec2-user
              IdentityFile ~/.ssh/pem키 이름
          ```
        + :wq로 vim 파일 저장 후 종료
        + config 파일 실행 권한 설정
            * chmod 700 ~/.ssh/config
    - ec2 실행하기
        + ssh config에 등록한 서비스명
        + 질문에 yes
   
4. 서버 사전 준비
   - Java 11 설치
      + Amazon Linux AMI 2에서 먼저 java -version으로 확인
      + yum list java*jdk-devel 명령어로 설치 가능한 버전 확인
      + java 11이 없는 경우, sudo yum install java-11-amazon-corretto-headless 로 설치 후 버전 확인
      + 참고: https://docs.aws.amazon.com/ko_kr/corretto/latest/corretto-11-ug/amazon-linux-install.html
   - 타임존 변경
      + sudo rm /etc/localtime
      + sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
      + date 명령어로 제대로 변경되었는지 확인
   - Hostname 변경
      + sudo vim /etc/sysconfig/network
      + HOSTNAME=본인이_원하는_서버명
      + sudo reboot 후 확인
      + sudo vim /etc/hosts
      + 127.0.0.1 위에서_지정한_서버명
      + curl 위에서_지정한_서버명 명령어로 호스트명 등록 확인
    
## RDS 설정
1. RDS Maria DB 인스턴스 생성
   
2. 운영환경에 맞는 파라미터 설정
    - 파라미터 그룹 새로 생성 후 아래 파라미터를 찾아 수정
        + 타임존   
            * time_zone 을 Asia/Seoul 로 수정
        + Character Set
            * character_ 로 시작하는 항목 6개를 utf8mb4로 수정
            * collation_ 로 시작하는 항목 2개를 utf8mb4_general_ci로 수정
            * utf8은 이모지 저장이 안되고 utf8mb4는 이모지 저장이 가능
        + Max Connection
            * max_connections 150으로 수정
    
3. 새로 생성한 파라미터 그룹을 DB 인스턴스에 적용

4. 로컬 PC에서 접속하기
    - RDS 보안그룹 인바운드 규칙으로 다음 2가지 추가
        + EC2 보안그룹
        + 내 IP
    - DB 플러그인 설치 및 파라미터 설정 확인
    
5. EC2에서 RDS 접근 확인
    - ssh ec2명
    - sudo yum install mysql
    - mysql -u 계정 -p -h Host주소
    - show databases; 로 생성한 데이터베이스 확인
    
## EC2 서버에 프로젝트 배포
1. EC2에 Git 설치
    - sudo yum install git 
    - git --version 으로 설치 확인
    - mkdir ~/app && mkdir ~/app/step1 로 프로젝트를 저장할 디렉토리 생성
    - cd ~/app/step1 디렉토리 이동
    - git clone 깃허브_레포지토리_주소
    - cd 프로젝트명
    - ./gradlew test 로 테스트가 잘 실행되는지 확인
    
2. 배포 스크립트 생성
    - vim ~/app/step1/deploy.sh 파일을 생성
    - git pull 받고 build 해서 생성된 jar 파일 실행하는 것까지 스크립트로 구현
    - chmod +x ./deploy.sh 로 실행 권한 추가
    - ./deploy.sh 로 실행
    - vim nohup.out 으로 실행된 애플리케이션에서 출력된 내용을 확인 => 에러!
    
3. 외부 Security 파일 등록하기
    - vim /home/ec2-user/app/application-oauth.yml 파일 생성
    - 로컬에 있는 application-oauth.yml 파일 내용 복사 후 붙여넣기
    - deploy.sh 수정 후 다시 실행
    
4. 스프링 부트 프로젝트로 RDS 접근
    - RDS에 테이블 생성 (posts, user, 스프링 세션 테이블)
    - build.gradle 파일에 mariadb 의존성 추가
    - src/main/resources/ 에 application-real.yml 파일 생성 후 RDS 환경 profile 설정 추가
    - EC2로 돌아와서
    - vim ~/app/application-real-db.yml 파일 생성 후 DB 접속을 위한 연결 정보 저장
    - deploy.sh 수정 후 실행
    - curl localhost:8080 실행 시 html 코드가 나오면 정상
    
5. EC2에서 소셜 로그인하기
    - AWS EC2 관련 보안 그룹 확인 (8080 포트가 오픈되어 있어야 함)
    - AWS EC2의 퍼블릭 DNS 주소의 8080 포트로 접속되는지 확인
    - 구글 및 네이버 승인 도메인으로 해당 EC2 도메인 주소 추가