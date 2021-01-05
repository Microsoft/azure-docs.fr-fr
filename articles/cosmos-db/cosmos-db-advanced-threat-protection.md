---
title: Advanced Threat Protection pour Azure Cosmos DB
description: Découvrez comment Azure Cosmos DB assure le chiffrement des données au repos, et comment s’effectue son implémentation.
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/13/2019
ms.custom: seodec18
ms.author: memildin
author: memildin
manager: rkarlin
ms.openlocfilehash: ea7524b32b7637aa7c36308f8b869aa5207c08a2
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334410"
---
# <a name="advanced-threat-protection-for-azure-cosmos-db-preview"></a>Advanced Threat Protection pour Azure Cosmos DB (préversion)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Advanced Threat Protection pour Azure Cosmos DB offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes Azure Cosmos DB. Cette couche de protection vous permet de traiter efficacement les menaces sans pour autant être un expert en sécurité et de les intégrer aux systèmes centraux de supervision de la sécurité.

Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. Ces alertes de sécurité sont intégrées avec [Azure Security Center](https://azure.microsoft.com/services/security-center/) et sont également envoyées par e-mail aux administrateurs d’abonnement, avec les détails des activités suspectes, ainsi que des recommandations sur la façon d’examiner et de corriger les menaces.

> [!NOTE]
>
> * Advanced Threat Protection pour Azure Cosmos DB n’est actuellement disponible que pour l’API SQL.
> * Advanced Threat Protection pour Azure Cosmos DB n’est actuellement pas disponible dans Azure government et les régions cloud souverain.

Pour une expérience d’investigation complète des alertes de sécurité, nous vous recommandons d’activer la [journalisation des diagnostics dans Azure Cosmos DB](./monitor-cosmos-db.md), ce qui journalise les opérations sur la base de données elle-même, notamment les opérations CRUD sur l’ensemble des documents, conteneurs et bases de données.

## <a name="threat-types"></a>Types de menaces

Advanced Threat Protection pour Azure Cosmos DB détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données. Il peut actuellement déclencher les alertes suivantes :

- **Accès à partir d’emplacements inhabituels**  : Cette alerte est déclenchée en cas de modification du modèle d’accès à un compte Azure Cosmos, où un utilisateur s’est connecté au point de terminaison Azure Cosmos DB à partir d’un emplacement géographique inhabituel. Dans certains cas, l’alerte détecte une action légitime indiquant une nouvelle application ou une opération de maintenance du développeur. Dans d’autres cas, l’alerte détecte une action malveillante, par exemple, d’un ancien employé, d’un attaquant externe, etc.

- **Extraction de données inhabituelles**  : Cette alerte est déclenchée quand un client extrait une quantité inhabituelle de données d’un compte Azure Cosmos DB. Il peut s’agir d’un symptôme d’exfiltration de données effectuée pour transférer toutes les données stockées dans le compte vers un magasin de données externe.



## <a name="configure-advanced-threat-protection"></a>Configurer Advanced Threat Protection

Vous pouvez configurer la protection avancée contre les menaces de plusieurs façons, qui sont décrites dans les sections suivantes.

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Lancez le portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com/).

2. À partir du compte Azure Cosmos DB, dans le menu **Paramètres** , sélectionnez **Sécurité avancée**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp.png" alt-text="Configurer ATP":::

3. Dans le panneau de configuration **Sécurité avancée**  :

    * Cliquez sur l’option **Advanced Threat Protection** pour la définir sur **Activé**.
    * Cliquez sur **Enregistrer** pour enregistrer la stratégie de protection avancée contre les menaces, nouvelle ou mise à jour.   

### <a name="rest-api"></a>[REST API](#tab/rest-api)

Utilisez les commandes de l’API REST pour créer, mettre à jour ou obtenir le paramètre Advanced Threat Protection pour un compte Azure Cosmos DB spécifique.

* [Advanced Threat Protection - Créer](/rest/api/securitycenter/advancedthreatprotection/create)
* [Advanced Threat Protection - Obtenir](/rest/api/securitycenter/advancedthreatprotection/get)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez les cmdlets PowerShell suivantes :

* [Activer Advanced Threat Protection](/powershell/module/az.security/enable-azsecurityadvancedthreatprotection?viewFallbackFrom=azps-2.4.0)
* [Obtenir Advanced Threat Protection](/powershell/module/az.security/get-azsecurityadvancedthreatprotection?viewFallbackFrom=azps-2.4.0)
* [Désactiver Advanced Threat Protection](/powershell/module/az.security/disable-azsecurityadvancedthreatprotection?viewFallbackFrom=azps-2.4.0)

### <a name="arm-template"></a>[Modèle ARM](#tab/arm-template)

Utilisez un modèle Azure Resource Manager (ARM) pour configurer Cosmos DB avec Advanced Threat Protection activé.
Pour plus d’informations, consultez [Créer un compte Cosmos DB avec Advanced Threat Protection](https://azure.microsoft.com/resources/templates/201-cosmosdb-advanced-threat-protection-create-account/).

### <a name="azure-policy"></a>[Azure Policy](#tab/azure-policy)

Utilisez Azure Policy pour activer Advanced Threat Protection pour Cosmos DB.

1. Lancez la page Azure **Policy - Définitions** et recherchez la stratégie **Déployer Advanced Threat Protection pour Cosmos DB**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db.png" alt-text="Rechercher dans la stratégie"::: 

1. Cliquez sur la stratégie **Déployer Advanced Threat Protection pour Cosmos DB** , puis sur **Attribuer**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-policy.png" alt-text="Sélectionnez Abonnement ou Groupe":::


1. Dans le champ **Étendue** , cliquez sur les trois points, sélectionnez un abonnement Azure ou un groupe de ressources, puis cliquez sur **Sélectionner**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-details.png" alt-text="Page Définitions de stratégies":::


1. Entrez les autres paramètres, puis cliquez sur **Attribuer**.




## <a name="manage-atp-security-alerts"></a>Gérer les alertes de sécurité ATP

Quand des anomalies d’activité Azure Cosmos DB sont détectées, une alerte est déclenchée. Elle contient des informations sur l’événement de sécurité suspect. 

 Dans Azure Security Center, vous pouvez examiner et gérer vos [alertes de sécurité](../security-center/security-center-alerts-overview.md) actuelles.  Cliquez sur l’alerte spécifiée dans [Security Center](https://ms.portal.azure.com/#blade/Microsoft_Azure_Security/SecurityMenuBlade/0) pour voir les causes possibles et les actions recommandées afin d’examiner et d’atténuer la menace potentielle. L’image suivante représente un exemple d’informations d’alerte fournies dans Security Center.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert-details.png" alt-text="Détails de la menace":::

Une notification par e-mail est également envoyée avec les détails de l’alerte et les actions recommandées. L’image suivante représente un exemple d’alerte par e-mail.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert.png" alt-text="Détails de l'alerte":::

## <a name="cosmos-db-atp-alerts"></a>Alertes ATP Cosmos DB

 Pour afficher la liste des alertes générées lors de la supervision des comptes Azure Cosmos DB, consultez la section [Alertes Cosmos DB](../security-center/alerts-reference.md#alerts-azurecosmos) dans la documentation Azure Security Center.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [Journalisation des diagnostics dans Azure Cosmos DB](cosmosdb-monitor-resource-logs.md)
* En savoir plus sur [Azure Security Center](../security-center/security-center-introduction.md)