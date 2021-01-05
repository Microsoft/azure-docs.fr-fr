---
title: Déplacer des machines virtuelles Azure d’une région à une autre avec Azure Resource Mover
description: Apprendre à déplacer des machines virtuelles Azure dans une autre région avec Azure Resource Mover
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 6f21db00ecc9ff2668698f53a4d20f5bae525721
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95520439"
---
# <a name="tutorial-move-azure-vms-across-regions"></a>Tutoriel : Déplacer des machines virtuelles Azure d’une région à une autre

Dans cet article, apprenez à déplacer dans une autre région Azure des machines virtuelles Azure ainsi que les ressources réseau et de stockage associées, en utilisant [Azure Resource Mover](overview.md).

> [!NOTE]
> Azure Resource Mover est actuellement en préversion publique.


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifier les prérequis et la configuration exigée.
> * Sélectionner les ressources que vous souhaitez déplacer.
> * Résoudre des dépendances de ressources.
> * Préparer et déplacer le groupe de ressources source. 
> * Préparer et déplacer les autres ressources.
> * Décider si vous souhaitez abandonner ou valider le déplacement. 
> * Le cas échéant, supprimer les ressources dans la région source après le déplacement.

> [!NOTE]
> Les tutoriels enseignent le moyen le plus rapide de tester un scénario et d’utiliser les options par défaut. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer. Puis, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Prérequis

