# AGENTS.md

moNa2 v2 (分割型キーボード、右手にトラックボールとロータリーエンコーダ) 用の ZMK ファームウェア設定リポジトリ。
[DYA Studio](https://studio.dya.cormoran.works/) (ZMK Studio の拡張版) に対応しており、キーマップやトラックボール設定を Web UI から変更できる。

元は sayu-hub/zmk-config-moNa2-v2 の fork。2026-10-20 に fork 時のブランチを `main` に改名し、itouuuuuuuuu の独立リポジトリとして運用している。
キーマップは [itouuuuuuuuu/zmk-config-roBa](https://github.com/itouuuuuuuuu/zmk-config-roBa) と同じものを移植している。

## 絶対に守ること

- **リモートは origin (itouuuuuuuuu/zmk-config-moNa2-v2) のみ**。fork 元 (sayu-hub) への push・PR・issue・`gh api` など一切の操作をしない。
  `.claude/hooks/block-upstream.sh` (PreToolUse hook) が、fork 元のオーナー名を含む Bash コマンドや他リポジトリ宛ての `gh` 操作をブロックする。
  この hook は文字列一致なので、fork 元のオーナー名を含むコミットメッセージや `grep` もブロックされる。コミットメッセージには書かない。
- **main に直接コミットしない**。必ず `git checkout main && git pull` してから英語 kebab-case の feature ブランチを切る。
- マージはユーザーが行う。「マージして」と明示されたときだけ Claude がマージし、マージ後は `git checkout main && git pull` してローカルの feature ブランチを削除する。
- 実機の動作確認は Claude にはできない。PR 本文にはユーザーが確認する項目として書く。
- `gh` 操作は `-R itouuuuuuuuu/zmk-config-moNa2-v2` を明示する。

## ディレクトリ構成

```
build.yaml                      GitHub Actions のビルドマトリクス (5 ターゲット)
config/
  mona2.keymap                  キーマップ本体 (レイヤー 0〜8)
  mona2.json                    DYA Studio / keymap-drawer 用の物理レイアウト定義
  mona2_r.conf                  右手 (central) の Kconfig。Studio・トラックボール・BLE 設定はここ
  mona2_l.conf                  左手 (peripheral) の Kconfig
  paw3222.conf                  トラックボールを PAW3222 に切り替える EXTRA_CONF_FILE
  west.yml                      ZMK 本体 (cormoran fork) と各モジュールの参照先
boards/shields/mona2/
  mona2.dtsi                    左右共通の devicetree (物理レイアウト、matrix transform、エンコーダ)
  mona2_r.overlay               右手: トラックボール (PMW3610 / PAW3222)、Runtime Input Processor の設定
  mona2_l.overlay               左手: キーとエンコーダのみ
  Kconfig.defconfig             右手を central にする、PMW3610/PAW3222 両ドライバを有効化
  mona2.zmk.yml                 shield のメタデータ
.github/workflows/
  build.yml                     ZMK の build-user-config.yml (v0.3) を取り込んだビルド
  draw.yml                      keymap-drawer による図の生成 (config/mona2.keymap の push で自動実行)
keymap-drawer/                  draw.yml が生成する mona2.svg / mona2.yaml
zephyr/module.yml               このリポジトリを Zephyr モジュールとして読ませる
.claude/
  settings.json                 PreToolUse hook の登録
  hooks/block-upstream.sh       fork 元への操作をブロックする hook
```

## キーマップ (config/mona2.keymap)

macOS 前提 (LG = Command、LA = Option)。日本語入力は `LANG1` / `LANG2` で切り替える。

### roBa との差分

moNa2 は roBa に比べて 2 行目の左内側キー (roBa では `Cmd+-`) が 1 つ無い (42 キー、roBa は 43 キー)。
roBa のキーマップを移植するときは、各レイヤーから roBa の index 15 を除く。combo の key-positions も 15 以降は 1 ずつ減らす。
代わりに `Cmd+-` はレイヤー 1 の右内側キーに置いている。

### レイヤー

| 番号 | ノード名 (display-name) | 入り方 | 内容 |
|---|---|---|---|
| 0 | default_layer (DEFAULT) | - | QWERTY。ホームロウの A と X・C は mod-tap |
| 1 | layer_1 (SYM) | `&mo 1` (左親指) | 記号と矢印。右内側キーが `Cmd+-` |
| 2 | layer_2_mouse_click_and_func (FUNC) | Space 長押し (`&lt 2 SPACE`) | F1〜F12、マウスクリック、Ctrl+矢印 |
| 3 | layer_3_text_select (TXT_SEL) | `&mo 3` | Cmd+矢印、Cmd+Shift+矢印 |
| 4 | layer_4_mouse_layer (MOUSE) | 入る経路なし | マウスクリック。DYA Studio の「一時レイヤー」(自動マウスレイヤー) 用に確保 |
| 5 | layer_5_mouse_layer (NUM_SCRL) | Tab 長押し (`&lt_to_layer_0 5 TAB`) | テンキー、スクリーンショット。トラックボールがスクロールになる |
| 6 | layer_6 (OPT_ARW) | `&mo 6` | Option+矢印、Option+Shift+矢印 |
| 7 | layer_7_change_browser_tab (TAB) | LANG2 長押し (`&lt 7 LANG2`) | Cmd+数字 (ブラウザタブ切替) |
| 8 | layer_8_move_window (WIN_MGMT) | Q 長押し (`&lt 8 Q`) | Ctrl+Shift+文字 (ウィンドウ管理アプリ用)。**W の位置が `&studio_unlock`** |

### カスタム behavior

| 名前 | 内容 |
|---|---|
| `mt_slow` | hold-tap、tapping-term 300ms、balanced。ホームロウ修飾キー用 |
| `mt_fast` | hold-tap、tapping-term 145ms、balanced。`RCTRL SLASH` 用 |
| `lt_to_layer_0` | 長押しで `&mo`、タップで `&to 0` してからキー送出。Tab に使用 |
| `to_layer_0` | `&to 0` + キー送出のマクロ |
| `rsr_default` | エンコーダ用 runtime-sensor-rotate。デフォルトは音量 (時計回りで上げる)。DYA Studio からレイヤーごとに変更可能 |
| combo `lang1` | 位置 33 + 39 (LANG2 キー + Backspace キー) の同時押しで `LANG1` |

### 編集時の注意

- 全レイヤーの binding 数は **42**。編集後は次のスクリプトで確認する。
  ```sh
  awk '/bindings = </{c=0;f=1;next} f&&/>;/{print NR": "c; f=0} f{n=gsub(/&/,"&"); c+=n}' config/mona2.keymap
  ```
- 全レイヤーに `display-name` と `sensor-bindings = <&rsr_default>;` を付ける (DYA Studio での表示とエンコーダ設定のため)。
- `#include <behaviors/runtime-sensor-rotate.dtsi>` と `rsr_default` の定義は消さない。
- レイヤー番号は `mona2_r.overlay` の `active-layers` bitmask (スクロールレイヤー) や `&lt 8 Q` などと結びついている。レイヤーを増減するときは参照先をすべて更新する。
- keymap を変えたら左右両方の `.uf2` を書き込む必要がある。

## DYA Studio

- 右手 (central) を USB 接続し、**Q 長押し + W** で `&studio_unlock` を実行してからブラウザで Connect する。
- Studio で変更したキーマップ・トラックボール設定・エンコーダ設定は **右手側の Flash に保存され、ファームウェアの初期値より優先される**。
  リポジトリ側でキーマップを変えても、Flash に保存済みの値があると実機には反映されない。反映するには Studio のキーマップ画面で「リセット」を押すか、`settings_reset` を書き込む。
- 設定の保存は操作から 10 秒後 (`CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE=10000`)。変更直後にケーブルを抜かない。
- ユーザーが Studio で決めた値をリポジトリに「反映して」と言ったら、keymap や overlay/conf に同じ値を書いて初期値として固定する。
  (例: スクロールの Y 軸反転 → `mona2_r.overlay` の `&scroll_runtime_input_processor` に `y-invert;`)

## トラックボール (右手)

センサーは PMW3610 (デフォルト) または PAW3222。ドライバは badjeff/zmk-pmw3610-driver (zmk-0.3) と sekigon-gonnoc/zmk-driver-paw3222。
入力処理は cormoran の Runtime Input Processor で、`mona2_r.overlay` の `&trackball_central_listener` に `mouse` → `scroll` の 2 段で繋いでいる。

| 設定 | 場所 | 現在値 | 意味 |
|---|---|---|---|
| `cpi` | `mona2_r.overlay` (trackball_central) | 1000 | カーソル感度。白く光沢のあるボール (Keychron Nape Pro 用) で小さな動きを拾うため 600 から上げた |
| `invert-x` | 同上 | 有効 | COROPIT 版の X 軸反転。`invert-y` はこの個体では不要 |
| `force-awake` + `force-awake-4ms-mode` | 同上 | 有効 | ZMK が ACTIVE の間はセンサーを RUN に固定し、サンプリングを 4ms にする。roBa (kumamuk-git 版ドライバ) が性能レジスタに書く 0x0D と同じ動作。IDLE/SLEEP では通常の省電力に戻る |
| `active-layers` | `mona2_r.overlay` (`&scroll_runtime_input_processor`) | `0x20` (layer 5) | ボールの動きがスクロールになるレイヤーの bitmask |
| `y-invert` | 同上 | 有効 | 縦スクロール方向の反転 |
| `scale-divisor` | 同上 | 4 | スクロール速度。大きいほど遅い |
| `CONFIG_PMW3610_RUN_DOWNSHIFT_TIME_MS` | `config/mona2_r.conf` | 3264 | 省電力に落ちるまでの時間 (最大値)。roBa と同じ |
| `CONFIG_PMW3610_REST1_SAMPLE_TIME_MS` | `config/mona2_r.conf` | 20 | 省電力時のサンプル間隔。roBa と同じ |
| `CONFIG_PMW3610_REPORT_INTERVAL_MIN` | `config/mona2_r.conf` | 8 | HID レポートの最小間隔 (ms)。センサーは 4ms で読み続け、移動量を合算して送る。BLE (接続間隔 15ms 前後) でキューが詰まりラグになるのを防ぐ。USB 専用なら 4 か 0 |

- 省電力の 2 項目はデフォルト (128ms / 40ms) だと動き出しが鈍く「慣性があるように」感じるため変更した。依頼がない限り戻さない。
- `force-awake` の 2 項目は、roBa では反応するボール (Keychron) が moNa2 で反応しなかったため追加した。badjeff 版ドライバはこの指定が無いと性能レジスタ (0x11) に触らず、センサー既定の 8ms で動く。電池持ちとのトレードオフなので、不要になったら外してよい。
- スクロール・反転・スケーリングは DYA Studio の「トラックボール」タブから再ビルドなしで変更できる。overlay の値は初期値。
- `mona2.dtsi` 内で `runtime-input-processor.dtsi` を include してはいけない (`RC()` マクロが衝突する)。overlay 側で `#include "mona2.dtsi"` の後に include する。
- badjeff 版ドライバには `CONFIG_PMW3610_POLLING_RATE_125_SW` や `CONFIG_PMW3610_FORCE_AWAKE` は無い。roBa の conf をそのまま持ち込まない。

## ビルドと CI

### ビルドターゲット (build.yaml)

| artifact | shield | 備考 |
|---|---|---|
| mona2_r-pmw3610 | mona2_r rgbled_adapter | 右手、PMW3610。snippet `studio-rpc-usb-uart` 付き。**通常はこれ** |
| mona2_r-paw3222 | 同上 + `paw3222.conf` | 右手、PAW3222 の個体用 |
| mona2_r-logging | 同上 + USB ロギング | デバッグ用。snippet は `cdc-acm-console` |
| mona2_l rgbled_adapter-seeeduino_xiao_ble-zmk | mona2_l rgbled_adapter | 左手。センサー非依存なので 1 種類のみ |
| settings_reset-seeeduino_xiao_ble-zmk | settings_reset | BLE ペアリングと Studio の保存設定のリセット用 |

GitHub Actions (`build.yml`) が push / PR ごとに全ターゲットをビルドし、Artifacts の `firmware` に `.uf2` をまとめる。
ローカルに west / Zephyr 環境は無い前提。ビルド確認は `gh run list` / `gh run view` で行う。

### バージョン固定の経緯 (変更しないこと)

- `config/west.yml` の zmk は cormoran の DYA Studio 対応 fork **`v0.3-branch+dya`**。DYA Studio の各モジュール (ble-management / settings-rpc / runtime-input-processor / runtime-sensor-rotate) は fork 互換のコミットに pin してある。pin を動かすと v0.4 系の API 変更でビルドが壊れる。
- `.github/workflows/build.yml` は ZMK の `build-user-config.yml` (v0.3) を取り込んだコピーで、actions だけ新しいバージョンに上げ、ランナーを `ubuntu-24.04` に固定している (PR #5)。
  本家の再利用ワークフロー `@v0.3.0` に戻すと Node.js 20 非推奨と `ubuntu-latest` 移行の警告が出る。ZMK main 側の再利用ワークフローは、west.yml が v0.3 系のままだと ZMK compat チェックで失敗するので使えない。
- fork 直後は push で Actions が自動起動しなかった。一度 `gh workflow run` で手動起動して以降は自動で動く。

### 書き込み

- 右手側の設定 (`mona2_r.conf` / `mona2_r.overlay`) だけ変えた場合は右手の `.uf2` のみ書き込めばよい。
- keymap (`config/mona2.keymap`) を変えた場合は左右両方を書き込む。
- 無線がおかしいとき、または Studio の保存設定を消したいときは `settings_reset` → 左 → 右の順に書き込む。
- 書き込みは XIAO のリセットボタン 2 回押しでブートローダーに入り、マウントされたドライブへ `.uf2` をコピーする。

### キーマップ図

`draw.yml` は `config/mona2.keymap` の push で自動実行され、`keymap-drawer/mona2.svg` を再生成して `[Draw] ...` コミットを同じブランチに積む。
PR をマージすると keymap 本体と図のコミットがセットで main に入る。README はこの SVG を表示している。

## 言語

- コミットメッセージ・PR タイトル・PR 本文は日本語。ブランチ名は英語の kebab-case。
- ユーザーへの返答は日本語。
