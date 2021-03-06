---
title: iOS バックグラウンド処理のテクニック
description: このドキュメントでは、iOS でのさまざまなバックグラウンド処理手法 (バックグラウンドタスク、バックグラウンド転送サービス、バックグラウンドフェッチ、リモート通知) について説明しているガイドにリンクしています。
ms.prod: xamarin
ms.assetid: 011A8D48-1CDC-486A-A2B0-C4946118E7A9
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 03/18/2017
ms.openlocfilehash: a81d6a651ee980b444da19341a89206afa8c4375
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73010814"
---
# <a name="ios-backgrounding-techniques"></a>iOS バックグラウンド処理のテクニック

以下のセクションでは、既存のバックグラウンド処理オプションと共に、次の iOS の機能について説明します。

- [便宜的なバックグラウンドタスク](~/ios/app-fundamentals/backgrounding/ios-backgrounding-techniques/ios-backgrounding-with-tasks.md#background_tasks_in_iOS_7)-デバイスが他の処理のために起動したときに、バックグラウンドタスクを便宜的な単位で実行して、バッテリの寿命を節約します。
- [バックグラウンド転送サービス](~/ios/app-fundamentals/backgrounding/ios-backgrounding-techniques/ios-backgrounding-with-tasks.md#background-transfers)-ネットワークの状態やファイルのサイズに関係なく、確実にファイルをアップロードしてダウンロードします。
- [バックグラウンドフェッチ](~/ios/app-fundamentals/backgrounding/ios-backgrounding-techniques/updating-an-application-in-the-background.md#background_fetch)-システムによって決定された間隔で、バックグラウンドからアプリケーションを更新します。
- [リモート通知](~/ios/app-fundamentals/backgrounding/ios-backgrounding-techniques/updating-an-application-in-the-background.md#remote_notifications)-プッシュ通知を使用して、ユーザーがアプリケーションを開く前にバックグラウンドでコンテンツの更新をトリガーします。また、ユーザーに通知するか、サイレントに更新するかを選択できます。
- **バックグラウンド UI 更新**-ユーザーのアプリケーション UI を準備し、アプリケーションのスナップショットをすべてバックグラウンドから更新します。
