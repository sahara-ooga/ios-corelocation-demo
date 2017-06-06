# Core Location Framework

## 概要
Core Locationは、デバイスの現在の緯度・経度を決定し、位置情報を利用するためのフレームワークです。

## クラス
[CLLocation Class](https://github.com/stv-yokudera/ios-corelocation-demo#cllocation)<br>
[CLLocationManager Class](https://github.com/stv-yokudera/ios-corelocation-demo#cllocationmanager)<br>
[CLGeocoder Class](https://github.com/stv-yokudera/ios-corelocation-demo#clgeocoder)

## プライバシー設定
iOS8以降でデバイスの位置情報を使用する場合は、Info.plistに位置情報を使用する目的を記述しなければなりません（使用目的を記述しない場合、後述する位置情報の許可要求処理が失敗します）。

- 常に位置情報を使用する場合
<br>`Privacy - Location Always Usage Description`に使用目的を記述する

- アプリがフォアグラウンドにある間のみ位置情報を使用する場合
<br>`Privacy - Location When In Use Usage Description`に使用目的を記述する

![info.plistのイメージ](https://github.com/stv-yokudera//ios-corelocation-demo/wiki/images/info-plist.png)

ここで記述した使用目的は、ユーザーに位置情報を使用する許可を求めるアラートに表示されます。

`CLLocationManager`クラスの`requestAlwaysAuthorization()`メソッドを呼んだ時 

![requestAlwaysAuthorizationのイメージ](https://github.com/stv-yokudera//ios-corelocation-demo/wiki/images/requestAlwaysAuthorization.jpg)

`CLLocationManager`クラスの`requestWhenInUseAuthorization()`メソッドを呼んだ時

![requestWhenInUseAuthorizationのイメージ](https://github.com/stv-yokudera//ios-corelocation-demo/wiki/images/requestWhenInUseAuthorization.jpg)

ただし、位置情報の仕様に関する許可がすでに確定している、言い換えると`CLAuthorizationStatus`が`.notDetermined`以外の場合、`requestAlwaysAuthorization()`、`requestWhenInUseAuthorization()`メソッドを呼んでもユーザに許可を求めるアラートは表示されません。

## バックグラウンドの位置情報取得設定
バックグラウンドでも位置情報を取得する場合は、プロジェクトファイルを選択し、<br>`Capabilities`タブ->`Background Modes`の`Location updates`にチェックを入れます。

![バックグラウンドモードのイメージ](https://github.com/stv-yokudera//ios-corelocation-demo/wiki/images/backgroundmodes.png)

## 開発環境

| Category | Version |
|:-----------:|:------------:|
| Swift | 3.0.2 |
| Xcode | 8.2.1 |
| iOS | 10.0~ |


## 参考
https://developer.apple.com/reference/corelocation

<br>https://developer.apple.com/library/prerelease/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html

<hr>

# CLLocation

## 概要
CLLocationは、デバイスの位置データを表すためのクラスです。<br>
デバイスの位置の地理的座標、高度、測定が行われた時刻などを取得することができます。

通常、CLLocationクラスのインスタンスは、生成する必要はなく、CLLocationManagerオブジェクトがデリゲートに伝えます。また、location managerの`location`プロパティから直近の位置情報を取得することもできます。

しかしながら、独自の位置情報をキャッシュしたり、二点の距離を取得するためにCLLocationオブジェクトを生成することもできます。

## 関連クラス
NSObject、CLLocationManager

## イニシャライザ

| イニシャライザ | 説明 | サンプル |
|:-----------|:------------|:------------|
| init(latitude:longitude:) | 緯度と経度を指定してCLLocationオブジェクトを生成する | let location = CLLocation(latitude: lat, longitude: lon) |

## 主要プロパティ

| プロパティ名 | 説明 | サンプル |
|:-----------|:------------|:------------|
| coordinate | 座標情報 | location.coordinate.latitude<br>location.coordinate.longitude |
| altitude | メートル単位の標高.GPS機能が無い場合は無効 | location.altitude |
| floor | 建物の階 | location.floor |
| horizontalAccuracy | 緯度・経度のメートル単位の精度.負の値は緯度・経度が無効であることを表す | location.horizontalAccuracy |
| verticalAccuracy | 高度のメートル単位の精度.負の値は高度が無効であることを表す | location.horizontalAccuracy |
| timestamp | 位置の測定が行われた時刻 | locationManager.delegate = self |

## 主要メソッド

| メソッド名 | 説明 | サンプル |
|:-----------|:------------|:------------|
| distance(from:) | 2点間の距離をメートル単位で測定する | location.distance(from: otherLocation) |

## 参考
https://developer.apple.com/reference/corelocation/cllocation

<hr>

# CLLocationManager

## 概要
CLLocationManagerは、アプリケーションに位置情報に関連するイベントの設定・通知をするためのクラスです。
<br>位置情報を取得する間隔や位置情報の精度を設定することができます。

## 関連クラス
NSObject、CLLocation

## 位置情報取得までの流れ
1. ユーザーに認可を得る

　ユーザーの認可状態は、`CLLocationManager`クラスのタイプメソッド`authorizationStatus()`で取得できます。

2. `CLLocationManager`インスタンスを生成し、強参照で保持する。
3. 同インスタンスの`delegate`プロパティ(`CLLocationManagerDelegate`)を設定する。
4. 必要なプロパティを設定する
5. イベント通知を開始するメソッドを呼ぶ

## 現在位置を取得する
standard location serviceとsignificant location change serviceの２つがある。

|  | standard location service | significant location change service |
|:-----------|:------------|:------------|
|特徴|任意の精度で位置情報を取得し、位置が変わるたびに更新を取得することができる | 省電力|

最初の位置と、位置が変わったときだけ情報が必要な場合、significant location change serviceがより適合する。

どちらのサービスを使用しても、位置情報はlocation managerのデリゲートオブジェクトに通知される。

#### standard location serviceを使う
1. `desiredAccuracy`・`distanceFilter`プロパティを設定する
2. メソッドを呼ぶ：`requestLocation()`, `startUpdatingLocation()`, or `allowDeferredLocationUpdates(untilTraveled:timeout:)`

精度とバッテリ消費量は相関するので必要以上の精度を要求しないようにする。

#### significant location change serviceを使う
`startMonitoringSignificantLocationChanges()`メソッドを呼ぶ。

## 主要プロパティ

| プロパティ名 | 説明 | サンプル |
|:-----------|:------------|:------------|
| `desiredAccuracy` | 位置データの精度を設定する | `locationManager.desiredAccuracy = kCLLocationAccuracyBest` |
| `distanceFilter` | 位置情報を取得する最小間隔をメートル単位で設定する | `locationManager.distanceFilter = 100.0` |
| `delegate` | CLLocationManagerDelegateの更新イベントを受け取るためのデリゲートオブジェクト | `locationManager.delegate = self` |


## 主要メソッド

| メソッド名 | 説明 | サンプル |
|:-----------|:------------|:------------|
| `requestWhenInUseAuthorization()` | アプリがフォアグラウンドにある間、位置情報サービスの使用を許可することの確認をする | `locationManager.requestWhenInUseAuthorization()` |
| `requestAlwaysAuthorization()` | 常に位置情報サービスの使用を許可することの確認をする | `locationManager.requestAlwaysAuthorization()` |
| `startUpdatingLocation()` | デバイスの位置情報の取得を開始する | `locationManager.startUpdatingLocation()` |

### CLLocationManagerDelegateプロトコルのメソッド

|メソッド名|説明|必須|
|---|---|---|
|locationManager(_:didChangeAuthorization:) | 位置情報の許可状態が変更された時の処理 | - |
|locationManager(_:didUpdateLocations:) | 位置情報が更新された時の処理 | - |
|locationManager(_:didFailWithError:) | 位置情報の取得に失敗した時の処理 | - |

## 参考
https://developer.apple.com/reference/corelocation/cllocationmanager

[Requesting Permission to Use Location Services](https://developer.apple.com/documentation/corelocation/requesting_permission_to_use_location_services)

[Determining the Availability of Location Services](https://developer.apple.com/documentation/corelocation/determining_the_availability_of_location_services)
<hr>


# CLGeocoder

## 概要
CLGeocoderは、座標とその座標の情報(街路、都市、州および国など)を変換するためのクラスです。

## 関連クラス
NSObject、CLLocation

## 主要メソッド
Reverse-geocodingメソッドは、緯度経度情報から住所情報を返します。
Forward-geocodingメソッドは住所から緯度経度情報を返します。
追加情報（その場所の建物・特徴的な情報）を返す場合もあります。この場合は`CLPlacemark`オブジェクトで表されます。

| メソッド名 | 説明 | サンプル |
|:-----------|:------------|:------------|
| reverseGeocodeLocation(_:completionHandler:) | 指定した位置の逆ジオコーディング要求を送信する | geocoder.reverseGeocodeLocation(location, completionHandler: { (placemarks, error) in<br>    placemarks?.forEach{ print("\($0)") }<br>}) |

## 参考
https://developer.apple.com/reference/corelocation/clgeocoder
