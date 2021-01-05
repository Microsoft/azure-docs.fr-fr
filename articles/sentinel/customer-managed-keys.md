---
title: Configurer des clés gérées par le client dans Azure Sentinel | Microsoft Docs
description: Découvrez comment configurer des clés gérées par le client (CMK) dans Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2020
ms.author: yelevin
ms.openlocfilehash: bd85936c86656a8ae59a04ccdf53a62bd852368f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655270"
---
# <a name="set-up-azure-sentinel-customer-managed-key"></a>Configurer une clé gérée par le client Azure Sentinel

Cet article fournit des informations générales et des étapes indiquant comment configurer une clé gérée par le client (CMK) pour Azure Sentinel. La fonctionnalité CMK permet de chiffrer toutes les données enregistrées ou envoyées à Azure Sentinel dans toutes les ressources de stockage pertinentes avec une clé Azure Key Vault que vous avez créée ou dont vous êtes propriétaire.

> [!NOTE]
> - La fonctionnalité Azure Sentinel CMK est fournie uniquement aux **nouveaux clients**.
>
> - L’accès à cette fonctionnalité est contrôlé par l’inscription à la fonctionnalité Azure.  Vous pouvez y demander l’accès en contactant azuresentinelCMK@microsoft.com. Les demandes en attente seront approuvées en fonction de la capacité disponible.
>
> - La fonctionnalité CMK est disponible uniquement pour les clients qui envoient au moins 1 To de données par jour. Vous recevrez d’autres informations sur les prix quand vous vous adresserez à Microsoft pour provisionner CMK sur votre abonnement Azure. Découvrez plus en détail les [tarifs de Log Analytics](../azure-monitor/platform/manage-cost-storage.md#log-analytics-dedicated-clusters).

## <a name="how-cmk-works"></a>Fonctionnement de CMK 

La solution Azure Sentinel utilise plusieurs ressources de stockage, notamment Log Analytics, pour la collecte de journaux et les fonctionnalités. Dans le cadre de la configuration de la fonctionnalité CMK d’Azure Sentinel, vous devez également configurer les paramètres CMK sur les ressources de stockage associées. Les données enregistrées dans des ressources de stockage autres que Log Analytics sont également chiffrées.

En savoir plus sur [CMK](../azure-monitor/platform/customer-managed-keys.md#customer-managed-key-overview).

> [!NOTE]
> Si vous activez CMK sur Azure Sentinel, toute fonctionnalité en préversion publique qui ne prend pas en charge CMK n’est pas activée.

## <a name="enable-cmk"></a>Activer CMK 

Pour provisionner CMK, effectuez les étapes suivantes : 

1.  Créez un coffre de clés Azure et une clé de stockage.

2.  Activez CMK sur votre espace de travail Log Analytics.

3.  Inscrivez-vous à Cosmos DB.

4.  Ajoutez une stratégie d’accès à votre instance Azure Key Vault.

5.  Activez CMK dans Azure Sentinel.

6.  Activez Azure Sentinel.

### <a name="step-1-create-an-azure-key-vault-and-storing-key"></a>ÉTAPE 1 : Créer un coffre de clés Azure et une clé de stockage

1.  [Créez une ressource Azure Key Vault](/azure-stack/user/azure-stack-key-vault-manage-portal), puis générez ou importez une clé à utiliser pour le chiffrement des données.
    > [!NOTE]
    >  Azure Key Vault doit être configuré comme récupérable pour protéger votre clé et l’accès.

1.  [Activez les options de récupération :](../key-vault/general/best-practices.md#turn-on-recovery-options)

    -   Vérifiez que la [suppression réversible](../key-vault/general/soft-delete-overview.md) est activée.

    -   Activez la [protection contre le vidage](../key-vault/general/soft-delete-overview.md#purge-protection) pour bénéficier d’une protection contre la suppression forcée du secret ou du coffre, même après activation de la suppression réversible.

### <a name="step-2-enable-cmk-on-your-log-analytics-workspace"></a>ÉTAPE 2 : Activer CMK sur votre espace de travail Log Analytics

Suivez les instructions dans [Configuration de clés gérées par le client dans Azure Monitor](../azure-monitor/platform/customer-managed-keys.md) afin de créer un espace de travail CMK que vous utiliserez comme espace de travail Azure Sentinel dans les étapes suivantes.

### <a name="step-3-register-for-cosmos-db"></a>ÉTAPE 3 : S’inscrire à Cosmos DB

Azure Sentinel fonctionne avec Cosmos DB en tant que ressource de stockage supplémentaire. Veillez à vous inscrire à Cosmos DB.

Suivez les instructions de Cosmos DB indiquant comment [inscrire le fournisseur de ressources Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md#register-resource-provider) dans l’abonnement Azure.

### <a name="step-4-add-an-access-policy-to-your-azure-key-vault-instance"></a>ÉTAPE 4 : Ajouter une stratégie d’accès à votre instance Azure Key Vault

Veillez à ajouter l’accès à partir de Cosmos DB à votre instance Azure Key Vault. Suivez les instructions Cosmos DB indiquant comment [ajouter une stratégie d’accès à votre instance Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#add-an-access-policy-to-your-azure-key-vault-instance) avec le principal Azure Cosmos DB.

### <a name="step-5-enable-cmk-in-azure-sentinel"></a>ÉTAPE 5 : Activez CMK dans Azure Sentinel

La fonctionnalité CMK d’Azure Sentinel est fournie aux nouveaux clients uniquement une fois qu’ils ont reçu l’accès directement à partir du groupe de produits Azure. Utilisez vos contacts chez Microsoft afin de recevoir l’approbation de l’équipe Azure Sentinel pour activer CMK dans votre solution.

Une fois l’approbation obtenue, vous êtes invité à fournir les informations suivantes pour activer la fonctionnalité CMK.

-  ID de l’espace de travail sur lequel vous souhaitez activer CMK

-  URL du coffre de clés : Copiez l’Identificateur de clé de la clé jusqu’à la dernière barre oblique :  
    

    ![Identificateur de clé](./media/customer-managed-keys/key-identifier.png)

    L’équipe Azure Sentinel active la fonctionnalité CMK d’Azure Sentinel pour votre espace de travail fourni.

-  L’équipe de produit Azure Sentinel vérifie que vous avez été autorisé à utiliser cette fonctionnalité. Vous devez satisfaire à cette condition avant de continuer.

### <a name="step-6-enable-azure-sentinel"></a>ÉTAPE 6 : Activer Azure Sentinel


Accédez au portail Azure et activez Azure Sentinel sur l’espace de travail sur lequel vous avez configuré CMK. Pour plus d’informations, consultez [Intégration d’Azure Sentinel](quickstart-onboard.md).

## <a name="key-encryption-key-revocation-or-deletion"></a>Révocation ou suppression d’une clé de chiffrement principale


Si un utilisateur révoque la clé de chiffrement principale, soit en la supprimant, soit en enlevant l’accès à Azure Sentinel, ce dernier prend en compte la modification dans l’heure qui suit, se comportant alors comme si les données n’étaient plus disponibles. À ce stade, toute opération qui utilise des ressources de stockage persistantes, telle que l’ingestion de données, la modification de configuration persistante et la création d’incident, est empêchée. Les données déjà stockées ne sont pas supprimées, mais restent inaccessibles. Les données inaccessibles sont régies par la stratégie de conservation des données et sont vidées conformément à cette stratégie.

La seule opération possible une fois que la clé de chiffrement est révoquée ou supprimée est la suppression du compte.

Si l’accès est restauré après la révocation, Azure Sentinel restaure l’accès aux données dans l’heure qui suit.

Pour en savoir plus sur la façon dont cela fonctionne dans Azure Monitor, consultez [Révocation de clé CMK dans Azure Monitor](../azure-monitor/platform/customer-managed-keys.md#key-revocation).

## <a name="key-encryption-key-rotation"></a>Rotation des clés de chiffrement principales


Azure Sentinel et Log Analytics prennent en charge la rotation des clés. Quand un utilisateur effectue une rotation de clé dans Key Vault, Azure Sentinel prend en charge la nouvelle clé dans l’heure qui suit.

Dans Key Vault, vous pouvez effectuer une rotation de clé en créant une version de la clé :

![rotation de clé](./media/customer-managed-keys/key-rotation.png)

Vous pouvez désactiver la version précédente de la clé après 24 heures, ou dès que les journaux d’audit Azure Key Vault ne font état d’aucune activité utilisant cette version.

Si vous utilisez la même clé dans Azure Sentinel et dans Log Analytics, il est nécessaire d’effectuer une rotation de clé ; vous devez mettre à jour explicitement la ressource de cluster dans Log Analytics avec la nouvelle version de la clé Azure Key Vault. Pour plus d’informations, consultez [Rotation de clé CMK dans Azure Monitor](../azure-monitor/platform/customer-managed-keys.md#key-rotation).

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à configurer une clé gérée par le client dans Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.