# Security Trends Archive

보안 트렌드 모니터링 수집 결과 아카이브입니다.

## 구조

```
daily/
  YYYY-MM-DD.json    # 하루치 알림을 통합 저장 (KST 기준)
```

예시:
```
daily/
  2026-05-13.json
  2026-05-14.json
  ...
```

스캔이 한 번이라도 끝까지 돈 날은 알림이 0건이어도 파일이 생깁니다
(`alerts: []`, `scan_runs` 만 증가). 따라서 **파일이 없는 날은 모니터가 한 번도
완료되지 않은 날**입니다. 단, 아래 [과거 데이터 주의](#과거-데이터-주의)의 기간은 예외입니다.

## 데이터 형식

`daily/YYYY-MM-DD.json` 한 파일에 그날 24시간 분량이 모두 들어있습니다.

```json
{
  "date": "2026-05-13",
  "tz": "Asia/Seoul",
  "stats": {
    "scan_runs": 24,
    "total_articles_max": 1862,
    "matched_articles_max": 169,
    "new_alerts": 27,
    "cve_total": 17,
    "alerts_with_cve": 11,
    "cve_count": 17,
    "ioc_count": 0,
    "merged_count": 1,
    "unique_cves": ["CVE-2026-40361", "..."],
    "keywords_hit": {"Malware": 14, "CVE": 12, "DDoS": 1}
  },
  "alerts": [
    {
      "scan_time_kst": "2026-05-13T20:01:10+09:00",
      "type": "single",
      "title": "...",
      "url": "...",
      "source": "...",
      "published": "...",
      "keywords": ["CVE"],
      "ioc_raw_text": "",
      "cve_ids": ["CVE-2026-..."],
      "cve_details": [{"cve_id": "...", "cvss_score": 9.8, "cvss_severity": "CRITICAL", "kev": false, ...}]
    }
  ]
}
```

### stats 필드

"알림"은 Slack 으로 나간 단위입니다. 같은 CVE 를 다룬 기사 여러 건은 병합돼 알림 1건이 됩니다.

- `scan_runs`: 그날 끝까지 돈 스캔 실행 횟수. 알림 0건인 실행도 셉니다. 정상이면 하루 약 24.
  수집 게이트 등으로 도중에 실패한 실행은 기록되지 않으므로 세지 않습니다.
- `total_articles_max` / `matched_articles_max`: 그날 실행들 중 수집 기사 수 / 키워드 매칭 기사 수의 최댓값
- `new_alerts`: 그날 보낸 알림 수 (병합 후)
- `cve_total`: CVE ID 등장 횟수의 합. 병합 전 기사 단위로 셉니다 — 기사 1건에 CVE 가 3개면 3,
  같은 CVE 가 기사 2건에 나오면 2. 고유 CVE 수는 `unique_cves` 의 길이입니다.
- `alerts_with_cve`: CVE 가 1개 이상 있는 알림 수 (`new_alerts` 와 같은 단위)
- `cve_count`: `cve_total` 과 항상 같은 값. 하위 호환용으로 남겨 둔 필드이므로 새로 읽는 쪽은
  `cve_total` 을 쓰세요. 예전 README 는 이 필드를 "CVE가 매칭된 알림 수"라고 설명했지만,
  실제 값은 처음부터 CVE ID 등장 횟수의 합이었습니다. 알림 수가 필요하면 `alerts_with_cve` 를 쓰세요.
- `ioc_count`: 그날 추출한 IoC 지표 개수의 합
- `merged_count`: 병합으로 줄어든 알림 수 (병합 전 기사 수 − 병합 후 알림 수)
- `unique_cves`: 그날 알림에 등장한 고유 CVE ID 목록
- `keywords_hit`: 키워드별로 그 키워드를 포함한 알림 수

### alert 필드
- `type`: `single` (개별 기사) | `merged` (동일 CVE 그룹 — `title`/`url` 대신 `sources[]` 에 기사별 `title`/`url`/`source`)
- `keywords`: 매칭된 관심 키워드 (CVE, Malware, DDoS, Botnet 등)
- `cve_ids` / `cve_details`: NVD/KEV에서 보강한 CVE 메타데이터
- `ioc_raw_text`: 침해지표 원문 (IP, Domain, Hash 등)

## 과거 데이터 주의

| 기간 | `scan_runs` 의 의미 | 알림 0건인 날 |
|------|------|------|
| ~ 2026-05-13 | 실제 실행 횟수 (시간별 파일에서 이관, 대부분 24) | 파일 있음 (예: `2026-04-19.json`) |
| 2026-05-14 ~ 0건 기록 복구(B-9) 반영 전 | **알림이 나간 실행 횟수만** 셈 | **파일 없음** |
| 0건 기록 복구(B-9) 반영 후 | 실제 실행 횟수 | 파일 있음 (`alerts: []`) |

- 두 번째 기간에는 알림이 없던 실행이 기록되지 않았습니다. 그래서 그 기간에 파일이 없는 날은
  "조용한 날"인지 "모니터가 돌지 않은 날"인지 아카이브만으로는 구분할 수 없고,
  `scan_runs` 로 실행 누락을 판단할 수도 없습니다. 소급 보정은 불가능합니다.
- `cve_total` / `alerts_with_cve` 는 B-9 반영 후에 쓰인 파일에만 있습니다. 그 전 파일에서는
  `cve_count` 가 `cve_total` 과 같은 의미이고, `alerts_with_cve` 는 `alerts[]` 중 `cve_ids` 가
  비어 있지 않은 항목 수로 직접 셀 수 있습니다. 반영 당일 파일은 두 필드가 이 방식으로 채워집니다.

## 자동 업데이트

[Security-trends](https://github.com/jim2478/Security-trends) GitHub Actions가 시간당 1회 실행됩니다.
매 실행마다 그날의 `daily/YYYY-MM-DD.json` 에 `stats` 를 누적하고 새 알림을 `alerts` 뒤에 붙입니다.
알림이 0건이어도 `scan_runs` 가 바뀌므로 거의 매 실행이 커밋 1개를 만듭니다.
