# amazon-r53-resolver-dns-firewall-security-best-practices

Amazon Route 53 Resolver DNS 방화벽을 사용해 VPC 에 대한 아웃바운드 DNS 요청을 필터링하고 제어할 수 있다.
이를 통해 명시적으로 신뢰하는 도메인을 제외한 모든 도메인에 대해 액세스를 거부할 수 있다.

자세한 내용은 [Route 53 Resolver DNS Firewall 공식 문서](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver-dns-firewall.html) 에서 확인할 수 있다.

여기서는 DNS 방화벽을 잘 운영하기 위해 참고할 수 있는 정보에 대해 설명한다.

## Fail Open vs Fail Close

DNS 방화벽에서 문제가 발생해 정상적으로 해당 서비스를 사용할 수 없는 경우, DNS Query 허용(Fail Open) 또는 차단(Fail Close) 여부를 결정할 수 있다.

* *Fail Open* : 모든 쿼리를 허용한다. 가용성을 우선시한다.
* *Fail Close* : 모든 쿼리를 차단하고 `SERVFAIL` DNS 응답을 보낸다. 이 방식은 가용성보다 보안을 우선시한다.

기본 동작 방식은 Fail Close 이다. ***즉, DNS 방화벽에서 문제가 발생할 경우 연동된 VPC에 대한 모든 DNS 요청을 차단하기 때문에 서비스 전체의 장애를 야기할 수 있다.***

보안도 중요하지만 서비스 가용성에 문제가 생길 경우의 비즈니스 영향도가 훨씬 크다고 생각하기 때문에, ***DNS 방화벽을 사용하는 조직에서는 Fail Open 방식으로 사용하는 것을 권장한다.***
