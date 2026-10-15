## VPC - Virtual Private Cloud
가상 사설 클라우드. AWS 안에서 논리적으로 분리된 나만의 네트워크 공간입니다.

- Virtual(가상): 실제 장비를 소프트웨어로 나눠 사용하는 것
- Private(사설): 다른 사용자와 논리적으로 구분된 나만의 공간
- Cloud(클라우드): 클라우드 제공자의 컴퓨터·네트워크 등을 빌려 사용하는 환경

즉, VPC는 AWS 안에 만드는 나만의 네트워크 공간입니다.

### 학교 컴퓨터실로 생각해 보면

컴퓨터실에는 여러 PC가 있고, 같은 네트워크에 연결되어 있습니다. 각 PC에는 IP 주소가 있고, 인터넷에 나가는 통로도 있습니다.

AWS에서는 이런 네트워크 환경을 VPC로 구성합니다.

| 학교 컴퓨터실 네트워크         | AWS                |
|------------------------|---------------------|
| 컴퓨터들이 연결된 네트워크     | VPC                |
| 네트워크를 나눈 구역          | Subnet             |
| 각각의 컴퓨터                | EC2                |

예를 들어 VPC의 IP 범위를 10.0.0.0/16으로 정하면, 그 범위 안에서 서버들이 사용할 사설 IP를 배정할 수 있습니다. 여기서 /16은 이
네트워크에 포함되는 IP 주소 범위를 나타내는 표기입니다.

Private라고 해서 인터넷에 연결할 수 없다는 뜻은 아닙니다. 내가 설정한 경로와 접근 규칙에 따라 인터넷 연결 여부가 결정됩니다.


## EC2 - Elastic Compute Cloud
필요에 따라 컴퓨팅 자원을 늘리거나 줄일 수 있는 가상 서버 서비스입니다. Compute와 Cloud, 두 개의 C를 줄여 C2라고 표기합니다.

- Elastic(탄력적인): 필요에 따라 자원 규모를 늘리거나 줄일 수 있다는 뜻
- Compute(컴퓨팅): CPU와 메모리를 사용해 프로그램을 실행하는 것
- Cloud(클라우드): AWS의 자원을 빌려 사용하는 환경

Elastic의 E, Compute와 Cloud의 C 두 개를 줄여서 EC2라고 부릅니다.

즉, EC2는 AWS에서 가상 컴퓨터를 빌려 사용하는 서비스입니다. 실제로 생성한 가상 컴퓨터 한 대를 EC2 인스턴스라고 합니다.

### 내 컴퓨터와 비교하면

| 내 컴퓨터                  | EC2 인스턴스                                 |
|---------------------------|----------------------------------------------|
| CPU와 RAM 사양을 고름      | 인스턴스 유형을 고름                         |
| Windows나 Linux를 설치     | 운영체제가 포함된 이미지인 AMI를 고름         |
| SSD에 파일을 저장          | EBS 같은 저장장치에 파일을 저장              |
| 터미널에서 프로그램 실행   | SSH로 원격 접속해 프로그램 실행              |

이번 과제에서는 EC2에 Linux를 설치하고, Nginx라는 웹 서버 프로그램을 실행합니다.

브라우저가 웹페이지를 요청하면 Nginx가 요청을 받아 HTML 등의 내용을 돌려주는 것입니다.

참고로 Elastic이라고 해서 서버가 저절로 커지는 것은 아닙니다. 직접 사양을 바꾸거나, 별도의 자동 확장 설정을 해야 합니다.

## 서브넷 - Subnet

**VPC의 IP 주소 범위를 나눈 구역**입니다. EC2 같은 리소스를 서브넷에 배치하면 해당 구역의 사설 IP를 사용할 수 있습니다. 학교 컴퓨터실의 네트워크를 용도별 구역으로 나누는 것과 비슷합니다.

