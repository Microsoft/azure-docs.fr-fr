---
title: Déplacer des machines virtuelles Azure vers une autre région Azure à l’aide d’Azure Site Recovery
description: Utilisez Azure Site Recovery pour déplacer des machines virtuelles Azure d’une région Azure vers une autre.
services: site-recovery
author: Sharmistha-Rai
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: sharrai
ms.custom: MVC
ms.openlocfilehash: 076adbfd4cecf7dae9ffc490e911fcb7ffce48e6
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394830"
---
# <a name="move-vms-to-another-azure-region"></a>Déplacer des machines virtuelles vers une autre région Azure

Il existe des scénarios dans lesquels vous pourriez souhaiter déplacer vos machines virtuelles Azure IaaS existantes d’une région à une autre. Par exemple, vous souhaitez améliorer la fiabilité et la disponibilité de vos machines virtuelles existantes, améliorer la facilité de gestion, ou effectuer un déplacement pour des raisons de gouvernance. Pour plus d’informations, consultez la [présentation du déplacement de machines virtuelles Azure](azure-to-azure-move-overview.md). 

Vous pouvez utiliser le service [Azure Site Recovery](site-recovery-overview.md) pour déplacer des machines virtuelles Azure vers une région secondaire.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> 
> * Vérifier les prérequis pour le déplacement
> * Préparer les machines virtuelles sources et la région cible
> * Copier les données et activer la réplication
> * Tester la configuration et effectuer le déplacement
> * Supprimer les ressources dans la région source


> [!IMPORTANT]
> Pour déplacer des machines virtuelles Azure vers une autre région, nous recommandons l’utilisation de [Azure Resource Mover](../resource-mover/tutorial-move-region-virtual-machines.md). Azure Resource Mover est en préversion publique et fournit ce qui suit :
> - Un hub unique pour déplacer des ressources entre les régions
> - Une réduction de la complexité et du temps de déplacement. Tout ce dont vous avez besoin se trouve dans un même emplacement.
> - Une expérience simple et cohérente pour le déplacement des différents types de ressources Azure.
> - Un moyen simple d’identifier les dépendances des ressources que vous souhaitez déplacer. Cela vous permet de déplacer les ressources associées ensemble, pour que tout fonctionne comme prévu dans la région cible après le déplacement.
> - Le nettoyage automatique des ressources dans la région source, si vous souhaitez les supprimer après le déplacement
> - Tests. Vous pouvez essayer un déplacement, puis l’annuler si vous ne souhaitez pas effectuer un déplacement complet.



> [!NOTE]
> Ce tutoriel montre comment déplacer des machines virtuelles Azure d’une région à une autre telles quelles. Si vous avez besoin d’améliorer la disponibilité en déplaçant des machines virtuelles d’un groupe à haute disponibilité vers des machines virtuelles épinglées de zone dans une autre région, consultez le [tutoriel Déplacer des machines virtuelles Azure vers des zones de disponibilité](move-azure-vms-avset-azone.md).

## <a name="prerequisites"></a>Conditions préalables requises

