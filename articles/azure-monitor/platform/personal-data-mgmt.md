---
title: Guide pour les données personnelles stockées dans Azure Log Analytics | Microsoft Docs
description: Cet article explique comment gérer les données personnelles stockées dans Azure Log Analytics et décrit les méthodes permettant de les identifier et les supprimer.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/18/2018
ms.openlocfilehash: 2bb1e667758a1430e34d222b9a5c537381c07624
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505271"
---
# <a name="guidance-for-personal-data-stored-in-log-analytics-and-application-insights"></a>Guide pour les données personnelles stockées dans Log Analytics et Application Insights

Log Analytics est une banque de données pouvant contenir des données personnelles. Application Insights stocke ses données dans une partition Log Analytics. Cet article explique où se trouvent généralement ces données dans Log Analytics et Application Insights, ainsi que les fonctionnalités disponibles pour gérer ces données.

> [!NOTE]
> Dans le cadre de cet article, les _données de journal d’activité_ font référence aux données envoyées à un espace de travail Log Analytics, tandis que les _données d’application_ font référence aux données collectées par Application Insights.

[!INCLUDE [gdpr-dsr-and-stp-note](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="strategy-for-personal-data-handling"></a>Stratégie de gestion des données personnelles

S’il dépend au final de vous et de votre entreprise de déterminer la stratégie de gestion de vos données privées (le cas échéant), voici quelques approches possibles. Elles sont répertoriées par ordre de préférence décroissant d’un point de vue technique :

* Là où c’est possible, arrêtez la collecte des données, masquez, anonymisez ou réduisez les données collectées pour les exclure de ce qui peut être considéré comme « privé ». Il s’agit _de loin_ de la meilleure approche, qui vous évite de devoir créer une stratégie de gestion de données très coûteuse et avec un impact fort.
* Quand ce n’est pas possible, essayez de normaliser les données pour réduire l’impact sur la plateforme de données et les performances. Par exemple, au lieu de journaliser un ID utilisateur explicite, créez des données de recherche qui vont établir une corrélation entre d’une part le nom d’utilisateur et ses informations détaillées et d’autre part un ID interne, qui peuvent ensuite être enregistrés ailleurs. De cette façon, si un de vos utilisateurs vous demande de supprimer ses données personnelles, il est possible que la seule suppression de la ligne dans la table de recherche correspondant à l’utilisateur soit suffisante. 
* Enfin, si des données privées doivent être collectées, créez un processus autour du chemin de l’API de vidage et du chemin de l’API de la requête existante pour répondre à vos obligations liées à l’exportation et à la suppression des données privées associées à un utilisateur. 

## <a name="where-to-look-for-private-data-in-log-analytics"></a>Où rechercher des données privées dans Log Analytics ?

Log Analytics est un store flexible qui, tout en prescrivant un schéma pour vos données, vous permet de remplacer chaque champ par des valeurs personnalisées. En outre, tous les schémas personnalisés peuvent être ingérés. Ainsi, il est impossible de dire exactement où les données privées se trouvent dans votre espace de travail spécifique. Cependant, les emplacements suivants constituent de bons points de départ pour votre inventaire :

### <a name="log-data"></a>Données de journal

* *Adresses IP* : Log Analytics collecte diverses informations sur les adresses IP dans toutes sortes de tables. Par exemple, la requête suivante affiche toutes les tables où des adresses IPv4 ont été collectées au cours des dernières 24 heures :
    ```
    search * 
    | where * matches regex @'\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' //RegEx originally provided on https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp
    | summarize count() by $table
    ```
* *ID d’utilisateur* : les ID d’utilisateur se trouvent dans une grande variété de solutions et de tables. Vous pouvez rechercher un nom d’utilisateur particulier dans tout votre jeu de données avec la commande de recherche :
    ```
    search "[username goes here]"
    ```
  N’oubliez pas de rechercher non seulement les noms d’utilisateur explicites, mais aussi les GUID, qui peuvent permettre de remonter directement à un utilisateur particulier !
* *ID d’appareil* : comme les ID d’utilisateur, les ID d’appareil sont parfois considérés « privés ». Utilisez la même approche que ci-dessus pour les ID d’utilisateur, pour identifier les tables où cela peut poser un problème. 
* *Données personnalisées* : Log Analytics permet la collecte selon différentes méthodes : journaux d’activité personnalisés et champs personnalisés, [l’API du collecteur de données HTTP](./data-collector-api.md) et les données personnalisées collectées dans le cadre de journaux des événements système. Tous ces éléments sont susceptibles de contenir des données privées et ils doivent être examinés pour vérifier si de telles données s’y trouvent.
* *Données capturées par les solutions* : comme le mécanisme des solutions est ouvert, nous vous recommandons d’examiner toutes les tables générées par les solutions pour vérifier leur conformité.

### <a name="application-data"></a>Données d'application

* *Adresses IP* : par défaut, Application Insights définit tous les champs d’adresse IP sur « 0.0.0.0 », mais il est assez courant de remplacer cette valeur par l’adresse IP réelle de l’utilisateur afin de gérer les informations de la session. La requête Analytics ci-dessous permet de rechercher dans la colonne des adresses IP toutes les tables contenant des valeurs autres que « 0.0.0.0 », au cours des 24 heures :
    ```
    search client_IP != "0.0.0.0"
    | where timestamp > ago(1d)
    | summarize numNonObfuscatedIPs_24h = count() by $table
    ```
* *ID d’utilisateur* : par défaut, Application Insights utilise les ID générés de manière aléatoire pour effectuer le suivi des utilisateurs et des sessions. Toutefois, il est courant de remplacer ces champs par un ID plus représentatif de l’application. Par exemple : noms d’utilisateurs, GUID AAD, etc. Ces ID sont souvent considérés comme faisant partie de l’étendue car ce sont des données personnelles et, par conséquent, ils doivent être gérés de façon appropriée. Nous vous recommandons de toujours masquer ou anonymiser ces ID. Ces valeurs sont couramment utilisées dans les champs tels que session_Id, user_Id, user_AuthenticatedId, user_AccountId et customDimensions.
* *Données personnalisées* : Application Insights vous permet d’ajouter un ensemble de dimensions personnalisées à n’importe quel type de données. Ces dimensions peuvent représenter *n’importe quelle* donnée. Utilisez la requête suivante pour identifier toutes les dimensions personnalisées collectées au cours des dernières 24 heures :
    ```
    search * 
    | where isnotempty(customDimensions)
    | where timestamp > ago(1d)
    | project $table, timestamp, name, customDimensions 
    ```
* *Données en mémoire et en transit* : Application Insights effectue le suivi des exceptions, requêtes, appels de dépendance et traces. Les données privées peuvent souvent être collectées au niveau du code et des appels HTTP. Examinez les tables contenant des exceptions, requêtes, dépendances et traces pour identifier ces données. Utilisez si possible des [initialiseurs de télémétrie](../app/api-filtering-sampling.md) afin de brouiller ces données.
* *Captures du débogueur de capture instantanée* : la fonctionnalité [Débogueur de capture instantanée](../app/snapshot-debugger.md) d’Application Insights vous permet de collecter des instantanés de débogage chaque fois qu’une exception est interceptée sur l’instance de production de votre application. Les instantanés exposeront la trace de pile complète conduisant à des exceptions, ainsi que les valeurs des variables locales à chaque étape de la pile. Malheureusement, cette fonctionnalité ne permet pas la suppression sélective de points d’ancrage, ou l’accès par programme aux données de l’instantané. Par conséquent, si le taux de rétention des instantanés par défaut ne répond pas à vos exigences de conformité, il est recommandé de désactiver cette fonctionnalité.

## <a name="how-to-export-and-delete-private-data"></a>Comment exporter et supprimer des données privées

Comme mentionné plus haut dans la section [Stratégie de gestion des données personnelles](#strategy-for-personal-data-handling), il est __fortement__ recommandé (si c’est possible) de restructurer votre stratégie de collecte des données pour éviter la collecte de données privées, de les masquer ou de les anonymiser, ou à défaut d’éviter qu’elles soient considérées comme « privées ». Une gestion des données va avant tout générer des coûts pour vous et votre équipe, pour définir et automatiser une stratégie et créer une interface destinée à vos clients qui leur permette d’interagir avec leurs données, ainsi que des coûts liés à la maintenance. De plus, elle est coûteuse en termes de ressources informatiques pour Log Analytics et Application Insights, et un grand nombre d’appels d’API de requête ou de vidage simultanés peut avoir un impact potentiellement négatif sur toutes les autres interactions avec les fonctionnalités de Log Analytics. Cela étant dit, il existe en effet des scénarios valides où des données privées doivent être collectées. Dans ces cas, les données doivent être gérées comme décrit dans cette section.

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-and-export"></a>Afficher et exporter

Pour les requêtes d’affichage et d’exportation des données, vous devez utiliser l’[API de requête Log Analytics](https://dev.loganalytics.io/) ou l’[API de requête Application Insights](https://dev.applicationinsights.io/quickstart). L’implémentation de la logique pour convertir la forme des données selon un format approprié pour vos utilisateurs dépend de vous. [Azure Functions](https://azure.microsoft.com/services/functions/) est l’endroit idéal pour héberger cette logique.

> [!IMPORTANT]
>  Alors que la grande majorité des opérations de vidage peuvent être effectuées beaucoup plus rapidement que ce que prévoit le contrat SLA, **le contrat SLA formel pour la réalisation des opérations de vidage est défini à 30 jours** en raison de leur impact important sur la plateforme de données utilisée. Ce contrat SLA répond aux exigences du RGPD. Comme il s’agit d’un processus automatisé, il n’est pas possible de demander qu’une opération soit gérée plus rapidement. 

### <a name="delete"></a>Supprimer

> [!WARNING]
> Les suppressions dans Log Analytics sont destructrices et non réversibles ! Soyez très prudents quand vous les réalisez.

Nous avons rendu disponible un chemin d’API *de vidage* dans le cadre d’une gestion de la confidentialité. Ce chemin doit être utilisé avec prudence en raison du risque associé à cette opération, de l’impact potentiel sur les performances, et de la possibilité d’influer sur les agrégations, les mesures et d’autres aspects de vos données Log Analytics. Pour d’autres approches de la gestion des données privées, consultez la section [Stratégie de gestion des données personnelles](#strategy-for-personal-data-handling).

> [!NOTE]
> Une fois l’opération de vidage effectuée, les données ne sont pas accessibles tant que l’[état de l’opération de vidage](https://docs.microsoft.com/rest/api/loganalytics/workspacepurge/getpurgestatus) est *en attente*. 

Le vidage est une opération nécessitant des privilèges élevés, qu’aucune application ni utilisateur dans Azure (y compris même le propriétaire de la ressource) n’a l’autorisation d’exécuter sans qu’un rôle lui soit explicitement accordé dans Azure Resource Manager. Ce rôle est _Videur de données_ et il doit être délégué avec prudence en raison du risque de perte de données. 

> [!IMPORTANT]
> Pour gérer les ressources système, les demandes de vidage sont limitées à 50 demandes par heure. Vous devez regrouper l’exécution des demandes de vidage en envoyant une seule commande dont le prédicat comprend toutes les identités des utilisateurs qui demandent un vidage. Utilisez [l’opérateur in](/azure/kusto/query/inoperator) pour spécifier plusieurs identités. Vous devez exécuter la requête avant d’exécuter la demande de vidage pour vérifier que les résultats sont tels qu’attendus. 



Une fois que le rôle Azure Resource Manager a été affecté, deux nouveaux chemins d’API sont disponibles : 

#### <a name="log-data"></a>Données de journal

* [POST purge](/rest/api/loganalytics/workspacepurge/purge) - prend un objet spécifiant les paramètres des données à supprimer et retourne un GUID de référence 
* GET purge status - l’appel de POST purge retourne un en-tête « x-ms-état-location » qui inclut une URL que vous pouvez appeler pour déterminer l’état de votre API de vidage. Par exemple :

    ```
    x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/Microsoft.OperationalInsights/workspaces/[WorkspaceName]/operations/purge-[PurgeOperationId]?api-version=2015-03-20
    ```

> [!IMPORTANT]
>  Alors que nous nous attendons à ce que la grande majorité des opérations de vidage soient effectuées beaucoup plus rapidement que ce que prévoit notre contrat SLA, en raison de leur impact important sur la plateforme de données utilisée par Log Analytics, **le contrat SLA formel pour la réalisation des opérations de vidage est défini à 30 jours**. 

#### <a name="application-data"></a>Données d'application

* [POST purge](/rest/api/application-insights/components/purge) - prend un objet spécifiant les paramètres des données à supprimer et retourne un GUID de référence
* GET purge status - l’appel de POST purge retourne un en-tête « x-ms-état-location » qui inclut une URL que vous pouvez appeler pour déterminer l’état de votre API de vidage. Par exemple :

   ```
   x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/microsoft.insights/components/[ComponentName]/operations/purge-[PurgeOperationId]?api-version=2015-05-01
   ```

> [!IMPORTANT]
>  Alors que la grande majorité des opérations de vidage peuvent être effectuées beaucoup plus rapidement que ce que prévoit le contrat SLA, en raison de leur impact important sur la plateforme de données utilisée par Application Insights, **le contrat SLA formel pour la réalisation des opérations de vidage est défini à 30 jours**.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur la façon dont les données Log Analytics sont collectées, traitées et sécurisées, consultez [Sécurité des données Log Analytics](./data-security.md).
- Pour plus d’informations sur la façon dont les données Application Insights sont collectées, traitées et sécurisées, consultez [Sécurité des données Application Insights](../app/data-retention-privacy.md).