서브넷 하나는 하나의 가용 영역(Availability Zone, AZ)에 속합니다. 가용 영역은 같은 AWS 리전 안에서 장애 영향을 분리하기 위한 인프라 구역입니다. 여러 가용 영역에 서버를 배치하려면 각 영역에 서브넷을 만들어야 합니다. [AWS 서브넷 개념](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)

### VPC와 서브넷의 IP 범위

이번 실습의 주소 구성을 보면 관계를 이해하기 쉽습니다.

| 구분 | 실습 설정 | 의미 |
|---|---|---|
| VPC | `10.0.0.0/16` | `10.0.0.0`부터 `10.0.255.255`까지의 주소 범위 |
| 서브넷 | `10.0.1.0/24` | VPC 범위 안의 `10.0.1.0`부터 `10.0.1.255`까지 |
| 서브넷의 가용 영역 | `ap-northeast-2c` | 서울 리전 안에서 이 서브넷이 속한 영역 |
| EC2 사설 IP | `10.0.1.80` | 서브넷 범위 안에서 EC2에 할당된 주소 |

이렇게 IP 주소 뒤에 `/16`, `/24`를 붙여 범위를 나타내는 방식을 **CIDR**이라고 합니다. IPv4 주소는 32비트이며 `/24`는 앞의 24비트를 네트워크 구분에 사용한다는 뜻입니다. 나머지 8비트로 `2^8 = 256`개의 주소를 표현할 수 있습니다.

이번과 같은 일반적인 AWS IPv4 서브넷에서는 처음 네 주소와 마지막 주소를 AWS가 예약합니다. 따라서 `10.0.1.0/24`에서 리소스에 할당 가능한 범위는 `10.0.1.4`부터 `10.0.1.254`까지, 총 251개입니다. 이는 주소 개수이며 EC2 생성 가능 대수를 보장하는 값은 아닙니다.

서브넷의 주소 범위는 VPC 범위 안에 있어야 하며, 같은 VPC의 다른 서브넷과 겹칠 수 없습니다. [AWS 서브넷 CIDR](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html)

### Public Subnet과 Private Subnet

두 종류는 서브넷에 적용되는 라우팅으로 구분합니다.

| 구분 | 인터넷 게이트웨이로 직접 가는 경로 | 예시 용도 |
|---|---|---|
| Public Subnet | 있음 | 외부에 공개할 웹 서버 배치 |
| Private Subnet | 없음 | 외부에 직접 공개하지 않을 내부 서버 배치 |

서브넷 이름을 `public`으로 지정하거나 퍼블릭 IP 자동 할당만 켜는 것으로는 Public Subnet이 되지 않습니다. 연결된 라우트 테이블에 인터넷 게이트웨이로 향하는 경로가 있어야 합니다.

Private Subnet에서도 IPv4 통신에 NAT 장치 등을 구성하면 서버가 먼저 인터넷으로 요청을 보낼 수 있습니다. 이번 실습은 Public Subnet 한 개를 사용했습니다. [AWS 서브넷 종류](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-types)

## 인터넷 게이트웨이 - Internet Gateway

인터넷 게이트웨이(IGW)는 **VPC와 인터넷 사이의 통신을 가능하게 하는 구성 요소**입니다. VPC에 연결하고, 라우트 테이블에서 인터넷 방향의 대상으로 지정합니다.

학교 컴퓨터실에 비유하면 인터넷으로 연결되는 출입구에 해당합니다. 이 출입구를 사용할 트래픽은 라우팅 설정으로 정합니다.

### IPv4에서 퍼블릭 IP와 사설 IP의 관계

이번 실습의 EC2에는 사설 IP `10.0.1.80`과 퍼블릭 IP `52.79.233.23`이 할당되어 있었습니다.

- 인터넷의 클라이언트는 퍼블릭 IP로 요청합니다.
- IGW는 EC2의 퍼블릭 IPv4와 사설 IPv4 사이의 주소 변환을 수행합니다.
- EC2의 운영체제에서는 사설 IP를 사용하고, 인터넷에서는 연결된 퍼블릭 IP로 통신합니다.

