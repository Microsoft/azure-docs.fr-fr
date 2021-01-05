---
title: Identifier les menaces avancées avec l’analyse du comportement des utilisateurs et des entités (User and Entity Behavior Analytics, UEBA) dans Azure Sentinel | Microsoft Docs
description: Créez des lignes de base comportementales pour des entités (utilisateurs, noms d’hôte, adresses IP), et utilisez-les pour détecter des comportements anormaux et identifier des menaces persistantes avancées (advanced persistent threat, APT) zero-day.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/19/2020
ms.author: yelevin
ms.openlocfilehash: a0a677342534cd92d5530199076e3d2e7084134d
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95494775"
---
# <a name="identify-advanced-threats-with-user-and-entity-behavior-analytics-ueba-in-azure-sentinel"></a>Identifier les menaces avancées avec l’analyse du comportement des utilisateurs et des entités (User and Entity Behavior Analytics, UEBA) dans Azure Sentinel

> [!IMPORTANT]
>
> - Les fonctionnalités UEBA et Entity Pages sont désormais en **disponibilité générale** dans les régions et zones géographiques Azure Sentinel suivantes :
>    - Géographie des États-Unis
>    - Région Europe Ouest
>    - Géographie de l’Australie
>
> - Dans toutes les autres zones géographiques et régions, ces fonctionnalités restent actuellement en **PRÉVERSION**. Consultez l’[**Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure**](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) pour connaître les conditions juridiques supplémentaires s’appliquant aux fonctionnalités Azure sont en version bêta, en préversion ou non encore en disponibilité générale.

## <a name="what-is-user-and-entity-behavior-analytics-ueba"></a>Qu’est-ce que l’analyse du comportement des utilisateurs et des entités ?

### <a name="the-concept"></a>Le concept

L’identification des menaces au sein de votre organisation, ainsi que de leur impact potentiel, qu’il s’agisse d’une entité compromise ou d’une personne malveillante, a toujours été un processus laborieux et chronophage. Le criblage des alertes, la connexion des points et la chasse active des menaces nécessitent du temps et des efforts considérables pour très peu de résultats, avec la possibilité que des menaces sophistiquées échappent tout simplement à la détection. En particulier, des menaces insaisissables, telles que des menaces persistantes zero-day, ciblées et avancées, peuvent être des plus dangereuses pour votre organisation, ce qui rend leur détection d’autant plus critique.

La fonctionnalité d’analyse du comportement des utilisateurs et des entités d’Azure Sentinel élimine la pénibilité des charges de travail de vos analystes et l’incertitude liée aux résultats de leurs efforts. Elle leur apporte également une intelligence hautement fiable et actionnable, qui leur permet de se concentrer sur les tâches d’investigation et de correction.

Azure Sentinel collecte les journaux et les alertes de toutes les sources de données connectées, les analyse, et génère au fil du temps des profils comportementaux de base des entités de votre organisation (utilisateurs, hôtes, adresses IP, applications, etc.), ainsi qu’un horizon de groupe d’homologues. À l’aide de diverses techniques et fonctionnalités d’apprentissage automatique, Sentinel peut ensuite identifier l’activité anormale et vous aider à déterminer si une ressource a été compromise. Outre cela, il peut déterminer la sensibilité relative de ressources particulières, identifier des groupes homologues de ressources et évaluer l’impact potentiel (« rayon d’impact ») de la compromission de toute ressource. Ces informations vous permettent de hiérarchiser efficacement l’investigation et la gestion de l’incident. 

### <a name="architecture-overview"></a>Vue d’ensemble de l’architecture

:::image type="content" source="media/identify-threats-with-entity-behavior-analytics/entity-behavior-analytics-architecture.png" alt-text="Architecture d’analyse du comportement des entités":::

### <a name="security-driven-analytics"></a>Analyse pilotée par la sécurité

Inspiré par le paradigme de Gartner pour les solutions d’analyse du comportement des utilisateurs et des entités, Azure Sentinel propose une approche de l’extérieur vers l’intérieur, basée sur trois images de référence :

