# team-report

사용자의 요청에 맞춰 반응형 HTML 보고서를 만들어주는 Claude Code 스킬입니다. 분석 주제와 범위에 따라 전문가 서브에이전트를 3~10개 동적으로 구성해 병렬 분석하고, 결과를 모바일/태블릿까지 대응하는 단일 파일 인터랙티브 HTML 보고서로 정리합니다. 보고서 안에서 제안마다 수락/보류/거절을 선택하면 `docs/reports/feedback.yaml`에 누적되어 다음 보고서 품질이 점진적으로 개선됩니다.

## 설치

```
/plugin marketplace add DevP0tion/DevP0tion
/plugin install team-report@devp0tion
```

## 사용법

```
/team-report color theme analysis
/team-report lobby component architecture review
/team-report performance optimization report
```

스킬 본문과 참조 문서는 [`skills/team-report/`](skills/team-report/)에 있습니다.
