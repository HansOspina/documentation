---
title: メトリクスモニター
kind: documentation
description: メトリクスの値をユーザー定義のしきい値と比較する
further_reading:
  - link: monitors/notifications
    tag: Documentation
    text: モニター通知の設定
  - link: monitors/downtimes
    tag: Documentation
    text: モニターをミュートするダウンタイムのスケジュール
  - link: monitors/monitor_status
    tag: Documentation
    text: モニターステータスの参照
---
## コンフィグレーション

1. 検出方法を選択します。
    {{< img src="monitors/monitor_types/metric/alert_type.png" alt="alert type" responsive="true" >}}

    **[しきい値アラート][1]**は、選択されたタイムフレーム内の値を所定のしきい値と比較します。アラート生成条件セクションで追加のオプションを設定できます。どのような値が異常値であるかがわかっている場合に使用される、標準的なアラートです。

    **[変化アラート][1]**は、現在の値と過去のある時点の値との絶対変化量または変化率を指定のしきい値と比較します。比較されるデータポイントは、単一ポイントではなく、アラート条件セクションのパラメーターを使用して計算された値です。

    このタイプのアラートは、正確な「異常値」のしきい値がわかっていない場合に、メトリクスの急激な増減に加えて、緩やかな変化を追跡するのにも役立ちます。
   注: 計算値は絶対値ではありません。つまり、下方向の変化は負の値になります。

    **[異常検知][2]**は、傾向や 1 週間または 1 日単位の季節性パターンなどを考慮して、メトリクスの挙動が過去とは異なる時間を特定するためのアルゴリズム機能です。これは、しきい値ベースのアラート設定では監視することが困難または不可能な強い傾向や反復パターンを持つメトリクスに適しています。

    **[外れ値検知][3]**は、グループの他のメンバーと比較して挙動が異なるメンバーを検出するためのアルゴリズム機能です。 たとえば、プール内の 1 台の Web サーバーが異常な数のリクエストを処理しているので、置き換えの必要があることを検出できます。あるいは、1 つの AWS アベイラビリティーゾーン (AZ) で、他の AZ に比べて大幅に多い 500 エラーが発生しているという警告を早期に受け取ることで、その AZ で問題が発生している可能性を察知できます。

    **[予測検知][4]**は、メトリクスが今後どこに向かうかを予測するためのアルゴリズム機能です。強い傾向や繰り返しパターンがあるメトリクスに適しています。

2. 監視するメトリクスとスコープを選択します。
  {{< img src="monitors/monitor_types/metric/metric_scope.png" alt="metric scope" responsive="true" >}}
  現在 Datadog に送信しているすべてのメトリクスについてモニターを作成できます。

3. アラートグループを選択します。
    {{< img src="monitors/monitor_types/metric/alert_grouping.png" alt="alert grouping" responsive="true" >}}

    **Simple alert** は、すべての報告元ソースをまとめて集計します。集計値が以下で設定する条件を満たしたときに、アラートを 1 つ受け取ります。これは、単一のホストから受け取るメトリクス (例: `host:bits` の `system.cpu.iowait` の `avg`) を監視する場合や、多くのホストにまたがる集計メトリクス (例: `region:us-east` の `nginx.bytes.net` の `sum`) を監視する場合に最適です。

    **Multi alert** は、グループパラメーターに従って各ソースにアラートを適用します。たとえば、ディスク容量に関するアラートを生成するには、クエリを次のように作成して、ホストとデバイスでグループ化します。

        avg:system.disk.in_use{*} by {host,device}

    これにより、各ホストのデバイスごとに、容量不足のアラートが個別にトリガーされます。

