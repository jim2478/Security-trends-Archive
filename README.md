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

알림이 없는 날은 파일이 생성되지 않습니다.

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
- `scan_runs`: 그날 스캔 실행 횟수
- `total_articles_max` / `matched_articles_max`: 마지막 스캔 시점의 코퍼스 크기
- `new_alerts`: 그날 발생한 신규 알림 총합
- `cve_count`: CVE가 매칭된 알림 수
- `unique_cves`: 그날 등장한 고유 CVE ID 목록
- `keywords_hit`: 키워드별 매칭 횟수

### alert 필드
- `type`: `single` (개별 기사) | `merged` (동일 CVE 그룹)
- `keywords`: 매칭된 관심 키워드 (CVE, Malware, DDoS, Botnet 등)
- `cve_ids` / `cve_details`: NVD/KEV에서 보강한 CVE 메타데이터
- `ioc_raw_text`: 침해지표 원문 (IP, Domain, Hash 등)

## 자동 업데이트

[Security-trends](https://github.com/jim2478/Security-trends) GitHub Actions가 시간당 1회 실행되며, 알림이 발생할 때마다 그날의 `daily/YYYY-MM-DD.json`에 append됩니다.
