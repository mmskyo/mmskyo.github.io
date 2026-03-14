---
title: cicd
nav_order: 3
parent: Projects
---
---
클라우드 아키텍처에서 CI/CD 파이프라인을 중앙에서 독립적으로 구축하는 건 대규모 팀이나 여러 서비스를 관리할 때 흔한 패턴입니다. Android 앱과 FastAPI 서버가 별도 리포지토리라도 중앙 파이프라인에서 각각 트리거되도록 설계하면 됩니다.
# 중앙 CI/CD
중앙 CI/CD 아키텍처 개요
중앙 파이프라인은 GitHub Actions, GitLab CI, Jenkins 등을 메인 파이프라인으로 두고 각 리포지토리의 이벤트를 받아 처리합니다.
```
GitHub (Android Repo) ─┐
                       ├─ Webhook → Central Pipeline (GitHub Actions/GitLab)
GitHub (FastAPI Repo) ─┘
         ↓
    Build → Test → Deploy (각각 독립)
```

# Github Actions (Repository Dispatch)로 구현
각 리포지토리에 webhook 트리거 추가하고 중앙 리포지토리에서 처리:
android-repo/.github/workflows/trigger.yml:
yaml
```
name: Trigger Central CI
on: [push]
jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
    - name: Trigger Central
      uses: actions/github-script@v7
      with:
        script: |
          await github.rest.repos.createDispatchEvent({
            owner: 'your-central-org',
            repo: 'ci-cd-pipeline',
            event_type: 'android-push',
            client_payload: { ref: '${{ github.ref }}', repo: '${{ github.repository }}' }
          })
```
중앙 리포
ci-cd-pipeline/.github/workflows/main.yml:
```
on:
  repository_dispatch:
    types: [android-push, fastapi-push]
jobs:
  android-ci:
    if: github.event.action == 'android-push'
    runs-on: ubuntu-latest
    steps:
    - checkout the android repo via API
    - build/test/deploy Android

```