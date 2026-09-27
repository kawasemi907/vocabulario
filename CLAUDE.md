# Vocabulario（西検4級 単語アプリ）

スペイン語技能検定（西検）4級の合格に必要な語彙（約1500語）を、100語ずつ15レベルに分けて学ぶ個人用のPWA。
GitHub Pagesで公開している。

## ファイル構成

- `index.html` … アプリ本体と単語データがすべてこの1ファイルに入っている。**編集するのはこのファイルだけ。**
- `manifest.webmanifest` / `sw.js` / `icon-192.png` / `icon-512.png` … 触らない。

単語データは `index.html` の `const LEVELS = [ ... ];` にある。各レベルの形式：

```js
{id:4, title:"Nivel 4", range:"301–400", words:[
  {w:"...", p:"...", en:"...", fr:"...", ex:"...", exEn:"...", exFr:"...", n:"..."},
  ...
]}
```

新しいレベルは、最後のレベルの `]}` の後ろに `,` を付けて追加し、`];` と `const TOTAL_LEVELS = 15;` の前に置く。

## 単語エントリのルール

| キー | 内容 |
|---|---|
| `w` | 見出し語。名詞は定冠詞付き（el / la / los / las）、動詞は不定詞、再帰動詞は -se 付き |
| `p` | 品詞。`nm` `nf` `nm pl` `nf pl` `v` `v refl` `adj` `adv` `prep` `conj` `pron` `interj` など |
| `en` / `fr` | 見出し語の英訳・仏訳。複数の意味はカンマ区切り |
| `ex` | スペイン語の例文。**ユーモアがある、またはスペイン語圏の文化・習慣・地名・食べ物が学べる内容**にする |
| `exEn` / `exFr` | 例文の自然な英訳・仏訳 |
| `n` | **日本語**の豆知識（2〜3文）。ラテン語などの語源、英語・フランス語との同源関係、関連する文法事項。語源が不確かな場合は断定しない |

- 各レベル **ちょうど100語**。
- 既存の全レベルと **見出し語 `w` が重複しない** こと。
- 西検4級（CEFR A2〜B1程度）の範囲から、頻出度の高い順に選ぶ。品詞のバランスは動詞・名詞・形容詞・その他（副詞・前置詞・接続詞など）を混ぜる。
- 例文はそのレベルの文法段階に合わせる：
  - レベル1〜3（作成済み）：直説法現在・点過去・現在完了・ir a・未来の基礎・再帰動詞
  - レベル4〜5：線過去と点過去の使い分け、未来、過去未来
  - レベル6〜8：命令法（肯定・否定）、接続法現在（願望・感情・疑い・目的・時の副詞節）
  - レベル9〜12：過去完了、未来完了、受身（ser + 過去分詞、再帰受身）、関係詞
  - レベル13〜15：4級範囲の総まとめ（上記を自然に混ぜる）
- 文字列はダブルクォートで囲み、**文字列の中にダブルクォートを使わない**（引用は — や言い換えで表現する）。
- 歌詞・詩・書籍の文章を引用しない。実在の人物に架空の発言をさせない。
- 例文のトーンや長さは既存のレベル1〜3に合わせる（1文〜2文程度）。

## 検証（レベルを追加するたびに必ず実行）

```bash
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/c.js && node --check /tmp/c.js && echo JS_OK
node -e '
const src=require("fs").readFileSync("/tmp/c.js","utf8");
const L=new Function(src.slice(src.indexOf("const LEVELS"),src.indexOf("const TOTAL_LEVELS"))+"; return LEVELS;")();
L.forEach(l=>console.log("Level",l.id,"words:",l.words.length));
const all=L.flatMap(l=>l.words.map(w=>w.w));
const dup=all.filter((w,i)=>all.indexOf(w)!==i);
console.log("duplicates:",dup.length?dup:"none");
const bad=L.flatMap(l=>l.words).filter(w=>!(w.w&&w.p&&w.en&&w.fr&&w.ex&&w.exEn&&w.exFr&&w.n));
console.log("incomplete entries:",bad.length);'
```

`JS_OK`、各レベル100語、`duplicates: none`、`incomplete entries: 0` を確認してからコミットする。

## 作業の進め方

- 1レベル作るごとに検証してコミットする（コミットメッセージ例：`Add Nivel 4`）。
- アプリのコード部分（HTML/CSS/関数）は、指示がない限り変更しない。

## 例文の単語タップ辞書（GLOSS）

例文の単語をタップすると意味が出る機能のデータ。`index.html` の `const TOTAL_LEVELS = 15;` の直後にある。

```js
const GLOSS_LEVELS = new Set([1]);   // 単語タップを有効にするレベル
const GLOSS = {
  "hablas":[["hablar","動詞","話す","tú・現在"]],
  "como":[["como","接続詞・副詞","〜のように、〜として",""],["comer","動詞","食べる","yo・現在"]],
  ...
};
```

- キーは例文に出てくる語形を**小文字**にしたもの（アクセント記号はそのまま）。値は `[元の形, 品詞, 日本語の意味, 活用・補足]` の配列で、文脈で意味が変わる語は複数並べる（よく使う意味を先に）。
- 名詞の元の形は定冠詞付き（la piedra）。品詞は日本語（名詞（男）・名詞（女）・動詞・動詞（再帰）・形容詞・副詞・前置詞・接続詞・代名詞・冠詞・数詞・間投詞・固有名詞 など）。
- 活用形の補足は「yo・現在」「él/ella・点過去」「tú・命令」「接続法現在」「過去分詞の女性形」などの書き方にそろえる。
- 新しいレベルに対応するときは、そのレベルの例文の全語形を追加してから `GLOSS_LEVELS` にレベル番号を足す。

検証（GLOSS を追加・変更したら実行）：

```bash
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/c.js && node -e '
const src=require("fs").readFileSync("/tmp/c.js","utf8");
const L=new Function(src.slice(src.indexOf("const LEVELS"),src.indexOf("const TOTAL_LEVELS"))+"; return LEVELS;")();
const G=new Function(src.slice(src.indexOf("const GLOSS_LEVELS"),src.indexOf("\n};",src.indexOf("const GLOSS ="))+3)+"; return [GLOSS_LEVELS,GLOSS];")();
const [lvs,GL]=G;
L.filter(l=>lvs.has(l.id)).forEach(l=>{
  const forms=[...new Set(l.words.flatMap(w=>w.ex.toLowerCase().match(/[a-záéíóúüñ]+/g)||[]))];
  console.log("Level",l.id,"forms:",forms.length,"missing:",forms.filter(f=>!GL[f]));
});
console.log("malformed:",Object.keys(GL).filter(k=>!Array.isArray(GL[k])||!GL[k].length||GL[k].some(s=>s.length!==4||!s[0]||!s[1]||!s[2])));'
```

各レベル `missing: []`、`malformed: []` を確認してからコミットする。
