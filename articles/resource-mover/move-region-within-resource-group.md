---
title: Déplacer des ressources vers une autre région avec le service Azure Resource Mover
description: Découvrez comment déplacer des ressources au sein d’un groupe de ressources vers une autre région avec le service Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 09/08/2020
ms.author: raynew
ms.openlocfilehash: 34064fe3fe88a34b0dd2430d7adec3ebcb17ebcc
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95528225"
---
# <a name="move-resources-across-regions-from-resource-group"></a>Déplacer des ressources entre régions (à partir d’un groupe de ressources)

Dans cet article explique comment déplacer des ressources au sein d’un groupe de ressources spécifique vers une autre région Azure. Dans le groupe de ressources, sélectionnez les ressources à déplacer. Ensuite, déplacez-les à l’aide du service [Azure Resource Mover](overview.md).

> [!IMPORTANT]
> Azure Resource Mover est actuellement en préversion publique.


## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un accès *Propriétaire* à l’abonnement dans lequel se trouvent les ressources que vous souhaitez déplacer.
    - La première fois que vous ajoutez une ressource pour un mappage de source et de destination spécifique dans un abonnement Azure, le service Resource Mover crée une [identité managée affectée par le système](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (anciennement Managed Service Identify, MSI), qui est approuvée par l’abonnement.
    - Afin de créer l’identité et lui affecter le rôle demandé (Contributeur ou Administrateur de l’accès utilisateur dans l’abonnement source), le compte que vous utilisez pour ajouter des ressources a besoin des autorisations *Propriétaire* sur l’abonnement. [Explorez en détail](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) les rôles Azure.
- L’abonnement a besoin d’un quota suffisant pour créer les ressources sources dans la région cible. Si ce n’est pas le cas, demandez des limites supplémentaires. [Plus d’informations](../azure-resource-manager/management/azure-subscription-service-limits.md)
- Vérifiez le tarif et les frais associés à la région cible vers laquelle vous déplacez des machines virtuelles. Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour vous aider.
- Vérifiez que les ressources que vous souhaitez déplacer sont prises en charge par le service Azure Resource Mover :
    - Machines virtuelles Azure et disques associés
    - Cartes réseau
    - Groupes à haute disponibilité
    - Réseaux virtuels Azure
    - Adresses IP publiques
    - Groupes de sécurité réseau (NSG)
    - Équilibreurs de charge internes et publics
    - Bases de données Azure SQL et pools élastiques


## <a name="check-vm-requirements"></a>Vérifier la configuration requise pour les machines virtuelles

1. Assurez-vous que les machines virtuelles que vous souhaitez déplacer sont prises en charge.

    - [Vérifiez](support-matrix-move-region-azure-vm.md#windows-vm-support) les machines virtuelles Windows prises en charge.
    - [Vérifiez](support-matrix-move-region-azure-vm.md#linux-vm-support) les machines virtuelles Linux et les versions du noyau prises en charge.
    - Contrôlez les paramètres de [calcul](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), de [stockage](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) et de [réseau](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings) pris en charge.
2. Assurez-vous que les machines virtuelles disposent des certificats racines approuvés les plus récents et d’une liste de révocation de certificats (CRL) mise à jour. 
    - Sur les machines virtuelles Azure exécutant Windows, installez les dernières mises à jour de Windows.
    - Sur les machines virtuelles exécutant Linux, suivez les instructions du distributeur Linux pour vérifier que la machine dispose des certificats et de la liste de révocation de certificats les plus récents. 
3. Autorisez une connexion sortante à partir des machines virtuelles :
    - Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connexion sortante, autorisez l’accès à ces [URL](support-matrix-move-region-azure-vm.md#url-access) :
    - Si vous utilisez des règles de groupe de sécurité réseau (NSG) pour contrôler la connexion sortante, créez ces [règles d’étiquette de service](support-matrix-move-region-azure-vm.md#nsg-rules).

## <a name="select-resources-to-move"></a>Sélectionner les ressources à déplacer

Sélectionnez les ressources que vous souhaitez déplacer. Vous déplacez des ressources vers une région cible au sein de l’abonnement de la région source. Si vous voulez changer d’abonnement, vous pouvez le faire après le déplacement des ressources.

1. Dans le portail Azure, ouvrez le groupe de ressources approprié.
2. Sélectionnez la page du groupe de ressources, sélectionnez les ressources que vous souhaitez déplacer.
3. Sélectionnez **Déplacer** > **Déplacer vers une autre région**.

    ![Sélection pour déplacer des ressources vers une autre région](./media/move-region-within-resource-group/select-move-region.png)
    
4. Dans **Source/destination**, sélectionnez la région cible vers laquelle vous souhaitez déplacer les ressources. Sélectionnez ensuite **Suivant**.


    ![Page de source et de destination pour sélectionner la région cible](./media/move-region-within-resource-group/source-target.png)


7. Dans **Ressources à déplacer**, sélectionnez **Suivant**.  
8. Dans **Sélectionnez des ressources**, sélectionnez la ressource que vous souhaitez déplacer. Vous pouvez uniquement ajouter des ressources prises en charge pour le déplacement. Ensuite, sélectionnez **Terminé**.
9. Dans **Déplacer des ressources**, sélectionnez **Suivant**. 
10. Dans **Vérifier + ajouter**, vérifiez les détails de la source et de la cible.
11. Confirmez que vous comprenez que les métadonnées relatives aux ressources déplacées seront stockées dans un groupe de ressources créé à cette fin, et que vous autorisez le service Resource Mover à créer une identité managée par le système pour accéder aux ressources de l’abonnement.
1. Cliquez sur **Continuer** pour commencer à ajouter les ressources.

    ![Page Résumé pour vérifier les détails et poursuivre le déplacement](./media/move-region-within-resource-group/summary.png)    

11. L’opération d’ajout de ressource démarre. Une fois l’opération terminée, les notifications indiquent que des ressources ont été ajoutées et que le déploiement a réussi.
13. Dans les notifications, sélectionnez **Ajout de ressources pour le déplacement**.

    ![Message affiché dans les notifications](./media/move-region-within-resource-group/notification.png)    


14. Après avoir sélectionné la notification, les ressources que vous avez sélectionnées sont ajoutées à une collection de déplacement dans le hub Azure Resource Mover.  Le service Resource Mover vous aide à vérifier les dépendances, puis à commencer à déplacer les ressources vers la région cible.

## <a name="resolve-dependencies"></a>Résoudre les erreurs de dépendance

Les ressources que vous déplacez s’affichent dans la page **Entre les régions**, dans un état *Préparation en attente*. Démarrez la validation comme suit :

1. Si des ressources affichent un message *Valider les dépendances* dans la colonne **Problèmes**, cliquez sur le bouton **Valider les dépendances**. Le processus de validation démarre.

    ![Bouton pour valider les dépendances](./media/move-region-within-resource-group/validate-dependencies.png)

2. Si des dépendances sont trouvées, cliquez sur **Ajouter des dépendances**. 
3. Dans **Ajouter des dépendances**, sélectionnez les ressources dépendantes > **Ajouter des dépendances**. Supervisez la progression dans les notifications.

    ![Bouton pour ajouter des dépendances](./media/move-region-within-resource-group/add-dependencies.png)

3. Ajoutez des dépendances supplémentaires si nécessaire, puis validez les dépendances en fonction des besoins. Sélectionnez **Actualiser** pour vous assurer que les ressources affichent un état à jour.

4. Dans la page **Entre régions**, vérifiez que les ressources présentent maintenant l’état *Préparation en attente*, sans aucun problème répertorié.

    ![Page pour afficher l’état de préparation en attente pour toutes les ressources](./media/move-region-within-resource-group/prepare-pending.png)

## <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source 

Pour pouvoir préparer et déplacer des ressources, le groupe de ressources source doit être présent dans la région cible. 

### <a name="prepare-to-move-the-source-resource-group"></a>Préparer le déplacement du groupe de ressources source

Préparez de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources source > **Préparer**.
2. Dans **Préparer les ressources**, sélectionnez **Préparer**.
1. 
    ![Bouton pour préparer le groupe de ressources source](./media/move-region-within-resource-group/prepare-source-resource-group.png)

    Pendant le processus de préparation, Resource Mover génère des modèles Azure Resource Manager (ARM) à l’aide des paramètres du groupe de ressources. Les ressources à l’intérieur du groupe de ressources ne sont pas affectées.
    
> [!NOTE]
>  Après sa préparation, le groupe de ressources présente l’état *Lancement du déplacement en attente*. Actualiser pour afficher l’état le plus récent.

![Statut présentant l’état Initiation en attente](./media/move-region-within-resource-group/initiate-resource-group-pending.png)

### <a name="move-the-source-resource-group"></a>Déplacer le groupe de ressources source

Lancez le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Lancer le déplacement**
2. Dans **Déplacer des ressources**, sélectionnez **Lancer le déplacement**. Le groupe de ressources passe à l’état *Lancement du déplacement en cours*.
3. Après l’initialisation du déplacement, le groupe de ressources cible est créé, en fonction du modèle ARM généré. Le groupe de ressources source passe à l’état *Validation du déplacement en attente*.

![Statut présentant la validation du déplacement](./media/move-region-availability-zone/commit-move-pending.png)

Pour valider et terminer la procédure de déplacement :

1. Dans **Entre régions**, sélectionnez le groupe de ressources > **Valider le déplacement**.
2. Dans **Déplacer des ressources**, sélectionnez **Valider**.

> [!NOTE]
> Après la validation du déplacement, le groupe de ressources source présente l’état *Suppression de la source en attente*.

## <a name="modify-target-settings"></a>Modifier les paramètres cibles

Si vous ne souhaitez pas déplacer une ressource source, vous pouvez effectuer l’une des opérations suivantes :

- Créez dans la région cible une ressource avec le même nom et les mêmes paramètres que la ressource dans la région source.
- Créez une ressource équivalente dans la région cible. À l’exception des paramètres que vous spécifiez, la ressource cible est créée avec les mêmes paramètres que la source.
- Utilisez une ressource existante dans la région cible.

Modifiez un paramètre comme suit :

1. Pour modifier un paramètre, sélectionnez l’entrée dans la colonne **Configuration de la destination** pour la ressource.
2. Dans la page **Configuration de la destination**, spécifiez les paramètres cibles que vous souhaitez utiliser.
    Les modifications ne sont apportées qu’à la ressource que vous modifiez. Vous devez mettre à jour toutes les ressources dépendantes séparément.   
    
Les paramètres exacts que vous modifiez dépendent du type de ressource. [Apprenez-en davantage](modify-target-settings.md) sur la modification des paramètres cibles.

## <a name="prepare-resources-to-move"></a>Préparer les ressources à déplacer

Maintenant que le groupe de ressources source est déplacé, vous pouvez préparer le déplacement des autres ressources.

1. Dans **Entre régions**, sélectionnez les ressources que vous souhaitez préparer. 

    ![Page de sélection pour la préparation d’autres ressources](./media/move-region-availability-zone/prepare-other.png)

2. Sélectionnez **Préparer**.

> [!NOTE]
> - Durant le processus de préparation, l’agent de mobilité Azure Site Recovery est installé sur les machines virtuelles, en vue de la réplication de celles-ci.
> - Les données des machines virtuelles sont répliquées régulièrement dans la région cible. Cette opération n’a pas d’incidence sur la machine virtuelle source.
> - Le déplacement de ressources génère des modèles ARM pour les autres ressources sources.
> - Après leur préparation, les ressources présentent l’état *Lancement du déplacement en attente*.

![Page montrant les ressources avec l’état de lancement du déplacement en attente](./media/move-region-availability-zone/initiate-move-pending.png)

## <a name="initiate-the-move"></a>Lancer le déplacement

Les ressources préparées, vous pouvez maintenant lancer le déplacement.

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Lancement du déplacement en attente*. Sélectionnez ensuite **Lancer le déplacement**.
2. Dans **Déplacer des ressources**, sélectionnez **Lancer le déplacement**.

    ![Sélectionnez le bouton Lancer le déplacement](./media/move-region-within-resource-group/initiate-move.png)

3. Suivez la progression du déplacement dans la barre de notification.


> [!NOTE]
> - Pour les machines virtuelles, les machines virtuelles de réplication sont créées dans la région cible. La machine virtuelle source est arrêtée, et un temps d’interruption se produit (généralement en minutes).<br/>
> - Resource Mover recrée d’autres ressources à l’aide des modèles ARM préparés. Il n’y a généralement pas de temps d’arrêt.<br/> 
> - Pour les équilibrages de charge, les règles de traduction d’adresses réseau (NAT) ne sont pas copiées. Créez-les dans la région cible après avoir validé le déplacement.
> - Pour les adresses IP publiques, l’étiquette du nom DNS n’est pas copiée. Recréez l’étiquette après avoir validé le déplacement.
> - Après leur préparation, les ressources présentent l’état *Validation du lancement en attente*.


## <a name="discard-or-commit"></a>Abandonner ou valider ?

Après le déplacement initial, vous pouvez décider de valider le déplacement ou de l’abandonner. 

- **Abandonner** : vous pouvez abandonner un déplacement si vous effectuez un test et que vous ne souhaitez pas déplacer réellement la ressource source. Si vous abandonnez le déplacement, la ressource reviendra à l’état *Lancement du déplacement en attente*.
- **Valider** : La validation termine le déplacement vers la région cible. Après sa validation, une ressource source se présente avec l’état de *Suppression de la source en attente*, et vous pouvez décider si vous voulez la supprimer.


## <a name="discard-the-move"></a>Abandonner le déplacement 

Vous pouvez abandonner le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Validation du déplacement en attente*, puis cliquez sur **Abandonner le déplacement**.
2. Dans **Abandonner le déplacement**, sélectionnez **Abandonner**.
3. Suivez la progression du déplacement dans la barre de notification.
4. Lorsque les notifications indiquent que le processus de déplacement a réussi, sélectionnez **Actualiser**. 

> [!NOTE]
> Pour les machines virtuelles, après l’abandon des ressources, les machines présentent l’état *Lancement du déplacement en attente*.

## <a name="commit-the-move"></a>Valider le déplacement

Si vous voulez terminer la procédure de déplacement, validez le déplacement. 


1. Dans **Entre régions**, sélectionnez les ressources avec l’état *Validation du déplacement en attente*, puis sélectionnez **Valider le déplacement**.
2. Dans **Valider les ressources**, sélectionnez **Valider**.

    ![Page de validation des ressources permettant de terminer le déplacement](./media/move-region-within-resource-group/commit-resources.png)

3. Suivez la progression de la validation dans la barre de notification.

> [!NOTE]
> - Après la validation du déplacement, les machines virtuelles arrêtent la réplication. La machine virtuelle source n’est pas affectée par la validation.
> - La validation n’a pas d’incidence sur les ressources réseau sources.
> - Après la validation du déplacement, les ressources présentent l’état de *Suppression de la source en attente*.

## <a name="configure-settings-after-the-move"></a>Configurer des paramètres après le déplacement

1. Étant donné que les étiquettes de nom DNS ne sont pas copiées pour les adresses IP publiques, une fois le déplacement effectué, accédez aux ressources cibles et mettez à jour l’étiquette. 
2. Pour les équilibreurs de charge internes, étant donné que les règles NAT ne sont pas copiées, accédez aux ressources créées dans la région cible et mettez à jour les règles NAT.
3. Le service Mobilité n’est pas désinstallé automatiquement sur les machines virtuelles. Désinstallez-le manuellement ou laissez-le si vous envisagez de déplacer à nouveau le serveur.
## <a name="delete-source-resources-after-commit"></a>Supprimer les ressources sources après la validation

Après le déplacement, vous pouvez, si vous le souhaitez, supprimer les ressources dans la région source. 

1. Dans **Entre régions**, sélectionnez le nom de chaque ressource source que vous souhaitez supprimer.
2. Dans la page des propriétés de chaque ressource, sélectionnez **Supprimer**.

## <a name="delete-additional-resources-created-for-move"></a>Supprimer les ressources supplémentaires créées pour le déplacement

Après le déplacement, vous pouvez supprimer manuellement la collection de déplacement ainsi que les ressources Site Recovery qui ont été créées.

- La collection de déplacement est masquée par défaut. Pour la voir, vous devez activer les ressources masquées.
- Le stockage de cache est équipé d’un verrou qui doit être supprimé avant de pouvoir procéder à la suppression.

Supprimez comme suit : 

1. Localisez les ressources dans le groupe de ressources ```RegionMoveRG-<sourceregion>-<target-region>```, dans la région source.
2. Vérifiez que la machine virtuelle et toutes les autres ressources sources de la collection de déplacement ont été déplacées ou supprimées. Cette vérification permet de s’assurer qu’aucune ressource en attente ne les utilise.
2. Supprimer les ressources :

    - Le nom de la collection de déplacement est ```movecollection-<sourceregion>-<target-region>```.
    - Le nom du compte de stockage de cache est ```resmovecache<guid>```.
    - Le nom du coffre est ```ResourceMove-<sourceregion>-<target-region>-GUID```.

## <a name="next-steps"></a>Étapes suivantes


[Découvrez](about-move-process.md) le processus de déplacement.