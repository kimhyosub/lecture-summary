# Git Flow 

 > Main Branch
  * master : merge 하는 브랜치
  * develop

 > Supporting Branch
  * **feature**
    * 추가 기능이 있는 경우에 생성
    * develop branch 을 기초로 만듬
    * 완료 후 develop branch 으로 merge
    * 완료 후에는 Branch 삭제
  * **release**
    * 배포 직전에 테스트 하기 위한 브랜치
    * README.md 등, 메타 데이터 수정
    * develop branch 기초로 만듦
    * 완료 후에는 master 와 develop 으로 merge
    * 완료 후에는 삭제
    * version 을 남김 (Tag)
  * **hotfix**
    * 배포 후 버그 발견 시 생성
    * master branch 를 기초로 만듦
    * 완료 후 master 와 develop 으로 merge
    * 완료 후 삭제
  * **Release and Hotfix**
    * 가끔 release branch 가 존재하고 있을 때, 배포판(master)에서 버그가 발견되어
     hotfix branch 가 필요할 때가 있다. 그렇게 되면 버그 수정 후 develop 대신에
     release 에 merge 해야 한다.
