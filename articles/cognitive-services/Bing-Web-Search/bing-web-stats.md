---
title: Ajouter une analytique à l’API Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Bing Statistiques fournit une analytique pour l’API Recherche d’images Bing. L’analytique inclut le volume d’appels, les principales chaînes de requête, la répartition géographique, et bien plus encore.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: scottwhi
ms.custom: seodec2018
ms.openlocfilehash: 02fea02cca5950ef8467377a866e9a765af9e2e1
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349552"
---
# <a name="add-analytics-to-the-bing-search-apis"></a>Ajouter une analytique à l’API Recherche Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Bing Statistiques fournit l’analytique pour les API Recherche Bing. L’analytique inclut le volume d’appels, les principales chaînes de requête, la répartition géographique, et bien plus encore. Vous pouvez activer Statistiques Bing dans le [portail Azure](https://ms.portal.azure.com) en accédant à votre ressource Azure et en cliquant sur **Activer Statistiques Bing**.

> [!IMPORTANT]
> * Statistiques Bing n’est pas disponible avec des ressources sur le niveau tarifaire gratuit `F0`.
> * Vous ne pouvez pas utiliser de données disponibles via le tableau de bord Bing statistiques pour créer des applications en vue de les distribuer à des tiers.
> * L’activation de Statistiques Bing augmente légèrement le prix de votre abonnement. Pour en savoir plus, voir les [tarifs](https://aka.ms/bingstatisticspricing).


L’illustration suivante montre l’analytique disponible pour chaque point de terminaison d’API Recherche Bing.

![Distribution par matrice de prise en charge de point de terminaison](./media/bing-statistics/bing-statistics-matrix.png)

## <a name="access-your-analytics"></a>Accéder à votre analytique

Bing met à jour les données d’analyque toutes les 24 heures et conserve jusqu’à 13 mois d’historique auxquels vous pouvez accéder à partir du [tableau de bord analytique](https://bingapistatistics.com). Assurez-vous que vous êtes connecté à l’aide du compte Microsoft (MSA) que vous avez utilisé pour vous inscrire à Statistiques Bing.

> [!NOTE]  
> * Jusqu’à 24 heures peuvent s’écouler avant que les mesures apparaissent sur tableau de bord. Le tableau de bord affiche la date et l’heure de dernière mise à jour des données.  
> * Les métriques sont disponibles à partir du moment où vous activez le complément Bing Statistiques.

## <a name="filter-the-data"></a>Filtrer les données

Par défaut, les graphiques et diagrammes affichent toutes les données et métriques auxquelles vous avez accès. Vous pouvez filtrer les données affichées dans les graphiques et diagrammes en sélectionnant les ressources, les marchés, les points de terminaison et la période qui vous intéressent. Vous pouvez modifier les filtres suivants :

- **ID de ressource** : ID de ressource unique identifiant votre abonnement Azure. La liste contient plusieurs ID si vous vous abonnez à plusieurs niveaux d’API Recherche Bing. Par défaut, toutes les ressources sont sélectionnées.  
  
- **Marchés** : marchés d’où proviennent les résultats. Par exemple, en-us (anglais, États-Unis). Par défaut, tous les objets sont sélectionnés. Le marché `en-WW` est le marché que Bing utilise si l’appel ne spécifie pas de marché et si Bing est incapable de déterminer le marché de l’utilisateur.  
  
- **Points de terminaison** : points de terminaison de l’API Recherche Bing. La liste contient tous les points de terminaison pour lesquels vous avez un abonnement payant. Par défaut, tous les points de terminaison sont sélectionnés.  

- **Délai d’exécution** : période de rapport. Vous pouvez spécifier les valeurs suivantes :
  - **Tout** : Inclut jusqu’à 13 mois de données  
  - **Dernières 24 heures** : Inclut l’analytique des dernières 24 heures  
  - **Semaine dernière** : Inclut l’analytique des 7 derniers jours  
  - **Mois dernier** : Inclut l’analytique des 30 derniers jours  
  - **Plage de dates personnalisée** : Inclut l’analytique de la plage de dates spécifiée, si elle est disponible  

## <a name="charts-and-graphs"></a>Diagrammes et graphiques

Le tableau de bord affiche les diagrammes et graphiques des métriques disponibles pour le point de terminaison sélectionné. Certaines métriques ne sont pas disponibles pour tous les points de terminaison. Les diagrammes et graphiques pour chaque point de terminaison sont statiques (vous ne pouvez pas choisir de les afficher). Le tableau de bord affiche uniquement les diagrammes et graphiques pour lesquelles il existe des données.

<!--
For example, if you don't include the User-Agent header in your calls, the dashboard will not include device-related graphs.
-->

Les mesures et restrictions de point de terminaison possibles sont les suivantes.

- **Volume d’appels** : indique le nombre d’appels effectués pendant la période de rapport. Si la période de rapport est d’un jour, le graphique montre le nombre d’appels par heure. Dans le cas contraire, le graphique montre le nombre d’appels effectués par jour durant la période du rapport.  
  
  > [!NOTE]
  > Le volume d’appels peut différer du rapport de facturation, qui n’inclut généralement que les appels réussis.

- **Requêtes les plus longues** : montre les principales requêtes et le nombre d’occurrences de chacune d’elles durant la période du rapport. Vous pouvez configurer le nombre de requêtes affiché. Par exemple, vous pouvez afficher les 25, 50 ou 75 principales requêtes. Les requêtes les plus courantes ne sont pas disponibles pour les points de terminaison suivants :  

  - /images/trending
  - /images/details
  - /images/visualsearch
  - /videos/trending
  - /videos/details
  - /news
  - /news/trendingtopics
  - /suggestions  
  
  > [!NOTE]  
  > Certains termes de requête peuvent être supprimés pour retirer des informations confidentielles telles que des e-mails, des numéros de téléphone, des numéros de sécurité sociale, etc...

- **Répartition géographique** : Les marchés d’où proviennent les résultats de la recherche. Par exemple, `en-us` (anglais, États-Unis). Bing utilise le paramètre de requête `mkt` pour déterminer le marché, s’il est spécifié. Autrement, Bing utilise des signaux tels que l’adresse IP de l’appelant pour déterminer le marché.

- **Distribution du code de réponse** : codes d’état HTTP de tous les appels pendant la période de rapport.

- **Distribution d’origine des appels** : types de navigateurs utilisés par les utilisateurs. Par exemple, Microsoft Edge, Chrome, Safari et FireFox. Les appels non établis par un navigateur, par exemple par des robots, Postman ou à l’aide de curl à partir d’une application console, sont regroupés sous Bibliothèques. L’origine est déterminée à l’aide de la valeur d’en-tête User-Agent de la demande. Si la demande n’inclut pas l’en-tête User-Agent, Bing tente de dériver l’origine à partir d’autres signaux.  

- **Distribution de recherche sécurisée** : distribution des valeurs de recherche sécurisée. Par exemple, off, moderate ou strict. Le paramètre de requête `safeSearch` contient la valeur, si celle-ci est spécifiée. Dans le cas contraire, la valeur par défaut de Bing est moderate.  

- **Distribution demandée des réponses** : réponses de l’API Recherche web que vous avez demandées dans le paramètre de requête `responseFilter`.  

- **Distribution retournées des réponses** : réponses que l’API Recherche web a retournées dans la réponse.

- **Distribution du serveur réponse** : serveur d’applications ayant servi vos requêtes d’API. Les valeurs possibles sont Bing.com (pour le trafic provenant d’appareils de bureau et portables) et Bing.com-mobile (pour le trafic provenant d’appareils mobiles). Le serveur est déterminé à l’aide de la valeur d’en-tête User-Agent de la demande. Si la demande n’inclut pas l’en-tête User-Agent, Bing tente de dériver le serveur à partir d’autres signaux.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation des API Recherche Bing](bing-api-comparison.md)
* [Conditions d’utilisation et d’affichage de l’API Recherche Bing](use-display-requirements.md)