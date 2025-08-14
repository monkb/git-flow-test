# GIT 브랜치 전략 테스트

## 브랜치별 역할

| 브랜치  | 역할          |
| ------- | ------------- |
| main    | 상용 브랜치   |
| release | QA 브랜치     |
| develop | 테스트 브랜치 |

## workflow

- 기능 개발 시 main 브랜치를 base 로 feature 브랜치를 생성합니다.
- feature 브랜치 개발이 완료되면 release 브랜치로 PR을 생성합니다.
  - 변경 범위에 따라 major-bump, minor-bump, patch-bump 레이블을 추가합니다.
  - PR이 승인되면 QA 일정에 따라 merge 합니다. ⚠️ QA 할 대상만 merge 합니다.
- release 에 feature 가 merge 되면 bump PR이 생성됩니다. (package.json의 rc버전을 올리는 PR입니다.)
  - QA 진행중에 생기는 이슈들은 release 로의 새로운 PR을 만들어서 수정합니다.
  - 이 때에는 rc-bump 레이블을 추가하여야 합니다.
- QA가 완료되면 github action에서 Create Promote PR action을 수행합니다.
  - 수행하면 release -> main 으로의 PR이 생성됩니다.
- PR이 merge가 되면 main에 대한 bump PR이 생성됩니다. (rc 버전을 제거하는 PR)
- bump PR이 merge 되면 배포 tag가 확정됩니다.

### 각 workflow의 역할

| 워크플로우                    | 트리거                              | 역할/결과                                                                  |
| ----------------------------- | ----------------------------------- | -------------------------------------------------------------------------- |
| Feature → Release PR 레이블링 | `feature` → `release` 로 PR 생성    | `major-bump`/`minor-bump`/`patch-bump` 레이블로 다음 버전 증가 범위를 결정 |
| Release Bump (rc 버전 상승)   | `release` 에 feature가 merge        | `package.json`의 rc 버전을 올리는 bump PR 자동 생성                        |
| QA Hotfix (rc-bump)           | QA 중 `release` 로 버그픽스 PR 생성 | `rc-bump` 레이블로 rc 재버전 상승 포함, QA 반복 지원                       |
| Promote PR 생성               | QA 완료 후 수동 실행                | GitHub Actions의 “Create Promote PR”로 `release` → `main` PR 생성          |
| Main Bump (rc 제거)           | Promote PR이 `main` 에 merge        | rc 접미사를 제거하는 bump PR 자동 생성, 최종 배포 버전 확정                |
| 배포 - 개발                   | `develop` 브랜치에 push             | 개발 환경 자동 배포                                                        |
| 배포 - QA                     | rc 태그 생성                        | QA 서버 자동 배포                                                          |
| 배포 - 상용                   | 버전 태그 생성                      | 프로덕션 자동 배포                                                         |

#### 레이블 규칙

- **major-bump**: 다음 배포에서 Major 버전 증가
- **minor-bump**: 다음 배포에서 Minor 버전 증가
- **patch-bump**: 다음 배포에서 Patch 버전 증가
- **rc-bump**: QA 중 rc 재발행 및 rc 버전 재상승

## 배포

- develop : develop 브랜치에 푸시되면 배포됩니다.
- QA : rc 태그가 생성되면 배포됩니다. (QA 서버)
- 상용서버 : 버전 태그가 생성되면 배포됩니다.

## 왜 이렇게 하나요?

- jira의 상태와 관리하기 위함입니다.

## trouble shooting

- QA가 2개 이상의 feature를 포함할 때의 버전
- release의 version과 main의 version의 동기화
- vercel 은 tag 별 배포가 가능한가?
  - cli로 처리가 가능하다고 함
- release, 즉 QA 서버를 활용하지 않는 캐시워크 쪽과 협업 시에는 어떻게 해야될까?

## jira의 workflow와의 연계

- TODO : feature 브랜치 생성되지 않은 상태
- In progress : feature 브랜치 생성하여 push 된 상태
- IN PR : release 브랜치로 PR이 생성된 상태
- IN QA : release 로 PR이 merge 된 상태
- QA Passed : rc 가 main 으로 promote 된 상태
- DONE : main 의 tag 가 확정되어 배포된 상태
