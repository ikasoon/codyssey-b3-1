# 실습 리소스 정리 체크리스트

## 2026-10-16 평가용 재배포 — 정리 대기

평가용 웹 서버는 운영 중이다. 아래 항목은 평가 종료 후 실행하고 실제 삭제 결과를 확인한 다음 완료 표시한다.

- [ ] EC2 `i-0fcd16407d5f5ede8`의 `terminated` 확인
- [ ] 루트 EBS `vol-0975c0d2343cddc9a` 삭제 확인 (`DeleteOnTermination: true` 설정 확인됨)
- [ ] 추가 실습 미사용 EBS 볼륨 확인 및 정리
- [ ] 자동 할당 퍼블릭 IPv4 `54.180.96.168` 반환 확인 (`PublicIp: null`)
- [ ] 실습에 사용한 Elastic IP가 남아 있는지 확인 및 정리
- [ ] Route Table 연결 `rtbassoc-024a91303aba8e958` 해제
- [ ] Route Table `rtb-0a353ca4943b177b6` 삭제 확인
- [ ] Subnet `subnet-00f709c7a6e5b3b3e` 삭제 확인
- [ ] Security Group `sg-0a50708191321b557` 삭제 확인
- [ ] Internet Gateway `igw-02d48cd6b6b46bd3c` 연결 해제 및 삭제 확인
- [ ] VPC `vpc-0c3c0e059bd61e85e` 삭제 확인
- [ ] 키페어 `codyssey-eval` (`key-0e93a99ca48f480b9`)의 AWS 등록 삭제 확인
- [ ] 실습 관련 NAT Gateway, ELB/ALB, RDS의 해당 없음 확인

정리 순서는 EC2 종료 → EBS·퍼블릭 IP 확인 → Route Table 연결 해제·삭제 → Subnet·Security Group 삭제 → IGW 연결 해제·삭제 → VPC 삭제 → 키페어 등록 삭제다. NAT Gateway·ELB/ALB·RDS는 이번 배포의 생성 대상에 포함하지 않았다.

## 2026-10-15 최초 실습 — 정리 완료

아래는 최초 실습의 EC2 `i-0e9ec3bf1dddb9edf`와 VPC `vpc-02da8a937c3799359` 등 당시 리소스의 정리 결과다.

### 리소스와 완료 근거

- [x] EC2 `terminated` 확인
- [x] 루트 EBS 볼륨 `vol-0a6a1d56710174f89` 삭제 확인
- [x] 추가 실습 미사용 EBS 볼륨 없음
- [x] 자동 할당 퍼블릭 IPv4 반환 확인 (`terminated`, `PublicIp: null`)
- [x] Elastic IP 미할당 확인
- [x] Subnet과 Route Table 및 연결 삭제 확인
- [x] Security Group 삭제 확인
- [x] Internet Gateway 연결 해제 및 삭제 확인
- [x] VPC 삭제 확인
- [x] 키페어 등록 삭제 확인
- [x] NAT Gateway, ELB/ALB, RDS의 해당 없음 확인

### 수행한 정리 순서

| 순서 | 수행 내용 | 완료 근거 |
|---|---|---|
| 1 | EC2 종료 및 종료 대기 | `State: terminated`, `PublicIp: null` |
| 2 | 루트 EBS 삭제 여부 조회 | 해당 볼륨 ID 조회 `[]` |
| 3 | Route Table 연결 해제 및 삭제 | 해당 Route Table 조회 `[]` |
| 4 | Subnet 및 Security Group 삭제 | 각 리소스 조회 `[]` |
| 5 | Internet Gateway 연결 해제 및 삭제 | 해당 IGW 조회 `[]` |
| 6 | VPC 삭제 | 해당 VPC 조회 `[]` |
| 7 | AWS 키페어 등록 삭제 | 해당 키페어 이름 조회 `[]` |
