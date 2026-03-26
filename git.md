---
title: git
---
---
git 
branch
```
# 지금 develop 브랜치에서 feature 브랜치 따기
git checkout develop
git checkout -b feature/android-base-setup

# 작업 후 커밋
git add .
git commit -m "chore: Android 프로젝트 초기 세팅 및 라이브러리 의존성 추가"

# develop에 merge
git checkout develop
git merge feature/android-base-setup
git push origin develop
```

```
git checkout main
git merge develop
git push origin main
git checkout develop
```

```
# develop 브랜치로 이동
git checkout develop

# 최신 상태로 업데이트
git pull origin develop

# feature 브랜치 생성 (내 컴퓨터에서만)
git checkout -b feature/login

# 깃허브 서버로 복사해서 만든 브랜치 전달
git push -u origin feature/login

# 브랜치 확인
git branch -a

# 편집모드 나가기 q 스페이스 다음페이지 엔터 한줄씩 더보기

# 서버 정보 업데이트
git fetch --all

```

`fetch`는 한마디로 **"GitHub에 무슨 새로운 소식(브랜치나 커밋)이 있는지 업데이트 확인만 하는 것"**
- **Commit (내 컴퓨터에 저장):** 게임으로 치면 **'세이브 포인트'**를 만드는 거예요. "지금까지 작업한 거 일단 내 컴퓨터에 박제!" 하는 거죠. 아직 인터넷(GitHub)에는 안 올라간 상태입니다.
    
- **Push (서버에 업로드):** 내 컴퓨터에 저장된 '세이브 파일'들을 **GitHub 서버로 전송**하는 겁니다. 이때 비로소 다른 사람도 내 코드를 볼 수 있게 됩니다.
- **`git fetch`**: 우체국(GitHub)에 가서 나한테 온 편지가 있는지 **목록만 확인**하고 오는 것. (편지를 읽지는 않음)
    
- **`git pull`**: 우체국에 가서 편지를 **가져와서 내 책상(로컬 코드)에 합쳐버리는 것.**

**순서:** 코드 수정 → `git add` (담기) → `git commit` (내 컴퓨터 저장) → `git push` (GitHub에 올리기)

# 정리
```
# 1. 새 브랜치 만들기 (내 위치 이동)
git checkout -b feature/login

# 2. 코드 열심히 수정하기 (VS Code 등에서 작업)

# 3. 상태 확인 (뭐 고쳤더라?)
git status

# 4. 바구니에 담고 내 컴퓨터에 저장
git add .
git commit -m "로그인 기능 UI 완성"

# 5. 드디어 GitHub에 올리기 (처음이면 -u 붙이기)
git push -u origin feature/login
```