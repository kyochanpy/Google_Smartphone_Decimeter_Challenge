# Google Smartphone Decimeter Challenge

## 目標
- indoorのリベンジ
- 今度こそはメダルが欲しい！
- indoorは結局終盤でNN使えなかったのでNN使えるようになりたい。
- indoorは記録をさぼってしまい、実験管理もできてなくてぐちゃぐちゃになったのでしっかりやる。


## Data
- ground_truth.csv(予想されるタイムスタンプでの参照位置)
  - millisSinceGpsEpoch : GPSのエポック（1980/1/6深夜UTC）からのミリ秒単位の整数値.
  - latDeg, lngDeg : 基準となるGNSS受信機（NovAtel SPAN）で推定されたWGS84の緯度、経度（10進法）。位置情報を非整数のタイムスタンプに合わせるため、必要に応じて線形補間を行っています。
  - heightAboveWgs84EllipsoidM : 基準となるGNSS受信機で推定されるWGS84楕円体からの高さ（単位：メートル）。
  - timeSinceFirstFixSeconds : 最初に位置情報を取得してからの経過時間（単位：秒）。
  - hDop : GGA文のHorizontal dilution of precision DOPは、測定値の誤差が最終的な水平方向の位置推定にどのように影響するかを表しています。
  - vDop : Vertical dilution of precision DOP」（GSA文より）は、測定値の誤差が最終的な垂直方向の位置推定にどのように影響するかを説明している。
  - speedMps : 地上での速度（単位：メートル毎秒）。
  - courseDegree : 地上の真北を基準にした時計回りのコース角度（単位：度）。

- _derived.csv(生のGNSS測定値から得られたGNSS中間値で、便宜的に提供されています。)
  - millisSinceGpsEpoch : GPSエポック（1980/1/6深夜UTC）からのミリ秒単位の整数値。
  - [constellationType](https://developer.android.com/reference/android/location/GnssMeasurement#getConstellationType%28%29) : GNSSコンステレーションタイプ。constellation_type_mapping.csvで提供されるマッピング文字列値を持つ整数値。
  - [svid](https://developer.android.com/reference/android/location/GnssMeasurement#getSvid%28%29) : 衛星のIDです。
  - signalType : GNSS信号タイプは、コンステレーション名と周波数帯を組み合わせたものです。スマートフォンで測定される一般的な信号タイプには次のようなものがあります。GPS_L1、GPS_L5、GAL_E1、GAL_E5A、GLO_G1、BDS_B1I、BDS_B1C、BDS_B2A、QZS_J1、QZS_J5などがあります。
  - receivedSvTimeInGpsNanos : チップセットが受信した信号の送信時間を、GPSエポック以降のナノ秒単位で表したもの。[ReceivedSvTimeNanos](https://developer.android.com/reference/android/location/GnssMeasurement#getReceivedSvTimeNanos%28%29)から変換すると、この派生値はすべての星座で統一された時間スケールになりますが、ReceivedSvTimeNanosはGLONASSでは一日の時間、GLONASS以外の星座では週の時間を参照します。
  - [x/y/z]SatPosM : ttx = receivedSvTimeInGpsNanos - satClkBiasNanos（以下に定義）として定義される "真の信号送信時間 "の最良の推定値における、[ECEF座標フレーム](https://en.wikipedia.org/wiki/ECEF)内の衛星位置（メートル）です。これらは、衛星放送の[エフェメリス](https://novatel.com/support/known-solutions/gnss-ephemerides-and-almanacs)を用いて計算され、真の衛星位置に対して約1mの誤差がある。
  - [x/y/z]SatVelMps : 信号送信時刻（receivedSvTimeInGpsNanos）におけるECEF座標フレーム内の衛星速度（meter per second）。これらは、衛星放送のエフェメリスを用いて、[このアルゴリズム](https://fenrir.naruoka.org/download/autopilot/note/080205_gps/gps_velocity.pdf)で計算されます。
  - satClkBiasM : 信号の送信時刻（receivedSvTimeInGpsNanos）に、メートル単位のハードウェア遅延を組み合わせた衛星時刻補正を行います。これと同等の時間をsatClkBiasNanosと呼びます。
    - satClkBiasNanosは、satelliteTimeCorrectionからsatelliteHardwareDelayを差し引いた値に相当します。
    - IS-GPS-200Hの20.3.3.3.1項に定義されているように、satelliteTimeCorrectionは∆tsv = af0 + af1(t - toc) + af2(t - toc)2 + ∆trから算出され、satelliteHardwareDelayは20.3.3.3.2項に定義されている用語である。
    - 上記の式のパラメータは、衛星放送のエフェメリスに記載されています。
  - satClkDriftMps : 信号送信時刻（receivedSvTimeInGpsNanos）における衛星時計のドリフト（単位：メートル毎秒）。これは、t+0.5sとt-0.5sにおける衛星時計のバイアスの差に相当します。
  - rawPrM : メートル単位の生の疑似距離です。光速と、信号送信時刻（receivedSvTimeInGpsNanos）から信号到達時刻（[Raw::TimeNanos](https://developer.android.com/reference/android/location/GnssClock#getTimeNanos%28%29) - [Raw::FullBiasNanos](https://developer.android.com/reference/android/location/GnssClock#getFullBiasNanos%28%29) - [Raw::BiasNanos](https://developer.android.com/reference/android/location/GnssClock#getBiasNanos%28%29)）までの時間差との積です。
  - rawPrUncM :  メートル単位の生の疑似レンジの不確かさ。これは、光速と[ReceivedSvTimeUncertaintyNanos](https://developer.android.com/reference/android/location/GnssMeasurement#getReceivedSvTimeUncertaintyNanos%28%29)の積である。
  - isrbM : 非GPS-L1信号からGPS-L1信号までの信号間距離の偏り（ISRB）をメートル単位で表したもの。例えば、GPS L5のISRBMが1000mの場合、同じGPS衛星から送信されるGPS L1の擬似レンジよりも、GPS L5の擬似レンジの方が1000m長いことを意味します。GPS-L1信号ではゼロです。ISRBは、GPSチップセット・レベルで導入され、重み付けされた最小二乗エンジンの状態として推定されます。
  - ionoDelayM : [Klobuchar](http://www.navipedia.net/index.php/Klobuchar_Ionospheric_Model)モデルを用いて推定された電離層遅延（メートル）。
  - tropoDelayM : Nigel Penna, Alan Dodson, W. Chen (2001)がEGNOSモデルを用いて推定した[対流圏遅延時間](https://www.cambridge.org/core/journals/journal-of-navigation/article/abs/assessment-of-egnos-tropospheric-correction-model/1F187CB66A815FE22B75A1C2BFB728E2)（メートル）。



## Log
