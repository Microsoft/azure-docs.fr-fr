---
title: Migrer des applications de BizTalk Services vers Azure Logic Apps
description: Procédure de déplacement de vos applications et solutions Microsoft Azure BizTalk Services (MABS) vers Azure Logic Apps
services: logic-apps
ms.suite: integration
author: jonfancey
ms.author: jonfan
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 05/30/2017
ms.openlocfilehash: 6c07ab4b18c017bd29723d2640129b8e67374e3c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023650"
---
# <a name="migrate-your-apps-and-solutions-from-biztalk-services-to-azure-logic-apps"></a>Migrer vos applications et solutions de BizTalk Services vers Azure Logic Apps

Microsoft Azure BizTalk Services (MABS) fait l’objet d’une mise hors service. Pour déplacer vos solutions d’intégration MABS vers[Azure Logic Apps](../logic-apps/logic-apps-overview.md), suivez les instructions de cet article. 

## <a name="introduction"></a>Introduction

BizTalk Services se compose de deux services secondaires :

* Le service Connexions hybrides Microsoft BizTalk Services
* L’intégration de pont IAE et EDI

Le service [Connexions hybrides d’Azure App Service](../app-service/app-service-hybrid-connections.md) remplace le service Connexions hybrides de BizTalk Services. Le service Connexions hybrides Azure est disponible avec Azure App Service via le portail Azure. Ce service fournit un Hybrid Connection Manager afin que vous puissiez gérer les connexions hybrides BizTalk Services existantes, ainsi que les nouvelles connexions hybrides que vous créez dans le portail. 

[Logic Apps](../logic-apps/logic-apps-overview.md) remplace l’intégration de pont IAE et EDI avec les mêmes fonctionnalités dans BizTalk Services et d’autres encore. Ce service offre un flux de travail de consommation à l’échelle du cloud et des fonctionnalités d’orchestration pour que vous puissiez créer rapidement et facilement des solutions d’intégration complexes via un navigateur ou dans Visual Studio.

Ce tableau mappe les fonctionnalités de BizTalk Services à Logic Apps.

| BizTalk Services   | Logic Apps            | Objectif                      |
| ------------------ | --------------------- | ---------------------------- |
| Connecteur          | Connecteur             | Envoyer et recevoir des données   |
| Pont             | Application logique             | Processeur de pipeline           |
| Étape de validation     | Action de validation XML | Valider un document XML par rapport à un schéma | 
| Étape d’enrichissement       | Jetons de données           | Promouvoir les propriétés dans des messages ou pour des décisions de routage |
| Étape de transformation    | Action de transformation      | Convertir les messages XML d’un format vers un autre |
| Étape de décodage       | Action de décodage de fichier plat | Convertir à partir d’un fichier plat au format XML |
| Étape d’encodage       | Action d’encodage de fichier plat | Convertir depuis XML vers un fichier plat |
| Inspecteur de message  | Azure Functions ou API Apps | Exécuter du code personnalisé dans vos intégrations |
| Action de routage       | Condition ou commutateur | Router les messages vers un des connecteurs spécifiés |
|||| 

## <a name="biztalk-services-artifacts"></a>Artefacts BizTalk Services

BizTalk Services comporte plusieurs types d’artefacts. 

## <a name="connectors"></a>Connecteurs

Les connecteurs BizTalk Services permettent aux ponts d’envoyer et de recevoir des données ; cela comprend les ponts bidirectionnels qui activent des interactions de requête/réponse HTTP. Logic Apps utilise la même terminologie et comporte des centaines de connecteurs ayant la même fonction en vous connectant à une large gamme de technologies et services. Par exemple, les connecteurs sont disponibles pour les services cloud SaaS et PaaS, tels que OneDrive, Office 365, Dynamics CRM et bien d’autres encore, ainsi que pour les systèmes locaux via la passerelle de données locale, qui remplace BizTalk Adapter Service pour BizTalk Services. Les sources dans BizTalk Services sont limitées aux abonnements FTP, SFTP et à une rubrique ou une file d’attente Service Bus.

![Diagramme montrant le flux de BizTalk Services.](media/logic-apps-move-from-mabs/sources.png)

Par défaut, chaque pont a un point de terminaison HTTP, qui est configuré avec les propriétés d’adresse d’exécution et d’adresse relative du pont. Pour obtenir les mêmes résultats avec Logic Apps, utilisez les actions de [requête et réponse](../connectors/connectors-native-reqres.md).

## <a name="xml-processing-and-bridges"></a>Traitement XML et ponts

Dans BizTalk Services, un pont est analogue à un pipeline de traitement. Un pont peut prendre les données reçues d’un connecteur, les traiter, puis envoyer les résultats à un autre système. Logic Apps fait de même en prenant en charge les mêmes modèles d’interaction basés sur un pipeline que BizTalk Services en fournissant également plusieurs autres modèles d’intégration. Le [pont requête-réponse XML](/previous-versions/azure/hh689781(v=azure.100)) dans BizTalk Services est appelé pipeline VETER, et comporte des étapes qui effectuent les tâches suivantes :

