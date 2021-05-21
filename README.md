# Google Smartphone Decimeter Challenge

## 目標
- indoorのリベンジ
- 今度こそはメダルが欲しい！
- indoorは結局終盤でNN使えなかったのでNN使えるようになりたい。
- indoorは記録をさぼってしまい、実験管理もできてなくてぐちゃぐちゃになったのでしっかりやる。


## Data
- ground_truth.csv(予想されるタイムスタンプでの参照位置)
  - millisSinceGpsEpoch : GPSのエポック（1980/1/6深夜UTC）からのミリ秒単位の整数値
  - latDeg, lngDeg : 基準となるGNSS受信機（NovAtel SPAN）で推定されたWGS84の緯度、経度（10進法）。位置情報を非整数のタイムスタンプに合わせるため、必要に応じて線形補間を行っています。
  - heightAboveWgs84EllipsoidM : 基準となるGNSS受信機で推定されるWGS84楕円体からの高さ（単位：メートル）。
  - timeSinceFirstFixSeconds : 最初に位置情報を取得してからの経過時間（単位：秒）。
  - hDop : GGA文のHorizontal dilution of precision DOPは、測定値の誤差が最終的な水平方向の位置推定にどのように影響するかを表しています。
  - vDop : Vertical dilution of precision DOP」（GSA文より）は、測定値の誤差が最終的な垂直方向の位置推定にどのように影響するかを説明している。
  - speedMps : 地上での速度（単位：メートル毎秒）。
  - courseDegree : 地上の真北を基準にした時計回りのコース角度（単位：度）。

- _derived.csv(生のGNSS測定値から得られたGNSS中間値で、便宜的に提供されています。)


## Log
