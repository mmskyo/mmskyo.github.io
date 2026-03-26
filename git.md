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