- **Cas d’usage :** en hiérarchisant les vecteurs et scénarios d’attaque pertinents basés sur une recherche en matière de sécurité alignée sur l’infrastructure de tactiques, techniques et sous-techniques MITRE ATT&CK, ayant pour effet de classer les diverses entités en victimes, auteurs ou points pivots dans la chaîne de destruction, Azure Sentinel se concentre spécifiquement sur les journaux les plus précieux que chaque source de données peut fournir.

- **Sources de données :** tout en prenant en charge d’abord et avant tout les sources de données Azure, Azure Sentinel sélectionne soigneusement des sources de données tierces afin de fournir des données correspondant à nos scénarios de menaces.

- **Analyse :** à l’aide de différents algorithmes d’apprentissage automatique, Azure Sentinel identifie les activités anormales et présente des preuves de manière claire et concise sous la forme d’enrichissements contextuels, dont certains exemples sont affichés ci-dessous.

    :::image type="content" source="media/identify-threats-with-entity-behavior-analytics/behavior-analytics-top-down.png" alt-text="Approche de l’extérieur vers l’intérieur de l’analyse du comportement":::

Azure Sentinel présente des artefacts qui aident vos analystes de sécurité à mieux comprendre les activités anormales en contexte et par rapport au profil de base de l’utilisateur. Les actions effectuées par un utilisateur (ou un hôte ou une adresse) sont évaluées dans leur contexte. Un résultat « true » indique une anomalie identifiée :
- dans des emplacements géographiques, appareils et environnements ;
- dans des horizons de temps et de fréquence (par rapport à l’historique de l’utilisateur) ;
- par rapport au comportement d’homologues ;
- par rapport au comportement de l’organisation.

    :::image type="content" source="media/identify-threats-with-entity-behavior-analytics/context.png" alt-text="Contexte d’entité":::


### <a name="scoring"></a>Notation

À chaque activité est attribué un « score de priorité d’investigation », qui détermine la probabilité qu’un utilisateur spécifique effectue une activité spécifique, en fonction de l’apprentissage comportemental de l’utilisateur et de ses pairs. Les activités identifiées comme les plus anormales reçoivent les scores les plus élevés (sur une échelle de 0-10).

