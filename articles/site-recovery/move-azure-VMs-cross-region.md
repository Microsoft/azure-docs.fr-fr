---
title: Déplacer des machines virtuelles Azure vers une autre région à l’aide d’Azure Site Recovery
description: Utilisez Azure Site Recovery pour déplacer des machines virtuelles IaaS Azure d’une région Azure à l’autre.
services: site-recovery
author: sideeksh
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: sideeksh
ms.custom: MVC
ms.openlocfilehash: 5ae930240872c00c8dbb45857e4e77d82766eadf
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398060"
---
# <a name="move-azure-vms-to-another-azure-region"></a>Déplacer des machines virtuelles Azure vers une autre région Azure

Vous souhaitez peut-être déplacer les machines virtuelles de l’infrastructure Azure en tant que service (IaaS) d’une région à une autre afin d’en améliorer la fiabilité, la disponibilité, la gestion ou la gouvernance. Ce tutoriel vous montre comment déplacer des machines virtuelles vers une autre région à l’aide d’Azure Site Recovery. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Vérifier la configuration requise
> * Préparer les machines virtuelles sources
> * Préparer la région cible
> * Copier les données vers la région cible
> * Tester la configuration
> * Effectuer le déplacement
> * Supprimer les ressources de la région source


> [!IMPORTANT]
> Cet article décrit comment déplacer des machines virtuelles Azure d’une région à une autre *telles quelles*. Si votre objectif est d’améliorer la disponibilité de votre infrastructure en déplaçant les machines virtuelles vers les zones de disponibilité, consultez [Déplacer des machines virtuelles Azure vers des zones de disponibilité](move-azure-vms-avset-azone.md).

## <a name="prerequisites"></a>Prérequis

