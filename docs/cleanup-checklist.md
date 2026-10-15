# 실습 리소스 정리 체크리스트

## 리소스와 완료 근거

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

## 수행한 정리 순서

| 순서 | 수행 내용 | 완료 근거 |
|---|---|---|
| 1 | EC2 종료 및 종료 대기 | `State: terminated`, `PublicIp: null` |
| 2 | 루트 EBS 삭제 여부 조회 | 해당 볼륨 ID 조회 `[]` |
| 3 | Route Table 연결 해제 및 삭제 | 해당 Route Table 조회 `[]` |
| 4 | Subnet 및 Security Group 삭제 | 각 리소스 조회 `[]` |
| 5 | Internet Gateway 연결 해제 및 삭제 | 해당 IGW 조회 `[]` |
| 6 | VPC 삭제 | 해당 VPC 조회 `[]` |
| 7 | AWS 키페어 등록 삭제 | 해당 키페어 이름 조회 `[]` |