Pour vois comment cela fonctionne, découvrez comment l’analyse du comportement est utilisée dans [Microsoft Cloud App Security](https://techcommunity.microsoft.com/t5/microsoft-security-and/prioritize-user-investigations-in-cloud-app-security/ba-p/700136).

## <a name="entities-in-azure-sentinel"></a>Entités dans Azure Sentinel

### <a name="entity-identifiers"></a>Identificateurs d’entité

Lorsque des alertes sont envoyées à Azure Sentinel, elles incluent des éléments de données qu’Azure Sentinel identifie et classe comme des entités, tels que des comptes d'utilisateur, des hôtes, des adresses IP, etc. À l’occasion, cette identification peut être un défi si l’alerte ne contient pas suffisamment d’informations sur l’entité.

Par exemple, les comptes d’utilisateurs peuvent être identifiés de plusieurs façons : à l’aide de l’identifiant numérique (GUID) d’un compte Azure AD, ou de sa valeur User Principal Name (UPN), ou encore, à l’aide d’une combinaison de son nom d'utilisateur et de son nom de domaine NT. Des sources de données différentes peuvent identifier le même utilisateur de manières différentes. Par conséquent, chaque fois que cela est possible, Azure Sentinel fusionne ces identificateurs dans une entité unique afin que celle-ci puisse être correctement identifiée.

Il peut toutefois arriver que l’un de vos fournisseurs de ressources crée une alerte dans laquelle une entité n’est pas suffisamment identifiée, par exemple,dans le cas d’un nom d’utilisateur sans le contexte du nom de domaine. Dans un tel cas, l’entité utilisateur ne peut pas être fusionnée avec d’autres instances du même compte utilisateur, qui seraient identifiées comme une entité distincte, et ces deux entités resteraient séparées au lieu d’être unifiées.

Afin de minimiser le risque que cela se produise, vous devez vérifier que tous vos fournisseurs d'alertes identifient correctement les entités dans les alertes qu'ils produisent. En outre, la synchronisation d’entités de compte d’utilisateur avec Azure Active Directory peut créer un répertoire d’unification, qui sera en mesure de fusionner les entités de compte d’utilisateur.

Les types d’entités suivants sont actuellement identifiés dans Azure Sentinel :

- Compte d’utilisateur (compte)
- Host
- Adresse IP (IP)
- Programme malveillant
- Fichier
- Process
- Application Cloud (CloudApplication)
- Nom de domaine (DNS)
- Ressource Azure
- Fichier (FileHash)
- Clé de Registre
- Valeur de Registre
- Groupe de sécurité
- URL
- Appareil IoT
- Mailbox
- Cluster de messagerie
- Message électronique
- E-mail d'envoi

### <a name="entity-pages"></a>Pages d’entité

Lorsque vous rencontrez une entité (les entités sont actuellement limitées à des utilisateurs et des hôtes) dans une recherche, une alerte ou une investigation, vous pouvez sélectionner l’entité et être redirigé vers une **page d’entité**, à savoir une feuille de données remplie d’informations utiles sur cette entité. Les types d’informations que contient cette page incluent des faits de base sur l’entité, une chronologie d’événements notables liés à cette entité, ainsi que des insights sur le comportement de l’entité.
 
Les pages d’entité se composent de trois parties :
- Le volet gauche contient les informations d’identification de l’entité, collectées à partir de sources de données telles qu’Azure Active Directory, Azure Monitor, Azure Security Center et Microsoft Defender.

- Le panneau central affiche une chronologie graphique et textuelle des événements notables liés à l’entité, tels que des alertes, des signets et des activités. Les activités sont des agrégations d’événements notables de Log Analytics. Les requêtes qui détectent ces activités sont développées par des équipes de recherche en matière de sécurité de Microsoft.

- Le panneau de droite présente des insights comportementales sur l’entité. Ces insights permettent d’identifier rapidement des anomalies et autres menaces de sécurité. Les insights sont développés par des équipes de recherche en matière de sécurité Microsoft, et sont basées sur des modèles de détection d’anomalies.

### <a name="the-timeline"></a>La chronologie

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/entity-pages-timeline.png" alt-text="Chronologie des pages d’entité":::

La chronologie est une partie majeure de la contribution de la page d’entité à l’analyse du comportement dans Azure Sentinel. Elle présente l’historique des événements liés à l’entité, qui vous aide à comprendre l’activité de celle-ci dans un délai d’exécution spécifique.

Vous pouvez choisir le **délai d’exécution** parmi plusieurs options prédéfinies (par exemple, *dernières 24 heures*), ou définir un délai d’exécution personnalisé. En outre, vous pouvez définir des filtres qui limitent les informations de la chronologie à des types spécifiques d’événements ou d’alertes.

Les types d’éléments suivants sont inclus dans la chronologie :

- Alertes – toute alerte dans laquelle l’entité est définie en tant qu’**entité mappée**. Notez que, si votre organisation a créé des [alertes personnalisées utilisant des règles d’analyse](./tutorial-detect-threats-custom.md), vous devez vous assurer que le mappage d’entité des règles est correctement effectué.

- Signets – signets incluant l’entité affichée sur la page.

- Activités – agrégation d’événements notables liés à l’entité. 
 
### <a name="entity-insights"></a>Insights sur l’entité
 
Les insights d’entité sont des requêtes définie par les chercheurs en matière de sécurité de Microsoft pour aider vos analystes à investiguer plus efficacement. Les insights sont présentées dans la page de l’entité. Elles fournissent des informations de sécurité précieuses sur les hôtes et les utilisateurs, sous la forme de graphiques et de données tabulaires. Les informations étant disponibles ici, vous n’avez pas à passer par Log Analytics. Les insights contiennent des données concernant les connexions, les ajouts de groupes, les événements anormaux et bien plus. Elles incluent également des algorithmes d’apprentissage automatique avancés pour détecter les comportements anormaux. Les insights sont basées sur les types de données suivants :
- syslog
- SecurityEvent
- Journaux d’audit
- Journaux de connexion
- Activité Office
- Analyse du comportement des utilisateurs et des entités (UEBA) 
 
### <a name="how-to-use-entity-pages"></a>Comment utiliser les pages d’entité

Les pages d’entité sont conçues pour faire partie de plusieurs scénarios d’utilisation. Elles sont accessibles à partir de la gestion des incidents, du graphique d’examen, des signets ou directement à partir de la page de recherche d’entité sous **Analyse du comportement des entités** dans le menu principal d’Azure Sentinel.

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/entity-pages-use-cases.png" alt-text="Cas d’usage de page d’entité":::


## <a name="data-schema"></a>Schéma de données

### <a name="behavior-analytics-table"></a>Tableau d’analyse des comportements

| Champ                     | Description                                                         |
|---------------------------|---------------------------------------------------------------------|
| TenantId                  | Numéro d’identification unique du locataire                                      |
| SourceRecordId            | Numéro d’identification unique de l’événement EBA                                   |
| TimeGenerated             | Horodateur de l’occurrence de l’activité                              |
| TimeProcessed             | Horodateur du traitement de l’activité par le moteur EBA            |
| ActivityType              | Catégorie de haut niveau de l’activité                                 |
| ActionType                | Nom normalisé de l’activité                                     |
| UserName                  | Nom d’utilisateur de l’utilisateur ayant lancé l’activité                    |
| UserPrincipalName         | Nom d’utilisateur complet de l’utilisateur ayant lancé l’activité               |
| EventSource               | Source de données ayant fourni l’événement d’origine                        |
| SourceIPAddress           | Adresse IP à partir de laquelle l’activité a été lancée                        |
| SourceIPLocation          | Pays à partir duquel l’activité a été lancée, enrichie à partir de l’adresse IP |
| SourceDevice              | nom d’hôte de l’appareil ayant lancé l’activité                  |
| DestinationIPAddress      | Adresse IP de la cible de l’activité                            |
| DestinationIPLocation     | Pays de la cible de l’activité, enrichi à partir de l’adresse IP     |
| DestinationDevice         | Nom de l’appareil cible                                           |
| **UsersInsights**         | Enrichissements contextuels des utilisateurs impliqués                            |
| **DevicesInsights**       | Enrichissements contextuels des appareils concernés                          |
| **ActivityInsights**      | Analyse contextuelle de l’activité basée sur notre profilage              |
| **InvestigationPriority** | Score d’anomalie, entre 0 et 10 (0 = bénigne, 10 = très anormale)         |
|

### <a name="querying-behavior-analytics-data"></a>Interrogation des données d’analyse du comportement

Une requête de mots clés [KQL](/azure/data-explorer/kusto/query/) nous permet d’interroger la table d’analyse du comportement.

Par exemple, si vous souhaitez rechercher tous les cas d’un utilisateur n’ayant pas réussi à se connecter à une ressource Azure, où il s’agissait de la première tentative de l’utilisateur de se connecter à partir d’un pays donné, alors que les connexions à partir de ce pays sont rares, même émanant de pairs de l’utilisateur, nous pouvons utiliser la requête suivante :

```Kusto
BehaviorAnalytics
| where ActivityType == "FailedLogOn"
| where FirstTimeUserConnectedFromCountry == True
| where CountryUncommonlyConnectedFromAmongPeers == True
```

### <a name="user-peers-metadata---table-and-notebook"></a>Métadonnées de pairs de l’utilisateur – table et bloc-notes

Les métadonnées de pairs de l’utilisateur fournissent un contexte important pour les détections de menaces, l’investigation sur un incident et la chasse de menace potentielle. Les analystes de la sécurité peuvent observer les activités normales des pairs d’un utilisateur pour déterminer si les activités de l’utilisateur sont inhabituelles par rapport à celles de ses pairs.

Azure Sentinel calcule et classe les pairs d’un utilisateur sur la base de l’appartenance à un groupe de sécurité Azure AD, de la liste de diffusion, etc. de l’utilisateur, et stocke les pairs classés de 1 à 20 dans la table **UserPeerAnalytics**. La capture d’écran ci-dessous montre le schéma de la table UserPeerAnalytics, avec les huit premiers pairs classés de l’utilisateur Kendall Collins. Azure Sentinel utilise l’algorithme *TF-IDF (Term Frequency-Inverse Document Frequency, fréquence de terme-fréquence inverse dans le document)* afin de normaliser la pondération pour le calcul du classement : plus le groupe est petit, plus le poids est élevé. 

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/user-peers-metadata.png" alt-text="Capture d’écran d’une table de métadonnées de pairs d’utilisateur":::

Vous pouvez utiliser le [bloc-notes Jupyter](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/BehaviorAnalytics/UserSecurityMetadata) fourni dans le dépôt GitHub Azure Sentinel pour visualiser les métadonnées de pairs d’utilisateurs. Pour obtenir des instructions détaillées sur l’utilisation du bloc-notes, consultez le bloc-notes [Analyse guidée – Métadonnées de sécurité utilisateur](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/BehaviorAnalytics/UserSecurityMetadata/Guided%20Analysis%20-%20User%20Security%20Metadata.ipynb).

### <a name="permission-analytics---table-and-notebook"></a>Analyse d’autorisation – table et bloc-notes

L’analyse d’autorisation permet de déterminer l’impact potentiel de la compromission d’une ressource d’organisation par un attaquant. Cet impact est également appelé « rayon d’impact » de la ressource. Les analystes de la sécurité peuvent utiliser ces informations pour hiérarchiser les investigations et la gestion des incidents.

Azure Sentinel détermine les droits d’accès direct et transitif aux ressources Azure détenus par un utilisateur donné, en évaluant les abonnements Azure auxquels l’utilisateur peut accéder directement ou via des groupes ou des principaux de service. Ces informations, ainsi que la liste complète d’appartenance de groupe de sécurité Azure AD de l’utilisateur, sont ensuite stockées dans la table **UserAccessAnalytics**. La capture d’écran ci-dessous montre un exemple de ligne dans la table UserAccessAnalytics pour l’utilisateur Alex Johnson. L’**entité source** est le compte d’utilisateur ou de principal de service, et l’**entité cible** est la ressource à laquelle l’entité source a accès. Les valeurs de **niveau d’accès** et de **type d’accès** dépendent du modèle de contrôle d’accès de l’entité cible. Vous pouvez voir qu’Alex dispose d’un accès contributeur à l’abonnement Azure du *Locataire Contoso Hotels*. Le modèle de contrôle d’accès de l’abonnement est RBAC Azure.   

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/user-access-analytics.png" alt-text="Capture d’écran du tableau d’analyse de l’accès utilisateur":::

Vous pouvez utiliser le [bloc-notes Jupyter](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/BehaviorAnalytics/UserSecurityMetadata) (le même que celui mentionné ci-dessus) à partir du dépôt GitHub Azure Sentinel pour visualiser les données d’analyse d’autorisation. Pour obtenir des instructions détaillées sur l’utilisation du bloc-notes, consultez le bloc-notes [Analyse guidée – Métadonnées de sécurité utilisateur](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/BehaviorAnalytics/UserSecurityMetadata/Guided%20Analysis%20-%20User%20Security%20Metadata.ipynb).

### <a name="hunting-queries-and-exploration-queries"></a>Requêtes de chasse et d’exploration

Azure Sentinel offre un ensemble prêt à l’emploi de requêtes de chasse et d’exploration, ainsi qu’un classeur, basés sur la table BehaviorAnalytics. Ces outils présentent des données enrichies, axées sur des cas d’usage spécifiques, qui indiquent un comportement anormal. 

Apprenez-en davantage sur [la chasse et le graphique d’examen](./hunting.md) dans Azure Sentinel.

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a montré les fonctionnalités d’analyse du comportement d’entité d’Azure Sentinel. Pour obtenir des conseils pratiques sur l’implémentation et pour utiliser les insights que vous avez acquises, consultez les articles suivants :

- [Activer l’analyse du comportement d’entité](./enable-entity-behavior-analytics.md) dans Azure Sentinel.
- [Chasse des menaces de sécurité](./hunting.md).