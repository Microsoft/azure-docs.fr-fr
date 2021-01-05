---
title: Déplacer des machines virtuelles Azure entre des régions Azure Government et des régions publiques à l’aide d’Azure Site Recovery
description: Utilisez Azure Site Recovery pour déplacer des machines virtuelles Azure entre des régions Azure Government et des régions publiques.
author: sideeksh
ms.service: site-recovery
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: sideeksh
ms.custom: MVC
ms.openlocfilehash: a76ebf95b92b6e1251a04daa9ffb48a9abe15b50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89425345"
---
# <a name="move-azure-vms-between-azure-government-and-public-regions"></a>Déplacer des machines virtuelles Azure entre Azure Government et des régions publiques 

Vous souhaitez peut-être déplacer vos machines virtuelles IaaS entre Azure Government et des régions publiques pour augmenter la disponibilité de vos machines virtuelles existantes, améliorer la facilité de gestion, ou pour des raisons de gouvernance, comme détaillé [ici](azure-to-azure-move-overview.md).

En plus de l’utilisation du service [Azure Site Recovery](site-recovery-overview.md) pour gérer et orchestrer la reprise d’activité après sinistre des ordinateurs locaux et des machines virtuelles Azure dans le cadre de la continuité d’activité et de la reprise d’activité, vous pouvez utiliser Site Recovery pour gérer le déplacement des machines virtuelles Azure vers une région secondaire.       

Ce tutoriel vous montre comment déplacer des machines virtuelles Azure entre Azure Government et des régions publiques à l’aide d’Azure Site Recovery. Ceci peut être étendu au déplacement de machines virtuelles entre des paires de régions ne se trouvant pas dans le même cluster géographique. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifier la configuration requise
> * Préparer les machines virtuelles sources
> * Préparer la région cible
> * Copier les données vers la région cible
> * Tester la configuration
> * Effectuer le déplacement
> * Supprimer les ressources dans la région source

