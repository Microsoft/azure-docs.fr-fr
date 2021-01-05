---
title: Stratégie de protection des informations SQL dans Azure Security Center
description: Découvrez comment personnaliser les stratégies de protection des informations dans Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 2ebf2bc7-232a-45c4-a06a-b3d32aaf2500
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/04/2020
ms.author: memildin
ms.openlocfilehash: 0a487f778693e87e680033edd0d80c55d1a85f66
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013783"
---
# <a name="sql-information-protection-policy-in-azure-security-center"></a>Stratégie de protection des informations SQL dans Azure Security Center
 
Le [mécanisme de découverte et de classification des données](../azure-sql/database/data-discovery-and-classification-overview.md) de la protection des informations SQL offre des fonctionnalités avancées pour la découverte, la classification, l’étiquetage et la génération de rapports sur les données sensibles dans vos bases de données. Il est intégré à [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md), [Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md) et à [Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md).

Le mécanisme de classification est basé sur les deux éléments suivants :

- **Étiquettes** : principaux attributs de classification utilisés pour définir le *niveau de sensibilité des données* stockées dans la colonne. 
- **Types d’informations** : spécifie une granularité supplémentaire au niveau du *type des données* stockées dans la colonne.

Les options de la stratégie de protection des informations dans Security Center fournissent un ensemble prédéfini d’étiquettes et de types d’informations qui servent de valeurs par défaut pour le moteur de classification. Vous pouvez personnaliser la stratégie en fonction des besoins de votre organisation, comme décrit ci-dessous.

> [!IMPORTANT]
> Pour personnaliser la stratégie de protection des informations de votre locataire Azure, vous devez disposer de privilèges administratifs sur le groupe d’administration racine du locataire. Apprenez-en davantage dans [Gagner en visibilité au niveau locataire dans Azure Security Center](security-center-management-groups.md).

:::image type="content" source="./media/security-center-info-protection-policy/sql-information-protection-policy-page.png" alt-text="Page affichant votre stratégie de protection des informations SQL":::
 



## <a name="how-do-i-access-the-sql-information-protection-policy"></a>Comment accéder à la stratégie de protection des informations SQL ?

Il existe trois façons d’accéder à la stratégie de protection des informations :

- **(Recommandé)** À partir de la page des tarifs et des paramètres de Security Center
- À partir de la recommandation de sécurité « Les données sensibles de vos bases de données SQL doivent être classifiées »
- À partir de la page de découverte des données Azure SQL DB

Chaque façon est affichée dans l’onglet correspondant, ci-dessous.



### <a name="from-security-centers-settings"></a>[**À partir des paramètres de Security Center**](#tab/sqlip-tenant)

### <a name="access-the-policy-from-security-centers-pricing-and-settings-page"></a>Accéder à la stratégie depuis la page des tarifs et des paramètres de Security Center<a name="sqlip-tenant"></a>

Dans la page des **tarifs et des paramètres** de Security Center, sélectionnez **Protection des informations SQL**.

> [!NOTE]
> Cette option s’affiche uniquement pour les utilisateurs disposant d’autorisations au niveau du locataire. 

:::image type="content" source="./media/security-center-info-protection-policy/pricing-settings-link-to-information-protection.png" alt-text="Accès à la stratégie de protection des informations SQL depuis la page des tarifs et des paramètres d’Azure Security Center":::



### <a name="from-security-centers-recommendation"></a>[**À partir de la recommandation de Security Center**](#tab/sqlip-db)

### <a name="access-the-policy-from-the-security-center-recommendation"></a>Accéder à la stratégie depuis la recommandation de Security Center<a name="sqlip-db"></a>

Utilisez la recommandation de Security Center, « Les données sensibles de vos bases de données SQL doivent être classifiées », pour afficher la page de découverte et de classification des données de votre base de données. Là, vous verrez également les colonnes découvertes qui contiennent des informations que nous vous recommandons de classifier.

1. Dans la page **Recommendations de Security Center**, recherchez la recommandation **Les données sensibles de vos bases de données SQL doivent être classifiées**.

    :::image type="content" source="./media/security-center-info-protection-policy/sql-sensitive-data-recommendation.png" alt-text="Recherche de la recommandation qui fournit l’accès aux stratégies de protection des informations SQL":::

1. Dans la page des détails de la recommandation, sélectionnez une base de données dans les onglets les répertoriant comme **saines** ou **non saines**.

1. La page **Découverte et classification** des données s’ouvre. Sélectionnez **Configurer**.

    :::image type="content" source="./media/security-center-info-protection-policy/access-policy-from-security-center-recommendation.png" alt-text="Ouverture de la stratégie de protection des informations SQL depuis la recommandation appropriée dans Azure Security Center":::



### <a name="from-azure-sql"></a>[**À partir de SQL Azure**](#tab/sqlip-azuresql)

