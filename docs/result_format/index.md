# Driving Log Replayer 結果ファイルフォーマット

driving_log_replayer で出力される結果ファイルのフォーマットについて述べる。

1 行毎に json フォーマットの文字列が入っている jsonl ファイル形式となっている。
評価モジュール毎に評価内容が違うので、出力される内容は異なるが大枠を揃えることでみやすさや、スクリプトでの加工しやすくする。

## フォーマット

各行以下の形式のフォーマットで出力される。
実際には一行の文字列だがみやすさのためにフォーマットしている。

```json
{
  "Result": {
    "Success": "true or false",
    "Summary": "評価した結果の要約"
  },
  "Stamp": {
    "System": "コンピュータの時刻 UNIX_TIME",
    "ROS": "シミュレーションの時刻"
  },
  "Frame": {
    "Ego": {
      "TransformStamped": { 
        "header": "mapからbase_linkへのtransform_stampedのヘッダ",
        "child_frame_id": "child_frameのid",
        "transform": "mapからbase_linkへのtransform",
        "rotation_euler": {
          "roll": "オイラー角での自車の姿勢（x軸）",
          "pitch": "オイラー角での自車の姿勢（y軸）",
          "yaw": "オイラー角での自車の姿勢（z軸）",
        },
      },
    },
    "評価毎に構成が異なる": "..."
  }
}
```

- Result: 実行したシナリオの評価結果
- Stamp: 評価した時刻
- Frame: 受け取った frame(topic)1 回分の評価結果と、判定に使用した値などの付属情報
  - Ego: `TransoformStamped`のデータ形式は<https://docs.ros2.org/galactic/api/geometry_msgs/msg/TransformStamped.html>を参照。

## json ファイルの出力

プログラムで結果ファイルを作る都合で、追記可能で扱いやすいファーマットとして jsonl を採用している。
しかし、人が目で見て結果を確認する場合は、json 形式にして vscode 等でフォーマットして構造化したほうが理解しやすい。
そのため、driving_log_replayer_cli を用いてシミュレーション実行した場合はデフォルトで jsonl ファイルを json に変換したファイルも出力される。

一方、ローカルで wasim で実行した場合や、クラウドで実行した場合は jsonl ファイルしか作られない。
jsonl ファイルを json に出力したい場合は、以下のコマンドで変換することができる。

```shell
# 結果ファイルの変換、output_directory以下のresult.jsonlをresult.jsonに変換する
driving_log_replayer simulation convert-result ${output_directory}
```

## 結果ファイルの分析

json 出力の項目でも述べた通り、json ファイルの出力はローカルで driving_log_replayer_cli を用いた場合のみ出力される。
なので、結果ファイルをグラフ化するなどの解析作業を行う場合は、jsonl のまま扱えるようにすると変換の手間が減らせる。

python を用いる場合は、[pandas を使用すると jsonl をのそのまま読み込むことができる](https://qiita.com/meshidenn/items/3ff72396fe85044bc74f "pandas")
