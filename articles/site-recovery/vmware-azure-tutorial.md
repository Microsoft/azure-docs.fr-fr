---
title: Configurer la reprise d’activité après sinistre des machines virtuelles VMware sur Azure à l’aide d’Azure Site Recovery
description: Découvrez comment configurer la récupération d’urgence de machines virtuelles VMware locales vers Azure avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 11/12/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: bd87265140a0bfaeb7ef4dada6dd76be1269654b
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369367"
---
# <a name="set-up-disaster-recovery-to-azure-for-on-premises-vmware-vms"></a>Configurer la récupération d’urgence vers Azure pour des machines virtuelles VMware locales

Cet article explique comment activer la réplication des machines virtuelles VMware locales pour permettre la reprise d’activité sur Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md).

Il s’agit du troisième tutoriel d’une série qui montre comment configurer la reprise d’activité sur Azure pour des machines virtuelles VMware locales. Dans le tutoriel précédent, nous avons [préparé l’environnement VMware local](vmware-azure-tutorial-prepare-on-premises.md) à la reprise d’activité sur Azure.


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer les paramètres de réplication source et un serveur de configuration Site Recovery local
> * Configurer les paramètres de la cible de réplication
> * Créer une stratégie de réplication.
> * Activer la réplication d’une machine virtuelle VMware

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour obtenir des instructions détaillées, consultez l’article de la section Procédures dans la table des matières de Site Recovery.

## <a name="before-you-start"></a>Avant de commencer

