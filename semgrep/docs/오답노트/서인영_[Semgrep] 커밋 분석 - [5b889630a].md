# [Semgrep] 커밋 분석 - [5b889630a]

**주제: 토큰 유출 방지를 위한 보안 패치**

---

## 🧩 개요

- **프로젝트명:** Semgrep
- **커밋 해시:** `5b889630aefba7ef0a6294ba7599a2adc4e93e02`
- **커밋 메시지:** `fix(intsec): don't forward SEMGREP_APP_TOKEN to 3rd party rule hosts`
- **변경 파일:**
  - `cli/src/semgrep/app/session.py`
  - `cli/src/semgrep/config_resolver.py`
  - `cli/tests/default/unit/test_app_session.py`
  - `changelog.d/gh-11016.fixed`
- **관련 공개 이슈:** [semgrep/semgrep#11016](https://github.com/semgrep/semgrep/issues/11016)
- **관련 내부 PR:** `semgrep/semgrep-proprietary#4393` (공개 접근 및 세부 내용 확인 불가)

---

## 🔍 변경 요약

Semgrep CLI가 `-f`, `-c`, `--config` 옵션으로 전달된 외부 URL에서 룰을 가져올 때, `SEMGREP_APP_TOKEN`이 `Authorization` 헤더에 포함되어 제3자 서버로 전달될 수 있는 문제를 수정했다.

변경 전에는 URL의 도메인과 관계없이 토큰을 헤더에 추가했지만, 변경 후에는 `semgrep.dev` 및 해당 도메인의 하위 도메인에만 토큰을 전달하도록 제한했다. 또한 설정 URL의 출처를 구분하도록 `config_resolver.py`를 수정하고, 관련 단위 테스트를 추가했다.

---

## 🔐 보안 관점 분석

| 항목 | 이전 버전 | 변경 버전 | 보안적 의미 |
|---|---|---|---|
| 토큰 처리 | 요청 URL과 관계없이 토큰 전달 | Semgrep 공식 도메인에만 토큰 전달 | 제3자 서버로의 자격 증명 유출 방지 |
| 룰 다운로드 | 외부 URL 요청에도 인증 헤더가 포함될 수 있음 | 비공식 도메인 요청에서는 인증 헤더 제외 | 악의적인 URL을 통한 토큰 탈취 위험 감소 |
| URL 출처 구분 | 설정 출처에 따른 구분이 부족함 | `REGISTRY`, `REMOTE` 등으로 출처 구분 | 공식 레지스트리와 일반 원격 URL을 구분하여 처리 |
| 회귀 테스트 | 해당 수정에 대한 테스트 없음 | 도메인별 토큰 전달 여부를 검증하는 단위 테스트 추가 | 향후 동일한 문제가 다시 발생하는 것을 방지 |

### 주요 원인

인증 토큰을 요청에 추가하는 과정에서 URL의 신뢰성을 확인하지 않은 것이 근본 원인이다. 사용자가 지정한 설정 URL이 외부 서버를 가리키는 경우, 요청 헤더에 포함된 토큰이 해당 서버로 전달될 수 있었다.

### 공격 시나리오

1. 사용자가 신뢰할 수 없는 외부 룰 URL을 Semgrep 설정으로 지정한다.
2. Semgrep CLI가 해당 URL에 요청을 보낸다.
3. 수정 전 코드에서는 요청에 `SEMGREP_APP_TOKEN`이 포함될 수 있다.
4. 외부 서버가 토큰을 확보하면 토큰에 부여된 권한과 유효 범위에 따라 악용될 가능성이 있다.

이번 패치는 공식 도메인에만 토큰을 전달하도록 제한하여 이러한 위험을 줄인다.

---

## 🧾 검증 및 확인

- [x] 관련 공개 이슈 및 커밋 변경 기록 확인
- [x] 코드 변경 및 관련 단위 테스트 확인
- [x] 보안 관련 단위 테스트 실행: `1 passed`
- [x] 전체 테스트 실행: `1,396 passed`, `758 failed`, `10 skipped`, `2 xfailed`, `4 errors`
- [ ] 전체 테스트 및 빌드 정상 동작 확인
- [ ] 해당 커밋에 연결되는 공식 CVE 식별자 확인

전체 테스트 중 일부는 실패했다. 개별 실패 테스트를 재실행한 결과 `semgrep-core` 실행 파일을 찾지 못해 종료 코드 2로 종료되는 문제가 확인되었다. 현재 개발 환경에 OCaml 빌드 도구도 설치되어 있지 않아, 전체 테스트 결과만으로 코드 결함 여부를 확정하기는 어렵다.

또한 해당 커밋과 직접 연결되는 공식 CVE 식별자는 현재까지 확인하지 못했다.

---

## 💬 결론

- [x] 업데이트 성격: 보안 강화
- [x] CI/CD 신뢰성 영향: 관련 단위 테스트 추가로 회귀 방지에 기여할 수 있으나, 전체 CI 통과 여부는 확인하지 못함
- [x] 추가 확인 필요사항: 빌드 환경 구성 후 전체 테스트 재실행, URL 검증의 예외 사례 확인, 공식 CVE 등록 여부 확인
- [x] 보안상 의의: 공식 도메인이 아닌 서버로 인증 토큰이 전달되는 위험을 줄임

---

## 📚 참고

- 공개 이슈: https://github.com/semgrep/semgrep/issues/11016
- 커밋: https://github.com/semgrep/semgrep/commit/5b889630aefba7ef0a6294ba7599a2adc4e93e02
- 변경 기록: `changelog.d/gh-11016.fixed`
- 관련 내부 PR: `semgrep/semgrep-proprietary#4393`