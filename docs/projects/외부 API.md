---
title: exapi
nav_order: 7
parent: Projects
---
---
whoisapi 등 외부 서비스 연동

```
qr-safe-scanner/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── security.py
│   │   └── cache.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   └── scan.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── scan.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── scan.py
│   └── db/
│       ├── __init__.py
│       └── session.py
├── requirements.txt
├── .env
└── .gitignore
```
