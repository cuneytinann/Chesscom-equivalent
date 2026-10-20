**[English](#chesscom-equivalent)** · **[Türkçe](#turkce)**

# Chesscom-equivalent

A chess arbiter that plays by **Chess.com's rules** rather than FIDE's, in a single HTML file of **2,667 bytes** — and the same arbiter stripped of its board, in **1,050**. Two players, one screen. No libraries, no build step, no server. Download a file, double-click, play.

The two rulebooks are close, but they are not the same book, and every place they part ways is written down below.

Part of the [Golfstack](https://www.fidelite.art/) project.

## Play

| file | interface | bytes | GitHub Pages | project site |
| --- | --- | --- | --- | --- |
| `index.html` | clickable board, clock, Chess.com colours | 2,667 | [Chesscom-equivalent](https://cuneytinann.github.io/Chesscom-equivalent/) | [Chesscom-equivalent.html](https://www.fidelite.art/special/Chesscom-equivalent.html) |
| `numerical_packed.html` | square numbers typed into a `prompt()` box, no board | 1,050 | [numerical_packed.html](https://cuneytinann.github.io/Chesscom-equivalent/numerical_packed.html) | [Chesscom-eqv_min.html](https://www.fidelite.art/builds/special/Chesscom-eqv_min.html) |

On the project site both builds live under `special`, off to the side of the `L1`–`L3` ladder. They are not another rung on it; they follow a different rulebook.

Anything from late 2020 onwards will run them: Chrome 85+, Firefox 79+, Safari 14+. Three things set that floor — BigInt, which the starting position is written with; the `safe` keyword in `place-content`, which keeps the board reachable on a narrow screen; and the `||=` operator, which the packed build uses.

**On the board.** Click a piece, then click where it should go. Legal squares pick up a dot, a piece you can take picks up a ring, the square you selected, the square you came from and the square you landed on all take the same tint, and the square of a king in check turns red. After every move the board turns around to face whoever is to play. When a pawn reaches the last rank the file it landed on becomes the picker: the four choices stand on the board itself, on a white panel, and you click the one you want — no dialog, no extra row, nothing to dismiss. `½` offers a draw or accepts one; `⚐` resigns, and the two controls are built to match, white on colour. The clock starts at ten minutes and hands back five seconds a move.

**Without the board.** `numerical_packed.html` draws nothing at all. A square is two digits, rank first and file second, each counted from 0: a1 is `00`, e2 is `14`, h8 is `77` — the engine's own 0x88 index, written in hexadecimal. A move is the two squares written end to end: e2–e4 is `1434`. An `8` or a `9` in either digit points off the board, and the arbiter refuses it like any other illegal move. To promote, add a fifth digit — `0` bishop, `1` rook, `2` knight, anything else queen. A draw is offered the same way, by hanging a non-digit on the end of the move: `1434x` plays e2–e4 with an offer attached. Type the non-digit on its own and the offer still stands, but you owe a move afterwards. The opponent accepts by answering in kind; a plain move declines it and wipes it off. To resign, leave the box empty or press Cancel.

That is the one place the two builds part company, and it is a difference of gesture rather than of rule. On the board a draw offer is a standing flag you raise with `½` and lower the same way; in the dialog it rides along with a move. Either way the arbiter reads the same two bits, and either way the opponent's plain move turns the offer down. The dialog title carries White's clock, Black's clock and the last move you got past the arbiter, so when an illegal move is quietly refused you will see that the last one never changed. Both clocks start at 900,000 milliseconds — fifteen minutes, no increment — and they keep running while the dialog is open, so thinking costs you what it would over the board. Keeping track of the position is your job.

## Two files, one arbiter

These are not a full version and a cut-down one. Same game, same rules, same core — and **1,050 bytes is what that core really costs.** Everything on top of it exists for you, not for chess.

The 1,617 bytes `index.html` spends beyond the packed file buy a board you can see, pieces you can click, a clock that ticks, a promotion picker drawn onto the board, colours that tell you which squares you may use, and a status line. The arbiter underneath is the same arbiter.

## Why Chess.com and not FIDE

Chess has a rulebook, and then chess servers have rulebooks of their own. They agree on almost everything and disagree in a handful of interesting places. This arbiter follows Chess.com deliberately, and the decision was made by measuring its behaviour rather than by guessing at it — Chess.com's source is closed, so the evidence here is its own help pages, 30,198 finished games pulled from its Published-Data API, and 24,610 Titled Tuesday games that ended in repetition.

| ending | FIDE | Chess.com | here |
| --- | --- | --- | --- |
| checkmate, stalemate | — | — | ✓ |
| insufficient material | automatic | automatic, **different test** | automatic |
| threefold repetition | **claim** | **automatic** | **automatic** |
| fivefold repetition | automatic | *never reached* | *never reached* |
| fifty moves | **claim** | **automatic** | **automatic** |
| seventy-five moves | automatic | *never reached* | *never reached* |
| draw by agreement | — | — | ✓ |
| flag fall | loss | loss | loss |
| flag fall, opponent cannot mate | **draw, helpmate test** | **draw, material test** | **draw, material test** |
| resignation | loss | loss | loss |
| resignation, opponent cannot mate | **draw** | **loss** | **loss** |

The bold rows are where the interesting part lives.

**Threefold and fifty moves.** FIDE hands both to the player as a claim and has the arbiter declare them unasked at fivefold and at seventy-five. Chess.com does not wait: the third repetition and the hundredth ply end the game on their own. Which means neither the fivefold rule nor the seventy-five-move rule ever gets a chance to fire, and neither has any reason to exist here. There is no claim channel in this file at all — the draw control offers and accepts, nothing else.

**Resigning against a bare king.** Article 5.1.2 says a resignation loses the game *unless* the opponent could not deliver mate by any series of legal moves, in which case it is a draw. That clause entered the handbook in 2023. Chess.com does not apply it: resignation is a loss, full stop. So the `RM` code that the FIDE and lichess builds carry does not exist here, and `F` is not a function — the resign button writes the result directly.

**Running out of time.** Article 6.9 asks whether the opponent could mate *by any series of legal moves*, the losing side helping. Chess.com asks a narrower question: strip the flagged player's army down to a bare king, and ask whether the surviving player's material could mate **that**. A knight against a rook mates under FIDE, because the rook can be walled in; against a bare king it cannot. So the same position is a win on one rulebook and a draw on the other, and the draw is what this file returns as `TM`. Two real games from the scan, both from 2026:

```
3k4/5R2/3PPn2/5K2/8/8/8/8 w - - 3 67        White R+2P flags, Black has K+N   -> draw
8/8/6K1/5Np1/5b2/1r2kq2/8/8 b - - 3 81      White K+N survives Q+R+B+P       -> draw
```

**The two tests are not the same test.** The material question gets asked twice, once on the board and once at flag fall, and Chess.com uses a different threshold each time. On the board it asks whether mate can be **forced**; at flag fall, whether mate is **possible at all**. King and two knights is the one material that answers differently: it cannot force mate, so against a bare king the game ends on the spot as `IM` — but it can construct one, so a player who flags against it loses. Chess.com's own help pages say both, in two separate articles, and the scan confirms both. Five games where king and two knights won on time, five where it drew immediately against a lone king.

**Multiple bishops and square colour.** Older reports, the most recent from December 2024, say Chess.com counted any two bishops as mating material and ignored which squares they stood on. That is no longer true. Tested directly in 2026: king and two dark-squared bishops against a bare king is declared drawn at once, while opposite-coloured bishops play on. The file does the same — the per-side test reads the bishops' square colours, and any number of bishops confined to one colour cannot force mate.

**Two knights against a piece.** The other half of the two-knight rule is easy to get wrong. King and two knights draws immediately against a **lone** king only; give that king anything at all to be walled in with and the game continues, because a forced mate exists. Here that falls out of a threshold rather than a branch: a side that cannot force mate is weighed — nought for a bare king, one for a single minor or for any number of same-coloured bishops, two for a pair of knights — and the game is drawn when the two weights together come to less than three.

**Blocked positions.** A position can be dead with pieces to spare: the pawns interlock, and mate becomes impossible even if both players set out to arrange one. Article 5.2.2 calls that a draw on the spot. Chess.com does not look for it, and neither does this file — both play on until repetition or the fifty-move counter closes the game instead. Taking that detector out is most of what separates this build from its FIDE sibling on the project site.

**En passant: what the FEN says and what the counter does.** For repetition, FIDE counts two positions as the same only if a legal en passant capture is available in both or in neither. Plenty of engines write the en passant square after every double pawn push, whether or not anything can actually take. Nothing looks broken, because the capture gets refused anyway — but the repetition key comes out different, and a threefold that should have triggered arrives late or never arrives at all. Chess.com's FEN is written that way: of 244 scanned positions carrying an en passant square, **234 had no enemy pawn anywhere near it.** The plainest of them is an opening:

```
rnbqkbnr/ppp1pppp/8/3p4/3P4/5N2/PPP1PPPP/RNBQKB1R b KQkq d3 0 2
```

White has just played d2–d4 and `d3` is recorded, though Black has nothing on c4 or e4 to take with.

Its repetition counter does not read that field, though. Of the [Titled Tuesday](https://github.com/IgorRigolon/titled.tuesday) games from 2020, 2022, 2023 and 2024, **24,610** ended in repetition. Replayed under all three conventions — the square written unconditionally, written when an enemy pawn stands alongside, written only when the capture is legal — 24,403 of them end on the same ply every time. In the other **207**, one of the three occurrences comes right after a double push with nothing to take, and Chess.com ended every one of those 207 where the second and third conventions predict, and none where the first does. Angry_Twin – Hikaru, 12 March 2024:

```
40.Kf3 h5 41.Bd4 Rb8 42.Bc3 Rg8 43.Bd4 Rb8 44.Bc3 Rg8      1/2-1/2, repetition
```

The position after 40...h5 comes back after 42...Rg8 and 44...Rg8, and the game ends there. Had `h6` been counted, 44...Rg8 would only have been the second occurrence.

What separates the second convention from the third is a pawn pinned to its own king, and a [2025 report](https://www.chess.com/forum/view/help-support/chess-com-does-not-handle-threefold-repetition-correctly-if-it-involves-pinned-en-passants) shows Chess.com counting such a pawn as able to capture. So Chess.com is **pseudo-legal**: an enemy pawn beside the one that moved is enough, and a pin is not checked. This file does the same. After a double push it looks at the two neighbouring squares and writes the square only if an enemy pawn there could take it — geometry alone, the attack test `G` already has. The call into the legal move generator that the FIDE and lichess builds make is still not here, so `M` still does not recurse. It costs 40 bytes on the board file and 31 in the packed one.

*Measured in September 2026 against Chess.com's help centre, 30,198 of its finished games and 24,610 Titled Tuesday games that ended in repetition. Chess.com can change its mind later; this file cannot.*

## Result codes

Twelve ways a game can end: six decisive, six drawn.

| | | | |
| --- | --- | --- | --- |
| `W#` White mates | `B#` Black mates | `SM` stalemate | `IM` insufficient material |
| `WT` White wins on time | `BT` Black wins on time | `TM` flag fall, mate impossible | `50` fifty-move rule |
| `WR` White wins by resignation | `BR` Black wins by resignation | `3R` threefold repetition | `DA` draw by agreement |

A code beginning with `W` is a win for White and one beginning with `B` a win for Black; everything ending in `M`, beginning with `D`, or carrying a digit is a draw. `numerical_packed.html` reports the same twelve endings as numbers, with `4`, `5` and `15` left unused — those slots belonged to the seventy-five-move rule, fivefold repetition and resignation against a bare king.

The FIDE build on the project site has fifteen codes and calls insufficient material `DP`, dead position, because it detects blocked ones as well. Here the code is `IM`, which names what the file actually tests.

## What's in it

- **Every piece's movement**, worked out with arithmetic on a 0x88 board. No direction tables, no offset arrays.
- **Full legality.** A move that would leave your own king in check is never let through.
- **Castling** on both wings, with all of it checked: the right still standing, the rook's path clear, the king not in check, not crossing an attacked square, not landing on one.
- **En passant**, with Chess.com's pseudo-legal repetition key: the square counts when an enemy pawn stands beside the pawn that moved, pinned or not.
- **Promotion** to queen, rook, bishop or knight — four squares of the landing file on the board, a fifth digit without it.
- **A running clock.** Ten minutes plus five seconds a move on the board, fifteen flat minutes in the dialog.
- **Draw offers**, resignation, flag fall, and the material test that turns a flag fall into a draw.
- **All twelve endings**, told apart from one another.

## What's not in it

- No claim channel. Threefold and fifty moves fire on their own, so there is nothing to claim.
- No blocked-position detection, as above. Chess.com has none either.
- No helpmate search at flag fall, and no draw on resignation. Chess.com has neither.
- No engine, no takebacks, no FEN in or out, no PGN.
- No coordinates around the board. It turns around every ply, and the status line keeps out of the way.

For the full FIDE arbiter — fifteen codes, dead positions, eleven front ends — see [fidelite.art](https://www.fidelite.art/). For the lichess rulebook, see [Lichess-equivalent](https://github.com/cuneytinann/Lichess-equivalent).

## Colours

Sampled from screenshots of a live game rather than taken from a stylesheet, then checked back against the pixels.

| | | |
| --- | --- | --- |
| light square | `#EBECD0` | |
| dark square | `#739552` | |
| page behind the board | `#302E2B` | |
| highlight | `#FFFF33` at 50% | `#F5F681` on light, `#B9CA42` on dark |
| legal move | black at 13% | a dot at 22% of the square, a ring from 49% to 61% for a capture |
| king in check | `#E004`, red at 27% laid over the square | `#ECAD99` on light, `#946D3C` on dark |

Two of these are shorter than what they replace. Chess.com paints the square you selected and the two squares of the last move in **the same** yellow, where lichess uses two different greens — one ternary instead of two. And the move dot is a flat black tint, where lichess tints it green with its own alpha. The ring costs more: lichess fills the corners of the square, Chess.com draws a true annulus, which needs two more colour stops.

The alpha is written as `#FF38`, four-digit hex, rather than `#FFFF3380`. That puts the alpha at 0.533 instead of 0.5 and every channel lands within five of the measured value — four bytes for a difference no eye will find.

## Read it in the browser

`index.html` is not packed and not minified. **Right-click → View Page Source** (`Ctrl` `U`, or `⌥` `⌘` `U` on macOS) puts the whole program on screen, and that is what this repository is for: open the page, play a few moves, then go and read the source that just refereed them.

`numerical_packed.html` is the exception. It unpacks itself, and there is a section further down on how to open it up.

---

## Anatomy

Every byte of `index.html`, by part.

| part | bytes | |
| --- | --- | --- |
| markup and CSS | 594 | board, panel, controls, colours, layout |
| `<script>` tags | 17 | |
| state | 122 | the 0x88 board, castling rights, en passant square, counters, selection |
| `C` | 29 | which castling right a square forfeits |
| `G` | 237 | can this piece reach that square |
| `T` | 53 | is this square attacked — the king's, unless told otherwise |
| `U` | 135 | make the move on a copy: promotion, en passant victim, rook hop |
| `L` | 55 | is this move legal — make it, ask, take it back |
| `M` | 110 | play it for real: counter, castling rights, side to move, en passant square if a neighbouring pawn could take, position key |
| `V` | 97 | one side's material, counted the way Chess.com counts it |
| repetition table and clock | 42 | |
| `j` | 73 | the clock, and the flag-fall material test |
| aliases and setup | 109 | `innerHTML`, the piece glyphs, the 64 cells generated at load |
| `F` | 613 | draw the board, and the promotion picker on it |
| `E`, `K` | 350 | play the move and give the verdict, the buttons, the click |
| first draw and interval | 31 | |
| **total** | **2,667** | |

Sliced the other way (the way the project site colours the source, with the verdict after each move, the flag-fall test and the resignation and agreement verdicts counted as rules), the rules come to **1,050** bytes and the page that shows them to **1,617**. The referee is cheap and the stage is expensive, which is exactly the argument the packed file makes.

The names are those of FideLite's `L3`, and so are most of the functions: `C`, `G`, `T`, `U` and `L` are byte for byte the ones in FideLite's `engine.js`. `M` differs in one place — it asks `G` whether a neighbouring pawn could take, where FideLite asks `L` whether it legally can. The table's id is `u` rather than `T`, because `T` is now the check test. Two helpers of the lichess build are missing here: `s`, because a resignation is always a loss and the flag verdict fits on one line inside `j`, and `J`, which went with the claims.

## Verification

Both builds were checked by running them, and the rule layer was checked against Chess.com itself.

- **Against real games.** 30,198 finished games were pulled from Chess.com's Published-Data API, spanning October 2025 to September 2026, and every one of them was classified by the material standing in its final position and the result Chess.com recorded. About 2,400 immediate insufficient-material rulings and a thousand flag-fall rulings, and the file's verdict matches on every single class. The material combinations that Chess.com ends on the spot are exactly the combinations this file calls drawn, and the ones it plays on through — bishop and knight, opposite-coloured bishops, three knights, two bishops and a knight — are exactly the ones this file calls sufficient.
- **Against Chess.com directly.** The same-coloured bishop case appears in none of those 30,198 games, so it was set up by hand on the site and played out: two dark-squared bishops against a bare king is drawn at once, opposite-coloured bishops are not, one bishop each is drawn whatever the colours.
- **The en passant convention**, in two halves: the FEN side read off 244 scanned positions that carried an en passant square, the repetition side off 24,610 Titled Tuesday games that ended in repetition, 207 of which tell the conventions apart.
- **The en passant square against a reference**, on 1,500 random games and 115,317 moves: after every move the square this file writes was compared with python-chess's pseudo-legal test, and the legal move lists with python-chess's own. Not one difference in either. The previous build, which wrote the square unconditionally, differed on the square 9,251 times.
- **Against Chess.com's own endings.** The 89 decisive games from 2020 and 2024 were replayed through both builds' own repetition counters. Both now end all 89 on the ply Chess.com did; the previous builds ended none of them there. On 1,500 games where the conventions agree, old and new both match on all 1,500.
- **The material test**, twenty-three cases on the board layer, covering both thresholds and every combination that separates them, including the ones no real game produced.
- **Scripted games on both builds**, in Node against a stubbed `prompt()`: checkmate, the automatic threefold, resignation, an offer carried on a move and accepted, and an en passant capture. These were run on the previous build; the change since touches only when the en passant square is written, and the two checks above cover it.
- **Earlier rounds.** The switch to 0–63 square numbers and the rule optimizations of September and early October 2026 were each checked against the file before them, on the 8×8 engine both builds used at the time. The rewrite below replaced that engine.
- **The 0x88 rewrite of October 2026.** Both builds dropped their own 8×8 engine for FideLite's 0x88 one, written with `L3`'s names. `V` was run against the old `I` on 600,000 random positions, 300,000 sparse and 300,000 crowded, and gave the same count for either side every time — so the insufficient-material sum and the flag-fall threshold come out the same. `index.html` was run in Chrome side by side with its previous version, both pages clicked on the same screen squares, comparing after every step the 64 cells, the status line, both clocks, the offer light, the radio button, and the board, side to move, castling rights, en passant square, counters, repetition count and offers: 1,082 comparisons over 6 games, from the starting position and from eleven special positions, 25 screenshots identical byte for byte. Scripted endings — `W#`, `B#`, `SM`, `TM`, `BT`, `BR`, `IM`, the automatic `3R`, `50` on ply 100, castling, en passant, and a pinned pawn beside a double step, where the en passant square is written although the capture is illegal — came out the same on both. `numerical_packed.html` ran in lock step against the previous file — the old one fed 0–63, the new one the same squares as rank and file digits, with illegal, off-board and junk input mixed in — over 140 games and 3,873 legal moves, comparing every dialog title and the result code: no difference. Chrome plays fool's mate through both dialogs to the same code. The sizes went from 2,833 to 2,682 bytes and from 1,158 to 1,081.
- **The engine update of October 2026.** Both builds took the rule optimizations of FideLite's new `engine.js`, so `C`, `G`, `T`, `U` and `L` in `index.html` are again its functions byte for byte: the pawn branch closes with `?…:0` and `>b[f]` instead of `&&(…)` and `&!b[f]`, castling asks `T()+T(i+s)<S()`, the rook hop sits in `U`'s return value, `d-8?d-1024?8:Q:U(r,Q)`, and the starting position is laid out with `for(j in s=…)` instead of a `.map()`. The material count writes `(W?W<3>m:m<3)` for `(W?!m&W<3:m<3)`, and its weights `p%2?9:p/4&1` for `p%8==4||p%8-2&&9`: the same weights, since pawn, queen and rook are the odd codes and the knight is the one even code whose quarter is odd. No rule changed. `numerical_packed.html` took the pawn branch, the castling test and the weights; the Packing section says why the rest stayed out. The sizes went from 2,682 to 2,667 bytes and from 1,081 to 1,077. Each build was run in lock step against its previous version. `index.html` in Node, the same squares clicked in both, comparing after every click the board, side to move, castling rights, en passant square, counters, repetition count, offers, selection, status line and every cell's piece and style: 24 games, 4,832 comparisons, 14 promotions through the picker, flag falls ending in `TM`, `BT` and `WT`, and `IM` and `SM` endings. `numerical_packed.html` against a stubbed `prompt()`, with junk input, offers and resignations mixed in: 12 games, 861 inputs, the same result and final state every time. The material count `V` against its previous form on 20,000 random positions, both sides: the same count every time, with 0, 1, 2 and 9 all reached.
- **The renamed packing of October 2026.** `numerical_packed.html` was packed again after the letter search described under *Packing*, and it now carries the count's shorter test `d<3>n` as well. The size went from 1,077 to 1,050 bytes. The new file was run in lock step against the previous one, under a stubbed `prompt()` and a stubbed clock that now and then jumps far enough to drop a flag, with illegal, off-board and junk input, offers and resignations mixed in, and with captures favoured so that games run down to bare material: 360 games, 41,545 inputs, every dialog title and every result code the same — eight of the twelve codes came up: a mate, `IM`, `DA`, both flag falls, both resignations and `TM`. Unpacking the new file gives back its source exactly; RegPack 5.0.1 and 5.0.4 both rebuild it from that source byte for byte; and `reassignVars` changes nothing at any of the 350 settings.

perft passes on both new builds over the five standard positions — 197,281 from the starting position at depth 4, 97,862 from Kiwipete at depth 3, 43,238 from CPW position 3 at depth 4, 9,467 from position 4 at depth 3, 62,379 from position 5 at depth 3 — and the W3C Nu Html Checker returns zero errors and zero warnings on the new `index.html`. Writing the en passant square more often cannot add a move: `G` lets a pawn onto that square, and a pinned one is still thrown out by `L`.

## Unpacking

`numerical_packed.html` unpacks itself. Its script is a RegPack decompression loop that ends in `eval(_)`, so to get the plain source back you replace that one call:

```js
eval(_)   →   console.log(_)
```

Nothing in the loop touches the game, so it is safe to do this in Node. What falls out is 1,175 bytes of source.

**Download the file rather than copying it out of the browser.** The dictionary keys are control characters from the `\x01`–`\x1f` range, and the clipboard — or any editor that tidies up line endings — will quietly destroy them.

## Packing

[RegPack 5.0.4](https://github.com/Siorki/RegPack); 5.0.1 produces the identical file. These settings rebuild `numerical_packed.html` from that source **byte for byte**:

| option | value |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `1` |
| `crushCopiesFactor` | `0.5` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Stage 2 wins, the regexp character class: `[\x01-\x1fP-U]`, 37 tokens. The bytes land like this:

```
   8 B  <script>
1033 B  packed payload
   9 B  </script>
----
1050 B
```

Turning `reassignVars` on changes nothing: 1,050 either way, at every one of the 350 settings below. The renaming it would do has already been done, as the next section explains.

350 crusher combinations were tried — gain 0 to 3, length 0 to 2, copies 0 to 2 in steps of a half, tiebreaker 0 or 1. Eight of them tie at 1,033; the settings above are the first in that order, and the worst of the 350 lands at 1,047.

### The packed source is renamed for the packer

The source above runs to 1,175 bytes. It began as the packed lichess source with the rules swapped — and that one was cut down and renamed for RegPack rather than for a reader. The legal-move iterator and the verdict are folded into the one place each is used, the clocks live beside the position keys, and every name sits on a lower-case letter, so that the only capitals left in the source are the `D` of `Date` and the `O` of `indexOf` and RegPack can take `P`–`S` as dictionary tokens.

On top of that come the Chess.com rules: the material count `q` (`V` in `index.html`), written as `k?!m&k<3?k:9:m<3?!!m:9`, which packs two bytes smaller than the equivalent `(k?!m&k<3:m<3)?k+!!m:9` and two more with its counters renamed; the en passant square written through `G` rather than `L`; resignation always a loss; the automatic threefold; and no claims. A straight swap packs to 1,068, the material count's two rewrites bring it to 1,064. The engine update of October 2026 — the pawn branch closed with `?…:0` and `>b[f]`, the castling test `p()+p(i+s)<o()`, and the weights written `d+=p%2?9:p/4&1` — took the source to 1,176 bytes and the payload to 1,060. Its other three items stayed out at first, because on those names they packed larger: the starting position laid out by `for…in` by 3 bytes, the rook hop folded into `m`'s return value by 1, and the count's shorter test `m<3>n` for `!n&m<3` by 1.

Last, the letters were handed out again, by search rather than by hand — the same search the lichess README describes. Every name, each global and each parameter inside its own function, was tried on every letter it could take: the lower-case ones, `_` and `$`; the capitals stay with the dictionary. A move was kept only if the file then packed smaller, and only if every identifier still pointed to the same variable as before, checked by parsing the source again after every move; single moves and pairwise swaps took turns until neither found anything more. On the searched letters the payload drops to 1,034, and one of the three items now pays: the count's shorter test, `d?d<3>n?d:9:n<3?!!n:9`, brings it to 1,033. The other two still stay out — with the starting position laid out by `for…in` and the rook hop as well, searched the same way, the payload is 1,036. The dictionary now takes `P`–`U`, six capitals. The names stand like this:

| straight port (`L3_numerical`'s names) | packed source | |
| --- | --- | --- |
| `G` | `f` | reach; here also the repetition table and the clocks |
| `T` | `p` | is this square attacked |
| `U` | `m` | make the move on a copy |
| `L` | `l` | is this move legal |
| `M` | `s` | play it |
| `V` | `q` | the material count |
| `C` | `j` | castling rights |
| `R` | `a` | how many times this position has come up |
| `n`, `z` | `z`, `n` | the fifty-move counter, the result code |
| `K`, `Y` | `d`, `x` | the typed move, the target square |
| `P`, `S` in `G`; `P`, `Q` in `U` | `n`, `o`; `d`, `n` | piece type, the square walker; piece type, the square in between |
| `s`, `m`, `W` in `V` | `t`, `n`, `d` | the side counted, the colours of its bishops' squares, its material |

Two variables could also share one letter outright, wherever one is finished with before the other starts. That was tried and left out: test games cannot prove that two variables are never alive at the same time. RegPack's own `reassignVars` finds nothing left to do.

---

## Related

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — the full FIDE arbiter, fifteen result codes, eleven front ends
- [Lichess-equivalent](https://github.com/cuneytinann/Lichess-equivalent) — the same exercise, one server over
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — the other direction entirely: dead positions carried to 99.97%
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — the same rules a level down, letters and numbers side by side
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — the bare rules, packed, in 1,023 bytes

## License

MIT

---
---

<a id="turkce"></a>

# Chesscom-equivalent (Türkçe)

FIDE'nin değil, **Chess.com'un kurallarıyla** oynayan bir satranç hakemi; tek bir HTML dosyasında **2.667 bayt** — ve aynı hakemin tahtasından soyulmuş hâli, **1.050** baytta. İki oyuncu, tek ekran. Kütüphane yok, derleme adımı yok, sunucu yok. Dosyayı indir, çift tıkla, oyna.

İki kural kitabı birbirine yakın ama aynı kitap değil, ve ayrıldıkları her yer aşağıda yazılı.

[Golfstack](https://www.fidelite.art/) projesinin bir parçası.

## Oyna

| dosya | arayüz | bayt | GitHub Pages | proje sitesi |
| --- | --- | --- | --- | --- |
| `index.html` | tıklanabilir tahta, saat, Chess.com renkleri | 2.667 | [Chesscom-equivalent](https://cuneytinann.github.io/Chesscom-equivalent/) | [Chesscom-equivalent.html](https://www.fidelite.art/special/Chesscom-equivalent.html) |
| `numerical_packed.html` | `prompt()` kutusuna yazılan kare numaraları, tahta yok | 1.050 | [numerical_packed.html](https://cuneytinann.github.io/Chesscom-equivalent/numerical_packed.html) | [Chesscom-eqv_min.html](https://www.fidelite.art/builds/special/Chesscom-eqv_min.html) |

Proje sitesinde iki yapı da `special` altında, `L1`–`L3` merdiveninin yanında duruyor. Merdivenin bir basamağı değiller; başka bir kural kitabını takip ediyorlar.

2020 sonundan itibaren her şey çalıştırır: Chrome 85+, Firefox 79+, Safari 14+. Bu tabanı üç şey belirliyor — başlangıç pozisyonunun yazıldığı BigInt; dar ekranda tahtayı erişilebilir tutan `place-content` içindeki `safe` anahtar sözcüğü; ve paketli yapının kullandığı `||=` operatörü.

**Tahtada.** Bir taşa tıkla, sonra gideceği kareye tıkla. Legal kareler nokta alır, alabileceğin taş halka alır, seçtiğin kare, geldiğin kare ve gittiğin kare aynı rengi alır, şahı tehdit altındaki tarafın şah karesi kırmızıya boyanır. Her hamleden sonra tahta sırası gelene dönüyor. Bir piyon son yatayı bulduğunda indiği sütun seçiciye dönüşür: dört seçenek tahtanın üstünde, beyaz bir panelin içinde durur, istediğine tıklarsın — kutu yok, fazladan satır yok, kapatılacak bir şey yok. `½` beraberlik teklif eder veya kabul eder; `⚐` terk eder, ve iki denetim birbirine benzesin diye kuruldu: renk üstüne beyaz. Saat on dakikadan başlar ve her hamlede beş saniye geri verir.

**Tahtasız.** `numerical_packed.html` hiçbir şey çizmiyor. Bir kare iki rakamdır, önce yatay sonra sütun, ikisi de 0'dan sayılır: a1 `00`, e2 `14`, h8 `77` — motorun kendi 0x88 indeksi, on altılık tabanda yazılmış hâliyle. Hamle iki karenin uç uca yazılmışı: e2–e4 `1434`. Rakamlardan birinde `8` ya da `9` tahtanın dışını gösterir; hakem onu da her yasadışı hamle gibi reddeder. Terfi için beşinci bir rakam ekle — `0` fil, `1` kale, `2` at, başka her şey vezir. Beraberlik de aynı yoldan teklif edilir, hamlenin sonuna rakam olmayan bir karakter asarak: `1434x` e2–e4 oynar ve teklifi yanında taşır. Rakam olmayanı tek başına yazarsan teklif yine durur ama sonrasında bir hamle borçlusun. Rakip aynı şekilde cevap vererek kabul eder; düz bir hamle teklifi reddeder ve siler. Terk için kutuyu boş bırak ya da Cancel'a bas.

İki yapının ayrıldığı tek yer burası, ve bu bir kural farkı değil jest farkı. Tahtada beraberlik teklifi `½` ile kaldırıp aynı şekilde indirdiğin duran bir bayrak; diyalogda hamleyle birlikte yolculuk ediyor. Her iki hâlde de hakem aynı iki biti okuyor, her iki hâlde de rakibin düz hamlesi teklifi reddediyor. Diyalog başlığı Beyaz'ın saatini, Siyah'ın saatini ve hakemden geçirebildiğin son hamleyi taşıyor, yani illegal bir hamle sessizce reddedildiğinde sonuncunun hiç değişmediğini görürsün. İki saat de 900.000 milisaniyeden başlıyor — on beş dakika, artırım yok — ve diyalog açıkken işlemeye devam ediyorlar, yani düşünmek sana tahta başındaki kadara mal oluyor. Pozisyonu takip etmek senin işin.

## İki dosya, tek hakem

Bunlar tam sürüm ve kırpılmış sürüm değil. Aynı oyun, aynı kurallar, aynı çekirdek — ve **o çekirdeğin gerçek maliyeti 1.050 bayt.** Üstündeki her şey satranç için değil, senin için var.

`index.html`'in paketli dosyanın ötesinde harcadığı 1.617 bayt şunları satın alıyor: görebildiğin bir tahta, tıklayabildiğin taşlar, işleyen bir saat, tahtanın üstüne çizilen bir terfi seçici, hangi kareleri kullanabileceğini söyleyen renkler ve bir durum satırı. Altındaki hakem aynı hakem.

## Neden Chess.com, neden FIDE değil

Satrancın bir kural kitabı var, bir de satranç sunucularının kendi kural kitapları. Neredeyse her şeyde anlaşıyor, bir avuç ilginç yerde ayrılıyorlar. Bu hakem bilerek Chess.com'u takip ediyor, ve karar davranışını tahmin ederek değil ölçerek verildi — Chess.com'un kaynağı kapalı, o yüzden buradaki kanıt kendi yardım sayfaları, Published-Data API'sinden çekilmiş 30.198 bitmiş oyun ve tekrarla biten 24.610 Titled Tuesday oyunu.

| bitiş | FIDE | Chess.com | burada |
| --- | --- | --- | --- |
| mat, pat | — | — | ✓ |
| yetersiz malzeme | otomatik | otomatik, **farklı test** | otomatik |
| üçlü tekrar | **iddia** | **otomatik** | **otomatik** |
| beşli tekrar | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| elli hamle | **iddia** | **otomatik** | **otomatik** |
| yetmiş beş hamle | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| anlaşmayla beraberlik | — | — | ✓ |
| bayrak düşmesi | kayıp | kayıp | kayıp |
| bayrak düşmesi, rakip mat edemiyor | **beraberlik, yardımlı mat testi** | **beraberlik, malzeme testi** | **beraberlik, malzeme testi** |
| terk | kayıp | kayıp | kayıp |
| terk, rakip mat edemiyor | **beraberlik** | **kayıp** | **kayıp** |

Kalın satırlar işin ilginç kısmının yaşadığı yer.

**Üçlü tekrar ve elli hamle.** FIDE ikisini de oyuncuya iddia olarak veriyor ve beşli tekrarda, yetmiş beş hamlede hakemin sormadan ilan etmesini istiyor. Chess.com o kadar beklemiyor: üçüncü tekrar ve yüzüncü yarım hamle oyunu kendiliğinden bitiriyor. Bu da beşli tekrarın ve yetmiş beş hamle kuralının hiç ateşlenme fırsatı bulamaması demek, ve ikisinin de burada var olması için bir sebep kalmıyor. Bu dosyada iddia kanalı diye bir şey hiç yok — beraberlik kontrolü teklif ediyor ve kabul ediyor, başka bir şey yapmıyor.

**Çıplak şaha karşı terk etmek.** 5.1.2. madde, terk etmenin oyunu kaybettirdiğini söylüyor — *rakip legal hamleler dizisiyle mat edemiyorsa* beraberlik olması dışında. Bu cümle kitaba 2023'te girdi. Chess.com uygulamıyor: terk kayıptır, nokta. Dolayısıyla FIDE ve lichess yapılarının taşıdığı `RM` kodu burada yok, ve `F` bir fonksiyon değil — terk düğmesi sonucu doğrudan yazıyor.

**Sürenin bitmesi.** 6.9. madde rakibin *herhangi bir legal hamleler dizisiyle* mat edip edemeyeceğini soruyor, kaybeden taraf yardım ederek. Chess.com daha dar bir soru soruyor: bayrağı düşenin ordusunu çıplak şaha indir, ve hayatta kalan tarafın malzemesinin **onu** mat edip edemeyeceğini sor. Bir at, kaleye karşı FIDE'de mat eder, çünkü kale kendini kapatabilir; çıplak şaha karşı edemez. Yani aynı pozisyon bir kural kitabında kazanç, diğerinde beraberlik, ve bu dosyanın `TM` olarak döndürdüğü beraberlik. Taramadan iki gerçek oyun, ikisi de 2026'dan:

```
3k4/5R2/3PPn2/5K2/8/8/8/8 w - - 3 67        Beyaz K+2P bayrak düşürüyor, Siyah'ta Ş+A  -> beraberlik
8/8/6K1/5Np1/5b2/1r2kq2/8/8 b - - 3 81      Beyaz Ş+A, karşısında V+K+F+P              -> beraberlik
```

**İki test aynı test değil.** Malzeme sorusu iki kez soruluyor, bir kez tahtada bir kez bayrak düştüğünde, ve Chess.com her seferinde farklı bir eşik kullanıyor. Tahtada matın **zorlanabilir** olup olmadığını soruyor; bayrakta matın **hiç mümkün** olup olmadığını. Şah ve iki at, farklı cevap veren tek malzeme: mat zorlayamıyor, o yüzden çıplak şaha karşı oyun orada `IM` ile bitiyor — ama mat kurabiliyor, o yüzden ona karşı bayrak düşüren kaybediyor. Chess.com'un kendi yardım sayfaları ikisini de söylüyor, iki ayrı makalede, ve tarama ikisini de doğruluyor. Şah ve iki atın süreden kazandığı beş oyun, yalın şaha karşı anında beraberlik verdiği beş oyun.

**Birden fazla fil ve kare rengi.** Eski raporlar, en yenisi Aralık 2024'ten, Chess.com'un herhangi iki fili mat malzemesi saydığını ve hangi karelerde durduklarına bakmadığını söylüyor. Bu artık doğru değil. 2026'da doğrudan test edildi: şah ve iki koyu kare fili, çıplak şaha karşı anında beraberlik ilan ediliyor, zıt renk filler ise oynamaya devam ediyor. Dosya da aynısını yapıyor — taraf başına test fillerin kare renklerini okuyor, ve tek bir renge hapsolmuş kaç fil olursa olsun mat zorlayamıyor.

**İki at, bir taşa karşı.** İki at kuralının diğer yarısını yanlış yapmak kolay. Şah ve iki at yalnızca **yalın** şaha karşı anında beraberlik; o şaha kendini kapatacak herhangi bir şey ver, oyun devam ediyor, çünkü zorunlu bir mat var. Burada bu bir daldan değil bir eşikten çıkıyor: mat zorlayamayan taraf tartılıyor — çıplak şah sıfır, tek bir hafif taş veya aynı renkte kaç fil olursa olsun bir, at çifti iki — ve iki ağırlık birlikte üçten küçük kalınca oyun beraberlikle bitiyor.

**Bloke pozisyonlar.** Bir pozisyon, taşlar fazlasıyla yerindeyken de ölü olabilir: piyonlar kenetlenir, ve iki oyuncu da mat kurmaya çalışsa bile mat imkânsız hâle gelir. 5.2.2. madde buna anında beraberlik diyor. Chess.com bunu aramıyor, bu dosya da aramıyor — ikisi de oyunu tekrar veya elli hamle sayacı kapatana kadar sürdürüyor. O dedektörü çıkarmak, bu yapıyı proje sitesindeki FIDE kardeşinden ayıran şeyin büyük kısmı.

**En passant: FEN'in söylediği ve sayacın yaptığı.** Tekrar için FIDE iki pozisyonu ancak legal bir en passant alışı ikisinde de varsa veya ikisinde de yoksa aynı sayıyor. Birçok motor en passant karesini her çift piyon adımından sonra, gerçekten alabilecek bir şey olsun olmasın yazıyor. Hiçbir şey bozuk görünmüyor, çünkü alış zaten reddediliyor — ama tekrar anahtarı farklı çıkıyor, ve tetiklenmesi gereken bir üçlü tekrar geç geliyor ya da hiç gelmiyor. Chess.com'un FEN'i böyle yazılıyor: en passant karesi taşıyan 244 taranmış pozisyonun **234'ünde o karenin yakınında hiç düşman piyonu yoktu.** En sadesi bir açılış:

```
rnbqkbnr/ppp1pppp/8/3p4/3P4/5N2/PPP1PPPP/RNBQKB1R b KQkq d3 0 2
```

Beyaz henüz d2–d4 oynadı ve `d3` kaydedilmiş, oysa Siyah'ın alacak hiçbir şeyi yok, ne c4'te ne e4'te.

Ama tekrar sayacı o alanı okumuyor. 2020, 2022, 2023 ve 2024 [Titled Tuesday](https://github.com/IgorRigolon/titled.tuesday) oyunlarından **24.610**'u tekrarla bitti. Üç kuralın üçüyle de yeniden oynatıldılar — kare koşulsuz yazılınca, yanında düşman piyonu varsa yazılınca, yalnızca alış legal ise yazılınca — ve 24.403'ü her seferinde aynı yarım hamlede bitiyor. Kalan **207**'sinde üç tekrardan biri, alacak bir şey olmayan bir çift adımın hemen ardında, ve Chess.com bu 207 oyunun hepsini ikinci ve üçüncü kuralın öngördüğü yerde bitirmiş, birinci kuralın öngördüğü yerde hiçbirini. Angry_Twin – Hikaru, 12 Mart 2024:

```
40.Şf3 h5 41.Fd4 Kb8 42.Fc3 Kg8 43.Fd4 Kb8 44.Fc3 Kg8      1/2-1/2, tekrar
```

40...h5'ten sonraki pozisyon 42...Kg8 ve 44...Kg8'de geri geliyor, ve oyun orada bitiyor. `h6` sayılsaydı 44...Kg8 yalnızca ikinci tekrar olurdu.

İkinci kuralı üçüncüden ayıran şey, kendi şahına bağlı bir piyon, ve [2025'teki bir rapor](https://www.chess.com/forum/view/help-support/chess-com-does-not-handle-threefold-repetition-correctly-if-it-involves-pinned-en-passants) Chess.com'un böyle bir piyonu alabilir saydığını gösteriyor. Yani Chess.com **pseudo-legal**: hamle yapan piyonun yanında bir düşman piyonu olması yetiyor, açmaza bakılmıyor. Bu dosya da aynısını yapıyor. Bir çift adımdan sonra iki komşu kareye bakıyor ve kareyi ancak oradaki bir düşman piyonu onu alabiliyorsa yazıyor — yalnızca geometri, `G`'nin zaten sahip olduğu saldırı testi. FIDE ve lichess yapılarının legal hamle üretecine yaptığı çağrı hâlâ burada yok, yani `M` hâlâ özyinelemiyor. Tahtalı dosyaya 40, paketli olana 31 bayta mal oluyor.

*Eylül 2026'da Chess.com'un yardım merkezi, 30.198 bitmiş oyunu ve tekrarla biten 24.610 Titled Tuesday oyunu üzerinden ölçüldü. Chess.com sonradan fikrini değiştirebilir; bu dosya değiştiremez.*

## Sonuç kodları

Bir oyunun bitebileceği on iki yol: altısı sonuçlu, altısı beraberlik.

| | | | |
| --- | --- | --- | --- |
| `W#` Beyaz mat eder | `B#` Siyah mat eder | `SM` pat | `IM` yetersiz malzeme |
| `WT` Beyaz süreden kazanır | `BT` Siyah süreden kazanır | `TM` bayrak düştü, mat imkânsız | `50` elli hamle kuralı |
| `WR` Beyaz terkle kazanır | `BR` Siyah terkle kazanır | `3R` üçlü tekrar | `DA` anlaşmayla beraberlik |

`W` ile başlayan kod Beyaz'ın kazancı, `B` ile başlayan Siyah'ın; `M` ile biten, `D` ile başlayan veya içinde rakam taşıyan her şey beraberlik. `numerical_packed.html` aynı on iki bitişi sayı olarak bildiriyor; `4`, `5` ve `15` boş duruyor — o yuvalar yetmiş beş hamle kuralına, beşli tekrara ve çıplak şaha karşı terke aitti.

Proje sitesindeki FIDE yapısının on beş kodu var ve yetersiz malzemeye `DP`, ölü pozisyon diyor, çünkü bloke olanları da tespit ediyor. Burada kod `IM`, yani dosyanın gerçekten test ettiği şeyin adı.

## İçinde neler var

- **Her taşın hareketi**, 0x88 tahtada aritmetikle çözülmüş. Yön tablosu yok, ofset dizisi yok.
- **Tam legallik.** Kendi şahını tehdit altında bırakacak bir hamle asla geçmiyor.
- **Rok**, iki kanatta da, hepsi kontrol edilerek: hakkın hâlâ ayakta olması, kalenin yolunun boş olması, şahın tehdit altında olmaması, tehdit edilen bir kareden geçmemesi, tehdit edilen bir kareye inmemesi.
- **En passant**, Chess.com'un pseudo-legal tekrar anahtarıyla: hamle yapan piyonun yanında bir düşman piyonu varsa kare sayılıyor, açmazda olsun olmasın.
- **Terfi**: vezir, kale, fil veya at — tahtada inilen sütunun dört karesi, tahtasız bir beşinci rakam.
- **İşleyen bir saat.** Tahtada on dakika artı hamle başına beş saniye, diyalogda düz on beş dakika.
- **Beraberlik teklifleri**, terk, bayrak düşmesi, ve bayrak düşmesini beraberliğe çeviren malzeme testi.
- **On iki bitişin hepsi**, birbirinden ayrılmış hâlde.

## İçinde neler yok

- İddia kanalı yok. Üçlü tekrar ve elli hamle kendiliğinden ateşleniyor, iddia edilecek bir şey kalmıyor.
- Bloke pozisyon tespiti yok, yukarıdaki gibi. Chess.com'da da yok.
- Bayrak düştüğünde yardımlı mat araması yok, terkte beraberlik yok. Chess.com'da ikisi de yok.
- Motor yok, geri alma yok, FEN girişi veya çıkışı yok, PGN yok.
- Tahtanın çevresinde koordinat yok. Her yarım hamlede dönüyor, ve durum satırı yoldan çekiliyor.

Tam FIDE hakemi için — on beş kod, ölü pozisyonlar, on bir ön yüz — [fidelite.art](https://www.fidelite.art/) adresine bak. Lichess kural kitabı için [Lichess-equivalent](https://github.com/cuneytinann/Lichess-equivalent).

## Renkler

Bir stil dosyasından alınmadı, canlı bir oyunun ekran görüntülerinden örneklendi ve sonra piksellere karşı geri kontrol edildi.

| | | |
| --- | --- | --- |
| açık kare | `#EBECD0` | |
| koyu kare | `#739552` | |
| tahtanın arkasındaki sayfa | `#302E2B` | |
| vurgu | `#FFFF33`, %50 | açıkta `#F5F681`, koyuda `#B9CA42` |
| legal hamle | siyah, %13 | karenin %22'sinde nokta, alışta %49'dan %61'e halka |
| şahı tehdit altında | `#E004`, karenin üstüne %27 kırmızı | açıkta `#ECAD99`, koyuda `#946D3C` |

Bunların ikisi, yerini aldıkları şeyden kısa. Chess.com seçtiğin kareyi ve son hamlenin iki karesini **aynı** sarıyla boyuyor, oysa lichess iki farklı yeşil kullanıyor — iki ternary yerine bir tane. Hamle noktası da düz bir siyah tonlama, lichess onu kendi alfasıyla yeşile boyuyor. Halka ise pahalıya geliyor: lichess karenin köşelerini dolduruyor, Chess.com gerçek bir halka çiziyor, bu da iki renk durağı daha istiyor.

Alfa `#FFFF3380` yerine dört haneli `#FF38` olarak yazıldı. Bu alfayı 0,5 yerine 0,533 yapıyor ve her kanal ölçülen değerin beş birim içinde kalıyor — hiçbir gözün bulamayacağı bir fark için dört bayt.

## Tarayıcıda okuyun

`index.html` paketli değil ve küçültülmüş değil. **Sağ tık → Sayfa Kaynağını Görüntüle** (`Ctrl` `U`, macOS'ta `⌥` `⌘` `U`) bütün programı ekrana getiriyor, ve bu deponun varlık sebebi tam olarak bu: sayfayı aç, birkaç hamle oyna, sonra git az önce onları yöneten kaynağı oku.

`numerical_packed.html` istisna. Kendini açıyor, ve aşağıda nasıl açılacağına dair bir bölüm var.

---

## Anatomi

`index.html`'in her baytı, parça parça.

| parça | bayt | |
| --- | --- | --- |
| markup ve CSS | 594 | tahta, panel, denetimler, renkler, yerleşim |
| `<script>` etiketleri | 17 | |
| durum | 122 | 0x88 tahta, rok hakları, en passant karesi, sayaçlar, seçim |
| `C` | 29 | bir karenin düşürdüğü rok hakkı |
| `G` | 237 | bu taş o kareye ulaşabilir mi |
| `T` | 53 | bu kare tehdit altında mı — aksi söylenmedikçe şahın karesi |
| `U` | 135 | hamleyi bir kopyada yap: terfi, en passant ile alınan piyon, kale sıçraması |
| `L` | 55 | bu hamle yasal mı — yap, sor, geri al |
| `M` | 110 | hamleyi gerçekten oyna: sayaç, rok hakları, sıra, komşu bir piyon alabiliyorsa en passant karesi, konum anahtarı |
| `V` | 97 | bir tarafın malzemesi, Chess.com'un saydığı gibi |
| tekrar tablosu ve saat | 42 | |
| `j` | 73 | saat ve bayrak düşmesinin malzeme testi |
| takma adlar ve kurulum | 109 | `innerHTML`, taş glifleri, açılışta üretilen 64 hücre |
| `F` | 613 | tahtayı ve üstündeki terfi seçiciyi çiz |
| `E`, `K` | 350 | hamleyi oyna ve hükmü ver, düğmeler, tıklama |
| ilk çizim ve interval | 31 | |
| **toplam** | **2.667** | |

Diğer yönden dilimlenince (proje sitesinin kaynağı boyadığı gibi, her hamleden sonraki hüküm, bayrak düşmesi testi, terk ve anlaşma hükümleri de kurala sayılarak) kurallar **1.050** bayta, onları gösteren sayfa **1.617** bayta çıkıyor. Hakem ucuz, sahne pahalı — paketli dosyanın öne sürdüğü argüman da tam olarak bu.

Adlar FideLite'ın `L3`'ündeki adlar, işlevlerin çoğu da öyle: `C`, `G`, `T`, `U` ve `L`, FideLite'ın `engine.js`'indekilerin bayt bayt aynısı. `M` tek bir yerde ayrılıyor — FideLite'ın `L`'ye sorduğu "komşu piyon yasal olarak alabilir mi" sorusunu burada `G`'ye "alabilecek konumda mı" diye soruyor. Tablonun kimliği `T` değil `u`, çünkü `T` artık şah denetimi. Lichess yapısının iki yardımcısı burada yok: terk her zaman yenilgi olduğu ve bayrak hükmü `j`'nin içinde tek satıra sığdığı için `s`; talepler gidince `J`.

## Doğrulama

İki yapı da okunarak değil çalıştırılarak kontrol edildi, ve kural katmanı Chess.com'un kendisine karşı sınandı.

- **Gerçek oyunlara karşı.** Chess.com'un Published-Data API'sinden Ekim 2025 ile Eylül 2026 arasını kapsayan 30.198 bitmiş oyun çekildi, ve her biri son pozisyonundaki malzemeye ve Chess.com'un kaydettiği sonuca göre sınıflandırıldı. Yaklaşık 2.400 anında yetersiz malzeme hükmü ve bin kadar bayrak hükmü, ve dosyanın kararı her sınıfta tutuyor. Chess.com'un anında bitirdiği malzeme kombinasyonları tam olarak bu dosyanın beraberlik dediği kombinasyonlar, ve sürdürdükleri — fil ve at, zıt renk filler, üç at, iki fil ve bir at — tam olarak bu dosyanın yeterli dediği kombinasyonlar.
- **Doğrudan Chess.com'a karşı.** Aynı renk fil durumu o 30.198 oyunun hiçbirinde geçmiyor, o yüzden sitede elle kurulup oynatıldı: iki koyu kare fili çıplak şaha karşı anında beraberlik, zıt renk filler değil, her tarafta birer fil renkler ne olursa olsun beraberlik.
- **En passant kuralı**, iki yarıda: FEN tarafı en passant karesi taşıyan 244 taranmış pozisyondan, tekrar tarafı tekrarla biten 24.610 Titled Tuesday oyunundan okundu; bunların 207'si kuralları birbirinden ayırıyor.
- **En passant karesi bir referansa karşı**, 1.500 rastgele oyun ve 115.317 hamlede: her hamleden sonra bu dosyanın yazdığı kare python-chess'in pseudo-legal testiyle, legal hamle listeleri de python-chess'in kendi listeleriyle karşılaştırıldı. İkisinde de tek bir fark yok. Kareyi koşulsuz yazan önceki yapı, karede 9.251 kez farklıydı.
- **Chess.com'un kendi bitişlerine karşı.** 2020 ve 2024'ten kuralları ayıran 89 oyun, iki yapının kendi tekrar sayaçlarıyla yeniden oynatıldı. İkisi de artık 89'unun hepsini Chess.com'un bitirdiği yarım hamlede bitiriyor; önceki yapılar hiçbirini orada bitirmiyordu. Kuralların ayrışmadığı 1.500 oyunda eski ve yeni yapı 1.500'ünde de tutuyor.
- **Malzeme testi**, tahta katmanında yirmi üç durum; iki eşiği de ve onları ayıran her kombinasyonu kapsıyor, hiçbir gerçek oyunun üretmediği olanlar dahil.
- **İki yapıda da senaryolu oyunlar**, Node'da sahte bir `prompt()` karşısında: mat, otomatik üçlü tekrar, terk, hamleyle taşınıp kabul edilen bir teklif, ve bir en passant alışı. Bunlar önceki yapıda koşuldu; o zamandan beri değişen tek şey en passant karesinin ne zaman yazıldığı, ve onu yukarıdaki iki kontrol kapsıyor.
- **Önceki turlar.** 0–63 kare numaralarına geçiş ile Eylül ve Ekim başı 2026 kural optimizasyonları, her biri kendinden önceki dosyaya karşı, iki yapının o sırada kullandığı 8×8 motor üzerinde sınandı. Aşağıdaki yeniden yazım o motorun yerini aldı.
- **Ekim 2026 0x88 yeniden yazımı.** İki yapı da kendi 8×8 motorunu bırakıp FideLite'ın 0x88 motoruna geçti, `L3`'ün adlarıyla. `V`, eski `I`'ya karşı 600.000 rastgele pozisyonda koşturuldu, 300.000'i seyrek, 300.000'i kalabalık; iki taraf için de her seferinde aynı sayıyı verdi — yani yetersiz malzeme toplamı da bayrak düşmesi eşiği de aynı çıkıyor. `index.html` Chrome'da önceki sürümüyle yan yana koşturuldu, iki sayfaya da ekranın aynı karelerinden tıklandı; her adımdan sonra 64 hücre, durum satırı, iki saat, teklif ışığı, radyo düğmesi ve tahta, sıra, rok hakları, en passant karesi, sayaçlar, tekrar sayısı ve teklifler karşılaştırıldı: başlangıç dizilişinden ve on bir özel pozisyondan 6 oyunda 1.082 karşılaştırma, bayt bayt aynı 25 ekran görüntüsü. Senaryolu bitişler — `W#`, `B#`, `SM`, `TM`, `BT`, `BR`, `IM`, otomatik `3R`, tam 100. yarım hamlede `50`, rok, en passant ve çift adımın yanında açmazdaki bir piyon (alış yasadışı olduğu hâlde en passant karesi yazılıyor) — ikisinde de aynı çıktı. `numerical_packed.html` önceki dosyayla kilit adımlı koşturuldu — eskisine 0–63, yenisine aynı kareler yatay ve sütun rakamlarıyla verildi, araya yasadışı, tahta dışı ve anlamsız girdiler karıştırıldı — 140 oyunda 3.873 yasal hamle boyunca her iletişim kutusu başlığı ve sonuç kodu karşılaştırıldı: fark yok. Chrome aptal matını iki iletişim kutusunda da aynı koda götürüyor. Boyutlar 2.833'ten 2.682 bayta, 1.158'den 1.081'e indi.
- **Ekim 2026 motor güncellemesi.** İki yapı da FideLite'ın yeni `engine.js`'inin kural optimizasyonlarını aldı; böylece `index.html`'deki `C`, `G`, `T`, `U` ve `L` yine onun işlevlerinin bayt bayt aynısı: piyon dalı `&&(…)` ve `&!b[f]` yerine `?…:0` ve `>b[f]` ile kapanıyor, rok `T()+T(i+s)<S()` diye soruyor, kale sıçraması `U`'nun dönüş değerinde duruyor (`d-8?d-1024?8:Q:U(r,Q)`), başlangıç konumu `.map()` yerine `for(j in s=…)` ile yerleşiyor. Malzeme sayımı `(W?!m&W<3:m<3)` yerine `(W?W<3>m:m<3)`, ağırlıkları da `p%8==4||p%8-2&&9` yerine `p%2?9:p/4&1` yazıyor: ağırlıklar aynı, çünkü piyon, vezir ve kale tek kodlar, at da dörtte biri tek olan tek çift kod. Hiçbir kural değişmedi. `numerical_packed.html` piyon dalını, rok testini ve ağırlıkları aldı; gerisinin neden dışarıda kaldığını Paketleme bölümü anlatıyor. Boyutlar 2.682 bayttan 2.667'ye, 1.081'den 1.077'ye indi. Her yapı bir öncekiyle adım adım koşturuldu. `index.html` Node'da, ikisinde de aynı karelere tıklanarak; her tıklamadan sonra tahta, sıra, rok hakları, en passant karesi, sayaçlar, tekrar sayısı, teklifler, seçim, durum satırı ve her hücrenin taşı ve stili karşılaştırıldı: 24 oyun, 4.832 karşılaştırma, seçiciyle 14 terfi, `TM`, `BT` ve `WT` ile biten bayrak düşüşleri, `IM` ve `SM` bitişleri. `numerical_packed.html` taklit bir `prompt()`'a karşı, arada anlamsız girdiler, teklifler ve terklerle: 12 oyun, 861 girdi, her seferinde aynı sonuç ve aynı son durum. Malzeme sayımı `V` önceki biçimine karşı 20.000 rastgele pozisyonda, iki taraf için: her seferinde aynı sayı; 0, 1, 2 ve 9'un hepsi görüldü.
- **Ekim 2026 yeniden adlandırılmış paket.** `numerical_packed.html`, *Paketleme* bölümünde anlatılan harf aramasından sonra yeniden paketlendi; artık sayımın kısa testi `d<3>n`'yi de taşıyor. Boyut 1.077 bayttan 1.050'ye indi. Yeni dosya öncekiyle kilit adımlı koşturuldu: taklit bir `prompt()` ve arada bir bayrak düşürecek kadar ileri sıçrayan taklit bir saat karşısında, araya yasadışı, tahta dışı ve anlamsız girdiler, teklifler ve terkler karıştırılarak, alışlar öne çıkarılarak ki oyunlar çıplak malzemeye kadar insin: 360 oyun, 41.545 girdi, her iletişim kutusu başlığı ve her sonuç kodu aynı — on iki kodun sekizi görüldü: bir mat, `IM`, `DA`, iki bayrak düşüşü, iki terk ve `TM`. Yeni dosyanın paketi açılınca kaynağı birebir geri geliyor; RegPack 5.0.1 de 5.0.4 de o kaynaktan dosyayı bayt bayt aynı kuruyor; `reassignVars` da 350 ayarın hiçbirinde bir şey değiştirmiyor.

perft iki yeni yapıda da beş standart pozisyonda geçiyor — başlangıç dizilişinden 4. derinlikte 197.281, Kiwipete'ten 3. derinlikte 97.862, CPW 3. pozisyondan 4. derinlikte 43.238, 4. pozisyondan 3. derinlikte 9.467, 5. pozisyondan 3. derinlikte 62.379 — ve W3C Nu Html Checker yeni `index.html` için sıfır hata, sıfır uyarı veriyor. En passant karesini daha sık yazmak hamle ekleyemez: `G` piyonu o kareye bırakır, açmazdaki piyonu yine `L` atar.

## Paketi açma

`numerical_packed.html` kendini açıyor. Betiği `eval(_)` ile biten bir RegPack açma döngüsü, yani düz kaynağı geri almak için o tek çağrıyı değiştiriyorsun:

```js
eval(_)   →   console.log(_)
```

Döngüde oyuna dokunan hiçbir şey yok, o yüzden bunu Node'da yapmak güvenli. Dökülen şey 1.175 baytlık kaynak.

**Dosyayı tarayıcıdan kopyalamak yerine indir.** Sözlük anahtarları `\x01`–`\x1f` aralığından kontrol karakterleri, ve pano — ya da satır sonlarını düzelten herhangi bir editör — onları sessizce yok eder.

## Paketleme

[RegPack 5.0.4](https://github.com/Siorki/RegPack); 5.0.1 birebir aynı dosyayı üretiyor. Şu ayarlar `numerical_packed.html`'i o kaynaktan **bayt bayt** yeniden kuruyor:

| seçenek | değer |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `1` |
| `crushCopiesFactor` | `0.5` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Kazanan aşama 2, yani düzenli ifade karakter sınıfı: `[\x01-\x1fP-U]`, 37 token. Baytlar şöyle yerleşiyor:

```
   8 B  <script>
1033 B  paketli yük
   9 B  </script>
----
1050 B
```

`reassignVars`'ı açmak hiçbir şey değiştirmiyor: aşağıdaki 350 ayarın her birinde iki durumda da 1.050. Onun yapacağı yeniden adlandırma zaten yapıldı; bir sonraki bölüm anlatıyor.

350 crusher kombinasyonu denendi — gain 0–3, length 0–2, copies 0–2 yarımlık adımlarla, tiebreaker 0 ya da 1. Sekizi 1.033'te berabere; yukarıdaki ayar bu sıradaki ilki, 350'nin en kötüsü 1.047'ye iniyor.

### Paket kaynağı paketleyici için yeniden adlandırıldı

Yukarıdaki kaynak 1.175 bayt tutuyor. Kuralları değiştirilmiş lichess paket kaynağı olarak başladı — o da okur için değil RegPack için kırpılıp yeniden adlandırılmıştı. Yasal hamle gezgini ile hüküm, her biri kullanıldığı tek yere gömülü; saatler konum anahtarlarının yanında duruyor; her ad küçük bir harfte, öyle ki kaynakta kalan büyük harfler yalnız `Date`'in `D`'si ile `indexOf`'un `O`'su ve RegPack `P`–`S`'yi sözlük tokeni olarak alabiliyor.

Üstüne Chess.com kuralları geliyor: malzeme sayımı `q` (`index.html`'de `V`), `k?!m&k<3?k:9:m<3?!!m:9` diye yazılmış — eşdeğeri `(k?!m&k<3:m<3)?k+!!m:9`'dan iki bayt küçük paketleniyor, sayaçları yeniden adlandırılınca iki bayt daha; en passant karesi `L` yerine `G` üzerinden yazılıyor; terk her zaman yenilgi; üçlü tekrar otomatik; talep yok. Düz bir takas 1.068'e paketleniyor, malzeme sayımının iki yeniden yazımı 1.064'e indiriyor. Ekim 2026 motor güncellemesi — piyon dalının `?…:0` ve `>b[f]` ile kapanması, `p()+p(i+s)<o()` rok testi ve `d+=p%2?9:p/4&1` diye yazılan ağırlıklar — kaynağı 1.176 bayta, yükü 1.060'a indirdi. Güncellemenin öbür üç maddesi önce dışarıda kaldı, çünkü o adlarla daha büyük paketleniyorlardı: başlangıç konumunu `for…in` ile yerleştirmek 3 bayt, kale sıçramasını `m`'nin dönüş değerine katlamak 1 bayt, sayımın `!n&m<3` yerine `m<3>n` diye kısalan testi 1 bayt.

Son olarak harfler bir kez daha dağıtıldı, bu sefer elle değil, aramayla — lichess README'sinde anlatılan aramanın aynısıyla. Her ad, her global ve her işlevin kendi parametreleri, alabileceği her harfte denendi: küçük harfler, `_` ve `$`; büyük harfler sözlükte kalıyor. Bir taşıma ancak dosya daha küçük paketleniyorsa ve her tanımlayıcı hâlâ öncekiyle aynı değişkene işaret ediyorsa tutuldu; bu, her taşımadan sonra kaynak yeniden ayrıştırılarak denetlendi. Tekli taşımalar ve ikili takaslar, ikisi de bir şey bulamayana dek sırayla döndü. Aranmış harflerle yük 1.034'e iniyor ve üç maddeden biri artık kazandırıyor: sayımın kısa testi, `d?d<3>n?d:9:n<3?!!n:9`, onu 1.033'e indiriyor. Öbür ikisi yine dışarıda — başlangıç konumunun `for…in` ile yerleşmesi ve kale sıçraması da eklenip aynı aramadan geçirilince yük 1.036. Sözlük artık `P`–`U`'yu, altı büyük harfi alıyor. Adlar şöyle duruyor:

| düz aktarım (`L3_numerical`'ın adları) | paket kaynağı | |
| --- | --- | --- |
| `G` | `f` | ulaşım; burada ayrıca tekrar tablosu ve saatler |
| `T` | `p` | bu kare tehdit altında mı |
| `U` | `m` | hamleyi bir kopyada yap |
| `L` | `l` | bu hamle yasal mı |
| `M` | `s` | oyna |
| `V` | `q` | malzeme sayımı |
| `C` | `j` | rok hakları |
| `R` | `a` | bu konumun kaçıncı kez geldiği |
| `n`, `z` | `z`, `n` | elli hamle sayacı, sonuç kodu |
| `K`, `Y` | `d`, `x` | yazılan hamle, hedef kare |
| `G`'de `P`, `S`; `U`'da `P`, `Q` | `n`, `o`; `d`, `n` | taş türü, kare yürüyücüsü; taş türü, aradaki kare |
| `V`'de `s`, `m`, `W` | `t`, `n`, `d` | sayılan taraf, fillerinin durduğu karelerin renkleri, malzemesi |

İki değişken, biri işini bitirmeden öbürü başlamıyorsa, aynı harfi doğrudan da paylaşabilirdi. Bu denendi ve dışarıda bırakıldı: deneme oyunları iki değişkenin hiçbir zaman aynı anda canlı olmadığını kanıtlayamaz. RegPack'in kendi `reassignVars`'ı yapacak bir şey bulamıyor.

---

## İlgili

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — tam FIDE hakemi, on beş sonuç kodu, on bir ön yüz
- [Lichess-equivalent](https://github.com/cuneytinann/Lichess-equivalent) — aynı egzersiz, bir sunucu öteye
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — tam ters yön: ölü pozisyonlar %99,97'ye taşınmış
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — aynı kurallar bir seviye aşağıda, harfler ve rakamlar yan yana
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — çıplak kurallar, paketli, 1.023 baytta

## Lisans

MIT
