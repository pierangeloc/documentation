---
algolia:
  tags:
  - エラー追跡
description: バックエンドサービスから収集したエラーを検索し、管理する方法をご紹介します。
further_reading:
- link: https://www.datadoghq.com/blog/service-page/
  tag: GitHub
  text: サービステレメトリー、エラー追跡、SLO などを一元的に把握します
- link: /tracing/trace_explorer/trace_view/
  tag: ドキュメント
  text: トレースエクスプローラーについて
- link: /tracing/error_tracking/explorer
  tag: ドキュメント
  text: エラートラッキングエクスプローラーについて
- link: /monitors/types/error_tracking/
  tag: ドキュメント
  text: エラー追跡モニターを作成する
kind: documentation
title: バックエンドサービスのエラー追跡
---

## 概要

Datadog によって収集されたエラーを一貫して監視することは、システムの健全性のために非常に重要です。個々のエラーイベントが多数存在する場合、トラブルシューティングのためにエラーの優先順位をつけることが困難になります。スタックトレースの追跡、トリアージ、デバッグを行うことで、バックエンドサービスの致命的なエラーの影響を最小化することができます。

**Backend Services** のエラー追跡のために [APM][4] をセットアップすると、問題リストにカードが入力されます。**APM** > **Error Tracking** に移動すれば、未解決の問題、無視されている問題、すべての問題を表示したり、量や年齢で問題をソートしたり、バックエンドサービスにあるすべてのカスタムおよびデフォルトのファセットで問題をフィルターしたりすることができます。

{{< img src="tracing/error_tracking/explorer_with_backend_issues.png" alt="APM のエラー追跡エクスプローラーは、バックエンドサービスの問題を表示します" style="width:100%;" >}}

エラー追跡は、以下のことを可能にします。

- エラー追跡イベントに関するアラートを設定します。これにより、致命的な問題が発生した場合に、常に情報を得ることができます。
- 類似のエラーを課題にまとめることで、重要なエラーをより簡単に特定し、ノイズを減らすことができます。
- 経時的に問題を監視するため、開始のタイミングや継続した場合の頻度を把握できます。
- 必要なすべてのコンテキストを 1 か所で収集することで、トラブルシューティングが容易になります。
- ソースコードリポジトリ内のトレース、Git blame、またはコミットにアクセスできます。

## エラースパンを追跡するために span タグを使用する

<div class="alert alert-info">エラー追跡は、APM でサポートされているすべての言語で利用でき、別の SDK を使用する必要はありません。</div>

Datadog トレーサーは、インテグレーションやバックエンドサービスのソースコードの手動インスツルメンテーションを通じて、エラーを収集します。トレース内のエラースパンは、**エラーがサービスエントリースパン (一番上のサービススパン) に位置する場合**、エラー追跡によって処理されます。このスパンには、追跡対象の `error.stack`、`error.message`、`error.type` [スパンタグ][1]が含まれている必要があります。

{{< img src="tracing/error_tracking/flamegraph_with_errors.png" alt="エラーのあるフレームグラフ" style="width:100%;" >}}

エラー追跡は、エラータイプ、エラーメッセージ、スタックトレースを形成するフレームを使用して、処理する各エラースパンのフィンガープリントを計算します。同じフィンガープリントを持つエラーはグループ化され、同じ問題に属します。詳しくは、[トレースエクスプローラーのドキュメント][2]を参照してください。

## トラブルシューティングやデバッグを開始するための問題点の検討

エラー追跡は、[エラー追跡エクスプローラー][3]でバックエンドのサービスから収集されたエラーを自動的に分類して表示します。

問題をクリックすると、エラーの概要、影響を受けたスパンの分散型トレーシング、最新の最も関連性の高いスタックトレース、スパンタグ、ホストタグ、コンテナタグ、およびメトリクスを表示します。

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/tracing/visualization/trace/?tab=spantags#more-information
[2]: /ja/tracing/trace_explorer/trace_view/?tab=spantags
[3]: /ja/tracing/error_tracking/explorer
[4]: /ja/tracing