> [!IMPORTANT]
> Ce tutoriel vous montre comment déplacer des machines virtuelles Azure entre Azure Government et des régions publiques, ou entre des paires de régions qui ne sont pas prises en charge par la solution habituelle de récupération d’urgence des machines virtuelles Azure. Si vos paires de régions source et cible sont [prises en charge](./azure-to-azure-support-matrix.md#region-support), reportez-vous à ce [document](azure-to-azure-tutorial-migrate.md) pour le déplacement. Si votre objectif est d’améliorer la disponibilité des machines virtuelles d’un groupe à haute disponibilité en les déplaçant vers une autre région pour les déployer en tant que machines virtuelles rattachées à une zone, consultez le tutoriel fourni [ici](move-azure-VMs-AVset-Azone.md).

> [!IMPORTANT]
> Il n’est pas recommandé d’utiliser cette méthode pour configurer la récupération d’urgence entre des paires de régions non prises en charge, car les paires sont définies en tenant compte de la latence des données, ce qui est essentiel pour un scénario de récupération d’urgence.

## <a name="verify-prerequisites"></a>Vérifier la configuration requise

> [!NOTE]
> Veillez à bien comprendre [l’architecture et les composants](physical-azure-architecture.md) de ce scénario. Cette architecture permet de déplacer des machines virtuelles Azure, **en traitant les machines virtuelles comme des serveurs physiques**.

- Vérifiez les [exigences de prise en charge](vmware-physical-secondary-support-matrix.md) pour tous les composants.
- Assurez-vous que les serveurs que vous souhaitez répliquer sont conformes aux [conditions requises pour les machines virtuelles Azure](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
- Préparez un compte pour l’installation automatique du service Mobilité sur chacun des serveurs que vous souhaitez répliquer.

- Notez qu’après avoir basculé vers la région cible dans Azure, vous ne pouvez pas effectuer directement un basculement pour revenir dans la région source. Vous devrez configurer à nouveau la réplication sur la cible.

### <a name="verify-azure-account-permissions"></a>Vérifier les autorisations de compte Azure

Assurez-vous que votre compte Azure dispose des autorisations nécessaires pour la réplication de machines virtuelles vers Azure.

- Vérifiez les [autorisations](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) nécessaires pour répliquer des machines vers Azure.
- Vérifiez et modifiez les autorisation de [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) . 

### <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

Configurer une cible [réseau Azure](../virtual-network/quick-create-portal.md).

- Les machines virtuelles Azure sont placées dans ce réseau une fois créées après le basculement.
- Ce réseau doit se trouver dans la même région que le coffre Recovery Services.


### <a name="set-up-an-azure-storage-account"></a>Configurer un compte de stockage Azure

Configurez un [compte de stockage Azure](../storage/common/storage-account-create.md).

- Site Recovery réplique les machines virtuelles locales sur le stockage Azure. Des machines virtuelles Azure sont créées à partir du stockage après le basculement.
- Le compte de stockage doit se trouver dans la même région que le coffre Recovery Services.


## <a name="prepare-the-source-vms"></a>Préparer les machines virtuelles sources

### <a name="prepare-an-account-for-mobility-service-installation"></a>Préparer un compte pour l’installation du service Mobilité

Le service Mobilité doit être installé sur chaque serveur que vous souhaitez répliquer. Site Recovery installe ce service automatiquement quand vous activez la réplication pour le serveur. Pour une installation automatique, vous devez préparer un compte qui sera utilisé par Site Recovery pour accéder au serveur.

- Vous pouvez utiliser un compte local ou de domaine
- Pour les machines virtuelles Windows, si vous n’utilisez pas un compte de domaine, désactivez le contrôle d’accès des utilisateurs distants sur la machine locale. Pour cela, dans le registre situé sous **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, ajoutez l’entrée DWORD **LocalAccountTokenFilterPolicy** avec une valeur de 1.
- Pour ajouter l’entrée de registre pour désactiver le paramètre à partir d’une interface CLI, tapez :       ``REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1.``
- Pour Linux, le compte doit être un utilisateur racine sur le serveur Linux source.


## <a name="prepare-the-target-region"></a>Préparer la région cible

1. Vérifiez que votre abonnement Azure vous permet de créer des machines virtuelles dans la région cible utilisée pour la récupération d’urgence. Contactez le support pour activer le quota requis.

2. Assurez-vous que votre abonnement dispose de suffisamment de ressources pour prendre en charge des machines virtuelles de tailles correspondant à vos machines virtuelles source. Si vous l’utilisez pour copier des données vers la cible, Site Recovery choisit une machine virtuelle cible de la même taille ou de la taille la plus proche possible.

3. Veillez à créer une ressource cible pour chaque composant identifié dans la topologie du réseau source. Cette étape est importante pour garantir que vos machines virtuelles, après leur déplacement final vers la région cible, offriront toutes les fonctionnalités et capacités qui étaient disponibles dans la région source.

    > [!NOTE]
    > Azure Site Recovery détecte et crée automatiquement un réseau virtuel quand vous activez la réplication de la machine virtuelle source. Vous pouvez aussi précréer un réseau et l’affecter à la machine virtuelle dans le flux utilisateur pour activer la réplication. En revanche, pour toutes les autres ressources, comme mentionné ci-dessous, vous devez les créer manuellement dans la région cible.

     Consultez la documentation suivante pour créer les ressources réseau courantes dont vous avez besoin, en fonction de la configuration de la machine virtuelle source.

    - [Groupes de sécurité réseau](../virtual-network/manage-network-security-group.md)
    - [Équilibreurs de charge](../load-balancer/index.yml)
    - [Adresse IP publique](../virtual-network/virtual-network-public-ip-address.md)
    
    Pour tous les autres composants réseau, reportez-vous à la [documentation](../index.yml?pivot=products&panel=network) sur la mise en réseau.

4. Vous devez [créer un réseau hors production](../virtual-network/quick-create-portal.md) manuellement dans la région cible si vous souhaitez tester la configuration avant d’effectuer le déplacement final vers la région cible. Cette pratique est recommandée dans la mesure où elle limite les interférences avec l’environnement de production.

## <a name="copy-data-to-the-target-region"></a>Copier les données vers la région cible
Suivez les étapes ci-dessous pour copier les données vers la région cible à l’aide d’Azure Site Recovery.

### <a name="create-the-vault-in-any-region-except-the-source-region"></a>Créez le coffre dans n’importe quelle région, à l’exception de la région source.

1. Connectez-vous au [portail Azure](https://portal.azure.com) > **Recovery Services**.
2. Cliquez sur **Créer une ressource** > **Outils de gestion** > **Backup and Site Recovery**.
3. Dans **Nom**, indiquez le nom convivial **ContosoVMVault**. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié.
4. Créez un groupe de ressources **ContosoRG**.
5. Spécifiez une région Azure. Pour découvrir les régions prises en charge, référez-vous à la disponibilité géographique de la page [Tarification de Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. Dans Coffres Recovery Services, cliquez sur **Vue d’ensemble** > **ConsotoVMVault** >  **+ Répliquer**
7. Sélectionnez **Vers Azure** > **Non virtualisé / Autre**.

### <a name="set-up-the-configuration-server-to-discover-vms"></a>Configurer le serveur de configuration pour détecter des machines virtuelles.


Configurez le serveur de configuration, inscrivez-le dans le coffre et découvrez les machines virtuelles.

1. Cliquez sur **Site Recovery** > **Préparer l’infrastructure** > **Source**.
2. Si vous n’avez pas de serveur de configuration, cliquez sur **+Serveur de configuration**.
3. Dans **Ajouter un serveur**, vérifiez que **Serveur de configuration** s’affiche dans **Type de serveur**.
4. Téléchargez le fichier d’installation unifiée de Site Recovery.
5. Téléchargez la clé d’inscription du coffre. Vous en aurez besoin lors de l’exécution du programme d’installation unifiée. Une fois générée, la clé reste valide pendant 5 jours.

   ![Configurer la source](./media/physical-azure-disaster-recovery/source-environment.png)


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

Une fois l’inscription terminée, le serveur de configuration s’affiche sur la page **Paramètres** > **Serveurs** du coffre.

### <a name="configure-target-settings-for-replication"></a>Configurer les paramètres cibles pour la réplication

Sélectionnez et vérifiez les ressources cibles.

1. Cliquez sur **Préparer l’infrastructure** > **Cible** et sélectionnez l’abonnement Azure à utiliser.
2. Spécifiez le modèle de déploiement cible.
3. Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles.

   ![Cible](./media/physical-azure-disaster-recovery/network-storage.png)


### <a name="create-a-replication-policy"></a>Créer une stratégie de réplication

1. Pour créer une nouvelle stratégie de réplication, cliquez sur **Infrastructure de Site Recovery** > **Stratégies de réplication** >  **+Stratégie de réplication**.
2. Dans **Créer une stratégie de réplication**, indiquez le nom de la stratégie.
3. Dans **Seuil d’objectif de point de récupération**, spécifiez la limite de l’objectif de point de récupération (RPO). Cette valeur spécifie la fréquence à laquelle les points de récupération des données sont créés. Une alerte est générée lorsque la réplication continue dépasse cette limite.
4. Dans **Rétention des points de récupération**, spécifiez la durée de la fenêtre de rétention pour chaque point de récupération (en heures). Les machines virtuelles répliquées peuvent être récupérées à n’importe quel point dans une fenêtre. Les machines virtuelles répliquées vers Premium Storage peuvent prendre en charge jusqu’à 24 heures de rétention et 72 heures en cas de stockage standard.
5. Dans **Fréquence des captures instantanées de cohérence d’application**, spécifiez la fréquence de création des points de récupération contenant des captures instantanées cohérentes avec les applications (en minutes). Cliquez sur **OK** pour créer la stratégie.

    ![Stratégie de réplication](./media/physical-azure-disaster-recovery/replication-policy.png)


La stratégie est automatiquement associée au serveur de configuration. Par défaut, une stratégie de correspondance est automatiquement créée pour la restauration automatique. Par exemple, si la stratégie de réplication est **rep-policy**, la stratégie de restauration automatique **rep-policy-failback** est créée. Cette stratégie n’est utilisée qu’à partir du moment où vous initiez une restauration automatique à partir d’Azure.

### <a name="enable-replication"></a>Activer la réplication

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

   > [!WARNING]
   > Vous devez entrer l’adresse IP de la machine virtuelle Azure que vous avez l’intention de déplacer

10. Dans **Propriétés** > **Configurer les propriétés**, sélectionnez le compte à utiliser par le serveur de processus pour installer automatiquement le service Mobilité sur la machine.
11. Dans **Paramètres de réplication** > **Configurer les paramètres de réplication**, vérifiez que la stratégie de réplication correcte est sélectionnée. 
12. Cliquez sur **Activer la réplication**. Vous pouvez suivre la progression du travail **Activer la protection** dans **Paramètres** > **Travaux** > **Travaux Site Recovery**. Une fois le travail **Finaliser la protection** exécuté, la machine est prête pour le basculement.


Pour effectuer le monitoring des serveurs, vous pouvez consulter l’heure de leur dernière détection dans **Serveurs de configuration** > **Dernier contact à**. Pour ajouter des machines sans attendre l’heure de la détection planifiée, mettez en surbrillance le serveur de configuration (sans cliquer dessus) et cliquez sur **Actualiser**.

## <a name="test-the-configuration"></a>Tester la configuration


1. Accédez au coffre puis, dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine virtuelle que vous souhaitez déplacer vers la région cible et cliquez sur l’icône **+ Test de basculement**.
2. Dans **Test de basculement**, sélectionnez un point de récupération à utiliser pour le basculement :

   - **Dernier point traité** : bascule la machine virtuelle vers le dernier point de récupération qui a été traité par le service Site Recovery. L’horodatage est affiché. Cette option, qui n’implique aucun traitement de données, offre un objectif de délai de récupération faible.
   - **Dernier point de cohérence des applications** : cette option bascule toutes les machines virtuelles vers le dernier point de récupération de cohérence des applications. L’horodatage est affiché.
   - **Personnalisé** : sélectionnez n’importe quel point de récupération.

3. Sélectionnez le réseau virtuel Azure cible vers lequel vous voulez déplacer les machines virtuelles Azure afin de tester la configuration. 

   > [!IMPORTANT]
   > Nous vous recommandons d’utiliser un réseau machines virtuelles Azure distinct pour faire le test de basculement, et pas le réseau de production vers lequel vous comptez déplacer vos machines virtuelles au final, qui a été configuré lorsque vous avez activé la réplication.

4. Pour démarrer le test du déplacement, cliquez sur **OK**. Pour suivre la progression, cliquez sur la machine virtuelle pour ouvrir ses propriétés. Vous pouvez également cliquer sur le travail **Test de basculement** dans le nom du coffre > **Paramètres** > **Travaux** > **Travaux Site Recovery**.
5. Une fois le basculement terminé, la machine virtuelle Azure de réplication apparaît dans le portail Azure > **Machines virtuelles**. Vérifiez que la machine virtuelle est en cours d’exécution, qu’elle est correctement dimensionnée et qu’elle est connectée au réseau approprié.
6. Si vous souhaitez supprimer la machine virtuelle qui a été créée pour les besoins du test du déplacement, cliquez sur **Nettoyer le test de basculement** sur l’élément répliqué. Cliquez sur **Notes** pour consigner et enregistrer d’éventuelles observations sur le test.

## <a name="perform-the-move-to-the-target-region-and-confirm"></a>Effectuez et validez le déplacement vers la région cible.

1. Accédez au coffre puis, dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine virtuelle et ensuite sur **Basculement**.
2. Dans **Basculement**, sélectionnez **Dernier**. 
3. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter la machine virtuelle source avant de déclencher le basculement. Le basculement est effectué même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**. 
4. Une fois que la tâche est terminée, vérifiez que la machine virtuelle apparaît bien dans la région Azure cible choisie.
5. Dans **Éléments répliqués**, cliquez avec le bouton droit sur la machine virtuelle > **Valider**. Cette étape termine le processus de déplacement vers la région cible. Attendez la fin de la tâche de validation.

## <a name="discard-the-resource-in-the-source-region"></a>Supprimer les ressources dans la région source 

- Accédez à la machine virtuelle.  Cliquez sur **Désactiver la réplication**.  Cela met fin au processus de copie des données pour la machine virtuelle.  

   > [!IMPORTANT]
   > Il est important d’effectuer cette étape pour arrêter la facturation liée à la réplication Azure Site Recovery.

Si vous n’envisagez pas de réutiliser les ressources sources, effectuez les étapes suivantes.

1. Supprimez toutes les ressources réseau appropriées dans la région source (celles que vous avez listées à l’étape 4 dans [Préparer les machines virtuelles sources](#prepare-the-source-vms)). 
2. Supprimez le compte de stockage associé dans la région source.



## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé une machine virtuelle Azure vers une autre région. Vous pouvez maintenant configurer la reprise d’activité après sinistre pour la machine virtuelle que vous avez déplacée.

> [!div class="nextstepaction"]
> [Configurer la récupération d’urgence après la migration](azure-to-azure-quickstart.md)
