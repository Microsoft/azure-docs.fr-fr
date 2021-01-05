---
title: Vue d’ensemble d’Azure Logic Apps
description: Azure Logic Apps est une solution cloud permettant la création et l’orchestration de workflows automatisés qui intègrent applications, données, services et systèmes, avec un minimum de code pour des scénarios de niveau entreprise.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: overview
ms.custom: mvc
ms.date: 03/11/2020
ms.openlocfilehash: 4c4e8c8f809452286fee493e20dbef2cc538c8bd
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921528"
---
# <a name="what-is-azure-logic-apps"></a>Qu’est-ce qu’Azure Logic Apps ?

[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps) est un service cloud qui vous permet de planifier, d’automatiser et d’orchestrer des tâches, des processus métier et des [workflows](#logic-app-concepts) quand vous avez besoin d’intégrer des applications, des données, des systèmes et des services entre des entreprises ou des organisations. Logic Apps simplifie la conception et la création de solutions scalables pour l’[intégration](https://azure.microsoft.com/product-categories/integration/) d’applications, l’intégration de données, l’intégration de systèmes, l’intégration d’applications d’entreprise (IAE) et la communication interentreprises (B2B), que ce soit dans un environnement cloud, un environnement local, ou les deux.

Par exemple, voici quelques charges de travail que vous pouvez automatiser avec les applications logiques :

* Traiter et router des commandes sur différents systèmes locaux et services cloud.

* Envoyer des notifications par e-mail avec Office 365 lorsque des événements se produisent dans différents systèmes, applications et services.

* Déplacer des fichiers chargés depuis un serveur SFTP ou FTP vers Stockage Azure.

* Surveiller des tweets sur un sujet spécifique, analyser les sentiments et créer des alertes ou des tâches pour les éléments à examiner.

Pour créer des solutions d’intégration d’entreprise avec Azure Logic Apps, vous pouvez faire votre choix dans une galerie grandissante contenant des [centaines de connecteurs faciles à utiliser](../connectors/apis-list.md), qui incluent des services comme Azure Service Bus, Azure Functions, Stockage Azure, SQL Server, Office 365, Dynamics, Salesforce, BizTalk, SAP, Oracle DB, des partages de fichiers et bien plus encore. Les [connecteurs](#logic-app-concepts) fournissent des [déclencheurs](#logic-app-concepts), des [actions](#logic-app-concepts), ou les deux, pour créer des applications logiques qui accèdent et traitent les données de manière sécurisée et en temps réel.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Introducing-Azure-Logic-Apps/player]

## <a name="how-do-logic-apps-work"></a>Comment les applications logiques fonctionnent-elles ? 

Chaque flux de travail d’application logique commence par un déclencheur, qui se déclenche lorsqu’un événement spécifique se produit, ou lorsque de nouvelles données disponibles répondent aux critères spécifiques. Plusieurs déclencheurs fournis par les connecteurs dans Logic Apps incluent des fonctionnalités de planification de base qui vous permettent de configurer la régularité d’exécution des charges de travail. Pour configurer une planification plus complexe ou des fréquences avancées, vous pouvez utiliser un déclencheur de fréquence comme première étape d’un workflow. Découvrez les [workflows basés sur une planification](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une instance d’application logique qui exécute les actions dans le flux de travail. Ces actions peuvent également inclure des conversions de données et des contrôles de workflow, par exemple des instructions conditionnelles, des instructions de basculement, des boucles et des créations de branches. Par exemple, cette application logique démarre par un déclencheur Dynamics 365 avec le critère intégré « lorsqu’un enregistrement est mis à jour ». Si le déclencheur détecte un événement correspondant à ce critère, il est activé et exécute les actions du flux de travail. Ici, ces actions incluent la transformation XML, les mises à jour de données, la création de branches de décision et les notifications par e-mail.

![Concepteur Logic Apps - exemple d’application logique](./media/logic-apps-overview/azure-logic-apps-designer.png)

Vous pouvez créer vos applications logiques visuellement avec le Concepteur d’applications logiques, disponible dans le portail Azure via votre navigateur et dans Visual Studio. Pour disposer d’applications logiques plus personnalisées, vous pouvez créer ou modifier des définitions d’application logique dans JavaScript Objet Notation (JSON) en travaillant dans l’éditeur « code ». Vous pouvez également utiliser des commandes Azure PowerShell et des modèles Azure Resource Manager pour sélectionner des tâches. Les applications logiques se déploient et s’exécutent dans le cloud sur Azure. Pour avoir une présentation plus complète, regardez cette vidéo : [Utiliser Azure Enterprise Integration Services pour exécuter des applications cloud à grande échelle](https://channel9.msdn.com/Events/Connect/2017/T119/)

## <a name="why-use-logic-apps"></a>Pourquoi utiliser des applications logiques ?

Avec des entreprises de plus en plus portées sur la numérisation, les applications logiques vous permettent de connecter facilement et rapidement des systèmes hérités, modernes et innovants, en fournissant des API prédéfinies en tant que connecteurs gérés par Microsoft. De cette façon, vous pouvez vous concentrer sur la logique métier et les fonctionnalités de vos applications. Vous n’avez pas à vous soucier de la création, de l’hébergement, de la mise à l’échelle, de la gestion, de la maintenance et de la surveillance de vos applications. Logic Apps gère ces problèmes pour vous. De plus, vous payez uniquement ce que vous utilisez, avec un [modèle de tarification](../logic-apps/logic-apps-pricing.md) basé sur la consommation.

Dans de nombreux cas, vous n’avez pas à écrire du code. Mais si vous devez en écrire, vous pouvez créer des extraits de code avec [Azure Functions](../azure-functions/functions-overview.md) et l’exécuter à la demande à partir d’applications logiques. En outre, si votre application logique a besoin d’interagir avec des événements à partir de services Azure, d’applications personnalisées ou d’autres solutions, vous pouvez utiliser [Azure Event Grid](../event-grid/overview.md) avec vos applications logiques pour la surveillance, le routage et la publication d’événements.

Logic Apps, Functions et Event Grid sont entièrement gérés par Microsoft Azure, vous évitant ainsi d’avoir à vous soucier de la compilation, de l’hébergement, de la mise à l’échelle, de la gestion, de la surveillance et du maintien de vos solutions. Avec la possibilité de créer des [applications et des solutions « sans serveur »](../logic-apps/logic-apps-serverless-overview.md), vous pouvez vous concentrer uniquement sur la logique métier. Ces services sont automatiquement redimensionnés pour répondre à vos besoins, rendre les intégrations plus rapides et vous aider à créer des applications cloud fiables avec un minimum de code.

Pour savoir comment les sociétés ont amélioré leur flexibilité et la concentration sur leurs activités principales grâce à la combinaison de Logic Apps avec d’autres services Azure et des produits Microsoft, consultez ces [témoignages](https://aka.ms/logic-apps-customer-stories).

Voici de plus amples détails sur les fonctionnalités et les avantages de Logic Apps :

### <a name="visually-build-workflows-with-easy-to-use-tools"></a>Concevoir visuellement des flux de travail avec des outils simples à utiliser

Gagnez du temps et simplifiez les processus complexes avec des outils de conception visuelle. Créez des applications logiques complètes à l’aide du Concepteur d’applications logiques via votre navigateur dans le portail Azure ou dans Visual Studio. Démarrez votre flux de travail à l’aide d’un déclencheur et ajoutez des actions de la [galerie des connecteurs](../connectors/apis-list.md).

### <a name="get-started-faster-with-logic-app-templates"></a>Prise en main plus rapide à l’aide des modèles d’application logique

Créez rapidement des solutions courantes plus rapidement lorsque vous choisissez des flux de travail prédéfinis dans la [galerie de modèles](../logic-apps/logic-apps-create-logic-apps-from-templates.md). Les modèles vont de la connectivité simple pour les applications software-as-a-service (SaaS) aux solutions avancées B2B avec des modèles « pour vous amuser ». Apprenez à [créer des applications logiques à partir de modèles prédéfinis](../logic-apps/logic-apps-create-logic-apps-from-templates.md).

### <a name="connect-disparate-systems-across-different-environments"></a>Connecter des systèmes disparates au sein d’environnements différents

Certains modèles et flux de travail sont faciles à décrire, mais difficiles à implémenter dans le code. Les applications logiques vous permettent de connecter en toute transparence différents systèmes entre des environnements cloud et locaux. Par exemple, vous pouvez connecter une solution marketing cloud à un système de facturation local, ou centraliser la messagerie entre les API et les systèmes avec un Service Bus d’entreprise. Les applications logiques offrent un moyen rapide, fiable et cohérent de fournir des solutions réutilisables et reconfigurables pour ces scénarios.

### <a name="first-class-support-for-enterprise-integration-and-b2b-scenarios"></a>Prise en charge de premier ordre de l’intégration en entreprise et des scénarios B2B

Les entreprises et les organisations communiquent électroniquement entre elles à l’aide des protocoles et de formats de message standard mais différents, tels que X12, AS2 et EDIFACT. Avec les fonctionnalités du pack [EIP (Enterprise Library Integration Pack)](../logic-apps/logic-apps-enterprise-integration-overview.md), vous pouvez créer des applications logiques qui convertissent les formats de message utilisés par vos partenaires en formats interprétables et utilisables par les systèmes de votre organisation. Logic Apps gère ces échanges en douceur et en sécurité avec le chiffrement et les signatures numériques.

Commencez par vos systèmes et services actuels et augmentez de façon incrémentielle à votre propre rythme. Lorsque vous êtes prêt, Logic Apps et le EIP vous aident à mettre en œuvre et à monter en puissance vers des scénarios d’intégration plus matures, grâce à ces fonctionnalités et bien plus encore :

* Profiter de ces produits et services :

  * [Microsoft BizTalk Server](/biztalk/core/introducing-biztalk-server)
  * [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)
  * [Azure Functions](../azure-functions/functions-overview.md)
  * [Gestion des API Azure](../api-management/api-management-key-concepts.md)

* Traiter les [messages XML](../logic-apps/logic-apps-enterprise-integration-xml.md)

* Gérer les [fichiers plats](../logic-apps/logic-apps-enterprise-integration-flatfile.md)

* Échanger des messages avec les protocoles [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md), [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md), et [X12](../logic-apps/logic-apps-enterprise-integration-x12.md)

* Stockez et gérez ces artefacts B2B et plus à un seul emplacement avec les [comptes d’intégration](./logic-apps-enterprise-integration-create-integration-account.md) :

  * [Partenaires](../logic-apps/logic-apps-enterprise-integration-partners.md)
  * [Contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md) 
  * [Mappages de transformation XML](../logic-apps/logic-apps-enterprise-integration-maps.md)
  * [Schémas de validation XML](../logic-apps/logic-apps-enterprise-integration-schemas.md)

Par exemple, si vous utilisez Microsoft BizTalk Server, les applications logiques peuvent communiquer avec votre instance BizTalk Server en utilisant le [connecteur BizTalk Server](../connectors/apis-list.md#on-premises-connectors). Vous pouvez alors étendre ou effectuer des opérations BizTalk dans vos applications logiques en incluant des [connecteurs de compte d’intégration](../connectors/apis-list.md#integration-account-connectors), qui sont disponibles avec le pack EIP (Enterprise Integration Pack).

Dans l’autre sens, BizTalk Server peut se connecter et communiquer avec les applications logiques en utilisant [Microsoft BizTalk Server Adapter pour Logic Apps](https://www.microsoft.com/download/details.aspx?id=54287). Découvrez comment [configurer et utiliser BizTalk Server Adapter](/biztalk/core/logic-app-adapter) dans votre instance BizTalk Server.

### <a name="write-once-reuse-often"></a>Écrire une seule fois pour réutiliser souvent

Créez vos applications logiques en tant que modèles Azure Resource Manager pour pouvoir [automatiser le déploiement d’applications logiques](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) dans plusieurs environnements et régions.

### <a name="access-resources-inside-azure-virtual-networks"></a>Accéder aux ressources situées dans les réseaux virtuels Azure

Logic Apps peut accéder à des ressources sécurisées, par exemple des machines virtuelles et d’autres systèmes ou services, dans un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) quand vous créez un [*environnement ISE* (environnement de service d’intégration)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). Un environnement ISE est une instance dédiée du service Logic Apps qui utilise des ressources dédiées et s’exécute séparément du service Logic Apps multilocataire « mondial ».

L’exécution d’applications logiques dans votre propre instance dédiée distincte aide à réduire l’impact que d’autres locataires Azure peuvent avoir sur le niveau de performance de vos applications. Cet impact est également appelé [effet « voisins bruyants »](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors). Un ISE offre également les avantages suivants :

* Vous avez vos propres adresses IP statiques, qui sont distinctes des adresses IP statiques partagées par les applications logiques dans le service multilocataire. Vous pouvez également configurer une adresse IP sortante publique, statique et prévisible pour communiquer avec les systèmes de destination. De cette façon, vous n’avez pas besoin de configurer d’ouvertures de pare-feu supplémentaires sur ces systèmes de destination pour chaque environnement ISE.

* Des limites accrues quant à la durée d’exécution, la conservation du stockage, le débit, le délai d’attente des requêtes et réponses HTTP, la taille des messages et les requêtes de connecteur personnalisé. Pour plus d’informations, consultez [Limites et configuration pour Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md).

Quand vous créez un ISE, Azure l’*injecte* ou le déploie sur votre réseau virtuel Azure. Vous pouvez ensuite utiliser cet ISE comme emplacement pour les applications logiques et les comptes d’intégration qui ont besoin d’un accès. Pour plus d’informations sur la création d’un environnement ISE, consultez [Se connecter à des réseaux virtuels Azure à partir d’Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

### <a name="built-in-extensibility"></a>Extensibilité intégrée

Si vous ne trouvez pas le connecteur souhaité pour exécuter du code personnalisé, vous pouvez étendre les applications logiques en créant et en appelant vos propres extraits de code à la demande via [Azure Functions](../azure-functions/functions-overview.md). Créez vos propres [API](../logic-apps/logic-apps-create-api-app.md) et [connecteurs personnalisés](../logic-apps/custom-connector-overview.md), que vous pouvez appeler à partir d’applications logiques.

### <a name="pay-only-for-what-you-use"></a>Payez uniquement pour ce que vous utilisez
  
Logic Apps utilise un [modèle tarifaire](../logic-apps/logic-apps-pricing.md) basé sur la consommation, sauf si vous avez déjà des applications logiques créées avec des plans App Service.

En savoir plus sur Logic Apps avec ces vidéos d’introduction :

* [Intégration avec Logic Apps - passer de zéro à héros](https://channel9.msdn.com/Events/Build/2017/C9R17)
* [Intégration d’entreprise avec Microsoft Azure Logic Apps](https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK2188)
* [Concevoir des processus d’entreprise avancés avec Logic Apps](https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3179)

## <a name="how-does-logic-apps-differ-from-functions-webjobs-and-power-automate"></a>En quoi Logic Apps diffère de Functions, WebJobs et Power Automate ?

Tous ces services vous aident à « coller » et connecter des systèmes hétérogènes. Chaque service ayant des avantages, combiner leurs fonctionnalités est la meilleure façon de créer rapidement un système d’intégration évolutif et complet. Pour plus d’informations, consultez [Choisir entre Logic Apps, Functions, WebJobs et Power Automate](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md).

<a name="logic-app-concepts"></a>

## <a name="key-terms"></a>Termes clés

* **Workflow** : visualiser, concevoir, générer, automatiser et déployer des processus métier comme une série d’étapes.

* **Connecteurs managés** : vos applications logiques ont besoin d’un accès aux données, services et systèmes. Vous pouvez utiliser des connecteurs gérés prégénérés par Microsoft et qui sont conçus pour se connecter, accéder et travailler avec vos données. Consultez [Connecteurs pour Azure Logic Apps](../connectors/apis-list.md).

* **Déclencheurs** : beaucoup de connecteurs managés par Microsoft fournissent des déclencheurs qui s’activent lorsque des événements ou de nouvelles données répondent aux conditions spécifiées. Par exemple, un événement peut être la réception d’un e-mail ou la détection de modifications dans votre compte de Stockage Azure. Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une nouvelle instance d’application logique qui exécute le flux de travail.

* **Actions** : Les actions représentent toutes les étapes qui se produisent après le déclencheur. Chaque action correspond généralement à une opération définie par un connecteur géré, une API personnalisée ou un connecteur personnalisé.

* **Enterprise Integration Pack** : pour les scénarios d’intégration plus avancés, Logic Apps intègre des fonctionnalités de BizTalk Server. Enterprise Integration Pack fournit des connecteurs permettant aux applications logiques d’effectuer facilement une validation, une transformation, et bien plus encore.

## <a name="get-started"></a>Bien démarrer

Logic Apps est l’un des nombreux services hébergés sur Microsoft Azure. Avant de commencer, vous avez besoin d’un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

Si vous avez un abonnement Azure, essayez ce [guide de démarrage rapide pour créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md), qui surveille le nouveau contenu sur un site web via un flux RSS et envoie un e-mail lorsqu’un nouveau contenu apparaît.

## <a name="next-steps"></a>Étapes suivantes

* [Contrôler le trafic à l’aide d’une application logique planifiée](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)
* En savoir plus sur [les solutions serverless avec Azure](../logic-apps/logic-apps-serverless-overview.md)
* En savoir plus sur [l’intégration B2B avec Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
