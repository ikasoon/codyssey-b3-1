# 실습 리소스 정리 체크리스트

**상태: 정리 미완료.** 아래는 학습자가 직접 수행할 절차다. 현재 확보된 마지막 EC2 조회 결과는 `running`이며, 종료·삭제 결과는 아직 제공되지 않았다. 실행 후 실제 확인한 항목만 체크하고 완료 시각과 출력을 기록한다.

## 정리 전 확인

- [x] 외부 접속 스크린샷 확보: [external-health.png](evidence/external-health.png)
- [x] 생성한 주요 리소스 ID 기록: 아래 목록
- [ ] SSH 접속·내부 응답·아웃바운드 검증 결과 저장
- [ ] 서버에서 보관할 설정·로그가 있다면 로컬에 저장
- [ ] 종료 대상이 이 실습의 EC2와 볼륨인지 확인

EC2 종료는 되돌릴 수 없으며 `DeleteOnTermination=true`인 볼륨의 데이터도 삭제된다. 증빙을 먼저 저장한다. [AWS EC2 종료 안내](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html)

## 리소스와 완료 근거

- [ ] EC2 `terminated` 확인
- [ ] EBS 볼륨과 실습에서 생성한 미사용 볼륨 삭제 확인
- [ ] 자동 할당 퍼블릭 IPv4 반환 및 Elastic IP 미할당 또는 Release 확인
- [ ] Subnet과 Route Table 및 연결 삭제 확인
- [ ] Security Group 삭제 확인
- [ ] Internet Gateway 연결 해제 및 삭제 확인
- [ ] VPC 삭제 확인
- [ ] 키페어 등록 삭제 확인
- [ ] NAT Gateway, ELB/ALB, RDS의 해당 없음 또는 삭제 확인
- [ ] Billing 내역과 잔여 리소스 확인

| 완료 | 리소스 | 식별자 | 완료 조건 | 시각과 실제 근거 |
|---|---|---|---|---|
| [ ] | EC2 | `i-0e9ec3bf1dddb9edf` | `terminated` 확인 | 미기록 |
| [ ] | EBS | `vol-0a6a1d56710174f89` | 볼륨 ID 조회 결과 없음 | 미기록 |
| [ ] | 추가 미사용 EBS | 실습 관련 볼륨 | 실습에서 생성한 잔여 볼륨 없음 | 미확인 |
| [ ] | 퍼블릭 IPv4 | 검증 당시 `52.79.233.23` | EC2 종료로 자동 할당 주소 반환 확인 | 미확인 |
| [ ] | Elastic IP | 생성 절차에 없음 | 미할당 확인 또는 할당한 실습 주소 Release | 미확인 |
| [ ] | Route Table 연결 | `rtbassoc-0233fcaadb0d0a795` | 연결 해제 | 미기록 |
| [ ] | Route Table | `rtb-06bc2d70e1478de6d` | 삭제 후 조회 결과 없음 | 미기록 |
| [ ] | Public Subnet | `subnet-0994c241f09fecbb1` | 삭제 후 조회 결과 없음 | 미기록 |
| [ ] | Security Group | `sg-027c54d832f751fd6` | 삭제 후 조회 결과 없음 | 미기록 |
| [ ] | Internet Gateway | `igw-071f5b0ea58be0c33` | 연결 해제 및 삭제 후 조회 결과 없음 | 미기록 |
| [ ] | VPC | `vpc-02da8a937c3799359` | 삭제 후 조회 결과 없음 | 미기록 |
| [ ] | 키페어 | `codyssey-lab` / `key-0f8f3fb1511d18c5b` | AWS 등록 삭제 | 미기록 |
| [ ] | NAT Gateway | 생성 절차에 없음 | 해당 없음 확인 또는 실습 리소스 삭제 | 미확인 |
| [ ] | ELB/ALB | 생성 절차에 없음 | 해당 없음 확인 또는 실습 리소스 삭제 | 미확인 |
| [ ] | RDS | 생성 절차에 없음 | 해당 없음 확인 또는 실습 리소스 삭제 | 미확인 |
| [ ] | Billing | 계정 비용·크레딧 | 잔여 과금 리소스와 사용 내역 확인 | 미확인 |

"생성 절차에 없음"은 계정 전체에 해당 리소스가 없음을 조회한 결과가 아니다. IAM 사용자·정책은 별도의 계정 관리 대상으로, 실습 전용이고 더 이상 필요 없다면 적절한 권한을 가진 담당자가 삭제한다. 개인 키는 재사용 여부를 확인한 뒤 본인이 로컬에서 정리한다.

