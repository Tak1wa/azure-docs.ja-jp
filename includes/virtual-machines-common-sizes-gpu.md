---
title: インクルード ファイル
description: インクルード ファイル
services: virtual-machines-windows, virtual-machines-linux
author: dlepow
ms.service: multiple
ms.topic: include
ms.date: 03/05/2018
ms.author: danlep;azcspmt;jonbeck
ms.custom: include file
ms.openlocfilehash: 96826b2f8acd579cbfe30f2e524d94ce4867df30
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/16/2018
---
GPU 最適化済み VM サイズは、1 つまたは複数の NVIDIA GPU を備えた、特殊な用途に特化した仮想マシンです。 これらのサイズは、コンピューティング処理やグラフィック処理の負荷が高い視覚化ワークロードを意図して設計されています。 この記事では、このグループ内の各サイズのストレージのスループットとネットワーク帯域幅に加え、GPU、vCPU、データ ディスク、NIC の数や種類に関する情報を提供します。 

* **NC、NCv2、NCv3、ND** の各サイズは、CUDA や OpenCL ベースのアプリケーションやシミュレーション、AI、ディープ ラーニングなどの、コンピューティング集中型およびネットワーク集中型のアプリケーション、アルゴリズム用に最適化されています。 
* **NV** サイズは、リモートの視覚化、ストリーミング、ゲーム、エンコーディング、および OpenGL や DirectX などのフレームワークを使用する VDI シナリオ用に最適化されています。  


## <a name="nc-series"></a>NC シリーズ

