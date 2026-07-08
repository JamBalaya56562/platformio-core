# PlatformIO Core オープンIssue トリアージレポート

- **対象リポジトリ**: [`platformio/platformio-core`](https://github.com/platformio/platformio-core)
- **調査日**: 2026-07-05
- **対象**: オープン中の全Issue **293件**（2015-03 〜 2026-07 に起票）
- **目的**: triage（コメントのみでcloseできるものへの投稿／実装による解決）の実行計画

> **注意**: 本リポジトリはフォーク（`JamBalaya56562/platformio-core`）です。upstream の `platformio/platformio-core` に対してコメント投稿は誰でも可能ですが、Issueのclose・ラベル付与はメンテナ権限が必要です。本レポートの「close」は *「クローズを促すコメントを投稿する」* 意味で、実際のクローズはメンテナ判断になります。掲載した英語のコメント草案はそのまま投稿できる体裁にしています。

## 1. エグゼクティブサマリー

全293件を精査した結果、大きく2つの対応ストリームに整理できます。

1. **コメントで処理できる: 約150件** — クローズ誘導コメント 112件 ＋ 回答 15件 ＋ 情報リマインド 23件。うち大半は「サポート対象が別リポジトリ（dev-platform / board パッケージ）」または「長期滞留・修正済み」で、定型コメント1本で整理できます。
2. **実装で解決できる: 64件** — バグ修正 35件、機能追加 56件。うち **trivial/small 工数のクイックウィンが約48件**あり、小さなPRで着手可能です。

残り 65件はロードマップ上の正当なオープン項目（継続監視）、14件は再現調査が必要な要調査項目です。

### 最優先の着手候補（本レポートの結論）

**A. まず投稿すべきクローズ誘導コメント（確度高）**

- 「修正済みの可能性」5件は最優先で確認コメント → 反応なければclose（[§3.1](#31-修正済みの可能性likely-fixed5件)）
- 「対象外」89件は dev-platform / board リポジトリへ誘導する定型コメント（[§3.2](#32-対象外out-of-scope89件)）

**B. まず出すべき実装PR（trivial、確度高）**

- [#4391](https://github.com/platformio/platformio-core/issues/4391) **Unescaped File Path Injected into Python String Constant in test_build_unflags** — Concrete, correct bug: test generates a Python file by %-formatting an absolute Windows path between quotes, so backslash escape sequences (\t) cause SyntaxError. Real test-suite defect on Windows with a clear fix.
- [#5053](https://github.com/platformio/platformio-core/issues/5053) **Redirecting `pio pkg list` output crashes** — Reproducible UnicodeEncodeError when redirecting `pio pkg list` on Windows: uses print()/raw writes with unicode tree chars (├──) instead of click.echo, which handles Windows unicode. Root cause pinpointed by commenter.
- [#5096](https://github.com/platformio/platformio-core/issues/5096) **Warning when extending from own env** — Confirmed bug: a section extending itself ([base] extends = base) causes ProjectConfig.walk_options to loop forever (no cycle detection). Fix PR #5303 already proposed. Real core hang.
- [#5403](https://github.com/platformio/platformio-core/issues/5403) **fix: validation relies on assert, which is stripped in optimized python mode** — Confirmed in source: platformio/app.py projects_dir_validate() uses `assert os.path.isdir(projects_dir)`, which is elided under python -O, silently accepting invalid dirs. Real, trivial correctness fix.
- [#5418](https://github.com/platformio/platformio-core/issues/5418) **PlatformIO fails in pipx/uvx environments — pip not declared as a dependency** — Confirmed: pip is NOT in get_pip_dependencies() (verified in platformio/dependencies.py), yet the tool manager shells out to `pip install` for python tool deps. In pipx/uvx isolated venvs pip is absent -> ModuleNotFoundError. Clear root cause and one-line fix.
- [#5180](https://github.com/platformio/platformio-core/issues/5180) **Spaces in Windows User name not supported in Inspect** — Confirmed real bug: in check/tools/base.py the toolchain-defines command interpolates an unquoted includes_file path into a shell=True command, so paths with spaces (e.g. C:\Users\Wim Foo) break with 'not recognized as a command'.
- [#5237](https://github.com/platformio/platformio-core/issues/5237) **Undefines in build_flags don't work if there's a space after -U** — Reproducible bug: `-U MACRO` (with space) is mishandled - SCons splits into ['-U','MACRO'] and only '-U' is captured (undef[2:] empty), breaking builds. Root cause identified; a contributor volunteered a PR.
- [#5449](https://github.com/platformio/platformio-core/issues/5449) **VSCode IntelliSense `forcedInclude` doesn't work** — Confirmed in code: c_cpp_properties.json.tpl emits forcedInclude entries via _find_forced_includes/_find_abs_path, but relative project paths (e.g. include/forcedInclude.hpp) that aren't in the include-path list stay relative, which cpptools can't resolve. All other include paths are already absolute. Clear, small fix.

## 2. 全体統計

### カテゴリ別

| カテゴリ | 件数 | 説明 |
|---|---:|---|
| `out-of-scope` | 89 | 対象外（別リポジトリへ誘導） |
| `implement-feature` | 56 | 機能実装候補 |
| `keep-open` | 50 | 継続監視（オープン維持） |
| `implement-bug` | 35 | バグ修正候補 |
| `needs-info` | 20 | 情報不足（要リマインド） |
| `stale-close` | 19 | 長期滞留（closeで整理） |
| `question-answerable` | 19 | 質問（回答してclose可） |
| `likely-fixed` | 5 | 修正済みの可能性（確認→close） |
| **合計** | **293** | |

### 推奨アクション別

| アクション | 件数 | 意味 |
|---|---:|---|
| `comment-close` | 112 | クローズを促すコメントを投稿 |
| `keep` | 65 | オープン維持（対応不要） |
| `implement` | 64 | コード実装で解決 |
| `comment-ping` | 23 | 情報を求めるリマインドを投稿 |
| `answer` | 15 | 質問に回答 |
| `investigate` | 14 | 再現・原因調査が必要 |

### 滞留状況（最終更新からの経過）

| 経過期間 | 件数 |
|---|---:|
| 5年以上 | 45 |
| 3〜5年 | 52 |
| 2〜3年 | 59 |
| 1〜2年 | 53 |
| 6〜12ヶ月 | 31 |
| 3〜6ヶ月 | 39 |
| 3ヶ月以内 | 14 |

> **209件（71%）が1年以上更新なし。** triageによる整理の余地が大きいことを示します。

## 3. コメントのみで整理できる候補

投稿するだけで（実装なしで）処理できるIssue群。**確実性の高い順**に並べています。各項目の「コメント草案」はそのまま upstream に投稿できる英文です。

### 3.1 修正済みの可能性（likely-fixed）5件

すでに新しいリリースで解決されている可能性が高いもの。**再確認を促し、反応がなければclose**が最も安全。最優先で着手推奨。

#### [#1131](https://github.com/platformio/platformio-core/issues/1131) Multiple Targets in ini file not executed
*ラベル: `known issue`, `build system` ｜ 最終更新: 7.1y前 ｜ 確度: medium*

Reported on 3.5.0a16 (2017). Target/`-t` handling has been reworked extensively across 4.x/5.x/6.x. Likely fixed; needs re-test on latest before any action.

<details><summary>コメント草案（英語・投稿用）</summary>

> This was reported against PlatformIO 3.5.0a16, and the build target handling has been reworked considerably in the 5.x and 6.x releases. Could you retest with the latest PlatformIO Core using `targets = ...` (or `-t` on the CLI) and confirm whether multiple targets now run as expected? If it still misbehaves on current core, please share your `platformio.ini` and I'll reopen. Closing pending feedback.

</details>

#### [#3302](https://github.com/platformio/platformio-core/issues/3302) pio remote run -t nobuild -t upload not working
*ラベル: `help wanted`, `remote` ｜ 最終更新: 5.0y前 ｜ 確度: low*

PIO 4.1 era remote-run nobuild/upload bug; comments show a separate 'sh: No such file' error confounding repro. 5yr stale, remote stack has changed substantially. Ask to re-verify on latest.

<details><summary>コメント草案（英語・投稿用）</summary>

> This report is against PlatformIO Core 4.1 and the remote stack has changed considerably since then. The thread also shows an unrelated `sh: No such file` environment error that likely confounded the original repro. Could you re-test `pio remote run -t nobuild -t upload` on the latest Core release? If it still fails, please share a fresh `-v` log. Otherwise we'll close this as stale.

</details>

#### [#3575](https://github.com/platformio/platformio-core/issues/3575) `pio remote device list` takes over 7 minutes to complete after an agent drops from network
*ラベル: `remote` ｜ 最終更新: 6.0y前 ｜ 確度: low*

Remote-agent connection-timeout bug against PIO 4.3.4 (2020). Remote client code has changed since; no repro from others. Ping to re-test on current release before closing.

<details><summary>コメント草案（英語・投稿用）</summary>

> Thanks for the report. This was filed against PlatformIO 4.3.4 and the remote agent/client stack has changed considerably since. Are you still seeing `pio remote device list` block for minutes after an agent drops offline on a current PlatformIO Core release? If we don't hear back we'll assume it's resolved and close this out.

</details>

#### [#4490](https://github.com/platformio/platformio-core/issues/4490) Only clone necessary part of git repository dependency (`--depth`)
*ラベル: `enhancement`, `package management` ｜ 最終更新: 0.5y前 ｜ 確度: medium*

vcsclient.py now uses `--depth 1` for branch/tag/head clones (export() line ~189). Only the pinned full-commit-id case still does a full clone (git limitation). Largely implemented.

<details><summary>コメント草案（英語・投稿用）</summary>

> Thanks for the request. Recent PlatformIO Core already performs shallow clones (`--depth 1`) for git dependencies pinned to a branch, tag, or default head (see platformio/package/vcsclient.py). Only dependencies pinned to a full commit SHA still require a full clone, which is a git limitation for arbitrary commits. Please update to the latest Core and verify with your tag-based example; if you still see full clones for a tag/branch, let us know. Closing as resolved.

</details>

#### [#4525](https://github.com/platformio/platformio-core/issues/4525) pio remote update --dry-run is broken
*ラベル: `remote` ｜ 最終更新: 3.4y前 ｜ 確度: medium*

`pio remote update` was deprecated in favor of `pio pkg update`; the deprecated command emitting a notice instead of running --dry-run is expected. Likely resolved by full removal of the deprecated command.

<details><summary>コメント草案（英語・投稿用）</summary>

> `pio remote update` was deprecated in favor of `pio pkg update`, which is why you saw the deprecation notice. Could you confirm the behavior on the latest PlatformIO Core (6.1.x)? `pio pkg update --dry-run` is the supported path now. If the deprecated command still misbehaves on the current release, let us know; otherwise we'll close this as superseded.

</details>

### 3.2 対象外（out-of-scope）89件

board / チップ / framework / IDE統合など、**platformio-core ではなく別リポジトリ（dev-platform、board パッケージ、platformio-home、VSCode拡張 等）が対応すべき**もの。誘導コメントを投稿してcloseを促せます。件数が多いため一覧表にまとめ、代表的なコメント草案を末尾（[付録B](#付録b-コメント草案out-of-scope-代表例)）に掲載します。

| Issue | タイトル | ラベル | 反応 | 最終更新 | 判断理由 |
|---|---|---|---:|---|---|
| [#3456](https://github.com/platformio/platformio-core/issues/3456) | RIOT-OS support | feature,framework | 192 | 1.8y | High demand (192 reactions). Framework integration like Zephyr, but RIOT devs declined to |
| [#2947](https://github.com/platformio/platformio-core/issues/2947) | Add support for rust | feature | 157 | 5.0y | 157 reactions but Rust embedded tooling is a large, separate ecosystem (cargo/probe-rs); n |
| [#704](https://github.com/platformio/platformio-core/issues/704) | Add support for all PIC microcontrollers | board,platform,feature | 54 | 0.9y | Huge demand (54 reactions) and active community work in external platform-pic8bit repo (s- |
| [#3805](https://github.com/platformio/platformio-core/issues/3805) | Please add support for RPi Pico (RP2040) | board,platform,feature,framework | 54 | 3.1y | RP2040 support now exists via community dev-platforms (platform-raspberrypi / Wiz-IO / max |
| [#3672](https://github.com/platformio/platformio-core/issues/3672) | Add support for the NRF9160 | board,platform,feature | 38 | 5.8y | nRF9160 chip/board support lives in platform-nordicnrf52 (and community platforms), not co |
| [#4028](https://github.com/platformio/platformio-core/issues/4028) | Add support for Renesas MCUs | board,platform,feature,framework | 36 | 2.0y | High-interest (36 reactions) but purely a new-MCU/dev-platform+framework request (Renesas |
| [#1035](https://github.com/platformio/platformio-core/issues/1035) | Mongoose OS support | feature,framework | 34 | 5.7y | Framework/toolchain support belongs in a dev-platform package, not core. Mongoose OS itsel |
| [#445](https://github.com/platformio/platformio-core/issues/445) | Consider adding Particle Platform | board,platform,feature | 19 | 5.2y | Dev-platform/board support lives in separate platform-* repos, not core. Dormant 5+ years |
| [#2052](https://github.com/platformio/platformio-core/issues/2052) | Cypress - FreeSoC / PSoC support | board,platform,feature | 18 | 2.6y | Board/dev-platform (Cypress PSoC) support belongs in a dedicated platform repo, not platfo |
| [#2709](https://github.com/platformio/platformio-core/issues/2709) | Sparkfun Edge Cortex M4 MCU Board Support | board,platform,feature | 16 | 5.6y | Single-board add request (Apollo3/SparkFun Edge). Board/dev-platform scope, not core. Dorm |
| [#4046](https://github.com/platformio/platformio-core/issues/4046) | Add support for BouffaloLab chips | board,platform,feature,framework | 15 | 1.8y | Request to support BL602/BL604 chips + Arduino core. This is dev-platform/framework work ( |
| [#4609](https://github.com/platformio/platformio-core/issues/4609) | Teknic Clearcore support | board,platform,feature | 14 | 2.8y | Board request (ClearCore, SAME53). Already solved in platform-atmelsam (community PR by pa |
| [#147](https://github.com/platformio/platformio-core/issues/147) | Add new platform "timsp432" and support for TI MSP432 Launch | board,platform,feature | 11 | 4.5y | Dev-platform/board support (TI MSP432/Energia) lives in a separate platform repo, not core |
| [#3978](https://github.com/platformio/platformio-core/issues/3978) | nRF5340-DK support | board,platform,feature,framework | 11 | 2.3y | nRF53 board/platform support belongs in platform-nordicnrf52 or a community platform, not |
| [#769](https://github.com/platformio/platformio-core/issues/769) | Add support for Realtek MCUs | board,platform,feature | 10 | 1.4y | Adding a new dev-platform/board (Realtek RTL87xx) belongs in a separate platform repo, not |
| [#2232](https://github.com/platformio/platformio-core/issues/2232) | New project using MCU device | feature | 9 | 3.5y | Standalone MCU (vs board) selection is largely a dev-platform/board-JSON concern; users ca |
| [#2308](https://github.com/platformio/platformio-core/issues/2308) | Support for Ai-Thinker A9G Boards | board,platform,feature | 9 | 2.5y | Pure board/chip (Ai-Thinker A9G GPRS module) request. Board and dev-platform support live |
| [#887](https://github.com/platformio/platformio-core/issues/887) | Add support for TI RTOS | platform,feature,framework | 6 | 7.1y | Framework/toolchain support belongs in a dev-platform repo (e.g. platform-titiva/ti-relate |
| [#1862](https://github.com/platformio/platformio-core/issues/1862) | Silicon Labs EFM8 Support | board,platform,feature | 6 | 3.8y | Request to support a new 8-bit MCU family (SiLabs EFM8). New chip/platform support is a de |
| [#969](https://github.com/platformio/platformio-core/issues/969) | Support parsing Arduino additional board json files? | feature | 5 | 3.5y | Auto-importing Arduino package_index.json board definitions is a board/platform packaging |
| [#3639](https://github.com/platformio/platformio-core/issues/3639) | Add support for WinnerMicro products | board,platform,feature | 5 | 5.2y | W600/W60X board support; discussion shows this is a community dev-platform effort (maxgerh |
| [#3802](https://github.com/platformio/platformio-core/issues/3802) | Please add support for CH32V103 MCU | board,platform,feature | 5 | 2.8y | WCH CH32V RISC-V support is handled by community dev-platforms (e.g. platform-ch32v), not |
| [#1326](https://github.com/platformio/platformio-core/issues/1326) | Z-Uno (Z-Wave Arduino prototyping board) | board,platform,feature | 4 | 1.7y | Board/platform support request. New dev-platforms and boards live in separate platform rep |
| [#1449](https://github.com/platformio/platformio-core/issues/1449) | New Board: Onion Omega2 | board,platform,feature | 4 | 5.4y | Board request (Onion Omega2, a Linux SBC/MIPS). Board support belongs in a platform repo, |
| [#2453](https://github.com/platformio/platformio-core/issues/2453) | Support for Nuvoton products | board,platform,feature | 4 | 2.7y | Nuvoton MCU/board support belongs in a dev-platform repo, not core. Just links to product |
| [#3935](https://github.com/platformio/platformio-core/issues/3935) | Industrial Shields Boards support | board,feature | 4 | 3.3y | Third-party Arduino board package request. Board support belongs in dev-platform packages, |
| [#160](https://github.com/platformio/platformio-core/issues/160) | Support popular RTOS projects: ChibiOS/RT & NuttX | board,platform,feature | 3 | 5.3y | RTOS/framework integration belongs in separate dev-platform repos, not core. Dormant 5+ ye |
| [#3291](https://github.com/platformio/platformio-core/issues/3291) | Support for Logic Green LGT8F328P based boards | board,platform,feature | 3 | 6.2y | Board/platform request (LGT8F328P). Belongs in a dev-platform repo, not core. There is alr |
| [#3418](https://github.com/platformio/platformio-core/issues/3418) | Support for On Semiconductor RSL10 boards | board,platform,feature,framework | 3 | 6.3y | New chip/board + framework support belongs in a dev-platform repo, not core. No traction i |
| [#4863](https://github.com/platformio/platformio-core/issues/4863) | Add Sparkfun Thing Plus Matter - MGM240P support | board,platform,feature,framework | 3 | 2.1y | Silicon Labs MGM240P board + Matter Arduino core support. Board/framework support belongs |
| [#1884](https://github.com/platformio/platformio-core/issues/1884) | Self-hosting the PIO account server | feature,account | 2 | 3.2y | The account/registry backend is a hosted PlatformIO service, not part of platformio-core. |
| [#3661](https://github.com/platformio/platformio-core/issues/3661) | Add support for Digilent BASYS 3 board | board,platform,feature | 2 | 5.8y | BASYS 3 is a Xilinx Artix-7 FPGA board; FPGA toolchains and this board are out of Platform |
| [#3710](https://github.com/platformio/platformio-core/issues/3710) | CLion: Unexpected compiler output. This compiler might be un | integration,ide:clion | 2 | 1.4y | CLion IDE integration/IntelliSense complaint ('compiler might be unsupported'). This is CL |
| [#3732](https://github.com/platformio/platformio-core/issues/3732) | Support for Texas Instruments MCUs | board,platform,feature | 2 | 0.6y | Umbrella request for TI MCU families (Cortex-R5F, MSPM0, C6000). New MCU/dev-platform supp |
| [#3773](https://github.com/platformio/platformio-core/issues/3773) | Plugin for Geany code editor | feature,integration | 2 | 1.3y | IDE plugins live in separate repos, not platformio-core. Core already emits compile_comman |
| [#3963](https://github.com/platformio/platformio-core/issues/3963) | Linux incorrect project folder path generation | bug,home | 2 | 0.7y | Root cause is platformio-home frontend IS_WINDOWS detection (client OS, not server) in rem |
| [#4375](https://github.com/platformio/platformio-core/issues/4375) | Add support for Spresense | platform,feature,framework | 2 | 2.3y | Sony Spresense board/framework support. Belongs in a dev-platform package, not core. No co |
| [#4800](https://github.com/platformio/platformio-core/issues/4800) | Support for MPLAB PICkit 5 In-Circuit Debugger | feature,debugging | 2 | 0.4y | PICkit 5 debugger/tool support is tied to a PIC dev-platform (see #704 community effort) a |
| [#4854](https://github.com/platformio/platformio-core/issues/4854) | Feature request for support for Milk-V Boards | board,platform,feature | 2 | 2.2y | Board/platform support request (Milk-V), reporter even checked the 'Development Platform o |
| [#5367](https://github.com/platformio/platformio-core/issues/5367) | arduino UNO Q | board,feature | 2 | 0.4y | 'Add this board to vs code' request. Board support lives in the relevant dev-platform pack |
| [#5458](https://github.com/platformio/platformio-core/issues/5458) | PIO platform Longan Nano GD32 bricked |  | 2 | 0.1y | Not 'bricked hardware' — install fails because the sipeed platform-gd32v pulls toolchain-g |
| [#714](https://github.com/platformio/platformio-core/issues/714) | Simblee board support | board,platform,feature | 1 | 9.7y | Board/chip support lives in dev-platform repos, not core. Simblee (Nordic nRF51-based) is |
| [#2951](https://github.com/platformio/platformio-core/issues/2951) | Support for RISC-V VegaBoard | board,platform,feature | 1 | 6.9y | Single-line board request (even filed by maintainer as a placeholder). Board/platform supp |
| [#3289](https://github.com/platformio/platformio-core/issues/3289) | Support for the Tomu board | board,platform,feature | 1 | 6.6y | Single-board request (Tomu, EFM32). Belongs in a dev-platform (Silicon Labs EFM32) repo, n |
| [#3560](https://github.com/platformio/platformio-core/issues/3560) | Add Support for LinkIt boards | board,platform,feature | 1 | 4.0y | Board support belongs in a dev-platform repo, not core. Duplicate/related to #992. Dormant |
| [#3722](https://github.com/platformio/platformio-core/issues/3722) | Support for AVR DA Product Family | board,platform,feature,framework | 1 | 5.6y | DxCore/AVR-DA support belongs in the atmelavr dev-platform, not core. No traction in ~5.5 |
| [#3862](https://github.com/platformio/platformio-core/issues/3862) | Project Configuration does not preserve comments in `platfor | feature,known issue,home,config | 1 | 2.3y | Maintainer (ivankravets) stated it's a known limitation of Python ConfigParser and the GUI |
| [#4792](https://github.com/platformio/platformio-core/issues/4792) | Add support for VEGA Aries boards | board,platform,feature | 1 | 2.3y | Reporter checked the 'Development Platform or Board' box themselves. Board support belongs |
| [#439](https://github.com/platformio/platformio-core/issues/439) | Support UDOO boards | board,platform,feature | 0 | 7.1y | Board/platform support lives in dev-platform repos, not core. 7 years dormant, no maintain |
| [#662](https://github.com/platformio/platformio-core/issues/662) | Visual Studio error parsing support and IntelliSense (easy w | enhancement,integration | 0 | 8.0y | About Visual Studio IDE error-parsing/IntelliSense integration; belongs in the IDE integra |
| [#768](https://github.com/platformio/platformio-core/issues/768) | UltraEdit (UEStudio) as Platformio IDE | integration | 0 | 7.1y | IDE integration request for a third-party editor; not core. Zero traction, dormant 7 years |
| [#927](https://github.com/platformio/platformio-core/issues/927) | Support Chibitronics Love-to-Code | board,feature | 0 | 7.1y | Custom board/platform to be upstreamed as a dev-platform package, not core. 7 years dorman |
| [#992](https://github.com/platformio/platformio-core/issues/992) | Add LinkIt RTOS | board,platform,feature | 0 | 6.7y | MediaTek LinkIt SDK/board support belongs in a dev-platform repo, not core. Dormant 6+ yea |
| [#1002](https://github.com/platformio/platformio-core/issues/1002) | Support for Mojo FPGA dev board | board,platform,feature | 0 | 3.0y | FPGA board support is out of scope for core; Mojo v3 is a discontinued product. Belongs in |
| [#1057](https://github.com/platformio/platformio-core/issues/1057) | NavSpark board support | board,platform,feature | 0 | 8.8y | Single-board request (NavSpark), belongs in a dev-platform repo, not core. 8+ years dorman |
| [#1790](https://github.com/platformio/platformio-core/issues/1790) | Include I/O voltage in board information | board,feature | 0 | 4.2y | Requests adding I/O voltage field to board metadata + the online board explorer. Board JSO |
| [#2077](https://github.com/platformio/platformio-core/issues/2077) | К1986ВЕ92VE9x (Milandr) support and settings | board,platform,feature | 0 | 2.7y | Milandr MCU support is a dev-platform/board matter, not core. Empty body ('Subj.'), no cor |
| [#2877](https://github.com/platformio/platformio-core/issues/2877) | ARM mbed: Compilation of project with long file paths not po | bug,known issue,build system | 0 | 2.9y | Root cause is Windows CreateProcess 32KB arg limit hit by mbed framework's excessive inclu |
| [#3367](https://github.com/platformio/platformio-core/issues/3367) | Build error when parameter is overridden in mbed_app.json (f | known issue,build system | 0 | 5.7y | mbed framework builder logic; mbed-os support was deprecated/frozen in PlatformIO. Reprodu |
| [#3461](https://github.com/platformio/platformio-core/issues/3461) | Support for WGM160P product family | platform,feature | 0 | 6.2y | SiliconLabs EFM32/WGM160 board support is a dev-platform (platform-siliconlabsefm32) matte |
| [#3474](https://github.com/platformio/platformio-core/issues/3474) | Support ppc64le architecture thru microwatt/chiselwatt or Re | platform,feature | 0 | 6.2y | Reporter even ticked the 'Development Platform or Board' box. A new architecture/toolchain |
| [#3580](https://github.com/platformio/platformio-core/issues/3580) | Failed Remote Unit Test / Uploading using Travis : Could not | unit testing,remote | 0 | 2.6y | Root cause is in platform-espressif8266 builder/main.py LD-script logic (contributors conf |
| [#3625](https://github.com/platformio/platformio-core/issues/3625) | Add support for Inkplate6 Board | board,feature | 0 | 2.7y | Board request (Inkplate6, an ESP32 board). ESP32 board definitions belong in platform-espr |
| [#3683](https://github.com/platformio/platformio-core/issues/3683) | Support for Analog Devices products | board,platform,feature | 0 | 5.8y | Vague board/vendor request with no specifics, 0 comments/reactions. Board support belongs |
| [#3687](https://github.com/platformio/platformio-core/issues/3687) | add SWM320VET7 mcu | board,platform,feature | 0 | 5.7y | Single MCU/board add request with a local zip. Board/MCU support belongs in a dev-platform |
| [#3927](https://github.com/platformio/platformio-core/issues/3927) | Add Support for GigaDevice GD32F and GD32E chips | board,feature,framework | 0 | 0.5y | GD32 ARM chip support; thread shows this is being built as community dev-platforms (maxger |
| [#4002](https://github.com/platformio/platformio-core/issues/4002) | Request: Support for NXP products | board,platform,feature,framework | 0 | 1.6y | NXP Freedom/Kinetis board support belongs in a dev-platform, not core. Roadmap question; n |
| [#4127](https://github.com/platformio/platformio-core/issues/4127) | Aurix TC27x family support | board,platform,feature | 0 | 1.2y | TriCore AURIX chip/board support. New architecture support belongs in a dedicated dev-plat |
| [#4442](https://github.com/platformio/platformio-core/issues/4442) | Add support for RealDigital FPGA boards | board,platform,feature | 0 | 3.7y | FPGA board support is out of scope for core and belongs in a dev-platform. No traction. |
| [#4619](https://github.com/platformio/platformio-core/issues/4619) | add board swm181 | board,platform,feature | 0 | 3.2y | Single-board request (Synwit SWM181). The issue body itself quotes the notice that board i |
| [#4638](https://github.com/platformio/platformio-core/issues/4638) | Support for Custom Toolchain | feature | 0 | 3.1y | Request to integrate a proprietary/paid MDK-ARM (armcc) toolchain. Toolchains are packaged |
| [#4703](https://github.com/platformio/platformio-core/issues/4703) | TriGorilla board with HC32F460 chip | board,platform,feature,framework | 0 | 2.9y | Single board/chip (HC32F460) request. Board/platform support belongs in a dev-platform pac |
| [#4771](https://github.com/platformio/platformio-core/issues/4771) | Add support for Air001 | board,platform,feature,framework | 0 | 2.6y | Board/platform request (Air001 / Air-duino Arduino core). Belongs in a dev-platform repo, |
| [#4833](https://github.com/platformio/platformio-core/issues/4833) | Bad CPU type in executable clang-tidy on Apple M3 Pro | enhancement,registry,static analysis | 0 | 2.4y | Root cause is a missing native arm64 binary in the tool-clangtidy registry package (packag |
| [#4920](https://github.com/platformio/platformio-core/issues/4920) | Telit boards | board,platform,feature | 0 | 2.0y | Board/module request belongs in a dev-platform repo, not core. Zero traction, no detail. |
| [#5129](https://github.com/platformio/platformio-core/issues/5129) | python curses module fails to load on MacOS Ventura |  | 0 | 1.2y | The bundled portable Python's _curses.so was built for macOS 14.3 and won't load on older |
| [#5208](https://github.com/platformio/platformio-core/issues/5208) | New Board BMduino(BM53A367A) Holtek | board,platform,feature | 0 | 1.0y | Board/MCU support (Holtek HT32/BMduino) belongs in a dev-platform repo (Holtek even ships |
| [#5257](https://github.com/platformio/platformio-core/issues/5257) | OpenOCD fails to read peripheral memory via sysbus/progbuf/a | help wanted,debugging | 0 | 0.8y | Custom in-development RISC-V SoC; OpenOCD reports 'progbuf=disabled, sysbus=skipped, abstr |
| [#5258](https://github.com/platformio/platformio-core/issues/5258) | Can it support FMD series microcontrollers? | board,platform,feature,framework | 0 | 0.6y | New MCU family (Fremont Micro) support belongs in a dev-platform, not core. One-line reque |
| [#5288](https://github.com/platformio/platformio-core/issues/5288) | SVD parser rejects valid 64-bit registers — request clarific |  | 0 | 0.3y | The 'invalid size: 64' SVD check is NOT in platformio-core (confirmed by grep). It lives i |
| [#5299](https://github.com/platformio/platformio-core/issues/5299) | RAK3172 | board,platform,feature,framework | 0 | 0.6y | Two-word board request (RAK3172, STM32WL-based). Board/platform support belongs in the pla |
| [#5313](https://github.com/platformio/platformio-core/issues/5313) | Allow the project-folder location dialog to honor the defaul |  | 0 | 0.6y | This is about the VSCode extension / PlatformIO Home 'create project' folder-picker UX (th |
| [#5352](https://github.com/platformio/platformio-core/issues/5352) | Add TKM32F499 Board |  | 0 | 0.5y | Single board request (STM32F429-derivative) belongs in the ststm32 dev-platform / a custom |
| [#5356](https://github.com/platformio/platformio-core/issues/5356) | Add linux_aarch64 support to platformio/toolchain-gccarmnone |  | 0 | 0.4y | Request to add a linux_aarch64 build of the toolchain-gccarmnoneeabi package. Toolchain pa |
| [#5357](https://github.com/platformio/platformio-core/issues/5357) | Please add support for WCH MCUs | board,platform,feature | 0 | 0.5y | WCH CH5xx MCU support is a dev-platform matter, not core. Community platforms exist (e.g. |
| [#5398](https://github.com/platformio/platformio-core/issues/5398) | Support Milk-V Duo and Duo 256M boards | board,platform,feature,framework | 0 | 0.3y | Board/platform/framework support request (Milk-V Duo, Sophgo SG200x Arduino). New boards/p |
| [#5420](https://github.com/platformio/platformio-core/issues/5420) | Support for uno q |  | 0 | 0.3y | New board request (Arduino UNO Q). Board support is delivered via dev-platform packages / |
| [#5454](https://github.com/platformio/platformio-core/issues/5454) | Proposal: URML (substrate-neutral robot intent) capability-m |  | 0 | 0.1y | Unsolicited 'proposal-only' outreach for a third-party spec (URML) with no concrete change |
| [#5460](https://github.com/platformio/platformio-core/issues/5460) | Add board file for Waveshare ESP32-S3-Touch-AMOLED-1.64 |  | 0 | 0.0y | A board JSON for an ESP32-S3 Waveshare board. Board definitions belong in the espressif32 |

### 3.3 長期滞留（stale-close）19件

低シグナル・長期無反応で、現行版では価値が薄いもの。丁寧なクローズコメントで整理できます。

| Issue | タイトル | 経過 | 判断理由 |
|---|---|---|---|
| [#337](https://github.com/platformio/platformio-core/issues/337) | Improving Linux distribution support | 10.4y | 10-year-old design proposal for system-wide toolchain packages. No maintainer traction; PlatformIO's |
| [#230](https://github.com/platformio/platformio-core/issues/230) | Ensuring that there is no 'main()' function in user's code w | 8.1y | 10-year-old low-value enhancement, no traction, no reactions. Detecting a user main() is niche; the |
| [#448](https://github.com/platformio/platformio-core/issues/448) | Platform IO Bundled Distribution for Windows | 8.0y | Superseded: PlatformIO now ships a self-contained installer (get-platformio) and the VSCode IDE bund |
| [#443](https://github.com/platformio/platformio-core/issues/443) | init --ide eclipse deletes linkedResources | 8.0y | 8yr dormant, no repro/traction, against very old PIO 2.6.3. Eclipse IDE generator has changed substa |
| [#1813](https://github.com/platformio/platformio-core/issues/1813) | Create a unified driver manager for PlatformIO ecosystem | 7.8y | Vague, very broad request (USB/debugger driver installer) with no traction in ~8 years and no mainta |
| [#988](https://github.com/platformio/platformio-core/issues/988) | Better Visual Studio support | 6.4y | 6+ years stale enhancement for the visualstudio project generator (multi-board .sln, non-destructive |
| [#3455](https://github.com/platformio/platformio-core/issues/3455) | Multi thread firmware uploading | 6.2y | Request to parallelize uploads across environments. Uploads are inherently serial/hardware-bound and |
| [#1535](https://github.com/platformio/platformio-core/issues/1535) | the package manager does not respect OS release version | 6.0y | BSD ABI-versioning enhancement referencing PIO 3.5.1 util.py which has since been rewritten (package |
| [#3591](https://github.com/platformio/platformio-core/issues/3591) | platformio remote: no synchronization | 6.0y | Feature request for locking/serialization of concurrent remote test runs on a shared device. 6 years |
| [#3737](https://github.com/platformio/platformio-core/issues/3737) | Add support for Azure RTOS | 5.6y | Framework request for Azure RTOS (ThreadX). Microsoft open-sourced it under Eclipse ThreadX and Azur |
| [#3427](https://github.com/platformio/platformio-core/issues/3427) | Implement `pio project import` CLI/API | 5.6y | Maintainer-filed aspirational enhancement (import external tools). No traction/design in 5+ years, 3 |
| [#3765](https://github.com/platformio/platformio-core/issues/3765) | Add support for Nim | 5.5y | Adding a new language/framework (Nim) is a large dev-platform/toolchain effort with no maintainer co |
| [#1750](https://github.com/platformio/platformio-core/issues/1750) | dynamically add or replace dependencies by extrascript.py | 5.0y | Niche request to mutate library.json deps from extra_scripts based on cppflags. 5yr stale, no tracti |
| [#3713](https://github.com/platformio/platformio-core/issues/3713) | Support Kotlin Native with Zephyr | 4.9y | Niche framework request (Kotlin/Native on embedded); JetBrains largely deprioritized Kotlin/Native e |
| [#3476](https://github.com/platformio/platformio-core/issues/3476) | Platformio 4.3+ generates unusable CMake files in CLion | 4.2y | Reported against PIO 4.3 (2020). CLion integration moved to the official CLion plugin; the old CMake |
| [#364](https://github.com/platformio/platformio-core/issues/364) | Automatically update IDE settings according to project data | 3.2y | 10-year-old maintainer-filed enhancement with a documented workaround (re-run pio project init --ide |
| [#4632](https://github.com/platformio/platformio-core/issues/4632) | Improving pio ci for download progress | 3.1y | Already labeled 'wontfix'. Minor cosmetic: non-interactive progress prints per-line percentages in C |
| [#3609](https://github.com/platformio/platformio-core/issues/3609) | Add support for mynewt | 3.0y | Framework request for Apache Mynewt. Would be a large new framework integration living in a platform |
| [#4747](https://github.com/platformio/platformio-core/issues/4747) | EmBitz support | 2.8y | One-line request for an EmBitz IDE integration/project generator. No detail, no traction, niche IDE. |

### 3.4 質問（回答してclose可）19件

サポート質問。回答コメントを投稿すればcloseできます。草案は付録Bにも含めます。

| Issue | タイトル | 回答の要点 |
|---|---|---|
| [#2228](https://github.com/platformio/platformio-core/issues/2228) | Integration of Pio remote in a Python project | Support question about a stable Python API for pio remote. No public Python API is provided; PlatformIO is inv |
| [#3425](https://github.com/platformio/platformio-core/issues/3425) | Is there a way to install OFFLINE????? | Support question about offline/portable installs. Answerable: community portable-install repos and manual pack |
| [#3744](https://github.com/platformio/platformio-core/issues/3744) | Clion: PlatformIO utility is not found but installed | Support/config issue: CLion can't find pio because it's in ~/.platformio/penv, not $PATH. Well-established wor |
| [#3902](https://github.com/platformio/platformio-core/issues/3902) | Question: running newest cppcheck I get warnings all fu | Support question about cppcheck 'unusedFunction' warnings on a 5.2.0a3 pre-release. This is expected cppcheck |
| [#4164](https://github.com/platformio/platformio-core/issues/4164) | Recent News | Filed via the bug template but is a question about disabling the PIO Home 'Recent News' panel. UI belongs to P |
| [#4178](https://github.com/platformio/platformio-core/issues/4178) | When I start a new PlatformIO project I can't open it i | User confusion: `--ide visualstudio` generates MSVS project files but VS2022 opening semantics changed; Platfo |
| [#5009](https://github.com/platformio/platformio-core/issues/5009) | [FEATURE] ability to add custom arguments to cli | Requests passing custom CLI args to targets/scripts. Already achievable via SCons ARGUMENTS: `pio run -- KEY=V |
| [#5048](https://github.com/platformio/platformio-core/issues/5048) | Apollo 4 blue lite support | Really a how-to: can I use a custom-SDK/Makefile MCU in PlatformIO? Answer: yes, via a custom board + dev-plat |
| [#5074](https://github.com/platformio/platformio-core/issues/5074) | `lib_extra_dirs` deprecated but replacement `lib_deps` | Confusion over deprecation. lib_extra_dirs still works (deprecated but functional); local path deps and symlin |
| [#5097](https://github.com/platformio/platformio-core/issues/5097) | PIO device monitor adds \r before echoed \n automatical | Serial monitor display behavior; the added CR is almost certainly terminal/TTY (ONLCR) or the pyserial-miniter |
| [#5119](https://github.com/platformio/platformio-core/issues/5119) | Pre/Post Actions on buildprog not called | User attaches Pre/Post actions to 'buildprog' target, which isn't the node they think. Actions must target the |
| [#5154](https://github.com/platformio/platformio-core/issues/5154) | Sibling folders | User wants source lib and unit tests in sibling (not child) folders and resorts to symlinks. Addressable via l |
| [#5166](https://github.com/platformio/platformio-core/issues/5166) | COMPILATIONDB_INCLUDE_TOOLCHAIN=True skips WiFi | Not a core bug: with COMPILATIONDB_INCLUDE_TOOLCHAIN clangd resolves toolchain headers but library headers sco |
| [#5188](https://github.com/platformio/platformio-core/issues/5188) | Neovim + Arduino + PlatformIO | Support question about arduino-language-server/clangd LSP for .ino files under Neovim. Not a Core bug; communi |
| [#5235](https://github.com/platformio/platformio-core/issues/5235) | How to manually specify the download server | Support question: user wants to pin the package download mirror/source. Related PR discussion shows the server |
| [#5264](https://github.com/platformio/platformio-core/issues/5264) | How can I customize the default set of files generated | Support question: how to add mbed_app.json to newly created projects. Answerable - there's no template hook, b |
| [#5373](https://github.com/platformio/platformio-core/issues/5373) | Plugin for Zed | In-thread answer already works: `pio run -t compiledb` generates compile_commands.json which Zed's clangd cons |
| [#5375](https://github.com/platformio/platformio-core/issues/5375) | Library dependencies are installed but not available du | Dependencies of a lib_deps library aren't found at compile time. Typically an LDF mode / library.json dependen |
| [#5461](https://github.com/platformio/platformio-core/issues/5461) | Home: Could not load recent projects | PIO Home 'could not load recent projects' with an offline/no-internet message. This is a PIO Home (home) / VSC |

## 4. 情報不足（needs-info）— リマインド後、無反応でclose

20件。再現手順やログが不足しており、報告者に追加情報を求めるコメントを投稿。一定期間反応がなければcloseできます。

| Issue | タイトル | 不足している情報 |
|---|---|---|
| [#3959](https://github.com/platformio/platformio-core/issues/3959) | PlatformIO Core does not work on msys 2 | MSYS2 environment incompatibility from 2021, unusual/unsupported Python environment. Body truncated, no clear |
| [#4097](https://github.com/platformio/platformio-core/issues/4097) | Visual Studio IntelliSense does not recognize basic usi | Reported by a contributor (maxgerhardt) about the visualstudio IDE template missing C++ std settings. No comme |
| [#4138](https://github.com/platformio/platformio-core/issues/4138) | Hangs while scanning dependencies only in Docker contai | Slow LDF scanning inside Docker with a Windows-mounted volume; classic host-FS crossing overhead, not a confir |
| [#4536](https://github.com/platformio/platformio-core/issues/4536) | Failed to find registry package after installing it wit | Caret range on a pre-release version (@^0.7.0-rc) fails to resolve for some systems. Partly a pre-release semv |
| [#4697](https://github.com/platformio/platformio-core/issues/4697) | `pio remote run --force-remote` doesn't propagate envir | Plausible bug (env vars like PLATFORMIO_BUILD_FLAGS not sent to remote agent) but pio remote is a closed/premi |
| [#4840](https://github.com/platformio/platformio-core/issues/4840) | Remote Upload and Monitor not using environment config | Plausible remote bug: upload_port/monitor_port per-env not honored by remote actions, defaulting to first. No |
| [#4862](https://github.com/platformio/platformio-core/issues/4862) | Can't build firmware on command line if I specify other | Reports that `buildprog` target won't run when combined with other targets (pre/buildfs/post). Plausible targe |
| [#4926](https://github.com/platformio/platformio-core/issues/4926) | Dependencies are being installed regardless of platform | Possible package-manager bug: platform-restricted dependencies (mbedtls) pulled/compiled for the wrong env. Co |
| [#4932](https://github.com/platformio/platformio-core/issues/4932) | Debugging with blackmagicprobe crashes at start. | Real-looking BMP debug-init failure (STM32G0), but not reproducible outside PIO and reporter couldn't isolate |
| [#4958](https://github.com/platformio/platformio-core/issues/4958) | Remote agent fail to start on raspberry pi | Remote agent startup fails on 32-bit Raspbian with an OpenSSL-related error. Likely an environment/OpenSSL-ver |
| [#4977](https://github.com/platformio/platformio-core/issues/4977) | Offer to use an available port when the configured port | Vague suggestion (empty template body) that when a specified port is invalid PIO could offer an available one. |
| [#5146](https://github.com/platformio/platformio-core/issues/5146) | extra_script global defines not working | User can't get global CPPDEFINES from a library extraScript to reach the build command; maintainer suggested P |
| [#5201](https://github.com/platformio/platformio-core/issues/5201) | symlinks in platformio.ini should use pre-binding to pa | Reports symlink:// with '..' relative paths resolving against ~/.platformio instead of the project dir. Plausi |
| [#5245](https://github.com/platformio/platformio-core/issues/5245) | Bug: build_src_flags has an impact on lib_deps | Terse report (mainly cross-links to #4667, #4253, #4578); claims build_src_flags like -Wpedantic leak into lib |
| [#5343](https://github.com/platformio/platformio-core/issues/5343) | the disassembly doesn't work, switch to assembly is not | Disassembly/'Switch to assembly' not working when debugging native platform; a second user confirms. Ambiguous |
| [#5348](https://github.com/platformio/platformio-core/issues/5348) | Environment Switching Requires .pio Cleanup on Windows | Recent, detailed Windows-only repro (header not found until libdeps/<env> deleted after switching envs). Plaus |
| [#5389](https://github.com/platformio/platformio-core/issues/5389) | KERNAL_TASK 263%. When using Project and Configuration | Vague, alarmist report of high kernel_task CPU on macOS M1 during OTA/project reloads. No repro, no logs, like |
| [#5413](https://github.com/platformio/platformio-core/issues/5413) | Native unit test runner intermittently reports SIGSEGV | Intermittent SIGSEGV in native test runs, same commit passes/fails randomly - strongly suggests a bug in the t |
| [#5430](https://github.com/platformio/platformio-core/issues/5430) | Can't use "(" character in a sysenv variable | Report that '(' in a sysenv-injected build flag causes a shell syntax error at compile time. Likely a shell-qu |
| [#5434](https://github.com/platformio/platformio-core/issues/5434) | VScode test skipping based on test_filter | Reports that `test_filter` in platformio.ini is ignored by the VS Code test explorer (only tests_ignore works) |

## 5. 実装して解決する候補

合計 **91件**（バグ 35 ／ 機能 56）。工数（effort）と確度でソートし、着手しやすい順に示します。`impl_notes` には想定変更箇所を記載しています。

### 5.1 クイックウィン（trivial / small・確度high）

最初のPRに最適。原因箇所が特定済みで小さな変更で解決するもの。

#### [#4391](https://github.com/platformio/platformio-core/issues/4391) Unescaped File Path Injected into Python String Constant in test_build_unflags
*種別: バグ ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: help wanted*

- **内容**: Concrete, correct bug: test generates a Python file by %-formatting an absolute Windows path between quotes, so backslash escape sequences (\t) cause SyntaxError. Real test-suite defect on Windows with a clear fix.
- **実装方針**: In tests/commands/test_run.py (test_build_unflags), the generated Python script embeds an absolute file path via `%` into a double-quoted string literal. On Windows the backslashes form invalid escapes (e.g. \t). Fix: use repr() of the path, or a raw string, or os.path with forward slashes / pathlib .as_posix(), when interpolating the path into the generated script. Trivial, test-only change.

#### [#5053](https://github.com/platformio/platformio-core/issues/5053) Redirecting `pio pkg list` output crashes
*種別: バグ ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: bug,known issue*

- **内容**: Reproducible UnicodeEncodeError when redirecting `pio pkg list` on Windows: uses print()/raw writes with unicode tree chars (├──) instead of click.echo, which handles Windows unicode. Root cause pinpointed by commenter.
- **実装方針**: platformio/package/commands/list.py (~lines 102-103) emits unicode box-drawing chars via print/raw stdout, failing under non-UTF8 redirected stdout on Windows. Fix: use click.echo (which handles Windows unicode) throughout the list output, or reconfigure stdout to utf-8, or fall back to ASCII connectors ('+--','\|--') when the output encoding can't represent them. Prefer switching to click.echo for the tree lines.

#### [#5096](https://github.com/platformio/platformio-core/issues/5096) Warning when extending from own env
*種別: バグ ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: help wanted,config*

- **内容**: Confirmed bug: a section extending itself ([base] extends = base) causes ProjectConfig.walk_options to loop forever (no cycle detection). Fix PR #5303 already proposed. Real core hang.
- **実装方針**: Bug is in ProjectConfig.walk_options in platformio/project/config.py (lines ~170-183): extends_queue/extends_done have no cycle guard, so a self- or mutually-referential 'extends' loops forever. Fix: skip enqueuing a section already in extends_done or extends_queue (and/or raise a clear config error on a detected cycle instead of hanging). A community fix already exists as PR #5303 - review/merge it and add a regression test with [base] extends = base and a longer cycle chain.

#### [#5205](https://github.com/platformio/platformio-core/issues/5205) Upgrade cppcheck again!
*種別: 機能 ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: enhancement,static analysis*

- **内容**: Bundled tool-cppcheck is stale (dependencies.py pins ~1.21100.0 ~ cppcheck 2.11) vs upstream 2.17+. Recurring request (see #4571). Bump the packaged version.
- **実装方針**: platformio/dependencies.py line ~23 pins 'tool-cppcheck': '~1.21100.0'. Requires publishing a newer tool-cppcheck package (2.17.x) to the registry, then bumping this constraint. Core change is one line; the real work is repackaging the tool. Also worth considering letting users pin the cppcheck version (see #5285 discussion). Coordinate with the tool package maintainers.

#### [#5403](https://github.com/platformio/platformio-core/issues/5403) fix: validation relies on assert, which is stripped in optimized python mode
*種別: バグ ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: (なし)*

- **内容**: Confirmed in source: platformio/app.py projects_dir_validate() uses `assert os.path.isdir(projects_dir)`, which is elided under python -O, silently accepting invalid dirs. Real, trivial correctness fix.
- **実装方針**: platformio/app.py lines 32-34: replace `assert os.path.isdir(projects_dir)` with an explicit check that raises on failure, e.g. `if not os.path.isdir(projects_dir): raise ValueError(...)` (or a project-config InvalidSettingValueError). Audit app.py for other config validators using bare `assert` and give them the same treatment so validation survives `python -O`. Trivial change; add a small unit test invoking the validator with a nonexistent path.

#### [#5418](https://github.com/platformio/platformio-core/issues/5418) PlatformIO fails in pipx/uvx environments — pip not declared as a dependency
*種別: バグ ｜ 工数: `trivial` ｜ 確度: high ｜ ラベル: (なし)*

- **内容**: Confirmed: pip is NOT in get_pip_dependencies() (verified in platformio/dependencies.py), yet the tool manager shells out to `pip install` for python tool deps. In pipx/uvx isolated venvs pip is absent -> ModuleNotFoundError. Clear root cause and one-line fix.
- **実装方針**: Root cause confirmed: platformio/dependencies.py `get_pip_dependencies()` core list has no `pip`. The tool/package manager invokes `python -m pip` for python-based tool deps (grep for 'pip' in platformio/package/ and platformio/__main__ tool install path). Fix Option A (low risk): add `"pip"` to the core list in get_pip_dependencies() so it is present in isolated pipx/uvx/uv-tool venvs. Option B (larger): stop shelling out to pip. Recommend Option A for now. Related: #5305.

#### [#4181](https://github.com/platformio/platformio-core/issues/4181) Please allow disabling automatic .gitignore generation; or any arbitrary .tpl file
*種別: 機能 ｜ 工数: `small` ｜ 確度: high ｜ ラベル: enhancement,integration*

- **内容**: Popular (15 reactions, 10 comments), recently active, well-scoped: add an option to disable auto-generation of .gitignore / IDE template files. Maintainer acknowledged. Users repeatedly hit this (hg users, monorepos).
- **実装方針**: Template generation lives in platformio/project/integration/ (ProjectGenerator writes tpls from tpls/<ide>/, including .gitignore.tpl). Add a config option (e.g. [platformio] no_gitignore or a broader disable_templates list) checked before writing each .tpl. Wire it through project/options.py (add ConfigPlatformioOption) and skip the corresponding tpl in the generator. Minimal version: honor an existing .gitignore or a flag to skip only .gitignore.

#### [#5105](https://github.com/platformio/platformio-core/issues/5105) Update SCons dependency to 4.9.0
*種別: 機能 ｜ 工数: `small` ｜ 確度: high ｜ ラベル: feature,build system*

- **内容**: Maintainer-filed, concrete dependency bump. Trivial-to-small: update pinned SCons version and run the build test suite. Clear scope, no ambiguity.
- **実装方針**: Bump the SCons pin in setup.py / pyproject dependencies (and platformio/__init__.py or requirements where SCons version is constrained). Then run the integration/build tests. Watch for SCons 4.9 API deltas affecting platformio/builder/ tools. Verify against the bundled scons in the build environment.

#### [#5180](https://github.com/platformio/platformio-core/issues/5180) Spaces in Windows User name not supported in Inspect
*種別: バグ ｜ 工数: `small` ｜ 確度: high ｜ ラベル: help wanted,static analysis*

- **内容**: Confirmed real bug: in check/tools/base.py the toolchain-defines command interpolates an unquoted includes_file path into a shell=True command, so paths with spaces (e.g. C:\Users\Wim Foo) break with 'not recognized as a command'.
- **実装方針**: platformio/check/tools/base.py, `_get_toolchain_defines._extract_defines` (~line 89-101). The cmd string `'echo \| "%s" -x %s %s %s -dM -E -'` interpolates `includes_file` (the last %s) without quotes, and it's run with shell=True. When the includes file path (derived from a dir with spaces, e.g. C:\Users\Wim ...) is unquoted, the shell splits on the space. Fix: quote the includes_file argument (and audit the @file / -I paths it references) or better, avoid shell=True by passing an argv list and feeding the `echo` input via stdin. Reporter's `check_skip_packages = yes` merely sidesteps it. Add a test with a path containing spaces.

#### [#5237](https://github.com/platformio/platformio-core/issues/5237) Undefines in build_flags don't work if there's a space after -U
*種別: バグ ｜ 工数: `small` ｜ 確度: high ｜ ラベル: help wanted,build system*

- **内容**: Reproducible bug: `-U MACRO` (with space) is mishandled - SCons splits into ['-U','MACRO'] and only '-U' is captured (undef[2:] empty), breaking builds. Root cause identified; a contributor volunteered a PR.
- **実装方針**: platformio/builder/tools/piobuild.py, ProcessFlags(): the undefines extraction assumes '-UMACRO' single-token form and does undef[2:]. When space-separated, CCFLAGS contains ['-U','MACRO']. Iterate CCFLAGS with an index: when a token equals '-U', consume the next token as the macro name (mirroring how '-D'/defines are paired), otherwise strip the '-U' prefix. Add tests for both '-UM' and '-U M' forms. A contributor (SpacePlushy) offered a PR.

#### [#5240](https://github.com/platformio/platformio-core/issues/5240) Add flag to disable self-upgrades
*種別: 機能 ｜ 工数: `small` ｜ 確度: high ｜ ラベル: integration*

- **内容**: Concrete, low-risk packaging feature requested by Fedora/Debian/Arch/OpenWrt maintainers who all currently patch out `pio upgrade`. A build-time/env flag to disable the upgrade command removes recurring downstream patching.
- **実装方針**: The upgrade command is platformio/commands/upgrade.py, and update checks live in platformio (self-update logic, e.g. platformio/__init__.py version checks and maintenance/telemetry). Add a build/runtime switch - simplest is an env var (e.g. PLATFORMIO_DISABLE_UPGRADE / a setting) that makes `pio upgrade` exit with a clear 'managed by your system package manager' message and disables background update nags. Distros can set it instead of patching. Document it. Matches the Fedora short-circuit-upgrades patch behavior.

#### [#5449](https://github.com/platformio/platformio-core/issues/5449) VSCode IntelliSense `forcedInclude` doesn't work
*種別: バグ ｜ 工数: `small` ｜ 確度: high ｜ ラベル: (なし)*

- **内容**: Confirmed in code: c_cpp_properties.json.tpl emits forcedInclude entries via _find_forced_includes/_find_abs_path, but relative project paths (e.g. include/forcedInclude.hpp) that aren't in the include-path list stay relative, which cpptools can't resolve. All other include paths are already absolute. Clear, small fix.
- **実装方針**: File: platformio/project/integration/tpls/vscode/.vscode/c_cpp_properties.json.tpl. `_find_forced_includes` calls `_find_abs_path(inc, inc_paths)`, which only resolves against known include dirs; a project-relative path like `include/forcedInclude.hpp` falls through and is emitted relative. Fix: when the resolved path is still relative, join it against the project root (workspace folder) to emit an absolute path, matching how other includePaths are absolutized. Verify with build_flags `-include "include/forcedInclude.hpp"`.

### 5.2 その他のバグ修正候補（medium 工数ほか）

| Issue | タイトル | 工数 | 確度 | 想定変更箇所 / 方針 |
|---|---|---|---|---|
| [#5305](https://github.com/platformio/platformio-core/issues/5305) | Does not work correctly in pipx/uvx-style environm | `medium` | high | Core shells out to / imports pip for some internal operations (e.g. installing contrib/pysite packages or python deps). |
| [#4029](https://github.com/platformio/platformio-core/issues/4029) | "pio check --json-output" may contain extra output | `small` | medium | In platformio/check/cli.py, when --json-output is set, tool/package installation progress (Tool Manager / download bars) |
| [#4440](https://github.com/platformio/platformio-core/issues/4440) | Invalid escape character in c_cpp_properties.json | `small` | medium | The VSCode c_cpp_properties.json generation is in platformio/project/integration/ (vscode template) and the defines/flag |
| [#4499](https://github.com/platformio/platformio-core/issues/4499) | WMI failures in getting local drive list show no i | `small` | medium | Find the Windows logical-disk enumeration that shells out to `wmic logicaldisk get name,VolumeName` (grep 'logicaldisk'/ |
| [#4896](https://github.com/platformio/platformio-core/issues/4896) | Unable to remove library from VS Code | `small` | medium | Package/library removal is in platformio/package/manager/_uninstall.py and the fs helpers in platformio/fs.py (rmtree). |
| [#4901](https://github.com/platformio/platformio-core/issues/4901) | Silent failure when dependency path is incorrect i | `small` | medium | Dependency spec parsing/installation is in platformio/package/manager/* (library manager) and the spec parser platformio |
| [#4927](https://github.com/platformio/platformio-core/issues/4927) | Static code analysis fails when project path conta | `small` | medium | Static analysis command building is in platformio/check/ (tool base + cppcheck/clang-tidy/pvs-studio defect tools). A pa |
| [#4943](https://github.com/platformio/platformio-core/issues/4943) | pio check reports different errors if run with -v | `small` | medium | In platformio/check/ (clang-tidy tool + defect parsing), the verbose path likely builds/parses the tool output different |
| [#5005](https://github.com/platformio/platformio-core/issues/5005) | Debug Mode Adds Unsupported `-g2` Flag to Assemble | `small` | medium | Debug build flags are applied in the build system where debug mode augments CCFLAGS/ASFLAGS (platformio/builder/tools/*, |
| [#5050](https://github.com/platformio/platformio-core/issues/5050) | PIO remote fails on new install (Ubuntu 24.04) due | `small` | medium | cffi is a transitive dependency of the remote agent stack (paramiko/cryptography -> cffi) but isn't guaranteed in the pe |
| [#5113](https://github.com/platformio/platformio-core/issues/5113) | pio monitor fails when stdin is a pipe | `small` | medium | In platformio/device/monitor/ (command.py / terminal handling) the miniterm console tries termios/tcgetattr on stdin whi |
| [#5439](https://github.com/platformio/platformio-core/issues/5439) | Unexpected error occours when specifying a custom | `small` | medium | `pio device monitor -d` sets the project directory; the monitor command (platformio/device/monitor/command.py) then load |
| [#3915](https://github.com/platformio/platformio-core/issues/3915) | pio-test: global library extra_script.py is execut | `medium` | medium | Root cause in platformio/builder/tools/piolib.py: LibBuilderBase.__init__ calls self.process_extra_options() uncondition |
| [#4414](https://github.com/platformio/platformio-core/issues/4414) | LDF doesn't evaluate macro properly | `medium` | medium | LDF preprocessor evaluation lives in platformio/builder/tools/piolib.py (LibBuilderBase._process_dependencies / the C-pr |
| [#4446](https://github.com/platformio/platformio-core/issues/4446) | pio check using clangtidy fails on large project | `medium` | medium | platformio/check/tools/clangtidy.py builds a single long clang-tidy invocation with all -I/-D args, which blows past the |
| [#4903](https://github.com/platformio/platformio-core/issues/4903) | Project is built with include directories for two | `medium` | medium | LDF (platformio/builder/tools/piolib.py) collects include paths from all discovered library versions without deduplicati |
| [#4934](https://github.com/platformio/platformio-core/issues/4934) | `pio run -t compiledb` does not generate compilati | `medium` | medium | compiledb target is generated in platformio/builder/tools/piobuild.py / builder/main.py from the SCons build nodes of th |
| [#4959](https://github.com/platformio/platformio-core/issues/4959) | pio debug --interface=gdb --interpreter=mi2 genera | `medium` | medium | pio debug MI handling is in platformio/debug/ (process/gdb client, output stream muxing). PlatformIO interleaves the gdb |
| [#5040](https://github.com/platformio/platformio-core/issues/5040) | Project Inspector fails when projects are compiled | `medium` | medium | Size/memory analysis is in platformio/builder/tools/piosize.py (invoked as the 'sizedata' action from builder/main.py) a |
| [#5073](https://github.com/platformio/platformio-core/issues/5073) | Platform restrictions on dependencies of submodule | `medium` | medium | Bug is in the Library Dependency Finder / lib_compat handling for nested dependencies. See platformio/builder/tools/piol |
| [#5126](https://github.com/platformio/platformio-core/issues/5126) | pio debug --interface=gdb --interpreter=mi2 does n | `medium` | medium | The debug server/gdb bridge in platformio/debug/ (process/server.py and the MI output handling in debug/process/*) forwa |
| [#5171](https://github.com/platformio/platformio-core/issues/5171) | Library with `libCompatMode: strict` is not actual | `medium` | medium | In platformio/builder/tools/piolib.py, libCompatMode is read per-library from its manifest; it isn't inherited by depend |
| [#5261](https://github.com/platformio/platformio-core/issues/5261) | Library Name Collision: External Registry Librarie | `medium` | medium | Library resolution / LDF is in platformio/builder/tools/piolib.py and dependency install in platformio/package/manager/l |
| [#5323](https://github.com/platformio/platformio-core/issues/5323) | Bug: sin1.contabostorage.com blocked by QUAD9 (DoH | `medium` | medium | Two layers. (1) platformio/package/manager/_registry.py install loop iterates `RegistryFileMirrorIterator`; on network/S |
| [#5388](https://github.com/platformio/platformio-core/issues/5388) | Unnecessary googletest installs | `medium` | medium | The dependency reconciliation treats the test_framework (googletest) as an unused lib during a non-test `pio run` (test_ |
| [#5421](https://github.com/platformio/platformio-core/issues/5421) | `lib_ldf_mode = off` fails to resolve symlinked de | `medium` | medium | Symlink handling on Windows in the package manager and LDF: platformio/package/manager/* (symlink install) and platformi |
| [#5463](https://github.com/platformio/platformio-core/issues/5463) | Libraries not installed from registry cannot be de | `medium` | medium | Dependency matching is in platformio/package/manager/ (base/library manager) + package/meta.py PackageSpec. Non-registry |

### 5.3 機能追加候補

反応数（👍等）が多い順。需要の裏付けがある機能を優先。

| Issue | タイトル | 反応 | 工数 | 確度 | 概要 |
|---|---|---:|---|---|---|
| [#4613](https://github.com/platformio/platformio-core/issues/4613) | Pinning transitive dependencies with a lockfile | 16 | `large` | high | High-value, well-argued package-management feature (lockfile / reproducible builds) with 1 |
| [#3990](https://github.com/platformio/platformio-core/issues/3990) | Add support for subdirectories when adding git r | 15 | `medium` | medium | Well-scoped, high-demand (15 reactions) feature: specify a subfolder within a git repo as |
| [#4455](https://github.com/platformio/platformio-core/issues/4455) | Feature Request: option to stop LDF from running | 10 | `medium` | medium | Active (updated 2026-05), 10 reactions, real pain: LDF re-scan on every build costs second |
| [#4247](https://github.com/platformio/platformio-core/issues/4247) | Generate project SBOM in SPDX format | 9 | `medium` | medium | Filed by maintainer, 9 reactions, clear CLI spec (`pio project sbom --standard=spdx`). Not |
| [#4460](https://github.com/platformio/platformio-core/issues/4460) | Add support for quantifiers to `***_port` option | 9 | `medium` | medium | Maintainer-authored, well-scoped enhancement (9 reactions) to allow vid:/pid: quantifier m |
| [#5427](https://github.com/platformio/platformio-core/issues/5427) | Support distro-packaged Python environments (rea | 7 | `medium` | high | Well-scoped, high-value request from a distro maintainer (7 reactions, recent). Three conc |
| [#2185](https://github.com/platformio/platformio-core/issues/2185) | Add support for Catch2 unit testing framework | 7 | `medium` | medium | Legitimate, well-scoped: add a Catch2 test-runner backend alongside Unity/GoogleTest/docte |
| [#4477](https://github.com/platformio/platformio-core/issues/4477) | cli: `pio run --list-targets` should show `envdu | 5 | `small` | medium | Valid UX gap: --list-targets omits real targets (test, exec, envdump). Improves discoverab |
| [#4641](https://github.com/platformio/platformio-core/issues/4641) | Please support multiple test_xxxx test cases in | 4 | `medium` | medium | Reasonable test-runner ergonomics request: allow multiple test files/cases in one test dir |
| [#4649](https://github.com/platformio/platformio-core/issues/4649) | Allow to enable the OpenOCD -rtos flag from plat | 3 | `small` | medium | Scoped debug config request: expose a way to inject OpenOCD '-rtos auto' for RTOS thread a |
| [#3788](https://github.com/platformio/platformio-core/issues/3788) | Options merge through extends-like option | 3 | `medium` | medium | Well-scoped, popular config enhancement. Maintainer (ivankravets) proposed concrete `exten |
| [#4264](https://github.com/platformio/platformio-core/issues/4264) | control names of IDE project files | 2 | `small` | medium | Concrete, well-scoped: project init hardcodes basename 'platformio' for generated IDE file |
| [#4478](https://github.com/platformio/platformio-core/issues/4478) | [feature request] cli: add the "build" target | 2 | `small` | medium | Reasonable UX enhancement: `pio run` implicitly builds but there's no explicit `build` tar |
| [#4961](https://github.com/platformio/platformio-core/issues/4961) | Add support to monorepo style tags for external | 2 | `medium` | medium | Concrete package-manager enhancement: support monorepo-style tags/subpaths in lib_deps git |
| [#4451](https://github.com/platformio/platformio-core/issues/4451) | Sharing dependencies across environments | 2 | `large` | medium | Popular quality-of-life request: dedupe identical lib_deps across envs instead of per-env |
| [#4769](https://github.com/platformio/platformio-core/issues/4769) | Feature request: Make COMPILATIONDB_INCLUDE_TOOL | 1 | `small` | medium | Small, concrete: expose COMPILATIONDB_INCLUDE_TOOLCHAIN as a first-class option (e.g. comp |
| [#4781](https://github.com/platformio/platformio-core/issues/4781) | Allow configuring registry host url | 1 | `small` | medium | Legitimate, well-scoped: registry host is hardcoded; make it configurable (env/setting) to |
| [#5233](https://github.com/platformio/platformio-core/issues/5233) | Add dynamic interpolation for package paths | 0 | `medium` | high | Well-argued core feature by a contributor (maxgerhardt), 13-comment design discussion. Add |
| [#4413](https://github.com/platformio/platformio-core/issues/4413) | Feature - disable banner output | 0 | `trivial` | medium | Reasonable, small CI-quality-of-life request to suppress banner/noise. Well-scoped and poi |
| [#4512](https://github.com/platformio/platformio-core/issues/4512) | Setup github issue templates | 0 | `trivial` | medium | Repo-hygiene enhancement: add .github/ISSUE_TEMPLATE forms to replace the clunky inline te |
| [#5414](https://github.com/platformio/platformio-core/issues/5414) | Minimum Python version >=3.6 is EOL — consider r | 0 | `trivial` | medium | Valid maintenance item, though premise is slightly stale: setup.py already declares `pytho |
| [#3839](https://github.com/platformio/platformio-core/issues/3839) | Feature Request: Add ${PROJECT_NAME} to Targets | 0 | `small` | medium | Concrete, well-scoped fix: generated CLion CMakeLists uses non-unique target names (Produc |
| [#4354](https://github.com/platformio/platformio-core/issues/4354) | monitor_rts/dtr not working with ESP32 with auto | 0 | `small` | medium | Legit gap: monitor_rts/dtr are set but timing/ordering causes ESP32 auto-reset. Maintainer |
| [#4456](https://github.com/platformio/platformio-core/issues/4456) | Hook after Clean | 0 | `small` | medium | Confirmed gap: env.AddPreAction/AddPostAction on 'clean' have no effect, so extra_scripts |
| [#4488](https://github.com/platformio/platformio-core/issues/4488) | Move "settings" and "upgrade" commands to "syste | 0 | `small` | medium | Small, clear CLI reorg filed by maintainer: expose 'pio system settings' and 'pio system u |
| [#4531](https://github.com/platformio/platformio-core/issues/4531) | Provide a `${this.__name__}` variable (like `${t | 0 | `small` | medium | Well-scoped config interpolation feature: expose `${this.__name__}` resolving to the curre |
| [#4642](https://github.com/platformio/platformio-core/issues/4642) | upload_protocol option missing in CLI params | 0 | `small` | medium | Reasonable request to override upload_protocol from CLI without editing platformio.ini (e. |
| [#4645](https://github.com/platformio/platformio-core/issues/4645) | JLink IP address setting from platformio.ini | 0 | `small` | medium | Reasonable feature: configure J-Link remote server IP from platformio.ini so networked J-L |
| [#4756](https://github.com/platformio/platformio-core/issues/4756) | Allow to specify 'project-conf' for pio init | 0 | `small` | medium | Reasonable, small consistency enhancement: `pio project init` lacks the `--project-conf` o |
| [#4882](https://github.com/platformio/platformio-core/issues/4882) | Provide standard `PIO*` and `PLATFORMIO_*` envir | 0 | `small` | medium | Well-scoped, low-risk: export PIOENV (and program path) into the environment when running |
| [#4915](https://github.com/platformio/platformio-core/issues/4915) | Update clang-tidy support to v17.0.1 (or later) | 0 | `small` | medium | Straightforward maintenance: bump the bundled clang-tidy tool package and adapt any check- |
| [#5262](https://github.com/platformio/platformio-core/issues/5262) | [feature request] PlatformIO CLI get board_build | 0 | `small` | medium | Reasonable, well-scoped CLI enhancement: a machine-readable way to query resolved board/MC |
| [#5402](https://github.com/platformio/platformio-core/issues/5402) | Consider omitting other envs when using "pio tes | 0 | `small` | medium | Reasonable UX enhancement: when `-e` selects specific envs, don't emit noisy SKIPPED lines |
| [#5415](https://github.com/platformio/platformio-core/issues/5415) | Migrate from setup.py to pyproject.toml (PEP 621 | 0 | `small` | medium | Valid packaging modernization: repo currently ships only setup.py (confirmed, no pyproject |
| [#4243](https://github.com/platformio/platformio-core/issues/4243) | Add semantic versioning to platforms property in | 0 | `medium` | medium | Reasonable, well-scoped enhancement: allow version constraints like `espressif32@^4.0.0` i |
| [#4303](https://github.com/platformio/platformio-core/issues/4303) | Allow global source filter | 0 | `medium` | medium | Well-motivated build-system enhancement: a global src filter applying to all sources/libs |
| [#4373](https://github.com/platformio/platformio-core/issues/4373) | Build directory based on the build type | 0 | `medium` | medium | Maintainer-authored (ivankravets) with a concrete one-line fix to avoid full rebuilds when |
| [#4398](https://github.com/platformio/platformio-core/issues/4398) | Allow to change Include paths for cppcheck | 0 | `medium` | medium | Valid, well-scoped enhancement: let users exclude/limit include dirs from static-analysis |
| [#4505](https://github.com/platformio/platformio-core/issues/4505) | Feature request: provide some default flags if s | 0 | `medium` | medium | Maintainer agreed in-thread this is a valid FR: provide built-in, cross-platform template |
| [#4864](https://github.com/platformio/platformio-core/issues/4864) | Feature request for support resuming downloads | 0 | `medium` | medium | Legitimate reliability feature: resumable downloads (HTTP Range) for large package files ( |
| [#4883](https://github.com/platformio/platformio-core/issues/4883) | Option to delete intermediate output when runnin | 0 | `medium` | medium | Reasonable CI-oriented feature: option to purge per-test intermediate .o/output during `pi |
| [#5128](https://github.com/platformio/platformio-core/issues/5128) | Building a library without linking attempt | 0 | `medium` | medium | Well-argued gap: no way to build a pure library project (compile + archive, no link) so CI |
| [#5193](https://github.com/platformio/platformio-core/issues/5193) | ExtraScripts: Allow hooking on errors | 0 | `medium` | medium | Reasonable scripting enhancement: run extra-script hooks on build/upload failure (e.g. Add |
| [#5231](https://github.com/platformio/platformio-core/issues/5231) | `pio device monitor` doesn't reconnect proactive | 0 | `medium` | medium | Legit gap: monitor only detects disconnect after a failed write (user keypress), so it can |
| [#5285](https://github.com/platformio/platformio-core/issues/5285) | Adding support for clang-format | 0 | `medium` | medium | Reasonable enhancement: distribute clang-format (alongside clang-tidy) so projects can enf |
| [#5384](https://github.com/platformio/platformio-core/issues/5384) | Feature: Track source file for each config optio | 0 | `medium` | medium | Well-specified, in-scope ProjectConfig enhancement: track originating file per option (get |
| [#5399](https://github.com/platformio/platformio-core/issues/5399) | Support running custom targets locally when usin | 0 | `medium` | medium | Concrete gap: custom targets from AddCustomTarget() are filtered out of local execution in |
| [#4823](https://github.com/platformio/platformio-core/issues/4823) | LDF downloads always all "lib" dependencies rega | 0 | `large` | medium | Maintainer confirmed & reopened with updated scope: local 'lib' folder dependencies are in |
| [#4911](https://github.com/platformio/platformio-core/issues/4911) | Support concurrent access to shared PLATFORMIO_P | 0 | `large` | medium | Well-articulated CI need: unversioned package/platform dir names cause races when concurre |
| [#4694](https://github.com/platformio/platformio-core/issues/4694) | CLI pio debug doesn't detect source file changes | 0 | `small` | low | Valid UX gap: `pio debug` doesn't rebuild when sources change (unlike IDE), so gdb loads a |
| [#4710](https://github.com/platformio/platformio-core/issues/4710) | Add UNITY_INCLUDE_CONFIG_H and path to it in c_c | 0 | `small` | low | Narrow IntelliSense enhancement: include UNITY_INCLUDE_CONFIG_H define and unity_config.h |
| [#5310](https://github.com/platformio/platformio-core/issues/5310) | Add "break" support to "pio test" ? | 0 | `small` | low | Niche but concrete: send a serial BREAK after connecting in `pio test` to reset STM32/ST-L |

## 6. 継続監視（keep-open）

50件。ロードマップ上の正当な項目や、メンテナが追跡中の課題。今回はアクション不要ですが、将来的な実装候補として記録します。

| Issue | タイトル | 反応 | 理由 |
|---|---|---:|---|
| [#728](https://github.com/platformio/platformio-core/issues/728) | Micropython support | 319 | Most-requested issue in batch (319 reactions), still active (staleDays 88). Framework-leve |
| [#882](https://github.com/platformio/platformio-core/issues/882) | Support generation of code coverage data | 22 | Genuinely popular request (22 reactions, active 2026) for first-class coverage in `pio tes |
| [#3689](https://github.com/platformio/platformio-core/issues/3689) | Remote Development without PIO account | 12 | Legit, popular feature request (12 reactions) that maintainer explicitly reopened. Wants s |
| [#936](https://github.com/platformio/platformio-core/issues/936) | Add support for CMock unit testing framework | 11 | Reasonable enhancement (11 reactions) to the Unity-based test runner to allow CMock mockin |
| [#3789](https://github.com/platformio/platformio-core/issues/3789) | Support Attaching Debugger to a Running Target | 11 | Legitimate, well-articulated debugging feature (attach without reset/reflash) with 11 reac |
| [#2872](https://github.com/platformio/platformio-core/issues/2872) | Use XDG_CONFIG_HOME for .platformio directory | 10 | Legitimate, still-relevant enhancement (10 reactions) to follow XDG Base Directory spec on |
| [#3113](https://github.com/platformio/platformio-core/issues/3113) | Replace LDF deep with native GCC Preprocessor | 10 | Maintainer-filed (ivankravets) core build-system improvement with active user pain (LDF ov |
| [#3681](https://github.com/platformio/platformio-core/issues/3681) | PlatformIO Home does not work over SSH tunnel | 10 | Real, well-known bug affecting VSCode Remote-SSH/WSL users (path separator + host detectio |
| [#2144](https://github.com/platformio/platformio-core/issues/2144) | Pass build flags to the specific scope | 8 | Well-scoped, legitimate build-system enhancement filed by maintainer (ivankravets): per-la |
| [#2367](https://github.com/platformio/platformio-core/issues/2367) | Import project from Keil µVision IDE | 8 | Filed by maintainer (ivankravets), 8 reactions, steady demand (many 'any progress?' commen |
| [#4126](https://github.com/platformio/platformio-core/issues/4126) | Override platformio dir options in env sections | 6 | Reasonable config enhancement: allow [platformio]-level dir options (e.g. data_dir) to be |
| [#4184](https://github.com/platformio/platformio-core/issues/4184) | GitHub dependency graph for lib_deps | 4 | Legitimate integration ask: make lib_deps compatible with GitHub's dependency graph/Depend |
| [#3704](https://github.com/platformio/platformio-core/issues/3704) | Docker image for PlatformIO remote | 3 | Reasonable request for an official Docker image running PlatformIO Remote Agent 24/7 with |
| [#3774](https://github.com/platformio/platformio-core/issues/3774) | Add support for CppUTest unit testing framework | 3 | Legitimate feature request to add another test framework alongside Unity/GoogleTest. PIO a |
| [#4059](https://github.com/platformio/platformio-core/issues/4059) | PlatformIO is architecturally wrong about RAM | 3 | Legitimate architectural limitation raised by a knowledgeable contributor (maxgerhardt): s |
| [#4160](https://github.com/platformio/platformio-core/issues/4160) | Implement license validation for project dependencies | 3 | Maintainer-filed feature (SPDX license conflict analysis of dependency packages). In-scope |
| [#4832](https://github.com/platformio/platformio-core/issues/4832) | Add support for installing Conan C/C++ libraries | 3 | Integration idea: consume Conan packages via conanfile. In-scope-ish for package managemen |
| [#4069](https://github.com/platformio/platformio-core/issues/4069) | Support viewing SWO data | 2 | Legitimate debug-feature request (SWO/ITM trace viewing) authored by a contributor with co |
| [#446](https://github.com/platformio/platformio-core/issues/446) | Templates engine & Examples | 1 | Legitimate, recurring feature request (project templates/scaffolding) with continued commu |
| [#1401](https://github.com/platformio/platformio-core/issues/1401) | Kdevelop integration Feature Request | 1 | Legitimate IDE-integration request still getting occasional interest (last comment 2025). |
| [#4438](https://github.com/platformio/platformio-core/issues/4438) | Upstream udev rules | 1 | Actionable, still-active (staleDays 166) request to upstream PlatformIO udev rules into sy |
| [#4454](https://github.com/platformio/platformio-core/issues/4454) | Precompiled libraries include path doesn't include FPU | 1 | Specific, valid bug/enhancement in precompiled-library resolution: the include path for pr |
| [#4574](https://github.com/platformio/platformio-core/issues/4574) | Remove environment name from `build_cache_dir` cache ke | 1 | High-engagement (41 comments) known issue: build cache keyed per-env prevents sharing acro |
| [#4752](https://github.com/platformio/platformio-core/issues/4752) | Support workflow to build remotely and upload locally | 1 | Reasonable remote-workflow feature: build on a beefy remote host, upload the produced bina |
| [#4838](https://github.com/platformio/platformio-core/issues/4838) | LDF can't work with latest Adafruit TinyUSB on any sett | 1 | Confirmed by maintainer (valeros reproduced): LDF fails on Adafruit TinyUSB >=2.2.2 due to |
| [#4935](https://github.com/platformio/platformio-core/issues/4935) | Integration with `nvim-dap` | 1 | Reasonable integration request for neovim DAP debugging. Legitimate enhancement, low activ |
| [#5217](https://github.com/platformio/platformio-core/issues/5217) | [feature] DFP (Device Family Pack) / atpack / CMSIS int | 1 | Broad, exploratory idea to integrate CMSIS-Pack/atpack device metadata (registers, SVD, me |
| [#2413](https://github.com/platformio/platformio-core/issues/2413) | Board manifest to v2.0 | 0 | Maintainer-authored (ivankravets) internal design/tracking issue for a board-manifest v2 s |
| [#3270](https://github.com/platformio/platformio-core/issues/3270) | PIO Remote builds tests and then executes them all, res | 0 | Real known bug in remote test orchestration (builds all tests, only last is uploaded). Doc |
| [#3360](https://github.com/platformio/platformio-core/issues/3360) | Extend package manifest validation for platform and fra | 0 | Internal maintainer-filed enhancement pointing at schema.py to validate platform/framework |
| [#3711](https://github.com/platformio/platformio-core/issues/3711) | pio access list should output the system for toolchain | 0 | Legitimate, well-scoped registry CLI enhancement (show package 'system' attribute in `pio |
| [#3745](https://github.com/platformio/platformio-core/issues/3745) | Windows: Set "intelliSenseMode" property in c_cpp_prope | 0 | Real IntelliSense-mode regression discussion; maintainers engaged, root cause partly on Mi |
| [#3951](https://github.com/platformio/platformio-core/issues/3951) | cppcheck code inspect fails on samd boards | 0 | Reproducible cppcheck preprocessor failure on SAMD (sam.h macro expansion). Still active ( |
| [#4146](https://github.com/platformio/platformio-core/issues/4146) | Switch to Registry API 3.0 | 0 | Internal maintainer-authored tracking item (ivankravets) for migrating off /v2/ registry e |
| [#4356](https://github.com/platformio/platformio-core/issues/4356) | Document Scripting Build API | 0 | Maintainer-filed docs task (ivankravets) to document the advanced-scripting/build API. Leg |
| [#4459](https://github.com/platformio/platformio-core/issues/4459) | pio remote support code coverage reporting | 0 | Request to add --junit-output-path (and coverage) to `pio remote test`, parity with local |
| [#4576](https://github.com/platformio/platformio-core/issues/4576) | escaping $ in build_flags | 0 | Labeled 'known issue'. Escaping literal $ in build_flags is hard due to SCons variable sub |
| [#4578](https://github.com/platformio/platformio-core/issues/4578) | Build flags -include ... does not update dependency tre | 0 | Confirmed real bug, labeled 'known issue', clean minimal repro by a maintainer-adjacent co |
| [#4621](https://github.com/platformio/platformio-core/issues/4621) | Semihosting stdin spammed with internal strings - scanf | 0 | Genuine debugging bug: GDB/MI interpreter-exec commands leak into semihosting stdin, break |
| [#4666](https://github.com/platformio/platformio-core/issues/4666) | `lib_compat_mode` should impact `lib_deps` | 0 | Reasonable enhancement: skip downloading libs incompatible with lib_compat_mode. Design nu |
| [#4690](https://github.com/platformio/platformio-core/issues/4690) | Drop deprecated package management commands | 0 | Maintainer-authored (ivankravets) internal cleanup task to remove deprecated `pio lib`/`pi |
| [#4738](https://github.com/platformio/platformio-core/issues/4738) | env.AddBuildMiddleware() breaks library includes | 0 | Real reproducible bug reported by maxgerhardt (top community expert) with minimal repro: w |
| [#4814](https://github.com/platformio/platformio-core/issues/4814) | LDF default-includes wrong libs for "Arduino IoT Cloud" | 0 | Real LDF bug reported by a contributor (maxgerhardt): LDF pulls in wrong libs (WiFiNINA et |
| [#4818](https://github.com/platformio/platformio-core/issues/4818) | LDF doesn't resolve dependencies in (+) modes if an int | 0 | Maintainer-filed (valeros), labeled bug, with clear repro: CCONDITIONAL_SCANNER misses mac |
| [#4963](https://github.com/platformio/platformio-core/issues/4963) | Add PIO remote agent as a valid `PORT` type for upload, | 0 | Reasonable enhancement to let upload_port/monitor_port/test_port target a remote PIO agent |
| [#4990](https://github.com/platformio/platformio-core/issues/4990) | Support specifying the *exact* library folder for a giv | 0 | Concrete enhancement to specify exact libdeps folder per-env (to share downloaded deps acr |
| [#5018](https://github.com/platformio/platformio-core/issues/5018) | FYI - Build speed optimization - ccache massively speed | 0 | Very active (65 comments) discussion on native ccache/build-cache support. Workarounds doc |
| [#5308](https://github.com/platformio/platformio-core/issues/5308) | ESPHome: build sequence does not stop cleanly when down | 0 | Recent, active. Two threads: (a) genuine Core issue — a ChunkedEncodingError mid-download |
| [#5371](https://github.com/platformio/platformio-core/issues/5371) | Feature Request: Register a `platformio` Package URL (P | 0 | Well-researched SBOM/supply-chain proposal to register a `pkg:platformio/...` PURL type (E |
| [#5393](https://github.com/platformio/platformio-core/issues/5393) | PIO on Apple Silicon without Rosetta | 0 | Recent, real: on Apple Silicon without Rosetta, PIO selects an x86_64 gcc-arm toolchain th |

## 付録A: 全295件 トリアージ一覧

| # | タイトル | カテゴリ | アクション | 確度 | 工数 | 反応 | 最終更新 |
|---|---|---|---|---|---|---:|---|
| [#147](https://github.com/platformio/platformio-core/issues/147) | Add new platform "timsp432" and support for TI MSP432 L | `out-of-scope` | `comment-close` | high | — | 11 | 4.5y |
| [#160](https://github.com/platformio/platformio-core/issues/160) | Support popular RTOS projects: ChibiOS/RT & NuttX | `out-of-scope` | `comment-close` | high | — | 3 | 5.3y |
| [#230](https://github.com/platformio/platformio-core/issues/230) | Ensuring that there is no 'main()' function in user's c | `stale-close` | `comment-close` | medium | — | 0 | 8.1y |
| [#337](https://github.com/platformio/platformio-core/issues/337) | Improving Linux distribution support | `stale-close` | `comment-close` | high | — | 1 | 10.4y |
| [#364](https://github.com/platformio/platformio-core/issues/364) | Automatically update IDE settings according to project | `stale-close` | `comment-close` | medium | — | 2 | 3.2y |
| [#439](https://github.com/platformio/platformio-core/issues/439) | Support UDOO boards | `out-of-scope` | `comment-close` | high | — | 0 | 7.1y |
| [#443](https://github.com/platformio/platformio-core/issues/443) | init --ide eclipse deletes linkedResources | `stale-close` | `comment-close` | high | — | 0 | 8.0y |
| [#445](https://github.com/platformio/platformio-core/issues/445) | Consider adding Particle Platform | `out-of-scope` | `comment-close` | high | — | 19 | 5.2y |
| [#446](https://github.com/platformio/platformio-core/issues/446) | Templates engine & Examples | `keep-open` | `keep` | medium | — | 1 | 3.6y |
| [#448](https://github.com/platformio/platformio-core/issues/448) | Platform IO Bundled Distribution for Windows | `stale-close` | `comment-close` | high | — | 2 | 8.0y |
| [#662](https://github.com/platformio/platformio-core/issues/662) | Visual Studio error parsing support and IntelliSense (e | `out-of-scope` | `comment-close` | medium | — | 0 | 8.0y |
| [#704](https://github.com/platformio/platformio-core/issues/704) | Add support for all PIC microcontrollers | `out-of-scope` | `keep` | medium | — | 54 | 0.9y |
| [#714](https://github.com/platformio/platformio-core/issues/714) | Simblee board support | `out-of-scope` | `comment-close` | high | — | 1 | 9.7y |
| [#728](https://github.com/platformio/platformio-core/issues/728) | Micropython support | `keep-open` | `keep` | high | — | 319 | 0.2y |
| [#768](https://github.com/platformio/platformio-core/issues/768) | UltraEdit (UEStudio) as Platformio IDE | `out-of-scope` | `comment-close` | high | — | 0 | 7.1y |
| [#769](https://github.com/platformio/platformio-core/issues/769) | Add support for Realtek MCUs | `out-of-scope` | `comment-close` | high | — | 10 | 1.4y |
| [#882](https://github.com/platformio/platformio-core/issues/882) | Support generation of code coverage data | `keep-open` | `keep` | medium | — | 22 | 0.3y |
| [#887](https://github.com/platformio/platformio-core/issues/887) | Add support for TI RTOS | `out-of-scope` | `comment-close` | high | — | 6 | 7.1y |
| [#927](https://github.com/platformio/platformio-core/issues/927) | Support Chibitronics Love-to-Code | `out-of-scope` | `comment-close` | high | — | 0 | 7.1y |
| [#936](https://github.com/platformio/platformio-core/issues/936) | Add support for CMock unit testing framework | `keep-open` | `keep` | low | — | 11 | 4.2y |
| [#969](https://github.com/platformio/platformio-core/issues/969) | Support parsing Arduino additional board json files? | `out-of-scope` | `comment-close` | medium | — | 5 | 3.5y |
| [#988](https://github.com/platformio/platformio-core/issues/988) | Better Visual Studio support | `stale-close` | `comment-close` | medium | — | 3 | 6.4y |
| [#992](https://github.com/platformio/platformio-core/issues/992) | Add LinkIt RTOS | `out-of-scope` | `comment-close` | high | — | 0 | 6.7y |
| [#1002](https://github.com/platformio/platformio-core/issues/1002) | Support for Mojo FPGA dev board | `out-of-scope` | `comment-close` | high | — | 0 | 3.0y |
| [#1035](https://github.com/platformio/platformio-core/issues/1035) | Mongoose OS support | `out-of-scope` | `comment-close` | high | — | 34 | 5.7y |
| [#1057](https://github.com/platformio/platformio-core/issues/1057) | NavSpark board support | `out-of-scope` | `comment-close` | high | — | 0 | 8.8y |
| [#1131](https://github.com/platformio/platformio-core/issues/1131) | Multiple Targets in ini file not executed | `likely-fixed` | `comment-ping` | medium | — | 0 | 7.1y |
| [#1326](https://github.com/platformio/platformio-core/issues/1326) | Z-Uno (Z-Wave Arduino prototyping board) | `out-of-scope` | `comment-close` | high | — | 4 | 1.7y |
| [#1401](https://github.com/platformio/platformio-core/issues/1401) | Kdevelop integration Feature Request | `keep-open` | `keep` | medium | — | 1 | 0.8y |
| [#1449](https://github.com/platformio/platformio-core/issues/1449) | New Board: Onion Omega2 | `out-of-scope` | `comment-close` | high | — | 4 | 5.4y |
| [#1535](https://github.com/platformio/platformio-core/issues/1535) | the package manager does not respect OS release version | `stale-close` | `comment-close` | medium | — | 1 | 6.0y |
| [#1750](https://github.com/platformio/platformio-core/issues/1750) | dynamically add or replace dependencies by extrascript. | `stale-close` | `comment-close` | medium | — | 0 | 5.0y |
| [#1790](https://github.com/platformio/platformio-core/issues/1790) | Include I/O voltage in board information | `out-of-scope` | `comment-close` | medium | — | 0 | 4.2y |
| [#1813](https://github.com/platformio/platformio-core/issues/1813) | Create a unified driver manager for PlatformIO ecosyste | `stale-close` | `comment-close` | high | — | 0 | 7.8y |
| [#1862](https://github.com/platformio/platformio-core/issues/1862) | Silicon Labs EFM8 Support | `out-of-scope` | `comment-close` | high | — | 6 | 3.8y |
| [#1884](https://github.com/platformio/platformio-core/issues/1884) | Self-hosting the PIO account server | `out-of-scope` | `comment-close` | high | — | 2 | 3.2y |
| [#2052](https://github.com/platformio/platformio-core/issues/2052) | Cypress - FreeSoC / PSoC support | `out-of-scope` | `comment-close` | high | — | 18 | 2.6y |
| [#2077](https://github.com/platformio/platformio-core/issues/2077) | К1986ВЕ92VE9x (Milandr) support and settings | `out-of-scope` | `comment-close` | high | — | 0 | 2.7y |
| [#2144](https://github.com/platformio/platformio-core/issues/2144) | Pass build flags to the specific scope | `keep-open` | `keep` | medium | — | 8 | 2.7y |
| [#2185](https://github.com/platformio/platformio-core/issues/2185) | Add support for Catch2 unit testing framework | `implement-feature` | `keep` | medium | `medium` | 7 | 4.2y |
| [#2228](https://github.com/platformio/platformio-core/issues/2228) | Integration of Pio remote in a Python project | `question-answerable` | `comment-close` | medium | — | 0 | 6.9y |
| [#2232](https://github.com/platformio/platformio-core/issues/2232) | New project using MCU device | `out-of-scope` | `comment-close` | medium | — | 9 | 3.5y |
| [#2308](https://github.com/platformio/platformio-core/issues/2308) | Support for Ai-Thinker A9G Boards | `out-of-scope` | `comment-close` | high | — | 9 | 2.5y |
| [#2367](https://github.com/platformio/platformio-core/issues/2367) | Import project from Keil µVision IDE | `keep-open` | `keep` | high | — | 8 | 0.6y |
| [#2413](https://github.com/platformio/platformio-core/issues/2413) | Board manifest to v2.0 | `keep-open` | `keep` | low | — | 0 | 4.2y |
| [#2453](https://github.com/platformio/platformio-core/issues/2453) | Support for Nuvoton products | `out-of-scope` | `comment-close` | high | — | 4 | 2.7y |
| [#2709](https://github.com/platformio/platformio-core/issues/2709) | Sparkfun Edge Cortex M4 MCU Board Support | `out-of-scope` | `comment-close` | high | — | 16 | 5.6y |
| [#2872](https://github.com/platformio/platformio-core/issues/2872) | Use XDG_CONFIG_HOME for .platformio directory | `keep-open` | `keep` | medium | — | 10 | 1.4y |
| [#2877](https://github.com/platformio/platformio-core/issues/2877) | ARM mbed: Compilation of project with long file paths n | `out-of-scope` | `comment-close` | medium | — | 0 | 2.9y |
| [#2947](https://github.com/platformio/platformio-core/issues/2947) | Add support for rust | `out-of-scope` | `comment-close` | medium | — | 157 | 5.0y |
| [#2951](https://github.com/platformio/platformio-core/issues/2951) | Support for RISC-V VegaBoard | `out-of-scope` | `comment-close` | high | — | 1 | 6.9y |
| [#3113](https://github.com/platformio/platformio-core/issues/3113) | Replace LDF deep with native GCC Preprocessor | `keep-open` | `keep` | high | — | 10 | 2.1y |
| [#3270](https://github.com/platformio/platformio-core/issues/3270) | PIO Remote builds tests and then executes them all, res | `keep-open` | `keep` | medium | — | 0 | 1.9y |
| [#3289](https://github.com/platformio/platformio-core/issues/3289) | Support for the Tomu board | `out-of-scope` | `comment-close` | high | — | 1 | 6.6y |
| [#3291](https://github.com/platformio/platformio-core/issues/3291) | Support for Logic Green LGT8F328P based boards | `out-of-scope` | `comment-close` | high | — | 3 | 6.2y |
| [#3302](https://github.com/platformio/platformio-core/issues/3302) | pio remote run -t nobuild -t upload not working | `likely-fixed` | `comment-ping` | low | — | 1 | 5.0y |
| [#3360](https://github.com/platformio/platformio-core/issues/3360) | Extend package manifest validation for platform and fra | `keep-open` | `keep` | low | — | 0 | 6.4y |
| [#3367](https://github.com/platformio/platformio-core/issues/3367) | Build error when parameter is overridden in mbed_app.js | `out-of-scope` | `comment-close` | medium | — | 0 | 5.7y |
| [#3418](https://github.com/platformio/platformio-core/issues/3418) | Support for On Semiconductor RSL10 boards | `out-of-scope` | `comment-close` | high | — | 3 | 6.3y |
| [#3425](https://github.com/platformio/platformio-core/issues/3425) | Is there a way to install OFFLINE????? | `question-answerable` | `answer` | medium | — | 0 | 2.7y |
| [#3427](https://github.com/platformio/platformio-core/issues/3427) | Implement `pio project import` CLI/API | `stale-close` | `comment-close` | medium | — | 3 | 5.6y |
| [#3455](https://github.com/platformio/platformio-core/issues/3455) | Multi thread firmware uploading | `stale-close` | `comment-close` | medium | — | 0 | 6.2y |
| [#3456](https://github.com/platformio/platformio-core/issues/3456) | RIOT-OS support | `out-of-scope` | `keep` | medium | — | 192 | 1.8y |
| [#3461](https://github.com/platformio/platformio-core/issues/3461) | Support for WGM160P product family | `out-of-scope` | `comment-close` | high | — | 0 | 6.2y |
| [#3474](https://github.com/platformio/platformio-core/issues/3474) | Support ppc64le architecture thru microwatt/chiselwatt | `out-of-scope` | `comment-close` | high | — | 0 | 6.2y |
| [#3476](https://github.com/platformio/platformio-core/issues/3476) | Platformio 4.3+ generates unusable CMake files in CLion | `stale-close` | `comment-close` | medium | — | 0 | 4.2y |
| [#3560](https://github.com/platformio/platformio-core/issues/3560) | Add Support for LinkIt boards | `out-of-scope` | `comment-close` | high | — | 1 | 4.0y |
| [#3575](https://github.com/platformio/platformio-core/issues/3575) | `pio remote device list` takes over 7 minutes to comple | `likely-fixed` | `comment-ping` | low | — | 0 | 6.0y |
| [#3580](https://github.com/platformio/platformio-core/issues/3580) | Failed Remote Unit Test / Uploading using Travis : Coul | `out-of-scope` | `comment-close` | medium | — | 0 | 2.6y |
| [#3591](https://github.com/platformio/platformio-core/issues/3591) | platformio remote: no synchronization | `stale-close` | `comment-close` | low | — | 0 | 6.0y |
| [#3609](https://github.com/platformio/platformio-core/issues/3609) | Add support for mynewt | `stale-close` | `comment-close` | medium | — | 8 | 3.0y |
| [#3625](https://github.com/platformio/platformio-core/issues/3625) | Add support for Inkplate6 Board | `out-of-scope` | `comment-close` | high | — | 0 | 2.7y |
| [#3639](https://github.com/platformio/platformio-core/issues/3639) | Add support for WinnerMicro products | `out-of-scope` | `comment-close` | high | — | 5 | 5.2y |
| [#3661](https://github.com/platformio/platformio-core/issues/3661) | Add support for Digilent BASYS 3 board | `out-of-scope` | `comment-close` | high | — | 2 | 5.8y |
| [#3672](https://github.com/platformio/platformio-core/issues/3672) | Add support for the NRF9160 | `out-of-scope` | `comment-close` | high | — | 38 | 5.8y |
| [#3681](https://github.com/platformio/platformio-core/issues/3681) | PlatformIO Home does not work over SSH tunnel | `keep-open` | `keep` | high | — | 10 | 0.9y |
| [#3683](https://github.com/platformio/platformio-core/issues/3683) | Support for Analog Devices products | `out-of-scope` | `comment-close` | high | — | 0 | 5.8y |
| [#3687](https://github.com/platformio/platformio-core/issues/3687) | add SWM320VET7 mcu | `out-of-scope` | `comment-close` | high | — | 0 | 5.7y |
| [#3689](https://github.com/platformio/platformio-core/issues/3689) | Remote Development without PIO account | `keep-open` | `keep` | medium | — | 12 | 1.8y |
| [#3704](https://github.com/platformio/platformio-core/issues/3704) | Docker image for PlatformIO remote | `keep-open` | `keep` | low | — | 3 | 2.7y |
| [#3710](https://github.com/platformio/platformio-core/issues/3710) | CLion: Unexpected compiler output. This compiler might | `out-of-scope` | `comment-close` | medium | — | 2 | 1.4y |
| [#3711](https://github.com/platformio/platformio-core/issues/3711) | pio access list should output the system for toolchain | `keep-open` | `keep` | medium | — | 0 | 4.7y |
| [#3713](https://github.com/platformio/platformio-core/issues/3713) | Support Kotlin Native with Zephyr | `stale-close` | `comment-close` | high | — | 0 | 4.9y |
| [#3722](https://github.com/platformio/platformio-core/issues/3722) | Support for AVR DA Product Family | `out-of-scope` | `comment-close` | high | — | 1 | 5.6y |
| [#3732](https://github.com/platformio/platformio-core/issues/3732) | Support for Texas Instruments MCUs | `out-of-scope` | `comment-close` | high | — | 2 | 0.6y |
| [#3737](https://github.com/platformio/platformio-core/issues/3737) | Add support for Azure RTOS | `stale-close` | `comment-close` | high | — | 3 | 5.6y |
| [#3744](https://github.com/platformio/platformio-core/issues/3744) | Clion: PlatformIO utility is not found but installed | `question-answerable` | `answer` | high | — | 0 | 2.6y |
| [#3745](https://github.com/platformio/platformio-core/issues/3745) | Windows: Set "intelliSenseMode" property in c_cpp_prope | `keep-open` | `keep` | medium | — | 0 | 1.9y |
| [#3765](https://github.com/platformio/platformio-core/issues/3765) | Add support for Nim | `stale-close` | `comment-close` | medium | — | 3 | 5.5y |
| [#3773](https://github.com/platformio/platformio-core/issues/3773) | Plugin for Geany code editor | `out-of-scope` | `comment-close` | medium | — | 2 | 1.3y |
| [#3774](https://github.com/platformio/platformio-core/issues/3774) | Add support for CppUTest unit testing framework | `keep-open` | `keep` | medium | — | 3 | 4.2y |
| [#3788](https://github.com/platformio/platformio-core/issues/3788) | Options merge through extends-like option | `implement-feature` | `implement` | medium | `medium` | 3 | 2.0y |
| [#3789](https://github.com/platformio/platformio-core/issues/3789) | Support Attaching Debugger to a Running Target | `keep-open` | `keep` | medium | — | 11 | 5.5y |
| [#3802](https://github.com/platformio/platformio-core/issues/3802) | Please add support for CH32V103 MCU | `out-of-scope` | `comment-close` | high | — | 5 | 2.8y |
| [#3805](https://github.com/platformio/platformio-core/issues/3805) | Please add support for RPi Pico (RP2040) | `out-of-scope` | `comment-close` | high | — | 54 | 3.1y |
| [#3839](https://github.com/platformio/platformio-core/issues/3839) | Feature Request: Add ${PROJECT_NAME} to Targets in gene | `implement-feature` | `implement` | medium | `small` | 0 | 4.2y |
| [#3862](https://github.com/platformio/platformio-core/issues/3862) | Project Configuration does not preserve comments in `pl | `out-of-scope` | `comment-close` | medium | — | 1 | 2.3y |
| [#3902](https://github.com/platformio/platformio-core/issues/3902) | Question: running newest cppcheck I get warnings all fu | `question-answerable` | `comment-close` | medium | — | 1 | 4.4y |
| [#3915](https://github.com/platformio/platformio-core/issues/3915) | pio-test: global library extra_script.py is executed wi | `implement-bug` | `implement` | medium | `medium` | 0 | 2.8y |
| [#3927](https://github.com/platformio/platformio-core/issues/3927) | Add Support for GigaDevice GD32F and GD32E chips | `out-of-scope` | `comment-close` | high | — | 0 | 0.5y |
| [#3935](https://github.com/platformio/platformio-core/issues/3935) | Industrial Shields Boards support | `out-of-scope` | `comment-close` | high | — | 4 | 3.3y |
| [#3951](https://github.com/platformio/platformio-core/issues/3951) | cppcheck code inspect fails on samd boards | `keep-open` | `keep` | medium | — | 0 | 0.8y |
| [#3959](https://github.com/platformio/platformio-core/issues/3959) | PlatformIO Core does not work on msys 2 | `needs-info` | `comment-ping` | low | — | 0 | 4.8y |
| [#3963](https://github.com/platformio/platformio-core/issues/3963) | Linux incorrect project folder path generation | `out-of-scope` | `comment-close` | high | — | 2 | 0.7y |
| [#3978](https://github.com/platformio/platformio-core/issues/3978) | nRF5340-DK support | `out-of-scope` | `comment-close` | high | — | 11 | 2.3y |
| [#3990](https://github.com/platformio/platformio-core/issues/3990) | Add support for subdirectories when adding git repos as | `implement-feature` | `implement` | medium | `medium` | 15 | 2.4y |
| [#4002](https://github.com/platformio/platformio-core/issues/4002) | Request: Support for NXP products | `out-of-scope` | `comment-close` | high | — | 0 | 1.6y |
| [#4028](https://github.com/platformio/platformio-core/issues/4028) | Add support for Renesas MCUs | `out-of-scope` | `comment-close` | high | — | 36 | 2.0y |
| [#4029](https://github.com/platformio/platformio-core/issues/4029) | "pio check --json-output" may contain extra output of p | `implement-bug` | `implement` | medium | `small` | 1 | 0.5y |
| [#4046](https://github.com/platformio/platformio-core/issues/4046) | Add support for BouffaloLab chips | `out-of-scope` | `comment-close` | high | — | 15 | 1.8y |
| [#4059](https://github.com/platformio/platformio-core/issues/4059) | PlatformIO is architecturally wrong about RAM | `keep-open` | `keep` | medium | — | 3 | 3.9y |
| [#4069](https://github.com/platformio/platformio-core/issues/4069) | Support viewing SWO data | `keep-open` | `keep` | medium | — | 2 | 2.7y |
| [#4097](https://github.com/platformio/platformio-core/issues/4097) | Visual Studio IntelliSense does not recognize basic usi | `needs-info` | `comment-ping` | medium | — | 0 | 4.7y |
| [#4126](https://github.com/platformio/platformio-core/issues/4126) | Override platformio dir options in env sections | `keep-open` | `keep` | medium | — | 6 | 4.3y |
| [#4127](https://github.com/platformio/platformio-core/issues/4127) | Aurix TC27x family support | `out-of-scope` | `comment-close` | high | — | 0 | 1.2y |
| [#4138](https://github.com/platformio/platformio-core/issues/4138) | Hangs while scanning dependencies only in Docker contai | `needs-info` | `comment-close` | medium | — | 0 | 4.2y |
| [#4146](https://github.com/platformio/platformio-core/issues/4146) | Switch to Registry API 3.0 | `keep-open` | `keep` | medium | — | 0 | 4.2y |
| [#4160](https://github.com/platformio/platformio-core/issues/4160) | Implement license validation for project dependencies | `keep-open` | `keep` | low | — | 3 | 3.5y |
| [#4164](https://github.com/platformio/platformio-core/issues/4164) | Recent News | `question-answerable` | `comment-close` | medium | — | 0 | 4.4y |
| [#4178](https://github.com/platformio/platformio-core/issues/4178) | When I start a new PlatformIO project I can't open it i | `question-answerable` | `comment-close` | medium | — | 0 | 3.7y |
| [#4181](https://github.com/platformio/platformio-core/issues/4181) | Please allow disabling automatic .gitignore generation; | `implement-feature` | `implement` | high | `small` | 15 | 0.3y |
| [#4184](https://github.com/platformio/platformio-core/issues/4184) | GitHub dependency graph for lib_deps | `keep-open` | `keep` | medium | — | 4 | 1.8y |
| [#4243](https://github.com/platformio/platformio-core/issues/4243) | Add semantic versioning to platforms property in librar | `implement-feature` | `keep` | medium | `medium` | 0 | 4.2y |
| [#4247](https://github.com/platformio/platformio-core/issues/4247) | Generate project SBOM in SPDX format | `implement-feature` | `implement` | medium | `medium` | 9 | 3.2y |
| [#4264](https://github.com/platformio/platformio-core/issues/4264) | control names of IDE project files | `implement-feature` | `implement` | medium | `small` | 2 | 4.1y |
| [#4303](https://github.com/platformio/platformio-core/issues/4303) | Allow global source filter | `implement-feature` | `implement` | medium | `medium` | 0 | 4.1y |
| [#4354](https://github.com/platformio/platformio-core/issues/4354) | monitor_rts/dtr not working with ESP32 with auto-upload | `implement-feature` | `implement` | medium | `small` | 0 | 2.3y |
| [#4356](https://github.com/platformio/platformio-core/issues/4356) | Document Scripting Build API | `keep-open` | `keep` | low | — | 0 | 4.0y |
| [#4373](https://github.com/platformio/platformio-core/issues/4373) | Build directory based on the build type | `implement-feature` | `implement` | medium | `medium` | 0 | 3.0y |
| [#4375](https://github.com/platformio/platformio-core/issues/4375) | Add support for Spresense | `out-of-scope` | `comment-close` | high | — | 2 | 2.3y |
| [#4391](https://github.com/platformio/platformio-core/issues/4391) | Unescaped File Path Injected into Python String Constan | `implement-bug` | `implement` | high | `trivial` | 0 | 3.8y |
| [#4398](https://github.com/platformio/platformio-core/issues/4398) | Allow to change Include paths for cppcheck | `implement-feature` | `implement` | medium | `medium` | 0 | 3.8y |
| [#4413](https://github.com/platformio/platformio-core/issues/4413) | Feature - disable banner output | `implement-feature` | `implement` | medium | `trivial` | 0 | 3.8y |
| [#4414](https://github.com/platformio/platformio-core/issues/4414) | LDF doesn't evaluate macro properly | `implement-bug` | `implement` | medium | `medium` | 1 | 1.1y |
| [#4438](https://github.com/platformio/platformio-core/issues/4438) | Upstream udev rules | `keep-open` | `keep` | medium | — | 1 | 0.5y |
| [#4440](https://github.com/platformio/platformio-core/issues/4440) | Invalid escape character in c_cpp_properties.json | `implement-bug` | `investigate` | medium | `small` | 0 | 1.9y |
| [#4442](https://github.com/platformio/platformio-core/issues/4442) | Add support for RealDigital FPGA boards | `out-of-scope` | `comment-close` | high | — | 0 | 3.7y |
| [#4446](https://github.com/platformio/platformio-core/issues/4446) | pio check using clangtidy fails on large project | `implement-bug` | `implement` | medium | `medium` | 2 | 1.3y |
| [#4451](https://github.com/platformio/platformio-core/issues/4451) | Sharing dependencies across environments | `implement-feature` | `keep` | medium | `large` | 2 | 1.8y |
| [#4454](https://github.com/platformio/platformio-core/issues/4454) | Precompiled libraries include path doesn't include FPU | `keep-open` | `keep` | low | — | 1 | 2.5y |
| [#4455](https://github.com/platformio/platformio-core/issues/4455) | Feature Request: option to stop LDF from running at eve | `implement-feature` | `implement` | medium | `medium` | 10 | 0.1y |
| [#4456](https://github.com/platformio/platformio-core/issues/4456) | Hook after Clean | `implement-feature` | `implement` | medium | `small` | 0 | 2.0y |
| [#4459](https://github.com/platformio/platformio-core/issues/4459) | pio remote support code coverage reporting | `keep-open` | `keep` | low | — | 0 | 3.2y |
| [#4460](https://github.com/platformio/platformio-core/issues/4460) | Add support for quantifiers to `***_port` options | `implement-feature` | `implement` | medium | `medium` | 9 | 1.9y |
| [#4477](https://github.com/platformio/platformio-core/issues/4477) | cli: `pio run --list-targets` should show `envdump`, `e | `implement-feature` | `implement` | medium | `small` | 5 | 3.5y |
| [#4478](https://github.com/platformio/platformio-core/issues/4478) | [feature request] cli: add the "build" target | `implement-feature` | `implement` | medium | `small` | 2 | 1.3y |
| [#4488](https://github.com/platformio/platformio-core/issues/4488) | Move "settings" and "upgrade" commands to "system" grou | `implement-feature` | `implement` | medium | `small` | 0 | 3.0y |
| [#4490](https://github.com/platformio/platformio-core/issues/4490) | Only clone necessary part of git repository dependency | `likely-fixed` | `comment-close` | medium | — | 2 | 0.5y |
| [#4499](https://github.com/platformio/platformio-core/issues/4499) | WMI failures in getting local drive list show no indica | `implement-bug` | `implement` | medium | `small` | 0 | 3.5y |
| [#4505](https://github.com/platformio/platformio-core/issues/4505) | Feature request: provide some default flags if setting | `implement-feature` | `keep` | medium | `medium` | 0 | 3.5y |
| [#4512](https://github.com/platformio/platformio-core/issues/4512) | Setup github issue templates | `implement-feature` | `implement` | medium | `trivial` | 0 | 3.5y |
| [#4525](https://github.com/platformio/platformio-core/issues/4525) | pio remote update --dry-run is broken | `likely-fixed` | `comment-ping` | medium | — | 0 | 3.4y |
| [#4531](https://github.com/platformio/platformio-core/issues/4531) | Provide a `${this.__name__}` variable (like `${this.__e | `implement-feature` | `implement` | medium | `small` | 0 | 2.3y |
| [#4536](https://github.com/platformio/platformio-core/issues/4536) | Failed to find registry package after installing it wit | `needs-info` | `comment-ping` | medium | — | 0 | 1.9y |
| [#4574](https://github.com/platformio/platformio-core/issues/4574) | Remove environment name from `build_cache_dir` cache ke | `keep-open` | `keep` | medium | — | 1 | 3.2y |
| [#4576](https://github.com/platformio/platformio-core/issues/4576) | escaping $ in build_flags | `keep-open` | `keep` | medium | — | 0 | 3.2y |
| [#4578](https://github.com/platformio/platformio-core/issues/4578) | Build flags -include ... does not update dependency tre | `keep-open` | `keep` | high | — | 0 | 3.2y |
| [#4609](https://github.com/platformio/platformio-core/issues/4609) | Teknic Clearcore support | `out-of-scope` | `comment-close` | high | — | 14 | 2.8y |
| [#4613](https://github.com/platformio/platformio-core/issues/4613) | Pinning transitive dependencies with a lockfile | `implement-feature` | `keep` | high | `large` | 16 | 0.3y |
| [#4619](https://github.com/platformio/platformio-core/issues/4619) | add board swm181 | `out-of-scope` | `comment-close` | high | — | 0 | 3.2y |
| [#4621](https://github.com/platformio/platformio-core/issues/4621) | Semihosting stdin spammed with internal strings - scanf | `keep-open` | `keep` | low | — | 0 | 2.9y |
| [#4632](https://github.com/platformio/platformio-core/issues/4632) | Improving pio ci for download progress | `stale-close` | `comment-close` | high | — | 0 | 3.1y |
| [#4638](https://github.com/platformio/platformio-core/issues/4638) | Support for Custom Toolchain | `out-of-scope` | `comment-close` | medium | — | 0 | 3.1y |
| [#4641](https://github.com/platformio/platformio-core/issues/4641) | Please support multiple test_xxxx test cases in a singl | `implement-feature` | `keep` | medium | `medium` | 4 | 2.4y |
| [#4642](https://github.com/platformio/platformio-core/issues/4642) | upload_protocol option missing in CLI params | `implement-feature` | `implement` | medium | `small` | 0 | 1.4y |
| [#4645](https://github.com/platformio/platformio-core/issues/4645) | JLink IP address setting from platformio.ini | `implement-feature` | `keep` | medium | `small` | 0 | 2.6y |
| [#4649](https://github.com/platformio/platformio-core/issues/4649) | Allow to enable the OpenOCD -rtos flag from platformio | `implement-feature` | `keep` | medium | `small` | 3 | 2.6y |
| [#4666](https://github.com/platformio/platformio-core/issues/4666) | `lib_compat_mode` should impact `lib_deps` | `keep-open` | `keep` | low | — | 0 | 3.0y |
| [#4690](https://github.com/platformio/platformio-core/issues/4690) | Drop deprecated package management commands | `keep-open` | `keep` | medium | — | 0 | 3.0y |
| [#4694](https://github.com/platformio/platformio-core/issues/4694) | CLI pio debug doesn't detect source file changes | `implement-feature` | `implement` | low | `small` | 0 | 2.4y |
| [#4697](https://github.com/platformio/platformio-core/issues/4697) | `pio remote run --force-remote` doesn't propagate envir | `needs-info` | `comment-ping` | medium | — | 0 | 2.9y |
| [#4703](https://github.com/platformio/platformio-core/issues/4703) | TriGorilla board with HC32F460 chip | `out-of-scope` | `comment-close` | high | — | 0 | 2.9y |
| [#4710](https://github.com/platformio/platformio-core/issues/4710) | Add UNITY_INCLUDE_CONFIG_H and path to it in c_cpp_prop | `implement-feature` | `implement` | low | `small` | 0 | 2.4y |
| [#4738](https://github.com/platformio/platformio-core/issues/4738) | env.AddBuildMiddleware() breaks library includes | `keep-open` | `keep` | high | — | 0 | 2.0y |
| [#4747](https://github.com/platformio/platformio-core/issues/4747) | EmBitz support | `stale-close` | `comment-close` | medium | — | 0 | 2.8y |
| [#4752](https://github.com/platformio/platformio-core/issues/4752) | Support workflow to build remotely and upload locally | `keep-open` | `keep` | low | — | 1 | 2.6y |
| [#4756](https://github.com/platformio/platformio-core/issues/4756) | Allow to specify 'project-conf' for pio init | `implement-feature` | `implement` | medium | `small` | 0 | 2.6y |
| [#4769](https://github.com/platformio/platformio-core/issues/4769) | Feature request: Make COMPILATIONDB_INCLUDE_TOOLCHAIN a | `implement-feature` | `keep` | medium | `small` | 1 | 2.6y |
| [#4771](https://github.com/platformio/platformio-core/issues/4771) | Add support for Air001 | `out-of-scope` | `comment-close` | high | — | 0 | 2.6y |
| [#4781](https://github.com/platformio/platformio-core/issues/4781) | Allow configuring registry host url | `implement-feature` | `implement` | medium | `small` | 1 | 2.3y |
| [#4792](https://github.com/platformio/platformio-core/issues/4792) | Add support for VEGA Aries boards | `out-of-scope` | `comment-close` | high | — | 1 | 2.3y |
| [#4800](https://github.com/platformio/platformio-core/issues/4800) | Support for MPLAB PICkit 5 In-Circuit Debugger | `out-of-scope` | `comment-close` | medium | — | 2 | 0.4y |
| [#4814](https://github.com/platformio/platformio-core/issues/4814) | LDF default-includes wrong libs for "Arduino IoT Cloud" | `keep-open` | `keep` | low | — | 0 | 2.5y |
| [#4818](https://github.com/platformio/platformio-core/issues/4818) | LDF doesn't resolve dependencies in (+) modes if an int | `keep-open` | `keep` | high | — | 0 | 2.2y |
| [#4823](https://github.com/platformio/platformio-core/issues/4823) | LDF downloads always all "lib" dependencies regardless | `implement-feature` | `implement` | medium | `large` | 0 | 2.4y |
| [#4832](https://github.com/platformio/platformio-core/issues/4832) | Add support for installing Conan C/C++ libraries | `keep-open` | `keep` | low | — | 3 | 2.5y |
| [#4833](https://github.com/platformio/platformio-core/issues/4833) | Bad CPU type in executable clang-tidy on Apple M3 Pro | `out-of-scope` | `comment-close` | medium | — | 0 | 2.4y |
| [#4838](https://github.com/platformio/platformio-core/issues/4838) | LDF can't work with latest Adafruit TinyUSB on any sett | `keep-open` | `keep` | medium | — | 1 | 1.8y |
| [#4840](https://github.com/platformio/platformio-core/issues/4840) | Remote Upload and Monitor not using environment config | `needs-info` | `comment-ping` | medium | — | 0 | 2.4y |
| [#4854](https://github.com/platformio/platformio-core/issues/4854) | Feature request for support for Milk-V Boards | `out-of-scope` | `comment-close` | high | — | 2 | 2.2y |
| [#4862](https://github.com/platformio/platformio-core/issues/4862) | Can't build firmware on command line if I specify other | `needs-info` | `comment-ping` | medium | — | 0 | 2.4y |
| [#4863](https://github.com/platformio/platformio-core/issues/4863) | Add Sparkfun Thing Plus Matter - MGM240P support | `out-of-scope` | `comment-close` | high | — | 3 | 2.1y |
| [#4864](https://github.com/platformio/platformio-core/issues/4864) | Feature request for support resuming downloads | `implement-feature` | `implement` | medium | `medium` | 0 | 2.3y |
| [#4882](https://github.com/platformio/platformio-core/issues/4882) | Provide standard `PIO*` and `PLATFORMIO_*` environment | `implement-feature` | `implement` | medium | `small` | 0 | 2.3y |
| [#4883](https://github.com/platformio/platformio-core/issues/4883) | Option to delete intermediate output when running tests | `implement-feature` | `keep` | medium | `medium` | 0 | 2.3y |
| [#4896](https://github.com/platformio/platformio-core/issues/4896) | Unable to remove library from VS Code | `implement-bug` | `investigate` | medium | `small` | 0 | 2.2y |
| [#4901](https://github.com/platformio/platformio-core/issues/4901) | Silent failure when dependency path is incorrect in lib | `implement-bug` | `implement` | medium | `small` | 0 | 2.2y |
| [#4903](https://github.com/platformio/platformio-core/issues/4903) | Project is built with include directories for two diffe | `implement-bug` | `investigate` | medium | `medium` | 1 | 2.2y |
| [#4911](https://github.com/platformio/platformio-core/issues/4911) | Support concurrent access to shared PLATFORMIO_PACKAGES | `implement-feature` | `implement` | medium | `large` | 0 | 2.2y |
| [#4915](https://github.com/platformio/platformio-core/issues/4915) | Update clang-tidy support to v17.0.1 (or later) | `implement-feature` | `implement` | medium | `small` | 0 | 2.0y |
| [#4920](https://github.com/platformio/platformio-core/issues/4920) | Telit boards | `out-of-scope` | `comment-close` | high | — | 0 | 2.0y |
| [#4926](https://github.com/platformio/platformio-core/issues/4926) | Dependencies are being installed regardless of platform | `needs-info` | `comment-ping` | low | — | 0 | 1.7y |
| [#4927](https://github.com/platformio/platformio-core/issues/4927) | Static code analysis fails when project path contains s | `implement-bug` | `implement` | medium | `small` | 0 | 1.7y |
| [#4932](https://github.com/platformio/platformio-core/issues/4932) | Debugging with blackmagicprobe crashes at start. | `needs-info` | `comment-ping` | medium | — | 0 | 1.7y |
| [#4934](https://github.com/platformio/platformio-core/issues/4934) | `pio run -t compiledb` does not generate compilation da | `implement-bug` | `implement` | medium | `medium` | 2 | 0.2y |
| [#4935](https://github.com/platformio/platformio-core/issues/4935) | Integration with `nvim-dap` | `keep-open` | `keep` | low | — | 1 | 2.0y |
| [#4943](https://github.com/platformio/platformio-core/issues/4943) | pio check reports different errors if run with -v flag | `implement-bug` | `investigate` | medium | `small` | 0 | 2.0y |
| [#4958](https://github.com/platformio/platformio-core/issues/4958) | Remote agent fail to start on raspberry pi | `needs-info` | `comment-ping` | low | — | 0 | 1.8y |
| [#4959](https://github.com/platformio/platformio-core/issues/4959) | pio debug --interface=gdb --interpreter=mi2 generates m | `implement-bug` | `investigate` | medium | `medium` | 1 | 1.2y |
| [#4961](https://github.com/platformio/platformio-core/issues/4961) | Add support to monorepo style tags for external Git res | `implement-feature` | `keep` | medium | `medium` | 2 | 1.9y |
| [#4963](https://github.com/platformio/platformio-core/issues/4963) | Add PIO remote agent as a valid `PORT` type for upload, | `keep-open` | `keep` | low | — | 0 | 1.8y |
| [#4977](https://github.com/platformio/platformio-core/issues/4977) | Offer to use an available port when the configured port | `needs-info` | `comment-ping` | low | — | 0 | 1.8y |
| [#4990](https://github.com/platformio/platformio-core/issues/4990) | Support specifying the *exact* library folder for a giv | `keep-open` | `keep` | low | — | 0 | 1.7y |
| [#5005](https://github.com/platformio/platformio-core/issues/5005) | Debug Mode Adds Unsupported `-g2` Flag to Assembler, Ca | `implement-bug` | `implement` | medium | `small` | 0 | 1.6y |
| [#5009](https://github.com/platformio/platformio-core/issues/5009) | [FEATURE] ability to add custom arguments to cli | `question-answerable` | `answer` | medium | — | 0 | 1.6y |
| [#5018](https://github.com/platformio/platformio-core/issues/5018) | FYI - Build speed optimization - ccache massively speed | `keep-open` | `keep` | medium | — | 0 | 0.9y |
| [#5040](https://github.com/platformio/platformio-core/issues/5040) | Project Inspector fails when projects are compiled with | `implement-bug` | `investigate` | medium | `medium` | 0 | 1.5y |
| [#5048](https://github.com/platformio/platformio-core/issues/5048) | Apollo 4 blue lite support | `question-answerable` | `answer` | high | — | 0 | 1.4y |
| [#5050](https://github.com/platformio/platformio-core/issues/5050) | PIO remote fails on new install (Ubuntu 24.04) due to m | `implement-bug` | `implement` | medium | `small` | 0 | 0.3y |
| [#5053](https://github.com/platformio/platformio-core/issues/5053) | Redirecting `pio pkg list` output crashes | `implement-bug` | `implement` | high | `trivial` | 0 | 1.4y |
| [#5073](https://github.com/platformio/platformio-core/issues/5073) | Platform restrictions on dependencies of submodule in p | `implement-bug` | `investigate` | medium | `medium` | 1 | 0.8y |
| [#5074](https://github.com/platformio/platformio-core/issues/5074) | `lib_extra_dirs` deprecated but replacement `lib_deps` | `question-answerable` | `answer` | medium | — | 0 | 1.4y |
| [#5096](https://github.com/platformio/platformio-core/issues/5096) | Warning when extending from own env | `implement-bug` | `implement` | high | `trivial` | 0 | 0.7y |
| [#5097](https://github.com/platformio/platformio-core/issues/5097) | PIO device monitor adds \r before echoed \n automatical | `question-answerable` | `answer` | low | — | 0 | 1.1y |
| [#5105](https://github.com/platformio/platformio-core/issues/5105) | Update SCons dependency to 4.9.0 | `implement-feature` | `implement` | high | `small` | 0 | 1.3y |
| [#5113](https://github.com/platformio/platformio-core/issues/5113) | pio monitor fails when stdin is a pipe | `implement-bug` | `investigate` | medium | `small` | 0 | 1.3y |
| [#5119](https://github.com/platformio/platformio-core/issues/5119) | Pre/Post Actions on buildprog not called | `question-answerable` | `answer` | medium | — | 0 | 0.4y |
| [#5126](https://github.com/platformio/platformio-core/issues/5126) | pio debug --interface=gdb --interpreter=mi2 does not wr | `implement-bug` | `implement` | medium | `medium` | 0 | 1.2y |
| [#5128](https://github.com/platformio/platformio-core/issues/5128) | Building a library without linking attempt | `implement-feature` | `implement` | medium | `medium` | 0 | 0.9y |
| [#5129](https://github.com/platformio/platformio-core/issues/5129) | python curses module fails to load on MacOS Ventura | `out-of-scope` | `comment-close` | medium | — | 0 | 1.2y |
| [#5146](https://github.com/platformio/platformio-core/issues/5146) | extra_script global defines not working | `needs-info` | `comment-ping` | medium | — | 0 | 0.4y |
| [#5154](https://github.com/platformio/platformio-core/issues/5154) | Sibling folders | `question-answerable` | `answer` | low | — | 0 | 1.1y |
| [#5166](https://github.com/platformio/platformio-core/issues/5166) | COMPILATIONDB_INCLUDE_TOOLCHAIN=True skips WiFi | `question-answerable` | `answer` | medium | — | 0 | 1.1y |
| [#5171](https://github.com/platformio/platformio-core/issues/5171) | Library with `libCompatMode: strict` is not actually be | `implement-bug` | `implement` | medium | `medium` | 0 | 0.9y |
| [#5180](https://github.com/platformio/platformio-core/issues/5180) | Spaces in Windows User name not supported in Inspect | `implement-bug` | `implement` | high | `small` | 0 | 1.0y |
| [#5188](https://github.com/platformio/platformio-core/issues/5188) | Neovim + Arduino + PlatformIO | `question-answerable` | `comment-close` | medium | — | 1 | 0.6y |
| [#5193](https://github.com/platformio/platformio-core/issues/5193) | ExtraScripts: Allow hooking on errors | `implement-feature` | `keep` | medium | `medium` | 0 | 1.0y |
| [#5201](https://github.com/platformio/platformio-core/issues/5201) | symlinks in platformio.ini should use pre-binding to pa | `needs-info` | `comment-ping` | low | — | 0 | 0.8y |
| [#5205](https://github.com/platformio/platformio-core/issues/5205) | Upgrade cppcheck again! | `implement-feature` | `implement` | high | `trivial` | 0 | 0.4y |
| [#5208](https://github.com/platformio/platformio-core/issues/5208) | New Board BMduino(BM53A367A) Holtek | `out-of-scope` | `comment-close` | high | — | 0 | 1.0y |
| [#5217](https://github.com/platformio/platformio-core/issues/5217) | [feature] DFP (Device Family Pack) / atpack / CMSIS int | `keep-open` | `keep` | low | — | 1 | 0.9y |
| [#5231](https://github.com/platformio/platformio-core/issues/5231) | `pio device monitor` doesn't reconnect proactively when | `implement-feature` | `implement` | medium | `medium` | 0 | 0.8y |
| [#5233](https://github.com/platformio/platformio-core/issues/5233) | Add dynamic interpolation for package paths | `implement-feature` | `keep` | high | `medium` | 0 | 0.8y |
| [#5235](https://github.com/platformio/platformio-core/issues/5235) | How to manually specify the download server | `question-answerable` | `answer` | medium | — | 0 | 0.8y |
| [#5237](https://github.com/platformio/platformio-core/issues/5237) | Undefines in build_flags don't work if there's a space | `implement-bug` | `implement` | high | `small` | 0 | 0.7y |
| [#5240](https://github.com/platformio/platformio-core/issues/5240) | Add flag to disable self-upgrades | `implement-feature` | `implement` | high | `small` | 1 | 0.4y |
| [#5245](https://github.com/platformio/platformio-core/issues/5245) | Bug: build_src_flags has an impact on lib_deps | `needs-info` | `comment-ping` | low | — | 0 | 0.8y |
| [#5257](https://github.com/platformio/platformio-core/issues/5257) | OpenOCD fails to read peripheral memory via sysbus/prog | `out-of-scope` | `comment-close` | medium | — | 0 | 0.8y |
| [#5258](https://github.com/platformio/platformio-core/issues/5258) | Can it support FMD series microcontrollers? | `out-of-scope` | `comment-close` | high | — | 0 | 0.6y |
| [#5261](https://github.com/platformio/platformio-core/issues/5261) | Library Name Collision: External Registry Libraries Ove | `implement-bug` | `implement` | medium | `medium` | 0 | 0.6y |
| [#5262](https://github.com/platformio/platformio-core/issues/5262) | [feature request] PlatformIO CLI get board_build.mcu | `implement-feature` | `implement` | medium | `small` | 0 | 0.8y |
| [#5264](https://github.com/platformio/platformio-core/issues/5264) | How can I customize the default set of files generated | `question-answerable` | `answer` | high | — | 0 | 0.6y |
| [#5285](https://github.com/platformio/platformio-core/issues/5285) | Adding support for clang-format | `implement-feature` | `implement` | medium | `medium` | 0 | 0.3y |
| [#5288](https://github.com/platformio/platformio-core/issues/5288) | SVD parser rejects valid 64-bit registers — request cla | `out-of-scope` | `answer` | high | — | 0 | 0.3y |
| [#5299](https://github.com/platformio/platformio-core/issues/5299) | RAK3172 | `out-of-scope` | `comment-close` | high | — | 0 | 0.6y |
| [#5305](https://github.com/platformio/platformio-core/issues/5305) | Does not work correctly in pipx/uvx-style environments | `implement-bug` | `investigate` | high | `medium` | 1 | 0.6y |
| [#5308](https://github.com/platformio/platformio-core/issues/5308) | ESPHome: build sequence does not stop cleanly when down | `keep-open` | `investigate` | medium | — | 0 | 0.3y |
| [#5310](https://github.com/platformio/platformio-core/issues/5310) | Add "break" support to "pio test" ? | `implement-feature` | `implement` | low | `small` | 0 | 0.6y |
| [#5313](https://github.com/platformio/platformio-core/issues/5313) | Allow the project-folder location dialog to honor the d | `out-of-scope` | `comment-close` | high | — | 0 | 0.6y |
| [#5323](https://github.com/platformio/platformio-core/issues/5323) | Bug: sin1.contabostorage.com blocked by QUAD9 (DoH) DNS | `implement-bug` | `implement` | medium | `medium` | 4 | 0.5y |
| [#5343](https://github.com/platformio/platformio-core/issues/5343) | the disassembly doesn't work, switch to assembly is not | `needs-info` | `comment-ping` | low | — | 2 | 0.2y |
| [#5348](https://github.com/platformio/platformio-core/issues/5348) | Environment Switching Requires .pio Cleanup on Windows | `needs-info` | `comment-ping` | medium | — | 0 | 0.5y |
| [#5352](https://github.com/platformio/platformio-core/issues/5352) | Add TKM32F499 Board | `out-of-scope` | `comment-close` | high | — | 0 | 0.5y |
| [#5356](https://github.com/platformio/platformio-core/issues/5356) | Add linux_aarch64 support to platformio/toolchain-gccar | `out-of-scope` | `comment-close` | medium | — | 0 | 0.4y |
| [#5357](https://github.com/platformio/platformio-core/issues/5357) | Please add support for WCH MCUs | `out-of-scope` | `comment-close` | high | — | 0 | 0.5y |
| [#5367](https://github.com/platformio/platformio-core/issues/5367) | arduino UNO Q | `out-of-scope` | `comment-close` | high | — | 2 | 0.4y |
| [#5371](https://github.com/platformio/platformio-core/issues/5371) | Feature Request: Register a `platformio` Package URL (P | `keep-open` | `keep` | medium | — | 0 | 0.4y |
| [#5373](https://github.com/platformio/platformio-core/issues/5373) | Plugin for Zed | `question-answerable` | `answer` | high | — | 9 | 0.4y |
| [#5375](https://github.com/platformio/platformio-core/issues/5375) | Library dependencies are installed but not available du | `question-answerable` | `answer` | medium | — | 0 | 0.4y |
| [#5384](https://github.com/platformio/platformio-core/issues/5384) | Feature: Track source file for each config option in Pr | `implement-feature` | `implement` | medium | `medium` | 0 | 0.4y |
| [#5388](https://github.com/platformio/platformio-core/issues/5388) | Unnecessary googletest installs | `implement-bug` | `investigate` | medium | `medium` | 0 | 0.4y |
| [#5389](https://github.com/platformio/platformio-core/issues/5389) | KERNAL_TASK 263%. When using Project and Configuration | `needs-info` | `comment-ping` | medium | — | 0 | 0.3y |
| [#5393](https://github.com/platformio/platformio-core/issues/5393) | PIO on Apple Silicon without Rosetta | `keep-open` | `keep` | medium | — | 0 | 0.3y |
| [#5398](https://github.com/platformio/platformio-core/issues/5398) | Support Milk-V Duo and Duo 256M boards | `out-of-scope` | `comment-close` | high | — | 0 | 0.3y |
| [#5399](https://github.com/platformio/platformio-core/issues/5399) | Support running custom targets locally when using Remot | `implement-feature` | `implement` | medium | `medium` | 0 | 0.3y |
| [#5402](https://github.com/platformio/platformio-core/issues/5402) | Consider omitting other envs when using "pio test -v" | `implement-feature` | `implement` | medium | `small` | 0 | 0.3y |
| [#5403](https://github.com/platformio/platformio-core/issues/5403) | fix: validation relies on assert, which is stripped in | `implement-bug` | `implement` | high | `trivial` | 0 | 0.3y |
| [#5413](https://github.com/platformio/platformio-core/issues/5413) | Native unit test runner intermittently reports SIGSEGV | `needs-info` | `comment-ping` | medium | — | 0 | 0.3y |
| [#5414](https://github.com/platformio/platformio-core/issues/5414) | Minimum Python version >=3.6 is EOL — consider raising | `implement-feature` | `implement` | medium | `trivial` | 0 | 0.3y |
| [#5415](https://github.com/platformio/platformio-core/issues/5415) | Migrate from setup.py to pyproject.toml (PEP 621) | `implement-feature` | `implement` | medium | `small` | 0 | 0.3y |
| [#5418](https://github.com/platformio/platformio-core/issues/5418) | PlatformIO fails in pipx/uvx environments — pip not dec | `implement-bug` | `implement` | high | `trivial` | 0 | 0.3y |
| [#5420](https://github.com/platformio/platformio-core/issues/5420) | Support for uno q | `out-of-scope` | `comment-close` | high | — | 0 | 0.3y |
| [#5421](https://github.com/platformio/platformio-core/issues/5421) | `lib_ldf_mode = off` fails to resolve symlinked depende | `implement-bug` | `investigate` | medium | `medium` | 0 | 0.3y |
| [#5427](https://github.com/platformio/platformio-core/issues/5427) | Support distro-packaged Python environments (read-only | `implement-feature` | `keep` | high | `medium` | 7 | 0.1y |
| [#5430](https://github.com/platformio/platformio-core/issues/5430) | Can't use "(" character in a sysenv variable | `needs-info` | `comment-ping` | low | — | 0 | 0.2y |
| [#5434](https://github.com/platformio/platformio-core/issues/5434) | VScode test skipping based on test_filter | `needs-info` | `comment-ping` | low | — | 0 | 0.2y |
| [#5439](https://github.com/platformio/platformio-core/issues/5439) | Unexpected error occours when specifying a custom direc | `implement-bug` | `investigate` | medium | `small` | 0 | 0.2y |
| [#5449](https://github.com/platformio/platformio-core/issues/5449) | VSCode IntelliSense `forcedInclude` doesn't work | `implement-bug` | `implement` | high | `small` | 0 | 0.1y |
| [#5454](https://github.com/platformio/platformio-core/issues/5454) | Proposal: URML (substrate-neutral robot intent) capabil | `out-of-scope` | `comment-close` | high | — | 0 | 0.1y |
| [#5458](https://github.com/platformio/platformio-core/issues/5458) | PIO platform Longan Nano GD32 bricked | `out-of-scope` | `comment-close` | medium | — | 2 | 0.1y |
| [#5460](https://github.com/platformio/platformio-core/issues/5460) | Add board file for Waveshare ESP32-S3-Touch-AMOLED-1.64 | `out-of-scope` | `comment-close` | high | — | 0 | 0.0y |
| [#5461](https://github.com/platformio/platformio-core/issues/5461) | Home: Could not load recent projects | `question-answerable` | `answer` | medium | — | 0 | 0.0y |
| [#5463](https://github.com/platformio/platformio-core/issues/5463) | Libraries not installed from registry cannot be detecte | `implement-bug` | `investigate` | medium | `medium` | 0 | 0.0y |

## 付録B: コメント草案（out-of-scope 代表例）

対象外Issue向けの誘導コメントの代表例。実際の投稿時は該当する dev-platform リポジトリ名に置き換えてください。

**[#2947](https://github.com/platformio/platformio-core/issues/2947) Add support for rust**

> Thanks, and we appreciate the strong interest (lots of reactions here). Rust embedded development has its own mature toolchain (cargo, probe-rs, cortex-m-rtic) that doesn't map onto PlatformIO's C/C++ SCons-based build system, and first-class Rust support would need to be a dedicated platform/toolchain effort rather than a core change. As there's been no movement toward that in several years, I'm closing this core issue. Anyone interested in a Rust platform package is welcome to prototype one against the Registry.

**[#3805](https://github.com/platformio/platformio-core/issues/3805) Please add support for RPi Pico (RP2040)**

> RP2040 / Raspberry Pi Pico support is now available through development platforms outside platformio-core, e.g. the community `platform-raspberrypi` (https://github.com/maxgerhardt/platform-raspberrypi and others). Since dev-platform support lives outside Core and this is now covered, I'm closing this issue. See the registry and community forum for the current recommended platform.

**[#3672](https://github.com/platformio/platformio-core/issues/3672) Add support for the NRF9160**

> Thanks for the request. Chip and board support (nRF9160) is added through the Nordic dev-platform packages, not platformio-core (see https://github.com/platformio/platform-nordicnrf52 and https://github.com/topics/platformio-platform). Please open/track this against the relevant platform repo. As there's no core change here and it's been inactive for years, we'll close it.

**[#4028](https://github.com/platformio/platformio-core/issues/4028) Add support for Renesas MCUs**

> Thanks for the strong interest here. Renesas MCU support requires a dedicated development platform (toolchain, Arduino/FSP framework, board definitions), which is maintained outside platformio-core. This isn't something Core itself implements. We're closing this in Core; a community-maintained Renesas platform (see https://github.com/topics/platformio-platform) is the right home, and the discussion/interest can continue there.

**[#1035](https://github.com/platformio/platformio-core/issues/1035) Mongoose OS support**

> Thanks for the request. Framework support like Mongoose OS is provided through a development-platform package rather than platformio-core itself. No maintained platform package materialized in the years since, and Mongoose OS development has largely wound down. I'm closing this as out of scope for core; if a community platform is built it can be published to the Registry and tracked in its own repo.

**[#445](https://github.com/platformio/platformio-core/issues/445) Consider adding Particle Platform**

> Thanks for the interest, and apologies for the long silence. Development-platform support (like Particle) lives outside platformio-core in dedicated `platform-*` repositories and the registry, not in Core itself. Anyone is welcome to create and publish a Particle dev-platform following https://docs.platformio.org/en/latest/platforms/creating_platform.html . Since this can't be actioned in Core, I'm closing it here; please track it in a dedicated dev-platform repo.

**[#2052](https://github.com/platformio/platformio-core/issues/2052) Cypress - FreeSoC / PSoC support**

> Thanks for the interest. Adding support for a new MCU family like Cypress PSoC requires a dedicated development platform package (toolchain, framework, board JSONs), which lives outside platformio-core. There is currently no maintained PSoC dev-platform. We're closing this in Core; a community-maintained platform under https://github.com/topics/platformio-platform would be the right home. Duplicates #4136 and #4465 are tracked here too.

**[#2709](https://github.com/platformio/platformio-core/issues/2709) Sparkfun Edge Cortex M4 MCU Board Support**

> Thanks for the interest. Adding a specific board is done through a development platform and board JSON package, which live outside platformio-core. See https://docs.platformio.org/en/latest/platforms/creating_board.html for creating a custom board definition. Since this can't be actioned in Core and has been inactive for years, I'm closing it here.

**[#4046](https://github.com/platformio/platformio-core/issues/4046) Add support for BouffaloLab chips**

> Thanks for the extensive discussion here. Support for the BouffaloLab BL602/BL604 chips is a development-platform and Arduino-core effort (a community `platform-sifive`/bouffalo fork already exists, as covered above) rather than something implemented in PlatformIO Core. The core repository does not host individual chip/platform packages. Closing here as out of scope for core; continued work belongs in the relevant platform/framework repositories.

**[#4609](https://github.com/platformio/platformio-core/issues/4609) Teknic Clearcore support**

> Good news: ClearCore (SAME53-based) support was added in the atmelsam development platform. You can use it today with:
> 
> ```ini
> [env:clearcore]
> platform = https://github.com/patrickwasp/platform-atmelsam
> board = clearcore
> framework = arduino
> ```
> 
> Board/platform support lives in the platform repo (platform-atmelsam), not platformio-core, so we're closing this here. Please follow https://github.com/platformio/platform-atmelsam/issues/175 for upstreaming.

**[#147](https://github.com/platformio/platformio-core/issues/147) Add new platform "timsp432" and support for TI MSP432 Launch**

> Thanks for the interest here. Development-platform and board support (like TI MSP432 / Energia) is maintained in dedicated platform repositories, not in platformio-core. There has been no maintained community platform for MSP432 in the ~10 years since this was filed, so I'm closing this core issue as out of scope. If someone builds a platform package, it can be published to the PlatformIO Registry and tracked in its own repo (see https://github.com/topics/platformio-platform).

**[#3978](https://github.com/platformio/platformio-core/issues/3978) nRF5340-DK support**

> Thanks for the interest (and for looking into it yourself). Adding the nRF5340-DK / nRF53 is a development-platform task (platform-nordicnrf52 or a dedicated community platform, https://github.com/topics/platformio-platform), not a platformio-core change. Please track it there. Closing this core issue as out of scope.


---

*本レポートは 2026-07-05 時点のオープンIssue 293件を、本文・コメントを精読して自動生成したトリアージ結果です。機械可読な全データは同ディレクトリの `triage-data.json` を参照してください。*
