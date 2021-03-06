<properties
	pageTitle="DevTest ラボのカスタム イメージと数式の比較 | Microsoft Azure"
	description="環境に最適なものを判断できるように、VM ベースとして使用するカスタム イメージと数式の違いについて説明します。"
	services="devtest-lab,virtual-machines"
	documentationCenter="na"
	authors="tomarcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="devtest-lab"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/22/2016"
	ms.author="tarcher"/>

# DevTest ラボのカスタム イメージと数式の比較

## 概要
[カスタム イメージ](./devtest-lab-create-template.md)と[数式](./devtest-lab-manage-formulas.md)の両方を、[新しく作成した VM](./devtest-lab-add-vm-with-artifacts.md) のベースとして使用することができます。ただし、カスタム イメージが単に VHD に基づくイメージであるのに対して、数式は VM サイズ、仮想ネットワークとサブネット、アーティファクトなどの構成済み設定*に加え*、VHD に基づくイメージであるという重要な相違点があります。これらの構成済みの設定は、VM の作成時にオーバーライド可能な既定値で設定されます。この記事では、カスタム イメージを使用する場合と数式を使用する場合の利点 (長所) と欠点 (短所) についていくつか説明します。
 
## カスタム イメージの長所と短所
カスタム イメージでは、必要な環境から VM を作成するための静的で不変の方法が提供されます。

**長所**
- イメージからの VM のスピンアップ後は何も変わらないため、カスタム イメージからすばやく VM をプロビジョニングできます。つまり、カスタム イメージは設定のない単なるイメージであるため、適用する設定はありません。 
- 1 つのカスタム イメージから作成される VM は同じです。

**短所**
- カスタム イメージの一部の要素を更新する必要がある場合、イメージを再作成する必要があります。  

## 数式の長所と短所
数式では、必要な構成/設定から VM を作成するための動的な方法が提供されます。

**長所**
- 環境の変更は、アーティファクトを介してすぐにキャプチャできます。たとえば、リリース パイプラインから最新ビットの VM をインストールする必要がある場合、またはリポジトリから最新コードを登録する場合は、最新のビットをデプロイするか、ターゲットのベース イメージと共に数式に最新のコードを登録するアーティファクトを単に指定できます。VM を作成するためにこの数式が使用される場合は常に、最新のビット/コードが VM にデプロイ/登録されます。 
- 数式では、VM サイズや仮想ネットワークの設定など、カスタム イメージで指定できない既定の設定を定義できます。 
- 数式で保存されている設定は既定値として表示されますが、VM の作成時に変更できます。 

**短所**
- 数式から VM を作成すると、カスタム イメージから VM を作成する場合よりも時間がかかることがあります。

<!---HONumber=AcomDC_0427_2016-->