### <a name="access-the-policy-from-azure-sql"></a>Accéder à la stratégie depuis SQL Azure<a name="sqlip-azuresql"></a>

1. Dans le portail Azure, ouvrez SQL Azure.

    :::image type="content" source="./media/security-center-info-protection-policy/open-azure-sql.png" alt-text="Ouverture de SQL Azure à partir du portail Azure":::

1. Sélectionnez une base de données.

1. Dans la zone **Sécurité** du menu, ouvrez la page **Découverte et classification des données** (1) et sélectionnez **Configurer** (2).

    :::image type="content" source="./media/security-center-info-protection-policy/access-policy-from-azure-sql.png" alt-text="Ouverture de la stratégie de protection des informations SQL depuis SQL Azure":::

--- 


## <a name="customize-your-information-types"></a>Personnaliser vos types d’informations

Pour gérer et personnaliser les types d’informations :

1. Sélectionnez **Gérer les types d’informations**.

    :::image type="content" source="./media/security-center-info-protection-policy/manage-types.png" alt-text="Gérer les types d’informations pour votre stratégie de protection des informations":::

1. Pour ajouter un nouveau type, sélectionnez **Créer un type d’informations**. Vous pouvez définir un nom, une description et des chaînes de motif de recherche pour le type d’informations. Les chaînes de motif de recherche peuvent éventuellement utiliser des mots clés avec des caractères génériques (en utilisant le caractère « % »), que le moteur de découverte automatique utilise pour identifier les données sensibles dans vos bases de données, à partir des métadonnées de la colonne.
 
    :::image type="content" source="./media/security-center-info-protection-policy/configure-new-type.png" alt-text="Configurer un nouveau type d’informations pour votre stratégie de protection des informations":::

1. Vous pouvez également modifier les types intégrés en ajoutant des chaînes de motif de recherche supplémentaires, en désactivant certaines chaînes existantes ou en modifiant la description. 

    > [!TIP]
    > Vous ne pouvez pas supprimer les types intégrés ni modifier leurs noms. 

1. Les **types d’informations** sont classés par un ordre croissant du classement de découverte, ce qui signifie que les types en haut de la liste tentent de correspondre en premier lieu. Pour modifier le classement entre des types d’informations, faites-les glisser à l’emplacement correct dans la table, ou bien utilisez les boutons **Monter** et **Descendre** pour modifier l’ordre. 

1. Sélectionnez **OK** lorsque vous avez terminé.

1. Une fois que vous avez terminé la gestion de vos types d’informations, veillez à associer les types pertinents avec les étiquettes appropriées, en cliquant sur **Configurer** pour une étiquette particulière et en ajoutant ou en supprimant les types d’informations qu’il faut.

1. Pour appliquer vos modifications, sélectionnez **Enregistrer** dans la page **Étiquettes** principale.
 

## <a name="exporting-and-importing-a-policy"></a>Exportation et importation d’une stratégie

Vous pouvez télécharger un fichier JSON avec vos étiquettes et types d’informations définis, modifier le fichier dans l’éditeur de votre choix, puis importer le fichier mis à jour. 

:::image type="content" source="./media/security-center-info-protection-policy/export-import.png" alt-text="Exportation et importation de votre stratégie de protection des informations":::

> [!NOTE]
> Vous devez disposer d’autorisations au niveau du locataire pour importer un fichier de stratégie. 


## <a name="manage-sql-information-protection-using-azure-powershell"></a>Gérer la protection des informations SQL à l’aide d’Azure PowerShell

- [Get-AzSqlInformationProtectionPolicy](/powershell/module/az.security/get-azsqlinformationprotectionpolicy) : récupère la stratégie de protection des informations SQL du locataire effective.
- [Set-AzSqlInformationProtectionPolicy](/powershell/module/az.security/set-azsqlinformationprotectionpolicy) : définit la stratégie de protection des informations SQL du locataire effective.
 

## <a name="next-steps"></a>Étapes suivantes
 
Dans cet article, vous avez appris à définir une stratégie de protection des informations dans Azure Security Center. Pour en savoir plus sur l’utilisation de la protection des informations SQL pour classifier et protéger des données sensibles dans vos bases de données SQL, consultez [Découverte et classification des données de base de données Azure SQL](../azure-sql/database/data-discovery-and-classification-overview.md).

Pour plus d’informations sur les stratégies de sécurité et la sécurité des données dans Security Center, consultez les articles suivants :
 
- [Définition de stratégies de sécurité dans Azure Security Center](tutorial-security-policy.md) : découvrez comment configurer des stratégies de sécurité pour vos groupes de ressources et abonnements Azure
- [Sécurité des données Azure Security Center](security-center-data-security.md) : découvrez comment Security Center gère et protège les données
