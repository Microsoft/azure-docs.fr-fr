---
title: Configurer la récupération d'urgence de serveurs locaux physiques avec Azure Site Recovery
description: Découvrez comment configurer la récupération d’urgence vers Azure de serveurs Windows et Linux locaux avec le service Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 9b05d9952628e550beae5cedc49e051936a9d633
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87927281"
---
# <a name="set-up-disaster-recovery-to-azure-for-on-premises-physical-servers"></a>Configurer la récupération d’urgence vers Azure pour des serveurs physiques locaux

Le service [Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de récupération d’urgence en gérant et en coordonnant la réplication, le basculement et la restauration automatique des machines locales et des machines virtuelles Azure.

Ce didacticiel vous montre comment configurer la récupération d’urgence de serveurs physiques Windows et Linux locaux vers Azure. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer les prérequis Azure et locaux
> * Créer un coffre Recovery Services pour Site Recovery 
> * Configurer les environnements de réplication source et cible
> * Créer une stratégie de réplication
> * Activer la réplication pour un serveur

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce tutoriel :

- Veillez à bien comprendre [l’architecture et les composants](physical-azure-architecture.md) de ce scénario.
- Vérifiez les [exigences de prise en charge](vmware-physical-secondary-support-matrix.md) pour tous les composants.
- Assurez-vous que les serveurs que vous souhaitez répliquer sont conformes aux [conditions requises pour les machines virtuelles Azure](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
- Préparez Azure. Vous avez besoin d’un abonnement Azure, d’un réseau virtuel Azure et d’un compte de stockage.
- Préparez un compte pour l’installation automatique du service Mobilité sur chacun des serveurs que vous souhaitez répliquer.

Avant de commencer, notez les choses suivantes :

- Après le basculement vers Azure, les serveurs physiques ne pourront plus être restaurés vers des machines physiques locales. Vous ne pourrez effectuer une restauration que vers des machines virtuelles VMware. 
- Ce tutoriel définit la récupération d’urgence d’un serveur physique vers Azure avec les paramètres les plus simples. Si vous souhaitez en savoir plus sur les autres options, lisez nos guides pratiques :
    - Configurez la [source de réplication](physical-azure-set-up-source.md), notamment le serveur de configuration Site Recovery.
    - Configurez la [cible de réplication](physical-azure-set-up-target.md).
    - Configurez une [stratégie de réplication](vmware-azure-set-up-replication.md) et [activez la réplication](vmware-azure-enable-replication.md).


### <a name="set-up-an-azure-account"></a>Configurer un compte Azure

Procurez-vous un [compte Microsoft Azure](https://azure.microsoft.com/).

- Vous pouvez commencer par une version d’ [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
- Lisez les informations relatives aux [prix de Site Recovery](site-recovery-faq.md#pricing) et prenez connaissance des [prix appliqués](https://azure.microsoft.com/pricing/details/site-recovery/).
- Identifiez les [régions prises en charge](https://azure.microsoft.com/pricing/details/site-recovery/) pour Site Recovery.

### <a name="verify-azure-account-permissions"></a>Vérifier les autorisations de compte Azure

Assurez-vous que votre compte Azure dispose des autorisations nécessaires pour la réplication de machines virtuelles vers Azure.

- Vérifiez les [autorisations](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) nécessaires pour répliquer des machines vers Azure.
- Vérifiez et modifiez les autorisation de [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) . 



### <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

Configurez un [réseau Azure](../virtual-network/quick-create-portal.md).

- Les machines virtuelles Azure sont placées dans ce réseau une fois créées après le basculement.
- Ce réseau doit se trouver dans la même région que le coffre Recovery Services.


## <a name="set-up-an-azure-storage-account"></a>Configurer un compte de stockage Azure

Configurez un [compte de stockage Azure](../storage/common/storage-account-create.md).

- Site Recovery réplique les machines virtuelles locales sur le stockage Azure. Des machines virtuelles Azure sont créées à partir du stockage après le basculement.
- Le compte de stockage doit se trouver dans la même région que le coffre Recovery Services.


### <a name="prepare-an-account-for-mobility-service-installation"></a>Préparer un compte pour l’installation du service Mobilité

Le service Mobilité doit être installé sur chaque serveur que vous souhaitez répliquer. Site Recovery installe ce service automatiquement quand vous activez la réplication pour le serveur. Pour une installation automatique, vous devez préparer un compte qui sera utilisé par Site Recovery pour accéder au serveur.

- Vous pouvez utiliser un compte local ou de domaine
- Pour les machines virtuelles Windows, si vous n’utilisez pas un compte de domaine, désactivez le contrôle d’accès des utilisateurs distants sur la machine locale. Pour cela, dans le registre situé sous **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, ajoutez l’entrée DWORD **LocalAccountTokenFilterPolicy** avec une valeur de 1.
- Pour ajouter l’entrée de registre pour désactiver le paramètre à partir d’une interface CLI, tapez :       ``REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1.``
- Pour Linux, le compte doit être un utilisateur racine sur le serveur Linux source.


## <a name="create-a-vault"></a>Création d'un coffre

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]

## <a name="select-a-protection-goal"></a>Sélectionner un objectif de protection

Sélectionnez l’élément à répliquer et l’emplacement où il doit être répliqué.

1. Cliquez sur **Coffres Recovery Services** > coffre.
2. Dans le menu Ressource, cliquez sur **Site Recovery** > **Préparer l’infrastructure** > **Objectif de protection**.
3. Dans la boîte de dialogue **Objectif de protection**, sélectionnez **Vers Azure** > **Non virtualisé / Autre**.

## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

Configurez le serveur de configuration, inscrivez-le dans le coffre et découvrez les machines virtuelles.

1. Cliquez sur **Site Recovery** > **Préparer l’infrastructure** > **Source**.
2. Si vous n’avez pas de serveur de configuration, cliquez sur **+Serveur de configuration**.
3. Dans **Ajouter un serveur**, vérifiez que **Serveur de configuration** s’affiche dans **Type de serveur**.
4. Téléchargez le fichier d’installation unifiée de Site Recovery.
5. Téléchargez la clé d’inscription du coffre. Vous en aurez besoin lors de l’exécution du programme d’installation unifiée. Une fois générée, la clé reste valide pendant 5 jours.

   ![Capture d’écran montrant les options de téléchargement du fichier d’installation et de la clé d’inscription.](./media/physical-azure-disaster-recovery/source-environment.png)


### <a name="register-the-configuration-server-in-the-vault"></a>inscrire le serveur de configuration dans le coffre

Avant de commencer, procédez comme suit : 

#### <a name="verify-time-accuracy"></a>Vérifier la précision de l’heure
Sur la machine du serveur de configuration, assurez-vous que l’horloge système est synchronisée avec un [Serveur de temps](/windows-server/networking/windows-time-service/windows-time-service-top). Elles doivent correspondre. S’il y a 15 minutes d’avance ou de retard, le programme d’installation peut échouer.

#### <a name="verify-connectivity"></a>Vérifier la connectivité
Vérifiez que la machine peut accéder à ces URL en fonction de votre environnement : 

[!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]  

Les règles de pare-feu basées sur l’adresse IP doivent autoriser les communications vers toutes les URL Azure qui sont répertoriées ci-dessus sur le port HTTPS (443). Pour simplifier et limiter les plages d’adresses IP, il est recommandé de filtrer les URL.

- **Adresses IP d’entreprise** : autorisez [les plages d’adresses IP de centres de données Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653) et le port HTTPS (443). Autorisez les plages d’adresses IP pour la région Azure de votre abonnement afin de prendre en charge les URL AAD, de sauvegarde, de réplication et de stockage.  
- **Adresses IP d’une administration** : autorisez les [plages d’adresses IP du centre de données Azure Government](https://www.microsoft.com/en-us/download/details.aspx?id=57063) et le port HTTPS (443) pour toutes les régions USGov (Virginie, Texas, Arizona et Iowa) afin de prendre en charge les URL AAD, de sauvegarde, de réplication et de stockage.  

#### <a name="run-setup"></a>Exécuter le programme d’installation
Exécutez le programme d’installation unifiée en tant qu’administrateur local pour installer le serveur de configuration. Le serveur de processus et le serveur cible maître sont également installés par défaut sur le serveur de configuration.

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]


## <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

Sélectionnez et vérifiez les ressources cibles.

1. Cliquez sur **Préparer l’infrastructure** > **Cible** et sélectionnez l’abonnement Azure à utiliser.
2. Spécifiez le modèle de déploiement cible.
3. Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles.

   ![Capture d’écran des options de configuration de l’environnement cible.](./media/physical-azure-disaster-recovery/network-storage.png)


## <a name="create-a-replication-policy"></a>Créer une stratégie de réplication

1. Pour créer une nouvelle stratégie de réplication, cliquez sur **Infrastructure de Site Recovery** > **Stratégies de réplication** >  **+Stratégie de réplication**.
2. Dans **Créer une stratégie de réplication**, indiquez le nom de la stratégie.
3. Dans **Seuil d’objectif de point de récupération**, spécifiez la limite de l’objectif de point de récupération (RPO). Cette valeur spécifie la fréquence à laquelle les points de récupération des données sont créés. Une alerte est générée lorsque la réplication continue dépasse cette limite.
4. Dans **Rétention des points de récupération**, spécifiez la durée de la fenêtre de rétention pour chaque point de récupération (en heures). Les machines virtuelles répliquées peuvent être récupérées à n’importe quel point dans une fenêtre. Les machines virtuelles répliquées vers Premium Storage peuvent prendre en charge jusqu’à 24 heures de rétention et 72 heures en cas de stockage standard.
5. Dans **Fréquence des captures instantanées de cohérence d’application**, spécifiez la fréquence de création des points de récupération contenant des captures instantanées cohérentes avec les applications (en minutes). Cliquez sur **OK** pour créer la stratégie.

    ![Capture d’écran des options de création d’une stratégie de réplication.](./media/physical-azure-disaster-recovery/replication-policy.png)


La stratégie est automatiquement associée au serveur de configuration. Par défaut, une stratégie de correspondance est automatiquement créée pour la restauration automatique. Par exemple, si la stratégie de réplication est **rep-policy**, la stratégie de restauration automatique **rep-policy-failback** est créée. Cette stratégie n’est utilisée qu’à partir du moment où vous initiez une restauration automatique à partir d’Azure.

## <a name="enable-replication"></a>Activer la réplication

Activez la réplication pour chaque serveur.

- Site Recovery installe automatiquement le service Mobilité quand la réplication est activée.
- Quand vous activez la réplication pour un serveur, il peut être nécessaire d’attendre 15 minutes ou plus avant que les modifications prennent effet et qu’elles apparaissent dans le portail.

1. Cliquez sur **Répliquer l’application** > **Source**.
2. Dans **Source**, sélectionnez le serveur de configuration.
3. Dans **Type de machine**, sélectionnez **Machines physiques**.
4. Sélectionnez le serveur de processus (serveur de configuration). Cliquez ensuite sur **OK**.
5. Dans **Cible**, sélectionnez l’abonnement et le groupe de ressources dans lesquels vous voulez créer les machines virtuelles Azure après le basculement. Choisissez le modèle de déploiement que vous souhaitez utiliser dans Azure (classique ou gestion des ressources).
6. Sélectionnez le compte de stockage Azure que vous souhaitez utiliser pour les données de réplication. 
7. Sélectionnez le sous-réseau et le réseau Azure auxquels les machines virtuelles Azure se connectent lorsqu’elles sont créées après le basculement.
8. Sélectionnez **Effectuez maintenant la configuration pour les machines sélectionnées** pour appliquer le paramètre réseau à l’ensemble des machines que vous sélectionnez à des fins de protection. Sélectionnez **Configurer ultérieurement** pour sélectionner le réseau Azure pour chaque machine. 
9. Dans **Machines physiques**, cliquez sur **+Machines physiques**. Spécifiez le nom et l’adresse IP. Sélectionnez le système d’exploitation de la machine que vous souhaitez répliquer. La détection et l’affichage de la liste des serveurs peuvent prendre quelques minutes. 
10. Dans **Propriétés** > **Configurer les propriétés**, sélectionnez le compte à utiliser par le serveur de processus pour installer automatiquement le service Mobilité sur la machine.
11. Dans **Paramètres de réplication** > **Configurer les paramètres de réplication**, vérifiez que la stratégie de réplication correcte est sélectionnée. 
12. Cliquez sur **Activer la réplication**. Vous pouvez suivre la progression du travail **Activer la protection** dans **Paramètres** > **Travaux** > **Travaux Site Recovery**. Une fois le travail **Finaliser la protection** exécuté, la machine est prête pour le basculement.


Pour effectuer le monitoring des serveurs, vous pouvez consulter l’heure de leur dernière détection dans **Serveurs de configuration** > **Dernier contact à**. Pour ajouter des machines sans attendre l’heure de la détection planifiée, mettez en surbrillance le serveur de configuration (sans cliquer dessus) et cliquez sur **Actualiser**.

## <a name="next-steps"></a>Étapes suivantes

[Effectuer un exercice de récupération d’urgence vers Azure](tutorial-dr-drill-azure.md)
