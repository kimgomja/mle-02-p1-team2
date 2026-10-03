# Project_1 학습 앱

기존 팀 앱 streamlit_app.py와 별도로 최신 개인 작업을 보존한 SIF/KOSHA Dual RAG 앱입니다.

## 로컬 실행

저장소 루트 /home/playdata/workspace/mle-02-p1-team2에서 실행합니다.

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r src/Project_1/requirements.txt
.venv/bin/python -m streamlit run src/Project_1/app.py
```

저장소 루트의 로컬 .env 또는 Streamlit Secrets에 DATABASE_URL(또는 SUPABASE_DB_URL)과 OPENAI_API_KEY를 설정합니다. Supabase에서는 Session pooler의 PostgreSQL URI(5432 포트)를 사용합니다. 비밀값은 커밋하지 않습니다.

통계 CSV는 Supabase Private Storage에서 읽습니다. 비공개 Storage 인증이 실패하면 `public.rag_day1_documents`의 `industry_stat` 문서를 읽기 전용으로 조회해 2025년 자료를 표시합니다. 이 대체 경로에는 2020~2024년 자료가 없어 장기 추세가 비어 있습니다. Storage 설정이 없는 로컬 환경은 `src/Project_1/data/`를 사용합니다. 사고 검색에는 `rag_day1_documents`, KOSHA 검색에는 `langchain_pg_collection`/`langchain_pg_embedding`의 `kosha_guides` 컬렉션이 필요합니다. 대화 이력은 `chat_history`를 사용합니다. 팀 앱의 DB 구조와 같다고 가정하지 않습니다.

## 오프라인 대화 흐름 검사

```bash
PYTHONPATH=src/Project_1 .venv/bin/python -m unittest discover -s src/Project_1/tests -v
```

이 검사는 DB 및 OpenAI 호출을 mock으로 대체합니다. 실제 API/DB 통합 검증은 별도입니다.

## Streamlit Cloud

Main file path는 src/Project_1/app.py입니다. 이 폴더의 requirements.txt를 사용하며 Python 3.12에서 검증했습니다. 연결 정보는 Cloud Secrets에 넣습니다. Private Storage 설정을 함께 넣으면 로컬 통계 CSV 없이 실행할 수 있습니다.

노트북은 코드와 설명을 보존하고 실행 출력과 실행 번호를 비운 상태로 옮겼습니다. 개인 원본 저장소는 수정하지 않았습니다.

## 공유 Supabase와 통계 Storage 사용

프로젝트 관리자가 Supabase 대시보드에서 팀원의 접근 권한을 부여합니다. 팀원은 본인 계정으로 접근하며, 비밀값은 이 문서나 GitHub에 적지 않습니다. 실행에 필요한 키는 비밀번호 관리 도구 등 별도 안전한 경로로 전달합니다.

로컬에서는 저장소 루트의 .env에, Cloud에서는 앱의 Settings → Secrets에 다음 항목을 설정합니다. 아래는 Cloud Secrets의 TOML 예시이며 실제 값은 각자 설정합니다.

```toml
DATABASE_URL = "postgresql://USER:PASSWORD@HOST:5432/postgres?sslmode=require"
OPENAI_API_KEY = "본인의 키"
SUPABASE_URL = "https://PROJECT_REF.supabase.co"
SUPABASE_SECRET_KEY = "sb_secret_본인의서버키"
SUPABASE_STORAGE_BUCKET = "industrial-statistics"
```

SUPABASE_SECRET_KEY는 서버에서만 사용하며 읽기 전용 키가 아닙니다. 브라우저·채팅·커밋에 포함하지 않습니다. Storage 설정 세 항목은 함께 설정하고, Cloud Secrets가 로컬 환경변수보다 우선합니다.

버킷은 Private로 유지합니다. 최상위에는 accident_death_2025.csv, accident_injured_2025.csv, fatality_rate_2025.csv, business_count_2025.csv를 올립니다. history/에는 2020~2024년 accident_death와 accident_injured 각 5개, fatality_rate는 2020·2022·2023·2024년 4개를 올립니다. 총 18개이며, 제공되지 않은 2021년 사망만인율은 기존 통계·차트 로직에서 NaN으로 유지합니다.

Storage 설정이 있으면 로컬 CSV 없이 통계를 로딩합니다. CSV는 다운로드 후 메모리에서 처리하고 통계 결과는 1시간 캐시합니다. 파일을 갱신한 직후 반영하려면 Streamlit 메뉴의 Clear cache 또는 앱 재시작을 사용합니다. Storage 인증 실패 시 DB에 들어 있는 2025년 통계를 사용하고 화면에 데이터 범위 안내를 표시합니다. DB 대체 자료가 없거나 형식 검증에 실패하면 로딩 오류를 표시합니다. Storage 설정이 없는 로컬 환경은 기존 `data/` CSV를 사용합니다.

## Cloud 배포 및 시연 확인

- Repository: kimgomja/mle-02-p1-team2 (개인 Fork)
- Branch: codex/cloud-statistics-db-fallback
- Main file path: src/Project_1/app.py
- Python: 3.12 (Streamlit Advanced settings에서 직접 선택)
- Dependencies: app.py 옆 requirements.txt

SIF·KOSHA 문서의 공개 이용 범위가 정리될 때까지 Streamlit 공개 앱 생성은 보류합니다. Fork 브랜치 코드는 배포 후보이며 Cloud 통합 검증 전입니다.

Cloud에는 `.env`나 원본 데이터를 업로드하지 않습니다. GitHub에 코드를 push한 뒤 Cloud Secrets를 설정합니다. 다른 PC 또는 시크릿 창에서 통계·규모 필터·6년 추세(2021 사망만인율 공백), SIF/KOSHA 검색, 출처, 후속 질문, 새 대화를 확인합니다. 마지막으로 로컬 Streamlit과 Docker를 종료한 뒤 같은 Cloud URL에서 재확인합니다. 실제 질문은 OpenAI API를 사용합니다.

공유 DB에 연결하더라도 앱은 대화별 UUID를 사용합니다. 현재 화면은 같은 DB의 모든 대화를 조회하는 팀 공유 대화 목록이나 로그인 기능을 제공하지 않습니다.

## 알게 된 점과 유의사항

- 현재 Storage 키 조합으로는 비공개 CSV 요청이 인증되지 않았습니다. 그래서 확인된 2025년 DB 통계를 읽는 대체 경로를 추가했습니다.
- 공단 통계 CSV에는 CP949 인코딩 파일도 있습니다. 로더는 UTF-8 BOM과 CP949를 모두 읽고, 30개 업종·10개 규모 열과 숫자 값을 검증합니다. 알 수 없는 문자를 결측치로 숨기지 않습니다.
- DB 대체 경로는 `industry_stat` 120개 문서에서 2025년 4개 지표와 산업중분류 30개를 검증해 사용합니다. `자료 없음`은 결측으로 유지합니다.
- 이 DB에는 2020~2024년 통계 문서가 없어 6년 추세 차트는 2025년만 값이 있고 나머지는 비어 있습니다.
- DB 연결은 읽기 전용 트랜잭션으로 실행합니다. 재적재나 테이블 변경은 하지 않습니다.
- SIF·KOSHA 자료의 공개 사용 범위가 확인되기 전에는 공개 Streamlit 배포를 진행하지 않습니다.
