# Ichigaya Bidding System — Practical Guide

> Based on system version `Ichigaya-v1.1.6`. CPU support status refers to bidding AI `Ichigaya-1.9.1` (Bridge Atelier 1.14.8).

[日本語](../ja/bidding-system.md) · [Terminology](terminology.md) · [Full reference](bidding-reference.md)

## 1. How to use this guide

A compact lookup for players who know bridge but need a reminder of exact point ranges, suit lengths, and priority rules. The complete conditions and exceptions are in the **[Bidding Reference](bidding-reference.md)**.

| Category | Meaning |
|---|---|
| **Guaranteed** | A condition fixed by the system. It carries forward into later decisions |
| **Typical** | A typical range. Not a guarantee; actual hands may differ |
| **CPU criterion** | Internal criteria the CPU uses to choose its own calls. Not a public guarantee |

**A convention can be part of the system even when the CPU does not support it.**

| Symbol | Meaning |
|---|---|
| `M` | Major (♥/♠) |
| `m` | Minor (♣/♦) |
| `x` | Any suit |
| `5+` | At least five cards |
| `HCP` | High-card points |
| `BAL` | Balanced. In this guide, the 1NT shapes 4333, 4432 and 5332 (five-card minor only) |
| `NF` | Non-forcing (creates no new obligation to respond) |
| `INV` | Invitational (to game or slam) |
| `F1` | One-round forcing |
| `GF` | Game forcing |
| `TO PLAY` | Proposes to stop in that contract |
| `NT` | No Trump |

