# 旅行タイプ診断アプリ — プロジェクト進捗ログ

## プロジェクト概要

| 項目 | 内容 |
|------|------|
| アプリ名 | 夏旅プランナー 2026 |
| 本番URL | https://horirick06080601-blip.github.io/travel-planner/ |
| リポジトリ | https://github.com/horirick06080601-blip/travel-planner |
| 技術スタック | HTML/CSS/JS（単一ファイル）、GitHub Pages |
| ローカルパス | `/Users/horiuchisatoshi/Desktop/claude-chan_S/travel-planner/index.html` |

---

## ロードマップ全体

### Phase 0 — 戦略・基盤
- [ ] ターゲット定義・競合調査・コンセプト言語化
- [ ] ドメイン取得
- [ ] プライバシーポリシー作成

### Phase 1 — 基盤修正 ✅ 完了
- [x] **Step 1**: 国内/海外フィルターのバグ修正
- [x] **Step 2**: 出発国・地域の選択追加
- [x] **Step 3**: 性格診断の質問追加（Q4:旅のペース / Q5:宿泊スタイル）
- [x] **Step 4**: ペルソナタイプを25種類に拡張
- [x] UI修正（アイコンはみ出し修正、「最初から始める」ボタン追加）
- [x] GitHub CLI セットアップ（git push 自動化）
- [ ] **Step 5（未着手）**: GA4・Clarity導入（計測開始）

### Phase 2 — ビジュアル強化
- [ ] Step 6: 目的地カードに写真・画像を追加（Unsplash/Pixabay or AI生成）
- [ ] Step 7: 診断ステップごとのビジュアル装飾
- [ ] Step 8: ペルソナキャラクターイラスト作成・表示
- [ ] OGP設定・SNSシェア機能

### Phase 3 — コンテンツ強化
- [ ] Step 9: 目的地データ追加
- [ ] Step 10: 旅程の具体的プラン表示（日数別モデルコース）
- [ ] Step 11: 診断精度の改善

### Phase 4 — 集客基盤
- [ ] Step 12: WordPressブログ開設・連携
- [ ] Step 13: SNSアカウント運用開始（Instagram・X・TikTok）
- [ ] Step 14: 診断結果のSNSシェア機能
- [ ] Step 15: ブログ記事と目的地の紐付け

### Phase 5 — ユーザー参加（UGC）
- [ ] Step 16: 旅行体験の投稿機能（バックエンド必要）
- [ ] Step 17: 体験レビューの目的地紐付け
- [ ] Step 18: ユーザー認証・プロフィール（Supabase/Firebase）

### Phase 6 — マネタイズ
- [ ] Step 19: ブログ経由アフィリエイト（じゃらん・楽天・Amazonリンク）
- [ ] Step 20: アプリ内アフィリエイトリンク（Booking.com・HIS等）
- [ ] Step 21: SNSからの案件・PR収益
- [ ] Step 22: 旅行パッケージAPI連携（HIS・JTB）
- [ ] Step 23: プレミアム機能（月額課金）
- [ ] Step 24: スポンサー目的地・PR枠

---

## Phase 1 実装詳細

### Step 1: バグ修正（2026-04-15）

**問題**: 「国内のみ」選択時に海外目的地が表示される

**原因**: `scoreDestination()` 内の `d.area.includes('any')` が国内選択時も海外にスコアを与えていた

**修正箇所**（index.html）:
```javascript
// 修正前（バグあり）
if (d.area && (d.area.includes(S.area) || S.area === 'any' || d.area.includes('any'))) score += 8;

// 修正後①: スコアリング
if (S.area === 'domestic') {
  if (d.domestic) score += 8;
} else {
  if (d.area && (d.area.includes(S.area) || S.area === 'any' || d.area.includes('any'))) score += 8;
}

// 修正後②: ハードフィルター（showResults内）
.filter(d => {
  if (S.area === 'domestic') return d.domestic === true;
  if (S.area === 'asia' || S.area === 'hawaii' || S.area === 'europe') return d.domestic === false;
  return true;
})
```

---

### Step 2: 出発国・地域の選択追加（2026-04-15）

**実装内容**:
- Step 1に「お住まいの国・地域」選択を追加（5択）
- 日本以外を選択した場合、「国内だけ」エリアオプションをグレーアウト
- `S.homeCountry` をステートに追加
- Step 1の「次へ」ボタンは出発国 + 同行者の両方選択が必要

**選択肢**:
| 値 | ラベル |
|----|--------|
| japan | 🇯🇵 日本 |
| asia | 🌏 アジア・オセアニア |
| europe | 🌍 ヨーロッパ・中東 |
| americas | 🌎 北米・南米 |
| other | 🌐 その他の国 |

---

### Step 3: 性格診断の質問追加（2026-04-15）

**追加した質問**:

**Q4: 理想の旅のペースは？**
| 値 | ラベル | 対応stylesタグ |
|----|--------|---------------|
| slow_travel | 🐢 ゆっくり滞在型 | onsen_slow, luxury_hotel, historic_town |
| active_travel | ⚡ アクティブ詰め込み型 | adventure_sport, mountain_forest, city_walk |
| balanced_travel | ⚖️ バランス型 | city_walk, beach_resort, historic_town, street_food |
| spontaneous_travel | 🎲 気まま型 | city_walk, street_food, mountain_forest, beach_resort |

**Q5: どんな宿に泊まりたい？**
| 値 | ラベル | 対応stylesタグ |
|----|--------|---------------|
| luxury_stay | 🏨 高級ホテル・リゾート | luxury_hotel, beach_resort |
| local_stay | 🏠 民宿・ゲストハウス | historic_town, street_food, city_walk |
| design_stay | ✨ デザインホテル | city_walk, luxury_hotel |
| nature_stay | ⛺ グランピング・キャンプ | mountain_forest, beach_resort, adventure_sport |
| onsen_stay | ♨️ 温泉旅館 | onsen_slow |

**技術ポイント**: 目的地データを変更せず、既存stylesタグへのマッピングで対応。

---

### Step 4: ペルソナタイプ拡張（2026-04-15）

**Before**: 6タイプ（q2+q3の組み合わせベース）
**After**: 25タイプ（q1〜q5 + 同行者のスコアリングベース）

**スコアリング方式**:
```javascript
// 全ペルソナにスコアを計算し、最高点を返す
let best = PERSONAS[PERSONAS.length - 1], bestScore = -1;
for (const p of PERSONAS) {
  const s = p.score();
  if (s > bestScore) { bestScore = s; best = p; }
}
return best;
```

**25タイプ一覧**:
| # | emoji | 名前 | 主要判定軸 |
|---|-------|------|-----------|
| 1 | 🌄 | 絶景ハンター | beauty+view_memory+active |
| 2 | 📸 | フォトグラファー旅人 | beauty+view_memory+slow |
| 3 | 🍜 | 本場グルメ探検家 | taste+food_memory |
| 4 | 🍣 | 食文化・美食研究者 | taste+learn+local_stay |
| 5 | 🍷 | 高級美食・グルメ旅行家 | taste+luxury_stay |
| 6 | 🏄 | アドベンチャー派 | achieve+activity+active |
| 7 | 🏔️ | 山岳・自然チャレンジャー | achieve+nature_stay+outdoor |
| 8 | 🤿 | 海洋冒険者 | beauty+activity+outdoor |
| 9 | 🏛️ | 文化・歴史探求者 | learn+culture_memory+slow |
| 10 | 📚 | 知的好奇心旅人 | learn+culture_memory |
| 11 | 🎨 | アート・デザイン巡り派 | beauty+learn+design_stay |
| 12 | 🎭 | ローカル没入旅人 | local_stay+slow+learn |
| 13 | ♨️ | 温泉・和文化癒し派 | onsen_stay+slow+relax_memory |
| 14 | 💎 | ラグジュアリーリゾート派 | luxury_stay+relax_memory |
| 15 | 🌿 | デジタルデトックス旅人 | nature_stay+slow+outdoor |
| 16 | 🧘 | ウェルネス・スパ旅人 | relax_memory+slow+wellness系stay |
| 17 | 🌸 | 日本の美・和文化探求者 | domestic+onsen_stay+beauty/learn |
| 18 | 💑 | ロマンチック二人旅 | couple+connect+luxury/onsen |
| 19 | 👨‍👩‍👧 | ファミリー旅人 | family_kids/adult+connect |
| 20 | 🎉 | 友人グループ旅の企画者 | friends+social+connect |
| 21 | 🗺️ | バックパッカー気質 | spontaneous+local_stay+explore |
| 22 | 🌐 | 世界を歩く探検家 | explore+learn+spontaneous+any |
| 23 | ⚡ | アクティブ計画派 | active_travel+achieve+activity |
| 24 | 😌 | のんびり自由人 | slow/spontaneous+relax_memory |
| 25 | ✈️ | 自由な旅人（フォールバック） | 全問スコア0時 |

---

## UI修正ログ

| 日付 | 内容 | 状態 |
|------|------|------|
| 2026-04-15 | 家族（大人のみ）アイコンはみ出し修正（絵文字変更+CSS高さ固定） | ✅ |
| 2026-04-15 | ウィザード途中に「↺ 最初から始める」ボタン追加 | ✅ |

---

## 開発環境

```bash
# ローカル確認
cd /Users/horiuchisatoshi/Desktop/claude-chan_S/travel-planner
python3 -m http.server 8765
# → http://localhost:8765/index.html

# GitHub反映
git add index.html
git commit -m "説明"
git push origin main
# → 数分後に本番反映
```

---

## KPI（未計測・今後設定）

| 指標 | 目標値 | 計測ツール |
|------|--------|-----------|
| 診断完了率 | 60%以上 | Google Analytics 4 |
| 平均セッション時間 | 2分以上 | GA4 |
| SNS流入率 | - | UTMパラメータ |
| アフィリエイトCTR | 3%以上 | GA4イベント |