4.  アラート条件を選択します。

  * **しきい値**のオプションは、選択したアラートタイプによって若干異なります。どちらのタイプの場合も、メトリクスに基づいてしきい値と比較方法を入力します。しきい値を変更すると、グラフのカットオフポイントを示すマーカーが更新されます。
  {{< img src="monitors/monitor_types/metric/metric_threshold.png" alt="metric threshold" responsive="true" >}}
  しきい値を入力する際は、メトリクスに基づいて、書式設定した値を使用できます。たとえば、`system.disk.used` のしきい値は、`20GB` と入力できます。
  **しきい値アラート**の場合は、データの時間集計方法を選択できます。アラートエンジンは、単一の系列を生成し、選択された集計を実行します。
  各オプションについて、以下で詳しく説明します。

    * **on average**: 系列の平均値が算出され、単一の値が生成されます。この値がしきい値と比較されます。このオプションは、モニタークエリの先頭に `avg()` [関数][5]を追加します。

    * **at least once**: 生成された系列内の値がいずれか 1 つでもしきい値を超えている場合に、アラートがトリガーされます。このオプションは、比較方法の選択に基づいて、モニタークエリの先頭に[関数][5]を追加します。below を選択した場合は `min()`、above を選択した場合は `max()` が追加されます。

    * **at all times**: 生成された系列内のすべてのポイントがしきい値から外れている場合に、アラートがトリガーされます。このオプションは、比較方法の選択に基づいて、モニタークエリの先頭に[関数][5]を追加します。above を選択した場合は `min()`、below を選択した場合は `max()` が追加されます。

    * **in total**: 系列内のすべてのポイントの合計値がしきい値から外れている場合に、アラートがトリガーされます。このオプションは、モニタークエリの先頭に `sum()` [関数][5]を追加します。

  - **変化アラート** オプションを選択した場合は、次の追加パラメーターを設定できます。

    -  change は値の絶対変化量であるのに対して、% change は過去の値と比較した値の変化率です (過去の値が 2 で現在の値が 4 の場合、% change は 100% になります)。
    - 指定のタイムフレーム内の値の変化を比較するには、比較の対象とする期間を選択します。期間は、5 分から 2 日の範囲で指定できます。
    - **しきい値アラート**と同様に、変化の計算に使用する時間集計とタイムウィンドウを選択します。

    - 異常検知の構成方法については、[異常検知のガイド][2]を参照してください。

    - 外れ値検知の構成方法については、[外れ値検知のガイド][3]を参照してください。

5. 評価を遅延させる **evaluation_delay** 時間 (秒単位) を負以外の整数で選択します。たとえば、この値を 300 (5 分)、タイムフレームを last_5m、時刻を 7:00 に設定すると、モニターは、6:50 から 6:55 までのデータを評価します。これは、モニターが評価時に常にデータを取得できるため、AWS CloudWatch などのバックフィルされるメトリクスで役立ちます。

6. オプションで、設定したタイムフレーム後に**データなしの通知**を行うことができます。選択するタイムフレームは、アラートウィンドウの少なくとも 2 倍以上である必要があります。たとえば、過去 5 分間のアラートを設定している場合は、データなしの通知を受け取るまでに、少なくとも 10 分間待つ必要があります。

**注:** データなしアラートのデフォルトの最大値は 24 時間です。この値を大きくしたい場合は、サポートにお問い合わせください。

7. **モニターのトリガー状態を自動的に解決する**オプションを選択することができます。このオプションを有効にすると、モニターは、定義されたリカバリしきい値を満たした場合ではなく、一定時間の経過後に解決します。一般には、このオプションは OFF のままにし、問題が解決したときだけアラートが解決することをお勧めします。

    このオプションは、エラー数のように、非常に疎なカウンターでよく使用されます。エラーの発生が停止すると、メトリクスの報告も停止します。したがって、解決をトリガーする値がなくなり、モニターが解決されなくなってしまいます。[リカバリしきい値][6]を設定することもできます。
    ただし、モニターが自動解決され、次回の評価時にクエリの値がリカバリしきい値を満たさない場合、モニターは再度アラートをトリガーします。



8. **通知オプション**を構成します。オプションの詳細については、[通知][7]ドキュメントを参照してください。

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/monitors/monitor_types/metric
[2]: /ja/monitors/monitor_types/anomaly
[3]: /ja/monitors/monitor_types/outlier
[4]: /ja/monitors/monitor_types/forecasts
[5]: /ja/graphing/miscellaneous/functions
[6]: /ja/monitors/faq/what-are-recovery-thresholds
[7]: /ja/monitors/notifications