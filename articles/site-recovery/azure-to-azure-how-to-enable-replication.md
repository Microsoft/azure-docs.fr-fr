---
title: Configurer la réplication pour les machines virtuelles Azure dans Azure Site Recovery
description: Découvrez comment configurer la réplication de machines virtuelles Azure dans une autre région à l’aide de Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 04/29/2018
ms.openlocfilehash: fe5feed4bb6f9b84a3f161692310922f7a6d2f00
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424785"
---
# <a name="replicate-azure-vms-to-another-azure-region"></a>Répliquer des machines virtuelles Azure dans une autre région Azure


Cet article décrit comment activer la réplication de machines virtuelles de Azure, d’une région Azure à l’autre.

## <a name="before-you-start"></a>Avant de commencer

Cet article part du principe que vous avez préparé le déploiement Site Recovery, comme décrit dans le [tutoriel sur la reprise d’activité Azure vers Azure](azure-to-azure-tutorial-enable-replication.md).

Les prérequis doivent être en place et vous devez avoir créé un coffre Recovery Services.


## <a name="enable-replication"></a>Activer la réplication

Activez la réplication. Cette procédure suppose que la région principale Azure est Asie Est et que la région secondaire est Asie Sud-Est.

1. Dans le coffre, cliquez sur **+ Répliquer**.
2. Notez les champs suivants :
   - **Source** : point d’origine des machines virtuelles, qui dans ce cas est **Azure**.
   - **Emplacement source**  : région Azure où vous souhaitez protéger vos machines virtuelles. Dans notre exemple, l’emplacement source est « Asie Est »
   - **Modèle de déploiement**  : modèle de déploiement Azure des machines sources.
   - **Abonnement source**  : abonnement auquel appartiennent vos machines virtuelles sources. Il peut s’agir de n’importe quel abonnement au sein du même locataire Azure Active Directory où se trouve votre coffre Recovery services.
   - **Groupe de ressources** : groupe de ressources auquel appartiennent vos machines virtuelles sources. Toutes les machines virtuelles du groupe de ressources sélectionné sont répertoriées pour la protection à l’étape suivante.

     ![Capture d'écran mettant en évidence les champs nécessaires à la configuration de la réplication.](./media/site-recovery-replicate-azure-to-azure/enabledrwizard1.png)

3. Dans **Machines virtuelles > Sélectionner les machines virtuelles** , cliquez sur chaque machine virtuelle à répliquer. Vous pouvez uniquement sélectionner les machines pour lesquelles la réplication peut être activée. Cliquez ensuite sur **OK**.
    ![Capture d'écran montrant où sélectionner les machines virtuelles.](./media/site-recovery-replicate-azure-to-azure/virtualmachine_selection.png)

4. Dans **Paramètres** , vous pouvez éventuellement configurer les paramètres du site cible :

   - **Emplacement cible**  : Emplacement où vos données de machines virtuelles sources sont répliquées. En fonction de l’emplacement des machines virtuelles sélectionné, Site Recovery vous fournit la liste des régions cibles appropriées. Nous vous recommandons de conserver le même emplacement cible que l’emplacement du coffre Recovery Services.
   - **Abonnement cible**  : abonnement cible utilisé pour la reprise d’activité. Par défaut, l’abonnement cible sera identique à l’abonnement source.
   - **Groupe de ressources cible**  : groupe de ressources auquel appartiennent toutes vos machines virtuelles répliquées.
       - Par défaut, Site Recovery crée un groupe de ressources dans la région cible dont le nom porte un suffixe « asr ».
       - Si le groupe de ressources créé par Site Recovery existe déjà, il est réutilisé.
       - Vous pouvez personnaliser les paramètres du groupe de ressources.
       - L’emplacement du groupe de ressources cible peut être n’importe quelle région Azure à l’exception de la région dans laquelle les machines virtuelles sources sont hébergées.
   - **Réseau virtuel cible** : Par défaut, Site Recovery crée un réseau virtuel dans la région cible dont le nom porte un suffixe « asr ». Il est mappé à votre réseau source et utilisé pour toute protection ultérieure. [En savoir plus](./azure-to-azure-network-mapping.md) sur le mappage réseau.
   - **Comptes de stockage cibles (la machine virtuelle source n’utilise pas de disques managés)**  : par défaut, Site Recovery crée un compte de stockage cible avec la même configuration que celle du compte de stockage de machines virtuelles source. Si le compte de stockage existe déjà, il est réutilisé.
   - **Disques managés de réplica (la machine virtuelle source utilise des disques managés)**  : Site Recovery crée des disques managés de réplica dans la région cible pour refléter les disques managés de la machine virtuelle source du même type de stockage (Standard ou Premium).
   - **Comptes de stockage de cache**  : Site Recovery a besoin d’un compte de stockage supplémentaire appelé « stockage de cache » dans la région source. Toutes les modifications effectuées sur les machines virtuelles sources sont suivies et envoyées au compte de stockage de cache avant leur réplication vers l’emplacement cible. Ce compte de stockage doit être Standard.
   - **Groupes à haute disponibilité cibles**  : par défaut, Site Recovery crée un groupe à haute disponibilité dans la région cible dont le nom porte un suffixe « asr », pour les machines virtuelles qui font partie d’un groupe à haute disponibilité dans la région source. Si le groupe à haute disponibilité créé par Site Recovery existe déjà, il est réutilisé.
   - **Zones de disponibilité cibles**  : par défaut, Site Recovery affecte à la région cible le même nombre de zones que la région source, si la région cible prend en charge les zones de disponibilité.

     Si la région cible ne prend pas en charge les zones de disponibilité, les machines virtuelles cibles sont configurées comme des instances uniques par défaut. Si nécessaire, vous pouvez configurer ces machines virtuelles comme faisant partie de groupes à haute disponibilité dans la région cible, en cliquant sur « Personnaliser ».

     >[!NOTE]
     >Vous ne pouvez plus modifier le type de disponibilité (instance unique, groupe à haute disponibilité ou zone de disponibilité) une fois que vous avez activé la réplication. Vous devez désactiver puis réactiver la réplication pour modifier le type de disponibilité.
     >

   - **Stratégie de réplication**  : définit les paramètres de l’historique de conservation des points de récupération et la fréquence des captures instantanées de cohérence des applications. Par défaut, Azure Site Recovery crée une stratégie de réplication avec les paramètres par défaut de « 24 heures » pour la rétention des points de récupération, et de « 4 heures » pour la fréquence des captures instantanées de cohérence des applications.

     ![Activer la réplication](./media/site-recovery-replicate-azure-to-azure/enabledrwizard3.PNG)

### <a name="enable-replication-for-added-disks"></a>Activer la réplication pour des disques ajoutés

Si vous ajoutez des disques à une machine virtuelle Azure pour laquelle la réplication est activée, voici ce qui se produit :
-   L’intégrité de la réplication pour la machine virtuelle affiche un avertissement et une note vous informe qu’un ou plusieurs disques sont disponibles pour la protection.
-   Si vous activez la protection pour les disques ajoutés, l’avertissement disparaît après la réplication initiale du disque.
-   Si vous choisissez de ne pas activer la réplication du disque, vous pouvez masquer l'avertissement.


    ![Nouveau disque ajouté](./media/azure-to-azure-how-to-enable-replication/newdisk.png)

Pour activer la réplication d'un disque ajouté, procédez comme suit :

1.  Dans le coffre > **Éléments répliqués** , cliquez sur la machine virtuelle à laquelle vous avez ajouté le disque.
2.  Cliquez sur **Disques** , puis sélectionnez le disque de données pour lequel vous voulez activer la réplication (ces disques présentent un état **Non protégé** ).
3.  Dans **Détails du disque** , cliquez sur **Activer la réplication**.

    ![Activer la réplication d'un disque ajouté](./media/azure-to-azure-how-to-enable-replication/enabled-added.png)

Une fois que le travail d’activation de la réplication s’exécute et que la réplication initiale est terminée, le problème d’avertissement sur l’intégrité de la réplication pour le disque a disparu.



## <a name="customize-target-resources"></a>Personnaliser les ressources cibles

Vous pouvez modifier les paramètres de cible par défaut utilisés par Site Recovery.

1. Cliquez sur **Personnaliser :** à côté de « Abonnement cible » pour modifier l’abonnement cible par défaut. Sélectionnez l’abonnement dans la liste de tous ceux disponibles, dans le même locataire Azure Active Directory (AAD).

2. Cliquez sur **Personnaliser :** pour modifier les paramètres par défaut :
    - Dans **Groupe de ressources cible** , sélectionnez le groupe de ressources dans la liste de tous les groupes de ressources à l’emplacement cible de l’abonnement.
    - Dans **Réseau virtuel cible** , sélectionnez le réseau à partir d’une liste de tous les réseaux virtuels dans l’emplacement cible.
    - Dans **Groupe à haute disponibilité** , vous pouvez ajouter les paramètres de groupe à haute disponibilité de la machine virtuelle, s’ils font partie d’un groupe à haute disponibilité dans la zone source.
    - Dans **Comptes de stockage cible** , sélectionnez le compte que vous souhaitez utiliser.

        ![Capture d'écran montrant comment personnaliser les paramètres de l'abonnement cible.](./media/site-recovery-replicate-azure-to-azure/customize.PNG)
3. Cliquez sur **Personnaliser :** pour modifier les paramètres de réplication.
4. Dans **Cohérence multimachine virtuelle** , sélectionnez les machines virtuelles à répliquer ensemble.
    - Toutes les machines d’un groupe de réplication ont des points de récupération cohérents après incident et avec les applications lorsqu’elles basculent.
    - L’activation de la cohérence multimachine virtuelle peut affecter les performances de la charge de travail (car elle utilise intensivement le processeur). Il convient de l’activer uniquement si les machines exécutent la même charge de travail et si vous avez besoin de cohérence sur plusieurs machines virtuelles.
    - Par exemple, si une application a 2 machines virtuelles SQL Server et deux serveurs web, alors vous devez ajouter uniquement les machines virtuelles SQL Server à un groupe de réplication.
    - Vous pouvez choisir d’avoir un maximum de 16 machines virtuelles dans un groupe de réplication.
    - Si vous activez la cohérence multimachine virtuelle, les machines du groupe de réplication communiquent entre elles sur le port 20004.
    - Vérifiez qu’aucun dispositif de pare-feu ne bloque la communication interne entre les machines virtuelles sur le port 20004.
    - Si vous voulez que les machines virtuelles Linux fassent partie d’un groupe de réplication, vérifiez que le trafic sortant sur le port 20004 est ouvert manuellement conformément aux instructions de la version Linux spécifique.
![Capture d'écran représentant les paramètres de la cohérence multimachine virtuelle.](./media/site-recovery-replicate-azure-to-azure/multivmsettings.PNG)

5. Cliquez sur **Créer une ressource cible** > **Activer la réplication**.
6. Une fois que les machines virtuelles sont activées pour la réplication, vous pouvez vérifier leur état d’intégrité sous **Éléments répliqués**

>[!NOTE]
>
> - Pendant la réplication initiale, l’état des peut prendre un certain temps à s’actualiser à défaut de progression. Cliquez sur le bouton **Actualiser** pour obtenir le dernier état.
> - Si aucun point de récupération n’a été généré au cours des 60 dernières minutes, l’intégrité de la réplication de la machine virtuelle devient critique.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](site-recovery-test-failover-to-azure.md) sur l’exécution d’un test de basculement.
