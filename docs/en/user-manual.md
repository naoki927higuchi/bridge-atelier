# User Manual — Bridge Atelier

[README](../../README.md) · [Getting Started](getting-started.md) · [日本語](../ja/user-manual.md) · [Terminology](terminology.md)

This manual covers controls, settings, scoring, and review. The **Windows version is the full reference version**; Chapter 11 covers differences in the browser version. For individual bidding conventions, see the [Ichigaya Bidding System](bidding-system.md).

> This manual is based on Bridge Atelier 1.14.10.

## Contents

1. [Screen Overview](#1-screen-overview)
2. [Starting a Game](#2-starting-a-game)
3. [Auction](#3-auction)
4. [Hints](#4-hints)
5. [Card Play](#5-card-play)
6. [AI Controls](#6-ai-controls)
7. [Review](#7-review)
8. [Scoring and IMP](#8-scoring-and-imp)
9. [Sets and Seeds](#9-sets-and-seeds)
10. [Settings and Files](#10-settings-and-files)
11. [Windows and Browser Versions](#11-windows-and-browser-versions)
12. [Troubleshooting](#12-troubleshooting)

## 1. Screen Overview

The table has four seats: North, East, South, and West. In human-player mode, you play South and AI controls the other seats. You can also watch a four-AI table. Available information and controls change between the auction, card play, and post-board review.

- **Nameplates:** Each seat's nameplate shows the seat name. During the auction and after the board ends, each nameplate also shows that seat's **last call**. It is hidden during card play.
- **Status:** The center of South's nameplate shows what is happening now (for example, "Your turn — choose a bid", "West is thinking…", or "Paused (West's turn)").
- **Notices:** Short notices, such as the result of an action, appear at the bottom of the center column. They disappear after a few seconds; click a notice to close it at once.

## 2. Starting a Game

1. Launch the Windows release as instructed in the package, or open the browser version's page.
2. On the title screen, choose [日本語] or [English] at the top right if needed.
3. Use [Mode: Human + 3 AI ⇄] to switch between playing South and [4 AI (auto)] viewing mode.
4. To reproduce deals, expand [Details (research) ▼] and enter a **set seed**. Leave it blank for automatic generation.
5. Press [4 boards], [8 boards] or [16 boards]. **The set starts immediately, with no confirmation screen**, so choose the mode and seed first. A notice such as "4-board set started." appears (it does not show the set seed; see Chapter 9).

**Single deal:** After starting a set, choose [Single deal] (F5 on Windows) on the table toolbar. It does not count toward set results. [Back to set (board n)] resumes the set. You can change modes on a single deal, but not during a set board.

**Return to title:** Open set results with [Set…] (F2), then press [Title screen]. After confirmation, the set in progress is discarded and cannot be resumed. Language can only be changed on the title screen.

## 3. Auction

On your turn, choose a **call**. A call can be a **bid** (level and suit or No Trump), **Pass**, **Double (X)**, or **Redouble (XX)**. On your turn, South's nameplate shows "Your turn — choose a bid". Four consecutive opening passes produce a passed-out board, with no card play (the result appears without confirmation).

**Auction over:** In human-plus-three-AI mode, when the contract is determined, an "Auction over" panel appears in the center column. It shows the contract and declarer, dummy, and the opening leader; the auction stays visible on the left. Press [Start play ▶] (Enter on the keyboard) to begin card play. The opening lead is not made until you press it. There is no such confirmation in 4 AI mode or when viewing the AI table.

The Ichigaya Bidding System defines meanings, guarantees, and obligations for calls. A convention can be defined in the system without every AI continuation being implemented. See the [Bidding System](bidding-system.md) for details.

**Numbered bids:** Select a bid in the grid, then press [Confirm] (while a bid is selected, South's nameplate shows "Press [Confirm] to make the call"). Before confirming, you can select a different bid; selecting the same bid again clears the selection.

**Pass, Double and Redouble are made immediately when pressed and cannot be undone.** Illegal calls are dimmed and disabled. System obligations do not disable the Pass button.

Windows keyboard: 1–7 followed by C/D/H/S/N selects a bid; Enter confirms. P = Pass, X = Double, R = Redouble (all immediate); Esc clears a selection.

## 4. Hints

Auction hints appear in the center column **on the human player's auction turns in human-plus-three-AI mode**. They do not appear during card play.

| Setting | What you see |
|---|---|
| Off | No auction hints |
| Show system meanings | System meanings, guarantees and related information; **default** |
| Meanings + AI suggestion | System meanings plus an AI suggestion |

**An AI suggestion is not guaranteed to be optimal.** A **guarantee**, a **typical range**, and an **AI estimate** are distinct concepts. The current UI may combine the last two in one display. AI suggestions remain available in review even if auction hints were off.

Change the setting in [Settings] → [Auction hints] on the toolbar; the change applies immediately, even during an auction. The hint shows these items from top to bottom:

| Item | Contents |
|---|---|
| Your obligation | Your obligation under the system, or "None (you may pass)". Even where the system does not allow a pass, you can still pass |
| If you pass | What happens to the auction if you pass |
| Partner's last call | Partner's most recent call and its meaning |
| Artificial bids so far | Artificial bids made so far, including the opponents', with their meanings (only when there are any) |
| Selected | Meaning, guarantee and typical range of the selected bid; a prompt before you select one |
| AI suggestion | Top setting only. The call the AI suggests (not necessarily best) |
| Candidates and scores | Top setting only. The candidates the AI compared and their scores |
| Why suggested | Top setting only. Shown in the remaining space |

An answer to an artificial ask is labeled "Artificial answer". When the asker has received an artificial answer, "Your obligation" says to bid a final contract or make the next ask the system allows. For the meaning of each call, see the [Bidding System](bidding-system.md) and the [Complete Reference](bidding-reference.md).

Pass, Double and Redouble are made immediately, so there is no selected state for checking their meaning.

## 5. Card Play

Play cards according to the contract. You must follow the led suit when you have a card of that suit.

- **Human as declarer:** You decide and play cards from both your hand and dummy.
- **Human as defender:** You control your own hand and defend with an AI partner.
- **Human as dummy:** The control mode described in Chapter 6 applies.

When it is your turn to play, South's nameplate shows "Your turn" or "Dummy's turn (you play)". Click a card to select it, then **click it again** or press [Play]. Turn on [Play cards with one click] in Settings to play with a single click. Cards you cannot play, for example because you must follow suit, are grayed out and cannot be selected. **A played card cannot be taken back.**

[Last trick] shows the previous trick again, but once your side has played to the next trick, you cannot view it until the board ends. On Windows, ←→ selects a playable card, Enter plays it, and Esc clears the selection. When you are declarer, choose dummy's card from dummy's hand on dummy's turn.

## 6. AI Controls

When the human player (South) is dummy, after the opening lead reveals dummy's hand, choose one of these options:

| On-screen choice | Who decides and plays |
|---|---|
| [Take over as declarer] | You control both declarer's and dummy's hands |
| [Let the AI play (watch)] | The AI controls both hands |

**This choice cannot be changed during the board.** Taking over is not simply clicking cards chosen by the AI.

During card play, if you control a seat, [Let CPU play the rest] delegates the remaining play to the AI. A confirmation appears, and **once delegated, you cannot take control back during that board.** The score and IMPs still count as your result.

In 4 AI mode or after delegation, use [Pause], [Step], [1 trick] and [To end] on the toolbar. [To end] is unavailable while a human-controlled seat remains.

## 7. Review

At the end of a board, the result shows contract, outcome and score. Press [Review] to open the review. Switch between [View: Play ⇄] and [View: Auction ⇄] on the toolbar (Tab on Windows).

- **Auction:** Select a call in the history to inspect its meaning, reasoning, hands, partner estimates, alternatives and scores. Human calls show both the actual call and the **AI suggestion**, with a match indicator. AI calls show that AI's own decision.
- **Card play:** Select a card in the trick list to inspect reasoning and candidate evaluations (success rate and average tricks). For human plays, a reference AI choice is calculated on demand and may show a calculating indicator.
- **Navigation:** On Windows, ←→ moves one play and ↑↓ moves one trick (four plays). There is **no automatic replay**.

The auction summary includes the score breakdown, all four hands and HCP, deal seed, AI decision seed and related details. [Export text] is available in review (whether this remains in the public build is undecided). Set results also provide [Review] and [AI table], but you must finish the current board before reviewing another. In 4 AI mode, decision reasons may be viewed during a board.

**On-screen review and saved decision logs are separate features.** Windows saves human and AI decisions for each board, but not the AI table's decisions. The browser version does not write decision-log files.

## 8. Scoring and IMP

Bridge Atelier scores each board using the **duplicate bridge scoring table (Law 77)**. Scores depend on contract level and denomination, tricks taken, Double/Redouble, and vulnerability. Contract trick points start beyond the first six tricks; overtricks, bonuses, and undertrick penalties are calculated separately.

- **Total points:** Board scores are summed across the set (corresponding to Law 78C).
- **IMP comparison:** In human-plus-three-AI sets, the score difference against the **AI table**, where four AIs play the same deal, is converted to IMPs (Law 78B scale).
- **Dealer and vulnerability:** For set boards, these follow the board number (Law 2).

This **does not reproduce a human duplicate-bridge tournament or rankings**.

## 9. Sets and Seeds

A set contains **4, 8, or 16 boards**. The **set seed** determines the **deal seed** and **AI decision seed** for each board. With the same game version and set seed, the deals at the start of a set match across set lengths.

- **Deal seed:** Determines the card distribution. For a single deal, it also determines dealer and vulnerability; in a set, those follow the board number.
- **AI decision seed:** Used by both bidding AI and card-play AI.
- **Limits of reproducibility:** Results are not guaranteed across game versions. Card-play AI thinking level and human calls/cards may change subsequent decisions. Bidding AI does not use the card-play thinking level.

The set seed and deal seed accept **integers of up to 15 characters**, including the sign. Automatically generated values range from 1 to 2,147,483,646. The notice at the start of a set does not show the set seed. To check the set seed of the set in progress, open set results ([Set…]) and expand [Details (research) ▼].

[Replay this deal] (F6) replays the same deal. On a set board, **the new result replaces the score**, and the set results show the number of replays. A set in progress cannot be saved or resumed. Opening a saved deal starts a single deal.

## 10. Settings and Files

Open [Settings] on the toolbar. Changes **take effect and are saved immediately**. Language can only be changed on the title screen.

| Setting | Options and meaning |
|---|---|
| CPU thinking level | Fast / Normal (default) / Thorough. Changes card-play AI computation; the browser version starts at Fast on first launch |
| Pace | Slow / Normal / Fast / No wait. Controls delays between actions |
| Play cards with one click | Off (default: select, then confirm) / On (click plays at once) |
| Auction hints | Off / Show system meanings (default) / Meanings + AI suggestion |
| Advanced (research) | Deal seed, AI decision seed, computation display, data folder and more |

On Windows, files are stored under `%LOCALAPPDATA%\AF-Bridge-Claude\`, accessible via [Advanced (research)] → [Open folder].

| Location | Contents |
|---|---|
| `settings.json` | Settings |
| `SavedDeals/` | Saved deals (JSON) |
| `Logs/` | Decision logs and set records |
| `Exports/` | Exported review text |

[Save deal] and [Open deal] (Ctrl+S / Ctrl+O on Windows) save and load **initial deals**. A loaded deal becomes a single deal. **A set in progress cannot be saved or resumed.** The storage folder name may change for the public release.

## 11. Windows and Browser Versions

| Feature | Windows | Browser |
|---|---|---|
| Edition | All features | Main features (some are Windows-only) |
| Decision logs and set-record files | Supported | Not written |
| Deal and text exports | Local files | Browser downloads (release-build test pending) |
| Open a saved deal file | Supported | Not available ([Open deal] is dimmed and explains why when pressed). Deals saved in the browser version can be opened in the Windows version |
| Data folder | Per-user folder | None (settings are stored in the browser; [Open folder] is dimmed and explains why when pressed) |
| Settings persistence | `settings.json` | Browser storage designed; persistence after restart not yet verified |
| Keyboard shortcuts | F5, F11 / Alt+Enter, Ctrl+S, Ctrl+O and others | Some shortcuts unavailable due to browser conflicts |
| Card-play AI | Computed on a separate thread | Computed between screen updates; heavy deals may briefly freeze the display |
| AI table and IMP | Supported | Supported; comparison may take longer to appear |

The browser version does not guarantee the same persistent storage as Windows. Browser compatibility, downloads and performance require release-build verification.

## 12. Troubleshooting

| Symptom | First checks |
|---|---|
| Cannot choose a call | Confirm it is your turn and the call is legal. System obligations and UI restrictions are not necessarily the same thing |
| Cannot play a card | Confirm it is your turn and check the follow-suit rule |
| Cannot see an AI suggestion | Check that you are on a human auction turn and the hint setting; review provides a separate view |
| Same seed, different outcome | Check game version, thinking level, human actions, and board number in the set |
| No decision log in the browser version | The browser version does not save decision-log or set-record files |
| Card play does not start after the contract is set | Press [Start play ▶] in the "Auction over" panel in the center, or press Enter |
| A notice covers cards on the table | Notices disappear after a few seconds; click one to close it at once |
| Want to check the set seed | Open [Details (research) ▼] in set results |
| Pass was made as soon as I pressed it | Pass, Double and Redouble are made immediately and cannot be undone |
| Want to change the language mid-game | Only possible on the title screen; returning there discards the set in progress |
| Cannot change the mode | The mode is fixed when the set starts; you can change it on a single deal |
| [To end] is unavailable | It is unavailable while you control a seat; use [Let CPU play the rest] |
| Cannot view the last trick | Once your side plays to the next trick, it cannot be viewed until the board ends |
| Cannot review another board | Finish the board in progress first |
| Want full screen on Windows | F11 or Alt+Enter |

Use [GitHub Discussions](https://github.com/naoki927higuchi/bridge-atelier/discussions) for questions, feedback, and bug reports (the only official place for discussion; replies and fixes are not guaranteed). GitHub Issues are disabled at the initial release.

---

© 2026 AF. This document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). For the terms of use of the game itself, see the [Terms of Use](../../TERMS.md).

