# evaluation_and_connection

2026-11-25

## 오늘 배운 내용

`evaluation_and_connection.ipynb`에서는 Base·SFT·LoRA·QLoRA·DPO가 같은 dev 12문항에 남긴 응답을 비교했다. 질문과 근거, 생성 설정이 같은지 확인하고, 저장된 SFT 모델은 문서 검색을 거쳐 FastAPI에서 답하도록 연결했다.

`POST /answer` → `AnswerRequest` → `answer()` → `retrieve_documents()` → `generate_answer()` → `inspect_response()` → HTTP 응답

| 코드 | 하는 일 |
|---|---|
| `AnswerRequest` | `team`과 `question`의 타입·길이를 검사한다. 잘못된 입력은 `answer()` 호출 전에 `422`로 반환한다. |
| `retrieve_documents()` | 팀과 키워드에 맞는 규정 문서를 찾아 모델에 줄 근거를 준비한다. |
| `generate_answer()` | 질문과 근거를 SFT 모델에 넣고 새로 생성한 답변 토큰만 꺼낸다. |
| `inspect_response()` | 모델이 출력한 JSON 구조와 필드, 인용 ID를 검사한다. |
| `answer()` | 검색·생성·검사를 연결한다. 출력 형식과 길이 검사를 통과하면 `200`, 실패하면 `502`를 반환한다. 생성 단계에서 입력 토큰 제한을 넘기면 `422`를 반환한다. |

`file_sha256()`과 `check_dev_records()`는 평가에 쓰는 파일과 문항을 확인한다. `check_format_examples()`와 `check_retrieval_examples()`는 예제로 각 기능을 점검한다. API를 실행하기 전에는 문서 목록과 SFT 모델·토크나이저를 준비한다.

## 질문과 답변

### 1. `citations`는 무엇이고, `not citations`는 왜 쓰나?

`source_ids`는 모델에게 제공한 문서 ID 집합이고, `citations`는 모델이 답변의 근거로 지목한 ID 목록이다. 파이썬에서 빈 리스트 `[]`는 거짓으로 평가되므로 `not citations`로 인용이 비었는지 확인할 수 있다.

- `answered`에는 인용이 하나 이상 있어야 하고 `abstained`의 인용은 `[]`여야 한다.
- `set(citations).issubset(source_ids)`로 인용한 ID가 모두 제공된 문서에 있는지 확인한다.
- 인용 ID가 맞아도 모델이 규정을 잘못 해석할 수 있다. 70,000원을 65,000원 이하라고 답한 사례는 문제의 숫자와 규정 원문을 대조해야 발견할 수 있었다.

### 2. 실패는 어차피 실패인데, `Counter`로 왜 첫 실패만 세나?

문항별 통과 여부는 같지만, 모든 `False`를 세면 원인 집계가 달라진다. JSON 파싱에서 종료한 문항은 스키마와 인용 ID를 검사하기 전에 두 값이 초기값 `False`로 남는다.

```python
stage = next(
    (key for key, passed in item['format_checks'].items() if not passed),
    'format_passed',
)
stages[stage] += 1
```

`summarize-format` 셀에서 `next()`는 첫 번째 `False`의 키를 고른다. 전부 통과하면 `format_passed`를 고른다. `Counter`는 문항당 한 번만 증가하고, `assert sum(stages.values()) == 12`로 모델별 집계가 dev 12문항과 일치하는지 확인한다. 예를 들어 JSON 실패가 2건이면 전체 dev 대비 비율은 `2/12`다.

### 3. `selected_raw == known_bad`는 무엇을 확인하나?

`selected_raw`는 현재 보고서의 SFT 응답이고 `known_bad`는 사람이 이전에 읽고 오답으로 기록한 원문이다. 같은 평가 문항에서 문자열이 정확히 같으면 `reviewed_same_text`로 기록하고 기존 판정을 재사용한다. `70,000원`이 `7만 원`으로만 바뀌어도 `new_text_needs_review`가 되어 다시 검토한다. 이 비교로 확인하는 건 기존 검토가 적용되는 답변인지다.

### 4. 학습 데이터가 적으면 증강해도 되나?

dev 12문항은 평가에 쓰이므로 이 숫자로 필요한 학습 데이터 양을 정할 수 없다. 증강은 기존 train에서만 하고, 같은 dev에서 `dev loss`, 형식 통과율, 실제 승인 판단을 비교한다.

- **조건을 바꾸는 증강**은 Python으로 만든다. 한도가 65,000원이면 64,999원·65,000원·65,001원에 맞춰 정답을 각각 계산한다.
- **표현을 바꾸는 증강**은 [Distilabel](https://github.com/argilla-io/distilabel)과 Ollama로 생성할 수 있다. 금액·단위·질문 의도가 유지된 후보를 골라 원래 근거와 정답을 붙인다.
- **DPO용 증강**은 같은 질문에 올바른 승인 판단을 `chosen`, 인용 형식은 갖췄지만 승인 조건을 뒤집은 응답을 `rejected`로 만든다.
