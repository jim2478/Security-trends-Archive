# Security Trends Archive

보안 트렌드 모니터링 수집 결과 아카이브입니다.

## 구조

```
{연도}/
  {날짜}/
    {날짜}_{시간}.json
```

예시:
```
2026/
  2026-04-16/
    2026-04-16_09.json
    2026-04-16_10.json
    ...
```

## 데이터 형식

각 JSON 파일에는 해당 시간대에 수집된 보안 알림이 포함됩니다:

- **scan_time**: 스캔 실행 시각
- **stats**: 수집 통계 (총 스캔, 키워드 매치, 중복 제거 등)
- **alerts**: 알림 목록
  - **title**: 기사 제목
  - **url**: 원본 링크
  - **keywords**: 매칭된 키워드 (DDoS, CVE, Botnet, Malware 등)
  - **cve_details**: CVE 상세 (CVSS, CWE, EPSS, KEV)
  - **ioc_raw_text**: 침해지표 원문 (IP, Domain, Hash 등)

## 자동 업데이트

GitHub Actions를 통해 매 1시간마다 자동으로 업데이트됩니다.
