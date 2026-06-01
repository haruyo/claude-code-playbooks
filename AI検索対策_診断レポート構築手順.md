# AI検索対策 診断レポート構築手順（GEO / AIO 診断｜GA4 + Search Console + サイト診断）

> **このドキュメントの読み手は「Claude Code」です。**
> お客様（人間）の代わりに、あなた（Claude Code）が上から順番に作業を進めてください。
>
> **この 1 ファイルだけで完結します。** STEP で必要な Python スクリプトの全文が掲載されているので、**各コードブロックをそのままファイルとして書き出して**ください。
>
> **前提：このファイルは「月次アクセスレポート構築手順」を実施済みのお客様向けです。** すでに作業フォルダに `gauth.py` / `token.json` / `sc_data.json`（場合により `ga4_data.json`）があり、GA4・Search Console に OAuth 認証でつながっている状態を前提にします。まだの場合は先に月次アクセスレポートの手順を完了させてください（この診断はその認証とデータをそのまま再利用します。**追加の費用はかかりません**）。
>
> 一部だけ人間にしかできない操作（サイトURLの確認、診断結果の確認）があります。その箇所は **【人間に依頼】** と明記してあります。
>
> **困ったときの相談先：<https://onoharyo.com>**

---

## ⏱ 情報の鮮度と更新方針（最初に必ず読む）

> **この手順書に書かれている「対策の中身」は、2026年5月31日時点の情報です。**
> AI検索（AI Overviews / AI Mode / ChatGPT / Perplexity など）は変化が非常に速く、**半年もすると推奨内容・クローラー名・計測方法が変わります**。そのため次のルールで運用してください。

