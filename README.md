# study-airflow
Airflow 배포 및 설정 테스트용 레퍼지토리

## 생성 목적
- Airflow를 빠르게 설치하고 서버 인프라를 충분히 활용하는 방법을 연구한다.
- Airflow를 모니터링하거나, 간단한 데이터 수집 파이프라인을 구축한다.

## 프로젝트 구조
- Airflow를 설치하기 위한 Docker / docker-compose
- Airflow의 백엔드 DB역할을 수행할 Postgres
- Airflow 시스템을 모니터링할 Grafana, Prometheus

## Airflow 설치 (3.0.6 버전 기준)
1. docker-compose 파일을 다운로드 한다.
```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.0.6/docker-compose.yaml'
```

2. `.env`파일을 생성해서 AIRFLOW_UID, AIRFLOW_GID를 지정한다.
```bash
id -u | id -g #UID, GID 조회 
```

<`.env` 파일 내용>
```.env
AIRFLOW_UID=1000
AIRFLOW_GID=0
```

3. `check_jwt` 파일을 만들어 apiserver(구 Webserver)에 접속하기 위한 웹토큰 생성
```bash
#!/bin/bash

python - <<'PY'
import secrets
print(secrets.token_urlsafe(48))
PY
```

4. docker-compose.yaml 파일에서 웹서버 및 백엔드 DB 사용자/비밀번호 접속정보 설정가능
```yaml
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://<Username>:<Password>@<Hostname>/<DBname>
AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://<Username>:<Password>@<Hostname>/<DBname>

# .env에 저장하여 아래와 같이 설정가능
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: "postgresql+psycopg2://${AF_DB_USER}:${AF_DB_PASS}@${AF_DB_HOST}:${AF_DB_PORT}/${AF_DB_NAME}"
AIRFLOW__CELERY__RESULT_BACKEND:    "db+postgresql://${AF_DB_USER}:${AF_DB_PASS}@${AF_DB_HOST}:${AF_DB_PORT}/${AF_DB_NAME}"

_AIRFLOW_WWW_USER_USERNAME: ${_AIRFLOW_WWW_USER_USERNAME:-<웹서버 로그인 Username>}
_AIRFLOW_WWW_USER_PASSWORD: ${_AIRFLOW_WWW_USER_PASSWORD:-<웹서버 로그인 Password>}
```