* (V) Valider
* (E) Enrichir
* (T) Transformer
* (E) Enrichir
* (R) Router

Cette image montre comment le traitement est partagé entre la requête et la réponse, ce qui permet de contrôler séparément les processus de requête et de réponse, par exemple, à l’aide de différents mappages pour chaque chemin d’accès :

![Capture d’écran montrant le fractionnement du traitement entre la requête et la réponse.](media/logic-apps-move-from-mabs/xml-request-reply.png)

De plus, un pont unidirectionnel XML ajoute les étapes de décodage et d’encodage au début et à la fin du traitement. Le pont intermédiaire contient une seule étape d’enrichissement.

### <a name="message-processing-decoding-and-encoding"></a>Traitement, décodage et encodage des messages

Dans BizTalk Services, vous pouvez recevoir différents types de messages XML et déterminer le schéma correspondant pour le message reçu. Cette opération est effectuée à l’étape *Types de messages* du pipeline de traitement de la réception. L’étape de décodage utilise ensuite le type de message détecté pour décoder le message avec le schéma fourni. Si le schéma est un schéma de fichier plat, cette étape convertit le fichier plat entrant au format XML. 

Logic Apps offre des fonctionnalités similaires. Vous recevez un fichier plat par le biais de différents protocoles en utilisant les différents déclencheurs de connecteur (système de fichiers, FTP, HTTP, etc.) et utilisez l’action de [décodage de fichier plat](../logic-apps/logic-apps-enterprise-integration-flatfile.md) pour convertir les données entrantes au format XML. Vous pouvez déplacer vos schémas de fichier plat existants directement vers Logic Apps sans apporter de modifications, puis charger les schémas sur votre compte d’intégration.

### <a name="validation"></a>Validation

Une fois les données entrantes converties au format XML (ou si XML correspondait au format du message reçu), la validation s’exécute pour déterminer si le message est conforme à votre schéma XSD. Pour effectuer cette tâche, dans Logic Apps, utilisez l’action [Validation XML](../logic-apps/logic-apps-enterprise-integration-xml-validation.md). Vous pouvez utiliser les mêmes schémas de BizTalk Services sans aucune modification.

### <a name="transform-messages"></a>Transformer des messages

Dans BizTalk Services, la phase de transformation convertit un format de message basé sur XML en un autre. Cette opération consiste à appliquer un mappage à l’aide du mappeur basé sur une transformation. Dans Logic Apps, le processus est similaire. L’action de transformation exécute un mappage à partir de votre compte d’intégration. La différence principale est que les mappages dans Logic Apps sont au format XSLT. XSLT inclut la possibilité de réutiliser le XSLT existant, y compris les mappages créés pour BizTalk Server qui contiennent des fonctoids. 

### <a name="routing-rules"></a>Règles de routage

BizTalk Services prend une décision de routage quant au point de terminaison ou connecteur pour l’envoi des données ou messages entrants. Vous pouvez sélectionner un point de terminaison préconfiguré grâce à l’option de filtre de routage :

![Capture d’écran montrant l’option de filtre de routage.](media/logic-apps-move-from-mabs/route-filter.png)

Dans BizTalk Services, s’il n’existe que deux options disponibles, la meilleure méthode de conversion des filtres de routage dans BizTalk Services consiste à utiliser une *condition*. S’il en existe plus de deux, utilisez un **commutateur**.

Logic Apps offre des fonctionnalités de logique sophistiquées ainsi qu’un flux de contrôle et un routage avancés avec des [instructions conditionnelles](../logic-apps/logic-apps-control-flow-conditional-statement.md) et des [instructions de commutation](../logic-apps/logic-apps-control-flow-switch-statement.md).

### <a name="enrich"></a>Enrichissement

Dans le traitement BizTalk Services, l’étape d’enrichissement ajoute des propriétés au contexte de message associé aux données reçues. Par exemple, la promotion d’une propriété à utiliser pour le routage à partir d’une recherche de base de données ou en extrayant une valeur à l’aide d’une expression XPath. Logic Apps permet d’accéder à toutes les sorties de données contextuelles des actions précédentes, simplifiant ainsi la réplication d’un même comportement. Par exemple, à l’aide de l’action de connexion SQL `Get Row`, vous retournez les données à partir d’une base de données SQL Server et utilisez les données dans une action de décision pour le routage. De même, les propriétés sur les messages Service Bus entrants mis en file d’attente par un déclencheur sont adressables, ainsi que XPath à l’aide de l’expression de langage de définition de workflow xpath.

### <a name="run-custom-code"></a>Exécuter un code personnalisé

