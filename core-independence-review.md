# 핵심 문서(①②) 독립성 검토: 수정 목록과 예시 모듈 초안

- 기준 파일: `attached-fix/` 최신본 (첨부하신 원고에 지금까지의 수정을 반영한 것)
- 행 번호는 각 파일 기준이다.

## 0. 원칙

1. ①②는 핵심(core) 문서다. 어떤 종속 문서도 참조하지 않는다. normative 참조와 informative 참조 모두 두지 않고, 예시와 설명에도 종속 문서의 내용을 쓰지 않는다.
2. ③과 앞으로 나올 기술별 문서는 종속 문서다. ①②를 normative로 참조하고 import한다.
3. ①②의 예시는 가상의 예시 모듈 `example-pm-parameters`(RFC 8407의 예시 모듈 관례)로 쓴다. 특정 기술의 파라미터와 프로파일 이름은 쓰지 않는다.
4. G.7710은 종속 문서가 아니라 요구 사항의 출처이므로, ①에서 기반으로 밝히는 것은 그대로 둔다.

분류:

- **A (필수):** ③에 대한 참조, ③의 프로파일 이름·identity·namespace, ③에만 필요한 용어
- **B (권장):** 특정 기술에 치우친 예시나 표현
- **C (유지):** 여러 기술을 고르게 나열한 목록, G.7710을 요구 사항 출처로 밝힌 부분

---

## 1. 범용 예시 모듈 초안

①②의 모든 예시에서 아래 모듈을 쓴다. ①에는 tree와 함께 "Example Module" 그림으로 한 번 싣고, ②는 ①의 예시 모듈을 그대로 쓴다고 적는다.

```yang
module example-pm-parameters {
  yang-version 1.1;
  namespace "urn:example:pm-parameters";
  prefix ex;

  import ietf-pm-collection {
    prefix pm-co;
    reference
      "RFC XXXX: A YANG Data Model for Performance Monitoring (PM)
       Collection";
  }

  organization
    "Example organization";
  contact
    "Example contact";
  description
    "This module defines example performance parameters, derived
     from the performance-parameter base identity of
     ietf-pm-collection, for use in the examples of this
     document only.  It is not intended for implementation.
     Parameters of specific network technologies are defined in
     other documents in the same way.";

  revision 2026-10-29 {
    description
      "Initial revision.";
    reference
      "RFC XXXX: A YANG Data Model for Performance Monitoring (PM)
       Collection";
  }

  identity error-count {
    base pm-co:performance-parameter;
    description
      "Example event parameter: the number of error events
       detected in each sampling interval.  It is used with the
       counts collection type.";
  }

  identity cpu-utilization {
    base pm-co:performance-parameter;
    description
      "Example gauge parameter: the processor utilization of the
       equipment, in percent, observed at each sampling interval.
       It is used with the snapshot and tidemarks collection
       types.";
  }
}
```

검증:

- pyang `--strict`와 yanglint를 통과했다. `--ietf` 경고 3개는 예시 모듈이라 생기는 것으로 정상이다(모듈 이름, namespace, 저작권 문구).
- 아래 설정 인스턴스를 ①과 함께 yanglint로 검증해 유효함을 확인했다.
  - 프로파일 `example-generic-maintenance-15min`
  - `ex:error-count`는 counts
  - `ex:cpu-utilization`은 snapshot과 tidemarks

예시 치환 규칙(①② 공통):

| 지금 | 바꿀 것 |
|---|---|
| 프로파일 `itu-transport-maintenance-15min` | `example-generic-maintenance-15min` |
| `xmlns:pm-pa="urn:ietf:params:xml:ns:yang:ietf-pm-parameters"` | `xmlns:ex="urn:example:pm-parameters"` |
| counts 예시의 `pm-pa:es` | `ex:error-count` |
| snapshot/tidemarks 예시의 `pm-pa:es` | `ex:cpu-utilization` |
| 본문의 "errored seconds (ES)" (예시 설명) | "error count" |

