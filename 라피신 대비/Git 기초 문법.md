# Git 기초 문법

- commit을 할때 기록되는 사용자명과 이메일 설정

```bash
git config --global user.name "user name" // 유저 설정
git config --global user.email "user@email" // 이메일 설정
git config --list // 유저명, 이메일명 확인
```

- 컴퓨터에 로컬 저장소 생성

```bash
mkdir [working directory adress] // []경로에 작업폴더생성
```

- 로컬 저장소로 이동

```bash
cd [working directory adress] // 작업폴더로 이동
```

- 원격 저장소 복사

```bash
git clone [Github repository adress] [directory name] // 원격저장소를 이름을 정해 복사
```

- 로컬 저장소에 변경내용 추가하기

```bash
git add [folder/file] // 파일 선택하여 추가
git add . // 현재위치의 모든파일 선택
```

- 로컬 저장소에 변경내용 확정하기

```bash
git commit -m "explain for code" // 커밋하면서 메시지 추가
```

- 원격 저장소로 변경된 로컬 저장소 내용 발행하기

```bash
git push origin master // origin저장소에 master브랜치에 추가 (clone시 origin저장소 자동 생성)
```

- 로컬 저장소를 원격 저장소 파일로 갱신하기

```bash
git pull // 원격저장소의 파일들을 다운로드
```

- git 상태 확인

```bash
git status
```

- git 기록확인

```bash
git log 
git log -[n] // n자리에 숫자입력, 최근 n개의 커밋 확인
```

reference
