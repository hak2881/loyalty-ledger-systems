# Simulating a Tier Change Before Shipping It

**Domain** · Racquet-sports and alpine-outdoor brands, Korea D2C
**Method** · Run the proposed rules against real purchase history, repeatedly, before any code ships

## Context

A membership tier redesign looks like a configuration change. New thresholds, maybe a new tier, maybe a different qualifying window. One migration, one deploy.

It is not a configuration change. It is a change that moves real, named customers between tiers — including downward.

## Problem

**Demotion is the failure mode nobody models.** Raising a threshold to make the top tier more exclusive also takes the top tier away from everyone who qualified under the old number. Those are, by construction, the best customers. The first they hear of it is when their discount stops working.

**Distribution is not intuitive.** Whether a proposed threshold puts 3% or 30% of the base in a tier depends on the actual shape of the purchase distribution, which is lumpy and does not resemble whatever anyone pictured in the meeting. A tier that turns out to hold 40% of customers isn't a tier, it's a discount.

**The qualifying window changes the answer as much as the threshold does.** "Spend in the last 12 months" and "spend in the previous calendar year" produce materially different populations from the same customers and the same thresholds.

**It cannot be tested in production.** There is no staged rollout for tier assignment. The moment the rules change, every customer is reassigned.

## Approach

Run the proposed rules against the real order history and produce the resulting distribution, before writing the code that implements them.

Each simulation run took the actual purchase records, applied a candidate rule set — thresholds, qualifying window, tier count — and produced the full outcome:

- how many customers land in each tier
- how many move up, how many move down, how many stay
- who specifically is demoted, and what they had spent

That last item is the one that changes conversations. "This threshold demotes 1,200 customers" is an abstraction; a list containing customers with substantial lifetime spend is not.

Runs were repeated over several weeks as parameters were adjusted, with each version kept rather than overwritten. Keeping the history mattered: proposals get revisited, and "we already tried that and it demoted the top of the base" is only a usable answer if the output still exists.

## Key decisions

**Simulate before implementing, not after.** The natural order is to build the tier engine and then check what it produces. Reversing it means the rules are settled before any code depends on them, and the engine is written once against a specification that has already survived contact with real data.

**The output is for the client, not for engineering.** The deliverable was a spreadsheet a business owner could sort and filter, not a report or a dashboard. Tier thresholds are a commercial decision — the job here is to make the consequences legible to whoever gets to make it, in a tool they already use.

**Every run kept, timestamped.** Parameters were adjusted many times across the process. Overwriting would have discarded the record of what was tried and rejected, which is most of the value once the discussion has been running for a few weeks.

**Demotion policy is a separate decision from thresholds.** Once the distribution was visible, whether to grandfather existing members, phase demotion over a period, or apply it immediately became its own explicit question — rather than an implementation detail that gets decided by whoever writes the migration.

## What this case is really about

There is no clever engineering here. The point is that a class of change looks like configuration and behaves like a product decision, and the correct response is to make its consequences visible to the person who owns the decision — before, not after.

The failure this avoids is the one where the tier rules ship, the support queue fills with people asking why their benefits disappeared, and the fix is a hurried grandfathering rule bolted on under pressure.

---

## 한국어 요약

멤버십 등급 개편은 **설정 변경처럼 생겼습니다.** 새 기준, 등급 추가, 산정 기간 변경. 마이그레이션 하나, 배포 하나. 하지만 설정 변경이 아닙니다. **실제로 이름이 있는 고객들을 등급 사이로 옮기는 변경이고, 아래쪽으로도 옮깁니다.**

**어려웠던 지점**

