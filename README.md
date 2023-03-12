# amazon-vpc-starter

## Amazon VPC(Virtual Private Cloud)
### 개요
> AWS VPC 는 클라우드의 가상의 `데이터 센터`라고 생각하면 된다.

### 특징
> 하나의 VPC 는 하나의 Region 에 국한된다.

---

## Subnet
> 서버 중 내부망 안에서만 접속이 가능해야하는 서버들과 내부망과 외부망 모두에서 접속이 가능해야하는 서버들이 존재한다.  
> 내부망 안에서만 접속이 가능해야하는 그룹은 private 세그먼트으로 묶으며 내부망 및 외부망 모두에서 접속이 가능해야하는 서버들은
> public 세그먼트로 묶는다. 여기서 세그먼트는 네트워크 용어로 subnet(서브넷)으로 불린다.  
> 즉, 네트워크에서는 private 서브넷과 public 서브넷으로 구분할 수 있다.

### 특징
> subnet 은 VPC 의 CIDR block 안에서만 CIDR block 을 설정할 수 있다.

### Public subnet
> Public subnet 의 외부 트래픽은 IGW(Internet GateWay) 를 통해서 외부로 나간다.

### Private subnet
> Private subnet 의 외부 트래픽은 NAT(Network Address Translation) Gateway 를 통해서 외부로 나간다.

### CIDR
> `CIDR` 는 할당할 수 있는 IP Range 를 나타낸다.  
> 예시) 10.0.0.0/16 인 CIDR 가 있으면 마지막 숫자(16)을 8로 나눈 몫(2)으로 IP Address 왼쪽 앞부터 . 기준 2개를 고정한다.
> 여기서는 . 기준 2개는 10, 0 이며 이 2자리는 고정하고 뒤의 2개의 자리는 범위로 사용한다.
> 범위: 10.0.0.0 ~ 10.0.255.255
>
> 예시 2) 10.0.1.1/32 이면 마지막 숫자(32)를 8로 나눈 몫 4 자리를 고정함으로 IP 범위는 10.0.1.1 한 개이다.

---

## Route tables
### 개요
> Subnet 은 하나의 Route table 에만 연결할 수 있지만, Route table 은 여러개의 subnet 을 연결할 수 있다.  
> 기본 Route table 이 존재하지만 해당 라우트 테이블을 사용하지 않고 신규로 생성해서 사용하는 것을 권장한다.

---

## IGWs(Internet GateWay)
### 개요
> Public subnet 에 포함된 인스턴스가 인터넷과 커뮤니케이션하기 위한 point.  
> Public subnet 에서 사용하는 라우팅 테이블에 인터넷(일반적으로 0.0.0.0/0) - IGW 라우팅을 추가해야
> 외부에서 Public subnet 에 포함된 인스턴스에 SSH 접근과 같이 접근이 가능해진다.
>
> IGW 는 가용성을 위해서 각 AZ 마다 IGW 를 만들 필요가 없다.

---

## NATs
### 개요
> Private subnet 에 포함된 인스턴스가 인터넷과 커뮤니케이션하기 위한 point.  
> 종류는 2가지로 NAT gateway, NAT instance 로 나뉜다.  
> NAT 서비스는 IPv4 네트워크 트래픽만 지원하며, 인스턴스와 IGW 사이의 중간 point 역할을 한다.  
> NAT gateway 및 NAT instance 는 public subnet 에 배포된다.
> Private subnet 에 포함된 인스턴스의 트래픽을 인터넷으로 라우팅하기 위해서는 Private subnet 에서 사용하는 라우팅 테이블에
> NAT device - 인터넷(일반적으로 0.0.0.0/0) 라우팅을 추가한다.
>
> NAT gateway 가 배포된 AZ 가 fail 되서 통신이 멈추게 되면 해당 NAT gateway 를 통해서 인터넷과 연결되는 private subnet 의
> 인스턴스들의 인터넷 연결이 중단되게 됨으로 NAT gateway 를 멀티 AZ 에 배포 및
> NAT gateway 가 배포된 AZ 마다 라우팅 테이블을 생성하여 NAT-gateway 라우팅 추가를 통해서 고가용성을 유지 하도록 한다.

### NAT gateway 
> AWS 에서 제공하는 서비스    
> NAT gateway 는 Elastic IP(고정 아이피)가 필수로 필요하다.  
 
### NAT instance
> NAT 서비스를 호스팅하는 EC2 instance

---

## DHCP
### 개요
> DHCP 는 네트워크 프로토콜로 인스턴스에게 Subnet 의 CIDR 블록에 따라 IP 주소를 동적으로 부여하기 위해서 사용된다.   
> DHCP 옵션 세트는 DHCP 의 Configuration 이다. 여러 항목 중 알아야할 항목은 다음과 같다.    
> *도메인 이름(domain-name): 인스턴스에 부여되는 도메인 이름 규칙 지정  
> *DNS(domain-name-servers): DNS 서버 지정  
> *ntp-servers: Network Time Protocol(NTP) 서버 지정.

---

## VPC Peering
### 개요
> 2개의 VPC 를 연결하는 서비스

---

## VPC Endpoint
### 개요
> AWS Service 와 EC2 연결 시 외부망(IGW, NAT) 을 통해서 연결하지 않고 내부망 통신을 가능하다록 해주는 서비스.    
> S3, DynamoDB, RDS 와 같은 AWS Service 에 대한 VPC Endpoint 를 사용하여 private EC2 인스턴스와
> 외부망을 거치지 않고 내부망 통신을 가능하게 할 수 있다.  

### 비용
> 2023.03.12 기준 시간당 0.013 USD 의 비용이 청구된다.  
> 참조사이트: [AWS PrivateLink 요금](https://aws.amazon.com/ko/privatelink/pricing/)  