- Vérifiez que vous avez des machines virtuelles Azure dans la région Azure source *à partir de laquelle* vous souhaitez effectuer le déplacement.
- Vérifiez que votre choix de [combinaison de région source/région cible est pris en charge](./azure-to-azure-support-matrix.md#region-support) et sélectionnez la région cible appropriée.
- Assurez-vous que vous comprenez [l’architecture et les composants du scénario](azure-to-azure-architecture.md).
- Examinez les [exigences et les limites de prise en charge](azure-to-azure-support-matrix.md).
- Vérifiez les autorisations du compte. Si vous venez de créer votre compte Azure gratuit, *vous* êtes l’administrateur de votre abonnement. Si vous n’êtes pas administrateur, contactez l’administrateur pour obtenir les autorisations dont vous avez besoin :
  -  Pour activer la réplication pour une machine virtuelle et copier des données vers l’emplacement cible à l’aide de Site Recovery, vous devez disposer des autorisations pour créer une machine virtuelle dans vos ressources Azure. Le rôle intégré Contributeur de machines virtuelles a ces autorisations. Avec les autorisations, vous pouvez :
        - Créer une machine virtuelle dans le groupe de ressources sélectionné
        - Créer une machine virtuelle dans le réseau virtuel sélectionné
        - Écrire dans le compte de stockage sélectionné

  - Vous devez également avoir les autorisations nécessaires pour gérer les opérations Site Recovery. Le rôle Contributeur Site Recovery a toutes les autorisations nécessaires pour gérer les opérations Site Recovery dans un coffre Azure Recovery Services.

## <a name="prepare-the-source-vms"></a>Préparer les machines virtuelles sources

1. Vérifiez que les machines virtuelles Azure que vous envisagez de déplacer disposent des certificats racines les plus récents. Si ce n’est pas le cas, vous ne pouvez pas activer la copie des données vers la région cible en raison de contraintes de sécurité.

    - Pour les machines virtuelles Windows, installez-y les mises à jour de Windows les plus récentes afin que tous les certificats racines approuvés s’y trouvent. Dans un environnement déconnecté, suivez les processus Windows Update et de mise à jour de certificat standard en vigueur pour votre organisation.
    - Pour les machines virtuelles Linux, suivez les instructions fournies par votre distributeur Linux pour obtenir les derniers certificats racines approuvés et la dernière liste de révocation de certificats.
2. N’utilisez pas de proxy d’authentification dans le but de contrôler la connectivité réseau pour les machines virtuelles que vous voulez déplacer.
3. Si une des machines virtuelles que vous voulez déplacer n’a pas d’accès à Internet et si elle utilise un proxy de pare-feu pour contrôler l’accès sortant, [vérifiez la configuration requise](azure-to-azure-tutorial-enable-replication.md#set-up-vm-connectivity).
4. Documentez la topologie du réseau source et toutes les ressources que vous utilisez actuellement pour votre vérification, notamment mais sans s’y limiter, les équilibreurs de charge, les groupes de sécurité réseau et l’adresse IP publique.

## <a name="prepare-the-target-region"></a>Préparer la région cible

1. Dans votre abonnement Azure, vérifiez que vous pouvez créer des machines virtuelles dans la région cible utilisée pour la reprise d’activité. Contactez le support technique pour activer le quota nécessaire au besoin.

2. Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge la taille de vos machines virtuelles sources. Si vous l’utilisez pour copier des données vers la cible, Site Recovery choisit une machine virtuelle cible de la même taille ou de la taille la plus proche possible.

3. Veillez à créer une ressource cible pour chaque composant que vous avez identifié dans la topologie du réseau source. Ceci permet de garantir que vos machines virtuelles offriront dans la région cible toutes les fonctionnalités et capacités dont elles disposaient dans la région source.

   Azure Site Recovery détecte et crée automatiquement un réseau virtuel et un compte de stockage quand vous activez la réplication pour la machine virtuelle source. Vous pouvez également créer au préalable ces ressources et les affecter à la machine virtuelle dans le cadre de l’étape d’activation de la réplication. Mais vous devez créer manuellement toutes les autres ressources dans la région cible. Consultez la documentation suivante pour créer les ressources réseau courantes dont vous avez besoin, en fonction de la configuration de votre machine virtuelle source :

   - [Groupes de sécurité réseau](../virtual-network/manage-network-security-group.md)
   - [Équilibreurs de charge](../load-balancer/index.yml)
   - [Adresse IP publique](../virtual-network/virtual-network-public-ip-address.md)
    
   Pour tous les autres composants réseau, consultez la [documentation Azure sur la mise en réseau](../index.yml?pivot=products&panel=network). 

4. Pour tester la configuration avant d’effectuer le déplacement final vers la région cible, vous devez [créer un réseau hors production](../virtual-network/quick-create-portal.md) manuellement dans la région cible. Le test de la configuration est une pratique recommandée dans la mesure où elle limite les interférences avec l’environnement de production.
    
## <a name="copy-data-to-the-target-region"></a>Copier les données vers la région cible
Les étapes ci-dessous copient les données vers la région cible à l’aide d’Azure Site Recovery.

### <a name="create-the-vault-in-any-region-except-the-source"></a>Créez le coffre dans n’importe quelle région, à l’exception de la région source.

1. Connectez-vous au [portail Azure](https://portal.azure.com) > **Recovery Services**.
2. Sélectionnez **Créer une ressource** > **Outils de gestion** > **Backup and Site Recovery**.
3. Dans **Nom** , indiquez le nom convivial **ContosoVMVault**. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié.
4. Créez un groupe de ressources **ContosoRG**.
5. Spécifiez une région Azure. Pour vérifier les régions prises en charge, consultez [Informations détaillées sur la tarification d’Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. Dans Coffres Recovery Services, sélectionnez **Vue d’ensemble** > **ConsotoVMVault** >  **+Répliquer**.
7. Dans **Source** , sélectionnez **Azure**.
8. Dans **Emplacement source** , sélectionnez la région Azure source où vos machines virtuelles s’exécutent actuellement.
9. Sélectionnez le modèle de déploiement Azure Resource Manager. Sélectionnez ensuite **Abonnement source** et **Groupe de ressources source**.
10. Sélectionnez **OK** pour enregistrer les paramètres.

### <a name="enable-replication-for-azure-vms-and-start-copying-the-data"></a>Activer la réplication des machines virtuelles Azure et démarrer la copie des données

Site Recovery récupère une liste des machines virtuelles associées à l’abonnement et au groupe de ressources.

1. Sélectionnez la machine virtuelle que vous souhaitez déplacer, puis sélectionnez **OK**.
2. Dans **Paramètres** , sélectionnez **Récupération d’urgence**.
3. Dans **Configurer la récupération d’urgence** > **Région cible** , sélectionnez la région cible vers laquelle vous effectuez la réplication.
4. Choisissez d’utiliser les ressources de la cible par défaut ou celles que vous avez préalablement créées.
5. Sélectionnez **Activer la réplication** pour démarrer le travail.


 

## <a name="test-the-configuration"></a>Tester la configuration


1. Accédez au coffre. Dans **Paramètres** > **Éléments répliqués** , sélectionnez la machine virtuelle que vous souhaitez déplacer vers la région cible. Sélectionnez ensuite **Test de basculement**.
2. Dans **Test de basculement** , sélectionnez un point de récupération à utiliser pour le basculement :

   - **Dernier point traité**  : bascule la machine virtuelle vers le dernier point de récupération qui a été traité par le service Site Recovery. L’horodatage est affiché. Comme le traitement des données ne prend pas de temps, cette option offre donc un objectif de délai de récupération faible (RTO).
   - **Dernier point de cohérence des applications**  : bascule toutes les machines virtuelles vers le dernier point de récupération de cohérence des applications. L’horodatage est affiché.
   - **Personnalisé** : sélectionnez n’importe quel point de récupération.

3. Sélectionnez le réseau virtuel Azure cible vers lequel vous voulez déplacer les machines virtuelles Azure afin de tester la configuration.

   > [!IMPORTANT]
   > Nous vous recommandons d’utiliser un réseau machines virtuelles Azure distinct pour faire le test de basculement, et pas le réseau de production dans la région cible.

4. Pour commencer à tester le déplacement, sélectionnez **OK**. Pour suivre la progression, sélectionnez la machine virtuelle pour afficher ses **propriétés**. Sinon, sélectionnez le travail **Test de basculement** dans le coffre. Ensuite, sélectionnez **Paramètres** > **Travaux** > **Travaux Site Recovery**.
5. Une fois le basculement terminé, la machine virtuelle Azure de réplication apparaît dans le portail Azure > **Machines virtuelles**. Vérifiez que la machine virtuelle est en cours d’exécution, qu’elle est correctement dimensionnée et qu’elle est connectée au réseau approprié.
6. Pour supprimer la machine virtuelle que vous avez créé pour le test, sélectionnez **Nettoyer le test de basculement** sur l’élément répliqué. À partir de **Notes** , consignez et enregistrez les éventuelles observations associées au test.

## <a name="perform-the-move-and-confirm"></a>Effectuer le déplacement et confirmer

1. Accédez au coffre puis, dans **Paramètres** > **Éléments répliqués** , sélectionnez la machine virtuelle et sélectionnez ensuite **Basculement**.
1. Dans **Basculement** , sélectionnez **Dernier**. 
2. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter la machine virtuelle source avant de déclencher le basculement. Mais le basculement se poursuit même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
3. Une fois le travail terminé, vérifiez que la machine virtuelle apparaît bien dans la région Azure cible choisie.
4. Dans **Éléments répliqués** , cliquez avec le bouton droit sur la machine virtuelle et sélectionnez **Valider**. Cette étape termine le déplacement. Attendez la fin de la tâche de validation.

## <a name="discard-the-resources-from-the-source-region"></a>Supprimer les ressources de la région source

- Accédez à la machine virtuelle et sélectionnez **Désactiver la réplication**. Cela met fin au processus de copie des données pour la machine virtuelle.

  > [!IMPORTANT]
  > Effectuez cette étape pour éviter d’être facturé pour la réplication Azure Site Recovery après le déplacement.

Si vous n’envisagez pas de réutiliser les ressources sources, effectuez les étapes suivantes :

1. Supprimez toutes les ressources réseau appropriées dans la région source (celles que vous avez listées à l’étape 4 dans [Préparer les machines virtuelles sources](#prepare-the-source-vms)).
2. Supprimez le compte de stockage associé dans la région source.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à déplacer des machines virtuelles Azure vers une autre région Azure. Vous pouvez maintenant configurer la reprise d’activité après sinistre pour ces machines virtuelles.

> [!div class="nextstepaction"]
> [Configurer la récupération d’urgence après la migration](azure-to-azure-quickstart.md)
