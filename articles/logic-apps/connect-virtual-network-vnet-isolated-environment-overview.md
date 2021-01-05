---
title: Accéder aux réseaux virtuels Azure
description: Découvrez la manière dont les environnements de service d’intégration (ISE) aident les applications logiques à accéder aux réseaux virtuels Azure
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 11/12/2020
ms.openlocfilehash: 19c9ec39d85bfc56b118498aba62c3752d6d771c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996319"
---
# <a name="access-to-azure-virtual-network-resources-from-azure-logic-apps-by-using-integration-service-environments-ises"></a>Accéder aux ressources Réseau virtuel Microsoft Azure à partir d’Azure Logic Apps à l’aide d’environnements de service d’intégration (ISE)

Vos applications logiques doivent dans certains cas accéder à des ressources sécurisées, comme des machines virtuelles et d’autres systèmes ou services, qui se trouvent à l’intérieur d’un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) ou y sont connectées. Pour configurer cet accès, vous pouvez [créer un *environnement de service d’intégration* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md). Un environnement ISE est une instance du service Logic Apps qui utilise des ressources dédiées et s’exécute séparément du service Logic Apps multilocataire « mondial ».

Par exemple, certains réseaux virtuels Azure utilisent des points de terminaison privés (configurables avec [Azure Private Link](../private-link/private-link-overview.md)) pour fournir un accès aux services PaaS Azure, comme le Stockage Azure, Azure Cosmos DB ou Azure SQL Database, aux services partenaires ou aux services clients hébergés sur Azure. Si vos applications logiques ont besoin d’accéder à des réseaux virtuels qui utilisent des points de terminaison privés, vous devez créer, déployer et exécuter ces applications logiques à l’intérieur d’un environnement ISE.

Quand vous créez un ISE, Azure l’*injecte* ou le déploie sur votre réseau virtuel Azure. Vous pouvez ensuite utiliser cet ISE comme emplacement pour les applications logiques et les comptes d’intégration qui ont besoin d’un accès.

![Sélection d’un environnement de service d’intégration](./media/connect-virtual-network-vnet-isolated-environment-overview/select-logic-app-integration-service-environment.png)

