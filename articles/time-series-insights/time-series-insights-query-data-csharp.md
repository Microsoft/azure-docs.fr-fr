---
title: Interroger des données à partir d’un environnement Gen1 à l’aide du code C# – Azure Time Series Insights Gen1 | Microsoft Docs
description: Découvrez comment interroger des données à partir d’un environnement Azure Time Series Insights Gen1 à l’aide d’une application personnalisée écrite en C#.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: 724bc80f8887ff2c47db93ecfc4cd2e20be7e7f8
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020042"
---
# <a name="query-data-from-the-azure-time-series-insights-gen1-environment-using-c-sharp"></a>Interroger des données à partir d’un environnement Azure Time Series Insights Gen1 en utilisant C Sharp

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet exemple de code C# montre comment utiliser les [API de requête Gen1 ](/rest/api/time-series-insights/gen1-query) pour interroger des données à partir d’environnements Azure Time Series Insights Gen1.

> [!TIP]
> Consultez des exemples de code C# Gen1 à l’adresse [https://github.com/Azure-Samples/Azure-Time-Series-Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen1-sample/csharp-tsi-gen1-sample).

## <a name="summary"></a>Résumé

L’exemple de code ci-dessous illustre les fonctionnalités suivantes :

* Comment acquérir un jeton d’accès via Azure Active Directory à l’aide de [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

* Comment passer ce jeton d’accès acquis dans l’en-tête `Authorization` des requêtes ultérieures d’API de requête.

* L’exemple appelle chacune des API de requête Gen1, illustrant la façon dont les requêtes HTTP sont adressées aux API :
  * [API Get Environments](/rest/api/time-series-insights/gen1-query-api#get-environments-api) pour retourner les environnements auxquels l’utilisateur a accès
  * [API Get Environment Availability](/rest/api/time-series-insights/gen1-query-api#get-environment-availability-api)
  * [API Get Environment Metadata](/rest/api/time-series-insights/gen1-query-api#get-environment-metadata-api) pour récupérer les métadonnées d’environnement
  * [API Get Environments Events](/rest/api/time-series-insights/gen1-query-api#get-environment-events-api)
  * [API Get Environment Aggregates](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-api)

* Comment interagir avec les API de requête Gen1 à l’aide de WSS pour envoyer un message aux API :

  * [API Get Environment Events Streamed](/rest/api/time-series-insights/gen1-query-api#get-environment-events-streamed-api)
  * [API Get Environment Aggregates Streamed](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-streamed-api)

## <a name="prerequisites-and-setup"></a>Composants requis et configuration

Effectuez les étapes suivantes avant de compiler et d'exécuter l’exemple de code :

1. [Approvisionnez un environnement Azure Time Series Insights Gen1](./time-series-insights-get-started.md).
1. Configurez votre environnement Azure Time Series Insights pour Azure Active Directory, comme décrit dans [Authentification et autorisation](time-series-insights-authentication-and-authorization.md).
1. Installez les dépendances de projet nécessaires.
1. Modifiez l’exemple de code ci-dessous en remplaçant chaque espace réservé **#DUMMY#** par l’identificateur d’environnement qui convient.
1. Exécutez le code dans Visual Studio.

## <a name="project-dependencies"></a>Dépendances du projet

Il est recommandé d’utiliser la version la plus récente de Visual Studio :

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) - Version 16.4.2+

L’exemple de code comporte deux dépendances obligatoires :

* [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/), package 3.13.9.
* [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json), package 9.0.1.

Téléchargez les packages dans Visual Studio 2019 en sélectionnant l’option **Générer** > **Générer la solution**.

Vous pouvez également ajouter les packages à l’aide de [NuGet 2.12+](https://www.nuget.org/) :

* `dotnet add package Newtonsoft.Json --version 9.0.1`
* `dotnet add package Microsoft.IdentityModel.Clients.ActiveDirectory --version 3.13.9`

## <a name="c-sample-code"></a>Exemple de code C#

Pour accéder à l’exemple de code C#, reportez-vous au dépôt [Azure Time Series Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen1-sample/csharp-tsi-gen1-sample/Program.cs)].

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur l’interrogation, consultez les [informations de référence sur l’API de requête](/rest/api/time-series-insights/gen1-query-api).

* Découvrez comment [connecter une application JavaScript avec le SDK client](https://github.com/microsoft/tsiclient) à Time Series Insights.
Azure-Samples/Azure-Time-Series-Insights/gen1-sample/csharp-tsi-gen1-sample/Program.cs