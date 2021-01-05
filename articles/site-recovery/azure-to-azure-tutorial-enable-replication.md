---
title: Tutoriel expliquant comment configurer la reprise d’activité après sinistre d’une machine virtuelle Azure à l’aide d’Azure Site Recovery
description: Dans ce tutoriel, vous allez voir comment configurer la reprise d’activité après sinistre pour des machines virtuelles Azure vers une autre région Azure, à l’aide du service Azure Site Recovery.
ms.topic: tutorial
ms.date: 11/03/2020
ms.custom: mvc
ms.openlocfilehash: 473a264ef497cab4bd4f88372600161b33178099
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656867"
---
# <a name="tutorial-set-up-disaster-recovery-for-azure-vms"></a>Tutoriel : Configurer la récupération d’urgence pour les machines virtuelles Azure

Ce tutoriel explique comment configurer la reprise d’activité après sinistre pour les machines virtuelles Azure à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Vérifier les paramètres et les autorisations Azure
> * Préparer les machines virtuelles à répliquer
> * Créer un coffre Recovery Services
> * Activer la réplication des machines virtuelles

Quand vous activez la réplication pour une machine virtuelle afin de configurer la reprise d’activité après sinistre, l’extension de service Mobilité Site Recovery est installée sur la machine virtuelle, puis elle est inscrite auprès d’Azure Site Recovery. Pendant la réplication, les écritures réalisées sur les disques de machine virtuelle sont envoyées à un compte de stockage de cache de la région source. Les données sont envoyées de cet emplacement vers la région cible, et des points de récupération sont générés à partir des données. Lorsque vous basculez une machine virtuelle pendant une reprise d’activité après sinistre, un point de récupération est utilisé pour restaurer la machine virtuelle dans la région cible.

