---
title: Considérations sur les navigateurs système Xamarin Android (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les considérations à prendre en compte lors de l’utilisation de navigateurs système sur Xamarin Android avec la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 8f755b42249a88a2d37117003d561f79aea6b170
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165989"
---
#  <a name="xamarin-android-system-browser-considerations-for-using-msalnet"></a>Considérations sur les navigateurs système Xamarin Android pour l’utilisation de MSAL.NET

Cet article traite de ce que vous devez prendre en compte lorsque vous utilisez le navigateur système sur Xamarin Android avec la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).

À compter de MSAL.NET 2.4.0 (préversion), MSAL.NET prend en charge des navigateurs autres que Chrome. Il n’est plus nécessaire d’installer Chrome sur l’appareil Android pour l’authentification.

Nous vous recommandons d’utiliser les navigateurs qui prennent en charge les onglets personnalisés. Voici quelques exemples de ces navigateurs :

| Navigateurs prenant en charge les onglets personnalisés | Nom du package |
|------| ------- |
|Chrome | com.android.chrome|
|Microsoft Edge | com.microsoft.emmx|
|Firefox | org.mozilla.firefox|
|Ecosia | com.ecosia.android|
|Kiwi | com.kiwibrowser.browser|
|Brave | com.brave.browser|

Outre l’identification des navigateurs qui offrent une prise en charge des onglets personnalisés, nos tests indiquent que quelques navigateurs qui ne prennent pas en charge les onglets personnalisés fonctionnent également pour l’authentification. Ces navigateurs incluent Opera, Opera Mini, InBrowser et Maxthon. 

## <a name="tested-devices-and-browsers"></a>Appareils et navigateurs testés
Le tableau suivant répertorie les appareils et les navigateurs dont la compatibilité avec l’authentification a été testée.

| Appareil | Browser     |  Résultats  | 
| ------------- |:-------------:|:-----:|
| Huawei/One+ | Chrome\* | Réussite|
| Huawei/One+ | Edge\* | Réussite|
| Huawei/One+ | Firefox\* | Réussite|
| Huawei/One+ | Brave\* | Réussite|
| One+ | Ecosia\* | Réussite|
| One+ | Kiwi\* | Réussite|
| Huawei/One+ | Opera | Réussite|
| Huawei | OperaMini | Réussite|
| Huawei/One+ | InBrowser | Réussite|
| One+ | Maxthon | Réussite|
| Huawei/One+ | DuckDuckGo | Authentification annulée par l’utilisateur|
| Huawei/One+ | UC Browser | Authentification annulée par l’utilisateur|
| One+ | Dolphin | Authentification annulée par l’utilisateur|
| One+ | CM Browser | Authentification annulée par l’utilisateur|
| Huawei/One+ | Aucun installé | Exception AndroidActivityNotFound|

\* prend en charge des onglets personnalisés

## <a name="known-issues"></a>Problèmes connus

Si l’utilisateur ne dispose d’aucun navigateur sur son appareil, MSAL.NET enverra une exception `AndroidActivityNotFound`.  
  - **Atténuation** : Demandez à l’utilisateur d’activer un navigateur sur son appareil. Recommandez un navigateur prenant en charge les onglets personnalisés.

Si l’authentification échoue (par exemple, si l’authentification est lancée avec DuckDuckGo), MSAL.NET retourne `AuthenticationCanceled MsalClientException`. 
  - **Problème racine** : Aucun navigateur prenant en charge les onglets personnalisés n’a été activé sur l’appareil. L’authentification a été lancée avec un navigateur qui n’a pas pu terminer l’authentification. 
  - **Atténuation** : Demandez à l’utilisateur d’activer un navigateur sur son appareil. Recommandez un navigateur prenant en charge les onglets personnalisés.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations et d’exemples de code, consultez les rubriques [Choix entre le navigateur web incorporé et le navigateur système sur Xamarin Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/MSAL.NET-uses-web-browser#choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid) et [Interface utilisateur web incorporée ou système](msal-net-web-browsers.md#embedded-vs-system-web-ui).  