- **生成する診断レポートには、必ず冒頭に「診断実施日」と「判断基準の情報日付（2026-05-31）」の両方を記載する。** どの時点の常識で診断したのかを後から分かるようにするためです。
- **このファイル自体を定期的に更新する。** 目安は **3〜6ヶ月に1回**。更新時は、後述の[「📌 メンテナンス：この手順書の更新のしかた」](#-メンテナンスこの手順書の更新のしかた)に沿って、Claude が `WebSearch` で最新情報を取り直し、変わった箇所（クローラー名・推奨スキーマ・Google公式見解・計測の正規表現など）を差し替えます。
- **古い可能性がある記述には診断レポート上で「※ ◯年◯月時点の情報。要再確認」と添える。** 断定しすぎないこと。

---

## この診断でわかること（成果物）

実行すると、作業フォルダに **`ai_search_diagnosis_YYYYMM.md`**（AI検索対策の診断レポート）が生成されます。内容は「**今できていること／できていないこと → どうすればよいか**」を、専門知識がなくても読める形でまとめたものです。構成は次の通りです。

| セクション | 内容 |
|------------|------|
| 診断サマリー | 4領域それぞれの達成度を ○ / △ / × で一覧。診断日と情報基準日を明記 |
| 領域① クロール許可 | AIクローラー（GPTBot・PerplexityBot 等）がサイトを読めるか。**最優先** |
| 領域② コンテンツの抽出しやすさ | 見出し構成・冒頭の答え・FAQ・最終更新日など、AIが引用しやすい作りか |
| 領域③ 構造化データ・運営者情報（E-E-A-T） | Organization / Article / FAQ スキーマ、運営者・著者情報の充実度 |
| 領域④ 計測（AI流入の見える化） | ChatGPT 等からの流入が実際にあるか、Search Console の質問型キーワード傾向 |
| 優先対策トップ5 | 上記から、いま着手すべき施策を効果・手間の観点で5つに絞って提示 |

> **スタンス（重要）：両論併記＋基本優先。** Google公式は「AI検索対策＝従来のSEOであり、llms.txt 等の専用施策は不要」と明言しています（[出典](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide)）。一方で業界では llms.txt 設置などを勧める声もあります。本診断は **エビデンスが固まっている基本施策を優先**し、効果が不確実な施策は「**任意：様子見でよい**」と明示します。中小企業が無駄な工数をかけないための配慮です。

---

## AI検索対策（GEO / AIO）とは何か — 4つの領域

「AI検索対策」とは、ChatGPT・Google AI Overviews / AI Mode・Perplexity・Microsoft Copilot などの **AIが生成する回答の中で、自社サイトが引用・参照・推奨される**ようにする取り組みです。従来のSEOが「検索結果ページで上位に出る」ことを狙うのに対し、AI検索対策は「**AIの回答本文に自社が登場する**」ことを狙います。やるべきことは大きく4領域に分かれます。

1. **クロール許可** — そもそもAIがサイトを読めなければ引用されません。引用ゼロの最大の原因はこれ。
2. **コンテンツの抽出しやすさ** — AIは各見出しの冒頭文や定義を抜き出して引用します。抜き出しやすい構造が有利。
3. **構造化データ・運営者情報（E-E-A-T）** — AIは「誰が・どんな根拠で」書いたかの信頼性を見ます。
4. **計測** — 効果を確認し改善するための見える化。GA4には標準のAIチャネルがありません。

> **大前提：AI検索対策は「魔法の裏技」ではありません。** Google公式の見解では、AI検索で評価される土台は従来のSEO（有用で独自性のあるコンテンツ＋技術的に正しいサイト構造）そのものです。本診断もその前提に立ちます。

---

## 全体の流れ

```
STEP 0  前提確認（月次レポート構築済みか・サイトURL）【人間に確認】
STEP 1  診断用データの自動収集（Claude が実行）
          1-A GA4 から「AI流入」を抽出（既存の token.json を再利用）
          1-B robots.txt を取得し AIクローラー許可状況を判定
          1-C 主要ページの HTML を取得し 構造化データ・見出し・更新日を点検
          1-D llms.txt の有無を確認
          → ai_diagnosis_data.json に保存
STEP 2  Claude による診断（自動データ＋目視レビューを突き合わせ）
STEP 3  診断レポート ai_search_diagnosis_YYYYMM.md を生成
STEP 4  お客様へ提示・優先対策の案内
定期メンテナンス（再診断の運用 ＋ この手順書自体の更新）
付録（robots.txt テンプレ・スキーマ例・GA4正規表現）
トラブルシューティング
```

---

## STEP 0: 前提確認 【人間に確認】

次を確認してください。

- **月次アクセスレポート構築を済ませているか**（作業フォルダに `gauth.py` と `token.json` があるか）。無ければ先にそちらを完了させる。
- **診断対象サイトのURL**（例：`https://example.com/`）。`profile.md` があればそこにも記載されているはず。
- `sc_data.json` があるか（Search Console データ。質問型キーワードの分析に使う。無くても診断は動くが領域④が一部省略される）。

> この診断は月次レポートと同じ作業フォルダ・同じ認証（`token.json`）で動きます。**新たな費用・新たなログインは原則不要**です。

---

## STEP 1: 診断用データの自動収集 【Claude が実行】

作業フォルダ直下に、以下のファイルを作成して実行します。

> **【Claude へ：コードの扱い方】**（月次レポート手順書と同じ方針）
> 掲載コードは **動作実績のあるベース（基準）** です。基本はこのまま使えますが、Google系ライブラリや各サイトのHTML構造・クローラー名は**時期によって変わります**。エラー・非推奨警告が出たら自己流に直さず、**まず `WebSearch` / `WebFetch` で最新の公式情報を確認**してから必要最小限の改修をしてください（取得する項目・出力構造・判定ロジックの意図は変えない）。原因不明なときは <https://onoharyo.com> へ相談を案内。

### `ai_config.py`（診断設定）

```python
# ===== AI検索対策 診断設定 =====
SITE_URL   = ""          # 診断対象サイト。例 "https://example.com/"（末尾スラッシュ推奨）
SITE_NAME  = "サイト名"   # レポートのタイトルに使う任意の名前
AI_REFERRAL_MONTHS = 3   # AI流入を調べる直近月数（GA4）

# 判断基準の情報日付（この手順書を更新したら必ず直す）
GUIDE_INFO_DATE = "2026-05-31"

# --- AIクローラー一覧（2026-05時点）。更新時はここを最新化する ---
# role: "search"=引用流入を生む / "training"=学習用 / "user"=ユーザー操作時の取得
AI_CRAWLERS = [
    ("GPTBot",            "OpenAI",     "training"),
    ("OAI-SearchBot",     "OpenAI",     "search"),
    ("ChatGPT-User",      "OpenAI",     "user"),
    ("ClaudeBot",         "Anthropic",  "training"),
    ("Claude-Web",        "Anthropic",  "search"),
    ("Claude-SearchBot",  "Anthropic",  "search"),
    ("PerplexityBot",     "Perplexity", "search"),
    ("Perplexity-User",   "Perplexity", "user"),
    ("Google-Extended",   "Google",     "training"),
    ("Applebot-Extended", "Apple",      "training"),
    ("CCBot",             "CommonCrawl","training"),
    ("Bytespider",        "ByteDance",  "training"),
    ("Meta-ExternalAgent","Meta",       "training"),
]

# --- AI流入の判定に使う参照元ドメイン（2026-05時点）。GA4の sessionSource と照合 ---
AI_REFERRAL_HOSTS = [
    "chatgpt.com", "chat.openai.com", "openai.com",
    "perplexity.ai", "claude.ai", "gemini.google.com",
    "copilot.microsoft.com", "bing.com", "deepseek.com",
    "grok.com", "x.ai", "meta.ai", "you.com", "felo.ai", "genspark.ai",
]

# --- 引用されやすさに効く代表的スキーマ（schema.org @type）---
KEY_SCHEMA_TYPES = ["Organization", "LocalBusiness", "Article", "BlogPosting",
                    "FAQPage", "Product", "BreadcrumbList", "Person", "WebSite"]
```

### `diagnose_ai.py`（自動診断データの収集 → ai_diagnosis_data.json）

```python
"""AI検索対策の機械的に判定できる項目を収集して ai_diagnosis_data.json に保存する。
   - 既存の gauth.py / token.json を再利用して GA4 から AI流入を抽出
   - robots.txt を取得し AIクローラーごとの allow/deny を判定
   - 主要ページの HTML から 構造化データ(JSON-LD)・H2見出し・更新日 を点検
   - llms.txt の有無を確認
   ※ このスクリプトは『機械的に分かること』のみ。コンテンツの質やE-E-A-Tの最終判断は
     STEP 2 で Claude が目視レビューして補う。"""
import sys, os, json, re, urllib.request, urllib.error
from datetime import date
from urllib.parse import urljoin, urlparse
import ai_config as C

sys.stdout.reconfigure(encoding="utf-8")
BASE = os.path.dirname(os.path.abspath(__file__))
OUTPUT = os.path.join(BASE, "ai_diagnosis_data.json")
UA = "Mozilla/5.0 (AI-Search-Diagnosis; +https://onoharyo.com)"


def fetch(url, timeout=20):
    """URL を取得して (status, text) を返す。失敗時は (None, "")。"""
    try:
        req = urllib.request.Request(url, headers={"User-Agent": UA})
        with urllib.request.urlopen(req, timeout=timeout) as r:
            charset = r.headers.get_content_charset() or "utf-8"
            return r.status, r.read().decode(charset, errors="replace")
    except urllib.error.HTTPError as e:
        return e.code, ""
    except Exception as e:
        print(f"  取得失敗 {url}: {e}", flush=True)
        return None, ""


# ---------- 領域① robots.txt / AIクローラー ----------
def parse_robots(text):
    """robots.txt を user-agent ブロックごとに { ua: [(directive, path)] } に分解。"""
    groups, current = {}, []
    for raw in text.splitlines():
        line = raw.split("#", 1)[0].strip()
        if not line or ":" not in line:
            continue
        field, value = [x.strip() for x in line.split(":", 1)]
        f = field.lower()
        if f == "user-agent":
            ua = value
            groups.setdefault(ua, [])
            current = [ua]
        elif f in ("allow", "disallow") and current:
            for ua in current:
                groups[ua].append((f, value))
    return groups


def crawler_allowed(groups, ua_name):
    """ua_name 専用ブロックがあればそれ、無ければ * を見て allow/deny を判定。
       returns: "allowed" / "blocked" / "no-rule"。"""
    block = None
    for ua in groups:
        if ua.lower() == ua_name.lower():
            block = groups[ua]; break
    if block is None:
        block = groups.get("*")
    if block is None:
        return "no-rule"   # ルール無し = 既定で許可
    # ルートを全面 Disallow しているか（Disallow: /）で簡易判定
    for directive, path in block:
        if directive == "disallow" and path.strip() == "/":
            return "blocked"
    return "allowed"


def diag_robots(site):
    robots_url = urljoin(site, "/robots.txt")
    status, text = fetch(robots_url)
    out = {"url": robots_url, "status": status, "found": bool(text), "crawlers": {}}
    groups = parse_robots(text) if text else {}
    for name, vendor, role in C.AI_CRAWLERS:
        out["crawlers"][name] = {
            "vendor": vendor, "role": role,
            "state": crawler_allowed(groups, name) if text else "no-robots",
        }
    return out


# ---------- 領域④ llms.txt ----------
def diag_llms(site):
    url = urljoin(site, "/llms.txt")
    status, text = fetch(url)
    return {"url": url, "status": status, "found": status == 200 and bool(text.strip())}


# ---------- 領域②③ 主要ページの中身 ----------
JSONLD_RE = re.compile(
    r'<script[^>]+type=["\']application/ld\+json["\'][^>]*>(.*?)</script>',
    re.DOTALL | re.IGNORECASE)
H2_RE = re.compile(r"<h2[^>]*>(.*?)</h2>", re.DOTALL | re.IGNORECASE)
TIME_RE = re.compile(r'datetime=["\']([0-9]{4}-[0-9]{2}-[0-9]{2})', re.IGNORECASE)
DATEMETA_RE = re.compile(
    r'(?:article:(?:published|modified)_time|datePublished|dateModified)["\']?\s*'
    r'[:=]\s*["\']([0-9]{4}-[0-9]{2}-[0-9]{2})', re.IGNORECASE)


def extract_schema_types(html):
    types = set()
    for block in JSONLD_RE.findall(html):
        for m in re.findall(r'"@type"\s*:\s*"([^"]+)"', block):
            types.add(m)
        for m in re.findall(r'"@type"\s*:\s*\[([^\]]+)\]', block):
            for t in re.findall(r'"([^"]+)"', m):
                types.add(t)
    return sorted(types)


def diag_page(url):
    status, html = fetch(url)
    if not html:
        return {"url": url, "status": status, "ok": False}
    h2s = [re.sub(r"<[^>]+>", "", h).strip() for h in H2_RE.findall(html)]
    dates = TIME_RE.findall(html) + DATEMETA_RE.findall(html)
    return {
        "url": url, "status": status, "ok": True,
        "schema_types": extract_schema_types(html),
        "h2_count": len([h for h in h2s if h]),
        "h2_sample": [h for h in h2s if h][:8],
        "has_date": bool(dates),
        "latest_date": max(dates) if dates else None,
        "text_len": len(re.sub(r"<[^>]+>", " ", html)),
    }


def pick_pages(site):
    """診断するページを選ぶ。トップ + sc_data.json の検索流入上位ページ数件。"""
    pages = [site]
    sc_path = os.path.join(BASE, "sc_data.json")
    if os.path.exists(sc_path):
        with open(sc_path, encoding="utf-8") as f:
            sc = json.load(f)
        months = sc.get("months", [])
        if months:
            last = sc["data"][months[-1]].get("pages", [])
            for p in sorted(last, key=lambda x: -x.get("clicks", 0))[:5]:
                u = p["page"]
                if u.startswith("http"):
                    pages.append(u)
    # 重複除去（順序維持）
    seen, uniq = set(), []
    for u in pages:
        if u not in seen:
            seen.add(u); uniq.append(u)
    return uniq[:6]


# ---------- 領域④ GA4 からの AI流入抽出（既存認証を再利用）----------
def diag_ai_referral():
    try:
        from gauth import get_credentials
        from google.analytics.data_v1beta import BetaAnalyticsDataClient
        from google.analytics.data_v1beta.types import (
            RunReportRequest, DateRange, Metric, Dimension)
        import config  # 月次レポートの config.py（GA4_PROPERTY_ID を持つ）
    except Exception as e:
        return {"available": False, "reason": f"GA4連携を再利用できません: {e}"}
    if not getattr(config, "GA4_PROPERTY_ID", ""):
        return {"available": False, "reason": "config.py の GA4_PROPERTY_ID が未設定"}

    from datetime import date, timedelta
    end = date.today()
    start = end - timedelta(days=30 * C.AI_REFERRAL_MONTHS)
    client = BetaAnalyticsDataClient(credentials=get_credentials())
    req = RunReportRequest(
        property=f"properties/{config.GA4_PROPERTY_ID}",
        date_ranges=[DateRange(start_date=start.isoformat(), end_date=end.isoformat())],
        metrics=[Metric(name="sessions")],
        dimensions=[Dimension(name="sessionSource")],
        limit=1000,
    )
    resp = client.run_report(req)
    ai_rows, total_ai = [], 0
    for row in resp.rows:
        src = row.dimension_values[0].value.lower()
        sess = int(row.metric_values[0].value)
        if any(host in src for host in C.AI_REFERRAL_HOSTS):
            ai_rows.append({"source": row.dimension_values[0].value, "sessions": sess})
            total_ai += sess
    ai_rows.sort(key=lambda x: -x["sessions"])
    return {"available": True, "period_days": 30 * C.AI_REFERRAL_MONTHS,
            "ai_sessions": total_ai, "by_source": ai_rows}


def main():
    if not C.SITE_URL:
        raise SystemExit("ai_config.py の SITE_URL が未設定です。診断対象サイトのURLを入れてください。")
    site = C.SITE_URL if C.SITE_URL.endswith("/") else C.SITE_URL + "/"
    print(f"[診断対象] {site}", flush=True)

    data = {
        "site": site,
        "site_name": C.SITE_NAME,
        "diagnosed_at": date.today().isoformat(),
        "guide_info_date": C.GUIDE_INFO_DATE,
    }

    print("① robots.txt / AIクローラー許可を確認...", flush=True)
    data["robots"] = diag_robots(site)

    print("④ llms.txt の有無を確認...", flush=True)
    data["llms_txt"] = diag_llms(site)

    print("②③ 主要ページの中身を点検...", flush=True)
    data["pages"] = [diag_page(u) for u in pick_pages(site)]

    print("④ GA4 から AI流入を抽出...", flush=True)
    data["ai_referral"] = diag_ai_referral()

    with open(OUTPUT, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"保存: {OUTPUT}", flush=True)


if __name__ == "__main__":
    main()
```

実行：

```powershell
python diagnose_ai.py
```

> このスクリプトは「機械的に分かること」だけを集めます。**コンテンツの独自性や E-E-A-T（誰が・どんな経験で書いたか）の最終評価は、STEP 2 で Claude が `ai_diagnosis_data.json` と実ページの目視で補います。**

---

## STEP 2: Claude による診断 【Claude が実行】

`ai_diagnosis_data.json` を読み込み、下記[「AI検索対策 診断チェック表」](#ai検索対策-診断チェック表早見表)の各項目について **○（できている）／△（一部）／×（できていない）** を判定します。機械データで分かる項目は自動で、コンテンツの質に関わる項目は **Claude が `WebFetch` で実ページを読んで目視判定**してください。`profile.md` があれば事業内容と照らして「売上につながるか」の観点も加えます。

判定の進め方：

1. `ai_diagnosis_data.json` を読む。`profile.md` があれば併せて読む。
2. **領域①（クロール許可）** … `robots.crawlers` を見て、`role="search"` のボット（OAI-SearchBot / Claude-Web / Claude-SearchBot / PerplexityBot）が `blocked` になっていないか確認。1つでも `blocked` なら最優先課題。`no-robots`（robots.txt 自体が無い）は基本問題なし（＝全許可）。
3. **領域②（抽出しやすさ）** … `pages` の `h2_count`・`has_date`・`latest_date` を見つつ、代表ページを `WebFetch` で読み、「各見出し直後に単独で答えになる文があるか」「冒頭に定義・結論があるか」「FAQや表があるか」「最終更新日が見えるか」を目視で確かめる。
4. **領域③（構造化データ・E-E-A-T）** … `pages[].schema_types` に `Organization`/`Article`/`FAQPage` 等があるか。`WebFetch` で運営者情報・著者・問い合わせ先・実体験に基づく独自記述があるかを確認。
5. **領域④（計測）** … `ai_referral` で AI からの流入が実際にあるか（`ai_sessions`）。`sc_data.json` の直近 `queries` から、**質問型（「とは」「方法」「比較」「おすすめ」「？」を含む）** や **指名検索（社名・商品名）** の割合を見る。AI は質問型クエリで引用されやすいので、質問型で表示があるテーマは強化候補。
6. 各領域の達成度（○/△/×）を集計し、**優先対策トップ5**を「効果の確実さ × 着手のしやすさ」で選ぶ。Google公式が効果を認めている基本施策（クロール許可・有用で独自なコンテンツ・正しい技術構造・Googleビジネスプロフィール）を上位に、効果が不確実な施策（llms.txt 等）は「任意・様子見」として下位または別枠に置く。

> ⚠️ **断定しすぎない。** AI検索の挙動は完全には観測できません。「引用されていない＝必ずこれが原因」とは言い切らず、「◯◯の可能性が高い」「まず確実な基本から」という表現にしてください。数字や固有名詞には出典・取得日を添える。

---

## STEP 3: 診断レポートの生成 【Claude が実行】

判定結果を **`ai_search_diagnosis_YYYYMM.md`** として作業フォルダに書き出します（`YYYYMM` は診断実施月）。次のテンプレートに沿って、データから引用した具体値で埋めてください。

```markdown
# AI検索対策 診断レポート — {SITE_NAME}

- 診断実施日：{diagnosed_at}
- 判断基準の情報日付：{guide_info_date}（この時点のAI検索の常識で診断しています）
- 対象サイト：{site}

> このレポートは「AIの検索・回答（ChatGPT / Google AI Overviews・AI Mode / Perplexity など）に
> 御社サイトが引用されやすくするための診断」です。変化の速い分野のため、3〜6ヶ月ごとの再診断を
> おすすめします。

## 診断サマリー
| 領域 | 達成度 | ひとこと |
|------|:----:|----------|
| ① クロール許可（最優先） | ○/△/× | … |
| ② コンテンツの抽出しやすさ | ○/△/× | … |
| ③ 構造化データ・運営者情報 | ○/△/× | … |
| ④ 計測（AI流入の見える化） | ○/△/× | … |

## いま着手すべき優先対策トップ5
1. （最優先）… — なぜ：… / どうする：… / 確実度：高
2. …
（効果が確実で手間が少ないものから順に。各項目に「確実度：高/中/低」を付ける）

## 領域①：クロール許可
- 現状：robots.txt は {found}。検索系AIボットの状態：OAI-SearchBot=…, Claude-Web=…, PerplexityBot=… 
- 診断：…（blockedがあれば「引用される前提が崩れている」と明記）
- 対策：…（付録の robots.txt テンプレを案内）

## 領域②：コンテンツの抽出しやすさ
- 現状：診断したページの見出し数・最終更新日・FAQ有無 …
- 診断：…
- 対策：…（見出し直後に結論／冒頭に定義／FAQ追加／最終更新日の明示 など）

## 領域③：構造化データ・運営者情報（E-E-A-T）
- 現状：検出スキーマ … / 運営者・著者情報 …
- 診断：…
- 対策：…（付録のスキーマ例を案内）

## 領域④：計測（AI流入の見える化）
- 現状：直近{period_days}日のAI経由セッション = {ai_sessions}（内訳：…）。質問型キーワードの傾向 …
- 診断：…（流入が0でも「まだ把握できていないだけ」の可能性。GA4のカスタムチャネル設定を案内）
- 対策：…

## 補足：いま「やらなくてよい／様子見でよい」こと
- llms.txt：Google公式は「不要」と明言。主要AI各社も本番採用していない（2026-05時点）。
  開発者向けドキュメントサイト以外では、設置しても流入への効果は確認されていない。**任意**。
- AI専用の文章書き換え・コンテンツの過度な細切れ：Google公式は不要としている。

## 次回の再診断
- 推奨時期：{diagnosed_at から3〜6ヶ月後}。同じ手順で再実行すると前回との比較ができます。
- この分野は変化が速いため、手順書（本体）も同時に最新化することをおすすめします。
```

> 出力後、**会話でも要点（サマリーと優先対策トップ3）を口頭で伝える**こと。レポートのファイルパスを必ず提示してください。

---

## STEP 4: お客様へ提示・優先対策の案内 【Claude が案内】

1. 生成した `ai_search_diagnosis_YYYYMM.md` をお客様に提示する（ファイルとして開けるようにする）。
2. **優先対策トップ3だけ**を会話で平易に説明する（全部を一度にやろうとしないよう促す）。特に領域①（クロール許可）に × があれば最優先。
3. 「対策の実装まで手伝いましょうか？」と確認し、希望があれば次を支援する。
   - robots.txt の修正案（付録テンプレを事業に合わせて調整）
   - 構造化データ（JSON-LD）の生成（付録例をベースに）
   - FAQ・見出し改善のための具体的な文面案（`profile.md` の事業内容を反映）
   - GA4 のAI流入カスタムチャネル設定手順（付録の正規表現）
4. 迷ったら **<https://onoharyo.com>** に相談できる旨を必ず添える。

---

## 📌 メンテナンス：この手順書の更新のしかた 【Claude が実行】

> AI検索は半年で景色が変わります。**3〜6ヶ月に1回**、この手順書本体を次の手順で最新化してください。

1. `WebSearch` で次を取り直す（クエリ例）。
   - 「AI crawlers robots.txt list (最新年)」 … `ai_config.py` の `AI_CRAWLERS` を更新
   - 「Google AI Overviews AI Mode optimization official guide」 … 領域②③の推奨と「やらなくてよいこと」を更新
   - 「track AI traffic GA4 referral regex (最新年)」 … `AI_REFERRAL_HOSTS` と付録の正規表現を更新
   - 「llms.txt adoption SEO impact (最新年)」 … 「様子見」扱いを続けるか見直す
2. 変わった箇所を差し替え、**冒頭の「情報の鮮度」日付と `ai_config.py` の `GUIDE_INFO_DATE` を更新日に直す**。
3. 変更点を簡単にメモして、お客様に「基準を◯年◯月時点に更新しました」と伝える。

> このファイル自体の定期更新を自動化したい場合（例：四半期ごとにリマインド）も <https://onoharyo.com> で相談できます。

---

## AI検索対策 診断チェック表（早見表）

> 各項目の「確実度」は **2026-05-31 時点**での効果の確からしさ。**高＝Google公式や複数の実データが支持／中＝有力だが決定的でない／低＝効果が不確実（様子見可）**。

| 領域 | 診断項目 | ○の基準 | 対策（×・△のとき） | 確実度 |
|------|----------|---------|----------------------|:----:|
| ① | 検索系AIボットの許可 | OAI-SearchBot / Claude-Web / PerplexityBot が robots.txt でブロックされていない | robots.txt の該当 Disallow を解除（付録テンプレ） | 高 |
| ① | CDN/WAFでの遮断 | Cloudflare等でAIボットを一括ブロックしていない | 管理画面でAIボット許可（検索系のみ許可も可） | 高 |
| ① | 本文がHTMLで読める | 重要本文がJSレンダリング無しで取得できる／ログイン・有料壁の外 | サーバーサイドレンダリング、重要情報を公開領域に | 高 |
| ② | 見出し直後の結論 | 各H2直後の1〜2文が単独で答えになっている | 各セクション冒頭に結論・定義を置く | 中 |
| ② | 冒頭の定義・要約 | ページ冒頭150〜200字に主題の定義/結論がある | リード文に要点を先出し | 中 |
| ② | FAQ・表・箇条書き | 質問形式の見出しや表・リストがある | よくある質問セクション・比較表を追加 | 中 |
| ② | 最終更新日の明示 | 記事に更新日が表示され、内容も新しい | 「最終更新：YYYY-MM-DD」を表示し定期更新 | 中 |
| ② | 独自性のある内容 | 一次情報・実体験・独自見解がある（焼き直しでない） | 自社の経験・データ・事例を加筆 | 高 |
| ③ | Organization スキーマ | トップ等に Organization/LocalBusiness のJSON-LD | 付録のJSON-LDを設置 | 中 |
| ③ | Article/FAQ スキーマ | 記事に Article、FAQに FAQPage がある | 記事テンプレに追加 | 中 |
| ③ | 運営者・著者情報 | 会社概要・著者・連絡先・実績が明確 | About/運営者ページを充実、著者プロフィール記載 | 高 |
| ③ | Googleビジネスプロフィール | （店舗/地域事業なら）登録・最新化済み | 登録・営業情報・写真を整備 | 高 |
| ④ | AI流入の可視化 | GA4でAI経由の流入を区別して見られる | カスタムチャネル/正規表現を設定（付録） | 中 |
| ④ | 指名・質問型での露出 | Search Consoleで指名検索・質問型クエリの表示がある | 該当テーマの記事・FAQを強化 | 中 |
| 任意 | llms.txt | （設置していれば）正しく置かれている | **当面は任意・様子見**。Google公式は不要としている | 低 |

---

## 付録A：robots.txt 推奨テンプレート（2026-05時点）

> **方針は事業判断**です。代表的な2パターンを示します。設置後は `python diagnose_ai.py` で再判定してください。クローラー名は変わるため、適用前に最新を確認すること。

**パターン1：AI検索に積極露出（おすすめ・多くの中小企業向け）** — 学習も含め広く許可し、引用機会を最大化。

```
User-agent: *
Allow: /

# 検索系AIを明示的に許可（引用流入を得る）
User-agent: OAI-SearchBot
Allow: /
User-agent: Claude-Web
Allow: /
User-agent: PerplexityBot
Allow: /

Sitemap: https://example.com/sitemap.xml
```

**パターン2：検索露出は得たいが、モデル学習には使わせたくない** — 検索系は許可、学習系は拒否。

```
User-agent: *
Allow: /

# 検索系は許可（引用流入を得る）
User-agent: OAI-SearchBot
Allow: /
User-agent: Claude-Web
Allow: /
User-agent: PerplexityBot
Allow: /

# 学習系はブロック（任意）
User-agent: GPTBot
Disallow: /
User-agent: ClaudeBot
Disallow: /
User-agent: CCBot
Disallow: /
User-agent: Google-Extended
Disallow: /
User-agent: Applebot-Extended
Disallow: /
```

> 注意：`Google-Extended` をブロックしても通常の `Googlebot`（検索インデックス）は止まりません。AI Overviews/AI Mode は Google検索のインデックスに依存するため、**Googlebot は必ず許可**しておくこと。

---

## 付録B：構造化データ（JSON-LD）の最小例

ページの `<head>` 内に設置。**表示内容と一致させる**こと（虚偽はNG）。

**運営者（Organization）**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "会社名",
  "url": "https://example.com/",
  "logo": "https://example.com/logo.png",
  "contactPoint": {"@type": "ContactPoint", "telephone": "+81-3-0000-0000", "contactType": "customer service"},
  "sameAs": ["https://x.com/...", "https://www.facebook.com/..."]
}
</script>
```

**よくある質問（FAQPage）**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "質問文をそのまま",
    "acceptedAnswer": {"@type": "Answer", "text": "簡潔な答え（ページ本文と一致させる）"}
  }]
}
</script>
```

> 記事ページは `Article`（著者・公開日・更新日を含む）も有効。地域・店舗事業は `Organization` の代わりに `LocalBusiness` を使う。設置後は [Rich Results Test](https://search.google.com/test/rich-results) で検証可能。

---

## 付録C：GA4 で AI流入を見える化する正規表現（2026-05時点）

GA4 の「カスタムチャネルグループ」または探索で、参照元に対して下記の正規表現でAI流入を束ねられます（時期で変わるため適用時に最新確認）。

```
chatgpt\.com|chat\.openai\.com|openai\.com|perplexity\.ai|claude\.ai|gemini\.google\.com|copilot\.microsoft\.com|bing\.com|deepseek\.com|grok\.com|x\.ai|meta\.ai|you\.com|felo\.ai|genspark\.ai
```

> 注意：AI経由の流入は**参照元が付かず「Direct」に分類されることが多い**（デスクトップアプリ等）。そのため数字は過小評価になりがちです。「0だから流入が無い」と断定せず、傾向把握の目安として扱ってください。

---

## トラブルシューティング

### `diagnose_ai.py` が GA4 部分で失敗する／`ai_referral.available=false`
月次レポートの `config.py`（`GA4_PROPERTY_ID`）と `gauth.py`/`token.json` が同じ作業フォルダにあるか確認。トークン失効時は月次レポート手順の STEP 2-1（同意画面を「本番公開」）を参照。GA4部分が使えなくても他の領域の診断は実行されます。

### robots.txt の判定が実態と合わない
本スクリプトの判定は「`Disallow: /`（全面ブロック）か」を簡易チェックするものです。部分的な Disallow（`/blog/` だけ等）や複雑な記述は、実ファイルを `WebFetch` で読んで Claude が目視で補ってください。

### ページ取得が 403 / タイムアウト
WAF・Cloudflare がボット風アクセスを弾いている可能性。これ自体が「AIボットも弾かれている」兆候のことがあるため、診断レポートに「クロール遮断の疑い」として記載する。`WebFetch` で人間ブラウザ相当の取得を試し、結果を補う。

### 構造化データが検出されない
JS で動的挿入している場合、本スクリプトの静的取得では拾えないことがあります。`WebFetch` やブラウザの検証で確認し、判定を補ってください。

### 情報が古いと感じる
冒頭「情報の鮮度」の日付を確認。3〜6ヶ月以上経っていれば、[「📌 メンテナンス」](#-メンテナンスこの手順書の更新のしかた)に従って `WebSearch` で最新化してから診断する。

### 認証情報の取り扱い
`token.json` / `client_secret.json` は秘密情報。共有・公開リポジトリへの commit は厳禁（`.gitignore` に追加）。

---

## 相談窓口

診断結果の読み方、対策の実装、再診断の自動化（定期リマインド）など、迷ったらいつでも **<https://onoharyo.com>** に相談できます。
