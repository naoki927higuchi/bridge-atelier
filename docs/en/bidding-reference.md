# Ichigaya Bidding System — Complete Reference

> **This is the official English translation of the Japanese Bidding Reference.** The [Japanese edition](../ja/bidding-reference.md) is the sole authoritative specification of the meaning of the Ichigaya Bidding System. If this translation and the Japanese edition differ, the Japanese edition prevails. The [Bidding System](bidding-system.md) is a practical guide.
> System version `Ichigaya-v1.1.4`.

[Practical guide (Bidding System)](bidding-system.md) · [Terminology](../terminology.md) · [日本語](../ja/bidding-reference.md)

## Contents

1. [Status, scope, notation, guarantees and management principles](#1-status-scope-notation-guarantees-and-management-principles)
2. [Obligations, trump agreement and obligation state transitions](#2-obligations-trump-agreement-and-obligation-state-transitions)
3. [Openings and responses to strong natural two-bids](#3-openings-and-responses-to-strong-natural-two-bids)
4. [First responses, raises, rebids and natural NF stops](#4-first-responses-raises-rebids-and-natural-nf-stops)
5. [2/1, two-step obligations and reverses](#5-21-two-step-obligations-and-reverses)
6. [Artificial FSF A–D and fixed continuations](#6-artificial-fsf-ad-and-fixed-continuations)
7. [Notrump system and Stayman](#7-notrump-system-and-stayman)
8. [Jacoby 2NT](#8-jacoby-2nt)
9. [Gerber, Blackwood and quantitative 4NT](#9-gerber-blackwood-and-quantitative-4nt)
10. [Ichigaya Refrain](#10-ichigaya-refrain)
11. [Competition, Double and intervention](#11-competition-double-and-intervention)
12. [Undefined, not adopted and off-system calls](#12-undefined-not-adopted-and-off-system-calls)
13. [Appendix A: CPU implementation status and constraints](#13-appendix-a-cpu-implementation-status-and-constraints)
14. [Appendix B: Index of system IDs and implementation IDs](#14-appendix-b-index-of-system-ids-and-implementation-ids)

---

## 1. Status, scope, notation, guarantees and management principles

### 1.1 Status of this document

- This document is the complete reference that records every condition of the Ichigaya Bidding System. The practical guide for quick lookup of the main rules is the [Bidding System](bidding-system.md). Where both documents state an important condition, the meanings must agree.
- The Japanese edition of this document is the sole authoritative specification of the meaning of the Ichigaya Bidding System. The meaning, guarantees and obligations of every call are fixed by the Japanese edition.
- This English edition is the official translation that follows the Japanese edition. If they differ, the Japanese edition prevails.
- The practical guide ([Bidding System](bidding-system.md)) is for quick reference to the key points of this document and adds no meaning to the system. If it differs from this document, this document prevails.
- The implementation (code) follows this document. If the implementation behaves differently from this document, that is treated as a defect in the implementation (known differences: [13.7](#137-known-differences-between-the-system-and-the-implementation)).
- This document defines the **meaning** of calls (guarantees, obligations, status, and the boundaries of what is undefined). Adjustable thresholds, coefficients and candidate scores that decide which call the CPU chooses are, in principle, not included here; they are managed in the implementation (code). Only the CPU guidelines needed to understand the meaning are recorded, marked **[CPU criterion]**.
- Following this system does not guarantee the best result or a good contract in every situation.
- The system version, the bidding AI version and the game version are managed separately ([1.8](#18-revision-procedure)).

### 1.2 Scope and how sequences are written

- The main scope is the ordinary auction in which **all of the opponents' calls are Pass and responder is not a passed hand**.
- Sequences show only our side's (N/S) calls; the opponents' passes in between are omitted. The roles are the same when the seats rotate.
- The system applies in its **complete context**. The complete context includes the history of initial passes, any intervention, the position of the opening, and natural/artificial meanings already known. **A rule does not apply merely because the last calls of a sequence match.**
- Numeric bids rank ♣→♦→♥→♠→NT. An auction ends only with the consecutive passes required by the Laws.
- **Legality and conformity with the system are separate.** A human may make any call that is legal under the Laws, even if it does not conform to the system ([12.1](#121-off-system-calls-by-a-human)).

### 1.3 Three layers of information

| Layer | Meaning |
|---|---|
| **[Guaranteed] GUARANTEE (public guarantee)** | Suit lengths, stated HCP, specific cards and obligations fixed by the system. They are carried forward into later decisions and used to detect contradictions |
| **[Typical] typical range** | Typical HCP, typical distribution, practice. Never treated as a strict minimum, maximum or denial |
| **[CPU criterion] CPU_ESTIMATE / CPU evaluation** | Estimated strength, contract quality and candidate scores that the CPU derives from its own hand and the public auction. The center of a typical range may be updated by later limited responses and the like, but the possibility of a strong hand is never reduced to zero merely because some call was not chosen |

- The CPU uses only its own 13 cards and the public auction as inputs. It never uses partner's or the opponents' actual cards, the 52 cards shown for post-board verification, or the opponents' decision reasons as hidden sources of information.
- Expecting partner's length to supply a stopper is an estimate, not a guarantee that partner holds an ace or king.

### 1.4 Meaning status

| Status | Meaning |
|---|---|
| DEFINED | Defined |
| PARTIAL | Only partly defined |
| UNDEFINED | No applicable definition |
| CONFLICT | Contradicts earlier guarantees or constraints |
| UNSUPPORTED | The meaning is defined, but the CPU does not support it |

- A status is never converted into Pass across the board.
- An undefined call is not assumed to be natural, and an artificial bid is not replaced by a natural candidate.

### 1.5 Order of interpretation

The meaning of a call is checked in this order:

1. An ask or report in progress that has its own dedicated meaning
2. An artificial initiation, or a specified natural continuation, that matches the complete context
3. The general natural rules
4. Undefined

Examples: 4NT as an ace-location report after Jacoby 2NT is not treated as Blackwood. The natural 4♣ in the D sequence ([chapter 6](#6-artificial-fsf-ad-and-fixed-continuations)) is not treated as Gerber.

### 1.6 Symbols and labels

| Symbol or label | Meaning |
|---|---|
| `M` | Major (♥/♠) |
| `m` | Minor (♣/♦) |
| `x` | Any suit |
| `5+` | Five or more cards |
| `HCP` | High card points |
| `BAL` | Balanced (in this document the 1NT shapes 4333, 4432 and 5332, with the five-card suit a minor only) |
| `NF`, `INV`, `F1`, `GF`, `TO PLAY` | Forcing status ([2.1](#21-forcing-status)) |
| `fit` | Combined length with partner (for example, an 8-card fit) |
| [Guaranteed] [Typical] [CPU criterion] | The three layers of information ([1.3](#13-three-layers-of-information)) |
| [Undefined] | The system has no definition |
| [Not adopted] | Explicitly not adopted as part of the system |
| [CPU unsupported] | Defined in the system, but not implemented by the CPU |

A "system ID" (D01, J02, and so on) is an identifier of this system. Rules without an ID are referred to by the section numbers of this document ([chapter 14](#14-appendix-b-index-of-system-ids-and-implementation-ids)).

### 1.7 Management principles

**Decided by humans**: the meaning of calls, public guarantees, F1/GF, trump agreement, the conditions for answering and ending, and the scope of application.

**May be decided by the implementation**: scores of candidates that fit the same meaning, thresholds, log format, data types, and how to reproduce results with the same seed.

- If CPU discretion would change a public guarantee, the matter goes back to human judgment.
- Anything not stated in this document is treated as [Undefined]. Even when an omission is suspected, it is not filled in by guesswork; adding a meaning is a human decision.
- Being undefined is never a reason to invent a new meaning ([chapter 12](#12-undefined-not-adopted-and-off-system-calls)).

### 1.8 Revision procedure

- **Fixing a defect in the implementation** and **changing the system itself** are kept distinct.
  - Fixing a defect: correcting behavior that did not follow the text of the system. The system version stays the same; the bidding AI version is raised.
  - Changing the system: changing the meaning of a call, a guarantee, or the conditions under which an obligation arises, is cancelled or is fulfilled. The Japanese edition of this document is revised, and both the system version and the bidding AI version are raised. The English edition and the practical guide are synchronized afterwards (meaning is never changed starting from the English edition or the practical guide).
- Guarantees of the system, the CPU's typical ranges (not guaranteed) and obligations are never mixed. The CPU's internal selection criteria are never written as public guarantees.
- Versions: the system version is raised when the system changes. The bidding AI version is raised for defect fixes and for system changes. The game version is managed on the game side. The current system version is stated at the top of this document; the version of the implementation covered is stated in [chapter 13](#13-appendix-a-cpu-implementation-status-and-constraints).
- The format of decision reasons and diagnostics, and the handling of regression tests, are covered in [13.6](#136-implementation-management).

---

## 2. Obligations, trump agreement and obligation state transitions

### 2.1 Forcing status

| Status | Meaning and how it ends |
|---|---|
| `NF` | Creates no new obligation to respond. If no other obligation remains, partner may pass |
| `INV` | Invitation to game or slam. Creates no new F1/GF. Partner may evaluate and decline |
| `F1` | Requires one valid response from the specified partner. Once fulfilled, the obligation from that source ends |
| `GF` | Continues until the partnership reaches game. Not an obligation to bid slam |
| `TO PLAY` | A natural choice of contract. Creates no new F1/GF. Any separate unfulfilled obligation is judged separately |
| Trump agreement | A state in which the trump suit to be used in the subsequent contract search has been publicly agreed. A different state from GF |
| Obligation to answer | Requires the specified answer to Gerber, Blackwood, the Refrain and similar asks. It remains while unanswered, even when the auction is already at game level |

### 2.2 Tracking obligations individually

- Each obligation is tracked by **the call that created it, the seat it applies to, and the condition that fulfills it**.
- Even when one response satisfies several obligations, each obligation is judged separately.
- Completing one F1 is never a reason to release another F1, an R02, a GF or an obligation to answer together with it.
- Cancellation is not fulfillment. When an obligation is cancelled, its source, the position of the call that cancelled it, and the reason are recorded ([chapter 11](#11-competition-double-and-intervention)).

### 2.3 Fulfilling GF and trump agreement

- GF is fulfilled by **choosing a natural game contract**, or by **finally adopting an artificial contract under chapter 9** ([9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles)).
- GF is not released merely because an artificial 3NT/4NT, or an ace answer such as 5♦, is at or above game level.
- Trumps are never inferred from GF alone. Information about natural support is not confused with the state of explicit trump agreement.

Sequences in which the system states that trump agreement arises (recorded as [Guaranteed] in the relevant chapters): J07 ([4.6](#46-jump-rebids-j05j07)), Jacoby 2NT ([chapter 8](#8-jacoby-2nt)), and in the D sequence 3♠ (after the 2♠ answer) and 4♣ ([6.5](#65-d-sequence-south-after-the-fsf-answer)). In the D sequence, 3♠ after the 2NT answer is GF but **does not agree trumps**. The limited natural raise after intervention ([11.5](#115-limited-natural-raise-after-intervention)) does not automatically create a new trump agreement.

### 2.4 Obligation state transitions

This table summarizes the conditions under which obligations arise, are fulfilled, are cancelled or are maintained, as defined by this system. Any combination not in the table is [Undefined].

| Obligation | Arises | Fulfilled by | Cancelled when | Maintained over intervention (Double) | Remaining effects and notes |
|---|---|---|---|---|---|
| Natural F1 (R1, R01, REVERSE, a first natural new suit, SECOND, and other "may not pass once" obligations) | That call | A valid response by the specified partner (a defined natural call, or a new F1 or GF). After intervention, fulfilled by **any numeric bid** by the obligated seat ([11.4](#114-intervention-over-natural-f1)) | Only the F1 that R1/R01 created for opener, when an opponent makes a numeric bid immediately after the natural first response ([11.4](#114-intervention-over-natural-f1)) | Maintained over an opponent's Double (a Pass is recorded as an obligation breach) | Redouble does not fulfill it. No guarantee is added to the call that fulfills it |
| R02 (responder's obligation to rebid) | After R01, while below game | A limited preference, a rebid of one's own suit, a natural NT, and so on | When an opponent makes a numeric bid after R01, the reservation of an R02 that has not yet arisen is cancelled. It is not revived by any later rebid ([11.4](#114-intervention-over-natural-f1)) | Maintained over an opponent's Double (a Pass is recorded as an obligation breach; [11.4](#114-intervention-over-natural-f1)) | A local NF label does not remove R02 ([5.2](#52-r02-and-ordinary-rebids)) |
| F1 of FSF | Artificial FSF (A–D) | North's answer (each answer fulfills the FSF F1; no new GF is created) | Not filled in for undefined continuations or intervention | (No specific provision in the system) | FSF is not GF |
| GF | J04, J06, J07, Jacoby 2NT, specified calls in the D sequence, and so on | Choosing a natural game contract, or finally adopting an artificial contract ([9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles)) | Not cancelled by intervention ([11.3](#113-intervention-over-artificial-asks), [11.4](#114-intervention-over-natural-f1)) | — | Not released merely by the level of an artificial 3NT/4NT and the like. The CPU never passes below game while GF is in force ([2.6](#26-ending-the-auction-under-gf)) |
| Obligation to answer (ST01, ST03, AF01, the 4NT ask over the special 4M, Gerber, Plain Blackwood) | The asking call | The specified answer | When an opponent makes a numeric bid before the answer ([11.3](#113-intervention-over-artificial-asks)) | Over an opponent's Double, the obligations of the ask and the answer are maintained ([11.3](#113-intervention-over-artificial-asks)) | Remains while unanswered, even at game level |
| Asker's declaration of the final contract (Ichigaya Refrain, the 4NT ask over the special 4M) | Arises for the asker when an answer (A, K or Q) is made | The next ask, or a numeric bid that is not an ask (the final contract) | When an opponent makes a numeric bid after the answer and before the asker's next turn (the asking sequence ends; [11.3](#113-intervention-over-artificial-asks)) | Maintained over an opponent's Double ([11.3](#113-intervention-over-artificial-asks)) | Passing an artificial answer is an off-system stop ([9.5](#95-4nt-ask-over-the-special-4m), [10.3](#103-asks-and-answers)) |
| INV, NF, TO PLAY | — | — | — | — | Create no new obligation. Existing obligations are judged separately |

### 2.5 When several obligations exist at once

- Example: when artificial FSF is used after the F1 of a reverse, then until the FSF answer is made, the fulfillment of the reverse's F1, the FSF F1, an independent R02, GF and any obligation to answer are each tracked separately.
- Cancellation by intervention immediately after a natural first response ([11.4](#114-intervention-over-natural-f1)) cancels only the F1 and the R02 reservation created by that R1/R01; **any F1, GF or R02 that has already arisen separately is maintained**. Obligations to answer follow [11.3](#113-intervention-over-artificial-asks).

### 2.6 Ending the auction under GF

- While GF is in force, the CPU never ends the auction by passing in a contract below game.
- [CPU criterion] When there is no ordinary candidate, the CPU fulfills the obligation by choosing, among natural and defined game contracts, the one with the greatest combined length of its own cards and partner's cards (public guarantees and typical ranges). This is an internal CPU selection criterion and adds no public meaning or guarantee. The meaning of the chosen call is as defined in this document (a raise, one's own suit, NT game, and so on).
- It does not apply in sequences with intervention by an opponent's numeric bid, or after an undefined or unsupported history.
- A human's legal Pass is not restricted (it is only recorded as an obligation breach).

---

## 3. Openings and responses to strong natural two-bids

Everything in this chapter is in the context of **the first numeric bid**. The Refrain-only evaluation ([10.2](#102-cpu-only-evaluation)) is made first; if the Refrain is not chosen, the ordinary path follows. A special hand is not sent to 4♣ on the basis of HCP alone.

### 3.1 Opening bids

| ID and call | Natural/artificial, [Guaranteed] | [Typical], [CPU criterion], continuations |
|---|---|---|
| D01: `1NT` | Natural, **16–18 HCP (strict)**, 4333 / 4432 / 5332 with a five-card minor, no five-card major | 4441 is not allowed. Continues with Stayman, natural responses, and the asks of chapters 8 and 9 |
| D02: `1♥ / 1♠` | Natural, **5+** in the major bid | [Typical] 13–21. A five-card major takes precedence over 1NT even with 16–18. Bid the longer major; with equal majors bid ♠ (including 5–5) |
| D02: `1♣ / 1♦` | Natural, **3+** in the minor bid | With no five-card major, when not opening NT or the like. Bid the longer minor; with 3–3 bid ♣, with equal minors of 4–4 or longer bid ♦. 1♦ is not 4+ |
| `2NT` | Natural, **22–24** | 19–21 is not shown with 2NT ([3.2](#32-opening-principles)). In practice the shape is assumed to have no suit of five or more, longest suit four or fewer (4333 / 4432 / 4441). Refining the public guarantee on shape is [Undefined] (U04). The shape conditions of 1NT are not carried over |
| Strong natural two-bid: `2♣ / 2♦ / 2♥ / 2♠` | Natural, **F1** for the first response. **Not an artificial 2♣** | [Typical] 22+. Builds the contract together with partner. The strict minimum length, and how to handle 25+ with a longest suit of four, are [Undefined] (U04). Not all strong hands are funneled into an artificial 2♣ |
| PREEMPT3: `3♣ / 3♦ / 3♥ / 3♠` | Weak three-level opening. See [3.4](#34-weak-three-level-openings) | — |
| D04: special 4M (`4♥ / 4♠`) | Natural. A long, strong major that can expect game without partner's help (stand-alone game strength). **Not a weak preempt** | No public guarantee of length, HCP or playing tricks. Partner's immediate 4NT asks for the total number of aces ([9.5](#95-4nt-ask-over-the-special-4m)). See [3.6](#36-special-4m-d04) |
| AF01: `4♣` | Artificial, Ichigaya Refrain, asks for the total number of aces | [Chapter 10](#10-ichigaya-refrain). Shows nothing about club length |

### 3.2 Opening principles

- 19–21 is, in principle, shown by starting with a natural one-of-a-suit opening.
- [Typical] 13–21 (one of a suit) and [Typical] 22+ (strong two-bids) are never promoted on one's own initiative into strict HCP guarantees.
- The 2NT range is 22–24. Refining the public guarantee on its shape belongs to U04.

### 3.3 Normal openings versus preempts

- A weak three-level opening in a long, good suit is compared with the information exchange of an ordinary one-of-a-suit opening. If side aces/kings, defense and overall strength are close to an ordinary opening, one of a suit is also a candidate.
- 12 points are not uniformly opened with a preempt. The [Typical] 13 points is not a strict minimum, and an ordinary opening with 12 points is not prohibited. No new strict HCP maximum is created.

### 3.4 Weak three-level openings

- [Guaranteed] `3♣`, `3♦`, `3♥` and `3♠` as the first numeric bid are natural and NF. They publicly guarantee **at least six cards** in the suit bid. No HCP maximum and no placement of honors is guaranteed.
- [Typical] The standard is seven or more cards; the CPU's typical estimate centers on seven.
- Five or fewer cards do not conform to the system, but a human's legal call is accepted. These meanings are not carried over to responses, rebids or overcalls.
- [CPU criterion] The CPU opens with 12 HCP or less. In addition to the quality conditions for seven cards (CPU internal), with exactly six cards it requires AKQ, or three or more of AKQJ plus the 10 or 9. This is a CPU criterion, not a public guarantee about honors. Solidity, missing aces/kings, side honors, another good suit and the risk of six cards are evaluated, and the opening is compared with an ordinary opening, Pass and the special openings.
- Responder keeps the public minimum of six apart from the [Typical] center of seven. With 2/3/4-card support, the minimum fit is 8/9/10 and the typically estimated fit is 9/10/11. Responder compares Pass with a natural raise to game, using its own strength, trump quality and so on. NF permits stopping; it neither forces a stop nor forces game. No new F1/GF or artificial meaning is added to responses whose details are undefined.

### 3.5 Responses to strong natural two-bids (F02)

- A weak raise to three of the same suit (for example, `2♥–3♥`) is a special weak response **with no guarantee of support length**. It fulfills the first-round F1 and creates no new GF. Opener chooses between Pass and a natural continuation, based on its own playing tricks and the public information. Being 22+ does not force game.
- A positive new suit is a new F1.
- [Typical] Guidelines: over a strong 2M, with typically 4+ and support of 3+, consider 4M; with typically 10+, a natural new suit; if weak, 3M. Over a strong 2m, compare a good five-card major, a suitable 3NT and a weak 3m. These CPU guidelines are not turned into strict guarantees of HCP or support for the response.
- Undefined details belong to U04.

### 3.6 Special 4M (D04)

- `4♥`/`4♠` as the first numeric bid. A natural opening.
- Shows a long, strong major that can expect game without partner's help (stand-alone game strength). **Not a weak preempt.**
- There is no [Guaranteed] item: neither a strict suit length, nor HCP, nor the number of playing tricks is publicly guaranteed.
- The Refrain evaluation ([chapter 10](#10-ichigaya-refrain)) is made first, and the principle of comparison is the same.
- A 4M opening creates no trump agreement. Nothing is guaranteed about the seat or about earlier passes.
- Responses: partner's immediate 4NT is an artificial ask for the total number of aces ([9.5](#95-4nt-ask-over-the-special-4m)). All other responses and continuations are [Undefined] (U04).
- [CPU criterion] For the conditions under which the CPU chooses this opening, and how the CPU responder chooses its response, see [13.4](#134-list-of-cpu-criterion-items-in-this-document).

---

## 4. First responses, raises, rebids and natural NF stops

### 4A First responses

The following are, in principle, first responses in an uncontested auction by a responder who is not a passed hand.

- A first natural new suit creates an F1 for opener.
- None of this prohibits a human's legal Pass.

### 4.1 First responses

| ID and complete-context pattern | [Guaranteed], natural/artificial, obligation | [Typical], responses / boundaries of what is undefined |
|---|---|---|
| J01: `1M–2M` | Natural, **3+** in M, NF | Typically 6–10. With a minimum, Pass; with extra strength, invite or bid game. Details of invitational new suits are [Undefined] (U07) |
| M01: `1m–2m` | Natural, **3+** in m, NF | Typically 6–10. An 8-card fit is not guaranteed |
| J02: `1M–3M` | Natural, **3+** in M, INV | Typically 10–12. Opener leans toward declining with 13–14, accepts with 16+, and judges 15 by shape (not a fixed boundary). Pass / 4M; with extra values, the established exploration |
| J03: `1M–4M` | Natural, **5+** in M with a side singleton/void, TO PLAY | Typically a weak distributional hand of 0–9. Normally Pass; continue only with special values |
| J04: `1x–jump new suit` | Natural, **5+** in the suit bid, **GF** | Typically 19+. Continues with a raise, one's own suit or NT. Not carried over to jumps in competition |
| R1: `1m–1M` and the like, the cheapest new suit at the one level | Natural, **4+** in the major, F1 for opener | Typically 6+. The strict minimum length of 1♣–1♦ is [Undefined] (U01). The CPU's current condition of 5+ in a minor is not made a public guarantee |
| R01: non-jump 2/1 | See [5.1](#51-non-jump-21-r01) | — |
| H1NT: `1♥–1NT` / S1NT: `1♠–1NT` | Natural, NF. **Does not guarantee** a balanced hand, stoppers in all suits or any length of support for M | Typically 6–10. Over 1♥, with 4+ ♠ bid 1♠ in principle; with suitable ♥ support, raise. The detailed route with 11–12 is [Undefined] (U01) |
| F01: natural 1M–direct 3NT | Natural, a balanced hand with enough strength for game, TO PLAY. No guarantee of support for M | Typically 13+, no strict upper or lower limit. Also usable with strong three-card support. Opener passes, or corrects to 4M or the like when there is reason to. Not automatically extended to 1m |
| D09: natural 1M–direct 2NT | Artificial, **4+ (strict)** in M, GF, agrees M | [Chapter 8](#8-jacoby-2nt). Not a natural NT proposal |

- The overlap of typically 10 points between J01 and J02 is a matter of discretion, not a double definition of meaning.
- The simple raises J01/M01 also apply to a passed hand. Other rules are not automatically extended to passed hands.

### 4.2 First-response choice with weak hands

The following combinations facing 1♣ are kept as explicit CPU priority conditions.

| Weak-hand combination | First response in principle |
|---|---|
| Five ♠ and four ♦ | 1♠ |
| Five or more ♥ and four ♠ | 1♥ (the same with six or more ♥) |
| Five ♦ and four ♥ | 1♦ |

- [Typical] A weak response of typically about 6–9 is assumed. No strict boundary of weakness and no full priority order for two-suited shapes not in the table is added.
- [CPU criterion] These are the CPU's selection policy and **do not guarantee the absence of the suit not chosen**. With 6+, some first response is in principle compared, but the [Undefined] 1m–1NT is not added as a rescue. This is kept distinct from the fact that a human may choose another legal call.

### 4.3 Game, invitation and support

- Being able to support and needing to bid game at once are separate questions. The contract quality of a low raise, an invitation, game and Pass are compared. If partner is a passed hand, partner's typical strength based on the public history is updated conservatively.
- The four-card guarantee or provisional estimate of a major shown at the one level is not upgraded to five. Three cards of one's own and four of partner's do not establish an 8-card fit.
- In addition to HCP, length, trump quality, shortness and controls such as aces are evaluated. However, the value lost by an isolated honor in a short suit is not added back twice, and the same length or winners are not counted more than once through several bonuses. 13 points, six cards, a singleton or three aces are also only material for comparison, not an obligation to bid game.
- No new public length or INV/GF is attached to [Undefined] raises in competition or by a passed hand. A provisional natural comparison is recorded as CPU_ESTIMATE. The limited four-card raise in [11.5](#115-limited-natural-raise-after-intervention) is not changed to three cards.

### 4.4 Responses to minor openings

- Facing 1♣ with five ♦ and club support, the low-cost information value of 1♦ is compared with the value of a 2♣ raise. 2♣ is not prohibited.
- The unsettled minimum length of 1♦ is not made a guarantee through the CPU's five-card condition. Undefined natural responses by a passed hand remain undefined.
- The normal 1♣ with 4-4-2-3 and the three-card support of M01 are maintained.

### 4B Rebids and natural NF stops

### 4.5 Ordinary rebids

| Context | [Guaranteed], obligation | [Typical] / continuations |
|---|---|---|
| `1♠–1NT–2♥` | Second suit ♥ 4+, carries over the opening ♠ 5+, NF | Normally a second suit. Exchange of shape information |
| `1♣–1♥–3NT` | Natural TO PLAY, no guarantee of ♥ support | Typically a choice of game with 19–21 |

Rebids after a non-jump 2/1 (`1♠–2♦–2♥`, `1♥–2♦–2♥`, `1♥–2♦–2NT`) are in [5.2](#52-r02-and-ordinary-rebids); reverses are in [5.3](#53-reverse-reverse).

### 4.6 Jump rebids (J05–J07)

| ID and context | [Guaranteed], obligation | [Typical] |
|---|---|---|
| J05: `1♠–1NT–3♠`, `1♣–1♠–3♣` and the like | Carries over the guarantee of the opening, INV | Typically 16–18 with a long, strong original suit. Normally six cards; exceptions such as a strong five. Not GF |
| J06: `1♦–1♥–3♣`, `1♣–1♥–2♠` and the like | Natural, second suit **4+**, GF | Typically 19–21. The first suit is not automatically made 5+ |
| J07: `1♣–1♥–3♥` and the like | Natural, **4+** support for responder's suit, GF, agrees that suit as trumps | Typically 16–18. Three-card support is not included |

- J05–J07 apply in an uncontested auction where the first response was at the one level. They are not extended without conditions to special jumps after a 2/1 ([Undefined]).
- A natural fourth suit other than FSF A–D ([chapter 6](#6-artificial-fsf-ad-and-fixed-continuations)) is treated as the F1 of a natural new suit; what is undefined is not made artificial.

### 4.7 Fixed natural NF stopping sequences

| Complete sequence | [Guaranteed] and obligation of the last call |
|---|---|
| `1♣–1♠–1NT–2♠` | Natural, ♠ 5+, NF |
| `1♣–1♠–2♣–2♠` | Natural, ♠ 5+, NF |
| `1♣–1♦–1♥–2♦` | Natural, ♦ 5+, NF |
| `1♣–1♦–1♠–2♦` | Natural, ♦ 5+, NF |
| `1♦–1♥–1♠–2♦` | Natural, a reluctant preference with ♦ 2+, NF. Does not guarantee a combined seven cards |
| `1♣–1♠–2♦–2♠` | Natural, ♠ 5+, NF. Fulfills the F1 of the reverse |

- These rebids of one's own suit and preferences for a suit already bid are not mistaken for a fourth suit. Any other obligation remains.
- No strict minimum length for general raises not in the table is added by analogy.

### 4.8 NT bids by context

| Sequence | Meaning |
|---|---|
| `1m–2m–2NT` | Natural INV (no new F1/GF). [Typical] suggests about 16–18; 19–21 also possible. Partner judges whether to continue |
| `1m–2m–3NT` | Natural TO PLAY. [Typical] centers on about 19–21, but a good shape with 17–18 is also possible. No strict HCP guarantee |
| `1m–1M–2NT` | A jump showing a strong balanced hand centered on about 19–21 (a defined route). **Whether its HCP is guaranteed, and its forcing status, are [Undefined] (U03)**. It is not defined as a 15–18 range or as INV |

- With `1m–2m–2NT` and `1m–2m–3NT`, the fit, stoppers, sources of tricks and combined estimates are compared; an 8-card fit is not manufactured from a simple raise.

**Scope of 1m–2m–2NT**: limited to natural 1m – natural simple 2m – 2NT by the same pair with the opponents passing throughout (including the simple raise by a passed hand to which M01 applies). Natural INV, no new F1/GF. Not extended to other 2NT bids.

- [Typical] Centers on about 16–18, 19–21 also possible. No strict guarantee of HCP, balanced shape or stoppers in all suits. A 3NT response is natural TO PLAY.
- Facing partner's 2NT, responder compares Pass and 3NT using partner's public typical strength and its own hand. Even if a human bids 2NT with 14 points, the CPU does not use those hidden 14 points as input. NF permits a Pass; it does not add value to passing in the evaluation.
- The direct 2NT in the D sequence is a weak NF, the 2NT answer to FSF is an answer with strict stopper requirements, and Jacoby 2NT is an artificial GF; the INV of this section is not carried over to them.

---

## 5. 2/1, two-step obligations and reverses

### 5.1 Non-jump 2/1 (R01)

| ID and sequence | [Guaranteed], obligation | [Typical] |
|---|---|---|
| R01: `1♦–2♣`, `1♥–2♣/2♦`, `1♠–2♣/2♦/2♥` | Natural, **4+** in the minor (♥ **5+** for `1♠–2♥`). **Two-step obligation** | Typically 11+. About 10 is at one's discretion with a good long suit and the like. No strict minimum |

- `1♣–2♦` is J04 (jump new suit, GF), not R01.

### 5.2 R02 and ordinary rebids

- After R01, opener rebids once. If the auction is still below game, responder also has an obligation to rebid once: **R02**.
- R02 can be fulfilled by a limited preference, a rebid of one's own suit or a natural NT. If a new F1/GF arises, it is tracked separately.
- **2/1 as a whole is not GF.**

| Context | [Guaranteed], obligation | [Typical] / continuations |
|---|---|---|
| `1♠–2♦–2♥` | Same lengths (second suit ♥ 4+, opening ♠ 5+); R02 remains | A local label of NF does not remove R02 |
| `1♥–2♦–2♥` (ordinary rebid of one's own suit) | Carries over the opening ♥ 5+. A sixth card is not guaranteed | Typically 6+, with exceptions such as a strong five. R02 remains |
| `1♥–2♦–2NT` | Natural; R02 remains | Typically 15–18, suitable for NT. Normally not chosen with 13, but that is not a strict minimum |

### 5.3 Reverse (REVERSE)

| ID and sequence | [Guaranteed], obligation | [Typical] |
|---|---|---|
| REVERSE: `1♦–1♠–2♥`, `1♣–1♠–2♦` and the like | Natural, second suit **4+**, **first suit longer than the second**, **F1** | Typically 17–21. 17–18 can also qualify depending on shape. Can be fulfilled by a limited natural response |

For the fixed sequences after a reverse, see the D sequence ([6.4](#64-d-sequence-south-after-the-reverse)).

---

## 6. Artificial FSF A–D and fixed continuations

### 6.1 Preconditions and initiation

Preconditions: the opponents pass throughout, the first responder is not a passed hand, and the opening and second suit are natural and conform to the system. **Only the following four complete sequences** are made artificial.

| Sequence | Complete initiating sequence | Public meaning |
|---|---|---|
| A | `1♦–1♥–1♠–2♣` | Artificial 2♣, no guarantee of ♣ length, F1 for North |
| B | `1♣–1♥–1♠–2♦` | Artificial 2♦, no guarantee of ♦ length, F1 for North |
| C | `1♣–1♦–1♥–1♠` | Artificial 1♠, no guarantee of ♠ length, F1 for North |
| D | `1♣–1♠–2♦–2♥` | Artificial 2♥, no guarantee of ♥ length, F1 for North. Fulfills the F1 of South's reverse |

- [Typical] Typically 10+ HCP is a guideline, not a strict minimum. Exceptions with 9 or less are allowed for shape or information value, but with about 6–9 a natural stop is preferred in principle. No additional condition is created for C alone at the one level.
- **FSF is not GF.** In A–C there is normally no F1 for South remaining just before FSF. An obligation that does not exist is not created.

### 6.2 North's answers

The context of each row is immediately after the complete sequence above. The answers are natural contract candidates; being answers to an artificial FSF does not make them all artificial.

| Sequence | Support (highest priority) | Original suit | NT |
|---|---|---|---|
| A | `2♥`: ♥ 3+ | `2♦`: ♦ 4+ | `2NT`: strictly guarantees ♣A or ♣KQ |
| B | `2♥`: ♥ 3+ | `3♣`: ♣ 4+ | `2NT`: strictly guarantees ♦A or ♦KQ |
| C | `2♦`: ♦ 3+ | `2♣`: ♣ 3+ | `2NT`: strictly guarantees ♠A or ♠KQ |
| D | `2♠`: ♠ 3–4 | `3♣`: ♣ 5+ | `2NT`: strictly guarantees ♥A or ♥KQ |

- If support is possible, support has the highest priority. If not, the original suit and NT are compared.
- NT requires the strict card condition in the table above and suitability for NT. The [Typical] 15+ is not a strict minimum, and the known strength of the reverse in D is not counted twice. Not limited to balanced hands. Since the ace may be a singleton, 2+ in the fourth suit is not guaranteed. A stopper does not force NT, and the absence of a stopper is not inferred from an answer in the original suit.
- Each answer fulfills the F1 of FSF and by itself creates no new GF. If no other obligation exists, South may pass. The known guarantees of the first and second suits are carried over. C's 2♣ (3+) and D's 3♣ (5+) are not unified. The 4+ in the original minor is not guaranteed for every hand that starts A/B; it is guaranteed only when that answer is made.
- In contexts that strictly keep support as the highest priority, D's 2NT/3♣ can be inferred to hold two or fewer ♠. A human's deviation is not repaired by looking at the actual hand. Rebidding the second major is not included in the standard answers (and 5+ is not guaranteed either). No additional artificial meaning is assigned to A's 3♣.

### 6.3 Fixed continuations and boundaries for A–C

`1♦–1♥–1♠–2♣–2♥–?`:

| South | Meaning, [Guaranteed] | Obligation / agreement, continuations |
|---|---|---|
| Pass | Adopts 2♥ | When there is no independent obligation |
| `3♥` | Natural, ♥ 5+, at least an 8-card fit, INV | Not GF. North evaluates Pass / 4♥ |
| `4♥` | Natural TO PLAY. Carries over the first-round ♥ 4+ | A 7-card fit is not excluded. No additional guarantee of five cards |

- Facing A's 3♥ invitation, North [Typical] leans toward declining with 13–14, compares with 15, accepts actively with 16–17, and strongly considers game with 18+. Shape exceptions are allowed; no fixed HCP decision and no double counting.
- After an NT answer in A–C, Pass / 3NT / a natural suit continuation are compared. A 2NT answer alone does not make the auction GF. After an answer in the original suit, the policy is to compare support, a rebid of one's own suit, NT and a safe stop.
- **The one fixed sequence for C**: in an uncontested auction by a responder who is not a passed hand, `1♣–1♦–1♥–1♠(artificial FSF-C)–2♣–4♥` is **natural TO PLAY**. No new guarantee of HCP or support length, no GF and no trump agreement is attached. The guarantees of the opening suit and the second suit are carried over. The FSF obligation to answer was fulfilled by 2♣, and independent obligations are kept. This is not carried over to the 3♥ invitation or to other B/C continuations. For the CPU's comparison, see [6.6](#66-choosing-fsf-and-comparing-3nt-with-4m).
- **Sequence-specific details other than the above — such as INV/GF of 3♥ after B's 2♥, or the minimum lengths and any new GF for C's individual rebids — are [Undefined] (U02).** The INV of A and the GF of D are not transplanted. Because the public minimum length of the first-round 1♦ is unsettled, an 8-card fit is not concluded from C's 2♦ support alone.

### 6.4 D sequence: South after the reverse

`1♣–1♠–2♦` is a reverse by North with ♣ 5+, ♦ 4+, ♣ longer than ♦, and [Typical] 17–21. South showed ♠ 4+ in the first round, and South has the F1 of the reverse. Every sequence below includes this beginning.

| South | Natural/artificial, [Guaranteed] | [Typical], obligation |
|---|---|---|
| `2♠` | Natural, ♠ 5+ | Weak, aiming to stop, NF |
| `2♥` | Artificial FSF, no ♥ length | [6.1](#61-preconditions-and-initiation); a new F1 for North |
| `2NT` | Natural. No additional strict guarantee of HCP, shape or ♥A/KQ | Suitability for NT is required. Typically 6–9, weak, aiming to stop, NF. Not a game invitation. Not limited to balanced hands |
| `3♣` | Natural, ♣ 2+ | Reluctant preference, NF. At least a 7-card fit |
| `3♦` | Natural, ♦ 3+ | Reluctant preference, NF. At least a 7-card fit |

- A valid response fulfills the F1 of the reverse. On an NF route, North may pass if there is no other obligation.
- 2♠ does not guarantee North's support. A ♣/♦ preference does not guarantee a positive 8-card fit. With a weak hand of ♥ 5+ and ♠ 4, the first response 1♥ is preferred by [4.2](#42-first-response-choice-with-weak-hands), but a human's legal 1♠ is not prohibited.

**North facing the direct 2NT**: after `1♣–1♠–2♦–2NT–?`, Pass is the standard candidate. 3NT is natural TO PLAY with no new GF. The prospect of making nine tricks is weighed, and 3NT is actively compared from about 19 HCP, but this is not a fixed threshold. South's typical 6–9 is used for the estimate; 2NT is interpreted neither as an invitation nor as a guarantee of a ♥ stopper. The evaluation uses only one's own hand and the public history. **Other continuations, such as North's 3♣, are [Undefined] (U05).**

### 6.5 D sequence: South after the FSF answer

| Beginning of the complete sequence | South | [Guaranteed], natural or not | Obligation, trump agreement, continuations |
|---|---|---|---|
| `1♣–1♠–2♦–2♥–2♠` | Pass | Adopts 2♠ | Allowed if there is no other obligation |
| Same | `3♠` | Natural, ♠ 5+, at least an 8-card fit | **GF, ♠ agreed**. North's 4♠ is a choice of game; 4NT is Blackwood |
| Same | `3NT` | Natural TO PLAY | No new GF |
| Same | `4♠` | Natural, ♠ 4+, TO PLAY | A 7-card fit is accepted |
| `1♣–1♠–2♦–2♥–2NT` | Pass | Adopts 2NT | Allowed if there is no other obligation |
| Same | `3♠` | Natural, ♠ 6+ | **GF, trumps not agreed**. With ♠ 0–1, North normally bids 3NT; with two, compares 3NT / 4♠ |
| Same | `3NT` | Natural TO PLAY | No new GF |
| `1♣–1♠–2♦–2♥–3♣` | Pass | Adopts 3♣ | Allowed if there is no other obligation |
| Same | `3NT` | Natural TO PLAY | South's ♥A/KQ is in principle a [CPU criterion], not a public guarantee |
| Same | `4♠` | Natural, ♠ 6+, TO PLAY | North holds ♠ 0–2, so an 8-card fit is not guaranteed |
| Same | `4♣` | Natural, ♣ 3+, at least an 8-card fit | **GF, ♣ agreed**. Not only for slam |
| Same | `5♣` | Natural TO PLAY | Evaluates the ♣ fit and eleven tricks. Not chosen merely because 3NT is dangerous |

- North's 2♠ shows 3–4 cards, so together with South's original four ♠ there is a 7-card fit. Even with the GF 3♠, the sequence after the 2NT answer creates no ♠ agreement. Additional continuations not in the table are [Undefined].

**After the natural 4♣ GF**: in `1♣–1♠–2♦–2♥–3♣–4♣–?`, North's 5♣ is natural TO PLAY and fulfills the GF. North's 4NT is Plain Blackwood after ♣ agreement ([9.2](#92-plain-blackwood)). A Pass of 4♣ that conforms to the system is not possible. No cue bids are added. The obligation to answer and the GF are kept separately.

### 6.6 Choosing FSF and comparing 3NT with 4M

- [CPU criterion] In the one fixed sequence for C ([6.3](#63-fixed-continuations-and-boundaries-for-ac), `1♣–1♦–1♥–1♠–2♣–4♥`), when its own hand plus the public minimum lengths give eight or more ♥, the CPU compares 4♥ with 3NT and Pass (4♥ is not mandatory). This is not carried over to the 3♥ invitation or to other B/C continuations.
- [CPU criterion] When choosing FSF, the marginal value of further information is rated low when a major 8-card fit is already known. Merely having a suit to check does not earn the maximum bonus across the board. Abrupt differences in the base score at HCP boundaries are smoothed. No bonus for the chance of slam success or the value of exploration is added without grounds. The coefficients are internal CPU criteria.
- [CPU criterion] When 3NT and 4M are evenly matched, a small preference for the major is allowed. A clear NT advantage is not overturned, and the same candidate is not adjusted twice.

---

## 7. Notrump system and Stayman

1NT carries the guarantees of [3.1](#31-opening-bids) (D01). Stayman is optional. Apart from the specified sequences below, Stayman and transfers are not extended to the 2NT opening.

### 7.1 Direct responses to 1NT

| Complete context | Meaning, [Guaranteed] | Obligation / continuations |
|---|---|---|
| `1NT–2♣` (ST01) | Stayman ([7.2](#72-stayman)) | F1 |
| `1NT–2♥ / 2♠` (direct) | Natural, **5+** in the suit bid, weak, aiming at a partscore, NF | Not a transfer. A long major with game interest normally goes through Stayman |
| `1NT–2♦` (direct) | **[Undefined]** | Not replaced by a transfer or the like |
| `1NT–2NT` | Natural INV | [Typical] 6–8 |
| `1NT–3NT` | Natural TO PLAY | [Typical] 9+ |

- [Typical] A direct weak 2M is typically 0–7. None of these is an HCP guarantee. "Always six cards" is not added to the 5+ of a direct 2M over 1NT.
- A direct 4♣ over 1NT is Gerber, and a direct 4NT is quantitative 4NT ([chapter 9](#9-gerber-blackwood-and-quantitative-4nt)).

### 7.2 Stayman

1. **Initiation**: `1NT–2♣` (ST01). Optional.
2. **Meaning of the ask**: artificial. Asks whether opener holds a four-card major. No guarantee of ♣ or major length. F1 (obligation to answer). [CPU criterion] The CPU uses it with typically 8+, as a guideline for game exploration with a four-card or longer major / a long major. The ask itself carries no strict HCP guarantee.
3. **Answers and guarantees**:

| Sequence | Meaning, [Guaranteed] | Obligation |
|---|---|---|
| `1NT–2♣–2♦` | Artificial, three or fewer cards in each major | Fulfills the first-round F1; no GF |
| `1NT–2♣–2♥` | Artificial, four ♥. **With four cards in both majors, ♥ first** | Does not deny four ♠. Fulfills the first-round F1 |
| `1NT–2♣–2♠` | Artificial, four ♠ and three or fewer ♥ | Fulfills the first-round F1 |

4. **Further asks and answers**:

| Sequence | Meaning, [Guaranteed] | Obligation / continuations |
|---|---|---|
| `1NT–2♣–2♥–2♠` | Artificial. Responder has ♠ 4+ and asks whether opener holds four ♠ | F1. With four ♠ opener bids `3♠` (natural raise, NF); otherwise `2NT` (artificial, three or fewer ♠, NF) |
| `1NT–2♣–2♦–2♥ / 2♠` | Natural, **5+** in the major bid, invitational or stronger | F1. With three-card support opener bids `3M` (natural, NF); with two or fewer, `2NT` (artificial, NF) |
| `1NT–2♣–2♠–3♥` | Natural, ♥ **5+**, invitational or stronger with game interest | F1. With three ♥ opener bids `4♥` (natural); with two or fewer, `3NT` (natural) |

5. **Stopping and choice of contract**: [CPU criterion] Continuations with a long major after Stayman are typically 8+. A natural raise of the answered 2M to three is an invitation with four-card support, typically 8 points. Not every Stayman sequence is GF. Even after the answer to the ask, any other obligation remains.
6. **Intervention**: if an opponent makes a numeric bid before the answer, the obligation to answer is cancelled (including `1NT–(Pass)–2♣–(2♠)`; [11.3](#113-intervention-over-artificial-asks)). Over an opponent's Double, the obligation to answer remains ([11.3](#113-intervention-over-artificial-asks)).
7. **Boundaries of what is undefined**: a direct 2♦ is [Undefined]. Connecting Stayman to Gerber afterwards, competition after Stayman and the like are [Undefined]. Stayman is not applied to the 2NT opening.

ST03: in the text of the system, this name is used only to refer to the artificial asks that follow Stayman; there is no defining text. The implementation uses the internal IDs ST03-S / ST03-5M / ST03-5H for the continuations above (`2♥–2♠`, `2♦–2M`, `2♠–3♥`) ([14.4](#144-name-only-ids)).

---

## 8. Jacoby 2NT

1. **Initiation**: a direct 2NT over a natural 1♥/1♠, by a responder who is not a passed hand, in an uncontested auction (D09).
2. **Meaning of the ask**: artificial, **4+ (strict)** in M, GF, agrees M as trumps. Not an NT proposal. [Typical] 13 HCP is the base proposal, but there is no guaranteed minimum. Fit, shortness, honors and sources of tricks are evaluated; it is an entry point to slam exploration as well as to game, but does not promise slam. Strong three-card support is not included; use a natural new suit or a suitable F01.
3. **Answers and guarantees (opener's artificial shortness answers)**:

| Agreed | `3♣` | `3♦` | `3♥` | `3♠` | `3NT` |
|---|---|---|---|---|---|
| ♠ | ♣ 0–1 | ♦ 0–1 | ♥ 0–1 | No side shortness | Not assigned |
| ♥ | ♣ 0–1 | ♦ 0–1 | Not assigned | ♠ 0–1 | No side shortness |

- "No side shortness" means 2+ in every side suit. With more than one shortness, any of them may be shown; [CPU criterion] the CPU shows the lower suit first. 0 and 1 are not distinguished. The artificial 3NT is not a contract proposal and does not release GF.

4. **Further asks and answers (responder's ace-location report)**: starting with the next numeric bid above the previous answer, the steps are assigned to no ace / ♣A / ♦A / ♥A / ♠A.

| Previous | No ace | ♣A | ♦A | ♥A | ♠A |
|---|---|---|---|---|---|
| `3♣` | `3♦` | `3♥` | `3♠` | `3NT` | `4♣` |
| `3♦` | `3♥` | `3♠` | `3NT` | `4♣` | `4♦` |
| `3♥` | `3♠` | `3NT` | `4♣` | `4♦` | `4♥` |
| `3♠` | `3NT` | `4♣` | `4♦` | `4♥` | `4♠` |
| `3NT` | `4♣` | `4♦` | `4♥` | `4♠` | `4NT` |

- At most one ace is reported. "No ace" means a total of zero. The absence of an ace that was not reported is not guaranteed. The ♣♦♥♠ priority when holding several aces is a [CPU criterion]. An ace in the shortness suit is also included. Kings are not reported at this stage.

5. **Stopping and choice of contract**:
   - `1♥–2NT–3NT–4NT` reports the ♠A and is not Blackwood. Since the same 4NT cannot be bid again, a legal choice is made from the next 5♥ / 6♥ / 7♥ and so on. Overshooting the stop in this way is accepted.
   - If the report is the agreed 4M itself, it can be adopted by passing under [9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles). Otherwise GF is kept, and a new legal contract or an available Blackwood is chosen. It is not treated as having located all the aces.
6. **Intervention**: [Undefined] (Jacoby 2NT is not among the asks covered by the rules on intervention over artificial asks ([11.3](#113-intervention-over-artificial-asks))).
7. **Boundaries of what is undefined (U09)**: further confirmation of 0/1, a strict numeric division between an ordinary GF raise and a slam-oriented one, and the full routes for strong three-card support hands with neither a suitable natural suit nor a balanced shape are [Undefined]. The shortness table, the one-ace report and F01 of this chapter are not changed, and no new artificial responses are added.

[CPU unsupported] The CPU does not initiate Jacoby 2NT and implements neither the answers nor the continuations above ([chapter 13](#13-appendix-a-cpu-implementation-status-and-constraints)).

---

## 9. Gerber, Blackwood and quantitative 4NT

### 9.1 Gerber

1. **Initiation**: a **direct 4♣ over a natural 1NT/2NT opening**. A 4♣ immediately after a 2NT answer to FSF does not automatically become Gerber.
2. **Meaning of the ask**: artificial, asks for aces, F1 (obligation to answer).
3. **Answers and guarantees**:

| Ask | 0 or 4 | 1 | 2 | 3 |
|---|---|---|---|---|
| `4♣`: aces | `4♦` | `4♥` | `4♠` | `4NT` |
| After the ace answer, the asker's `5♣`: kings | `5♦` | `5♥` | `5♠` | `5NT` |

4. **Continuations**: the king ask is, in principle, for when all four aces are confirmed and a grand slam is being considered. **There is no queen ask [Not adopted].**
5. **Stopping and choice of contract**: a natural final contract other than 5♣ is TO PLAY. If there is no other obligation, the answerer passes.
6. **Intervention**: [11.3](#113-intervention-over-artificial-asks).
7. **Boundaries of what is undefined**: connecting Stayman to Gerber afterwards is [Undefined] ([7.2](#72-stayman)). The natural 4♣ of the D sequence is not Gerber.

### 9.2 Plain Blackwood

1. **Initiation**: **4NT after trump agreement**, except in explicitly specified artificial report or asking sequences.
2. **Meaning of the ask**: artificial, asks for the total number of aces. An independent obligation to answer arises.
3. **Answers and guarantees**:

| Ask | 0 or 4 | 1 | 2 | 3 |
|---|---|---|---|---|
| `4NT`: aces | `5♣` | `5♦` | `5♥` | `5♠` |
| After the ace answer, the asker's `5NT`: kings | `6♣` | `6♦` | `6♥` | `6♠` |

- Neither the trump king nor a void counts as an ace. An ace already reported after Jacoby 2NT is included in the total and not counted twice. Kings are counted as a total including the trump king.
- 0 and 4 are kept as a set of possibilities and resolved only when one's own hand and the public information make it unique. Hidden hands of other seats are not consulted.
- **RKCB and a dedicated queen ask are [Not adopted].**

4. **Continuation (5NT)**: 5NT is limited to the context of a king ask by that asker. In principle it is used after confirming all four aces, judging that at least a small slam can be played. Reasonable exceptions with a missing ace, because of a void or a source of tricks, are allowed. It is not automatically applied to 5NT in other contexts.
5. **Stopping and choice of contract**:

| Agreed trump suit | Constraints on stopping after the ace answer |
|---|---|
| ♠ | From 5♣ / 5♦ / 5♥, 5♠ is possible. A 5♠ answer can be adopted by passing |
| ♥ | From 5♣ / 5♦, 5♥ is possible. A 5♥ answer can be adopted by passing. From 5♠ there is no way back to 5♥ |
| ♣ (including the D sequence) | Only the 5♣ answer can be adopted by passing to stop in 5♣. After 5♦ / 5♥ / 5♠, a ♣ contract is at least 6♣ |

- Bidding the same contract again is prohibited. King answers also follow the actual levels of 6♣ / 6♦ / 6♥ / 6♠; for example, with ♥ agreed there is no way back from 6♠ to 6♥. The risk that an answer overshoots the intended stop is accepted, and no new stopping exception is created. 5NT is not used as an escape.
6. **Intervention**: [11.3](#113-intervention-over-artificial-asks).
7. **Boundaries of what is undefined**: advanced slam exploration is not covered. Cue bids are [Not adopted] ([12.4](#124-not-adopted)).

### 9.3 Quantitative 4NT

- When the strength of a natural NT is known and there is no need to look for a trump suit, **4NT with no trump suit agreed** is an NF invitation to 6NT. Artificial asks and Jacoby reports take precedence. No aces/kings and no GF are guaranteed.
- [Typical] Direct 4NT over 1NT (16–18): responder typically has a balanced 15–16. Opener declines with 16, accepts with 18, and with 17 compares sources of tricks and the like.
- [Typical] Direct 4NT over 2NT (22–24): responder typically has a balanced 9–10. Opener declines with 22, accepts with 24, and compares with 23.
- The numbers are guidelines, not strict guarantees. With 13–14 opposite 1NT, normally 3NT; with special working values there is room to explore. Unspecified details are not reinterpreted as asks and answers.

[CPU unsupported] The CPU neither initiates nor answers quantitative 4NT ([chapter 13](#13-appendix-a-cpu-implementation-status-and-constraints)).

### 9.4 Ending by passing an artificial answer, and slam principles

- After the answer, if the contract of the artificial answer **coincides with game or higher in the agreed trump suit** and no other obligation is unfulfilled, the asker/opener may adopt it as final by passing. Because that adoption meets the condition for reaching game, GF does not circularly prohibit the Pass.
- A GF below game, an unanswered ask, or another F1 is not released. This rule does not allow passing a 5♦ answer with ♣ agreed (this is separate from a human's Pass that is legal under the Laws).
- The number and location of aces, shortness, trumps, kings and sources of tricks are evaluated. Even four aces do not guarantee success, and with too few aces exploration is still possible through shortness and the like. Extreme distribution and answers that overshoot the stop are accepted; the height of an answer itself is not mistaken for strength. **Cue bids are [Not adopted] (not used).**
- This section is a rule for when a trump suit has been agreed. It does not apply to the 4NT ask over the special 4M ([9.5](#95-4nt-ask-over-the-special-4m)), where there is no trump agreement.

### 9.5 4NT ask over the special 4M

1. **Initiation**: `4NT` made by partner as their first bid immediately after a special 4M opening (`4♥`/`4♠`, [3.6](#36-special-4m-d04)), with no intervention by the opponents (no call other than Pass).
2. **Meaning of the ask**: artificial. Asks for the total number of aces held by opener. F1 (obligation to answer). It is distinguished by context from Plain Blackwood (after trump agreement), quantitative 4NT (a natural NT context) and the 4NT ace report after Jacoby 2NT ([1.5](#15-order-of-interpretation)).
3. **Answers and guarantees**:

| Sequence | Answer | Total aces |
|---|---|---|
| `4M–4NT` | `5♣` | 0 or 4 |
| Same | `5♦` | 1 |
| Same | `5♥` | 2 |
| Same | `5♠` | 3 |

4. **Further asks and answers**: if the asker bids **the step directly above** the ace answer, it asks for the total number of kings; directly above the king answer, it asks for the total number of queens. King and queen answers use the same four steps (0/4, 1, 2, 3), starting with the next numeric bid above the ask. There is no further ask after the queen answer.
   - The bid directly above an answer is always the next ask and never a natural stop. Example: 5♥ in `4♥–4NT–5♦–5♥` asks for kings.
5. **Stopping and choice of contract**:
   - When the asker makes a legal numeric bid that is not an ask, it is a natural final contract (TO PLAY).
   - The asker declares the final contract after the answer. **Adopting a contract by passing an artificial answer is not a stop that conforms to the system.** A human's legal Pass is accepted and recorded as an off-system stop. The CPU never stops by passing an artificial answer. Adoption by passing under [9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles) does not apply, because there is no trump agreement.
   - The answers give totals, not locations. 0 and 4 can be told apart only when one's own hand and the public information make it unique.
6. **Intervention**: if an opponent makes a numeric bid before the answer to the ask or to a further ask, the obligation to answer is cancelled. Over an opponent's Double, the obligations and meanings of the ask and the answer are maintained. If an opponent makes a numeric bid after the answer and before the asker's next turn, the asking sequence ends and the asker's obligation to declare the final contract is released (the meaning of later calls is [Undefined]). After an opponent's Double, the asker still declares the final contract ([11.3](#113-intervention-over-artificial-asks)). If an opponent intervenes before the 4NT, that 4NT is not the ask of this section ([Undefined]).
7. **Boundaries of what is undefined**: responses and continuations over 4M other than 4NT are [Undefined] (U04).

[CPU criterion] The CPU asker chooses the final contract **in the suit of opener's 4M**: at the six level if its own aces (or kings) plus the answer total three or more, otherwise at the five level (raising the level if that bid is illegal or would be the step directly above the answer). If a total of four is confirmed, it makes the next ask.

---

## 10. Ichigaya Refrain

### 10.1 Position

- Official name **Ichigaya Refrain**, system ID AF01. The nickname when it is used is **Ichigaya Fever**, shown in logs as `ICHIGAYA FEVER`.
- The 4♣ opening is **an artificial start to confirm the last missing piece of information**, made when opener has already settled on the suit/NT to play and the target contract. Its role differs from the strong natural two-bid ([3.5](#35-responses-to-strong-natural-two-bids-f02)), which builds the contract by looking at partner's distribution and fit.

### 10.2 CPU-only evaluation

Everything in this section is a [CPU criterion], not a public guarantee.

- Before the ordinary openings, if `HCP≥21 OR longest suit≥7 OR expected playing tricks≥7`, the dedicated evaluation is entered. Passing the entry is not the same as choosing the Refrain.
- Playing tricks are evaluated from the actual cards, independently of HCP. AKQJ109 plus another ace counts as seven tricks for the entry. The Kx in AKQ / AKQ / Kx is not counted as a sure trick.
- The final evaluation is a ledger of pluses and minuses for expected tricks, solid top cards, length, conditional kings/queens, shortness, typical distribution, missing cards, entries, control, and the value of the ask. Winners and shape are not counted twice. The three unknown hands, including partner's, are taken into account; partner is not assumed to hold the needed cards. Somewhat optimistic initial coefficients and an adjustable threshold are used. The policy of accepting an unfavorable distribution of up to about 50% is not a strict probability guarantee.
- The threshold for choosing the Refrain is an adjustable internal CPU value, not a value of the system.
- The OR entry and the total-count A→K→Q are maintained. An 8-card suit headed by AKQ, several aces and the like are not required conditions. Self-sufficiency and the value of an answer changing the evaluation of the target contract go into the ledger, and the Refrain is compared with an ordinary opening, the special 4M and a direct final contract. The coefficients are adjustable internal CPU values. Unknown locations are not added at full value as the value of the ask. A location is settled only when it can be uniquely inferred from the answer. If one's own 13 cards already contain the needed cards, no ask is needed and a direct seven-level contract in the suit can be compared. Before starting, the legal stopping contracts other than the answer steps and the next ask are checked; the CPU does not steer into 4♣ every time.

### 10.3 Asks and answers

1. **Initiation**: `4♣` as the first numeric bid (AF01). Shows nothing about club length.
2. **Meaning of the ask**: artificial, asks for the total number of aces. Obligation to answer.
3. **Answers and guarantees**: four steps starting with the next numeric bid above the ask: 0/4, 1, 2, 3.

| Sequence | Total aces |
|---|---|
| `4♣–4♦` | 0 or 4 |
| `4♣–4♥` | 1 |
| `4♣–4♠` | 2 |
| `4♣–4NT` | 3 |

4. **Further asks and answers**: if the asker bids the step directly above the ace answer, it is a **king ask**; directly above the king answer, a **queen ask**. King and queen answers use the same four steps. There is no further ask after the queen answer, and the step directly above it is also a candidate for the final contract.
5. **Stopping and choice of contract**:
   - There is no obligation to ask all of A→K→Q. The asker can stop by proposing any legal final contract other than the step directly above an ace or king answer. Example: 4♥ in `4♣–4♦–4♥` is a king ask, not a sign-off in 4♥.
   - The asker must not treat a trump suit that only the asker has in mind as publicly agreed and adopt a contract by passing an artificial answer. A human's legal Pass is accepted and recorded as an off-system stop ([12.1](#121-off-system-calls-by-a-human)). The CPU never stops by passing an artificial answer.
   - The asks obtain **the total number of aces/kings/queens**, not their location. With three aces of one's own plus partner's one ace, the location of the missing ace is unique; with two of one's own plus partner's one, it is unknown which one is missing. The value of information from before the ask is not kept after the answer; only conditions that are logically settled are updated incrementally.
   - Before the ask and before each further ask, a legal stopping plan is checked. If an ordinary answer would leave no reasonable stop, the evaluation for starting is lowered. Overshooting caused by a rare high answer is accepted, and no complicated new exception is created. Once the entry has been passed, the entry reason, base tricks, shape, expected value of the ask, risk, final value, threshold and the reason for choosing or not choosing are recorded, including when the Refrain is not chosen.
6. **Intervention** ([11.3](#113-intervention-over-artificial-asks)): even after an opponent's Double, the meanings, the obligation to answer and the asker's choice of contract after the answer are unchanged. An opponent's numeric bid before the answer (for example, `4♣–(4♥)`) cancels the obligation to answer.
   - **An opponent's numeric bid after the answer**: if, after an artificial answer about aces, kings or queens, an opponent intervenes with a numeric bid before the asker's next turn, that asking sequence ends and the asker's obligation to declare the final contract arising from it is released. Over an opponent's Double after the answer, the sequence and the obligation to declare are maintained. Independent obligations that have arisen separately, such as F1 or GF, are not cancelled. No new meaning or guarantee is given to our side's calls after the intervention (undefined competition). A human may still make any legal call.
7. **Boundaries of what is undefined**: advanced re-evaluation after the answers is not specified in detail by the system (the principles are in item 5 above).

---

## 11. Competition, Double and intervention

### 11.1 Scope and common principles

- Of the opponents' interventions, this chapter covers **only the complete contexts** defined in the sections below. General competition (overcalls, takeout), low-level Doubles and Redoubles are [Undefined] except as defined in this chapter.
- Where there is no provision for intervention (uncontested R1/R01/R02, Stayman, FSF, Jacoby, Gerber, Blackwood, the Refrain), the rules of each chapter apply. This chapter does not change them.
- A human's legal off-system call is accepted.
- Cancellation is not fulfillment. When an obligation is cancelled, its source, the position of the call that cancelled it, and the reason are recorded (record format: [13.3](#133-basic-fallback-principles)).
- Cancellation and a Pass made for lack of candidates or by evaluation are different diagnostics.
- The meaning received is never changed on the basis of hidden cards or the internal intentions of the opponents' CPU.

### 11.2 Penalty Double (PENALTY-HIGH)

- A **Double of the opponents' natural suit contract at the four level or higher** is compared as a penalty candidate aiming to defeat the contract on defense.
- [CPU criterion] From its own defensive cards, the opponents' public fit and partner's public strength, the CPU estimates the defensive tricks needed and the shortfall. Relatively stable winners such as aces are kept apart from conditional winners such as kings/queens, and a ruffing risk is applied to the second round of AK in a long side suit. Even an ace is not called an absolutely guaranteed trick. Partner's initial Pass lowers the expectation of help, but not to zero.
- No strict HCP, number of defensive tricks or obligation for partner to respond is added to the Double. Four sufficiently sure tricks are not the only necessary condition; the Double is compared relative to the gains and risks of Pass.
- **The meanings of low-level Doubles, takeout Doubles and Doubles of artificial contracts are [Undefined].** They are not confused with the penalty Double of a high natural contract. The CPU does not start an undefined takeout on its own.

### 11.3 Intervention over artificial asks

Asks covered: Stayman (ST01) and the asks that follow it (ST03), Ichigaya Refrain (AF01), the 4NT ask over the special 4M, Gerber, and Plain Blackwood. Jacoby 2NT is not covered ([chapter 8](#8-jacoby-2nt), item 6, [Undefined]).

**(a) An opponent's numeric bid before the answer**

- If an opponent makes a numeric bid after an artificial ask and before the answer, **the obligation to answer that ask is cancelled**. Because the level has risen, the original answer is not forced unconditionally. This applies to every numeric bid, not only 2♠ (`1NT–(Pass)–2♣–(2♦/2♥/2♠/3♣…)`, `4♣–(4♥)` and so on). The same applies when the seats rotate.
- The ask remains artificial. No guarantee such as denial of a major, a strength limit or stoppers is added to a Pass after the cancellation.
- The answerer may pass. The asker may also pass if there is no other obligation.
- Example: in `1NT–(Pass)–2♣ Stayman–(2♠)`, the obligation to give the specified ST01 answer is cancelled, and opener may pass. 2♣ remains artificial, and the Pass carries no information about denying a major or about stoppers. Responder may also pass if there is no other obligation. 3♦ is not used as a substitute for the original 2♦ answer.
- Even if, after the intervention, the same bid as the original answer is made, it carries no answer meaning (treated as undefined competition). No substitute answers above the intervention and no new artificial Doubles are created [Not adopted].
- How a renewed ask after the cancellation is treated is [Undefined].
- A GF or natural F1 that has arisen separately is not cancelled.

**(b) An opponent's Double**

- Even if an opponent doubles the artificial ask or its answer, the level has not risen, so **the obligations of the ask and the answer are maintained**.
- The answers use the same calls with the same meanings as without the Double (total counts, whether a four-card major is held, and so on). The answerer's obligation remains, and a Pass is recorded as an obligation breach.
- The asker's choice of contract after the answer is also the same as without the Double. An artificial ask or answer is not left doubled as the final contract.
- The meanings of the Ichigaya Refrain and its obligation to answer do not change after a Double.
- The meaning of the Double itself remains [Undefined]. No guarantee is added about the doubler's hand. The Double does not make the asking sequence "unsupported".
- **A Redouble by the asking side is [Undefined].**

**(c) An opponent's numeric bid after the answer (Ichigaya Refrain, the 4NT ask over the special 4M)**

- If, after an artificial answer about aces, kings or queens, an opponent intervenes with a numeric bid before the asker's next turn, that asking sequence ends and the asker's obligation to declare the final contract arising from it is released ([9.5](#95-4nt-ask-over-the-special-4m), [10.3](#103-asks-and-answers)).
- Over an opponent's Double after the answer, the sequence and the obligation to declare are maintained ((b) above).
- Independent obligations that have arisen separately, such as F1 or GF, are not cancelled.
- No new meaning or guarantee is given to our side's calls after the intervention ([Undefined]).
- A human may still make any legal call.

### 11.4 Intervention over natural F1

Obligations covered: natural F1 (R1, R01, R02, REVERSE, SECOND and other "may not pass once" obligations).

**(a) An opponent's numeric bid immediately after a natural first response**

- Covered: natural one-of-a-suit opening – (Pass) – R1/R01 response – (opponent's numeric bid), only before opener's first rebid. The conditions of application concerning passes before the opening do not change.
- **Only the F1** that the R1/R01 created for opener is cancelled. Opener may pass, but may also compare natural candidates. The reservation of an R02 that has not yet arisen after R01 is also cancelled and is not revived by any later rebid.
- Any F1, GF or R02 that has already arisen separately is kept. Obligations to answer follow [11.3](#113-intervention-over-artificial-asks).
- Cancellation is not fulfillment of an answer. No guarantee of a strength limit, shortness or stoppers is added to a Pass. No assumption is made about whether the opponent's numeric bid is natural or artificial (including the example of 2♦ in a suit the opponents have already bid).

**(b) An opponent's Double, or a numeric bid not covered by (a)**

- Over an opponent's Double, the obligation is maintained (a Pass is recorded as an obligation breach).
- After the intervention (a Double, or a numeric bid not covered by (a)), the meaning of our side's calls becomes [Undefined]. In that case, if the obligated seat makes a numeric bid, that in itself fulfills the obligation (because it satisfies the content of the obligation, "do not pass once").
- Only a numeric bid fulfills it. A Redouble does not.
- No guarantee of meaning, length or strength is added to the call that fulfills it (it remains undefined).
- This does not apply to obligations to answer an ask ([11.3](#113-intervention-over-artificial-asks)) or to GF ([2.6](#26-ending-the-auction-under-gf)). GF is fulfilled only by a natural game contract.
- Without intervention, only a defined natural call (or a new F1 or GF) fulfills the obligation.

### 11.5 Limited natural raise after intervention

- Covered: only opener's cheapest raise of responder's major after natural one-of-a-suit opening – (Pass) – R1 response of one of a major – (opponent's numeric bid).
- Natural, **four or more cards of support**, NF, no strict HCP guarantee, no new GF. Not extended to three-card support or jump raises. Combined with the 4+ of responder's major, at least an 8-card fit. It does not automatically create a new trump agreement.

### 11.6 Intervention and obligations at a glance

The same content as [2.4](#24-obligation-state-transitions), arranged so that it can be looked up by type of intervention. Any combination not in the table is [Undefined].

| Obligation | Opponent's numeric bid | Opponent's Double | Reference |
|---|---|---|---|
| Obligation to answer (before the answer) | Cancelled (reason recorded). Both answerer and asker may pass if there is no other obligation | Maintained (same answers with the same meanings; a Pass is an obligation breach) | [11.3](#113-intervention-over-artificial-asks) (a)(b) |
| Asker's declaration of the final contract (after an answer to the Ichigaya Refrain or the 4NT ask over the special 4M) | The asking sequence ends; released | Maintained | [11.3](#113-intervention-over-artificial-asks) (c) |
| F1 that R1/R01 created for opener (immediately after the first response, before opener's first rebid) | Cancelled | Maintained (a Pass is an obligation breach) | [11.4](#114-intervention-over-natural-f1) (a)(b) |
| R02 reservation (after R01, not yet arisen) | Cancelled (not revived later) | — | [11.4](#114-intervention-over-natural-f1) (a) |
| Natural F1 not covered by the cancellations above (REVERSE, SECOND, an R02 that has arisen, and so on) | Maintained. After the intervention, fulfilled by a numeric bid by the obligated seat | Maintained (a Pass is an obligation breach). After the intervention, fulfilled by a numeric bid (not by Redouble) | [11.4](#114-intervention-over-natural-f1) (b) |
| GF | Not cancelled. However, the CPU's criterion for ending under GF does not apply in sequences with intervention by an opponent's numeric bid | Not cancelled | [2.6](#26-ending-the-auction-under-gf) |
| Obligations that have already arisen separately | Not cancelled (for obligations to answer, see the first row) | Not cancelled | [2.5](#25-when-several-obligations-exist-at-once) |

---

## 12. Undefined, not adopted and off-system calls

### 12.1 Off-system calls by a human

- A human may choose an off-system bid or Pass if it is legal. A human's actual call and the CPU's suggestion or hypothetical candidates are separate data. Only the actual history is used for the interpretation received; it is never replaced by the suggestion.
- The CPU follows the rules that apply. When it cannot interpret the auction, it logs the status, the remaining obligations and the lack of candidates, and continues legally. A fallback to prevent freezing is never disguised as conforming to the system.
- Even with UNDEFINED and similar statuses, natural candidates that can be evaluated from one's own hand and the known public information are compared. However, no natural guarantee is added to an artificial call. Pass is compared with the quality of the current contract, a lack of fit, missing a game, partner's chance to choose again, and the cost of raising the level. The CPU does not pretend to have evaluated every legal candidate; it separates what was actually compared, what was excluded and what is unsupported.

### 12.2 Undefined items (U01–U09)

The undefined items at present. This table does not approve any new meaning for a bid. The "related defined parts" are not included in the undefined scope.

| ID | Undefined scope | Related defined parts | Principles to keep |
|---|---|---|---|
| U01 | The strict minimum length of 1♣–1♦. A first-response 1NT over 1m. The detailed route for H1NT/S1NT with 11–12 when there is no suitable natural response | R1, H1NT/S1NT ([4.1](#41-first-responses)); first-response choice with weak hands ([4.2](#42-first-response-choice-with-weak-hands)) | The CPU's current minor 5+ is not made a public guarantee. A first-response 1NT is not newly created as a rescue. Adding meaning is a human decision. Logging the lack of candidates and continuing play are required |
| U02 | Lengths and INV/GF of the individual continuations of FSF B/C. Sequence-specific details other than after A's specified 2♥. The meaning of rebidding the fourth suit after FSF | After A's 2♥, and C's `1♣–1♦–1♥–1♠–2♣–4♥` (natural TO PLAY) ([6.3](#63-fixed-continuations-and-boundaries-for-ac)). The D sequence ([6.4](#64-d-sequence-south-after-the-reverse), [6.5](#65-d-sequence-south-after-the-fsf-answer)) | A's 3♥ INV is not carried over to B, nor D's 3♠/4♣ GF to C. Human judgment is required before the CPU enables candidates carrying new guarantees. Comparing candidates with known information and tracking the reasons for nonconformity are required |
| U03 | Whether the HCP of `1m–1M–2NT` is strictly guaranteed, and its forcing status. Details of showing a 19–21 hand with a long single suit unsuitable for NT | The route of a jump showing a strong balanced hand centered on about 19–21 ([4.8](#48-nt-bids-by-context)) | It is not defined as a 15–18 range or INV (INV is not treated as adopted). J05 (typically 16–18, INV) is not changed to GF. Human judgment is required before implementing the meaning. The difference from the implementation: [13.7](#137-known-differences-between-the-system-and-the-implementation) |
| U04 | Refining the shape guarantee of the 2NT opening. The strict minimum length of strong natural two-bids and how to handle 25+ with a longest suit of four. Detailed responses to weak three-level openings. Responses and continuations over the special 4M other than the 4NT ask | 2NT 22–24 ([3.1](#31-opening-bids)). The F1 of strong natural two-bids and response guidelines ([3.1](#31-opening-bids), [3.5](#35-responses-to-strong-natural-two-bids-f02)). The 6+ guarantee of weak three-level openings and response guidelines ([3.4](#34-weak-three-level-openings)). The meaning of the special 4M ([3.6](#36-special-4m-d04)) and the 4NT ask ([9.5](#95-4nt-ask-over-the-special-4m)) | The range and shape practice of 2NT are not changed, but conditions in the code are not made new public guarantees. **Making 2NT 19–21, and moving to an artificial strong 2♣, are [Not adopted]**. No public guarantee of length or HCP is added to the special 4M. Gaps are judged individually before declaring full support |
| U05 | Additional continuations not in the D tables, such as North's 3♣ after the direct 2NT in the D sequence | Pass / 3NT after the direct 2NT ([6.4](#64-d-sequence-south-after-the-reverse)). The 4♣ GF after the 3♣ answer ([6.5](#65-d-sequence-south-after-the-fsf-answer)) | Anything else is detected as undefined; cue bids and the like are not added on one's own initiative |
| U06 | The best contract after a human's off-system bid. Unspecified application to competition and passed hands. Within intervention over artificial asks: the meaning of the Double itself, a Redouble by the asking side, a renewed ask after the cancellation, and the meaning of our side's calls after the intervention | Obligations over numeric-bid and Double intervention on artificial asks, and intervention after the answer to the Ichigaya Refrain or the 4NT ask over the special 4M ([11.3](#113-intervention-over-artificial-asks)). Intervention over natural F1 ([11.4](#114-intervention-over-natural-f1)) | A human's legal actions are never stopped. Even when the best action is unspecified, play continues legally, and contradictions, unsupported cases and forcing violations are recorded. An exceptional choice of contract is never presented as conforming to the system. Guaranteeing that play continues is required |
| U07 | An invitational new suit after a simple raise of 1M. Takeout/overcalls in competition. Guarantees and obligations of special jumps | The simple raises J01/M01 by a passed hand ([4.1](#41-first-responses)). The penalty Double of a natural suit contract at the four level or higher ([11.2](#112-penalty-double-penalty-high)) | A new meaning is a human decision |
| U08 | (An internal CPU matter) Coefficients and boundaries for NT suitability, fit quality, accepting invitations, the Refrain and so on | The meanings and guarantees of each chapter | Coefficients of the CPU evaluation within an established meaning are an implementation decision (managed in code). The A/KQ of the FSF 2NT, support lengths and obligations are not relaxed. Borrowing a "balanced required" condition is avoided, and boundaries are tested against controls |
| U09 | Further confirmation of 0/1 after Jacoby. A strict numeric division between an ordinary GF raise and a slam-oriented one. The full routes for strong three-card support hands with neither a suitable natural suit nor a balanced shape | Shortness answers and ace reports ([chapter 8](#8-jacoby-2nt)), F01 ([4.1](#41-first-responses)) | The shortness table, the one-ace report and F01 of chapter 8 are not changed. No new artificial responses are added. These are stated explicitly as unsupported |

- The full priority order of first responses with weak two-suited hands is [Undefined]. The three conditions of [4.2](#42-first-response-choice-with-weak-hands) are applied explicitly, but they are not extended uniformly to lengths or strengths not in the table. The CPU's priority among candidates is handled separately from publicly shown guarantees of denial.

### 12.3 Other undefined items

| Item | Reference |
|---|---|
| A direct 2♦ over 1NT | [7.1](#71-direct-responses-to-1nt) |
| Connecting Stayman to Gerber afterwards; competition after Stayman | [7.2](#72-stayman) |
| Stayman and transfers over a 2NT opening (not applied) | [7.2](#72-stayman) |
| Low-level Doubles, takeout Doubles, Doubles of artificial contracts | [11.2](#112-penalty-double-penalty-high) |
| General competition, Doubles and Redoubles outside the complete contexts defined in chapter 11 | [11.1](#111-scope-and-common-principles) |
| The meaning of our side's calls after intervention | [11.3](#113-intervention-over-artificial-asks), [11.4](#114-intervention-over-natural-f1) |
| A Redouble by the asking side; a renewed ask after the cancellation | [11.3](#113-intervention-over-artificial-asks) |
| Intervention over Jacoby 2NT | [Chapter 8](#8-jacoby-2nt), item 6 |
| A 4NT after the opponents intervene immediately over the special 4M | [9.5](#95-4nt-ask-over-the-special-4m), item 6 |
| Extending J05–J07 to special jumps after a 2/1 | [4.6](#46-jump-rebids-j05j07) |
| Detailed responses to strong natural two-bids (beyond the guidelines in [3.5](#35-responses-to-strong-natural-two-bids-f02)) | U04 |

### 12.4 Not adopted

| Convention | Reference |
|---|---|
| Transfers (a direct 2♦ over 1NT is not a transfer; not extended to 2NT) | [Chapter 7](#7-notrump-system-and-stayman) |
| RKCB, and a dedicated queen ask for Blackwood | [9.2](#92-plain-blackwood) |
| A queen ask for Gerber | [9.1](#91-gerber) |
| Cue bids ("not used", "not added") | [6.5](#65-d-sequence-south-after-the-fsf-answer), [9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles) |
| Making 2NT 19–21; moving to an artificial strong 2♣ | U04 |
| Substitute answers over intervention; new artificial Doubles | [11.3](#113-intervention-over-artificial-asks) |

A takeout Double is [Undefined], not [Not adopted].

### 12.5 Required exception handling

Even where parts are undefined, the following are always required.

- Keep the status as DEFINED / PARTIAL / UNDEFINED / CONFLICT / UNSUPPORTED. Being undefined is never a reason to invent a new meaning.
- In a declared sequence for which a valid answer exists, keep the obligation. Do not enable a state in which only the artificial entry is implemented and there is no answer.
- Even if there is no preferred candidate because of a human's deviation or an unsupported case, never crash, freeze or stop play. Keep a legal fallback distinct from conformity with the system.
- If something unsettled in the system blocks a needed candidate, state that the sequence is partially supported and log the cause. Never present the product as a whole as "all rules implemented".
- Never forget independent obligations, and keep a human's actual history separate from the AI's suggestion. Never rescue a situation using the hidden cards of other seats.

---

## 13. Appendix A: CPU implementation status and constraints

This appendix describes the status of the CPU implementation, not the meaning of the system. The implementation covered is bidding AI `Ichigaya-1.8.0` (Bridge Atelier 1.14.6). Adjustable coefficients, thresholds and candidate scores are not included (they are managed in code).

### 13.1 Support by convention

| Convention | Initiates | Answers | Continuations | When unsupported |
|---|---|---|---|---|
| Stayman | Yes | Yes | Yes (the continuations of [7.2](#72-stayman)) | In undefined NT sequences and after intervention, continues legally (existing obligations are kept; cancellations are recorded with the reason) |
| Jacoby 2NT | **No** | **[CPU unsupported]** | **[CPU unsupported]** | — |
| Gerber | **No** | Yes (aces and kings) | Not applicable, because the asker is the human | In undefined cases and after intervention, continues legally and records obligation breaches separately |
| Plain Blackwood | Yes | Yes | Yes (stopping, comparing small slam and grand slam, 5NT, adoption by passing an artificial answer) | Same as above |
| Quantitative 4NT | **No** | **[CPU unsupported]** | — | — |
| Artificial FSF A–D | Yes | Yes | Yes, for continuations fixed by the system | Undefined continuations and intervention are not filled in (independent obligations are kept; continues legally) |
| Ichigaya Refrain | Yes | Yes | Keeps track of the king/queen ask state. Evaluation after the answers is simple. Never passes an artificial answer (a human's Pass is recorded as an obligation breach) | — |
| Special 4M (D04) | Yes (conditions in [13.4](#134-list-of-cpu-criterion-items-in-this-document)) | Responder bids 4NT or passes | Responses other than 4NT are undefined in the system | — |
| 4NT ask over the special 4M | Yes (as responder; conditions in [13.4](#134-list-of-cpu-criterion-items-in-this-document)) | Yes (aces, kings, queens) | Yes (next ask; final contract in the suit of opener's 4M. Never passes an artificial answer) | After intervention before the 4NT, it is not an ask (undefined) |
| Penalty Double | Yes (natural suit contracts at the four level or higher) | — | — | Does not start low-level Doubles or takeout |
| Handling of intervention ([chapter 11](#11-competition-double-and-intervention)) | — | Handles cancellation, maintenance and fulfillment | — | No general competition |

### 13.2 Partial implementation

- 2NT opening and strong natural two-bids: the CPU chooses them with its older selection logic (2NT is 22–24 with no five-card suit; a strong two-bid is 22+ in the longest suit). The meaning shown is PARTIAL (only partly defined), and the on-screen hint shows no numbers.
- Responses to strong natural two-bids (F02): the CPU responds with its older response logic.
- Competition: only the complete contexts of [chapter 11](#11-competition-double-and-intervention) and the penalty Double at the four level or higher. There are no general overcalls or takeout Doubles, and the CPU is passive when the opponents have bid first.
- Additional natural comparisons for passed hands and competition: partially supported (CPU hypotheses, not new public guarantees).
- Showing hints on hover is not implemented. The full post-board copy does not yet include the heading for the 52 cards.

### 13.3 Basic fallback principles

- Even when the auction cannot be interpreted or there are too few candidates, play continues legally ([12.1](#121-off-system-calls-by-a-human), [12.5](#125-required-exception-handling)).
- A Pass resulting from evaluation, a lack of candidates because the system is undefined, a case not supported by the implementation, and a fallback are recorded separately. A forcing violation is an independent marker.
- The reason markers shown on the decision review screen and in the logs (`UNSUPPORTED_FALLBACK`, `EXCLUDED:`, `PASS_OUTLOOK:` and so on) express this distinction.
- Recording the reason for a cancellation: intervention immediately after a natural first response ([11.4](#114-intervention-over-natural-f1) (a)) and the cancellation of the Stayman obligation to answer in `1NT–(Pass)–2♣–(2♠)` (the example in [11.3](#113-intervention-over-artificial-asks) (a)) are recorded as `P0-INTERVENTION`; other intervention by a numeric bid over an artificial ask ([11.3](#113-intervention-over-artificial-asks) (a)) is recorded as `INTERVENTION-BID`.

### 13.4 List of [CPU criterion] items in this document

Items recorded as [CPU criterion] in the body of this document:

| Item | Location in this document |
|---|---|
| Opening the special 4M: the longest suit is a major of eight or more cards headed by the A, K and Q, and the length of that suit plus the number of side aces is 10 or more. No HCP condition. Compared with other candidates by evaluation score | [3.6](#36-special-4m-d04) |
| Responding to the special 4M: 4NT with 14+ HCP and two or more aces; otherwise Pass | [3.6](#36-special-4m-d04) |
| The final contract chosen by the asker of the 4NT ask over the special 4M (in the suit of opener's 4M; at the six level with a total of 3 or more, otherwise at the five level) | [9.5](#95-4nt-ask-over-the-special-4m) |
| Conditions for opening a weak three-level bid (12 HCP or less, quality with six cards) | [3.4](#34-weak-three-level-openings) |
| First-response choice with weak hands (the three conditions facing 1♣) | [4.2](#42-first-response-choice-with-weak-hands) |
| Guideline for using Stayman (typically 8+) and for its continuations | [7.2](#72-stayman) |
| Priority among several shortnesses and several aces after Jacoby | [Chapter 8](#8-jacoby-2nt) |
| Evaluation when choosing FSF, major preference between 3NT and 4M, comparison of C's 4♥ | [6.6](#66-choosing-fsf-and-comparing-3nt-with-4m) |
| The Refrain's entry, ledger and handling of the threshold | [10.2](#102-cpu-only-evaluation) |
| Evaluation of the penalty Double | [11.2](#112-penalty-double-penalty-high) |
| How to end the auction under GF | [2.6](#26-ending-the-auction-under-gf) |
| South's ♥A/KQ for 3NT in the D sequence | [6.5](#65-d-sequence-south-after-the-fsf-answer) |

### 13.5 Hint and record principles

- Context-sensitive hints are required for artificial and forcing calls. They are shown mainly at the first step of selection, with hover as a supplement, and can be switched on or off. From the shared interpretation they show natural/artificial, guarantees, typical ranges, the answer requested, remaining obligations and the CPU's implementation status. In addition to explaining the meaning, they can show the same AI's suggestion, the candidates compared and the reasons for the evaluation. The suggestion is not guaranteed to be the best answer. Other seats' actual cards and forced corrections of the human are never used. Operating the display alone never changes the random state or the history.
- On a human's turn, the AI evaluation is obtained from the human's own hand, the public history and the same random state. Switching the display on or off, or showing it several times, does not advance the random state. The same result as the decision made when the call is confirmed is saved in the log, and the human's actual call, the suggestion and whether they match are kept apart. Candidates distinguish actual comparison from exclusion, and for each candidate the shared interpretation shows natural/artificial, obligation, guarantees and typical ranges. The meaning of partner's last call, the accumulated guarantees/estimates, the remaining obligations and whether Pass is allowed are also shown.
- A legal human call that differs from the suggestion is allowed, and later calls read only the actual history. The hint display never receives other seats' hands. Before the board ends, other seats' decisions and hands are not revealed. Post-board copies and logs use the same record of the evaluation. Verification through the GUI and verification through a test harness are reported separately.
- The full post-board copy includes both seeds, each seat's 13 cards and HCP, and the decisions at every turn (the heading for the 52 cards has not yet been added; [13.2](#132-partial-implementation)). The 52 cards are only for the human's post-board verification and are never passed to the AI as input. Candidates, evaluation scores, reasons for the choice and reasons for not evaluating must be traceable at every turn.
- For the artificial answers of an asking sequence (Ichigaya Refrain, the 4NT ask over the special 4M), the hint shows "Artificial answer" and the total count of aces, kings or queens, and does not show the "may pass" of the forcing status (the internal distinction NF). The asker's obligation is shown on the "Your obligation" line as "Bid a final contract, or the next ask the system allows. A pass is off-system". After the sequence has ended (for example, after an opponent's numeric bid following the answer), this obligation is not shown.

### 13.6 Implementation management

- The format of decision reasons and diagnostics (PASS_DIAGNOSTIC, PASS_OUTLOOK, GUARANTEE, CPU_ESTIMATE) is maintained.
- After every change, all regression tests are run (the Ichigaya AI regression tests and the regression tests of the whole game). Tests whose behavior was changed on purpose are updated with the reason written down.

### 13.7 Known differences between the system and the implementation

| Item | System | Implementation |
|---|---|---|
| `1m–1M–2NT` | A jump showing a strong balanced hand centered on about 19–21. Whether its HCP is guaranteed, and its forcing status, are [Undefined] ([4.8](#48-nt-bids-by-context), U03) | Treats opener's 2NT rebid as "NF, typically 15–18" regardless of context (based on reading the code; the on-screen display has not been checked) |

---

## 14. Appendix B: Index of system IDs and implementation IDs

- **Public system ID**: an identifier of this system. The official identifier of a rule.
- **Implementation identifier (for reference)**: an identifier used by the Bridge Atelier code. It appears as the "system ID" in the decision review, as `ruleId` in the decision log, and in the text export. **It is not an official identifier of the system.**
- Rules without a public system ID are referred to by the section numbers of this document.

### 14.1 Public system IDs

| Public system ID | Reference section | Meaning | Implementation identifier (for reference) | Shown on screen and in logs |
|---|---|---|---|---|
| D01 | [3.1](#31-opening-bids) | 1NT opening | D01 | D01 |
| D02 | [3.1](#31-opening-bids) | 1♥/1♠ and 1♣/1♦ openings (the same ID is used for both) | D02 | D02 |
| D04 | [3.6](#36-special-4m-d04) | Special 4M | D04 | D04 |
| AF01 | [Chapter 10](#10-ichigaya-refrain) | Ichigaya Refrain (4♣ opening) | AF01, AF01-14, AF01-ANSWER, AF01-ASK, AF01-FINAL | Same |
| F02 | [3.5](#35-responses-to-strong-natural-two-bids-f02) | Responses to strong natural two-bids | (none) | — |
| PREEMPT3 | [3.4](#34-weak-three-level-openings) | Weak three-level opening | PREEMPT3 | PREEMPT3 |
| J01 | [4.1](#41-first-responses) | 1M–2M | J01 | J01 |
| M01 | [4.1](#41-first-responses) | 1m–2m | M01 | M01 |
| J02 | [4.1](#41-first-responses) | 1M–3M | J02 | J02 |
| J03 | [4.1](#41-first-responses) | 1M–4M | J03 | J03 |
| J04 | [4.1](#41-first-responses) | 1x–jump new suit | J04 | J04 |
| R1 | [4.1](#41-first-responses) | The cheapest new suit at the one level | R1 | R1 |
| R01 | [5.1](#51-non-jump-21-r01) | Non-jump 2/1 | R01 | R01 |
| R02 | [5.2](#52-r02-and-ordinary-rebids) | Responder's obligation to rebid | (managed as a type of obligation) | — |
| H1NT | [4.1](#41-first-responses) | 1♥–1NT | H1NT | H1NT |
| S1NT | [4.1](#41-first-responses) | 1♠–1NT | S1NT | S1NT |
| F01 | [4.1](#41-first-responses) | Natural 1M–direct 3NT | F01 | F01 |
| D09 | [Chapter 8](#8-jacoby-2nt) | Jacoby 2NT | D09 | D09 |
| J05 | [4.6](#46-jump-rebids-j05j07) | Jump rebid carrying over the guarantee of the opening (INV) | J05 | J05 |
| J06 | [4.6](#46-jump-rebids-j05j07) | Jump in a second suit (GF) | J06 | J06 |
| J07 | [4.6](#46-jump-rebids-j05j07) | Jump raise of responder's suit (GF, trump agreement) | J07 | J07 |
| REVERSE | [5.3](#53-reverse-reverse) | Reverse | REVERSE | REVERSE |
| FSF A/B/C/D | [6.1](#61-preconditions-and-initiation) | The four sequences of artificial FSF | FSF-A, FSF-B, FSF-C, FSF-D | Same |
| FSF-C (notation referring to the context of C's fixed continuation) | [6.3](#63-fixed-continuations-and-boundaries-for-ac) | The context of C's fixed continuation | FSFC-C-4H | FSFC-C-4H |
| ST01 | [7.2](#72-stayman) | Stayman | ST01 | ST01 |
| ST03 | [7.2](#72-stayman) | The asks that follow Stayman (name only; [14.4](#144-name-only-ids)) | ST03-S, ST03-5M, ST03-5H | Same |
| PENALTY-HIGH | [11.2](#112-penalty-double-penalty-high) | Penalty Double of a high natural suit contract | PENALTY-HIGH | PENALTY-HIGH |
| SECOND | [11.4](#114-intervention-over-natural-f1) | Named only as an example of a natural F1 ([14.4](#144-name-only-ids)) | SECOND | SECOND |

### 14.2 Rules without a public ID, and implementation identifiers

| Reference section | Rule | Implementation identifier (for reference) |
|---|---|---|
| [3.1](#31-opening-bids) | 2NT opening, strong natural two-bids | LEGACY-OPEN |
| [4.5](#45-ordinary-rebids), [5.2](#52-r02-and-ordinary-rebids) | Opener's natural NT rebids | NT1, NT2, NT3 |
| [4.5](#45-ordinary-rebids), [5.2](#52-r02-and-ordinary-rebids) | Limited natural NT | LIMIT-NT |
| [4.5](#45-ordinary-rebids)–[4.7](#47-fixed-natural-nf-stopping-sequences) | General natural raises, preferences, rebids of one's own suit, second suits | SUPPORT, PREFERENCE, SELF, SECOND |
| [4.7](#47-fixed-natural-nf-stopping-sequences), [6.4](#64-d-sequence-south-after-the-reverse) | Fixed natural NF stopping sequences, weak responses in the D sequence | NFS-01 to NFS-09 |
| [4.8](#48-nt-bids-by-context) | `1m–2m–2NT` and accepting with 3NT | MINOR-NT-INV, MINOR-NT-ACCEPT |
| [6.3](#63-fixed-continuations-and-boundaries-for-ac), [6.5](#65-d-sequence-south-after-the-fsf-answer) | Fixed continuations of FSF | FSFC-A-INV, FSFC-D-CHOICE-GF, FSFC-D-CLUB-GF, FSFC-D-FIT-GF, FSF-CONTINUATION |
| [7.1](#71-direct-responses-to-1nt) | Direct 2M, 2NT and 3NT over 1NT | NT-WEAK-M, NT-INV, NT-GAME |
| [7.2](#72-stayman) | Stayman answers | ST02-D, ST02-H, ST02-S |
| [7.2](#72-stayman) | Answers to the asks that follow Stayman | ST04-S-YES, ST04-S-NO, ST04-3FIT, ST04-NOFIT, ST04-H-GAME, ST04-NT-GAME |
| [7.2](#72-stayman) | Game and invitations after Stayman | NT-M-GAME, ST-RAISE-INV |
| [9.1](#91-gerber) | Gerber | GERBER, GERBER-A, GERBER-K |
| [9.2](#92-plain-blackwood), [9.4](#94-ending-by-passing-an-artificial-answer-and-slam-principles) | Plain Blackwood; adoption by passing an artificial answer | D08, D08-A, D08-K, D08-ADOPT |
| [9.3](#93-quantitative-4nt) | Quantitative 4NT | QUANT4NT |
| [11.5](#115-limited-natural-raise-after-intervention) | Limited natural raise after intervention | P0-SUPPORT |
| [11.3](#113-intervention-over-artificial-asks), [11.4](#114-intervention-over-natural-f1) | Reason for a cancellation by intervention (recorded) | INTERVENTION-BID, P0-INTERVENTION |
| [9.5](#95-4nt-ask-over-the-special-4m) | 4NT ask over the special 4M, its answers, the next asks (kings, queens), the final contract | LEGACY4NT, LEGACY4NT-14 / -13 / -12, LEGACY4NT-ANSWER, LEGACY4NT-ASK, LEGACY4NT-FINAL |
| [1.4](#14-meaning-status) | Pass (existing obligations are checked independently) | PASS |
| — | Human input (card play) | human |
| [1.4](#14-meaning-status) | Cannot be interpreted / undefined | (empty, or UNDEFINED) |

### 14.3 Notes on IDs

- **D02** is used for both rows, 1M and 1m. They are not separate IDs.
- **D-series numbers**: the only numbers used in this system are D01, D02, D04 and D09. D03 and D05–D08 are not used. D08 is used as an implementation identifier for Plain Blackwood (it is not a system ID).
- **R1 and R01** are different. R1 = the cheapest new suit at the one level; R01 = non-jump 2/1.
- **J01–J07**: J01–J04 are first responses; J05–J07 are rebids.
- **F01 and F02** are unrelated (F01 = 1M–3NT; F02 = responses to strong natural two-bids).
- The **"D sequence"** (FSF D, [6.4](#64-d-sequence-south-after-the-reverse)) is different from D01/D02/D04/D09.
- The implementation identifier `J0x` is a form used inside the code to build J01–J03 and is not shown by itself.

### 14.4 Name-only IDs

| ID | Treatment in the text of the system | Status |
|---|---|---|
| ST03 | Used only as the name of one of the artificial asks; there is no defining text | The implementation uses ST03-S / ST03-5M / ST03-5H (the continuations of [7.2](#72-stayman)). A formal definition has not been decided |
| SECOND | Used only as the name of an example of a natural F1 | In the implementation, a natural new suit with 4+ (NF if it is opener's rebid, otherwise F1). A formal definition has not been decided |

---

© 2026 AF. This document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). For the terms of use of the game itself, see the [Terms of Use](../../TERMS.md).
