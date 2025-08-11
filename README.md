# GIT 브랜치 전략 테스트

## 브랜치별 역할

| 브랜치  | 역할          |
| ------- | ------------- |
| main    | 상용 브랜치   |
| release | QA 브랜치     |
| develop | 테스트 브랜치 |

## 기능
- release bump는 github action으로 수행하고, 슬랙에 알림이 발송됩니다.
