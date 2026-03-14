---
title: cicd
nav_order: 3
parent: Projects
---
---

# 중앙 CI/CD
중앙 CI/CD 아키텍처 개요
중앙 파이프라인은 GitHub Actions, GitLab CI, Jenkins 등을 메인 파이프라인으로 두고 각 리포지토리의 이벤트를 받아 처리합니다.

`GitHub (Android Repo) ─┐`
                       `├─ Webhook → Central Pipeline (GitHub Actions/GitLab)`
`GitHub (FastAPI Repo) ─┘`
         `↓`
    `Build → Test → Deploy (각각 독립)`