BizTalk Services vous permet d’[exécuter un code personnalisé](/previous-versions/azure/dn232389(v=azure.100)) qui est chargé dans vos propres assemblys. Cette fonction est implémentée par l’interface [IMessageInspector](). Chaque étape du pont inclut deux propriétés (On Enter Inspector et On Exit Inspector) qui fournissent le type .NET créé par vos soins qui implémente cette interface. Le code personnalisé vous permet d’effectuer un traitement plus complexe sur les données et vous permet de réutiliser le code existant dans les assemblys qui effectuent une logique métier courante. 

Logic Apps offre deux méthodes principales pour exécuter du code personnalisé : Azure Functions et API Apps. Les fonctions Azure Functions peuvent être créées et appelées à partir d’applications logiques. Consultez [Ajout et exécution d’un code personnalisé pour des applications logiques avec Azure Functions](../logic-apps/logic-apps-azure-functions.md). Utilisez API Apps, qui fait partie d’Azure App Service, pour créer vos propres déclencheurs et actions. Découvrez-en plus sur la [création d’une API personnalisée à utiliser avec Logic Apps](../logic-apps/logic-apps-create-api-app.md). 

Si vous avez un code personnalisé dans des assemblys que vous appelez depuis BizTalk Services, vous pouvez déplacer ce code vers Azure Functions, ou créer des API personnalisées avec API Apps, suivant ce que vous implémentez. Par exemple, si vous avez du code qui encapsule un autre service pour lequel Logic Apps n’a pas de connecteur, créez une application API et utilisez les actions que votre application API fournit au sein de votre application logique. Si vous avez des bibliothèques ou des fonctions d’assistance, Azure Functions est probablement la meilleure solution.

### <a name="edi-processing-and-trading-partner-management"></a>Traitement EDI et gestion des partenaires commerciaux

BizTalk Services et Logic Apps incluent le traitement EDI et B2B avec prise en charge d’AS2 (Applicability Statement 2), X12 et EDIFACT. Dans BizTalk Services, vous créez des ponts EDI et créez ou gérez des partenaires commerciaux et des contrats dans le portail de suivi et de gestion dédié.
Dans Logic Apps, vous disposez de cette fonctionnalité via [Enterprise Integration Pack (EIP)](../logic-apps/logic-apps-enterprise-integration-overview.md). EIP fournit un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) et des actions B2B pour le traitement EDI et B2B. Vous pouvez également utiliser un compte d’intégration pour créer et gérer des [partenaires commerciaux](../logic-apps/logic-apps-enterprise-integration-partners.md) et des [contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md). Après avoir créé un compte d’intégration, vous pouvez y lier une ou plusieurs applications logiques. Vous pouvez alors utiliser les actions B2B pour accéder aux informations des partenaires commerciaux de votre application logique. Les actions suivantes sont fournies :

* Encodage AS2
* Décodage AS2
* Encodage X12
* Décodage X12
* Encodage EDIFACT
* Décodage EDIFACT

Contrairement à BizTalk Services, ces actions sont dissociées des protocoles de transport. Ainsi, lorsque vous créez vos applications logiques, vous avez plus de flexibilité quant aux connecteurs que vous utilisez pour envoyer et recevoir des données. Par exemple, vous pouvez recevoir les fichiers X12 en tant que pièces jointes de courrier électronique, puis traiter ces fichiers dans une application logique. 

## <a name="manage-and-monitor"></a>Gestion et surveillance

Dans BizTalk Services, un portail dédié fournissait des fonctions de suivi pour surveiller et résoudre les problèmes. Logic Apps offre des fonctionnalités de suivi et de surveillance enrichies pour [surveiller les applications logiques par le biais du portail Azure ](../logic-apps/monitor-logic-apps.md), et inclut une application mobile pour vous permettre de vous tenir informé lorsque vous êtes en déplacement.

## <a name="high-availability"></a>Haute disponibilité

Pour la haute disponibilité dans BizTalk Services, vous pouvez partager la charge de traitement en utilisant plusieurs instances dans une région donnée. Logic Apps fournit la haute disponibilité dans une région sans coût supplémentaire. 

Dans BizTalk Services, la récupération d’urgence hors région d’un traitement B2B requiert un processus de sauvegarde et de restauration. Pour garantir la continuité d’activité, Logic Apps fournit une [fonction de récupération d’urgence](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md) active/passive entre régions, qui vous permet de synchroniser les données B2B entre les comptes d’intégration dans différentes régions.

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce que Logic Apps ?](../logic-apps/logic-apps-overview.md)
* [Créez votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md), ou devenez rapidement opérationnel à l’aide d’un [modèle prédéfini](../logic-apps/logic-apps-create-logic-apps-from-templates.md)  
* [Affichez tous les connecteurs disponibles](../connectors/apis-list.md) que vous pouvez utiliser dans des applications logiques