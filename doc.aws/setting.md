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