See [2.4 Obligations](#24-obligations) for the full meaning of `NF`, `INV`, `F1`, `GF` and `TO PLAY`.

## 2. Quick reference

### 2.1 Opening bids

| Call | Guarantees and obligations | Typical ranges, priorities, exceptions |
|---|---|---|
| `1♣ / 1♦` | Natural, minor **3+**. No HCP guarantee | Typically 13–21 HCP. Used without a five-card major. Longer minor first; **3–3: 1♣; 4–4 or longer equal minors: 1♦**. 1♦ does not promise 4+ |
| `1♥ / 1♠` | Natural, major **5+**. No HCP guarantee | Typically 13–21 HCP. Longer major first; **equal length: ♠** (including 5–5). With a five-card major, open 1M even with 16–18 HCP rather than 1NT |
| `1NT` | Natural, **16–18 HCP**, BAL. No five-card major | 4441 is not allowed |
| `2NT` | Natural, **22–24 HCP** | Current practice: 4333 / 4432 / 4441 with no five-card suit. Exact shape guarantees are undefined. **Show 19–21 HCP by opening one of a suit** |
| `2♣–2♠` | Strong natural two-bid. Natural, **F1** for responder. Not an artificial 2♣ | Typically 22+ HCP (not guaranteed). Minimum suit length is **undefined** |
| `3♣–3♠` | Weak three-level opening (preempt). First numeric bid only. Natural, **6+**, NF. No HCP guarantee (no upper limit either) | Typically about seven cards. Five or fewer does not conform to the system |
| `4♥ / 4♠` | Special 4M opening. Natural. A long, strong major that can expect game without partner's help. **Not a weak preempt** | No public guarantee of length, HCP or playing tricks. Creates no trump agreement. Partner's immediate 4NT asks for the total number of aces ([8.2](#82-4nt-ask-over-the-special-4m)); other responses are **undefined**. For the CPU's selection criteria, see [Reference 13.4](bidding-reference.md#134-list-of-cpu-criterion-items-in-this-document) |
| `4♣` | **Ichigaya Refrain**. `4♣` as the first numeric bid. Artificial, asks for the total number of aces, F1. **Shows nothing about clubs** | King and queen total-count asks are made only when needed |

### 2.2 Common responses

| Sequence | Guarantees and obligations | Typical ranges, priorities, exceptions |
|---|---|---|
| `1NT → 2♣` | **Stayman**. Artificial, asks about four-card majors, F1. Promises no club or major length and no HCP | Optional. Not used over a 2NT opening |
| `1NT → 2♥ / 2♠` | Natural, **5+** in the suit, weak, aiming at a partscore, NF | Typically 0–7 HCP. Not a transfer. With a long major and game interest, normally go through Stayman |
| `1NT → 2♦` | **Undefined** | Not treated as a transfer either |
| `1NT → 2NT` | Natural, INV | Typically 6–8 HCP |
| `1NT → 3NT` | Natural, TO PLAY | Typically 9+ HCP |
| `1♠ → 1NT` | Natural, NF. Does not promise BAL, stoppers in all suits, or any particular ♠ length | Typically 6–10 HCP |
| `1M → 2M` | Simple raise. Natural, **3+** support, NF | Typically 6–10 HCP |
| `1M → 3M` | Natural, **3+** support, **INV** | Typically 10–12 HCP. Opener typically declines with 13–14, accepts with 16+, and judges 15 by shape |
| `1M → 4M` | Natural, **5+** support with a side singleton or void, TO PLAY | Typically a weak distributional hand, 0–9 HCP |
| `1M → 3NT` | Natural, BAL with enough strength for game, TO PLAY. Promises no support for M | Typically 13+ HCP. Also usable with strong three-card support |
| `1♥ / 1♠ → 2NT` | **Jacoby 2NT**. Artificial, **4+** support, GF, agrees that major as trumps | Typically based on 13 HCP (no guaranteed minimum). Not a notrump proposal |
| `1x → non-jump 2/1` | Natural, minor **4+** (♥ **5+** for `1♠–2♥`). F1 on opener; responder must also rebid while below game (R01 / R02). **2/1 is not GF as a whole** | Typically 11+ HCP. Applies to `1♦–2♣`, `1♥–2♣/2♦`, `1♠–2♣/2♦/2♥` |
| `1x → jump new suit` | Natural, **5+** in the suit, **GF** | Typically 19+ HCP. `1♣–2♦` belongs here (not a 2/1) |

### 2.3 Stayman replies

| Sequence | Guarantees and meaning | Priorities and exceptions |
|---|---|---|
| `1NT–2♣–2♦` | Artificial. **At most three** cards in each major | — |
| `1NT–2♣–2♥` | Artificial. **Exactly four** ♥ | **With four cards in both majors, reply 2♥**. Does not deny four ♠ |
| `1NT–2♣–2♠` | Artificial. **Exactly four** ♠, **at most three** ♥ | — |

A 1NT opening never holds a five-card major, so the major shown is exactly four cards.

### 2.4 Obligations

| Label | Meaning |
|---|---|
| `NF` | Creates no new obligation to respond; **does not cancel existing obligations** |
| `INV` | Invitation to game or slam; partner may decline |
| `F1` | Partner must make one valid response |
| `GF` | The partnership must continue until game; no obligation to bid slam |
| `TO PLAY` | Proposes to stop in that contract; creates no new obligation |
| `R01 / R02` | Non-jump 2/1: opener rebids once; responder also rebids if still below game |

## 3. Reading the opening bids

Remember **five-card majors, three-card minors**. Open the longer major, with ♠ first when the majors are equal. Open the longer minor, with 1♣ on 3–3 and 1♦ on 4–4 or longer equal minors. A five-card major takes precedence regardless of minor length, and even with 16–18 HCP. One-of-a-suit openings are typically 13–21 HCP; this is not a guarantee (a normal opening with 12 HCP is not prohibited).

`1NT = 16–18 / BAL`; `2NT = 22–24`. A 5332 shape qualifies for 1NT only when the five-card suit is a minor. Show 19–21 HCP by opening one of a suit.

Strong natural two-bids (`2♣–2♠`) are natural, not an artificial 2♣. They are typically 22+ HCP, but neither HCP nor a minimum suit length is guaranteed. A weak three-level opening (`3♣–3♠`) applies only as the first numeric bid, guarantees 6+ cards in the suit, and typically has about seven. A special 4M opening (`4♥ / 4♠`) is a natural opening showing a long, strong major that can expect game without partner's help; it is not a weak preempt. There is no public guarantee of length or HCP, and no trump agreement arises. The only defined response is the immediate 4NT (an ask for the total number of aces, [8.2](#82-4nt-ask-over-the-special-4m)). See [Reference 3.6](bidding-reference.md#36-special-4m-d04).

## 4. Common responses and rebids

Focus on natural raises, the 1NT response, and non-jump 2/1. A simple raise of 1M (2M) is NF; a jump raise (3M) is INV, not GF.

After a non-jump 2/1, opener must rebid; responder must also rebid if the auction remains below game. **A locally NF call does not remove R02.** A jump new-suit response (including `1♣–2♦`) is GF.

A reverse is opener's natural rebid in a new suit ranking above the first suit, at the two level: second suit 4+, first suit longer, F1. Typically 17–21 HCP.

## 5. No Trump responses and Stayman

`1NT–2♣` is artificial Stayman and F1. Replies follow 2.3; with four cards in both majors, reply 2♥. Direct `2♥ / 2♠` are weak natural responses with 5+ cards (NF); `2♦` is undefined. **Transfers are not adopted.**

Main continuations after a Stayman reply:

- `1NT–2♣–2♥–2♠`: artificial. Responder shows 4+ ♠ and asks whether opener has four ♠ (F1). Opener bids `3♠` (natural, NF) with four ♠, otherwise `2NT` (artificial, NF).
- `1NT–2♣–2♦–2♥ / 2♠`: natural, 5+ in the major shown, invitational or better (F1). Opener bids `3M` (natural, NF) with three-card support, otherwise `2NT` (artificial, NF).
- `1NT–2♣–2♠–3♥`: natural, 5+ ♥, invitational or better with game interest (F1). Opener bids `4♥` with three ♥, otherwise `3NT`.

If an opponent intervenes with a numeric bid, the obligation to answer Stayman is cancelled. After an opponent's Double, the obligation remains.

## 6. Forcing and obligations

`NF` does not mean that passing is always allowed. Outstanding response and rebid obligations remain in force. Keep `F1`, `GF`, obligations to answer, and 2/1 `R01 / R02` distinct. GF is fulfilled by choosing a natural game contract, among other ways.

## 7. Representative artificial bids

| Convention | Initiation and meaning | CPU support (1.14.8) |
|---|---|---|
| Stayman | `1NT–2♣`, asks for a four-card major | Initiates, answers, and handles the main continuations |
| Jacoby 2NT | Direct `2NT` over natural 1M, 4+ support, GF | Does not initiate / answers and continuations not implemented |
| Gerber | Direct `4♣` over natural 1NT/2NT, asks for total aces; `5♣` asks for kings. The same whether or not responder is a passed hand | Does not initiate / answers total A and K counts |
| Plain Blackwood | `4NT` after trump agreement, asks total aces; `5NT` asks total kings | Initiates, answers, and chooses the contract |
| Quantitative 4NT | Invites 6NT (NF) when no trump suit is agreed and natural NT strength is known. Not an ace ask. Opener declines with Pass and accepts with `6NT`. A direct `4NT` over 1NT/2NT is the same whether or not responder is a passed hand | Initiates over 2NT (9–10 HCP), not over 1NT / as opener, decides whether to decline or accept |
| Direct 6NT | Direct `6NT` over natural 1NT/2NT. Names the natural final contract (TO PLAY, NF). Neither an invitation nor an ask. The same whether or not responder is a passed hand | Initiates over 2NT (11+ HCP), not over 1NT / as opener, adopts it by Pass |
| Artificial fourth suit forcing | Artificial fourth suit in four specified sequences; F1, not GF | Initiates, answers, and handles the continuations fixed by the system. Undefined continuations are not filled in |
| 4NT ask over the special 4M | `4NT` immediately after `4♥ / 4♠`, asks total aces, then total kings and queens ([8.2](#82-4nt-ask-over-the-special-4m)) | Initiates, answers, and chooses the final contract |

## 8. Ichigaya Refrain and the 4NT ask over the special 4M

Both start with an artificial ask for the total number of aces, and the bid directly above each answer can ask for the total number of kings, then queens. They do **not** ask where the aces, kings or queens are.

### 8.1 Ichigaya Refrain (4♣)

An artificial `4♣` made as the first numeric bid, asking for the **total number** of aces. It shows nothing about clubs. Partner must answer.

| Sequence | Answer (total aces) |
|---|---|
| `4♣–4♦` | 0 or 4 |
| `4♣–4♥` | 1 |
| `4♣–4♠` | 2 |
| `4♣–4NT` | 3 |

The answers are artificial: four steps starting with the next numeric bid above `4♣`. Zero and four share `4♦`; they can be told apart only when the hand and public information make it certain.

The asker may continue with king and queen total-count asks when needed. The bid directly above the ace answer asks for kings (for example, `4♥` in `4♣–4♦–4♥` is a king ask, not a sign-off), and the bid directly above the king answer asks for queens; the answers use the same four steps. **The bid directly above an answer is never a natural stop.** Asking all three is not required, and the asker may stop in any legal contract other than the bid directly above an answer. There is no further ask after the queen answer, so the bid directly above it is also a candidate final contract. Detailed sequences: [Reference 10.3](bidding-reference.md#103-asks-and-answers).

Its nickname when used is **Ichigaya Fever**. The CPU initiates it, answers it and chooses the final contract (its evaluation after the answers is a simple one).

### 8.2 4NT ask over the special 4M

`4NT` bid by partner as their first bid immediately after a special 4M opening (`4♥ / 4♠`), with no intervention by the opponents, is an artificial ask (F1) for opener's **total number of aces**. It is not Plain Blackwood (which needs an agreed trump suit) and not quantitative 4NT (a natural NT context). If an opponent intervenes immediately after the 4M, that 4NT is not this ask (undefined).

| Answer | Total aces |
|---|---|
| `5♣` | 0 or 4 |
| `5♦` | 1 |
| `5♥` | 2 |
| `5♠` | 3 |

The bid directly above the ace answer asks for kings, and the bid directly above the king answer asks for queens, with the same four steps. There is no further ask after the queen answer. **The bid directly above an answer is never a natural stop** (for example, `5♥` in `4♥–4NT–5♦–5♥` is a king ask). Any legal numeric bid that is not an ask is the final contract (TO PLAY). Responses and continuations other than 4NT are undefined. For the king and queen answer steps and how the CPU chooses the contract, see [Reference 9.5](bidding-reference.md#95-4nt-ask-over-the-special-4m).

### 8.3 Passing an artificial answer

After an answer in 8.1 or 8.2, the asker bids either the next ask or a numeric bid that is not an ask (the final contract). **Passing an artificial answer to make it the contract is an off-system stop.**

- A human's Pass is legal and is accepted. It is recorded as an obligation breach, and the hint's "Your obligation" line shows the obligation.
- The CPU follows the system and never stops by passing an artificial answer.
- Because no trump suit has been agreed, the Blackwood rule that lets an artificial answer be adopted by passing does not apply.

### 8.4 Intervention by the opponents

| Situation | Opponent's numeric bid | Opponent's Double |
|---|---|---|
| Before the answer to an artificial ask | Cancels the obligation to answer | The obligation to answer remains (same answers, same meanings) |
| After an artificial answer, until the asker's next turn | The asking sequence ends, and the obligation to declare the final contract is released | The asking sequence and the obligation to declare remain |

- The first row applies to artificial asks in general, including Stayman, Gerber and Blackwood.
- **The second row applies only to the Ichigaya Refrain and the 4NT ask over the special 4M.** It is not extended to Stayman, Gerber, Blackwood or other asks.
- Other obligations that arose separately, such as F1 or GF, are not cancelled along with it.
- Our side's calls after the intervention have no new meaning (undefined). A human may still make any legal call.
- Details: [Reference 11.3](bidding-reference.md#113-intervention-over-artificial-asks).

## 9. CPU support and notes

**Adopted, undefined, not adopted, and unsupported by the CPU are different statuses.** For example, Jacoby 2NT is part of the system, but the CPU does not initiate it and does not implement its answers or continuations. `1NT–2♦` is undefined in the system, and transfers are not adopted. Legal calls outside the system's defined meanings can still be made by a human player.

## 10. Full reference

The [Bidding Reference](bidding-reference.md) documents the complete guarantees, typical ranges, exceptions, answer steps, system IDs, and undefined cases (intervention: [chapter 11](bidding-reference.md#11-competition-double-and-intervention); undefined and not adopted: [chapter 12](bidding-reference.md#12-undefined-not-adopted-and-off-system-calls); CPU implementation status: [chapter 13](bidding-reference.md#13-appendix-a-cpu-implementation-status-and-constraints)). This is a practical guide and adds no meaning to the system. Important conditions may appear in both documents, but their meanings must agree; if they differ, the Bidding Reference prevails (its Japanese edition is the authoritative specification).

---

© 2026 AF. This document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). For the terms of use of the game itself, see the [Terms of Use](../../TERMS.md).