## 실행 원칙

아래 명령은 로컬 터미널에서 한 단계씩 직접 실행한다. 리전은 서울, 프로필은 `codyssey`다. 오류가 나면 다음 삭제로 넘어가지 말고 원인을 확인한다. 삭제 명령이 출력 없이 끝났다는 사실만으로 완료를 판정하지 않고 마지막 조회 결과를 함께 남긴다.

EC2 종료 후 볼륨과 네트워크 의존성을 해제하고 VPC를 마지막에 삭제한다. 기본 Security Group, 기본 Route Table, 기본 Network ACL은 VPC 삭제 시 함께 제거되므로 별도로 삭제하지 않는다. [AWS VPC 삭제 안내](https://docs.aws.amazon.com/vpc/latest/userguide/delete-vpc.html)

## 1 EC2 종료

종료 전 대상과 볼륨 삭제 설정을 확인한다.

```bash
aws ec2 describe-instances \
  --profile codyssey --region ap-northeast-2 \
  --instance-ids i-0e9ec3bf1dddb9edf \
  --query 'Reservations[].Instances[].{
    Id:InstanceId,
    State:State.Name,
    PublicIp:PublicIpAddress,
    Volumes:BlockDeviceMappings[].Ebs
  }' --output json
```

증빙 저장과 대상 확인을 마쳤을 때 종료한다.

```bash
aws ec2 terminate-instances \
  --profile codyssey --region ap-northeast-2 \
  --instance-ids i-0e9ec3bf1dddb9edf
```

```bash
aws ec2 wait instance-terminated \
  --profile codyssey --region ap-northeast-2 \
  --instance-ids i-0e9ec3bf1dddb9edf
```

```bash
aws ec2 describe-instances \
  --profile codyssey --region ap-northeast-2 \
  --instance-ids i-0e9ec3bf1dddb9edf \
  --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name,PublicIp:PublicIpAddress}' \
  --output json
```

완료 기준: `State`가 `terminated`. 오래 지나면 종료한 인스턴스가 목록에서 사라질 수 있으므로 종료 직후 결과를 보관한다. 자동 할당 퍼블릭 IPv4는 종료 시 반환되며 Elastic IP와 달리 별도의 Release 명령을 사용하지 않는다.

## 2 EBS와 Elastic IP 확인

루트 볼륨이 자동 삭제됐는지 확인한다.

```bash
aws ec2 describe-volumes \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=volume-id,Values=vol-0a6a1d56710174f89 \
  --query 'Volumes[].{Id:VolumeId,State:State,Attachments:Attachments}' \
  --output json
```

완료 기준: 성공한 조회의 결과가 `[]`. 볼륨이 남았다면 EC2 종료와 연결 해제가 완료되어 `available`인지 먼저 확인한다. 해당 실습 볼륨이 남은 경우에만 다음 명령으로 삭제하고 다시 조회한다.

```bash
aws ec2 delete-volume \
  --profile codyssey --region ap-northeast-2 \
  --volume-id vol-0a6a1d56710174f89
```

추가 미사용 볼륨과 Elastic IP를 조회한다.

```bash
aws ec2 describe-volumes \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=status,Values=available \
  --query 'Volumes[].{Id:VolumeId,Size:Size,Tags:Tags}' \
  --output json
```

```bash
aws ec2 describe-addresses \
  --profile codyssey --region ap-northeast-2 \
  --query 'Addresses[].{AllocationId:AllocationId,PublicIp:PublicIp,Tags:Tags}' \
  --output json
```

위 명령은 서울 리전의 계정 리소스를 조회한다. 결과가 있으면 이 실습에서 생성한 것인지 식별하고 관련 리소스만 정리한다. 본 실습의 생성 절차에는 Elastic IP 할당이 없으므로 자동 할당 주소 `52.79.233.23` 자체를 Elastic IP로 해제하지 않는다. 별도 실습 Elastic IP가 발견되면 현재 정책에는 Release 권한이 없으므로 담당 권한으로 연결 해제·Release를 수행하고 근거를 남긴다. 권한 오류는 "리소스 없음"을 뜻하지 않는다.

## 3 Route Table 연결 해제와 삭제

```bash
aws ec2 disassociate-route-table \
  --profile codyssey --region ap-northeast-2 \
  --association-id rtbassoc-0233fcaadb0d0a795
```

```bash
aws ec2 delete-route-table \
  --profile codyssey --region ap-northeast-2 \
  --route-table-id rtb-06bc2d70e1478de6d
```

## 4 Subnet과 Security Group 삭제

```bash
aws ec2 delete-subnet \
  --profile codyssey --region ap-northeast-2 \
  --subnet-id subnet-0994c241f09fecbb1
```

```bash
aws ec2 delete-security-group \
  --profile codyssey --region ap-northeast-2 \
  --group-id sg-027c54d832f751fd6
```

`DependencyViolation`이 발생하면 EC2 종료 후 네트워크 인터페이스 해제가 끝났는지 확인한다. 아래 조회로 사용 대상을 확인하며, 관련 없는 인터페이스는 삭제하지 않는다.

```bash
aws ec2 describe-network-interfaces \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=vpc-id,Values=vpc-02da8a937c3799359 \
  --query 'NetworkInterfaces[].{Id:NetworkInterfaceId,Status:Status,Description:Description,Attachment:Attachment}' \
  --output json
```

## 5 Internet Gateway와 VPC 삭제

```bash
aws ec2 detach-internet-gateway \
  --profile codyssey --region ap-northeast-2 \
  --internet-gateway-id igw-071f5b0ea58be0c33 \
  --vpc-id vpc-02da8a937c3799359
```

```bash
aws ec2 delete-internet-gateway \
  --profile codyssey --region ap-northeast-2 \
  --internet-gateway-id igw-071f5b0ea58be0c33
```

```bash
aws ec2 delete-vpc \
  --profile codyssey --region ap-northeast-2 \
  --vpc-id vpc-02da8a937c3799359
```

## 6 키페어 등록 삭제

```bash
aws ec2 delete-key-pair \
  --profile codyssey --region ap-northeast-2 \
  --key-name codyssey-lab
```

AWS 키페어 등록 삭제는 로컬의 `~/.ssh/codyssey-lab.pem` 파일을 삭제하지 않는다.

## 7 삭제 후 최종 조회

아래 각 명령이 성공하고 `[]`를 반환하는지 확인한다. 인증 실패나 `AccessDenied`는 삭제 완료 근거가 아니다.

```bash
aws ec2 describe-route-tables \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=route-table-id,Values=rtb-06bc2d70e1478de6d \
  --query 'RouteTables[].RouteTableId' --output json
```

```bash
aws ec2 describe-subnets \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=subnet-id,Values=subnet-0994c241f09fecbb1 \
  --query 'Subnets[].SubnetId' --output json
```

```bash
aws ec2 describe-security-groups \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=group-id,Values=sg-027c54d832f751fd6 \
  --query 'SecurityGroups[].GroupId' --output json
```

```bash
aws ec2 describe-internet-gateways \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=internet-gateway-id,Values=igw-071f5b0ea58be0c33 \
  --query 'InternetGateways[].InternetGatewayId' --output json
```

```bash
aws ec2 describe-vpcs \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=vpc-id,Values=vpc-02da8a937c3799359 \
  --query 'Vpcs[].VpcId' --output json
```

```bash
aws ec2 describe-key-pairs \
  --profile codyssey --region ap-northeast-2 \
  --filters Name=key-name,Values=codyssey-lab \
  --query 'KeyPairs[].KeyPairId' --output json
```

## 8 추가 리소스와 Billing 확인

본 실습의 명령에는 NAT Gateway, ELB/ALB, RDS 생성이 없다. 별도로 생성한 실습 리소스가 없는지 확인하고, 있다면 해당 서비스에서 삭제한다. 현재 실습 정책은 ELB·RDS 조회 권한을 제공하지 않으므로 권한 있는 계정 관리 담당자가 확인한다.

비용 확인에는 EC2 실행 시간, EBS, 퍼블릭 IPv4, 데이터 전송과 별도 생성한 리소스를 포함한다. Billing에 이미 발생한 비용이 남아 있는 것과 아직 과금 리소스가 실행 중인 것은 구분한다. 크레딧 적용 내역과 잔여 리소스를 확인하며, 삭제 직후 표시만으로 최종 청구액을 단정하지 않는다.

### 완료 기록

- 정리 실행자: 미기록
- 종료·삭제 확인 시각과 시간대: 미기록
- 결과 저장 위치: 실행 후 `docs/evidence/cleanup-results.txt`에 기록
- Billing 확인 시각과 결과: 미기록
- 미정리 리소스 또는 예외: 미확인

기록에는 실제 실행 명령과 출력만 넣는다. 모든 항목의 완료 또는 해당 없음을 확인한 뒤 이 문서와 README의 상태를 갱신한다.