-  Veillez à disposer d’un accès *Propriétaire* sur l’abonnement contenant les ressources que vous souhaitez déplacer.
    - La première fois que vous ajoutez une ressource pour une paire source et destination spécifique dans un abonnement Azure, Resource Mover crée une [identité managée affectée par le système](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (anciennement Managed Service Identify, MSI) qui est approuvée par l’abonnement.
    - Afin de créer l’identité et lui affecter le rôle demandé (Contributeur ou Administrateur de l’accès utilisateur dans l’abonnement source), le compte que vous utilisez pour ajouter des ressources a besoin des autorisations *Propriétaire* sur l’abonnement. [Explorez en détail](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) les rôles Azure.
- L’abonnement a besoin d’un quota suffisant pour créer les ressources que vous déplacez dans la région cible. S’il n’a pas le quota, [demandez des limites supplémentaires](../azure-resource-manager/management/azure-subscription-service-limits.md).
- Vérifiez le tarif et les frais associés à la région cible vers laquelle vous déplacez des machines virtuelles. Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour vous aider.
    

## <a name="check-vm-requirements"></a>Vérifier la configuration requise pour les machines virtuelles

1. Assurez-vous que les machines virtuelles que vous souhaitez déplacer sont prises en charge.

    - [Vérifiez](support-matrix-move-region-azure-vm.md#windows-vm-support) les machines virtuelles Windows prises en charge.
    - [Vérifiez](support-matrix-move-region-azure-vm.md#linux-vm-support) les machines virtuelles Linux et les versions du noyau prises en charge.
    - Contrôlez les paramètres de [calcul](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), de [stockage](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) et de [réseau](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings) pris en charge.
2. Vérifiez que les machines virtuelles que vous souhaitez déplacer sont activées.
3. Assurez-vous que les machines virtuelles disposent des certificats racine approuvés les plus récents et d’une liste de révocation de certificats (CRL) mise à jour. Pour ce faire :
    - Sur les machines virtuelles Windows, installez les dernières mises à jour Windows.
    - Sur les machines virtuelles Linux, suivez les instructions du distributeur afin que les machines disposent des derniers certificats et des listes de révocation de certificats les plus récentes. 
4. Autorisez une connexion sortante à partir des machines virtuelles :
    - Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connexion sortante, autorisez l’accès à ces [URL](support-matrix-move-region-azure-vm.md#url-access) :
    - Si vous utilisez des règles de groupe de sécurité réseau (NSG) pour contrôler la connexion sortante, créez ces [règles d’étiquette de service](support-matrix-move-region-azure-vm.md#nsg-rules).

## <a name="select-resources"></a>Sélection des ressources 

Sélectionnez les ressources que vous souhaitez déplacer.

- Tous les types de ressources pris en charge dans les groupes de ressources de la région source sélectionnée sont affichés.
- Les ressources qui ont déjà été ajoutées pour passer d’une région à une autre ne sont pas affichées.
- Vous déplacez des ressources dans une région cible du même abonnement que la région source. Si vous voulez changer d’abonnement, vous pouvez le faire après le déplacement des ressources.

1. Dans le portail Azure, recherchez *resource mover*. Ensuite, sous **Services**, sélectionnez **Azure Resource Mover**.

    ![Résultats de la recherche de Resource Mover dans le portail Azure](./media/tutorial-move-region-virtual-machines/search.png)

2. Dans **Vue d’ensemble**, cliquez sur **Démarrer**.

    ![Bouton pour ajouter des ressources à déplacer dans une autre région](./media/tutorial-move-region-virtual-machines/get-started.png)

3. Dans **Déplacer des ressources** > **Source + destination**, sélectionnez l’abonnement et la région sources.
4. Dans **Destination**, sélectionnez la région vers laquelle vous souhaitez déplacer les machines virtuelles. Cliquez ensuite sur **Suivant**.

    ![Page de sélection de la région source et de la région de destination](./media/tutorial-move-region-virtual-machines/source-target.png)

6. Dans **Ressources à déplacer**, cliquez sur **Sélectionner des ressources**.
7. Dans **Sélectionner des ressources**, sélectionnez la machine virtuelle. Vous pouvez uniquement ajouter des [ressources prises en charge pour le déplacement](#check-vm-requirements). Cliquez ensuite sur **Terminé**.

    ![Page de sélection des machines virtuelles à déplacer](./media/tutorial-move-region-virtual-machines/select-vm.png)

8.  Dans **Ressources à déplacer**, cliquez sur **Suivant**.
9. Dans **Vérifier + ajouter**, contrôlez les paramètres de source et de destination. 

    ![Page de vérification des paramètres et de la poursuite du déplacement](./media/tutorial-move-region-virtual-machines/review.png)
10. Cliquez sur **Continuer** pour commencer à ajouter les ressources.
11. Une fois le processus d’ajout terminé, cliquez sur **Ajout de ressources pour le déplacement** dans l’icône de notification.
12. Après avoir cliqué sur la notification, passez en revue les ressources sur la page **Entre régions**.

> [!NOTE]
> - Les ressources ajoutées présentent l’état *Préparation en attente*.
> - Si vous souhaitez supprimer une ressource d’une collection de déplacement, la méthode permettant d’effectuer cette opération dépend de l’étape où vous vous trouvez dans la procédure de déplacement. [Plus d’informations](remove-move-resources.md)

## <a name="resolve-dependencies"></a>Résoudre les erreurs de dépendance

1. Si des ressources affichent un message *Valider les dépendances* dans la colonne **Problèmes**, cliquez sur le bouton **Valider les dépendances**. Le processus de validation démarre.
2. Si des dépendances sont trouvées, cliquez sur **Ajouter des dépendances**. 
3. Dans **Ajouter des dépendances**, sélectionnez les ressources dépendantes > **Ajouter des dépendances**. Supervisez la progression dans les notifications.

    ![Ajout de dépendances](./media/tutorial-move-region-virtual-machines/add-dependencies.png)

4. Ajoutez des dépendances supplémentaires si nécessaire, puis validez de nouveau les dépendances. 
    ![Page d’ajout des dépendances supplémentaires](./media/tutorial-move-region-virtual-machines/add-additional-dependencies.png)

4. Dans la page **Entre régions**, vérifiez que les ressources présentent maintenant l’état *Préparation en attente*, sans aucun problème répertorié.

    ![Page montrant les ressources avec l’état de préparation en attente](./media/tutorial-move-region-virtual-machines/prepare-pending.png)

> [!NOTE]
> Si vous souhaitez modifier les paramètres cibles avant de commencer le déplacement, sélectionnez le lien dans la colonne **Configuration de la destination** pour la ressource, puis modifiez les paramètres. Si vous modifiez les paramètres de la machine virtuelle cible, la taille de cette machine ne doit pas être inférieure à celle de la machine virtuelle source.  

## <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source 

Avant de pouvoir préparer et déplacer une machine virtuelle, le groupe de ressources de cette machine virtuelle doit être présent dans la région cible. 

### <a name="prepare-to-move-the-source-resource-group"></a>Préparer le déplacement du groupe de ressources source

Pendant le processus de préparation, Resource Mover génère des modèles Azure Resource Manager (ARM) à l’aide des paramètres du groupe de ressources. Les ressources à l’intérieur du groupe de ressources ne sont pas affectées.

Préparez de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources source > **Préparer**.
2. Dans **Préparer les ressources**, cliquez sur **Préparer**.

    ![Préparer un groupe de ressources](./media/tutorial-move-region-virtual-machines/prepare-resource-group.png)

> [!NOTE]
> Après sa préparation, le groupe de ressources présente l’état *Lancement du déplacement en attente*. 

 
### <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source

Lancez le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Lancer le déplacement**
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**. Le groupe de ressources passe à l’état *Lancement du déplacement en cours*.
3. Après l’initialisation du déplacement, le groupe de ressources cible est créé, en fonction du modèle ARM généré. Le groupe de ressources source passe à l’état *Validation du déplacement en attente*.

    ![Cliquer sur le bouton Lancer le déplacement](./media/tutorial-move-region-virtual-machines/commit-move-pending.png)

Pour valider et terminer la procédure de déplacement :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Valider le déplacement**.
2. Dans **Déplacer des ressources**, cliquez sur **Valider**.

> [!NOTE]
> Après la validation du déplacement, le groupe de ressources source présente l’état *Suppression de la source en attente*.

## <a name="prepare-resources-to-move"></a>Préparer les ressources à déplacer

Maintenant que le groupe de ressources source est déplacé, vous pouvez préparer le déplacement des autres ressources.

1. Dans **Entre régions**, sélectionnez les ressources que vous souhaitez préparer. 

    ![Page de sélection pour la préparation d’autres ressources](./media/tutorial-move-region-virtual-machines/prepare-other.png)

2. Sélectionnez **Préparer**. 

> [!NOTE]
> - Durant le processus de préparation, l’agent de mobilité Azure Site Recovery est installé sur les machines virtuelles, en vue de leur réplication.
> - Les données des machines virtuelles sont répliquées régulièrement dans la région cible. Cette opération n’a pas d’incidence sur la machine virtuelle source.
> - Le déplacement de ressources génère des modèles ARM pour les autres ressources sources.
> - Après leur préparation, les ressources présentent l’état *Lancement du déplacement en attente*.

![Page montrant les ressources avec l’état de lancement du déplacement en attente](./media/tutorial-move-region-virtual-machines/initiate-move-pending.png)


## <a name="initiate-the-move"></a>Lancer le déplacement

Les ressources préparées, vous pouvez maintenant lancer le déplacement. 

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Lancement du déplacement en attente*. Cliquez ensuite sur **Lancer le déplacement**.
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**.

    ![Cliquer sur le bouton Lancer le déplacement](./media/tutorial-move-region-virtual-machines/initiate-move.png)

3. Suivez la progression du déplacement dans la barre de notification.

> [!NOTE]
> - Pour les machines virtuelles, les machines virtuelles de réplication sont créées dans la région cible. La machine virtuelle source est arrêtée, et un temps d’interruption se produit (généralement en minutes).
> - Resource Mover recrée d’autres ressources à l’aide des modèles ARM préparés. Il n’y a généralement pas de temps d’arrêt.
> - Après leur déplacement, les ressources présentent l’état *Validation du lancement en attente*.

![Page montrant les ressources avec l’état *Suppression de la source en attente*](./media/tutorial-move-region-virtual-machines/delete-source-pending.png)


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
> Après l’abandon des ressources, les machines virtuelles présentent l’état *Lancement du déplacement en attente*.

## <a name="commit-the-move"></a>Valider le déplacement

Si vous voulez terminer la procédure de déplacement, validez le déplacement. 

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Validation du déplacement en attente*, puis cliquez sur **Valider le déplacement**.
2. Dans **Valider les ressources**, cliquez sur **Valider**.

    ![Page de validation des ressources permettant de terminer le déplacement](./media/tutorial-move-region-virtual-machines/commit-resources.png)

3. Suivez la progression de la validation dans la barre de notification.

> [!NOTE]
> - Après la validation du déplacement, les machines virtuelles arrêtent la réplication. La machine virtuelle source n’est pas affectée par la validation.
> - La validation n’a pas d’incidence sur les ressources réseau sources.
> - Après la validation du déplacement, les ressources présentent l’état de *Suppression de la source en attente*.

![Page montrant les ressources avec l’état *Suppression de la source en attente*](./media/tutorial-move-region-virtual-machines/delete-source-pending.png)


## <a name="configure-settings-after-the-move"></a>Configurer des paramètres après le déplacement

- Le service Mobilité n’est pas désinstallé automatiquement sur les machines virtuelles. Désinstallez-le manuellement ou laissez-le si vous envisagez de déplacer à nouveau le serveur.
- Modifiez les règles de contrôle d’accès en fonction du rôle (Azure RBAC) après le déplacement.

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

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Avez déplacé des machines virtuelles Azure dans une autre région Azure.
> * Avez déplacé les ressources associées aux machines virtuelles dans une autre région.

À présent, vous allez vous intéresser au déplacement des pools élastiques et des bases de données Azure SQL vers une autre région.

> [!div class="nextstepaction"]
> [Déplacer des ressources Azure SQL](./tutorial-move-region-sql.md)