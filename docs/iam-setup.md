# 실습용 IAM 사용자와 CLI 인증 준비

상태: 학습자가 IAM 사용자 생성, 정책 연결, MFA 및 CLI 인증 완료를 보고했다. 정책 파일은 로컬 JSON 구문 검사를 통과했으며, 에이전트가 AWS에 직접 적용하거나 연결 정책을 조회하지는 않았다.

## 구성

- IAM 사용자 이름: `codyssey-lab`
- 사용자 관리형 정책 이름: `CodysseyLabInfrastructure`
- 정책 본문: [infra/iam/lab-policy.json](../infra/iam/lab-policy.json)
- 추가 AWS 관리형 정책: `SignInLocalDevelopmentAccess`
- CLI 프로필: `codyssey`
- CLI 인증: AWS CLI 2.32.0 이상에서 `aws login`을 통한 임시 인증 정보

이 구성은 신규 개인 계정에서 과제용 IAM 사용자를 사용하는 경로다. 루트의 MFA와 별개로 실습 사용자에도 MFA를 등록한다. 장기 액세스 키를 생성할 필요는 없다.

## 정책 범위와 한계

EC2/VPC 생성·조회·연결·삭제에 사용하는 작업을 나열하고 서울 리전으로 제한한다. IAM 관리, S3, RDS, NAT Gateway, 로드 밸런서 생성 권한은 포함하지 않는다. AMI는 EC2 DescribeImages로 조회하고 EC2에 IAM Role을 연결하지 않는 기본 Nginx 배포를 전제로 한다. Elastic IP는 생성하지 않고 EC2의 자동 할당 퍼블릭 IPv4를 사용한다.

`Resource: "*"`이므로 나열된 작업은 서울 리전의 다른 리소스에도 적용될 수 있다. 리소스 ID나 태그 단위 격리까지 구현한 정책은 아니다. 인스턴스 종류·개수·볼륨 용량·보안 그룹 포트도 이 정책이 강제하지 않으므로 실행 명령에서 과제 조건을 지켜야 한다. 이 정책만으로 무료 사용이 보장되지는 않는다.

필요한 작업이 거부되면 오류의 작업 이름과 대상 리소스를 확인한 후 해당 권한만 검토한다. `AdministratorAccess`나 `AmazonEC2FullAccess`로 대체하지 않는다.

## 사용자가 콘솔에서 수행할 작업

아래 최초 IAM 준비는 계정 관리 작업이다. 과제 리소스 생성·운영은 준비한 실습 사용자로 수행한다.

1. IAM → 정책 → 정책 생성 → JSON 편집기에 `infra/iam/lab-policy.json` 내용을 넣는다.
2. 검증 오류를 확인하고 `CodysseyLabInfrastructure`라는 이름으로 정책을 생성한다.
3. IAM → 사용자 → 사용자 생성에서 `codyssey-lab`을 입력한다.
4. 콘솔 액세스를 활성화하고 IAM 사용자 생성 옵션을 선택한다. 로그인 비밀번호를 설정한다.
5. 직접 정책 연결 방식으로 `CodysseyLabInfrastructure`와 `SignInLocalDevelopmentAccess`만 연결한다.
6. 사용자 생성 후 해당 사용자의 보안 자격 증명에서 MFA 장치를 등록한다. 현재 관리 세션에서 등록하면 실습 사용자에게 IAM 관리 권한을 추가할 필요가 없다.
7. IAM 사용자 로그인 URL과 사용자 이름을 보관한다. 비밀번호·MFA 복구 정보는 저장소에 넣지 않는다.
8. 루트 세션에서 로그아웃하고 IAM 사용자로 로그인한다. 요구되는 비밀번호 변경이 있다면 완료한다.

## 사용자가 로컬 터미널에서 수행할 작업

```bash
aws --version
aws login --profile codyssey
```

CLI 버전이 2.32.0 이상인지 확인한다. 리전 질문에는 `ap-northeast-2`를 입력하고 브라우저에서 반드시 `codyssey-lab` 사용자를 선택한다.

```bash
aws configure set region ap-northeast-2 --profile codyssey
aws configure set output json --profile codyssey
aws sts get-caller-identity --profile codyssey
```

반환된 `Arn`이 `arn:aws:iam::<계정 ID>:user/codyssey-lab` 형태인지 확인한다. `:root`이면 과제 명령을 실행하지 말고 실습 사용자로 다시 로그인한다. 이 확인은 인증 신원만 검증하며, EC2 생성 권한 전체를 검증하지는 않는다.

로그인 URL의 인증 코드, 토큰, 비밀번호는 공유하지 않는다. 결과 공유 시 계정 ID를 가려도 된다.

## 공식 문서

- [IAM 사용자 생성](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
- [사용자 관리형 정책 생성과 연결](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_managed-policies.html)
- [브라우저 로그인을 이용한 CLI 인증](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html)
