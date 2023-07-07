# Amazon Route 53 Resolver DNS Firewall

Amazon Route 53 Resolver DNS 방화벽(이하 DNS 방화벽)을 사용해 VPC 에 대한 아웃바운드 DNS 요청을 필터링하고 제어할 수 있다.
이를 통해 명시적으로 신뢰하는 도메인을 제외한 모든 도메인에 대해 액세스를 거부할 수 있다.

여기에서는 DNS 방화벽 운영에 도움이 되는 정보를 제공하는 것이 목적이며, DNS 방화벽에 대한 자세한 내용은 [Route 53 Resolver DNS Firewall 공식 문서](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver-dns-firewall.html) 에서 확인할 수 있다. 


## Domain Rules

기본적으로 DNS 방화벽에서는 [AWS 관리형 도메인 규칙](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver-dns-firewall-managed-domain-lists.html) 및 [사용자 도메인 규칙](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver-dns-firewall-user-managed-domain-lists.html)을 정의할 수 있다.
그리고 정의된 규칙과 일치하는 DNS Query 요청에 대해 각 규칙에서는 아래 옵션 중 중 하나를 지정해야 한다.

* *Alert(탐지) : DNS Query 검사를 중지하고 통과하도록 허용한 다음 로그에 기록한다.*
* *Allow(허용) : DNS Query 검사를 중단하고 통과하도록 허용한다.*
* *Block(차단) : DNS Query 검사를 중지하고, 의도한 대상으로 이동하지 못하도록 차단하고 로그에 기록한다.*

AWS 관리형 도메인 규칙은 악의적인 활동이나 기타 잠재적 위협과 관련된 도메인을 포함한다.
다만, AWS 관리형 도메인 규칙은 정책 리스트를 제공하지 않는다. *그렇기 때문에 프로덕션 환경에 적용하기 전에 동작 모드를 Alert 으로 설정해 테스트하는 것을 권고한다.*


## Fail Open vs Fail Close

DNS 방화벽에서 문제가 발생해 정상적으로 해당 서비스를 사용할 수 없는 경우, DNS Query 허용(Fail Open) 또는 차단(Fail Close) 여부를 결정할 수 있으며, 설정은 Resolver > VPCs > "DNS 방화벽 실패 열기" 에서 설정할 수 있다.

* *Fail Open* : 모든 쿼리를 허용한다. 가용성을 우선시한다.
* *Fail Close* : 모든 쿼리를 차단하고 `SERVFAIL` DNS 응답을 보낸다. 이 방식은 가용성보다 보안을 우선시한다.

***DNS 방화벽 활성화 시, 기본 동작 방식은 Fail Close 이다. 이 말인 즉슨, DNS 방화벽에서 문제가 발생하면 연동된 VPC에 대한 모든 DNS 요청을 차단하기 때문에 서비스 전체의 장애를 초래할 수 있다.
그러므로, DNS 방화벽을 사용하는 조직에서는 Fail Open 방식으로 사용하는 것을 권장한다.***


## Sharing DNS Firewall rule groups between AWS accounts

아쉽게도 Amazon Route53 Resolver DNS 방화벽에서는 다른 서비스(ex. GuardDuty, CloudTrail 등)와 같이 Organizations 단위로 정책을 중앙 관리할 수 있는 기능을 제공하지 않는다.

DNS 방화벽 관리자를 사용해 조직 전체의 VPC에 적용하고 중앙에서 관리하기 위해서는 AWS Firewall Manager를 사용하거나, AWS Resource Access Manager(AWS RAM)를 사용해 규칙 그룹을 공유함으로써 중앙 관리할 수 있다.

자세한 내용은 [여기](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver-dns-firewall-rule-group-sharing.html) 에서 참고할 수 있다.