> [!NOTE]
> Les tutoriels fournissent des instructions avec les paramètres par défaut les plus simples. Si vous souhaitez configurer la reprise d’activité après sinistre pour les machines virtuelles Azure avec des paramètres personnalisés, consultez [cet article](azure-to-azure-how-to-enable-replication.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

- [Passez en revue les régions prises en charge](azure-to-azure-support-matrix.md#region-support). Vous pouvez configurer la reprise d’activité après sinistre pour les machines virtuelles Azure entre deux régions appartenant à la même zone géographique.
- Pour cela, vous avez besoin d’une ou de plusieurs machines virtuelles Azure. Vérifiez que les machines virtuelles [Windows](azure-to-azure-support-matrix.md#windows) ou [Linux](azure-to-azure-support-matrix.md#replicated-machines---linux-file-systemguest-storage) sont prises en charge.
- Passez en revue les conditions de [calcul](azure-to-azure-support-matrix.md#replicated-machines---compute-settings), de [stockage](azure-to-azure-support-matrix.md#replicated-machines---storage) et de [réseau](azure-to-azure-support-matrix.md#replicated-machines---networking).
- Ce tutoriel suppose que les machines virtuelles ne sont pas chiffrées. Si vous souhaitez configurer la reprise d’activité après sinistre pour des machines virtuelles chiffrées, [suivez les instructions de cet article](azure-to-azure-how-to-enable-replication-ade-vms.md).

## <a name="check-azure-settings"></a>Vérifier les paramètres Azure

Vérifiez les autorisations et les paramètres de la région cible.

### <a name="check-permissions"></a>Vérifier les autorisations

Votre compte Azure a besoin d’autorisations pour créer un coffre Recovery Services et pour créer des machines virtuelles dans la région cible.

- Si vous venez de créer un abonnement Azure gratuit, vous êtes l’administrateur du compte et aucune autre action n’est nécessaire.
- Si vous n’êtes pas l’administrateur, demandez à celui-ci de vous accorder les autorisations dont vous avez besoin.
    - **Créer un coffre** : Autorisations d’administrateur ou de propriétaire d’abonnement. 
    - **Gérer les opérations Site Recovery dans le coffre** : Rôle Azure intégré *Contributeur Site Recovery*.
    - **Créer des machines virtuelles Azure dans la région cible** : Soit le rôle intégré *Contributeur de machines virtuelles*, soit des autorisations permettant d’effectuer les opérations suivantes :
        - Créer une machine virtuelle dans le réseau virtuel sélectionné
        - Écrire sur un compte de stockage Azure.
        - Écrire sur un disque managé Azure.

### <a name="verify-target-settings"></a>Vérifier les paramètres cibles

Pendant une reprise d’activité, lorsque vous effectuez un basculement à partir de la région source, les machines virtuelles sont créées dans la région cible. 

Vérifiez que votre abonnement dispose de suffisamment de ressources dans la région cible. Vous devez pouvoir créer des machines virtuelles dont la taille correspond à celles des machines virtuelles de la région source. Quand vous configurez la reprise d’activité après sinistre, Site Recovery choisit la même taille (ou la taille la plus proche possible) pour la machine virtuelle cible.


## <a name="prepare-vms"></a>Préparer les machines virtuelles

Vérifiez que les machines virtuelles disposent d’une connectivité sortante et des certificats racines les plus récents. 


### <a name="set-up-vm-connectivity"></a>Configurer la connectivité des machines virtuelles

Les machines virtuelles que vous souhaitez répliquer nécessitent une connectivité réseau sortante.

> [!NOTE]
> Site Recovery ne prend pas en charge l’utilisation d’un proxy d’authentification pour contrôler la connectivité réseau.

#### <a name="outbound-connectivity-for-urls"></a>Connectivité sortante pour les URL

Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connectivité sortante, autorisez l’accès à ces URL :

| **Nom**                  | **Commercial**                               | **Secteur public**                                 | **Description** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Stockage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net` | Permet d’écrire les données dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Fournit l’autorisation et l’authentification aux URL du service Site Recovery. |
| Réplication               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permet à la machine virtuelle de communiquer avec le service Site Recovery. |
| Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permet à la machine virtuelle d’écrire des données de surveillance et de diagnostic Site Recovery. |

#### <a name="outbound-connectivity-for-ip-address-ranges"></a>Connectivité sortante pour les plages d’adresses IP

Si vous utilisez des groupes de sécurité réseau pour contrôler la connectivité, créez des règles de groupe de sécurité réseau basées sur les étiquettes de service qui autorisent le trafic HTTPS sortant sur le port 443 pour ces [étiquettes de service](../virtual-network/service-tags-overview.md#available-service-tags) (groupes d’adresses IP) :

**Tag** | **Autoriser** 
--- | ---
Étiquette du stockage  |Permet d’écrire des données entre la machine virtuelle et le compte de stockage de cache.   
Étiquette Azure AD | Autorise l’accès à toutes les adresses IP qui correspondent à Azure AD.   
Étiquette EventsHub | Permet d’accéder à la supervision de Site Recovery.  
Étiquette AzureSiteRecovery | Permet d’accéder au service Site Recovery dans n’importe quelle région.   
Étiquette GuestAndHybridManagement | Utilisez-la si vous souhaitez mettre automatiquement à niveau l’agent Mobilité Site Recovery qui est exécuté sur les machines virtuelles où est activée la réplication.

[Cliquez ici](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags) pour en savoir plus sur les étiquettes nécessaires et obtenir des exemples d’étiquettes.

### <a name="verify-vm-certificates"></a>Vérifier les certificats des machines virtuelles

Vérifiez que les machines virtuelles disposent des certificats racines les plus récents. Si ce n’est pas le cas, les machines virtuelles ne pourront pas être inscrites auprès du service Site Recovery en raison de contraintes de sécurité.

- **Machines virtuelles Windows** : Installez toutes les mises à jour Windows les plus récentes sur la machine virtuelle pour que tous les certificats racines approuvés s’y trouvent. Dans un environnement déconnecté, suivez vos processus standard pour Windows Update et pour les mises à jour des certificats.
- **Machines virtuelles Linux** : Suivez les instructions fournies par votre distributeur Linux pour obtenir les certificats racines approuvés les plus récents et la dernière liste de révocation de certificats (CRL).

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Vous pouvez créer un coffre Recovery Services dans n’importe quelle région, sauf dans la région source à partir de laquelle vous souhaitez répliquer les machines virtuelles.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la zone de recherche, tapez *recovery*. Sous **Services**, sélectionnez **Coffres Recovery Services**.

    ![Rechercher les coffres Recovery Services](./media/azure-to-azure-tutorial-enable-replication/search.png)

3. Dans **Coffres Recovery Services**, sélectionnez **Ajouter**.
4. Dans **Créer un coffre Recovery Services** > **De base**, sélectionnez l’abonnement dans lequel créer le coffre.
5. Dans **Groupe de ressources**, sélectionnez un groupe de ressources existant pour le coffre ou créez-en un.
6. Dans **Nom du coffre**, indiquez un nom convivial permettant d’identifier le coffre.
7. Dans **Région**, sélectionnez la région Azure dans laquelle placer le coffre. [Vérifiez les régions prises en charge](https://azure.microsoft.com/pricing/details/site-recovery/).
8. Sélectionnez **Revoir + créer**.

   ![Paramètres du coffre dans la page de création d’un coffre](./media/azure-to-azure-tutorial-enable-replication/vault-basics.png)

9. Dans **Vérifier + créer**, sélectionnez **Créer**.

10. Le déploiement du coffre commence. Suivez la progression dans les notifications.
11. Une fois le coffre déployé, sélectionnez **Épingler au tableau de bord** pour y accéder rapidement la prochaine fois. Sélectionnez **Accéder à la ressource** pour ouvrir le nouveau coffre. 
    
    ![Boutons permettant d’ouvrir le coffre après son déploiement et de l’épingler au tableau de bord](./media/azure-to-azure-tutorial-enable-replication/vault-deploy.png)

### <a name="enable-site-recovery"></a>Activer Site Recovery

Dans les paramètres du coffre, sélectionnez **Activer Site Recovery**.

![Activation de Site Recovery dans le coffre](./media/azure-to-azure-tutorial-enable-replication/enable-site-recovery.png)

## <a name="enable-replication"></a>Activer la réplication

Sélectionnez les paramètres de la source, puis activez la réplication de la machine virtuelle. 

### <a name="select-source-settings"></a>Sélectionner les paramètres de la source

1. Dans la page **Site Recovery** du coffre, sous **Machines virtuelles Azure**, sélectionnez **Activer la réplication**.

    ![Activation de la réplication pour les machines virtuelles Azure](./media/azure-to-azure-tutorial-enable-replication/enable-replication.png)

2. Dans **Source**> **Emplacement source**, sélectionnez la région Azure source où vos machines virtuelles s’exécutent actuellement.
3. Dans **Modèle de déploiement de machine virtuelle Azure**, conservez le paramètre par défaut **Resource Manager**.
4. Dans **Abonnement source**, sélectionnez l’abonnement dans lequel les machines virtuelles sont exécutées. Vous pouvez sélectionner n’importe quel abonnement situé dans le locataire Azure Active Directory (AD) où se trouve votre coffre.
5. Dans **Groupe de ressources source**, sélectionnez le groupe de ressources qui contient les machines virtuelles.
6. Dans **Reprise d’activité entre zones de disponibilité**, conservez le paramètre par défaut **Aucune**.

     ![Configurer la source](./media/azure-to-azure-tutorial-enable-replication/source.png)

7. Sélectionnez **Suivant**.

### <a name="select-the-vms"></a>Sélectionner les machines virtuelles

Site Recovery récupère les machines virtuelles associées à l’abonnement ou au groupe de ressources sélectionné.

1. Dans **Machines virtuelles**, sélectionnez les machines virtuelles pour lesquelles vous souhaitez activer la reprise d’activité après sinistre.

     ![Page de sélection des machines virtuelles pour la réplication](./media/azure-to-azure-tutorial-enable-replication/select-vm.png)

2. Sélectionnez **Suivant**.

### <a name="review-replication-settings"></a>Passer en revue les paramètres de réplication

1. Dans **Paramètres de réplication**, passez en revue les paramètres. Site Recovery crée les paramètres ou la stratégie par défaut pour la région cible. Pour ce tutoriel, nous utilisons les paramètres par défaut.
2. Sélectionnez **Activer la réplication**.

    ![Page de personnalisation des paramètres et d’activation de la réplication](./media/azure-to-azure-tutorial-enable-replication/enable-vm-replication.png)   

3. Suivez la progression de la réplication dans les notifications.

     ![Suivre la progression de la réplication dans les notifications](./media/azure-to-azure-tutorial-enable-replication/notification.png) ![Notification de la réussite de la réplication](./media/azure-to-azure-tutorial-enable-replication/notification-success.png)

4. Les machines virtuelles que vous activez s’affichent dans la page **Éléments répliqués** du coffre.

    ![Machine virtuelle dans la page Éléments répliqués](./media/azure-to-azure-tutorial-enable-replication/replicated-items.png)


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez activé la reprise d’activité après sinistre pour une machine virtuelle Azure. À présent, testez la reprise d’activité après sinistre afin de vérifier que le basculement fonctionne comme prévu.

> [!div class="nextstepaction"]
> [Effectuer un test de récupération d’urgence](azure-to-azure-tutorial-dr-drill.md)