두 파라미터를 나누면 효과가 하나 더 있다. 지금은 ES 하나로 snapshot까지 보여 주는데, G.7710에서 ES는 counts 전용이다. 파라미터를 나누면 세 collection type을 모두 올바른 쓰임으로 보여 줄 수 있다.

---

## 2. ① draft-yoon-ippm-collection-measure

| 분류 | 행 | 지금 | 수정안 |
|---|---|---|---|
| A | 62 | informative: `I-D.yoon-ccamp-pm-streaming:` | 삭제 |
| A | 405–407 | The network and transport performance parameters listed above (for example, ES, SES, UAS, BBE, BBC) are the primary parameters in scope, and each is expected to be specified… | Each performance parameter is expected to be specified… (transport를 "주 대상"으로 두는 표현 삭제) |
| A | 411–416 | The remaining categories (network environment, … application-aware parameters) are listed as illustrative examples of the model's extensibility and are not themselves defined by this document; | The parameters listed above are illustrative and are not themselves defined by this document; (위 수정과 맞춤) |
| A | 422–426 | …for example, the transport parameters (ES, SES, BBE, BBC, UAS, PJE, and SEP) are defined in the `ietf-pm-parameters` module of {{I-D.yoon-ccamp-pm-streaming}}, and parameters for other network domains can be defined in the same way by the relevant standards body. | …parameters of specific network technologies are defined in the same way in other documents, by the relevant standards body. |
| A | 454–458 | For example, the `itu-transport-maintenance-15min` profile includes parameters such as errored seconds (ES), severely errored seconds (SES), and unavailable seconds (UAS), which are typically monitored together for network maintenance and fault detection purposes (see {{I-D.yoon-ccamp-pm-streaming}}). Similarly, the `example-ip-qos-24hr` profile may include delay, jitter, and loss parameters… | For example, a maintenance profile may group the error counters of an interface that are monitored together for fault detection, and a QoS profile may group delay, jitter, and loss parameters… |
| A | 504–508 | 이름 예시: itu-transport-maintenance-15min, itu-transport-qos-24hr, ieee-access-availability, example-ip-qos-24hr, vendorx-otn-sla | example-generic-maintenance-15min, example-ip-qos-24hr, example-mobile-availability-24hr, vendorx-otn-sla (모두 가상의 이름, 여러 기술을 고르게) |
| A | 515–516 | The term 'transport' means that the profile applies to multiple technologies (e.g., OTN, MPLS-TP, Transport Ethernet, etc.). | 삭제 (③에서만 필요한 정의) |
| A | 1091–1095 | …retrieves the current errored-seconds (ES) count for the `itu-transport-maintenance-15min` profile (identity `pm-pa:es` of {{I-D.yoon-ccamp-pm-streaming}}), sampled every second… | …retrieves the current count of the `ex:error-count` parameter (see {{fig-example-module}}) in the `example-generic-maintenance-15min` profile, sampled every second… |
| A | 1100–1134 | 폴링 요청 XML | 치환 규칙 적용 |
| A | 1136–1137 | …returns the current ES count of 7… | …returns the current count of 7… |
| A | 1142–1171 | 폴링 응답 XML | 치환 규칙 적용 |
| A | 1190–1192 | …for the ES parameter in the `itu-transport-maintenance-15min` profile. | …for the `ex:error-count` parameter in the `example-generic-maintenance-15min` profile. |
| A | 1198–1224 | 구독 XML (namespace, XPath의 프로파일과 파라미터) | 치환 규칙 적용 |
| A | 1228–1232 | …counts collection value of the ES parameter in the `itu-transport-maintenance-15min` profile… The collected value (10) represents the total errored seconds… | …counts collection value of the `ex:error-count` parameter in the `example-generic-maintenance-15min` profile… The collected value (10) represents the total number of error events… |
| A | 1236–1276 | push-update XML | 치환 규칙 적용 |
| A | 1351–1354 | …for the snapshot collection type of the ES parameter in the `itu-transport-maintenance-15min` profile… | …for the snapshot collection type of the `ex:cpu-utilization` parameter in the `example-generic-maintenance-15min` profile… |
| A | 1357–1393 | threshold 알림 XML | 치환 규칙 적용 (`ex:cpu-utilization`) |
| A | 새 그림 | — | 예시 모듈 그림 `{:#fig-example-module title="Example Module Used in the Examples"}`를 Examples 절 앞에 추가 |
| B | 143 | Passive methods, such as … the reading of protocol counters (for example ES, SES, BBE)… | …the reading of protocol counters (for example interface error counters)… |
| B | 650–658, 710–712 | Use Cases의 "errored seconds (ES)"와 표의 "NOC (ES)" | "error count"와 "NOC (errors)". 내용과 interval 조합은 그대로 |
| B | 860 | …different parts of the transport network. | …different parts of the network. |
| C | 69, 166, 206, 303–310, 320 | G.7710을 요구 사항 출처로 밝힌 문장 | 유지 |
| C | 352–401 | 여러 기술의 파라미터 유형 목록(transport, packet, 물리, 환경, 에너지, 보안, 모바일 등) | 유지 (고르게 나열됨) |
| C | 1051–1052 | CSES 정의 | 유지 (①이 모델링하는 G.7710 이벤트) |

