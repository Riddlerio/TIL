# 5장. FastAPI 기초와 Routing

## 핵심 키워드 10개

1. **클라이언트**: 서버에 HTTP 요청을 보내는 브라우저, 앱, 테스트 코드 등의 프로그램
2. **서버**: 요청을 받아 처리하고 HTTP 응답을 돌려주는 프로그램
3. **REST API**: URL의 자원과 HTTP method의 작업을 조합해 만든 API 방식
4. **HTTP method**: `GET`, `POST`처럼 자원에 할 일을 나타내는 요청 방식
5. **FastAPI**: API 경로와 처리 함수를 Python으로 정의하는 웹 프레임워크
6. **Uvicorn**: FastAPI 앱을 주소와 포트에서 실행해 HTTP 요청을 받게 하는 서버
7. **Routing**: method와 path를 특정 처리 함수에 연결하는 작업
8. **Path parameter**: URL 경로 안에서 특정 자원을 지정하는 값
9. **Query parameter**: 조회 범위·정렬·개수 등 조회 방법을 조절하는 값
10. **상태 코드**: 요청 처리 결과를 짧게 알려 주는 HTTP 숫자 코드

## 핵심 내용 정리

### 5-1. 첫 FastAPI 서버

- 요청과 응답은 한 쌍입니다. 클라이언트가 method, URL, 필요하면 데이터를 보내면 서버가 상태 코드와 JSON 같은 응답을 반환합니다.
- REST API에서는 **path가 주로 자원**을, **HTTP method가 자원에 할 일**을 나타냅니다. 따라서 같은 `/documents`라도 `GET`과 `POST`는 서로 다른 계약입니다.
- FastAPI는 `FastAPI()` 객체에 `@app.get()` 같은 데코레이터로 경로와 함수를 등록합니다. Uvicorn은 그 앱을 실제 네트워크에서 실행합니다.
- `GET /`와 `GET /documents`처럼 작은 endpoint부터 만들 수 있습니다. 서버가 실행 중이면 `/docs`에서 자동 생성 문서를, `/openapi.json`에서 API 명세를 확인할 수 있습니다.
- `TestClient`를 사용하면 실제 브라우저나 별도 서버 실행 없이도 요청·응답·상태 코드를 코드로 검증할 수 있습니다.

### 5-2. Routing과 요청 데이터

- Routing은 **method + path**를 처리 함수에 연결합니다. 함수 이름은 서버 내부 구현 이름일 뿐, 외부 계약은 데코레이터의 method와 path입니다.
- Path parameter는 `/documents/{document_id}`처럼 문서 한 건을 고르는 위치 정보입니다. 보통 자원 식별에 쓰므로 필수입니다.
- Query parameter는 `/documents?limit=12`처럼 목록의 개수나 필터 조건을 전달합니다. 함수의 기본값과 `ge`, `le` 같은 범위를 줄 수 있습니다.
- Request body는 `POST`·`PUT` 등에서 JSON 전체를 전달할 때 쓰며, 여러 필드를 하나의 요청 모델로 묶기에 적합합니다.
- 대표 상태 코드는 성공의 `200`, 찾는 자원이 없는 `404`, 입력 형식이나 제약조건이 잘못된 `422`입니다.
- `GET /health`는 데이터베이스 같은 복잡한 정보를 보여 주기보다 서버가 살아 있는지 외부 점검 도구가 확인하도록 만드는 단순한 endpoint입니다.

## 헷갈리기 쉬운 부분

| 구분 | 정확한 이해 |
| --- | --- |
| FastAPI vs Uvicorn | FastAPI는 API 규칙과 처리 함수를 정의하고, Uvicorn은 이를 네트워크에서 실행합니다. |
| path vs HTTP method | path는 어떤 자원인지, method는 그 자원에 무엇을 할지 나타냅니다. 둘을 함께 봐야 route가 완성됩니다. |
| path parameter vs query parameter | path는 특정 자원을 고르고, query는 목록 개수·필터 같은 조회 조건을 조절합니다. |
| query vs request body | 짧은 조회 조건은 query, 여러 구조화된 입력 데이터는 JSON body가 적합합니다. |
| 404 vs 422 | 404는 대상 자원이 없을 때, 422는 요청 값의 타입·범위·형식이 계약에 맞지 않을 때입니다. |
| `/docs` vs `/openapi.json` | `/docs`는 사람이 보는 대화형 문서이고, `/openapi.json`은 도구가 읽을 수 있는 명세 데이터입니다. |

## 실전에 많이 쓰는 부분

- **자동 API 문서**: endpoint와 Pydantic 모델을 잘 선언하면 Swagger UI(`/docs`)와 OpenAPI 명세가 자동으로 유지됩니다.
- **목록 API 설계**: `GET /documents?limit=20`처럼 query로 페이지 크기, 정렬, 날짜 필터를 전달합니다.
- **단건 조회**: `GET /documents/{document_id}`처럼 path parameter로 명확한 자원 ID를 사용합니다.
- **입력 방어**: query 범위와 request body 스키마를 선언해 잘못된 값이 내부 로직까지 들어오는 일을 줄입니다.
- **모니터링**: 로드밸런서, 컨테이너 플랫폼, 배포 도구가 `GET /health`를 호출해 서비스 상태를 확인합니다.
- **회귀 테스트**: TestClient로 200·404·422 응답을 고정해 두면 routing이나 검증 규칙 변경으로 인한 오류를 빨리 발견할 수 있습니다.

## 한 줄 복습

**FastAPI에서는 method와 path로 요청 계약을 만들고, path·query·body를 목적에 맞게 나누며, 상태 코드와 자동 문서로 API를 검증한다.**
