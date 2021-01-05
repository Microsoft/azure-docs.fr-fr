---
title: Déplacer des machines virtuelles Azure vers des zones de disponibilité dans une autre région avec Azure Resource Mover
description: Apprenez à déplacer des machines virtuelles Azure dans des zones de disponibilité avec Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 09/10/2020
ms.author: raynew
ms.openlocfilehash: 88006fb354af2673496c6476090d7f73c8a005e6
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542998"
---
# <a name="move-azure-vms-to-an-availability-zone-in-another-region"></a>Déplacer des machines virtuelles Azure vers une zone de disponibilité dans une autre région

Dans cet article, apprenez à déplacer dans une zone de disponibilité située dans une autre région Azure des machines virtuelles Azure (ainsi que les ressources réseau et de stockage associées), en utilisant [Azure Resource Mover](overview.md).

Les [zones de disponibilité Azure](../availability-zones/az-overview.md#availability-zones) contribuent à protéger votre déploiement Azure contre les défaillances de centre de données. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un refroidissement et d’un réseau indépendants. Pour garantir la résilience, il existe un minimum de trois zones distinctes dans toutes les [régions activées](../availability-zones/az-region.md). À l’aide de Resource Mover, vous pouvez déplacer :

- Une seule instance d’une machine virtuelle vers une zone de disponibilité/un groupe à haute disponibilité dans la région cible.
- Une machine virtuelle dans un groupe à haute disponibilité avec une zone de disponibilité/un groupe à haute disponibilité dans la région cible.
- Une machine virtuelle dans une zone de disponibilité de la région source vers une zone de disponibilité dans la région cible.


> [!IMPORTANT]
> Azure Resource Mover est actuellement en préversion publique.

Si vous souhaitez déplacer des machines virtuelles vers une autre zone de disponibilité de la même région, [voir cet article](../site-recovery/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md).

## <a name="prerequisites"></a>Prérequis

- L’accès *Propriétaire* à l’abonnement dans lequel se trouvent les ressources que vous souhaitez déplacer.
    - La première fois que vous ajoutez une ressource pour un mappage source et destination spécifique dans un abonnement Azure, Resource Mover crée une [identité managée affectée par le système](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (anciennement Managed Service Identify, MSI) qui est approuvée par l’abonnement.
    - Afin de créer l’identité et lui affecter le rôle demandé (Contributeur ou Administrateur de l’accès utilisateur dans l’abonnement source), le compte que vous utilisez pour ajouter des ressources a besoin des autorisations *Propriétaire* sur l’abonnement. [Explorez en détail](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) les rôles Azure.
- L’abonnement a besoin d’un quota suffisant pour créer les ressources dans la région cible. Si ce n’est pas le cas, demandez des limites supplémentaires. [Plus d’informations](../azure-resource-manager/management/azure-subscription-service-limits.md)
- Vérifiez le tarif et les frais associés à la région cible vers laquelle vous déplacez des machines virtuelles. Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour vous aider.
    


## <a name="check-vm-requirements"></a>Vérifier la configuration requise pour les machines virtuelles

1. Assurez-vous que les machines virtuelles que vous souhaitez déplacer sont prises en charge.

    - [Vérifiez](support-matrix-move-region-azure-vm.md#windows-vm-support) les machines virtuelles Windows prises en charge.
    - [Vérifiez](support-matrix-move-region-azure-vm.md#linux-vm-support) les machines virtuelles Linux et les versions du noyau prises en charge.
    - Contrôlez les paramètres de [calcul](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), de [stockage](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) et de [réseau](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings) pris en charge.
2. Vérifiez que les machines virtuelles que vous souhaitez déplacer sont activées.
3. Assurez-vous que les machines virtuelles disposent des certificats racine approuvés les plus récents et d’une liste de révocation de certificats (CRL) mise à jour. 
    - Sur les machines virtuelles Azure exécutant Windows, installez les dernières mises à jour de Windows.
    - Sur les machines virtuelles exécutant Linux, suivez les instructions du distributeur Linux pour vérifier que la machine dispose des derniers certificats et listes de révocation de certificats. 
4. Autorisez une connexion sortante à partir des machines virtuelles :
    - Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connexion sortante, autorisez l’accès à ces [URL](support-matrix-move-region-azure-vm.md#url-access) :
    - Si vous utilisez des règles de groupe de sécurité réseau (NSG) pour contrôler la connexion sortante, créez ces [règles d’étiquette de service](support-matrix-move-region-azure-vm.md#nsg-rules).

## <a name="select-resources-to-move"></a>Sélectionner les ressources à déplacer

Sélectionnez les ressources que vous souhaitez déplacer.

- Vous pouvez sélectionner n’importe quel type de ressource pris en charge dans les groupes de ressources de la région source sélectionnée.
- Vous déplacez des ressources dans une région cible dans le même abonnement que la région source. Si vous voulez changer d’abonnement, vous pouvez le faire après le déplacement des ressources.

1. Dans le portail Azure, recherchez resource mover. Ensuite, sous **Services**, sélectionnez **Azure Resource Mover**.

    ![Recherchez le bouton Resource Mover](./media/move-region-availability-zone/search.png)

2. Dans **Vue d’ensemble**, sélectionnez **Bien démarrer**.

    ![pour démarrer](./media/move-region-availability-zone/get-started.png)

3. Dans **Déplacer des ressources** > **Source + destination**, sélectionnez l’abonnement et la région sources.
4. Dans **Destination**, sélectionnez la région vers laquelle vous souhaitez déplacer les machines virtuelles. Cliquez ensuite sur **Suivant**.

     ![Page à remplir dans l’abonnement/la région source et de destination](./media/move-region-availability-zone/source-target.png)

6. Dans **Ressources à déplacer**, cliquez sur **Sélectionner des ressources**.
7. Dans **Sélectionner des ressources**, sélectionnez la machine virtuelle. Vous pouvez uniquement ajouter des ressources prises en charge pour le déplacement. Cliquez ensuite sur **Terminé**. Dans **Ressources à déplacer**, cliquez sur **Suivant**.

    ![Page de sélection des machines virtuelles à déplacer](./media/move-region-availability-zone/select-vm.png)
8. Dans **Vérifier + ajouter**, contrôlez les paramètres de source et de destination.

    ![Page de vérification des paramètres et de la poursuite du déplacement](./media/move-region-availability-zone/review.png)

9. Cliquez sur **Continuer** pour commencer à ajouter les ressources.
10. Une fois le processus d’ajout terminé, cliquez sur **Ajout de ressources pour le déplacement** dans l’icône de notification.

    ![Message dans les notifications](./media/move-region-availability-zone/notification.png)

Après avoir cliqué sur la notification, les ressources s’affichent sur la page **Entre régions**

> [!NOTE]
> Après avoir cliqué sur la notification, les ressources s’affichent sur la page **Entre régions**, dans l’état *Préparation en attente*.
> - Si vous souhaitez supprimer une ressource d’une collection de déplacement, la méthode permettant d’effectuer cette opération dépend de l’étape où vous vous trouvez dans la procédure de déplacement. [Plus d’informations](remove-move-resources.md)

## <a name="resolve-dependencies"></a>Résoudre les erreurs de dépendance

1. Si des ressources affichent un message *Valider les dépendances* dans la colonne **Problèmes**, cliquez sur le bouton **Valider les dépendances**. Le processus de validation démarre.
2. Si des dépendances sont trouvées, cliquez sur **Ajouter des dépendances**. 
3. Dans **Ajouter des dépendances**, sélectionnez les ressources dépendantes > **Ajouter des dépendances**. Supervisez la progression dans les notifications.

    ![Bouton pour ajouter des dépendances](./media/move-region-availability-zone/add-dependencies.png)

3. Ajoutez des dépendances supplémentaires si nécessaire, puis validez de nouveau les dépendances. 

    ![Page d’ajout des dépendances supplémentaires](./media/move-region-availability-zone/add-additional-dependencies.png)

4. Dans la page **Entre régions**, vérifiez que les ressources présentent maintenant l’état *Préparation en attente*, sans aucun problème répertorié.

    ![Page montrant les ressources avec l’état de préparation en attente](./media/move-region-availability-zone/prepare-pending.png)

## <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source 

Avant de pouvoir préparer et déplacer une machine virtuelle, le groupe de ressources source doit être présent dans la région cible. 

### <a name="prepare-to-move-the-source-resource-group"></a>Préparer le déplacement du groupe de ressources source

Préparez de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources source > **Préparer**.
2. Dans **Préparer les ressources**, cliquez sur **Préparer**.

    ![Bouton pour préparer le déplacement du groupe de ressources source](./media/move-region-availability-zone/prepare-resource-group.png)

    Pendant le processus de préparation, Resource Mover génère des modèles Azure Resource Manager (ARM) à l’aide des paramètres du groupe de ressources. Les ressources à l’intérieur du groupe de ressources ne sont pas affectées.

> [!NOTE]
>  Après sa préparation, le groupe de ressources présente l’état *Lancement du déplacement en attente*. 

![Statut présentant l’état Initiation en attente](./media/move-region-availability-zone/initiate-resource-group-pending.png)

### <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source

Lancez le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Lancer le déplacement**
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**. Le groupe de ressources passe à l’état *Lancement du déplacement en cours*.
3. Après l’initialisation du déplacement, le groupe de ressources cible est créé, en fonction du modèle ARM généré. Le groupe de ressources source passe à l’état *Validation du déplacement en attente*.

![Statut présentant la validation du déplacement](./media/move-region-availability-zone/commit-move-pending.png)

Pour valider et terminer la procédure de déplacement :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Valider le déplacement**
2. Dans **Déplacer des ressources**, cliquez sur **Valider**.

> [!NOTE]
> Après la validation du déplacement, le groupe de ressources source présente l’état *Suppression de la source en attente*.


## <a name="add-a-target-availability-zone"></a>Ajouter une zone de disponibilité cible

Avant de déplacer les ressources restantes, nous allons définir une zone de disponibilité cible pour la machine virtuelle.

1. Dans la page **Entre les régions**, cliquez sur le lien dans la colonne **Configuration de destination** de la machine virtuelle que vous déplacez.

    ![Propriétés de machine virtuelle](./media/move-region-availability-zone/select-vm-settings.png)

3. Dans **Paramètres de configuration**, spécifiez le paramètre de la machine virtuelle de destination. Vous pouvez configurer une machine virtuelle dans la région cible comme suit :
    -  Créez une nouvelle ressource équivalente dans la région cible. À l’exception des paramètres que vous spécifiez, la ressource cible est créée avec les mêmes paramètres que la source.
    - Sélectionnez une machine virtuelle existante dans la région cible. 

4. Dans **Zones**, sélectionnez la zone dans laquelle vous souhaitez placer la machine virtuelle lors de son déplacement.

    Les modifications ne sont apportées qu’à la ressource que vous modifiez. Vous devez mettre à jour toutes les ressources dépendantes séparément.

5. Dans **SKU**, spécifiez le [niveau Azure](..//virtual-machines/sizes.md) que vous souhaitez affecter à la machine virtuelle cible.
6. Dans **Groupe à haute disponibilité**, sélectionnez un groupe à haute disponibilité si vous souhaitez que la machine virtuelle cible s’exécute dans un groupe à haute disponibilité dans la zone de disponibilité.
7. Sélectionnez **Enregistrer les modifications**.

    ![Paramètres de machine virtuelle](./media/move-region-availability-zone/vm-settings.png)


## <a name="prepare-resources-to-move"></a>Préparer les ressources à déplacer

Maintenant que le groupe de ressources source est déplacé, vous pouvez préparer le déplacement des autres ressources.

1. Dans **Entre régions**, sélectionnez les ressources que vous souhaitez préparer. 

    ![Page de sélection pour la préparation d’autres ressources](./media/move-region-availability-zone/prepare-other.png)

2. Sélectionnez **Préparer**. 

> [!NOTE]
> - Durant le processus de préparation, l’agent de mobilité Azure Site Recovery est installé sur les machines virtuelles, en vue de leur réplication.
> - Les données des machines virtuelles sont répliquées régulièrement dans la région cible. Cette opération n’a pas d’incidence sur la machine virtuelle source.
> - Le déplacement de ressources génère des modèles ARM pour les autres ressources sources.
> - Après leur préparation, les ressources présentent l’état *Lancement du déplacement en attente*.

![Page montrant les ressources avec l’état de lancement du déplacement en attente](./media/move-region-availability-zone/initiate-move-pending.png)

## <a name="initiate-the-move"></a>Lancer le déplacement

Les ressources préparées, vous pouvez maintenant lancer le déplacement. 

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Lancement du déplacement en attente*. Cliquez ensuite sur **Lancer le déplacement**
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**.

    ![Page pour initier le déplacement des ressources](./media/move-region-availability-zone/initiate-move.png)

3. Suivez la progression du déplacement dans la barre de notification.


> [!NOTE]
> - Pour les machines virtuelles, les machines virtuelles de réplication sont créées dans la région cible. La machine virtuelle source est arrêtée, et un temps d’interruption se produit (généralement en minutes).
> - Resource Mover recrée d’autres ressources à l’aide des modèles ARM préparés. Il n’y a généralement pas de temps d’arrêt.
> - Après leur préparation, les ressources présentent l’état *Validation du lancement en attente*.


![Page pour afficher les ressources dans un état d’attente de validation de déplacement](./media/move-region-availability-zone/resources-commit-move-pending.png)

## <a name="discard-or-commit"></a>Abandonner ou valider ?

Après le déplacement initial, vous pouvez décider de valider le déplacement ou de l’abandonner. 

- **Abandonner** : vous pouvez abandonner un déplacement si vous effectuez un test et que vous ne souhaitez pas déplacer réellement la ressource source. Si vous abandonnez le déplacement, la ressource reviendra à l’état *Lancement du déplacement en attente*.
- **Valider** : La validation termine le déplacement vers la région cible. Après sa validation, une ressource source se présente avec l’état de *Suppression de la source en attente*, et vous pouvez décider si vous voulez la supprimer.

## <a name="discard-the-move"></a>Abandonner le déplacement 

Vous pouvez abandonner le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Validation du déplacement en attente*, puis cliquez sur **Abandonner le déplacement**.
2. Dans **Abandonner le déplacement**, cliquez sur **Abandonner**.
3. Suivez la progression du déplacement dans la barre de notification.
 

> [!NOTE]
> Pour les machines virtuelles, après l’abandon des ressources, celles-ci présentent l’état *Lancement du déplacement en attente*.

## <a name="commit-the-move"></a>Valider le déplacement

Si vous voulez terminer la procédure de déplacement, validez le déplacement. 

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Validation du déplacement en attente*, puis cliquez sur **Valider le déplacement**.
2. Dans **Valider les ressources**, cliquez sur **Valider**.

    ![Page de validation des ressources permettant de terminer le déplacement](./media/move-region-availability-zone/commit-resources.png)

3. Suivez la progression de la validation dans la barre de notification.

> [!NOTE]
> - Après la validation du déplacement, les machines virtuelles arrêtent la réplication. La machine virtuelle source n’est pas affectée par la validation.
> - La validation n’a pas d’incidence sur les ressources réseau sources.
> - Après la validation du déplacement, les ressources présentent l’état de *Suppression de la source en attente*.

![Page montrant les ressources avec l’état *Suppression de la source en attente*](./media/move-region-availability-zone/delete-source-pending.png)

## <a name="configure-settings-after-the-move"></a>Configurer des paramètres après le déplacement

Le service Mobilité n’est pas désinstallé automatiquement sur les machines virtuelles. Désinstallez-le manuellement ou laissez-le si vous envisagez de déplacer à nouveau le serveur.


## <a name="delete-source-resources-after-commit"></a>Supprimer les ressources sources après la validation

Après le déplacement, vous pouvez, si vous le souhaitez, supprimer les ressources dans la région source.

1. Dans **Entre régions**, cliquez sur le nom de chaque ressource source que vous décidez de supprimer.
2. Dans la page des propriétés de chaque ressource, sélectionnez **Supprimer**.

## <a name="delete-additional-resources-created-for-move"></a>Supprimer les ressources supplémentaires créées pour le déplacement

Après le déplacement, vous pouvez supprimer manuellement la collection de déplacement ainsi que les ressources Site Recovery qui ont été créées.

- La collection de déplacement est masquée par défaut. Pour la voir, vous devez activer les ressources masquées.
- Le stockage de cache est équipé d’un verrou qui doit être supprimé avant de pouvoir procéder à la suppression.

Supprimez comme suit : 

1. Localisez les ressources dans le groupe de ressources ```RegionMoveRG-<sourceregion>-<target-region>```.
2. Vérifiez que toutes les machines virtuelles et les autres ressources sources de la région source ont été déplacées ou supprimées. Cette vérification permet de s’assurer qu’aucune ressource en attente ne les utilise.
2. Supprimer les ressources :

    - Le nom de la collection de déplacement est ```movecollection-<sourceregion>-<target-region>```.
    - Le nom du compte de stockage de cache est ```resmovecache<guid>```.
    - Le nom du coffre est ```ResourceMove-<sourceregion>-<target-region>-GUID```.

## <a name="next-steps"></a>Étapes suivantes

[Découvrez](about-move-process.md) le processus de déplacement.