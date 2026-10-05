# Docker · CI/CD 세미나 사전 준비

세미나 전까지 아래 준비와 검증을 완료해주세요. 설치 과정에서 재부팅이 필요할 수 있으니 당일에 시작하지 말고 미리 준비해주세요.

이번 실습에서는 작은 Python FastAPI 앱을 Docker 컨테이너로 실행하고 GitHub Actions로 테스트와 이미지 빌드를 자동화합니다. 외부 서버, AWS 계정, 유료 서비스 가입은 필요하지 않습니다.

## 1. 준비할 것

| 준비물 | 용도 | 완료 기준 |
| --- | --- | --- |
| 개인 노트북과 충전기 | 실습 | 인터넷 연결 가능 |
| Docker Desktop | 컨테이너 실행, Compose | `hello-world` 실행 성공 |
| Python 3.12 | 로컬 API 실행과 테스트 | Python 3.12.x 출력 |
| Git | 코드 업로드 | `git --version` 성공 |
| GitHub 계정 | GitHub Actions 사용 | 웹 로그인 가능 |
| 코드 편집기 | 파일 작성 | 프로젝트 폴더 열기 가능 |

편집기는 VS Code를 권장합니다. IntelliJ 설치는 필요하지 않습니다. 기존 편집기를 사용해도 괜찮으며, Python 코드 실행은 터미널에서 진행합니다.

## 2. Docker Desktop 설치

### Windows