NC シリーズ VM は [NVIDIA の Tesla K80](http://images.nvidia.com/content/pdf/kepler/Tesla-K80-BoardSpec-07317-001-v05.pdf) カードを備えています。 エネルギー調査アプリケーション向け CUDA やクラッシュ シミュレーション、レイ トレーシング レンダリング、ディープ ラーニングなどを活用することで、データをさらに高速に処理することができます。 NC24r 構成には、密結合並列コンピューティングのワークロード向けに最適化された、低待機時間かつ高スループットのネットワーク インターフェイスが搭載されています。


| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 | 最大 NIC 数 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_NC6 |6 |56 | 380 | 1 | 24 | 1 |
| Standard_NC12 |12 |112 | 680 | 2 | 48 | 2 |
| Standard_NC24 |24 |224 | 1440 | 4 | 64 | 4 |
| Standard_NC24r* |24 |224 | 1440 | 4 | 64 | 4 |

1 GPU = K80 カードの 2 分の 1 相当。

*RDMA 対応

## <a name="ncv2-series"></a>NCv2 シリーズ

NCv2 シリーズ VM は [NVIDIA Tesla P100](http://images.nvidia.com/content/tesla/pdf/nvidia-tesla-p100-datasheet.pdf) GPU を備えています。 これらの GPU は、NC シリーズの 2 倍以上の計算性能を有しています。 貯留層モデリング、DNA シーケンシング、タンパク質解析、モンテ カルロ シミュレーションをはじめとする従来の HPC ワークロードに、これらの最新の GPU を活用することができます。 NC24rs v2 構成には、密結合並列コンピューティングのワークロード向けに最適化された、低待機時間かつ高スループットのネットワーク インターフェイスが搭載されています。

> [!IMPORTANT]
> このサイズ ファミリーでは、ご利用のサブスクリプションの vCPU (コア) クォータが、各リージョンで 0 に初期設定されています。 このファミリーについては、[提供リージョン](https://azure.microsoft.com/regions/services/)で [vCPU クォータの引き上げを要求](../articles/azure-supportability/resource-manager-core-quotas-request.md)してください。
>

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 | 最大 NIC 数 |
| --- | --- | --- | --- | --- | --- | ---  |
| Standard_NC6s_v2 |6 |112 | 336 | 1 | 12 | 4 |
| Standard_NC12s_v2 |12 |224 | 672 | 2 | 24 | 8 |
| Standard_NC24s_v2 |24 |448 | 1344 | 4 | 32 | 8 |
| Standard_NC24rs_v2* |24 |448 | 1344 | 4 | 32 | 8 |

1 GPU = P100 カード 1 枚

*RDMA 対応

## <a name="ncv3-series"></a>NCv3 シリーズ

NCv3 シリーズ VM は [NVIDIA Tesla V100](http://www.nvidia.com/content/PDF/Volta-Datasheet.pdf) GPU を備えています。 これらの GPU は、NCv2 シリーズの 1.5 倍以上の計算性能を有しています。 貯留層モデリング、DNA シーケンシング、タンパク質解析、モンテ カルロ シミュレーションをはじめとする従来の HPC ワークロードに、これらの最新の GPU を活用することができます。 NC24rs v3 構成には、密結合並列コンピューティングのワークロード向けに最適化された、低待機時間かつ高スループットのネットワーク インターフェイスが搭載されています。

> [!IMPORTANT]
> このサイズ ファミリーでは、ご利用のサブスクリプションの vCPU (コア) クォータが、各リージョンで 0 に初期設定されています。 このファミリーについては、[提供リージョン](https://azure.microsoft.com/regions/services/)で [vCPU クォータの引き上げを要求](../articles/azure-supportability/resource-manager-core-quotas-request.md)してください。
>

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 | 最大 NIC 数 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_NC6s_v3 |6 |112 | 336 | 1 | 12 | 4 |
| Standard_NC12s_v3 |12 |224 | 672 | 2 | 24 | 8 |
| Standard_NC24s_v3 |24 |448 | 1344 | 4 | 32 | 8 | 
| Standard_NC24rs_v3* |24 |448 | 1344 | 4 | 32 | 8 |

1 GPU = V100 カード 1 枚。

*RDMA 対応

## <a name="nd-series"></a>ND シリーズ

ND シリーズは、AI やディープ ラーニングのワークロードを想定して GPU ファミリーに新たに追加された仮想マシンです。 トレーニングや推論で優れたパフォーマンスを発揮します。 ND インスタンスは [NVIDIA Tesla P40](http://images.nvidia.com/content/pdf/tesla/184427-Tesla-P40-Datasheet-NV-Final-Letter-Web.pdf) GPU を備えています。 これらのインスタンスは、Microsoft Cognitive Toolkit、TensorFlow、Caffe などのフレームワークを活用する AI ワークロードの単精度浮動小数点演算において、非常に高いパフォーマンスを発揮します。 ND シリーズでは GPU のメモリ サイズ (24 GB) も大幅に増強されているため、より大規模なニューラル ネット モデルにも対応できます。 NC シリーズと同様に、ND シリーズでは 2 番目に少ない待機時間、RDMA を利用した高スループットのネットワーク、InfiniBand との接続性などを備えた構成が利用できます。これにより、多数の GPU を利用した大規模なトレーニング ジョブを実行できます。

> [!IMPORTANT]
> このサイズ ファミリーでは、ご利用のサブスクリプションの vCPU (コア) クォータが、各リージョンで 0 に初期設定されています。 このファミリーについては、[提供リージョン](https://azure.microsoft.com/regions/services/)で [vCPU クォータの引き上げを要求](../articles/azure-supportability/resource-manager-core-quotas-request.md)してください。
>

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 | 最大 NIC 数 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_ND6s |6 |112 | 336 | 1 | 12 | 4 |
| Standard_ND12s |12 |224 | 672 | 2 | 24 | 8 | 
| Standard_ND24s |24 |448 | 1344 | 4 | 32 | 8 |
| Standard_ND24rs* |24 |1448 | 1344 | 4 | 32 | 8 |

1 GPU = P40 カード 1 枚

*RDMA 対応

## <a name="nv-series"></a>NV シリーズ

NV シリーズの仮想マシンは、[NVIDIA Tesla M60](http://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) GPU およびデスクトップ アクセラレータ アプリケーションや仮想デスクトップ向けの NVIDIA GRID テクノロジを備えていて、お客様は、データやシミュレーションを視覚化することができます。 NV インスタンスでは、グラフィックス処理を要するワークフローを視覚化して優れたグラフィックス機能を活用し、さらにエンコードやレンダリングなどの単精度のワークロードを実行することもできます。 

NV インスタンスの GPU ごとに GRID ライセンスが付属します。 このライセンスでは柔軟性が確保され、NV インスタンスを仮想ワークステーションとして 1 人のユーザーに対して使用したり、仮想アプリケーションのシナリオで 25 人のユーザーが同時に VM に接続したりできます。

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 | 最大 NIC 数 | 仮想ワークステーション | 仮想アプリケーション | 
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NV6 |6 |56 |380 | 1 | 24 | 1 | 1 | 25 |
| Standard_NV12 |12 |112 |680 | 2 | 48 | 2 | 2 | 50 |
| Standard_NV24 |24 |224 |1440 | 4 | 64 | 4 | 4 | 100 |

1 GPU = M60 カードの 2 分の 1 相当。

 