Effectuez les tutoriels précédents :
1. Vérifiez que vous avez [configuré Azure](tutorial-prepare-azure.md) pour permettre la reprise d’activité VMware locale sur Azure.
2. Suivez [ces étapes](vmware-azure-tutorial-prepare-on-premises.md) pour préparer votre déploiement VMware local à la reprise d’activité sur Azure.
3. Ce didacticiel explique comment répliquer une machine virtuelle unique. Si vous déployez plusieurs machines virtuelles VMware, vous devez utiliser l’[outil Planificateur de déploiement](https://aka.ms/asr-deployment-planner). [En savoir plus](site-recovery-deployment-planner.md) sur cet outil.
4. Ce tutoriel utilise plusieurs options que vous pouvez suivre de manière différente :
    - Le tutoriel utilise un modèle OVA pour créer la machine virtuelle VMware du serveur de configuration. Si vous ne pouvez pas le faire pour une raison quelconque, suivez [ces instructions](physical-manage-configuration-server.md) afin de configurer le serveur de configuration manuellement.
    - Dans ce tutoriel, Site Recovery télécharge et installe automatiquement MySQL sur le serveur de configuration. Si vous préférez, vous pouvez l’installer manuellement. [Plus d’informations](vmware-azure-deploy-configuration-server.md#configure-settings)




## <a name="select-a-protection-goal"></a>Sélectionner un objectif de protection

1. Dans **Archivages de Recovery Services** , sélectionnez le nom de l’archivage. Nous utilisons **ContosoVMVault** pour ce scénario.
2. Dans **Prise en main** , sélectionnez Site Recovery. Sélectionnez ensuite **Préparer l’infrastructure** .
3. Dans **Objectif de protection** > **Où se trouvent vos machines** , sélectionnez **Local** .
4. Dans **Où voulez-vous répliquer vos machines** , sélectionnez **Dans Azure** .
5. Dans **Vos machines sont-elles virtualisées** , sélectionnez **Oui, avec l’hyperviseur VMware vSphere** . Sélectionnez ensuite **OK** .



## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

Dans votre environnement source, vous avez besoin d’une seule machine locale hautement disponible pour héberger les composants Site Recovery locaux :

- **Serveur de configuration**  : Le serveur de configuration coordonne la communication entre les ordinateurs locaux et Azure.et gère la réplication des données.
- **Serveur de traitement** : Le serveur de processus fait office de passerelle de réplication. Il reçoit les données de réplication, les optimise grâce à une mise en cache, une compression et un chiffrement, puis les envoie à un compte de stockage de cache dans Azure. De plus, le serveur de processus installe l’agent du service Mobilité sur les machines virtuelles que vous souhaitez répliquer, puis effectue la détection automatique des machines virtuelles VMware locales.
- **Serveur cible maître**  : Le serveur cible maître gère les données de réplication pendant la restauration automatique à partir d’Azure.


Tous ces composants sont installés ensemble sur une seule machine locale, appelée *serveur de configuration* . Par défaut, pour permettre la reprise d’activité VMware, nous configurons le serveur de configuration en tant que machine virtuelle VMware hautement disponible. Pour ce faire, vous téléchargez un modèle OVA (Open Virtualization Application) préparé, que vous importez dans VMware afin de créer la machine virtuelle.

- La dernière version du serveur de configuration est disponible sur le portail. Vous pouvez également la télécharger directement à partir du [Centre de téléchargement Microsoft](https://aka.ms/asrconfigurationserver).
- Si, pour une raison quelconque, vous ne pouvez pas utiliser un modèle OVA pour configurer une machine virtuelle, suivez [ces instructions](physical-manage-configuration-server.md) afin de configurer le serveur de configuration manuellement.
- La licence fournie avec le modèle OVF est une licence d’évaluation valide pendant 180 jours. Windows sur la machine virtuelle doit être activé avec la licence requise.


### <a name="download-the-vm-template"></a>Télécharger le modèle de machine virtuelle

1. Dans le coffre, allez dans **Préparer l’infrastructure** > **Source** .
2. Dans **Préparer la source** , sélectionnez **+Serveur de configuration** .
3. Dans **Ajouter un serveur** , vérifiez que **Serveur de configuration pour VMware** s’affiche dans **Type de serveur** .
4. Téléchargez le modèle OVA pour le serveur de configuration.



## <a name="import-the-template-in-vmware"></a>Importer le modèle dans VMware


1. Connectez-vous au serveur VMware vCenter ou à l’hôte vSphere ESXi, à l’aide du client VMware vSphere.
2. Dans le menu **Fichier** , sélectionnez **Déployer le modèle OVF** pour démarrer l’ **Assistant Déploiement du modèle OVF** .

     ![Capture d’écran de la commande Déployer le modèle OVF dans le client VMWare vSphere](./media/vmware-azure-tutorial/vcenter-wizard.png)

3. Dans **Sélectionner une source** , entrez l’emplacement du modèle OVF téléchargé.
4. Dans **Examiner les détails** , sélectionnez **Suivant** .
5. Dans **Sélectionner le nom et le dossier** et **Sélectionner la configuration** , acceptez les paramètres par défaut.
6. Dans **Sélectionner un stockage** , pour des performances optimales, sélectionnez **Remise à zéro rapide d’un approvisionnement important** dans **Sélectionner le format de disque virtuel** .
7. Dans le reste des pages de l’Assistant, acceptez les paramètres par défaut.
8. Sur la fenêtre **Prêt à terminer** , pour configurer la machine virtuelle avec les paramètres par défaut, sélectionnez **Mettre sous tension après le déploiement** > **Terminer** .

   > [!TIP]
   > Si vous souhaitez ajouter une carte réseau supplémentaire, décochez la case **Mettre sous tension après le déploiement** > **Terminer** . Par défaut, le modèle contient une seule carte réseau. mais vous pouvez en ajouter d’autres après le déploiement.

## <a name="add-an-additional-adapter"></a>Ajouter une carte supplémentaire

Si vous souhaitez ajouter une carte d’interface réseau supplémentaire au serveur de configuration, faites-le avant d’inscrire le serveur dans le coffre. L’ajout de cartes supplémentaires n’est pas pris en charge après l’inscription.

1. Dans l’inventaire du client vSphere, faites un clic droit sur la machine virtuelle, puis sélectionnez **Modifier les paramètres** .
2. Dans **Matériel** , sélectionnez **Ajouter** > **Adaptateur Ethernet** . Sélectionnez ensuite **Suivant** .
3. Sélectionnez un type d’adaptateur et un réseau.
4. Pour connecter la carte réseau virtuelle lorsque la machine virtuelle est allumée, sélectionnez **Connecter à la mise sous tension** . Sélectionnez **Suivant** > **Terminer** . Sélectionnez ensuite **OK** .


## <a name="register-the-configuration-server"></a>Inscrire le serveur de configuration

Une fois le serveur de configuration configuré, vous devez l’inscrire dans le coffre.

1. À partir de la console du client VMware vSphere, activez la machine virtuelle.
2. La machine virtuelle démarre sur une expérience d’installation de Windows Server 2016. Acceptez le contrat de licence et entrez un mot de passe administrateur.
3. Une fois l’installation terminée, connectez-vous à la machine virtuelle en tant qu’administrateur.
4. L’outil de configuration d’Azure Site Recovery démarre en quelques secondes la première fois que vous vous connectez.
5. Saisissez un nom utilisé pour inscrire le serveur de configuration sur Site Recovery. Sélectionnez ensuite **Suivant** .
6. L’outil vérifie que la machine virtuelle peut se connecter à Azure. Une fois la connexion établie, sélectionnez **Connecter** pour vous connecter à votre abonnement Azure. Les informations d’identification doivent avoir accès au coffre dans lequel vous souhaitez inscrire le serveur de configuration. Vérifiez que les [rôles](vmware-azure-deploy-configuration-server.md#azure-active-directory-permission-requirements) nécessaires sont attribués à cet utilisateur.
7. L’outil effectue des tâches de configuration, puis redémarre.
8. Reconnectez-vous à la machine. En quelques secondes, l’Assistant Gestion de serveur de configuration démarre automatiquement.


### <a name="configure-settings-and-add-the-vmware-server"></a>Configurer les paramètres et ajouter le serveur VMware

Finissez l’installation et l’inscription du serveur de configuration. Avant de continuer, vérifiez que tous les [prérequis](vmware-azure-deploy-configuration-server.md#prerequisites) sont respectés pour la configuration du serveur de configuration.


1. Dans l’assistant de gestion de serveur de configuration, sélectionnez **Configurer la connectivité** . Dans les menus déroulants, commencez par sélectionner la carte réseau que le serveur de processus intégré utilise pour l’installation push du service Mobilité sur les machines sources, puis sélectionnez la carte réseau que le serveur de configuration utilise pour la connectivité avec Azure. Ensuite, sélectionnez **Enregistrer** . Vous ne pouvez pas modifier ce paramètre une fois qu’il a été configuré.
2. Dans **Sélectionner le coffre Recovery Services** , sélectionnez votre abonnement Azure ainsi que le groupe de ressources et le coffre appropriés.
3. Dans **Installer le logiciel tiers** , acceptez le contrat de licence. Sélectionnez **Télécharger et installer** pour installer MySQL Server. Si vous avez placé MySQL dans le chemin, vous pouvez ignorer cette étape. En savoir [plus](vmware-azure-deploy-configuration-server.md#configure-settings)
4. Dans **Valider la configuration de l’appliance** , les conditions préalables sont vérifiées avant que vous ne poursuiviez.
5. Dans **Configurer le serveur vCenter Server/vSphere ESXi** , saisissez le nom de domaine complet ou l’adresse IP du serveur vCenter Server, ou l’hôte vSphere, sur lequel sont situées les machines virtuelles que vous souhaitez répliquer. Entrez le numéro de port sur lequel le serveur écoute. Entrez un nom convivial à utiliser pour le serveur VMware dans le coffre.
6. Entrez les informations d’identification utilisateur à utiliser par le serveur de configuration pour se connecter au serveur VMware. Vérifiez que le nom d’utilisateur et le mot de passe sont corrects et qu’il fait partie du groupe Administrateurs de la machine virtuelle à protéger. Site Recovery utilise ces informations d’identification pour détecter automatiquement les machines virtuelles VMware disponibles pour la réplication. Sélectionnez **Ajouter** , puis **Continuer** .
7. Dans **Configurer les informations d’identification de la machine virtuelle** , entrez le nom d’utilisateur et le mot de passe qui seront utilisés pour installer automatiquement le service Mobilité sur les machines virtuelles, une fois la réplication activée.
    - Pour les machines Windows, le compte doit disposer des privilèges d’administrateur local sur les machines que vous souhaitez répliquer.
    - Pour Linux, fournissez les détails du compte racine.
8. Sélectionnez **Finaliser la configuration** pour terminer l’inscription.
9. Une fois l’inscription terminée, ouvrez le Portail Azure, vérifiez que le serveur de configuration et le serveur VMware sont listés dans **Coffre Recovery Services** > **Gérer** > **Infrastructure Site Recovery** > **Serveurs de configuration** .


Une fois le serveur de configuration inscrit, Site Recovery se connecte aux serveurs VMware à l’aide des paramètres spécifiés et découvre les machines virtuelles.

> [!NOTE]
> L’affichage du nom de compte dans le portail peut prendre plus de 15 minutes. Pour procéder à une mise à jour immédiate, sélectionnez **Serveurs de configuration** > **_nom du serveur_ *_ > _* Actualiser le serveur** .

## <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

Sélectionnez et vérifiez les ressources cibles.

1. Sélectionnez **Préparer l’infrastructure** > **Cible** . Sélectionnez l’abonnement Azure à utiliser. Nous utilisons un modèle Resource Manager.
2. Site Recovery vérifie que vous disposez d’un ou plusieurs réseaux virtuels. Vous devez en disposer lors de la configuration des composants Azure dans le [premier didacticiel](tutorial-prepare-azure.md) de cette série.

   ![Capture d’écran des options Préparer l’infrastructure > Cible](./media/vmware-azure-tutorial/storage-network.png)

## <a name="create-a-replication-policy"></a>Créer une stratégie de réplication

1. Ouvrez le [portail Azure](https://portal.azure.com). Recherchez et sélectionnez **Coffres Recovery Services** .
2. Sélectionnez le coffre Recovery Services ( **ContosoVMVault** dans ce didacticiel).
3. Pour créer une stratégie de réplication, sélectionnez **Infrastructure Azure Site Recovery** > **Stratégies de réplication** >  **+Stratégie de réplication** .
4. Dans **Créer une stratégie de réplication** , entrez le nom de stratégie. Nous utilisons **VMwareRepPolicy** .
5. Dans **Seuil RPO** , utilisez la valeur par défaut de 60 minutes. Cette valeur définit la fréquence à laquelle les points de récupération sont créés. Une alerte est générée lorsque la réplication continue dépasse cette limite.
6. Dans **Rétention des points de récupération** , précisez le temps de rétention de chaque point de récupération. Pour ce didacticiel, nous utilisons 72 heures. Les machines virtuelles répliquées peuvent être récupérées à n’importe quel point dans une fenêtre de rétention.
7. Dans **Fréquence des captures instantanées cohérentes au niveau de l’application** , spécifiez la fréquence de création des captures instantanées cohérentes au niveau de l’application. Nous utilisons la valeur par défaut, 60 minutes. Sélectionnez **OK** pour créer la stratégie.

   ![Capture d’écran des options Créer une stratégie de réplication](./media/vmware-azure-tutorial/replication-policy.png)

- La stratégie est automatiquement associée au serveur de configuration.
- Par défaut, une stratégie de correspondance est automatiquement créée pour la restauration automatique. Par exemple, si la stratégie de réplication est **rep-policy** , la stratégie de restauration automatique correspond alors à **rep-policy-failback** . Cette stratégie n’est utilisée qu’à partir du moment où vous initiez une restauration automatique à partir d’Azure.

Remarque : Dans le scénario VMware vers Azure, l’instantané de cohérence en cas d’incident est pris à un intervalle de 5 minutes.

## <a name="enable-replication"></a>Activer la réplication

Activez la réplication des machines virtuelles comme suit :

1. Sélectionnez **Répliquer l’application** > **Source** .
2. Dans **Source** , sélectionnez **Local** , puis le serveur de configuration dans **Emplacement source** .
3. Dans **Type de machine** , sélectionnez **Machines virtuelles** .
4. Dans **Hyperviseur vCenter/vSphere** , sélectionnez l’hôte vSphere, ou le serveur vCenter qui gère l’hôte.
5. Sélectionnez le serveur de processus (installé par défaut sur la machine virtuelle du serveur de configuration). Sélectionnez ensuite **OK** . L'état d’intégrité de chaque serveur de traitement est indiqué en fonction des limites recommandées et d’autres paramètres. Sélectionnez un serveur de traitement sain. Un serveur de traitement [critique](vmware-physical-azure-monitor-process-server.md#process-server-alerts) ne peut pas être sélectionné. Vous pouvez [détecter et résoudre](vmware-physical-azure-troubleshoot-process-server.md) les erreurs **ou** configurer un [serveur de traitement de montée en puissance](vmware-azure-set-up-process-server-scale.md).
6. Dans **Cible** , sélectionnez l’abonnement et le groupe de ressources dans lesquels vous voulez créer les machines virtuelles basculées. Nous utilisons le modèle de déploiement Resource Manager.
7. Sélectionnez le sous-réseau et le réseau Azure auxquels les machines virtuelles Azure se connectent lorsqu’elles sont créées après le basculement.
8. Sélectionnez **Effectuez maintenant la configuration pour les machines sélectionnées** pour appliquer les paramètres réseau à l’ensemble des machines virtuelles pour lesquelles vous activez la réplication. Sélectionnez **Configurer ultérieurement** pour sélectionner le réseau Azure pour chaque machine.
9. Dans **Machines virtuelles** > **Sélectionner les machines virtuelles** , sélectionnez chaque machine à répliquer. Vous pouvez uniquement sélectionner les machines pour lesquelles la réplication peut être activée. Sélectionnez ensuite **OK** . Si vous ne parvenez pas à voir/sélectionner une machine virtuelle particulière, vous pouvez chercher à [en savoir plus](./vmware-azure-troubleshoot-replication.md) sur la résolution du problème.
10. Dans **Propriétés** > **Configurer les propriétés** , sélectionnez le compte que le serveur de processus doit utiliser pour installer automatiquement le service Mobilité sur la machine.
11. Dans **Paramètres de réplication** > **Configurer les paramètres de réplication** , vérifiez que la stratégie de réplication correcte est sélectionnée.
12. Sélectionnez **Activer la réplication** . Site Recovery installe automatiquement le service Mobilité quand la réplication est activée pour une machine virtuelle.
13. Vous pouvez suivre la progression du travail **Activer la protection** dans **Paramètres** > **Travaux** > **Travaux Site Recovery** . Après l’exécution du travail **Finaliser la protection** , et une fois la génération du point de récupération terminée, la machine est prête pour le basculement.
14. Il peut être nécessaire d’attendre 15 minutes ou plus avant que les modifications prennent effet et qu’elles apparaissent dans le portail.
15. Pour surveiller les machines virtuelles que vous ajoutez, consultez l’heure de la dernière découverte des machines virtuelles dans **Serveurs de configuration** > **Dernier contact à** . Pour ajouter des machines virtuelles sans attendre la découverte planifiée, mettez en surbrillance le serveur de configuration (sans le sélectionner) et sélectionnez **Actualiser** .

## <a name="next-steps"></a>Étapes suivantes
Après avoir activé la réplication, exécutez une simulation pour vérifier que tout fonctionne comme prévu.
> [!div class="nextstepaction"]
> [Effectuer un test de récupération d’urgence](site-recovery-test-failover-to-azure.md)