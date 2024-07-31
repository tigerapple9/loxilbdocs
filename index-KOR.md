<link rel="stylesheet" href="stylesheets/extra.css" />

## [![loxilb 로고](photos/loxilb-logo-small.png)](https://github.com/loxilb-io/loxilb) loxilb 문서에 오신 것을 환영합니다

[![eBPF Emerging App](https://img.shields.io/badge/ebpf.io-Emerging--App-success)](https://ebpf.io/projects#loxilb) [![Go Report Card](https://goreportcard.com/badge/github.com/loxilb-io/loxilb)](https://goreportcard.com/report/github.com/loxilb-io/loxilb)  ![apache](https://img.shields.io/badge/license-Apache-blue.svg) ![gpl](https://img.shields.io/badge/license-BSD-blue.svg)  [![Stargazers][stars-shield]][stars-url]   
<!--[![eBPF Emerging Project](photos/ebpflogo.png)](https://ebpf.io/projects#loxilb)-->
   
[stars-shield]: https://img.shields.io/github/stars/loxilb-io??style=for-the-badge&logo=appveyor
[stars-url]: https://github.com/loxilb-io/loxilb/stargazers

## 배경 
loxilb는 엣지 클라우드 환경에서 클라우드 네이티브/쿠버네티스 워크로드 배포를 용이하게 하기 위한 프로젝트로 시작되었습니다. AWS/GCP와 같은 퍼블릭 클라우드에 서비스를 배포할 때, 서비스는 외부 세계에 쉽게 접근할 수 있습니다. 퍼블릭 클라우드는 일반적으로 클라이언트로부터 들어오는 요청을 이 서비스로 연결하기 위해 로드 밸런서 인스턴스를 기본적으로 제공합니다.

그러나 온프레미스 및 엣지 클라우드에서 배포하는 경우 기본적으로 *서비스 유형 - 외부 로드 밸런서* 을 제공하지 않습니다. 오랜 시간 동안 MetalLB가 유일한 방안 이었습니다. 그러나 엣지 클라우드 서비스는 GTP, SCTP, SRv6 등과 같은 다양한 프로토콜이 사용되기 때문에 완전히 다른 환경 입니다. 이와같이 모든 인프라 환경을 통합하는 솔루션을 만드는 것은 매우 어려습니다.

loxilb 개발 팀은 이러한 문제를 해결하고자 하는 많은 사람들로부터 개발 요청을 받았습니다. 문제를 해결하기 위한 첫 단계로, 리눅스 커널이 제공하는 네트워킹 스택은 매우 견고하지만 다양한 프로토콜과 Stateful 로드 밸런싱을 신속하게 지원하기 위한 개발 민첩성이 부족하다는 것을 확인 하였습니다. 이 과정에서 eBPF 라는 신규 리눅스 커널 기술을 발견하였고, OS 커널에서 안전한 샌드박스 프로그램으로 새로운 기능을 도입할 수 있는 유연성은 우리의 설계 철학에 완벽하게 맞아떨어졌습니다. 또한 전용 CPU 코어가 필요하지 않기 때문에 에너지 효율적인 엣지 아키텍처 설계에 이상적이었습니다. 

## loxilb란 ?

loxilb는 GoLang/eBPF 기반의 오픈 소스 클라우드 네이티브 로드 밸런서로, 온-프레미스, 퍼블릭 클라우드 또는 하이브리드 K8s 환경 전반에서의 호환성을 제공 합니다. loxilb는 텔코 클라우드, 모빌리티, 엣지 컴퓨팅에서 원할한 클라우드 네이티브 서비스를 지원하기 위해 개발되고 있습니다.

## 쿠버네티스와 loxilb

쿠버네티스는 pod 간, pod와 서비스 간, 외부와 서비스 간의 통신을 위해 ClusterIP, NodePort, LoadBalancer, Ingress 등 다양한 서비스 타입을 정의하고 있습니다.

![loxilb 커버](photos/loxilb-cover.png)

이 모든 서비스는 Layer4/Layer7에서 작동하는 로드 밸런서/프록시에 의해 제공됩니다. 쿠버네티스는 매우 모듈화되어 있기 때문에, 이러한 서비스는 다양한 소프트웨어 모듈에 의해 제공될 수 있습니다. 예를 들어, kube-proxy는 기본적으로 ClusterIP 및 NodePort 서비스를 제공하는 데 사용됩니다.

LoadBalancer 서비스 유형은 일반적으로 퍼블릭 클라우드 사업자가 관리하는 인스턴스로 제공됩니다. 그러나 온-프레미스 및 자가 관리 클러스터의 경우, 사용할 수 있는 좋은 옵션이 몇 가지 있습니다. EKS와 같은 매니지드 K8s 서비스 에서도 사용자 정의형 로드 밸런서를 클러스터에서 사용하고자 하는 사람들이 많이 있습니다. <b>loxilb는 LoadBalancer 서비스 유형의 주요 사용 사례로 제공합니다</b>. loxilb는 사용자 필요에 따라 클러스터 내 또는 클러스터 외부에서 실행할 수 있습니다.

loxilb는 기본적으로 L4 로드 밸런서/서비스 프록시로 작동합니다. L4 로드 밸런싱은 훌륭한 성능과 기능을 제공하지만, 때로는 다양한 사용 사례에 대해 K8s에서 동등한 성능의 L7 로드 밸런서가 필요합니다. 또한 loxilb는 쿠버네티스 Ingress 형태로 L7 로드 밸런싱을 지원합니다. 이는 동일한 시스템에서 L4 및 L7 로드 밸런싱이 필요한 사용자에게도 이점이 됩니다.

추가적으로, loxilb는 다음을 지원합니다:
- [x] eBPF로 kube-proxy 대체(쿠버네티스의 전체 클러스터 메시 구현)
- [x] Ingress 지원
- [x] 쿠버네티스 Gateway API
- [ ] 쿠버네티스 네트워크 정책(진행 중)

## 텔코-클라우드와 loxilb
클라우드 네이티브 기능을 가진 텔코-클라우드를 배포하기 위해, loxilb는 SCP(서비스 통신 프록시)로 사용될 수 있습니다. SCP는 클라우드 네이티브 환경에서 실행되는 텔코 마이크로 서비스 최적화를 목표로 하는 [3GPP](https://www.etsi.org/deliver/etsi_ts/129500_129599/129500/16.04.00_60/ts_129500v160400p.pdf)에서 정의한 통신 프록시입니다. 자세한 내용은 [여기](https://dev.to/nikhilmalik/5g-service-communication-proxy-with-loxilb-4242)를 참조하세요.

![loxilb svc](photos/scp.svg)

텔코-클라우드는 N2, N4, E2(ORAN), S6x, 5GLAN, GTP 등 다양한 인터페이스와 표준을 통한 로드 밸런싱 및 통신을 필요로 합니다. 각각은 고유한 과제를 제시하며, loxilb는 이를 해결하기 위해 노력합니다. 예를 들어:
- N4는 PFCP 수준의 세션 지능이 필요합니다.
- N2는 NGAP 구문 분석 기능이 필요합니다(관련 블로그 - [블로그-1](https://www.loxilb.io/post/ngap-load-balancing-with-loxilb), [블로그-2](https://futuredon.medium.com/5g-sctp-loadbalancer-using-loxilb-b525198a9103), [블로그-3](https://medium.com/@ben0978327139/5g-sctp-loadbalancer-using-loxilb-applying-on-free5gc-b5c05bb723f0))
- S6x는 Diameter/SCTP 멀티 호밍 로드 밸런싱 지원이 필요합니다(관련 [블로그](https://www.loxilb.io/post/k8s-introducing-sctp-multihoming-functionality-with-loxilb))
- MEC 사용 사례는 UL-CL 이해가 필요할 수 있습니다(관련 [블로그](https://futuredon.medium.com/5g-uplink-classifier-using-loxilb-7593a4d66f4c))
- 미션 크리티컬 애플리케이션을 위해 Hitless Fail-over 지원이 필수적일 수 있습니다.
- E2는 OpenVPN이 번들로 제공되는 SCTP-LB가 필요할 수 있습니다.
- 클라우드 네이티브 VOIP를 가능하게 하기 위해 SIP 지원이 필요합니다.

## 왜 loxilb를 선택해야 하나요?
   
- 다양한 아키텍처에서 경쟁자보다 ```더 나은 성능```을 발휘합니다   
    * [단일 노드 성능](https://loxilb-io.github.io/loxilbdocs/perf-single/)  
    * [멀티 노드 성능](https://loxilb-io.github.io/loxilbdocs/perf-multi/) 
    * [ARM에서의 성능](https://www.loxilb.io/post/running-loxilb-on-aws-graviton2-based-ec2-instance)
    * [성능에 관한 짧은 데모](https://www.youtube.com/watch?v=MJXcM0x6IeQ)
- ebpf를 활용하여 ```유연```하고 ```프로그래머빌러티```을 동시에 제공할 수 있습니다
- 워크로드에 대한 고급 ```서비스 품질``` (로드 밸런서별, 엔드포인트별, 클라이언트별)
- ```모든``` 쿠버네티스 배포판/CNI와 호환 - k8s/k3s/k0s/kind/OpenShift + Calico/Flannel/Cilium/Weave/Multus 등
- 쿠버네티스에서 ```SCTP 워크로드```에 대한 광범위한 지원 (멀티 호밍 포함)
- ```NAT66, NAT64```를 지원하는 듀얼 스택
- 쿠버네티스 ```멀티 클러스터``` 지원 (계획 중 🚧)
- ```모든``` 클라우드에서 실행 가능 (퍼블릭 클라우드/온-프레미스) 또는 ```독립 실행형``` 환경

## loxilb의 기능
- L4/NAT Stateful 로드 밸런서
    * NAT44, NAT66, NAT64로 One-ARM, FullNAT, DSR 등 지원
    * TCP, UDP, SCTP (멀티 호밍 포함), QUIC, FTP, TFTP 등 지원
- 히트리스/마그레브/cgnat 클러스터링을 통한 고가용성 지원
- 클라우드 네이티브 환경을 위한 광범위하고 확장 가능한 엔드포인트 상태 확인
- Stateful 방화벽 및 IPSEC/Wireguard 지원
- [Conntrack](https://thermalcircle.de/doku.php?id=blog:linux:connection_tracking_1_modules_and_hooks), QoS 등의 기능에 대한 최적화된 구현
- ipvs와 완벽한 호환성 (ipvs 정책 자동 상속 가능)
- 정책 기반 L7 프록시 지원 - HTTP1.0, 1.1, 2.0 등 (계획 중 🚧)

## loxilb의 구성 요소
- GoLang 기반 제어 평면 구성 요소
- 확장 가능하고 효율적인 [eBPF](https://ebpf.io/) 기반 데이터 경로 구현
- goBGP 기반 라우팅 스택
- Go로 작성된 쿠버네티스 에이전트 [kube-loxilb](https://github.com/loxilb-io/kube-loxilb)

## 아키텍처 고려 사항   
- [쿠버네티스에서 kube-loxilb와 함께 loxilb 모드 및 배포 이해하기](kube-loxilb.md)
- [loxilb를 사용한 고가용성 이해하기](ha-deploy.md)

## 다양한 쿠버네티스 배포판 및 도구로 시작하기

#### 클러스터 외부 pod로 loxilb 사용   
- [K3s : 기본 flannel과 함께 loxilb 사용](k3s_quick_start_flannel.md)
- [K3s : calico와 함께 loxilb 사용](k3s_quick_start_calico.md)
- [K3s : cilium과 함께 loxilb 사용](quick_start_with_cilium.md)
- [K0s : 기본 kube-router 네트워킹과 함께 loxilb 사용](k0s_quick_start.md)
- [EKS : 클러스터 외부 모드로 loxilb 사용](eks-external.md)    

#### 클러스터 내부 pod로 loxilb 사용   
- [K3s : 클러스터 내부 모드로 loxilb 사용](k3s_quick_start_incluster.md)
- [K0s : 클러스터 내부 모드로 loxilb 사용](k0s_quick_start_incluster.md)
- [MicroK8s : 클러스터 내부 모드로 loxilb 사용](microk8s_quick_start_incluster.md)
- [EKS : 클러스터 내부 모드로 loxilb 사용](eks-external.md)     

#### 서비스 프록시로 loxilb 사용
- [K3s : flannel과 함께 loxilb 서비스 프록시 사용](service-proxy-flannel.md)
- [K3s : calico와 함께 loxilb 서비스 프록시 사용](service-proxy-calico.md)

#### 쿠버네티스 인그레스로 loxilb 사용
- [K3s: loxilb-ingress 실행 방법](loxilb-ingress.md)

#### 독립 실행형 모드로 loxilb 사용
- [loxilb 독립 실행형 모드로 실행](standalone.md)

## 고급 가이드   
- [How-To : loxilb로 서비스 그룹 영역 설정](service-zones.md)
- [How-To : K8s 외부 엔드포인트에 접근](ext-ep.md)
- [How-To : loxilb로 멀티 서버 K3s HA 배포](k3s-multi-master.md)
- [How-To : AWS에서 멀티 AZ HA 지원과 함께 loxilb 배포](aws-multi-az.md)
- [How-To : 멀티 클라우드 HA 지원과 함께 loxilb 배포](multi-cloud-ha.md)
- [How-To : ingress-nginx와 함께 loxilb 배포](loxilb-nginx-ingress.md)

## 지식 기반   
- [eBPF란 무엇인가](ebpf.md)
- [쿠버네티스 서비스 - 로드 밸런서란 무엇인가](lb.md)
- [간략한 아키텍처](arch.md)
- [코드 구조](code.md)
- [loxilb의 eBPF 내부](loxilbebpf.md)
- [loxilb NAT 모드란 무엇인가](nat.md)
- [loxilb 로드 밸런서 알고리즘](lb-algo.md)
- [수동으로 빌드/실행하는 단계](run.md)
- [loxilb 디버깅](debugging.md)
- [loxicmd 커맨드 라인 사용법](cmd.md)
- [loxicmd 개발자 가이드](cmd-dev.md)
- [loxilb API 개발자 가이드](api-dev.md)
- [loxilb 웹 API 참조](api.md)
- [성능 보고서](perf.md)
- [개발 로드맵](roadmap.md)
- [기여하기](contribute.md)
- [시스템 요구 사항](requirements.md)
- [자주 묻는 질문 - FAQs](faq.md)

## 블로그
- [K8s - 클러스터 네트워킹 향상](https://www.loxilb.io/post/loxilb-cluster-networking-elevating-k8s-networking-capabilities)    
- [eBPF - Go를 사용한 맵 동기화](https://www.loxilb.io/post/state-synchronization-of-ebpf-maps-using-go-a-tale-of-two-frameworks)    
- [K8s 인클러스터 서비스 LB와 함께 LoxiLB 사용](https://www.loxilb.io/post/k8s-nuances-of-in-cluster-external-service-lb-with-loxilb)   
- [K8s - LoxiLB와 함께 SCTP 멀티 호밍 소개](https://www.loxilb.io/post/k8s-introducing-sctp-multihoming-functionality-with-loxilb)   
- [Amazon Graviton2에서 로드 밸런서 성능 비교](https://www.loxilb.io/post/running-loxilb-on-aws-graviton2-based-ec2-instance)   
- [HA와 함께 하이퍼스케일 애니캐스트 로드 밸런싱](https://www.loxilb.io/post/loxilb-anycast-service-load-balancing-with-high-availability)   
- [Amazon EKS에서 loxilb 시작하기](https://www.loxilb.io/post/loxilb-load-balancer-setup-on-eks)   
- [K8s - "Hitless" 로드 밸런싱 배포](https://www.loxilb.io/post/k8s-deploying-hitless-and-ha-load-balancing)   
- [Oracle Cloud - Hitless HA 로드 밸런싱](https://www.loxilb.io/post/oracle-cloud-bring-your-own-lb-for-self-managed-clusters)
- [쿠버네티스에서 IPv6 마이그레이션을 쉽게 하기](https://www.loxilb.io/post/k8s-exposing-ipv4-services-externally-as-ipv6)   

## 커뮤니티 게시물
- [5G SCTP 로드 밸런서로 loxilb 사용](https://futuredon.medium.com/5g-sctp-loadbalancer-using-loxilb-b525198a9103)   
- [5G UL/CL loxilb 사용](https://futuredon.medium.com/5g-uplink-classifier-using-loxilb-7593a4d66f4c)   
- [K3s - 외부 서비스 로드 밸런서로 loxilb 사용](https://cloudybytes.medium.com/k3s-using-loxilb-as-external-service-lb-2ea4ce61e159)   
- [K8s - Multus 워크로드에 로드 밸런싱 도입](https://cloudybytes.medium.com/k8s-bringing-load-balancing-to-multus-workloads-with-loxilb-a0746f270abe)
- [5G SCTP 로드 밸런서로 LoxiLB 사용 (free5GC에서)](https://medium.com/@ben0978327139/5g-sctp-loadbalancer-using-loxilb-applying-on-free5gc-b5c05bb723f0)
- [쿠버네티스 서비스: 최적의 성능 달성은 어렵다](https://cloudybytes.medium.com/kubernetes-services-achieving-optimal-performance-is-elusive-5def5183c281)

## 연구 논문 (loxilb가 포함된)
- [Mitigating Spectre-PHT using Speculation Barriers in Linux BPF](https://arxiv.org/pdf/2405.00078)   

## 최신 CI/CD 상태
<div class="grid cards" markdown>

-   __기능(Ubuntu20.04)__

    ---
    ![build workflow](https://github.com/loxilb-io/loxilb/actions/workflows/docker-image.yml/badge.svg)    
    ![simple workflow](https://github.com/loxilb-io/loxilb/actions/workflows/basic-sanity.yml/badge.svg)    
    [![tcp-lb-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/tcp-sanity.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/tcp-sanity.yml)    
    [![udp-lb-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/udp-sanity.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/udp-sanity.yml)    
    [![sctp-lb-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/sctp-sanity.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/sctp-sanity.yml)    
    ![extlb workflow](https://github.com/loxilb-io/loxilb/actions/workflows/advanced-lb-sanity.yml/badge.svg)    
    ![ipsec-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/ipsec-sanity.yml/badge.svg)    
    ![scale-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/scale-sanity.yml/badge.svg)    
    [![liveness-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/liveness-sanity.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/liveness-sanity.yml)    
    ![nat66-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/nat66-sanity.yml/badge.svg)    
    [![perf-CI](https://github.com/loxilb-io/loxilb/actions/workflows/perf.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/perf.yml)      

-   __기능(Ubuntu22.04)__

    ---
    [![Docker-Multi-Arch](https://github.com/loxilb-io/loxilb/actions/workflows/docker-multiarch.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/docker-multiarch.yml)    
    [![Sanity-CI-Ubuntu-22](https://github.com/loxilb-io/loxilb/actions/workflows/basic-sanity-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/basic-sanity-ubuntu-22.yml)    
    [![tcp-lb-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/tcp-sanity-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/tcp-sanity-ubuntu-22.yml)    
    [![udp-lb-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/udp-sanity-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/udp-sanity-ubuntu-22.yml)    
    ![ipsec-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/ipsec-sanity-ubuntu-22.yml/badge.svg)    
    ![nat66-sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/nat66-sanity-ubuntu-22.yml/badge.svg)    
    [![Scale-Sanity-CI-Ubuntu-22](https://github.com/loxilb-io/loxilb/actions/workflows/scale-sanity-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/scale-sanity-ubuntu-22.yml)    
    [![perf-CI](https://github.com/loxilb-io/loxilb/actions/workflows/perf.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/perf.yml)    
    [![k3s-calico-ubuntu22-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-calico-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-calico-ubuntu-22.yml)    

-   __K3s 테스트__

    ---
    [![K3s-Base-Sanity-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-base-sanity.yml/badge.svg?branch=main)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-base-sanity.yml)     
    [![k3s-flannel-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel.yml)     
    [![k3s-flannel-ubuntu22-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-ubuntu-22.yml)     
    [![k3s-flannel-cluster-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-cluster.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-cluster.yml)     
    [![k3s-flannel-incluster-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-incluster.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-incluster.yml)     
    [![k3s-flannel-incluster-l2-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-incluster-l2.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-flannel-incluster-l2.yml)     
    [![k3s-calico-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-calico.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-calico.yml)     
    [![k3s-cilium-cluster-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-cilium-cluster.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-cilium-cluster.yml)      
    [![k3s-sctpmh-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh.yml)     
    [![k3s-sctpmh-ubuntu22-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh-ubuntu-22.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh-ubuntu22.yml)      
    [![k3s-sctpmh-2-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh-2.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k3s-sctpmh-2.yml)     

-   __K8s 클러스터 테스트__

    ---
    [![K8s-Calico-Cluster-IPVS-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs.yml)     
    [![K8s-Calico-Cluster-IPVS2-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs2.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs2.yml)     
    [![K8s-Calico-Cluster-IPVS3-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs3.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs3.yml)      
    [![K8s-Calico-Cluster-IPVS3-HA-CI](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs3-ha.yml/badge.svg)](https://github.com/loxilb-io/loxilb/actions/workflows/k8s-calico-ipvs3-ha.yml)     
     

-   __EKS 테스트__

    ---
    ![EKS](https://github.com/loxilb-io/loxilb/actions/workflows/eks.yaml/badge.svg?branch=main)   
     
    
</div>
