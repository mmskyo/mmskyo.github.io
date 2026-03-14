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

import 할 때 경로 설정 제대로!\
from .core.config X
from app.core.config import settings X
만약 db에서 쓴다면
from core.config O 
why? app.을 붙이는건 자기 자신을 호출하는것!
.. 또한 마찬가지
같은 디렉토리에 이미 존재하기에 다른 폴더라도 붙일 필요가 없다. 그냥 폴더 이름만


class baseurl ? 그거 이메일 어쩌구 못한다그러면
pip install 'pydantic[email]'

fastapi dev app/main.py

   FastAPI   Starting development server 🚀

             Searching for package file structure from directories with
             __init__.py files
ERROR    Import error: No module named 'db'
WARNING  Ensure all the package directories have an __init__.py file

╭───────────────────── Traceback (most recent call last) ──────────────────────╮
│ /Users/mac/my_app_server/.venv/lib/python3.13/site-packages/fastapi_cli/cli. │
│ py:328 in dev                                                                │
│                                                                              │
│   325 │                                                                      │
│   326 │   Otherwise, it uses the first [bold]FastAPI[/bold] app found in the │
│   327 │   """                                                                │
│ ❱ 328 │   _run(                                                              │
│   329 │   │   path=path,                                                     │
│   330 │   │   host=host,                                                     │
│   331 │   │   port=port,                                                     │
│                                                                              │
│ /Users/mac/my_app_server/.venv/lib/python3.13/site-packages/fastapi_cli/cli. │
│ py:152 in _run                                                               │
│                                                                              │
│   149 │   │   try:                                                           │
│   150 │   │   │   # Resolve import data with priority: CLI path/app > config │
│   151 │   │   │   if path or app:                                            │
│ ❱ 152 │   │   │   │   import_data = get_import_data(path=path, app_name=app) │
│   153 │   │   │   elif config.entrypoint:                                    │
│   154 │   │   │   │   import_data = get_import_data_from_import_string(confi │
│   155 │   │   │   else:                                                      │
│                                                                              │
│ /Users/mac/my_app_server/.venv/lib/python3.13/site-packages/fastapi_cli/disc │
│ over.py:125 in get_import_data                                               │
│                                                                              │
│   122 │   │   raise FastAPICLIException(f"Path does not exist {path}")       │
│   123 │   mod_data = get_module_data_from_path(path)                         │
│   124 │   sys.path.insert(0, str(mod_data.extra_sys_path))                   │
│ ❱ 125 │   use_app_name = get_app_name(mod_data=mod_data, app_name=app_name)  │
│   126 │                                                                      │
│   127 │   import_string = f"{mod_data.module_import_str}:{use_app_name}"     │
│   128                                                                        │
│                                                                              │
│ /Users/mac/my_app_server/.venv/lib/python3.13/site-packages/fastapi_cli/disc │
│ over.py:69 in get_app_name                                                   │
│                                                                              │
│    66                                                                        │
│    67 def get_app_name(*, mod_data: ModuleData, app_name: str | None = None) │
│    68 │   try:                                                               │
│ ❱  69 │   │   mod = importlib.import_module(mod_data.module_import_str)      │
│    70 │   except (ImportError, ValueError) as e:                             │
│    71 │   │   logger.error(f"Import error: {e}")                             │
│    72 │   │   logger.warning(                                                │
│                                                                              │
│ /opt/anaconda3/envs/fastapi-env/lib/python3.13/importlib/__init__.py:88 in   │
│ import_module                                                                │
│                                                                              │
│    85 │   │   │   if character != '.':                                       │
│    86 │   │   │   │   break                                                  │
│    87 │   │   │   level += 1                                                 │
│ ❱  88 │   return _bootstrap._gcd_import(name[level:], package, level)        │
│    89                                                                        │
│    90                                                                        │
│    91 _RELOADING = {}                                                        │
│ in _gcd_import:1387                                                          │
│ in _find_and_load:1360                                                       │
│ in _find_and_load_unlocked:1331                                              │
│ in _load_unlocked:935                                                        │
│ in exec_module:1023                                                          │
│ in _call_with_frames_removed:488                                             │
│                                                                              │
│ /Users/mac/my_app_server/app/main.py:3 in <module>                           │
│                                                                              │
│    1 from fastapi import FastAPI                                             │
│    2 from fastapi.middleware.cors import CORSMiddleware                      │
│ ❱  3 from db import session as db                                            │
│    4 from models import user, scan                                           │
│    5 from api.v1 import auth, scan as scan_router                            │
│    6 from core.config import settings                                        │
╰──────────────────────────────────────────────────────────────────────────────╯
ModuleNotFoundError: No module named 'db'