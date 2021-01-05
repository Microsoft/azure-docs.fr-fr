---
title: Configurer Azure Defender pour le stockage
titleSuffix: Azure Storage
description: Configurez Azure Defender pour le stockage de façon à détecter les anomalies dans l’activité de votre compte et à être notifié en cas de tentatives d’accès potentiellement dangereuses à votre compte.
services: storage
author: tamram
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: tamram
ms.reviewer: ozgun
ms.openlocfilehash: 0bda32aaab301fe9ed685f0bfd6d4596fab4e5db
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92789010"
---
# <a name="configure-azure-defender-for-storage"></a>Configurer Azure Defender pour le stockage

Azure Defender pour le stockage offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes de stockage. Cette couche de protection vous permet de traiter les menaces sans pour autant être un expert en sécurité ni avoir besoin de gérer des systèmes de supervision de la sécurité.

Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. Ces alertes de sécurité sont intégrées avec [Azure Security Center](https://azure.microsoft.com/services/security-center/) et sont également envoyées par e-mail aux administrateurs d’abonnement, avec les détails des activités suspectes et des recommandations sur la façon d’examiner et de corriger les menaces.

Le service ingère les journaux de ressources des demandes de lecture, d’écriture et de suppression adressées au Stockage Blob et à Azure Files en vue de détecter les menaces. Pour examiner les alertes d’Azure Defender, vous pouvez afficher l’activité de stockage associée à l’aide de la journalisation Storage Analytics. Pour plus d’informations, consultez **Configuration de la journalisation** dans [Surveillance d’un compte de stockage dans le portail Azure](storage-monitor-storage-account.md#configure-logging).

## <a name="availability"></a>Disponibilité

Azure Defender pour le stockage est actuellement disponible pour Stockage Blob, Azure Files et Azure Data Lake Storage Gen2. Les types de compte prenant en charge Azure Defender incluent les comptes à usage général v2, les objets blob de blocs et les comptes Stockage Blob. Azure Defender pour le stockage est disponible dans tous les clouds publics et US Government, mais pas dans d’autres régions de cloud souverain ou de cloud Azure Government.

Les comptes avec des espaces de noms hiérarchiques activés pour Data Lake Storage prennent en charge les transactions à l’aide des API Stockage Blob Azure et des API Data Lake Storage. Les partages de fichiers Azure prennent en charge les transactions via SMB.

Pour plus d’informations sur la tarification, y compris une version d’évaluation gratuite de 30 jours, consultez la [page de tarification d’Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/).

La liste suivante récapitule la disponibilité d’Azure Defender pour le stockage :

- État de sortie :
  - [Stockage Blob](https://azure.microsoft.com/services/storage/blobs/) (disponibilité générale)
  - [Azure Files](../files/storage-files-introduction.md) (disponibilité générale)
  - Azure Data Lake Storage Gen2 (disponibilité générale)
- Clouds :<br>
    ✔ Clouds commerciaux<br>
    ✔ US Gov<br>
    ✘ Chine Gov, autres Gov

## <a name="set-up-azure-defender"></a>Configurer Azure Defender

Vous pouvez configurer Azure Defender pour le stockage de plusieurs façons, qui sont décrites dans les sections suivantes.

### <a name="azure-security-center"></a>[Centre de sécurité Azure](#tab/azure-security-center)

Lorsque vous vous abonnez au niveau Standard dans Azure Security Center, Azure Defender est automatiquement configuré sur tous vos comptes de stockage. Vous pouvez activer ou désactiver Azure Defender pour vos comptes de stockage sous un abonnement spécifique comme suit :

1. Lancez **Azure Security Center** dans le [portail Azure](https://portal.azure.com).
1. Dans le menu principal, sous **Gestion** , sélectionnez **Tarification et paramètres** .
1. Sélectionnez l’abonnement pour lequel vous souhaitez activer ou désactiver Azure Defender.
1. Sélectionnez **Azure Defender sur** pour activer Azure Defender pour l’abonnement.
1. Sous **Sélectionner un plan Azure Defender par type de ressource** , recherchez la ligne **Stockage** , puis sélectionnez **Activé** dans la colonne **Plan** .
1. Enregistrez vos modifications.

    :::image type="content" source="media/azure-defender-storage-configure/enable-azure-defender-security-center.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

Azure Defender est maintenant activé pour tous les comptes de stockage de cet abonnement.

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Lancez le [portail Azure](https://portal.azure.com/).
1. Accédez à votre compte de stockage. Sous **Paramètres** , sélectionnez **Sécurité avancée** .
1. Sélectionnez **Activer Azure Defender pour le stockage** .

    :::image type="content" source="media/azure-defender-storage-configure/enable-azure-defender-portal.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

Azure Defender est maintenant activé pour ce compte de stockage.

### <a name="template"></a>[Modèle](#tab/template)

Utilisez un modèle Azure Resource Manager pour déployer un compte Stockage Azure avec Azure Defender activé. Pour plus d’informations, consultez [Compte de stockage avec la protection avancée contre les menaces](https://azure.microsoft.com/resources/templates/201-storage-advanced-threat-protection-create/).

### <a name="azure-policy"></a>[Azure Policy](#tab/azure-policy)

Utilisez Azure Policy pour activer Azure Defender dans les comptes de stockage sous un abonnement ou un groupe de ressources spécifiques.

1. Lancez la page **Azure Policy - Définitions** .
1. Recherchez la stratégie **Déployer Azure Defender sur les comptes de stockage** .

    :::image type="content" source="media/azure-defender-storage-configure/storage-atp-policy-definitions.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

1. Sélectionnez un groupe de ressources ou un abonnement Azure.

    :::image type="content" source="media/azure-defender-storage-configure/storage-atp-policy2.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

1. Attribuez la stratégie.

    :::image type="content" source="media/azure-defender-storage-configure/storage-atp-policy1.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

### <a name="rest-api"></a>[REST API](#tab/rest-api)

Utilisez les commandes de l’API Rest pour créer, mettre à jour ou obtenir le paramètre Azure Defender pour un compte de stockage spécifique.

- [Protection avancée contre les menaces – Créer](/rest/api/securitycenter/advancedthreatprotection/create)
- [Protection avancée contre les menaces – Obtenir](/rest/api/securitycenter/advancedthreatprotection/get)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez les cmdlets PowerShell suivantes :

- [Activer la protection avancée contre les menaces](/powershell/module/az.security/enable-azsecurityadvancedthreatprotection)
- [Obtenir la protection avancée contre les menaces](/powershell/module/az.security/get-azsecurityadvancedthreatprotection)
- [Désactiver la protection avancée contre les menaces](/powershell/module/az.security/disable-azsecurityadvancedthreatprotection)

---

## <a name="explore-security-anomalies"></a>Explorer les anomalies de sécurité

Quand des anomalies d’activité du stockage sont détectées, vous recevez une notification par e-mail qui contient des informations sur l’événement de sécurité suspect. Les détails de l’événement fournis sont les suivants :

- Nature de l’anomalie
- Nom du compte de stockage
- Heure de l’événement
- Type de stockage
- Causes potentielles
- Procédure d’investigation
- Procédure de correction

L’e-mail fournit également des détails sur les causes possibles et les actions recommandées pour examiner et atténuer la menace potentielle.

:::image type="content" source="media/azure-defender-storage-configure/storage-advanced-threat-protection-alert-email.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

Vous pouvez examiner et gérer vos alertes de sécurité actuelles à partir de la [vignette Alertes de sécurité](../../security-center/security-center-managing-and-responding-alerts.md) dans Azure Security Center. Cliquez sur une alerte pour afficher les détails et les actions recommandées pour examiner la menace actuelle et atténuer les menaces futures.

:::image type="content" source="media/azure-defender-storage-configure/storage-advanced-threat-protection-alert.png" alt-text="Capture d’écran montrant comment activer Azure Defender pour le stockage dans Security Center":::

## <a name="security-alerts"></a>Alertes de sécurité

Les alertes sont générées en cas de détection de tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes de stockage. Pour obtenir la liste des alertes pour Stockage Azure, consultez [Alertes pour Stockage Azure](../../security-center/alerts-reference.md#alerts-azurestorage).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [Journaux dans les comptes de stockage Azure ](/rest/api/storageservices/About-Storage-Analytics-Logging)
- En savoir plus sur [Azure Security Center](../../security-center/security-center-introduction.md)