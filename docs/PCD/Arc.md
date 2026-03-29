---
title: architecture
nav_order: 3
parent: PCD
published: "false"
---
---
구현 순서

Fragment ( 화면 ) --- 사용자 입력 받음 ---> ViewModel (두뇌) --- 로직 처리 ---> Repository (창고지기) --- 서버 또는 DB 요청 ---> ApiService / Dao --- 실제 데이터 ---> 다시 ViewModel로 결과 전달 ---> Fragment에서 화면 업데이트