이 설명은 배포 당시 주소를 사용한 예시입니다. 현재는 EC2 종료로 퍼블릭 IP가 반환된 상태입니다. [AWS 인터넷 게이트웨이와 주소 변환](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#InternetGateway-NAT)

### IGW를 연결하면 바로 인터넷에 접속할 수 있는가?

이번처럼 EC2가 IGW를 통해 직접 IPv4 인터넷 통신을 하려면 다음 조건이 함께 필요합니다.

1. IGW가 EC2의 VPC에 연결되어 있어야 합니다.
2. EC2의 서브넷에 적용되는 라우트 테이블에 `0.0.0.0/0 → IGW` 같은 인터넷 경로가 있어야 합니다.
3. EC2에 퍼블릭 IPv4 또는 Elastic IP가 있어야 합니다.
4. 보안 그룹과 네트워크 ACL 등에서 필요한 통신을 허용해야 합니다.

외부에서 웹 서비스를 이용하려면 이 네트워크 조건에 더해 Nginx가 요청을 받을 포트에서 실행 중이어야 합니다. [AWS 인터넷 연결 조건](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html), [서브넷 보안](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html#subnet-security)

## 라우트 테이블 - Route Table

**목적지 IP에 따라 트래픽을 어디로 보낼지 정하는 경로 규칙의 모음**입니다. 학교 컴퓨터실에서 “교내 주소는 내부망으로, 인터넷 주소는 외부 출입구로” 안내하는 표지판에 비유할 수 있습니다.

라우트 테이블의 각 규칙을 **라우트(Route)**라고 하며, 기본적으로 다음 두 항목을 읽습니다.

| 항목 | 의미 | 예시 |
|---|---|---|
| 목적지(Destination) | 트래픽이 향하는 IP 주소 범위 | `10.0.0.0/16`, `0.0.0.0/0` |
| 대상(Target) | 그 목적지로 보내기 위해 사용할 경로 | `local`, 인터넷 게이트웨이 |

라우트 테이블은 VPC 라우터가 참조하는 설정입니다. 웹 요청을 받아 처리하는 서버나 프록시처럼 별도의 통신 지점으로 생각하면 안 됩니다. [AWS 라우트 테이블](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

### 이번 실습의 라우트 테이블 읽기

배포 당시 설정한 두 경로는 다음과 같습니다.

| 목적지 | 대상 | 의미 |
|---|---|---|
| `10.0.0.0/16` | `local` | VPC 내부 주소로 향하는 통신 경로 |
| `0.0.0.0/0` | `igw-071f5b0ea58be0c33` | 더 구체적인 경로에 해당하지 않는 IPv4 트래픽의 인터넷 경로 |

`local`은 VPC 내부 통신을 위한 경로입니다. EC2 자기 자신을 가리키는 `localhost`와는 의미가 다릅니다. `0.0.0.0/0`은 모든 IPv4 주소를 포함하는 범위이며 기본 경로에 사용합니다. IPv6 전체 범위는 별도의 `::/0`으로 표현합니다. [서브넷 라우팅 규칙](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html#route-table-routes)

여러 경로에 동시에 해당하면 **더 구체적인 주소 범위의 경로**를 선택합니다. 이를 최장 접두사 일치(Longest prefix match)라고 합니다.

- 목적지가 `10.0.1.80`이면 `/16`과 `/0`에 모두 해당하지만, 더 구체적인 `/16`의 `local` 경로를 사용합니다.
- 일반적인 외부 웹 서버의 퍼블릭 IP는 위 `10.0.0.0/16`에 해당하지 않으므로 `/0`의 IGW 경로를 사용합니다.

따라서 `0.0.0.0/0 → IGW`가 있어도 VPC 내부 통신이 전부 인터넷으로 나가지는 않습니다. [AWS 경로 우선순위](https://docs.aws.amazon.com/vpc/latest/userguide/route-tables-priority.html)

### 서브넷과 라우트 테이블 연결

서브넷에는 한 번에 하나의 서브넷 라우트 테이블이 적용됩니다. 하나의 라우트 테이블을 여러 서브넷이 공유하는 것은 가능합니다.

- 별도로 지정하지 않은 서브넷에는 VPC의 **메인 라우트 테이블**이 적용됩니다.
- 특정 서브넷에 다른 경로를 적용하려면 원하는 라우트 테이블과 **명시적으로 연결(Association)**합니다.
- 라우트 테이블에 IGW 경로를 추가했더라도 해당 서브넷이 그 테이블을 사용하지 않으면 그 경로가 적용되지 않습니다. [AWS 서브넷 연결 규칙](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html)

이번 실습의 조회 결과에서는 인터넷 경로가 `active`, 서브넷 연결이 `associated`였습니다. 배포 구성은 [README](../README.md)에 정리되어 있습니다.

### 라우트 테이블과 보안 그룹의 차이

| 구분 | 결정하는 것 | 이번 실습 예시 |
|---|---|---|
| 라우트 테이블 | 목적지로 가는 경로 | `0.0.0.0/0 → IGW` |
| 보안 그룹 | 프로토콜·포트·출발지 또는 목적지에 따른 통신 허용 | TCP 80은 전체 IPv4에서, TCP 22는 개인 IP `/32`에서 허용 |

라우트 테이블의 `0.0.0.0/0`은 **목적지 범위**입니다. 보안 그룹의 HTTP 인바운드 규칙에서 사용하는 `0.0.0.0/0`은 **접속을 허용할 출발지 범위**입니다. 같은 표기라도 설정 위치에 따라 역할이 다릅니다.

인터넷 경로를 추가하는 것만으로 모든 포트가 열리지는 않습니다. 경로와 접근 허용 규칙이 함께 맞아야 원하는 통신이 가능합니다.

## VPC와 EC2를 함께 사용하기

VPC가 네트워크 환경을 제공하고, EC2가 그 안에서 실제 프로그램을 실행합니다.

이번 과제의 구조를 간단히 그리면 다음과 같습니다.

```text
사용자의 브라우저
       │
       │ HTTP 요청
       ▼
┌────────────── VPC ──────────────┐
│       Internet Gateway         │
│       인터넷 연결 통로          │
│                │               │
│   ┌──── Public Subnet ──────┐  │
│   │                         │  │
│   │  EC2                    │  │
│   │  └─ Linux               │  │
│   │     └─ Nginx 웹 서버    │  │
│   │                         │  │
│   └─────────────────────────┘  │
└────────────────────────────────┘
```

이 그림 외에도 요청이 도착하려면 라우팅 설정, EC2의 퍼블릭 IP, 보안 그룹의 접근 허용 규칙이 필요합니다.

## IAM - Identity and Access Management

AWS에서 **누가 어떤 리소스에 어떤 작업을 할 수 있는지** 관리하는 서비스입니다.

- Identity(신원): 요청을 보낸 사람이나 프로그램이 누구인지
- Access(접근): EC2 조회·생성·종료처럼 어떤 작업을 할 수 있는지
- Management(관리): 신원과 접근 규칙을 만들고 관리하는 것

### 인증과 권한 부여

학교 컴퓨터실에 들어갈 때 학생증으로 본인을 확인하고, 장비별 사용 허가를 확인한다고 생각하면 됩니다.

| 구분 | 확인하는 내용 | AWS 예시 |
|---|---|---|
| 인증(Authentication) | 누구인가? | 비밀번호·MFA 또는 임시 인증 정보로 신원 확인 |
| 권한 부여(Authorization) | 이 작업을 해도 되는가? | 해당 신원으로 EC2를 생성할 수 있는지 정책 평가 |

로그인에 성공해도 EC2 생성이나 결제 내역 조회 권한은 없을 수 있습니다. MFA는 본인 확인을 강화하며, MFA 등록 자체가 서비스 사용 권한을 추가하지는 않습니다. [AWS IAM 소개](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html), [IAM 보안 권장 사항](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### 루트 사용자, IAM 사용자, 그룹, 역할

| 구분 | 의미 | 사용 예시 |
|---|---|---|
| 루트 사용자(Root user) | AWS 계정을 처음 만들 때 생기는 강력한 권한의 신원 | 루트 전용 계정 관리 작업 |
| IAM 사용자(User) | 계정 안에 생성하는 개별 신원 | 이번 실습의 `codyssey-lab` |
| IAM 사용자 그룹(Group) | 여러 사용자에게 공통 정책을 적용하기 위한 묶음 | 개발자 그룹에 같은 조회 정책 연결 |
| IAM 역할(Role) | 허용된 주체가 맡아 임시 인증 정보로 사용하는 신원 | EC2의 프로그램이 S3에 접근할 때 사용하는 역할 |

그룹은 소속 사용자들의 권한을 함께 관리하며, 그룹 자체로 로그인하지는 않습니다. 역할을 맡는 과정인 **AssumeRole** 등을 거치면 해당 역할의 임시 인증 정보를 사용합니다. [AWS IAM 사용자 그룹](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html), [AWS IAM 역할](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)

역할에는 다음 두 종류의 정책이 관여합니다.

- **신뢰 정책(Trust policy):** 누가 이 역할을 맡을 수 있는지 정합니다.
- **권한 정책(Permissions policy):** 이 역할을 맡은 뒤 어떤 AWS 작업을 할 수 있는지 정합니다.

예를 들어 EC2용 역할의 신뢰 정책은 EC2 서비스가 역할을 맡도록 허용하고, 권한 정책은 특정 S3 데이터 읽기를 허용할 수 있습니다. 이는 역할을 설명하기 위한 예시이며 이번 Nginx 실습에 추가한 구성은 아닙니다. [AWS IAM 역할](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)

AWS는 일상적인 작업에 루트 사용을 피하고, 사람은 IAM Identity Center 같은 로그인 연동으로, 프로그램은 역할로 임시 인증 정보를 사용하도록 권장합니다. 이번 실습에서는 별도의 IAM 사용자로 작업했습니다. [IAM 보안 권장 사항](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

## AWS 정책과 권한

**정책(Policy)은 접근 규칙을 적은 문서이고, 권한(Permission)은 그 규칙에 따라 허용되는 작업 범위입니다.** IAM 정책은 주로 JSON 형식으로 작성합니다.

예를 들어 “서울 리전의 EC2 인스턴스를 조회하도록 허용한다”는 규칙을 정책에 적으면, AWS는 요청할 때 그 정책과 다른 적용 정책들을 평가해 조회 허용 여부를 결정합니다. 콘솔·CLI·SDK 모두 AWS 작업을 요청하므로 권한 검사를 받습니다. [AWS 정책과 권한](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)

저장소의 JSON 파일은 정책 내용을 보관한 파일입니다. 실제 권한을 부여하려면 해당 내용을 AWS 정책으로 만들고 적절한 사용자·그룹·역할 등에 연결해야 합니다. 로컬 파일을 수정하는 것만으로 AWS 권한이 변경되지는 않습니다.

### 정책을 어디에 연결하는가?

| 구분 | 연결 위치 | 예시 |
|---|---|---|
| 자격 증명 기반 정책(Identity-based policy) | 사용자·그룹·역할 | `codyssey-lab`에 EC2 조회 권한 부여 |
| 리소스 기반 정책(Resource-based policy) | 해당 정책을 지원하는 리소스 | S3 버킷에서 접근할 주체 지정 |

리소스 기반 정책의 `Principal`은 접근을 허용하거나 거부할 주체를 지정합니다. 사용자에게 붙이는 자격 증명 기반 정책은 연결 대상이 정해져 있으므로 `Principal`을 넣지 않습니다.

자격 증명 기반 정책은 관리 방식으로도 나눕니다.

| 구분 | 관리 방식 |
|---|---|
| AWS 관리형 정책 | AWS가 만들고 관리하며, 여러 사용자·그룹·역할에 연결 가능 |
| 고객 관리형 정책 | 내 AWS 계정에서 직접 만들고 관리하며, 여러 대상에 연결 가능 |
| 인라인 정책 | 특정 사용자·그룹·역할에 직접 포함되는 정책 |

이번 실습의 `CodysseyLabInfrastructure`는 고객 관리형 정책입니다. “AWS 관리형”이라는 이름만으로 이번 작업에 필요한 권한만 들어 있다고 판단할 수는 없습니다. 실제 허용 작업과 리소스 범위를 읽어야 합니다. [AWS 정책 유형](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#access_policy-types)

### JSON 정책 읽는 법

아래는 서울 리전의 EC2 인스턴스 조회만 허용하는 학습용 예시입니다. 실습에 사용한 전체 정책은 [lab-policy.json](../infra/iam/lab-policy.json)에 있습니다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadInstancesInSeoul",
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        }
      }
    }
  ]
}
```

| 항목 | 읽는 방법 |
|---|---|
| `Version` | 정책 문법의 버전. 정책을 만든 날짜가 아님 |
| `Statement` | 규칙들의 목록 |
| `Sid` | 규칙을 구분하기 위한 선택적 이름 |
| `Effect` | `Allow`는 허용, `Deny`는 명시적 거부 |
| `Action` | 허용·거부할 AWS 작업. `ec2:DescribeInstances`는 인스턴스 조회 |
| `Resource` | 규칙이 적용될 리소스 범위. `*`는 개별 리소스로 범위를 좁히지 않음 |
| `Condition` | 규칙이 적용될 조건. 예시는 요청 리전이 서울인 경우 |

각 필드의 문법은 [IAM 정책 요소](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html)에 정의되어 있습니다. 특히 `Version`은 정책 문법 버전을 지정하며, 정책 변경 이력을 관리하는 버전 번호와도 구분됩니다. [Version 정의](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_version.html)

리소스를 구체적으로 지정할 때는 ARN(Amazon Resource Name)이라는 AWS 리소스 식별 문자열을 사용합니다. 다만 작업마다 지원하는 리소스 지정 방식이 달라, 모든 작업을 개별 ARN으로 제한할 수 있는 것은 아닙니다. [리소스 지정 방식](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_resource.html)

`Resource: "*"`가 있어도 이 예시가 허용하는 작업은 `Action`에 적힌 인스턴스 조회입니다. 생성·종료 권한까지 포함하는 것은 아닙니다. `Action`, `Resource`, `Condition`을 함께 읽어야 범위를 알 수 있습니다.

### 여러 정책이 있으면 권한은 어떻게 결정되는가?

IAM 사용자·역할의 요청을 이해할 때 기본 원칙은 다음과 같습니다.

1. 기본적으로 허용되지 않은 작업은 거부됩니다. 이를 **암묵적 거부(Implicit deny)**라고 합니다.
2. 요청의 작업·리소스·조건에 맞는 `Allow`가 있고 적용되는 다른 제한도 충족하면 허용됩니다.
3. 적용되는 정책에 명시적인 `Deny`가 있으면 다른 정책의 `Allow`보다 우선합니다.

예를 들어 한 정책이 EC2 조회만 허용하고 다른 정책이 EC2 종료를 허용하면, 첫 정책에 종료 작업이 없다는 이유만으로 종료가 거부되지는 않습니다. 반면 적용되는 정책에 종료 작업의 명시적 `Deny`가 있으면 종료할 수 없습니다.

권한 경계(Permissions boundary)나 조직의 서비스 제어 정책(SCP) 같은 제한이 있으면 그것도 함께 평가합니다. 이런 제한 정책은 그 자체로 작업 권한을 부여하지 않습니다. 따라서 정책 파일 하나의 `Allow`만 보고 계정의 실제 권한 전체를 판단하면 안 됩니다. [AWS 정책 평가 방식](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)

## 이번 실습에서의 최소권한

최소권한은 **지금 해야 하는 작업에 필요한 범위만 허용하는 것**입니다. 작업 종류뿐 아니라 대상 리소스와 적용 조건도 가능한 범위에서 좁힙니다. [IAM 최소권한 원칙](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)

[실습 정책](../infra/iam/lab-policy.json)의 내용을 읽으면 다음과 같습니다.

| 정책 내용 | 의미 |
|---|---|
| `ReadLabInfrastructureInSeoul` | EC2·VPC·Subnet·보안 그룹·볼륨 등의 조회 작업 허용 |
| `ManageLabInfrastructureInSeoul` | 실습에 필요한 생성·연결·태그·삭제 작업 허용 |
| `aws:RequestedRegion = ap-northeast-2` | 이 정책의 허용 규칙이 서울 리전 요청에 적용 |
| `Resource: "*"` | 특정 실습 리소스 ID나 태그로 대상 범위를 제한하지 않음 |
| IAM 관리·S3·RDS·Billing 작업 없음 | 이 정책은 해당 작업의 권한을 부여하지 않음 |

이 정책은 작업 종류와 리전을 제한하지만, 같은 계정의 서울 리전에 있는 다른 EC2/VPC 리소스에도 허용 작업이 적용될 수 있습니다. 또한 인스턴스 한 대, `t3.micro`, EBS 8GiB, SSH 소스 `/32` 같은 실습 조건을 정책 자체가 강제하지는 않습니다.

다른 정책을 추가하면 실제 권한도 달라질 수 있습니다. 특히 이 파일의 서울 리전 조건이 다른 정책의 허용까지 제한하는 것은 아닙니다. 관리자 권한을 추가해 오류를 없애기보다는, 거부된 작업과 실제 필요한 범위를 확인해야 합니다.

### IAM, CLI 프로필, 보안 그룹, SSH 키의 차이

| 항목 | 담당하는 것 | 이번 실습 예시 |
|---|---|---|
| IAM 사용자·정책 | AWS API에서 누가 어떤 작업을 할 수 있는지 | `codyssey-lab`의 EC2 생성·종료 권한 |
| CLI 프로필 | 로컬 CLI가 사용할 인증·리전 등의 설정 묶음 선택 | `--profile codyssey` |
| 보안 그룹 | EC2로 들어오고 나가는 네트워크 트래픽 허용 | TCP 80 공개, TCP 22는 개인 IP `/32` |
| SSH 키와 Linux 사용자 | 서버 운영체제의 SSH 로그인 인증 | 개인 키로 `ec2-user`에 로그인 |

CLI 프로필은 사용할 설정을 고르는 이름이므로 이름을 바꾸어도 권한이 늘어나지는 않습니다. [AWS CLI 프로필](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

IAM에서 EC2 조회를 허용해도 SSH의 네트워크 경로나 로그인 인증이 해결되지는 않습니다. 또한 브라우저의 일반적인 Nginx `/health` 요청은 서버의 웹 요청 처리와 네트워크 설정을 따릅니다.

### Billing 조회 권한으로 이해하기

EC2를 생성할 수 있어도 결제 내역을 볼 수 없는 이유는 **EC2 관리 작업과 Billing 조회 작업의 권한이 별개이기 때문**입니다. 실습 정책에는 Billing 권한이 포함되어 있지 않습니다.

청구서·크레딧 등 Billing 콘솔의 해당 화면에 IAM 사용자나 역할로 접근하려면 계정의 IAM Billing 접근 활성화와 필요한 조회 정책이 모두 갖춰져야 합니다. IAM 접근 활성화는 루트 사용자가 수행하는 계정 설정이며, 활성화만으로 개별 사용자에게 조회 권한이 생기지는 않습니다. [AWS Billing 접근 권한](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html)
