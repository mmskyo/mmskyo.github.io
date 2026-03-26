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

# 편집모드 나가기 q 스페이스 다음페이지 

```

