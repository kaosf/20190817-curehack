% 電子投票の超々簡易デモもどき
% ka
% 2019-08-17

# Author

ka

[![Gravatar](https://gravatar.com/avatar/884be098693425b409d25aaec5091de8?s=150)](https://gravatar.com/ka000)

Website: [kaosfield](https://www.kaosfield.net)

Twitter: [ka](https://twitter.com/ka_)

GitHub: [kaosf](https://github.com/kaosf)

# License

[![CC BY-NC-SA 4.0](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/4.0/)

Copyright (C) 2019 ka

# このページとリポジトリ

[https://kaosf.github.io/20190817-curehack](https://kaosf.github.io/20190817-curehack)

Repository: [kaosf/20190817-curehack - GitHub](https://github.com/kaosf/20190817-curehack)

#

このスライドは未調整状態です

発表時のまま残っていたり誤字があったりします

# 準同型暗号

平文 $m$ を暗号化する操作を $Enc(m)$ とする

暗号化ということは平文がノイズのように変わってしまう

…がそのノイズのようなもの同士でも計算を行うと  
平文で計算したものを暗号化したのと同じ結果になる性質

$$Enc(m_1) \otimes Enc(m_2) = Enc(m_1 \star m_2)$$

# 加法準同型暗号

$$Enc(m_1) \oplus Enc(m_2) = Enc(m_1 + m_2)$$

特に足し算が出来るものを加法準同型暗号という

今回この中でも特に Paillier 暗号について扱う

Paillierの読み方は「ペイエ」

参考: [Paillier暗号](https://ja.wikipedia.org/wiki/Paillier%E6%9A%97%E5%8F%B7)

Paillier暗号では

$$Enc(m_1) * Enc(m_2) = Enc(m_1 + m_2)$$

となる

暗号化したものを掛け算すると平文を足し算して暗号化したものと等しい

#

掛け算が出来るものは乗法準同型暗号

足し算と掛け算が両方出来るものは2009年になってようやく発見された

参考: [暗号文のままで計算しよう - 準同型暗号入門 -](https://www.slideshare.net/herumi/ss-59758244)

# 加法準同型暗号の応用

匿名性を完全に維持して電子投票が可能になる

「電子投票」と「インターネット投票」は別の話なので混同注意

# 匿名性を考えない電子投票について考える

例えば *きのこの山*, *たけのこの里*, *アルフォート* のどれが買うかを9人で投票して決めるとする

ここで

- きのこの山への投票は1を足す
- たけのこの里への投票は10を足す
- アルフォートへの投票は100を足す
- 棄権は0を足す

という操作を考える

#

もしも投票結果が $214$ になっていたなら…

$214 = 0 * 2 + 1 * 2 + 10 * 1 + 100 * 4$

であるはずなので

- きのこの山: 4
- たけのこの里: 1
- アルフォート: 2
- 棄権: 2

であったはずである

#

もし99人で投票する場合は

- きのこの山への投票は1を足す
- たけのこの里への投票は100を足す
- アルフォートへの投票は10000を足す
- 棄権は0を足す

とすれば良い

集計結果が

- 9900 なら 99 人がたけのこの里
- 990000 なら 99 人がアルフォート
- 198 なら 1 人がたけのこの里で 98 人がきのこの山


# プライバシーの問題

自分が何に投票したかは 0, 1, 10 or 100 で表現されるがこれを暗号化したい

そして暗号化されたまま合算されて復号されるとうれしい

# プリキュア大投票(縮小版)を考える

デモ

#

ふたりはプリキュアに投票→Splash☆Starに投票→ふたりはプリキュアに投票→Yesに投票

5GoGoにするとなんかバグるので赦して(TODO: Fix)

先程のお菓子の例と同様でそれぞれの作品への投票は

- 初代: 1
- MaxHeart: 8
- Splash☆Star: 64
- Yes: 512
- 5GoGo: 4096

を加算する扱いとなっている

# 実装

## 暗号化

平文と暗号文を $m$, $C$ とする

2つの異なる素数の積が $n$ でありそれを公開鍵をする

$$g = (1 + \alpha n) * \beta^n$$
$$C = g^m \pmod{n^2}$$

$\alpha$, $\beta$ は準同型の性質を崩さないように調整できる定数

$g$ は $n$ と $\alpha$, $\beta$ から計算出来る公開鍵や公開情報とだけ認識してくれればOK

(もうちょっと言っておくと $g$ は巡回群の生成元 *generator* です)

#

ただしこれでは同じ平文(投票先)からは常に同じ暗号文が生成されてしまう

そこで

$$C = g^m * r^n \pmod{n^2}$$

とする

ここで $r$ は $n$ と互いに素となる整数値から選ばれる乱数である

乱数を使って暗号化しても一意に復号が出来る(カーマイケルの定理より)

# 実装

## 復号

復号するための秘密鍵 $\lambda$ は以下の通り

$$\lambda = lcm(p-1, q-1)$$

#

以下のように復号する

$$C^\lambda = 1 + \alpha \lambda mn \pmod{n^2}$$

$$g^\lambda = 1 + \alpha \lambda n \pmod{n^2}$$

(↑これは二項定理を展開してみると証明出来ます)

$$L(x) = \frac{x - 1 \mod{n}}{n}$$

$$\frac{L(C^\lambda)}{L(g^\lambda)} = \frac{\alpha\lambda mn}{\alpha\lambda n} = m$$

ただしこれはモジュロ演算の割り算なのでモジュラ逆数を見つけて掛け算する操作になるので注意

$$L(C^\lambda) * (L(g^\lambda))^{-1} = m$$

# 加法準同型性の確認(これが言いたいだけ)

$$C_1 = g^{m_1} * r_1^n \pmod{n^2}$$
$$C_2 = g^{m_2} * r_2^n \pmod{n^2}$$

$$C_1 * C_2 = g^{m_1 + m_2} * (r_1 r_2)^n \pmod{n^2}$$

これを復号すると

$$m_1 + m_2$$

が得られることになる

# 問題点

- 有効票以外の値の投票を判別出来ない(暗号化されているので)
- 二重投票は防げない(これは別次元の問題になるので今は考えない)
- 秘密鍵保持者と集計者が結託するとダメ(これはどうにかする方法あるんだろうかよく分からない)

# 有効票以外の値の投票を弾きたい

この話をする

# ゼロ知識証明

相手に情報を与えずに(※相手が得る知識はゼロ)自分が持っている情報が正しいことを証明する方法

参考: [ゼロ知識証明](https://ja.wikipedia.org/wiki/%E3%82%BC%E3%83%AD%E7%9F%A5%E8%AD%98%E8%A8%BC%E6%98%8E)

今回は自分の票を明かすことは無く，しかし自分は有効な票を暗号化してあることを示したい

#

このデモを作りたかった…なにもできなかった…

# ざっくり紹介

まず投票者が4つの乱数 $z_k$ と $e_k$ を決める(この時点では隠しておく)

そこから計算出来る $a_k$ を投票所の人に提出する(Commitment)

投票所の人は $e_s$ をランダムに選んで返す(Challenge)

投票者は $e_s$ を受けて，$z_k$ と $e_k$ を(Response)

投票所の人は $a_k$, $z_k$, $e_k$ をゴニョゴニョ計算すると投票者が不正をしているかどうかが確率で分かる

何度も繰り返すと不正をしている人は高確率であぶり出される

参考:

- [Paillier Cryptosystem](http://security.hsr.ch/msevote/paillier)
