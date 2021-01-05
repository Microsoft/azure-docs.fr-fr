---
title: Prérequis d’Avere vFXT - Azure
description: Découvrez les tâches à effectuer avant de créer un cluster dans Avere vFXT pour Azure, notamment la gestion des abonnements, des quotas et des points de terminaison de service de stockage.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/21/2020
ms.author: rohogue
ms.openlocfilehash: d87f57873a4bb84b20df3da3880017d9ef2484a5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008410"
---
# <a name="prepare-to-create-the-avere-vfxt"></a>Se préparer à la création du système Avere vFXT

Cet article décrit les tâches prérequises pour la création d’un cluster Avere vFXT.

## <a name="create-a-new-subscription"></a>Créer un abonnement

Commencez par créer un abonnement Azure. Utilisez un abonnement distinct pour chaque projet Avere vFXT de façon à simplifier le suivi des dépenses et le nettoyage, et pour garantir que d’autres projets ne sont pas affectés par les quotas ou la limitation des ressources lors du provisionnement du workflow du cluster.

Pour créer un abonnement Azure dans le portail Azure :

1. Accédez au [panneau Abonnements](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)
1. Cliquez sur le bouton **+ Ajouter** situé en haut
1. Si vous y êtes invité, connectez-vous
1. Sélectionnez une offre, puis suivez les étapes permettant de créer un abonnement

## <a name="configure-subscription-owner-permissions"></a>Configurer des autorisations de propriétaire d’abonnement

Un utilisateur disposant d’autorisations de propriétaire pour l’abonnement doit créer le cluster vFXT. La création du cluster nécessite qu’un propriétaire accepte les conditions d’utilisation du logiciel et autorise les modifications des ressources réseau et de stockage.

