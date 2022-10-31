# AHC015
## 問題文を読み解く
$10\times10$のマス目に100個のキャンディーが配置される。  
キャンディは3種類の味があり、配置されるキャンディの種類の順番は事前に与えられる。
一方で、キャンディが配置される位置はその時点での空いている位置からランダムに選ばれ、操作の後に与えられる。

配置されるたびに前後左右に箱を傾け、  
> ![動かす前](00_before_move.png)
> 図のように、前に傾けると上にキャンディが寄る。
> ![動かす前](01_after_forward_move.png)


同じ味のキャンディが集まっていると高得点。  
![スコア計算](02_compute_score.png)

## ビジュアライザを読み解く


ビジュアライザには
- 入力をまとめる構造体
- 入力、出力の受け取り関数
- 盤面を表す構造体
- スコア計算の関数
- （その他ビジュアライズ用のあれこれ）

が書かれています。
どんな戦法で戦うにしろスコア計算は必須ですね！？
さらに、問題文だけよりも、問題文とコードを両方読むほうが理解が深まりますね。深まりますよね？今の所、問題文で(~~数式とかは読み飛ばしつつ~~)概要を掴んだ後、ビジュアライザのコードを一行ずつ追っていくのが題意を理解する最短ルートです、私は…

というわけでビジュアライザを読んでみましょう
ローカルテスタのzipファイルを解凍すると
```
tools
├── Cargo.lock
├── Cargo.toml
├── README.html
├── README.md
├── in
│   └── 0000.txt ~ 0099.txt
├── seeds.txt
└── src
    ├── bin
    │   ├── gen.rs
    │   └── vis.rs
    └── lib.rs // これを読む
```
このようにファイルが展開されます。読むべきは`tools/src/lib.rs`です！
#### lib.rsの解説


##### ~38行目 ライブラリのインポート、マクロ
使うライブラリの指定と頻用マクロについて。私もよくわかっていません。
`SetMinMax` : chmin, chmaxが書いてある。ビジュアライザに書いてあることが多いけど、使われているのは見たこと無い。多分便利なんでしょう。便利なんですか？
`macro rules! mat` : 二次元以上の配列の初期化マクロ。

##### 40~43行目 定数と型の定義
```Rust
40 pub const N: usize = 10; // 箱の一辺の長さ
41 pub const M: usize = 3;  // 飴の種類の数
42
43 pub type Output = Vec<char>;
```
##### 45行目~97行目　入力と出力の受け取り

この辺りをコピペすると簡単に入力を受け取ってInput構造体にまとめてくれることが多いのですが、
今回は**インタラクティブ形式**(こちらの出力に合わせて入力が与えられる)のため、そのままだと使えません…


```Rust
45 #[derive(Clone, Debug)]
46 pub struct Input {
47    pub fs: Vec<usize>,  // 飴の種類の配列
48    pub ps: Vec<usize>,  // 飴の場所の配列
49 }
50
51 impl std::fmt::Display for Input {  // pythonで言う__str__の実装
52     fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
53         for i in 0..self.fs.len() {
54             if i > 0 {
55                 write!(f, " ")?;
56             }
57             write!(f, "{}", self.fs[i])?;
58         }
59         writeln!(f)?;
60         for &p in &self.ps {
61             writeln!(f, "{}", p)?;
62         }
63         Ok(())
64     }
65 }
66 
```
この実装のおかげでprintln!("{}", input)すると入力を確認することができるんですね〜

``` Rust
67 pub fn parse_input(f: &str) -> Input {
68     let f = proconio::source::once::OnceSource::from(f);
69     input! {
70         from f,
71         fs: [usize; N * N],
72         ps: [usize; N * N],
73     }
74     Input { fs, ps }
75 }
76 
```
本来はここを変更して(以下のコード)
``` Rust
pub fn parse_input(/* f: &str */) -> Input {
    // let f = proconio::source::once::OnceSource::from(f);
    input! {
        // from f,
        fs: [usize; N * N],
        ps: [usize; N * N],
    }
    Input { fs, ps }
}
```
そのまま本番も入力を受け取ることができるコンテストもあります！今回は無理です

<details>
<summary>出力の受け取りの部分(本筋と関係なし)</summary>
ビジュアライザなので、出力も受け取らなければならないです。以下が出力の受け取りの部分です。  
``` Rust
77 fn read(v: &str) -> Result<char, String> {
78     if v.len() != 1 {
79         Err(format!("Illegal output: {}", v))
80    } else {
81         Ok(v.chars().next().unwrap())
82    }
83 }
84 
85 pub fn parse_output(_input: &Input, f: &str) -> Result<Output, String> {
86     let tokens = f.lines();
87     let mut out = vec![];
88     for v in tokens {
89         let v = v.trim();
90         if v.len() == 0 {
91             continue;
92         }
93         out.push(read(v)?);
94     }
95     Ok(out)
96 }
97 
```
</details>


260 ~ 432行目 : 3種のキャンディの画像がSVG形式で埋め込まれている