Cette vue d’ensemble fournit des informations complémentaires sur les [bonnes raisons d’utiliser un environnement ISE](#benefits), les [différences entre le service dédié et le service Logic Apps multilocataire](#difference) et la procédure permettant d’accéder directement aux ressources qui se trouvent dans votre réseau virtuel Azure ou y sont connectées.

<a name="benefits"></a>

## <a name="why-use-an-ise"></a>Pourquoi utiliser un environnement ISE ?

L’exécution d’applications logiques dans votre propre instance dédiée distincte aide à réduire l’impact que d’autres locataires Azure peuvent avoir sur le niveau de performance de vos applications. Cet impact est également appelé [effet « voisins bruyants »](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors). Un ISE offre également les avantages suivants :

* Accès direct aux ressources qui se trouve à l’intérieur de votre réseau virtuel ou y sont connectées

  Les applications logiques créées et exécutées dans un environnement ISE peuvent utiliser des [connecteurs spécifiquement conçus qui s’exécutent dans votre environnement ISE](../connectors/apis-list.md#ise-connectors). S’il existe un connecteur ISE pour une source de données ou un système local, vous pouvez vous connecter directement sans avoir à utiliser la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md). Pour plus d’informations, consultez [Dédié ou multilocataire](#difference) et [Accès aux systèmes locaux](#on-premises) plus loin dans cette rubrique.

* Accès continu aux ressources qui se trouvent en dehors de votre réseau virtuel ou n’y sont pas connectées

  Les applications logiques créées et exécutées dans un environnement ISE peuvent toujours utiliser des connecteurs qui s’exécutent dans le service Logic Apps multilocataire lorsque aucun connecteur propre à l’environnement ISE n’est disponible. Pour plus d’informations, consultez [Dédié ou multilocataire](#difference).

* Vous avez vos propres adresses IP statiques, qui sont distinctes des adresses IP statiques partagées par les applications logiques dans le service multilocataire. Vous pouvez également configurer une adresse IP sortante publique, statique et prévisible pour communiquer avec les systèmes de destination. De cette façon, vous n’avez pas besoin de configurer d’ouvertures de pare-feu supplémentaires sur ces systèmes de destination pour chaque environnement ISE.

* Des limites accrues quant à la durée d’exécution, la conservation du stockage, le débit, le délai d’attente des requêtes et réponses HTTP, la taille des messages et les requêtes de connecteur personnalisé. Pour plus d’informations, consultez [Limites et configuration pour Azure Logic Apps](logic-apps-limits-and-config.md).

<a name="difference"></a>

## <a name="dedicated-versus-multi-tenant"></a>Dédié ou multilocataire

Quand vous créez et exécutez des applications logiques dans un ISE, vous bénéficiez des mêmes expériences utilisateur et de fonctionnalités similaires au service Logic Apps multilocataire. Vous pouvez utiliser les mêmes actions, déclencheurs intégrés et connecteurs managés que ceux disponibles dans le service Logic Apps multilocataire. Certains connecteurs managés offrent des versions d’ISE supplémentaires. La différence entre les connecteurs ISE et les connecteurs non-IS réside dans l’emplacement où ils s’exécutent et les étiquettes qu’elles ont dans le concepteur d’applications logiques quand vous travaillez dans un ISE.

![Connecteurs avec et sans étiquettes dans un ISE](./media/connect-virtual-network-vnet-isolated-environment-overview/labeled-trigger-actions-integration-service-environment.png)

* Les déclencheurs et les actions intégrés (par exemple, HTTP) présentent l’étiquette **CORE** et s’exécutent dans le même environnement ISE que vos applications logiques.

* Les connecteurs managés qui comportent l’étiquette **ISE** sont spécialement conçus pour les environnements ISE et *s’exécutent toujours dans le même environnement ISE que votre application logique*. Voici par exemple quelques [connecteurs qui proposent des versions d’environnements ISE](../connectors/apis-list.md#ise-connectors) :<p>

  * Stockage Fichier, stockage Table et stockage Blob Azure
  * Azure Service Bus, Files d’attente Azure, Azure Event Hubs
  * Azure Automation, Azure Key Vault, Azure Event Grid et journaux Azure Monitor
  * FTP, SFTP-SSH, File System et SMTP
  * SAP, IBM MQ, IBM DB2 et IBM 3270
  * SQL Server, Azure Synapse Analytics, Azure Cosmos DB
  * AS2, X12 et EDIFACT

  Sauf rares exceptions, si un connecteur ISE est disponible pour une source de données ou un système local, vous pouvez vous connecter directement sans utiliser la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md). Pour plus d’informations, consultez [Accès aux systèmes locaux](#on-premises) plus loin dans cette rubrique.

* Les connecteurs managés qui ne présentent pas l’étiquette **ISE** continuent de fonctionner pour les applications logiques situées à l’intérieur d’un environnement ISE. Ces connecteurs *s’exécutent toujours dans le service Logic Apps multilocataire*, et non dans l’environnement ISE.

* Que la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md) soit ou non requise, les connecteurs personnalisés créés *en dehors d’un environnement ISE*, continuent de fonctionner pour les applications logiques situées à l’intérieur d’un environnement ISE. Cependant, les connecteurs personnalisés créés *au sein d’un environnement ISE* ne fonctionnent pas avec la passerelle de données locale. Pour plus d’informations, consultez [Accès aux systèmes locaux](#on-premises).

<a name="on-premises"></a>

## <a name="access-to-on-premises-systems"></a>Accès aux systèmes locaux

Les applications logiques qui s’exécutent dans un environnement ISE peuvent accéder directement aux sources de données et systèmes locaux qui se trouvent à l’intérieur d’un réseau virtuel Azure ou y sont connectés à l’aide des éléments suivants :<p>

* Le déclencheur ou l’action HTTP, qui présente l’étiquette **CORE**

* Le connecteur **ISE**, s’il est disponible, pour une source de données ou un système local

  Si un connecteur ISE est disponible, vous pouvez accéder directement au système ou à la source de données sans la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md). Si toutefois vous devez accéder à SQL Server à partir d’un environnement ISE et utiliser l’Authentification Windows, vous devez opter pour la version hors environnement ISE du connecteur et la passerelle de données locale. La version ISE du connecteur ne prend pas en charge l’Authentification Windows. Pour plus d’informations, consultez [Connecteurs ISE](../connectors/apis-list.md#ise-connectors) et [Connexion à partir d’un environnement de service d’intégration](../connectors/apis-list.md#integration-service-environment).

* Un connecteur personnalisé

  * Que la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md) soit ou non requise, les connecteurs personnalisés créés *en dehors d’un environnement ISE*, continuent de fonctionner pour les applications logiques situées à l’intérieur d’un environnement ISE.

  * Les connecteurs personnalisés créés *au sein d’un environnement ISE* ne fonctionnent pas avec la passerelle de données locale. Cependant, ils peuvent accéder directement aux sources de données et systèmes locaux qui se trouvent à l’intérieur du réseau virtuel qui héberge votre environnement ISE ou y sont connectés. Ainsi, les applications logiques qui se trouvent à l’intérieur d’un environnement ISE n’ont généralement pas besoin de la passerelle de données pour accéder à ces ressources.

Pour accéder à des sources de données et systèmes locaux qui ne possèdent pas de connecteurs ISE, se trouvent en dehors de votre réseau virtuel ou n’y sont pas connectés, vous devez toujours utiliser la passerelle de données locale. Les applications logiques qui se trouvent au sein d’un environnement ISE peuvent continuer à utiliser les connecteurs dépourvus de l’étiquette **CORE** ou **ISE**. Ces connecteurs s’exécutent dans le service Logic Apps multilocataire, plutôt que dans votre environnement ISE. 

<a name="ise-level"></a>

## <a name="ise-skus"></a>Références (SKU) de l’environnement de service d’intégration

Lorsque vous créez votre environnement ISE, vous pouvez sélectionner la référence SKU de développeur ou la référence SKU Premium. Voici les différences entre ces SKU :

* **Développeur**

  Fournit un environnement ISE à moindre coût que vous pouvez utiliser pour l’expérimentation, le développement et les tests, mais pas pour les tests de production ou de performances. La référence SKU Développeur comprend des déclencheurs et des actions intégrés, des connecteurs standard, des connecteurs d'entreprise et un seul compte d’intégration de [niveau gratuit](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits) pour un tarif mensuel fixe. Toutefois, cette référence (SKU) n’inclut aucun contrat de niveau de service (SLA), d’options de mise à l’échelle de la capacité ou de redondance pendant le recyclage, ce qui signifie que vous pouvez rencontrer des retards ou des temps d’arrêt.

* **Premium**

  Fournit un environnement ISE que vous pouvez utiliser pour la production et inclut la prise en charge des contrats SLA, les déclencheurs et les actions intégrés, les connecteurs standard, les connecteurs d’entreprise, un seul compte d’intégration de [niveau standard](../logic-apps/logic-apps-limits-and-config.md#artifact-number-limits), des options de mise à l’échelle de la capacité et une redondance pendant le recyclage pour un tarif mensuel fixe.

> [!IMPORTANT]
> L’option SKU est disponible uniquement lors de la création de votre ISE et ne peut pas être modifiée ultérieurement.

Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/). Pour plus d’informations sur la tarification et la facturation des environnements de service d’intégration, consultez le [modèle de tarif pour Logic Apps](../logic-apps/logic-apps-pricing.md#fixed-pricing).

<a name="endpoint-access"></a>

## <a name="ise-endpoint-access"></a>Point de terminaison d’accès ISE

Lorsque vous créez votre ISE, vous pouvez choisir d’utiliser des points de terminaison d’accès internes ou externes. Votre sélection détermine si les déclencheurs de demande ou de webhook sur les applications logiques dans votre ISE peuvent recevoir des appels en dehors de votre réseau virtuel. Ces points de terminaison influencent également la manière dont vous pouvez accéder aux entrées et aux sorties à partir de l’historique des exécutions de vos applications logiques.

> [!IMPORTANT]
> Vous pouvez sélectionner le point de terminaison d’accès uniquement lors de la création de l’ISE et ne pouvez pas modifier cette option ultérieurement.

* **Interne** : Les points de terminaison privés autorisent les appels aux applications logiques dans votre ISE, où vous pouvez voir les entrées et sorties dans l’historique des exécutions des applications logiques *uniquement depuis l’intérieur de votre réseau virtuel*.

  > [!IMPORTANT]
  > Si vous devez utiliser ces déclencheurs basés sur un webhook, utilisez des points de terminaison externes, et *non* des points de terminaison internes, lorsque vous créez votre ISE :
  > 
  > * Azure DevOps
  > * Azure Event Grid
  > * Common Data Service
  > * Office 365
  > * SAP (version ISE)
  > 
  > Vérifiez également que vous disposez d’une connectivité réseau entre les points de terminaison privés et l’ordinateur à partir duquel vous souhaitez accéder à l’historique des exécutions. Dans le cas contraire, lorsque vous essayez d’afficher l’historique des exécutions de votre application logique, un message d’erreur indiquant « Erreur inattendue. Échec de récupération » s’affiche.
  >
  > ![Erreur d’action Stockage Azure résultant de l’impossibilité d’envoyer du trafic via le pare-feu](./media/connect-virtual-network-vnet-isolated-environment-overview/integration-service-environment-error.png)
  >
  > Par exemple, votre ordinateur client peut exister à l’intérieur du réseau virtuel de l’environnement de service d’intégration ou à l’intérieur d’un réseau virtuel connecté au réseau virtuel de l’environnement de service d’intégration, par le biais d’un peering ou d’un réseau privé virtuel. 

* **Externe** : Les points de terminaison publics autorisent les appels à des applications logiques dans votre ISE, où vous pouvez voir les entrées et sorties de l’historique des exécutions des applications logiques *depuis l’extérieur de votre réseau virtuel*. Si vous utilisez des groupes de sécurité réseau (NSG), assurez-vous qu’ils sont configurés avec des règles de trafic entrant pour autoriser l’accès aux entrées et sorties de l’historique des exécutions. Pour plus d'informations, consultez [Activer l’accès à ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access).

Pour déterminer si votre environnement ISE utilise un point de terminaison d’accès interne ou externe, dans le menu de votre environnement ISE, sous **Paramètres**, sélectionnez **Propriétés** et recherchez la propriété **Point de terminaison d’accès** :

![Trouver le point de terminaison de l’accès ISE](./media/connect-virtual-network-vnet-isolated-environment-overview/find-ise-access-endpoint.png)

<a name="pricing-model"></a>

## <a name="pricing-model"></a>Modèle de tarification

Les applications logiques, les déclencheurs et actions intégrés et les connecteurs qui s’exécutent dans votre environnement ISE suivent un plan tarifaire fixe différent de celui qui dépend de la consommation. Pour plus d’informations, consultez [Modèle de tarif pour Azure Logic Apps](../logic-apps/logic-apps-pricing.md#fixed-pricing). Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

<a name="create-integration-account-environment"></a>

## <a name="integration-accounts-with-ise"></a>Comptes d’intégration et environnement de service d’intégration

Vous pouvez utiliser des comptes d’intégration avec des applications logiques à l’intérieur d’un environnement de service d’intégration. Cependant, ces comptes d’intégration doivent utiliser le *même ISE* que les applications logiques liées. Les applications logiques d’un ISE ne peuvent faire référence qu’aux comptes d’intégration se trouvant dans le même ISE. Lorsque vous créez un compte d’intégration, vous pouvez sélectionner votre ISE en tant qu’emplacement pour votre compte d’intégration. Pour plus d’informations sur la tarification et la facturation des comptes d’intégration avec un ISE, consultez [Modèle de tarification Logic apps](../logic-apps/logic-apps-pricing.md#fixed-pricing). Pour connaître la tarification, consultez [Tarification Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/). Pour plus d’informations sur les limites, consultez [Limites du compte d’intégration](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits).

## <a name="next-steps"></a>Étapes suivantes

* [Se connecter à des réseaux virtuels Azure à partir d’Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
* En apprendre davantage sur le [Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md)
* Découvrir l’[Intégration d’un réseau virtuel pour les services Azure](../virtual-network/virtual-network-for-azure-services.md)