Certaines solutions de contournement permettent à un non-propriétaire de créer un cluster Avere vFXT pour Azure. Ces scénarios impliquent la restriction de ressources et l’affectation de rôles Azure supplémentaires au créateur. Dans tous ces cas, un propriétaire d’abonnement doit aussi préalablement [accepter les conditions d’utilisation du logiciel Avere vFXT](#accept-software-terms).

| Scénario | Restrictions | Rôles d’accès requis pour créer le cluster Avere vFXT |
|----------|--------|-------|
| L’administrateur du groupe de ressources crée le vFXT | Le réseau virtuel, le contrôleur de cluster et les nœuds du cluster doivent être créés dans le groupe de ressources. | Les rôles [Administrateur de l’accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) et [Contributeur](../role-based-access-control/built-in-roles.md#contributor), tous deux limités au groupe de ressources cible. |
| Utiliser un réseau virtuel externe existant | Le contrôleur de cluster et les nœuds du cluster sont créés dans le groupe de ressources du vFXT, mais utilisent un réseau virtuel existant dans un autre groupe de ressources. | (1) Rôles [administrateur de l’accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) et [contributeur](../role-based-access-control/built-in-roles.md#contributor) étendus au groupe de ressources vFXT ; et (2) rôles [contributeur de machines virtuelles](../role-based-access-control/built-in-roles.md#virtual-machine-contributor), [administrateur de l’accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) et [contributeur Avere](../role-based-access-control/built-in-roles.md#avere-contributor) étendus au groupe de ressources du réseau virtuel. |
| Rôle personnalisé pour les créateurs de cluster | Aucune restriction de placement des ressources. Cette méthode confère des privilèges significatifs aux non-propriétaires. | Le propriétaire de l’abonnement crée un rôle personnalisé Azure comme expliqué dans [cet article](avere-vfxt-non-owner.md). |

## <a name="quota-for-the-vfxt-cluster"></a>Quota pour le cluster vFXT

Veillez à disposer d’un quota suffisant pour les composants Azure suivants. Si nécessaire, [demandez une augmentation du quota](../azure-portal/supportability/resource-manager-core-quotas-request.md).

> [!NOTE]
> Les machines virtuelles et les composants SSD répertoriés ici sont destinées au cluster vFXT proprement dit. N’oubliez pas que vous avez également besoin d’un quota pour les machines virtuelles et les disques SSD que vous allez utiliser pour votre batterie de serveurs de calcul.
>
> Vérifiez que le quota est activé pour la région dans laquelle vous envisagez d’exécuter le flux de travail.

|Composant Azure|Quota|
|----------|-----------|
|Machines virtuelles|3 E32s_v3 ou plus (une par nœud de cluster) |
|Stockage SSD Premium|200 Go d’espace de système d’exploitation, et de 1 To à 4 To d’espace de cache par nœud |
|Compte de stockage (facultatif) |v2|
|Stockage back-end de données (facultatif) |Un nouveau conteneur d’objets blob LRS |
<!-- this table also appears in the overview - update it there if updating here -->

## <a name="accept-software-terms"></a>Accepter les conditions d'utilisation du logiciel

> [!TIP]
> Ignorez cette étape si un propriétaire d’abonnement crée le cluster Avere vFXT.

Pendant la création du cluster, vous devez accepter les conditions d'utilisation du logiciel Avere vFXT. Si vous n’êtes pas propriétaire d’abonnement, demandez à un propriétaire d’abonnement d’accepter préalablement les conditions.

Cette étape ne doit être effectuée qu’une fois par abonnement.

Pour accepter les conditions du logiciel à l’avance :

1. Ouvrez un cloud shell dans le portail Azure ou en accédant à <https://shell.azure.com>. Connectez-vous avec votre ID d’abonnement.

   ```azurecli
    az login
    az account set --subscription <subscription ID>
   ```

1. Émettez la commande suivante pour accepter les conditions d’utilisation et activer l’accès programmatique du système Avere vFXT pour l’image de logiciels Azure :

   ```azurecli
   az vm image accept-terms --urn microsoft-avere:vfxt:avere-vfxt-controller:latest
   ```

## <a name="create-a-storage-service-endpoint-in-your-virtual-network-if-needed"></a>Créer un point de terminaison de service de stockage dans votre réseau virtuel (si nécessaire)

Ce [point de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) conserve le trafic d’objets blob Azure au niveau local au lieu de l’acheminer vers l’extérieur du réseau virtuel. Il est recommandé que tout cluster Avere vFXT pour Azure utilise le Stockage Blob Azure pour le stockage de données principal.

Si vous créez un réseau virtuel lors de la création du cluster, un point de terminaison est créé automatiquement. Si vous fournissez un réseau virtuel existant, celui-ci doit disposer d’un point de terminaison de service de stockage Microsoft si vous voulez créer un conteneur de stockage d’objets blob lors de la création du cluster.<!-- if there is no endpoint in that situation, the cluster creation will fail -->

> [!TIP]
>
>* Ignorez cette étape si vous créez un réseau virtuel dans le cadre de la création du cluster.
>* Un point de terminaison est facultatif si vous ne créez pas de stockage d’objets blob lors de la création du cluster. Dans ce cas, vous pouvez créer le point de terminaison de service ultérieurement si vous décidez d’utiliser un Stockage Blob Azure.

Créer le point de terminaison de service de stockage à partir du portail Azure.

1. Dans le portail, ouvrez la liste des réseaux virtuels.
1. Sélectionnez le réseau virtuel de votre cluster.
1. Cliquez sur **Points de terminaison de service** dans le menu à gauche.
1. Cliquez sur **Ajouter** dans la partie supérieure.
1. Sélectionnez le service ``Microsoft.Storage``.
1. Sélectionnez le sous-réseau du cluster.
1. Dans la partie inférieure, cliquez sur **Ajouter**.

   ![Capture d’écran du portail Azure montrant les étapes de création du point de terminaison de service](media/avere-vfxt-service-endpoint.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois ces prérequis effectués, vous pouvez créer le cluster. Pour obtenir des instructions, lisez [Déployer le cluster vFXT](avere-vfxt-deploy.md).