### ① YANG: ietf-pm-collection

| 분류 | 행 | 지금 | 수정안 |
|---|---|---|---|
| A | 89–93 | `profile-names` 예시: itu-transport-maintenance-15min, itu-transport-maintenance-24hr, itu-transport-qos-24hr, ieee-access-availability | example-generic-maintenance-15min, example-ip-qos-24hr, example-mobile-availability-24hr |
| A | 193–195 | …derived identities in other modules, for example the transport parameters in ietf-pm-parameters."; | …derived identities in other modules, for example modules that define the parameters of specific network technologies."; |
| C | 83 | `<network>` 예시 (e.g., transport, access, core) | 유지 (형식 설명용으로 고르게 나열) |
| C | 34, 136 | "companion modules" | 유지 (②를 가리킴, 핵심 문서끼리의 관계) |

---

## 3. ② draft-yoon-ippm-collection-interval-capabilities

| 분류 | 행 | 지금 | 수정안 |
|---|---|---|---|
| A | 70 | informative: `I-D.yoon-ccamp-pm-streaming:` | 삭제 |
| A | 205–208 | Specific performance parameters, such as the transport parameters, are defined as identities derived from "performance-parameter" in other documents, for example {{I-D.yoon-ccamp-pm-streaming}}. | Specific performance parameters are defined as identities derived from "performance-parameter" in other documents. |
| A | 337–339 | …parameter profiles (… such as "itu-transport-maintenance-15min"), PM parameters (individual parameters such as ES, SES, BBE, identified by identities…) | …parameter profiles (… such as "example-generic-maintenance-15min"), PM parameters (individual parameters identified by identities…) |
| A | 464–470 | …the capabilities advertised for the ES parameter in the "itu-transport-maintenance-15min" profile. … The parameter is identified by the identity "pm-pa:es" defined in {{I-D.yoon-ccamp-pm-streaming}}. | …the capabilities advertised for the `ex:error-count` parameter in the "example-generic-maintenance-15min" profile. … The parameter is identified by the identity "ex:error-count" of the example module in {{I-D.yoon-ippm-collection-measure}}. |
| A | 474–500 | 능력 조회 요청 XML | 치환 규칙 적용 |
| A | 507–545 | 능력 조회 응답 XML. `supported-collection-types`가 counts, snapshot, tidemarks | 치환 규칙 적용. `ex:error-count`에는 counts만 둔다 |
| A | 549–556 | …supports the counts, snapshot, and tidemarks collection types for the parameter; … configure the ES parameter … with any of the three collection types… | …supports the counts collection type for the parameter; … configure the `ex:error-count` parameter … with the counts collection type… |
| B | 259–262 | …As illustrated by the use cases in ①, Errored Seconds (ES) sampled every second may be aggregated… | …an error-count parameter sampled every second may be aggregated… |
| B | 288–301, 309 | 장비 등급 설명과 표의 "ES" | "error-count parameter"와 표 제목의 "(error-count parameter)" |
| C | 85, 91, 119, 133 등 | ①을 "companion"으로 부르는 표현 | 유지 (핵심 문서끼리의 관계) |

