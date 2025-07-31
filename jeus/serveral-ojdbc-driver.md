# JEUS OJDBC 여러 버전 사용  

- 이슈사항
OS는 AIX7.2 이고 JEUS8 버전을 사용 중이고, jdk1.8 버전을 사용 중인데 오라클 DB의 CLOB 처리 문제로 ojdbc5.jar 사용 중
신규 프로젝트 진행시 동일 서버에 JEUS 컨테이너 추가가 필요하고, 업체의 소스 포팅시 ojdbc5.jar 사용으로 isclosed() 관련 AbstractMethodError 발생  
이에 ojdbc5버전과 ojdbc8버전에 대한 동시 사용 가능한지에 대한 문의를 진행했고, 티맥스 측에서는 사용은 가능하지만 많은 사용은 하지 않고 있다고 답변


- 테스트#1
기존의 $JEUS_HOME/lib/datasource 에 기본으로 로드되는 ojdbc5.jar 는 그대로 두고, 배포될 소스 디렉토리(예: /tmp/source/test/WEB-INF/lib/ojdbc8.jar)에 ojdbc8.jar 파일을 둠.
신규 프로젝트의 소스가 배포될 신규 컨테이너 생성
신규 컨테이너의 Boot Classloader Append Class Path 에 ojdbc8.jar 디렉토리를 명시(/tmp/source/test/WEB-INF/lib/ojdbc8.jar)
JEUS 컨테이너 기동시 /tmp/source/test/WEB-INF/lib/ojdbc8.jar 가 로드가 잘 된 것을 확인


- 테스트#2
ojdbc8.jar가 잘 로드된 것은 확인했으나 실제로 데이터소스 생성후 컨테이너에서 사용시 ojdbc8.jar를 사용하는지 ojdbc5.jar를 사용하는지 알 수 없음.
데이터소스 생성 후 컨테이너 쪽에 타겟팅 진행
실제로 해당 데이터소스를 ojdbc8.jar를 통해 사용하는지 확인하기 위해 테스트 용도 소스를 작성함.
/tmp/source/test123/WEB-INF/web.xml , /tmp/source/test123/dbtest.jsp 파일을 만들어서 신규 컨테이너에 배포. -> 챗 gpt 통해 간단하게 작성 가능
배포 후에 웹페이지 호출시 데이터소스가 타겟 DB와 연결이 된것을 확인했고(jeusadmin -> connection-pool-info) 정상 호출된 것을 확인
위 테스트 이후 정상적으로 컨테이너 사용이 가능한것과 ojdbc드라이버 통한 DB 연결됨을 확인했으나 ojdbc8.jar로 접속했는지 여부가 불투명
다시 테스트 진행시 기존 $JEUS_HOME/lib/datasource/ojdbc5.jar 파일의 파일명을 ojdbc5.ja_ 로 변경 후에 JEUS 컨테이너 재기동 -> 정상적으로 호출 됨을 확인
또다른 테스트로 ojdbc5.ja_ 변경 및 ojdbc8.ja_ 로 변경후 기동시 페이지 오류 발생하는것을 확인


위 테스트 결과로 기존 ojdbc5.jar로 기동된 컨테이너에는 영향을 미치지 않고, 신규 컨테이너로 분리 구성시 다른 버전의 ojdbc 드라이버를 사용 가능함을 확인.