- **아무도 모델링하지 않는 실패 모드는 강등입니다.** 최상위 등급을 더 희소하게 만들려고 기준을 올리면, 예전 기준으로 자격이 있던 사람들에게서 그 등급을 **빼앗습니다.** 구조상 그들이 가장 좋은 고객입니다. 그리고 그들은 **할인이 안 먹히는 순간** 이 사실을 처음 알게 됩니다.
- **분포는 직관과 다릅니다.** 어떤 기준이 고객의 3%를 담을지 30%를 담을지는 실제 구매 분포의 형태에 달렸고, 그 분포는 울퉁불퉁하며 회의실에서 상상한 모습과 닮지 않았습니다. 고객의 40%가 들어가는 등급은 등급이 아니라 그냥 할인입니다.
- **산정 기간이 기준만큼이나 답을 바꿉니다.** "최근 12개월 구매액"과 "직전 연도 구매액"은 같은 고객·같은 기준에서 상당히 다른 모집단을 만듭니다.
- **프로덕션에서 테스트할 수 없습니다.** 등급 배정에는 단계적 롤아웃이 없습니다. 규칙이 바뀌는 순간 **전 고객이 재배정**됩니다.

**접근** — 구현 코드를 쓰기 전에, 제안된 규칙을 **실제 주문 이력에 돌려서 결과 분포를 만들었습니다.**

각 시뮬레이션은 실제 구매 기록에 후보 규칙(기준·산정 기간·등급 수)을 적용해 전체 결과를 산출했습니다.

- 등급별 인원 분포
- 상승 / 하락 / 유지 인원
- **구체적으로 누가 강등되는지, 그 사람이 얼마를 썼는지**

마지막 항목이 대화를 바꿉니다. "이 기준은 1,200명을 강등시킵니다"는 추상이지만, 누적 구매액이 큰 고객들이 들어 있는 명단은 추상이 아닙니다.

파라미터를 조정하며 몇 주에 걸쳐 반복 실행했고, **매 버전을 덮어쓰지 않고 보관**했습니다. 이게 중요했습니다 — 제안은 다시 올라오고, "그건 이미 해봤는데 상위 고객이 강등됐습니다"는 **결과물이 남아 있을 때만 쓸 수 있는 답**입니다.

**핵심 결정**

- **구현 후가 아니라 구현 전에 시뮬레이션.** 자연스러운 순서는 등급 엔진을 만들고 결과를 확인하는 것입니다. 뒤집으면 어떤 코드도 의존하기 전에 규칙이 확정되고, 엔진은 **이미 실제 데이터와의 접촉을 견딘 명세**에 대해 한 번만 작성됩니다.
- **산출물은 엔지니어링이 아니라 고객사를 위한 것.** 리포트나 대시보드가 아니라 **비즈니스 담당자가 직접 정렬·필터할 수 있는 스프레드시트**로 냈습니다. 등급 기준은 상업적 결정이고, 여기서 할 일은 그 결정을 내릴 사람에게 결과를 **그 사람이 이미 쓰는 도구로** 읽히게 만드는 것입니다.
- **모든 실행을 타임스탬프와 함께 보관.** 과정 내내 파라미터를 여러 번 조정했습니다. 덮어썼다면 무엇을 시도하고 무엇을 기각했는지의 기록이 사라졌을 텐데, 논의가 몇 주 이어지면 그 기록이 가치의 대부분입니다.
- **강등 정책은 기준과 별개의 결정.** 분포가 보이고 나서야 기존 회원 유예(grandfathering) 여부, 단계적 강등, 즉시 적용이 **명시적인 질문**이 됐습니다 — 마이그레이션을 작성하는 사람이 얼떨결에 정하는 구현 세부사항이 아니라.

**이 케이스의 요점** — 여기 영리한 엔지니어링은 없습니다. 요점은 **설정처럼 생겼지만 제품 결정처럼 동작하는 변경**이 있다는 것, 그리고 올바른 대응은 그 결과를 **결정 권한을 가진 사람에게, 사후가 아니라 사전에** 보이게 만드는 것이라는 점입니다. 이렇게 해서 피하는 실패는 — 등급 규칙이 배포되고, 혜택이 왜 사라졌냐는 문의가 큐를 채우고, 압박 속에서 급조한 유예 규칙을 덧붙이는 — 그 시나리오입니다.
