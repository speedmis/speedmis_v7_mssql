# db/ — 초기 데이터 번들 (MSSQL)

`install.php` 가 최초 구동 시 여기서 초기 데이터(`speedmis_db`)를 읽어 설치합니다.

## 파일

| 파일 | 설명 |
|------|------|
| `speedmis_db.sql.gz` | MSSQL 초기 데이터 (스키마 + 데이터). gzip 압축. **설치 시 우선 사용** |
| `speedmis_db.sql` | 위의 비압축 버전 (둘 중 있는 것을 사용) |

설치 마법사 동작 순서:
1. 로컬 `db/speedmis_db.sql.gz` → `db/speedmis_db.sql` 순으로 찾아 사용
2. 둘 다 없으면 `.env` 의 `DB_BUNDLE_URL`(Public 레포 raw)에서 자동 다운로드

## 마스킹 정책 (중요)

이 번들은 **운영 데이터를 거의 그대로** 담되, 아래만 마스킹합니다:
- `mis_users.passwd_decrypt` → 비움 (로그인은 `.env` 의 `MASTER_PASSWORD=4321` 만능비번으로)
- 고객/사용자의 개인정보 컬럼(전화·휴대폰·이메일·주소·주민/사업자번호 등) → 더미값으로 치환

> Public 레포이므로 **실제 개인정보·실접속정보는 절대 커밋하지 않습니다.**

## 덤프 생성 절차 (관리자용)

서버(MSSQL)에서 schema+data 를 스크립트로 추출 후 마스킹합니다. 권장 도구: `mssql-scripter`.

```bash
# 예시 (서버에서 실행)
pip install mssql-scripter
mssql-scripter -S localhost -d speedmis_db -U sa -P '***' \
  --schema-and-data --target-server-version 2019 \
  --exclude-objects "..." > speedmis_db.raw.sql
# → 마스킹 패스 적용 후 gzip → 이 폴더에 speedmis_db.sql.gz 로 커밋
```

생성 규칙:
- `CREATE DATABASE` / `USE` 구문 제외 (설치 마법사가 DB 를 만들고 선택함)
- 배치 구분자는 `GO` (설치 마법사가 GO 단위로 분리 실행)