### ② YANG: ietf-pm-interval-capabilities

| 분류 | 행 | 지금 | 수정안 |
|---|---|---|---|
| A | 260–262 | …(e.g., itu-transport-maintenance-15min).";` | …(e.g., example-generic-maintenance-15min)."; |

---

## 4. ③ draft-yoon-ccamp-pm-streaming: 종속 문서로 다시 위치 지정

③은 ①②를 normative로 참조하고 import하는 구조가 이미 맞다. 고칠 것은 "companion"과 "three-document set" 같은 표현과 문서 관계 그림이다.

| 행 | 지금 | 수정안 |
|---|---|---|
| 80–82 | This document specifies how the generic Performance Monitoring (PM) collection YANG data model and its interval-capabilities companion model are applied to common transport equipment. | This document applies the core Performance Monitoring (PM) collection and interval-capabilities YANG data models to common transport equipment. |
| 107 | Two companion documents define the generic models used here. | This document depends on two core documents that define the technology-neutral models used here. |
| 145 | # Relationship to the Companion Documents | # Relationship to the Core Documents |
| 147–148 | This document is the transport-equipment member of a three-document set, … | This document is a technology-specific document that depends on the two core documents, as shown in {{fig-doc-relationship}}. Other technology-specific documents can depend on the core documents in the same way. |
| 150–178 | 문서 관계 그림 | 아래 그림으로 교체 |
| 179 | title="Relationship between this document and the companion documents" | title="Relationship between the core documents and this document" |
| 181 | The three documents play distinct, layered roles: | The documents play distinct, layered roles: |
| 200 | * (3) This document … | 유지. 단 "one feature for each transport profile"은 Open Issues 결정에 따라 정리 |

교체할 그림(최대 폭 61자):

```
   +--------------------------------------------------------+
   |  Core documents (technology-neutral)                   |
   |                                                        |
   |  (1) ietf-pm-collection  <----  (2) ietf-pm-interval-  |
   |      PM collection     imports      capabilities       |
   |      (base identity                 (augments          |
   |       performance-parameter)         RFC 9196)         |
   +--------------------------------------------------------+
               ^                               ^
               | imports                       | advertises
               | (normative)                   | capabilities
   +--------------------------------------------------------+
   |  Technology-specific documents                         |
   |                                                        |
   |  (3) ietf-pm-parameters (THIS DOCUMENT): transport     |
   |  (other documents for other technologies)              |
   +--------------------------------------------------------+
```

---

## 5. 영향 정리

- **바뀌지 않는 것:** ①②의 스키마 구조, 규범적 내용(MUST/SHOULD), 트리 다이어그램.
- **바뀌는 것:**
  - 예시와 설명
  - 참고문헌 1개씩 삭제
  - YANG description 3곳 (①에서 2곳, ②에서 1곳)
  - 예시 모듈 그림 1개 추가
- **검증할 것:** 치환 후 ①②의 XML 예시 전체를 ①②와 예시 모듈로 yanglint 검증한다. ③ 모듈 없이 통과해야 한다.
- **같이 해결되는 것:** ② 능력 조회 예시와 ③ 프로파일(counts만) 사이의 불일치.
