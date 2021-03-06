---
title: マルチプロセス デバッグ
description: このドキュメントでは、Visual Studio for Mac を使用して、同時に実行されている複数のプロセスをデバッグする方法について説明します。 たとえば、この機能を使用して、モバイル アプリケーションおよび Web サービス プロジェクトを同時にデバッグすることができます。
ms.prod: xamarin
ms.assetid: 852F8AB1-F9E2-4126-9C8A-12500315C599
author: davidortinau
ms.author: daortin
ms.date: 03/24/2017
ms.openlocfilehash: fb96dab2d9979a365964d4993d9c7fc7fee299f5
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73016556"
---
# <a name="multi-process-debugging"></a>マルチプロセス デバッグ

Visual Studio for Mac で開発された最新のソリューションには、一般的に、複数のプラットフォームをターゲットとする複数のプロジェクトが含まれています。 たとえば、Web サービス プロジェクトが提供するデータに依存するモバイル アプリケーション プロジェクトが含まれるソリューションがあります。 このソリューションを開発する場合、開発者はエラーを解決するために、必要に応じて両方のプロジェクトを同時に実行します。 [Xamarin Cycle 9 リリース](https://releases.xamarin.com/stable-release-cycle-9/)以降、同時に実行されている複数のプロセスを Visual Studio for Mac でデバッグできるようになりました。 その結果、ブレークポイントを設定し、変数を検査し、実行中の複数のプロジェクトでスレッドを表示することができるようになりました。 これは、_複数プロセスのデバッグ_と呼ばれます。 

このガイドでは、複数プロセスのデバッグをサポートする Visual Studio for Mac の主な変更点、複数プロセスをデバッグするソリューションを構成する方法、Visual Studio for Mac で既存のプロセスにアタッチする方法について説明します。

## <a name="requirements"></a>必要条件

複数プロセスのデバッグには Visual Studio for Mac が必要です。

## <a name="ide-changes"></a>IDE の変更

開発者が複数プロセスをデバッグしやすくするために、Visual Studio for Mac はユーザー インターフェイスにいくつか変更を加えました。 Visual Studio for Mac の**デバッグ ツールバー**は更新され、 **[ソリューション オプション]** フォルダーに新しい **[ソリューション構成]** が追加されました。 また、**スレッド パッド**に、実行中のプロセスと各プロセスのスレッドが表示されます。 Visual Studio for Mac には、 **[アプリケーション出力]** など、特定の情報に関する複数のデバッグ パッド (各プロセスに 1 つ) も表示されます。

### <a name="solution-configurations"></a>ソリューション構成

既定で、Visual Studio for Mac のデバッグ ツールバーの **[ソリューション構成]** 領域には、個々のプロジェクトが表示されます。 デバッグ セッションが開始されると、これは Visual Studio for Mac が開始し、デバッガーがアタッチされるプロジェクトになります。

Visual Studio for Mac で複数のプロセスを開始し、デバッグするには、_ソリューション構成_を作成する必要があります。 ソリューション構成には、 **[開始]** ボタンをクリックするか、&#8984;&#8617; (**Cmd + Enter キー**) を押してデバッグ セッションを開始するときに、ドメイン グループ ソリューションのプロジェクトに含める内容を記述します。 次のスクリーンショットは、複数のソリューション構成がある Visual Studio for Mac のソリューション例です。

![](multi-process-debugging-images/mpd01-xs.png "A solution with multiple solution configurations")

### <a name="parts-of-the-debug-toolbar"></a>[デバッグ] ツールバーの部分

[デバッグ] ツールバーは、ポップアップ メニューでソリューション構成を選択できるように変更されました。 このスクリーンショットは、[デバッグ] ツールバーの部分です。

![](multi-process-debugging-images/mpd02-xs.png "The parts of the debug toolbar")

1. **ソリューション構成** - [デバッグ] ツールバーの [ソリューション構成] をクリックし、ポップアップ メニューから構成を選択して、ソリューション構成を設定することができます。

    ![](multi-process-debugging-images/mpd03-xs.png "A sample popup with solution configurations")

2. **ビルド ターゲット** - プロジェクトのビルド ターゲットを特定します。 この機能は、前のバージョンの Visual Studio for Mac から変更されていません。
3. **デバイス ターゲット** - ソリューションを実行するデバイスを選択します。 プロジェクトごとに別のデバイスまたはエミュレーターを指定することができます。

    ![](multi-process-debugging-images/mpd04-xs.png "Popup showing the devices for a project")

### <a name="multiple-debug-pads"></a>複数のデバッグ パッド

複数ソリューション構成を開始すると、各プロセスに 1 つずつ、複数の Visual Studio for Mac パッドが表示されます。 たとえば、次のスクリーンショットでは、2 つのプロジェクトを実行しているソリューションの 2 つの **[アプリケーション出力]** パッドがあります。

![](multi-process-debugging-images/mpd05-xs.png "Output Pad for a solution configuration")

### <a name="multiple-processes-and-the-_active-thread_"></a>複数のプロセスと_アクティブ スレッド_

1 つのプロセスでブレークポイントに到達すると、そのプロセスは実行が一時停止されますが、他のプロセスは実行が続行されます。 単一プロセスのシナリオでは、1 組のパッドにスレッド、ローカル変数、アプリケーション出力などの情報を簡単に Visual Studio for Mac に表示できます。 ただし、複数のブレークポイントがあるプロセスが複数あり、場合によっては複数のスレッドもある場合、すべてのスレッド (とプロセス) のすべての情報を一度で表示しようとすると、開発者がデバッグ セッションの情報に対処しきれなくなる可能性があります。

この問題に対処するために、今後、Visual Studio for Mac は一度に 1 スレッド (_アクティブ スレッド_と呼ばれます) の情報のみを表示します。 ブレークポイントで一時停止する最初のスレッドが_アクティブ スレッド_と考えられます。 アクティブ スレッドは、開発者の注意が集中するスレッドです。 **ステップ オーバー** &#8679;&#8984;O (Shift + Cmd + O キー) などのデバッグ コマンドは、アクティブ スレッドに発行されます。

**スレッド パッド**には、ソリューション構成の検査対象によるすべてのプロセスとスレッドの情報が表示され、アクティブ スレッドに関する視覚的なキューが示されます。

![](multi-process-debugging-images/mpd06-xs.png "Thread pad for a solution configuration")

スレッドは、スレッドをホストするプロセスごとにグループ化されます。 アクティブ スレッドのプロジェクト名と ID は太字で表示され、右向きの矢印が、アクティブ スレッドの横の列に表示されます。 上のスクリーンショットで、**プロセス ID 48703** (**FirstProject**) の**スレッド番号 1** はアクティブ スレッドです。

複数のスレッドをデバッグする場合、**スレッド パッド**を使用して、アクティブ スレッドに切り替えて、そのプロセス (またはスレッド) のデバッグ情報を表示することができます。 アクティブ スレッドを切り替えるには、**スレッド パッド**で目的のスレッドを選択してダブルクリックします。

#### <a name="stepping-through-code-when-multiple-projects-are-stopped"></a>複数のプロジェクトが停止している場合のコードのステップ実行

複数のプロジェクトにブレーク ポイントがある場合、Visual Studio for Mac は両方のプロセスを一時停止します。 アクティブ スレッドのコードの**ステップ オーバー**のみが可能です。 スコープの変更によって、デバッガーがフォーカスをアクティブ スレッドから切り替えることができるようになるまで、他のプロセスは一時停止されます。 たとえば、次のスクリーンショットのように、Visual Studio for Mac で 2 つのプロジェクトをデバッグしているとします。

![](multi-process-debugging-images/mpd09-xs.png  "Visual Studio for Mac debugging two projects")

このスクリーンショットで、各ソリューションにはそれぞれブレークポイントがあります。 デバッグが始まり、最初に到達するブレークポイントは、**SecondProject** の `MainClass` の**行 10** です。 どちらのプロジェクトにもブレークポイントがあるので、各プロセスが停止します。 ブレークポイントに到達すると、**ステップ オーバー**の各呼び出しによって、Visual Studio for Mac はアクティブ スレッドのコードをステップ オーバーします。

コードのステップ実行はアクティブ スレッドに限定されるので、Visual Studio for Mac は一度にコード行をステップ実行するときに、他のプロセスは一時停止されたままです。

前のスクリーンショットを例として説明します。`for` ループが完了すると、`MainClass` の**行 11** のブレークポイントに到達するまで、Visual Studio for Mac は **FirstProject** の実行を許可します。 各**ステップ オーバー** コマンドについて、Visual Studio for Mac 内部のヒューリスティック アルゴリズムによってアクティブ スレッドが **SecondProject** に戻るまで、デバッガーは **FirstProject** を 1 行ずつ進みます。

いずれかのプロジェクトにのみブレークポイントが設定されている場合、そのプロセスのみが一時停止されます。 開発者が一時停止するまで、またはブレークポイントが追加されるまで、他のプロジェクトは引き続き実行されます。

### <a name="pausing-and-resuming-a-processes"></a>プロセスの一時停止と再開

プロセスを一時停止または再開するには、プロセスを右クリックし、コンテキスト メニューから **[一時停止]** または **[再開]** を選択します。

![](multi-process-debugging-images/mpd08-xs.png "Pause or resume in the Thread pad")

デバッグ ツールバーの外観は、デバッグ対象のプロジェクトの状態に応じて変わります。 複数のプロジェクトが実行中の場合、少なくとも 1 つのプロジェクトが実行中で 1 つのプロジェクトが一時停止であれば、デバッグ ツールバーには **[一時停止]** ボタンと **[再開]** ボタンの両方が表示されます。

![](multi-process-debugging-images/mpd07-xs.png  "Debug toolbar")

**[デバッグ] ツールバー**の **[一時停止]** ボタンをクリックすると、デバッグ対象のすべてのプロセスが一時停止されます。 **[再開]** ボタンをクリックすると、一時停止されているすべてのプロセスが再開されます。

### <a name="debugging-a-second-project"></a>2 つ目のプロジェクトのデバッグ

Visual Studio for Mac で 1 つ目のプロジェクトの開始後に、2 つ目のプロジェクトをデバッグすることもできます。 1 つ目のプロジェクトが開始された後に、**Solution Pad** のプロジェクトを**右クリック*し、 **[デバッグの開始]** を選択します。

![](multi-process-debugging-images/mpd13-xs.png  "Start Debugging Item")

## <a name="creating-a-solution-configuration"></a>ソリューション構成の作成

_ソリューション構成_は、 **[開始]** ボタンを押してデバッグを開始するときに、実行するプロジェクトを Visual Studio for Mac に指示します。 1 つのソリューションに複数のソリューション構成が存在する可能性があります。 そのため、プロジェクトをデバッグするときに実行されるプロジェクトを指定することができます。

Xamarin Studio で新しいソリューション構成を作成するには、次の操作を実行します。

1. Visual Studio for Mac で **[ソリューション オプション]** を開き、 **[実行] > [構成]** の順に選択します。

    ![](multi-process-debugging-images/mpd10-xs.png "Solution Configuration in the Solution Options dialog")

2. **[新規作成]** ボタンをクリックし、新しいソリューション構成の名前を入力し、 **[作成]** をクリックします。 新しいソリューション構成が **[構成]** ウィンドウに表示されます。

    ![](multi-process-debugging-images/mpd11-xs.png  "Naming a new solution configuration")

3. 構成一覧で新しい実行構成を選択します。 **[ソリューション オプション]** ダイアログには、ソリューションの各プロジェクトが表示されます。 デバッグ セッションの開始時に開始する各プロジェクトをオンにします。

    ![](multi-process-debugging-images/mpd12-xs.png "Selecting the project to start")

**MultipleProjects** ソリューション構成が **[デバッグ] ツールバー**に表示されます。開発者はこのツールバーで、2 つのプロジェクトを同時に実行することができます。

## <a name="summary"></a>まとめ

このガイドでは、Visual Studio for Mac で複数のプロセスをデバッグする方法について説明しました。 また、複数のプロセスのデバッグのための IDE のいくつかの変更点と、関連するいくつかの動作についても説明しました。

## <a name="related-links"></a>関連リンク

- [Xamarin Cycle 9 のリリース ノート](https://releases.xamarin.com/stable-release-cycle-9/)
