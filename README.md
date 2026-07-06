# team-report

인라인 Accept/Defer/Reject 결정 카드와 피드백 루프를 갖춘 인터랙티브 페이지네이션 HTML 리포트를 생성하는 멀티 에이전트 팀 분석 스킬.

분석 범위에 따라 전문가 서브에이전트를 3~10개 동적으로 구성해 병렬 분석을 수행하고, 단일 파일로 완결된 HTML 리포트를 생성합니다. 결정 사항은 `localStorage`에 저장되며 `docs/reports/feedback.yaml`에 누적되어 이후 분석 품질을 점진적으로 개선합니다.

## 설치 (Claude Code)

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
