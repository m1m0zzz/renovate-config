# renovate-config

m1m0zzz のリポジトリ共通の Renovate 設定プリセット

## 使い方

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>m1m0zzz/renovate-config"]
}
```

## プリセット

| プリセット | extends の書き方 | 更新の出し方 |
| :-- | :-- | :-- |
| `default` | `github>m1m0zzz/renovate-config` | 週次（金曜 20 時〜土曜終日・JST）。major 以外を 1 本の PR にまとめる |
| `active-update` | `github>m1m0zzz/renovate-config:active-update` | 随時。更新ごと（またはグループごと）に PR を出す |

両者とも共通の設定 `base` を extends する。`active-update` は `base` そのもので、`default` は `base` にスケジュールとグループ化を足したもの。`base` は部品として置いているため、リポジトリからは `default` か `active-update` を extends する。

### `default`（週次）

- PR を作るのは金曜 20 時〜土曜終日のみ（`schedule: ["* 20-23 * * 5", "* * * * 6"]`、`timezone: Asia/Tokyo`）。作成済みの PR の rebase と automerge は曜日を問わず続く
- `minor` / `patch` / `pin` / `pinDigest` / `digest` を `all non-major dependencies` の 1 本にまとめる
- ただし eslint 関連と prettier 関連は `lint and format tools` の別の 1 本にする。グループの PR には含まれる全更新のラベルが合算されて付くため、混ぜると `dependencies:lint` / `dependencies:format` の付いた PR に見た目に関わる更新が紛れ込み、ラベルで見分けられなくなる
- `major` はまとめない。automerge しない更新が入るとグループ全体が automerge されなくなるため、従来どおり個別（または下記のグループ単位）の PR になる
- `lockFileMaintenance` も同じ枠で出す。Renovate は lockfile の再生成を他の更新と同じ PR にまとめられないため、別の PR になる
- 脆弱性の更新は `vulnerabilityAlerts` の既定のまま。スケジュールに関係なく、別の PR ですぐに出る

## 方針（`base`。両プリセット共通）

`config:best-practices` + major 以外 automerge

| 更新の種類 | automerge | 補足 |
| :-- | :-- | :-- |
| `major` | ❌ | 破壊的変更を含みうるため人間が判断する |
| `minor` / `patch` | ✅ | CI が緑であることが条件 |
| `pin` / `pinDigest` / `digest` | ✅ | `config:best-practices` による GitHub Actions の digest 固定を含む |
| `lockFileMaintenance` | ✅ | lockfile 全体の再生成 |

TypeScript v7 / ESLint v10 / `eslint-plugin-astro` v3 の major は、上流の peer 対応まで更新を止めている。

その他の設定（`default` ではグループ化により major 以外は `all non-major dependencies` にまとまるため、以下のグループは主に major で効く）:

- 全 PR に `dependencies` ラベル。eslint 関連には `dependencies:lint`、prettier 関連には `dependencies:format` を追加
- `cloudflare` - wrangler と workers-types は peer 依存で結合しているためまとめて更新する
- `astro` - astro と `@astrojs/*` は peer で結合しているためまとめて更新する
- `lucide` - lucide のアイコンパッケージ（`lucide` / `lucide-react` / `@lucide/astro` など 11 個）は同一モノレポで同時にリリースされるためまとめて更新する。`@lucide/lab` と内部ツールは別のリリース列なので含めない

## 参照

- [Shareable Config Presets - Renovate Docs](https://docs.renovatebot.com/config-presets/#github)
- [Full Config Presets - Renovate Docs](https://docs.renovatebot.com/presets-config/#configbest-practices)
