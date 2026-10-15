# EC2 조회 명령의 줄바꿈 오류 해결

EC2 배포 후 접속 정보를 조회하는 과정에서 AWS CLI의 `--query` 문법 오류가 발생했다. `BlockDeviceMappings` 식별자 중간에 들어간 줄바꿈을 제거하고 다시 조회해 인스턴스 상태·IP·볼륨 ID를 확인했다.

이 사례는 로컬 CLI 조회식 오류다. 네트워크 장애나 IAM 권한 오류를 재현한 사례는 아니다.

## 증상

다음 조회식으로 `describe-instances`를 실행했을 때 JSON 결과 대신 파싱 오류가 출력됐다. 아래는 실제 입력에서 문제가 된 부분이다.

```text
Reservations[].Instances[].
  {InstanceId:InstanceId,State:State.Name,PublicIp:PublicIpAddress,PrivateIp:PrivateIpAddress,KeyName:KeyName,Volumes:BlockDeviceMappin
  gs[].Ebs.VolumeId}
```

실제 오류 메시지 핵심:

```text
aws: [ERROR]: An error occurred (ParamValidation): Bad value for --query
Expecting: colon, got: flatten: Parse error at column 168, token "[]" (FLATTEN)
```

## 원인 가설

복사 과정에서 `BlockDeviceMappings`가 `BlockDeviceMappin`과 `gs`로 나뉘어 서로 다른 토큰처럼 해석됐다고 판단했다. `--query`는 JMESPath 조회식이며, 필드 이름 중간의 실제 줄바꿈은 이름을 보존하지 않는다.

오류가 `--query`와 파싱 위치를 명시하므로 서버 장애나 보안 그룹을 먼저 수정할 근거는 없었다. [AWS CLI 출력 필터링 설명](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html)

## 검증

1. 오류가 가리킨 `gs[]` 앞부분과 원래 필드명 `BlockDeviceMappings`를 비교했다.
2. 인스턴스 ID와 AWS 설정을 유지하고 조회식의 줄바꿈만 수정했다.
3. 수정한 명령 실행 후 정상 JSON이 반환되는지 비교했다.

문제는 터미널의 시각적인 자동 줄바꿈이 아니라, 입력 문자열의 식별자 안에 들어간 실제 개행이었다. 필드 사이의 공백·줄바꿈과 식별자 내부의 개행을 구분했다.

## 조치

필드 하나를 한 줄에 배치해 식별자가 분리되지 않도록 수정했다. 다음은 같은 인스턴스에 대한 수정 명령이다. 재실행이 필요하면 학습자가 직접 수행한다.

```bash
aws ec2 describe-instances \
  --profile codyssey \
  --region ap-northeast-2 \
  --instance-ids i-0e9ec3bf1dddb9edf \
  --query 'Reservations[].Instances[].{
    InstanceId: InstanceId,
    State: State.Name,
    PublicIp: PublicIpAddress,
    PrivateIp: PrivateIpAddress,
    KeyName: KeyName,
    Volumes: BlockDeviceMappings[].Ebs.VolumeId
  }' \
  --output json
```

## 결과

수정 후 조회 결과:

```json
[
  {
    "InstanceId": "i-0e9ec3bf1dddb9edf",
    "State": "running",
    "PublicIp": "52.79.233.23",
    "PrivateIp": "10.0.1.80",
    "KeyName": "codyssey-lab",
    "Volumes": ["vol-0a6a1d56710174f89"]
  }
]
```

조회식 수정으로 오류가 해결됐다. EC2 재생성이나 네트워크·IAM 설정 변경은 해결 과정에 필요하지 않았다. 이후 외부 PC의 `/health` 요청에서도 HTTP 200과 `OK`를 확인했다. [외부 접속 스크린샷](evidence/external-health.png)

## 재발방지

- 긴 `--query`는 중괄호 안에서 필드별로 줄을 나누고 식별자 내부는 끊지 않는다.
- 복사한 명령에서 따옴표, 역슬래시, 필드 이름을 실행 전에 확인한다.
- `Bad value for --query`가 나오면 우선 조회식을 점검한다. 이후 오류가 바뀌는 경우에만 해당 오류에 맞춰 진단한다.
- 조회 실패를 리소스 생성 실패로 간주하지 않는다. 생성 명령을 반복하기 전에 기존 리소스 ID와 상태를 확인한다.
- 정상 동작한 조회식과 결과를 함께 보관해 다음 검증에서 재사용한다.