1. [Docker 공식 Windows 설치 안내](https://docs.docker.com/desktop/setup/install/windows-install/)를 열어 PC에 맞는 설치 파일을 받습니다. 일반적인 Intel/AMD PC는 x86_64 버전입니다.
2. 안내의 지원 OS와 시스템 요구사항을 확인합니다.
3. WSL 2를 사용하는 방식으로 설치합니다. 이 세미나는 Linux 컨테이너를 사용합니다.
4. WSL이 없거나 업데이트가 필요하면 관리자 PowerShell에서 필요한 명령을 실행합니다.

```powershell
wsl --version
```

WSL이 설치되어 있지 않은 경우:

```powershell
wsl --install
```

이미 설치되어 있지만 업데이트가 필요한 경우:

```powershell
wsl --update
```

5. 재부팅 안내가 나오면 재부팅합니다.
6. Docker Desktop을 직접 열어 초기 설정을 완료하고 Engine이 실행될 때까지 기다립니다.

WSL 설치가 실패하거나 가상화 관련 오류가 나오면 메시지를 진행자에게 알려주세요. PC의 가상화 설정을 추가로 확인해야 할 수 있습니다.

### macOS

1. Apple 메뉴 → 이 Mac에 관하여에서 칩을 확인합니다.
2. [Docker 공식 Mac 설치 안내](https://docs.docker.com/desktop/setup/install/mac-install/)에서 맞는 설치 파일을 받습니다.
   - M1/M2/M3 등 Apple 칩: Apple Silicon
   - Intel 프로세서: Intel
3. 다운로드한 파일을 열고 Docker를 Applications 폴더에 넣습니다.
4. Docker 앱을 열고 초기 설정을 완료한 뒤 Engine이 실행될 때까지 기다립니다.

Docker Desktop에는 Docker Compose가 포함되어 있어 별도 Compose 설치가 필요하지 않습니다. [공식 Compose 설치 안내](https://docs.docker.com/compose/install/)

### Linux를 사용하는 경우

[Docker Engine 설치 안내](https://docs.docker.com/engine/install/)와 [Compose 플러그인 설치 안내](https://docs.docker.com/compose/install/linux/)를 따라 준비합니다. 아래 검증 명령이 정상 실행되는 상태로 준비해주세요. 권한 오류가 나면 진행자에게 미리 알려주세요.

## 3. Docker가 제대로 설치되었는지 확인

**Docker Desktop을 켠 상태에서** Windows는 PowerShell, Mac은 터미널을 열고 아래 명령을 하나씩 실행합니다.

```bash
docker --version
docker version
docker info
docker compose version
docker run --rm hello-world
```

| 명령 | 성공 기준 |
| --- | --- |
| `docker --version` | Docker 버전 출력. 이것만으로 실행 준비가 끝난 것은 아닙니다. |
| `docker version` | Client와 Server 정보가 모두 출력 |
| `docker info` | 서버 정보 출력, daemon 연결 오류 없음 |
| `docker compose version` | Docker Compose v2 계열 버전 출력 |
| `docker run --rm hello-world` | `Hello from Docker!` 출력 |

첫 실행은 이미지를 다운로드하므로 인터넷 연결이 필요합니다. `hello-world` 컨테이너는 메시지를 출력한 뒤 종료하는 것이 정상입니다.

### 브라우저 접속까지 확인하기

`hello-world` 성공 후, 포트 연결도 확인합니다.

```bash
docker run --rm -d --name seminar-docker-check -p 18080:80 nginx:alpine
```

브라우저에서 **http://localhost:18080**을 열어 `Welcome to nginx!`가 보이는지 확인합니다.

```bash
docker ps
docker logs seminar-docker-check
docker stop seminar-docker-check
```

`docker ps`에서 컨테이너를 확인하고, 확인이 끝나면 `stop`으로 정리합니다. `--rm` 옵션으로 종료된 검증용 컨테이너가 자동 삭제됩니다.

18080번 포트를 이미 쓰고 있다면 `-p 18081:80`으로 바꾸고 http://localhost:18081 에 접속합니다.

## 4. Python, Git, 편집기 준비

- Python: [공식 다운로드](https://www.python.org/downloads/)에서 Python 3.12.x 설치 파일을 선택합니다.
- Windows에서는 설치 중 Python을 PATH에 추가하는 옵션을 선택합니다. 설치 후 터미널을 새로 엽니다.
- Git: [공식 설치 안내](https://git-scm.com/install/)
- VS Code: [공식 다운로드](https://code.visualstudio.com/Download). Python 확장은 편집 편의를 위해 설치해도 좋습니다.

Windows PowerShell:

```powershell
py -3.12 --version
git --version
```

macOS/Linux 터미널:

```bash
python3.12 --version
git --version
```

Python 3.12.x와 Git 버전이 출력되면 성공입니다. Mac에서 `python3.12`를 찾지 못하면 설치를 확인하고 터미널을 다시 열어주세요.

## 5. 세미나 저장소 Fork 및 클론

1. [세미나 원본 저장소](https://github.com/bluebomin/docker-cicd-seminar)를 열고 **Fork**를 눌러 본인 GitHub 계정으로 복사합니다.
2. 본인 Fork 저장소의 Code → HTTPS 주소를 복사합니다.
3. 기존 동아리 레포 안이 아니라, 개인 GitHub 작업 폴더에서 아래 명령을 실행합니다. `YOUR_USERNAME`은 본인 GitHub 아이디로 바꿉니다.

```bash
git clone https://github.com/YOUR_USERNAME/docker-cicd-seminar.git
cd docker-cicd-seminar
```

VS Code에서 클론한 `docker-cicd-seminar` 폴더를 엽니다. 기존 동아리 레포와 별개의 저장소입니다. 원본 저장소를 바로 클론하면 참가자에게 push 권한이 없으므로 본인의 Fork를 클론해주세요.

```bash
git remote -v
```

`origin` 주소가 본인 Fork 저장소인지 확인합니다. 아래 파일은 이미 제공되어 있으므로 직접 만들거나 ZIP을 받을 필요가 없습니다. Dockerfile과 Actions workflow는 세미나에서 작성합니다.

```text
docker-cicd-seminar/
├── README.md
├── main.py
├── requirements.txt
├── test_main.py
├── .gitignore
└── .dockerignore
```

외부 서버에 올리지 않아도 실습할 수 있습니다. 다음 코드는 제공된 파일의 내용 설명입니다.

### main.py

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def health():
    return {"message": "hello docker"}
```

### requirements.txt

```text
fastapi
uvicorn[standard]
pytest
pytest-cov
httpx
```

이 파일에는 로컬 실습과 CI 테스트용 패키지를 함께 넣었습니다. 실제 운영 프로젝트에서는 실행용·개발용 의존성을 나누고 버전을 고정하는 것이 좋습니다. 이 안내의 패키지 목록은 특정 버전 조합을 고정한 목록은 아닙니다.

### test_main.py

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)


def test_health():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "hello docker"}
```

### .gitignore

```text
.venv/
__pycache__/
.pytest_cache/
.coverage
htmlcov/
.env
.DS_Store
```

### .dockerignore

```text
.venv
.git
__pycache__
.pytest_cache
.coverage
htmlcov
.env
.DS_Store
```

`.gitignore`는 Git 업로드 대상에서, `.dockerignore`는 Docker 빌드에 전달되는 파일에서 제외합니다.

## 6. 가상환경 만들고 패키지 설치

아래 명령은 **클론한 docker-cicd-seminar 폴더 안에서** 실행합니다. 가상환경 활성화 없이 해당 Python을 직접 사용하므로 PowerShell의 스크립트 실행 정책을 바꿀 필요가 없습니다.

### Windows PowerShell

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
.\.venv\Scripts\python.exe -m pytest -q
```

### macOS/Linux

```bash
python3.12 -m venv .venv
./.venv/bin/python -m pip install --upgrade pip
./.venv/bin/python -m pip install -r requirements.txt
./.venv/bin/python -m pip check
./.venv/bin/python -m pytest -q
```

성공 기준:

- `pip check`: `No broken requirements found.`
- `pytest`: `1 passed` 포함

## 7. FastAPI 로컬 실행 확인

Windows PowerShell:

```powershell
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

macOS/Linux:

```bash
./.venv/bin/python -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

실행 중인 터미널을 그대로 두고 브라우저에서 확인합니다.

- http://localhost:8000 → `{"message":"hello docker"}` 응답
- http://localhost:8000/docs → Swagger UI 화면

`/docs`에서 GET `/` → Try it out → Execute를 눌러 200 응답을 확인합니다.

확인 후 터미널에서 **Ctrl+C**를 눌러 서버를 종료해주세요. 세미나에서 Docker 컨테이너가 같은 8000번 포트를 사용합니다.

## 8. GitHub 준비

GitHub 로그인과 본인 계정으로 Fork한 저장소를 확인합니다. 새 개인 저장소를 따로 만들 필요는 없습니다. 저장소의 Actions 탭에서 Fork workflow 활성화를 요청하면 안내에 따라 활성화합니다. 현재 starter에는 workflow가 없으며 세미나에서 직접 작성합니다.

코드 업로드와 Actions 설정은 세미나에서 진행합니다. 실습은 본인 Fork 저장소에 push하고 해당 저장소의 Actions 결과를 확인하는 방식입니다.

이번 사전 준비에서는 GitHub 토큰, Docker Hub 계정, 클라우드 계정을 만들 필요가 없습니다. GHCR 자동 업로드를 진행한다면 GitHub Actions가 제공하는 `GITHUB_TOKEN`을 사용하도록 진행자가 안내합니다. 수동 GHCR push는 별도 인증이 필요하므로 진행자 시연으로 진행합니다.

## 9. 최종 제출 체크리스트

- [ ] Docker Desktop 실행 완료
- [ ] `docker version`에 Client·Server 정보 출력
- [ ] `docker info` 성공
- [ ] `docker compose version` 성공
- [ ] `docker run --rm hello-world`에서 `Hello from Docker!` 확인
- [ ] 검증용 nginx 화면 접속 후 컨테이너 종료
- [ ] Python 3.12.x와 Git 버전 확인
- [ ] 프로젝트 파일 준비 및 패키지 설치 완료
- [ ] `pip check` 성공, `pytest`에서 `1 passed` 확인
- [ ] API 응답과 `/docs` 확인 후 Ctrl+C로 종료
- [ ] GitHub 로그인, 본인 Fork 생성 및 클론 완료

진행자에게 **운영체제(Windows/Mac/Linux), Docker 검증 성공 여부, FastAPI 실행·테스트 성공 여부**를 알려주세요. 오류가 있으면 실행한 명령과 오류 메시지를 함께 전달해주세요. 토큰·비밀번호가 포함된 화면은 공유하지 마세요.

## 10. 자주 막히는 문제

| 증상 | 확인할 것 |
| --- | --- |
| `docker` 명령을 찾을 수 없음 | 설치 완료 여부 확인 후 터미널 다시 열기 |
| `Cannot connect to the Docker daemon` 또는 연결 오류 | Docker Desktop 실행, Engine 시작 완료 여부 확인 |
| Windows WSL·가상화 관련 오류 | `wsl --version` 확인, 설치·업데이트 후 재부팅, 오류 메시지 전달 |
| `hello-world` 다운로드 실패 | 인터넷·프록시 상태 확인, 재시도 후 오류 전달 |
| 컨테이너 이름이 이미 사용 중 | `docker ps -a`로 확인. 본인이 만든 검증 컨테이너가 실행 중이면 `docker stop seminar-docker-check`, 종료 상태면 `docker rm seminar-docker-check` |
| `port is already allocated` | 해당 서버 종료 또는 호스트 포트 변경 |
| Python 명령을 찾을 수 없음 | 설치와 PATH 확인, 터미널 다시 열기 |
| `requirements.txt` 또는 `main`을 찾지 못함 | 현재 위치가 프로젝트 폴더인지 확인. Windows는 `Get-Location`, Mac은 `pwd` |
| `ModuleNotFoundError` | 위 가상환경 Python으로 패키지 설치·실행했는지 확인 |
| 브라우저 접속 실패 | 서버 터미널의 오류 확인, 실행 중인지 확인, 주소와 포트 확인 |

## 세미나 시작 직후 재확인

설치 완료 보고를 했더라도 당일 Docker Desktop을 켜고 아래 명령을 다시 실행합니다.

```bash
docker version
docker compose version
docker run --rm hello-world
```

가상환경 Python으로 `-m pytest -q`도 다시 실행합니다. Docker 검증과 테스트가 성공하면 본 실습을 시작합니다. 실패하면 오류를 먼저 해결하며, 시간이 오래 걸리는 설치 문제는 진행자와 함께 화면을 보며 실습에 참여합니다.

## 참고 문서

- [Python 가상환경](https://docs.python.org/3.12/tutorial/venv.html)
- [FastAPI 시작하기](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [FastAPI 테스트 및 httpx](https://fastapi.tiangolo.com/tutorial/testing/)

---

## 공지용 문구

Docker·CI/CD 세미나 사전 준비 안내입니다! 세미나 전까지 이 문서를 따라 Docker와 Python을 설치하고 세미나 저장소를 본인 계정으로 Fork한 뒤 클론해주세요. 설치만 하지 말고 `hello-world` 실행, FastAPI 응답, 테스트 `1 passed`까지 꼭 확인해주세요. 외부 서버나 AWS 계정은 필요하지 않습니다. 오류가 생기면 운영체제·실행 명령·오류 메시지를 미리 알려주세요. 당일에는 노트북과 충전기를 가져와주세요.
