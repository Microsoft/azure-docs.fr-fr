---
title: Préparer les ressources Azure Site Recovery pour la reprise d’activité après sinistre des machines virtuelles Azure VMware Solution
description: Découvrez comment préparer les ressources Azure pour la reprise d’activité des machines Azure VMware Solution à l’aide d’Azure Site Recovery.
services: site-recovery
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/29/2020
ms.author: harshacs
ms.custom: MVC
ms.openlocfilehash: 2bd305e3760a8c3d743037e7d90b71f5e9579eda
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395476"
---
# <a name="prepare-azure-site-recovery-resources-for-disaster-recovery-of-azure-vmware-solution-vms"></a>Préparer les ressources Azure Site Recovery pour la reprise d’activité après sinistre des machines virtuelles Azure VMware Solution

Cet article explique comment préparer les ressources et composants Azure pour pouvoir configurer la reprise d’activité des machines virtuelles Azure VMware Solution à l’aide du service [Azure Site Recovery](site-recovery-overview.md). [Azure VMware Solution](../azure-vmware/introduction.md) fournit des clouds privés dans Azure. Ces clouds privés contiennent des clusters vSphere, créés à partir d’une infrastructure Azure complète dédiée.

Cet article est le premier tutoriel d’une série qui vous montre comment configurer la récupération d’activité pour des machines virtuelles Azure VMware Solution. 


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifiez que le compte Azure dispose d’autorisations de réplication.
> * Créez un coffre Recovery Services. Un coffre conserve des métadonnées et des informations de configuration pour les machines virtuelles et les autres composants de réplication.
> * Configurer un réseau virtuel Azure (VNet). Une fois les machines virtuelles Azure créées après le basculement, elles sont jointes à ce réseau.

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour obtenir des instructions détaillées, consultez l’article de la section Procédures dans la table des matières de Site Recovery.

> [!NOTE]
> Certains des concepts de l’utilisation d’Azure Site Recovery pour Azure VMware Solution se recoupent avec ceux de la récupération d’activité des machines virtuelles VMware locales. C’est pourquoi vous trouverez des références croisées vers la documentation quand cela sera pertinent.

## <a name="before-you-start"></a>Avant de commencer

- [Déployer](../azure-vmware/tutorial-create-private-cloud.md) un cloud privé Azure VMware Solution dans Azure
- Examiner l’architecture de la récupération d’activité [VMware](vmware-azure-architecture.md)
- Lire les questions courantes pour [VMware](vmware-azure-common-questions.md)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer. Puis, connectez-vous au [portail Azure](https://portal.azure.com).


## <a name="verify-account-permissions"></a>Vérifier les autorisations du compte

Si vous venez de créer votre compte Azure gratuit, vous êtes l’administrateur de votre abonnement et vous disposez des autorisations nécessaires. Si vous n’êtes pas administrateur de l’abonnement, collaborez avec l’administrateur pour affecter les autorisations dont vous avez besoin. Pour activer la réplication pour une nouvelle machine virtuelle, vous devez avoir l’autorisation d’effectuer les actions ci-après :

- Créer une machine virtuelle dans le groupe de ressources sélectionné
- Créer une machine virtuelle dans le réseau virtuel sélectionné
- Écrire sur un compte de stockage Azure.
- Écrire sur un disque managé Azure.

Pour effectuer ces tâches, le rôle prédéfini de contributeur de machines virtuelles doit être assigné à votre compte. De plus, pour gérer les opérations Site Recovery dans un coffre, le rôle prédéfini de contributeur Site Recovery doit être assigné à votre compte.


## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

1. Dans le menu du portail Azure, sélectionnez **Créer une ressource**, puis recherchez **Recovery** dans la Place de marché.
2. Sélectionnez **Backup and Site Recovery** dans les résultats de la recherche puis, dans la page éponyme, cliquez sur **Créer**. 
3. Sur la page **Créer un coffre Recovery Services**, sélectionnez l’**abonnement**. Nous utilisons l’**abonnement Contoso**.
4. Dans **Groupe de ressources**, sélectionnez un groupe de ressources existant ou créez-en un. Pour ce tutoriel, nous utilisons **contosoRG**.
5. Dans **Nom du coffre**, entrez un nom convivial pour identifier le coffre. Pour cette série de didacticiels, nous utilisons **ContosoVMVault**.
6. Dans **Région**, sélectionnez la région dans laquelle le coffre doit se trouver. Nous utilisons **Europe Ouest**.
7. Sélectionnez **Revoir + créer**.

   ![Capture d’écran de la page Créer un coffre Recovery Services](./media/tutorial-prepare-azure/new-vault-settings.png)

   Le nouveau coffre est maintenant répertorié dans **Tableau de bord** > **Toutes les ressources** et dans la page principale **Coffres Recovery Services**.

## <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

 Les machines virtuelles Azure VMware Solution sont répliquées sur des disques managés Azure. Durant le basculement, les machines virtuelles Azure sont créées à partir de ces disques managés et jointes au réseau Azure spécifié dans cette procédure.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**.
2. Laissez **Resource Manager** sélectionné en tant que modèle de déploiement.
3. Dans **Nom**, entrez un nom de réseau. Le nom doit être unique au sein du groupe de ressources Azure. Dans ce tutoriel, nous utilisons **ContosoASRnet**.
4. Dans **Espace d’adressage**, entrez la plage d’adresses du réseau virtuel en notation CDR. Nous utilisons **10.1.0.0/24**.
5. Dans **Abonnement**, sélectionnez l’abonnement dans lequel créer le réseau.
6. Spécifiez le **groupe de ressources** dans lequel le réseau sera créé. Nous utilisons le groupe de ressources existant **contosoRG**.
7. Dans **Emplacement**, sélectionnez la même région que celle dans laquelle le coffre Recovery Services a été créé. Dans notre tutoriel, il s’agit de la région **Europe Ouest**. Ce réseau doit se trouver dans la même région que le coffre.
8. Dans **Plage d’adresses**, entrez la plage du réseau. Nous utilisons **10.1.0.0/24** sans utiliser de sous-réseau.
9. Nous laissons les options par défaut de la protection DDoS de base, sans point de terminaison de service ou pare-feu sur le réseau.
9. Sélectionnez **Create** (Créer).

   ![Capture d’écran des options Créer un réseau virtuel](media/tutorial-prepare-azure/create-network.png)

La création du réseau virtuel prend quelques secondes. Une fois qu’il est créé, vous le voyez dans le tableau de bord du portail Azure.




## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Préparer l’infrastructure](avs-tutorial-prepare-avs.md)
- [En savoir plus](../virtual-network/virtual-networks-overview.md) sur les réseaux Azure.
- [En savoir plus](../virtual-machines/managed-disks-overview.md) sur les disques managés.