- Vérifiez que les machines virtuelles Azure se trouvent dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.
- Vérifiez que votre choix de [combinaison région source/région cible est prise en charge](./azure-to-azure-support-matrix.md#region-support), et sélectionnez la région cible appropriée.
- Assurez-vous que vous comprenez [l’architecture et les composants du scénario](azure-to-azure-architecture.md).
- Examinez les [exigences et les limites de prise en charge](azure-to-azure-support-matrix.md).
- Vérifiez les autorisations du compte. Si vous avez créé votre compte Azure gratuit, vous êtes l’administrateur de votre abonnement. Si vous n’êtes pas administrateur de l’abonnement, collaborez avec l’administrateur pour affecter les autorisations dont vous avez besoin. Pour activer la réplication d’une machine virtuelle et copier essentiellement des données à l’aide d’Azure Site Recovery, vous devez avoir :

    - Des autorisations pour créer une machine virtuelle dans les ressources Azure. Le rôle prédéfini Contributeur de machines virtuelles dispose de ces autorisations, notamment :
    - L’autorisation de créer une machine virtuelle dans le groupe de ressources sélectionné
    - L’autorisation de créer une machine virtuelle dans le réseau virtuel sélectionné
    - L’autorisation d’écrire dans le compte de stockage sélectionné
    
    - Des autorisations pour gérer les opérations Azure Site Recovery. Le rôle Contributeur Site Recovery a toutes les autorisations nécessaires pour gérer les opérations Site Recovery dans un coffre Recovery Services.

- Vérifiez que tous les certificats racines les plus récents sont sur les machines virtuelles Azure à déplacer. Si les certificats racines les plus récents ne sont pas sur la machine virtuelle, les contraintes de sécurité empêcheront la copie des données vers la région cible.

- Pour les machines virtuelles Windows, installez-y toutes les mises à jour de Windows les plus récentes afin que tous les certificats racines approuvés s’y trouvent. Dans un environnement déconnecté, suivez les processus Windows Update et de mise à jour de certificat standard en vigueur pour votre organisation.
    
- Pour des machines virtuelles Linux, suivez les instructions fournies par votre distributeur Linux pour obtenir les certificats racines approuvés les plus récents et la dernière liste de révocation de certificats sur la machine virtuelle.
- N’utilisez pas de proxy d’authentification dans le but de contrôler la connectivité réseau pour les machines virtuelles que vous voulez déplacer.

- Si la machine virtuelle que vous voulez déplacer n’a pas d’accès à Internet, ou si elle utilise un proxy de pare-feu pour contrôler l’accès sortant, [vérifiez la configuration requise](azure-to-azure-tutorial-enable-replication.md#set-up-vm-connectivity).

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cela comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les adresses IP publiques.

- Vérifiez que votre abonnement Azure vous permet de créer des machines virtuelles dans la région cible utilisée pour la reprise d’activité. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge des machines virtuelles de tailles correspondant à vos machines virtuelles sources. Si vous l’utilisez pour copier des données vers la cible, Site Recovery choisit une machine virtuelle cible de la même taille ou de la taille la plus proche possible.

- Veillez à créer une ressource cible pour chaque composant identifié dans la topologie du réseau source. Cette étape est importante pour garantir que vos machines virtuelles offrent dans la région cible toutes les fonctionnalités et capacités dont vous disposiez dans la région source.

     > [!NOTE] 
     > Azure Site Recovery détecte et crée automatiquement un réseau virtuel quand vous activez la réplication pour la machine virtuelle source. Vous pouvez également créer au préalable un réseau et l’affecter à la machine virtuelle dans le flux d’utilisateur pour activer la réplication. Comme mentionné plus loin, vous devez créer manuellement toute autre ressource dans la région cible.

    Pour créer les ressources réseau courantes dont vous avez besoin, en fonction de la configuration de la machine virtuelle source, consultez la documentation suivante :
    - [Groupes de sécurité réseau](../virtual-network/manage-network-security-group.md)
    - [Équilibreurs de charge](../load-balancer/index.yml)
    -  [Adresse IP publique](../virtual-network/virtual-network-public-ip-address.md)
    - Pour tous les autres composants réseau, consultez la [documentation sur les réseaux](../index.yml?pivot=products&panel=network).



## <a name="prepare"></a>Préparation
Les étapes suivantes montrent comment préparer la machine virtuelle en vue du déplacement à l’aide d’Azure Site Recovery. 

### <a name="create-the-vault-in-any-region-except-the-source-region"></a>Créer le coffre dans n’importe quelle région, sauf la région source

1. Connectez-vous au [portail Azure](https://portal.azure.com)
1. Dans Rechercher, tapez Recovery Services > cliquez sur Coffres Recovery Services.
1. Dans le menu Coffres Recovery Services, cliquez sur +Ajouter.
1. Dans **Nom** , indiquez le nom convivial **ContosoVMVault**. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié.
1. Créez le groupe de ressources **ContosoRG**.
1. Spécifiez une région Azure. Pour découvrir les régions prises en charge, référez-vous à la disponibilité géographique dans la page de [détails de tarification Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
1. Dans **Coffres Recovery Services** , sélectionnez **ContosoVMVault** > **Éléments répliqués** >  **+Répliquer**.
1. Dans la liste déroulante, sélectionnez **Machines virtuelles Azure**.
1. Dans **Emplacement source** , sélectionnez la région Azure source où vos machines virtuelles s’exécutent actuellement.
1. Sélectionnez le modèle de déploiement Resource Manager. Sélectionnez ensuite **Abonnement source** et **Groupe de ressources source**.
1. Sélectionnez **OK** pour enregistrer les paramètres.

### <a name="enable-replication-for-azure-vms-and-start-copying-the-data"></a>Activer la réplication des machines virtuelles Azure et démarrer la copie des données

Site Recovery récupère une liste des machines virtuelles associées à l’abonnement et au groupe de ressources.

1. À l’étape suivante, sélectionnez la machine virtuelle que vous souhaitez déplacer, puis sélectionnez **OK**.
1. Dans **Paramètres** , sélectionnez **Récupération d’urgence**.
1. Dans **Configurer la récupération d’urgence** > **Région cible** , sélectionnez la région cible vers laquelle vous allez effectuer la réplication.
1. Pour ce didacticiel, acceptez les autres paramètres par défaut.
1. Sélectionnez **Activer la réplication**. Cette étape démarre un travail consistant à activer la réplication pour la machine virtuelle.



## <a name="move"></a>Déplacer

Les étapes suivantes montrent comment effectuer le déplacement vers la région cible.

1. Accédez au coffre. Dans **Paramètres** > **Éléments répliqués** , sélectionnez la machine virtuelle, puis **Basculement**.
2. Dans **Basculement** , sélectionnez **Dernier**.
3. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter la machine virtuelle source avant de déclencher le basculement. Le basculement est effectué même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
4. Une fois le travail terminé, vérifiez que la machine virtuelle apparaît bien dans la région Azure cible comme prévu.


## <a name="discard"></a>Abandonner 

Si vous avez vérifié la machine virtuelle déplacée et que vous devez modifier le point de basculement ou que vous souhaitez revenir à un point antérieur, sous **Éléments répliqués** , cliquez sur la machine virtuelle avec le bouton droit, puis sélectionnez **Modifier le point de récupération**. Cette étape vous permet de spécifier un autre point de récupération et de définir le basculement vers celui-ci. 


## <a name="commit"></a>Commit 

Une fois que vous avez vérifié la machine virtuelle déplacée et que vous êtes prêt à valider la modification, sous **Éléments répliqués** , cliquez avec le bouton droit sur la machine virtuelle, puis sélectionnez **Valider**. Cette étape termine le processus de déplacement vers la région cible. Attendez la fin de la tâche de validation.

## <a name="clean-up"></a>Nettoyer

Les étapes suivantes montrent comment nettoyer la région source et les ressources associées qui ont été utilisées pour le déplacement.

Pour toutes les ressources qui ont été utilisées pour le déplacement :

- Accédez à la machine virtuelle. Sélectionnez **Désactiver la réplication**. Cette étape empêche que le processus ne copie des données pour la machine virtuelle.

   > [!IMPORTANT]
   > Il est important d’effectuer cette étape pour éviter d’être facturé pour la réplication Azure Site Recovery.

Si vous n’envisagez de réutiliser aucune des ressources sources, suivez les étapes supplémentaires ci-après :

1. Supprimez toutes les ressources réseau concernées dans la région source que vous avez identifiée dans la section [Prérequis](#prerequisites).
1. Supprimez le compte de stockage associé dans la région source.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé une machine virtuelle Azure vers une autre région. Vous pouvez maintenant configurer la reprise d’activité après sinistre pour la machine virtuelle que vous avez déplacée.

> [!div class="nextstepaction"]
> [Configurer la récupération d’urgence après la migration](azure-to-azure-quickstart.md)
