---
title: Déployer le serveur de configuration dans Azure Site Recovery
description: Cet article explique comment déployer un serveur de configuration pour la récupération d’urgence de VMware avec Azure Site Recovery
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 10/15/2019
ms.author: ramamill
ms.openlocfilehash: a8ceb3df68ebe42f83c70ed62327bf59c0dfc225
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92359813"
---
# <a name="deploy-a-configuration-server"></a>Déployer un serveur de configuration

Vous devez déployer un serveur de configuration local quand vous utilisez [Azure Site Recovery](site-recovery-overview.md) pour la récupération d’urgence de machines virtuelles VMware et de serveurs physiques sur Azure. Le serveur de configuration coordonne la communication entre les machines locales VMware et Azure. Il gère également la réplication des données. Cet article vous guide tout au long des étapes nécessaires pour déployer le serveur de configuration lorsque vous répliquez des machines virtuelles WMware sur Azure. Si vous devez configurer un serveur de configuration pour la réplication de serveur physique, consultez [Configurer le serveur de configuration pour la récupération d’urgence de serveurs physiques sur Azure](physical-azure-set-up-source.md).

> [!TIP]
> Pour en savoir plus sur le rôle du serveur de configuration dans l’architecture d’Azure Site Recovery, voir [Architecture pour la reprise d’activité de VMware sur Azure](vmware-azure-architecture.md).

## <a name="deploy-a-configuration-server-through-an-ova-template"></a>Déployer un serveur de configuration avec un modèle OVA

Le serveur de configuration doit être configuré comme une machine virtuelle VMware hautement disponible avec certaines exigences matérielles et dimensionnelles minimales. Pour un déploiement simple et pratique, Site Recovery fournit un modèle OVA (Open Virtualization Application) téléchargeable pour configurer le serveur de configuration conformément à toutes les exigences listées ci-dessous.

## <a name="prerequisites"></a>Conditions préalables requises

Les conditions matérielles minimales pour un serveur de configuration sont synthétisées dans les sections suivantes.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

## <a name="azure-active-directory-permission-requirements"></a>Autorisations Azure Active Directory requises

Vous devez avoir un utilisateur avec une de ces autorisations définie dans AAD (Azure Active Directory) pour inscrire le serveur de configuration auprès de services Azure Site Recovery.

1. L’utilisateur doit avoir le rôle « Développeur d’applications » pour créer une application.
    - Pour vérifier, connectez-vous au Portail Azure.</br>
    - Accédez à **Azure Active Directory** > **Rôles et administrateurs** .</br>
    - Vérifiez si le rôle « Développeur d’applications » est attribué à l’utilisateur. Si ce n’est pas le cas, utilisez un utilisateur disposant de cette autorisation ou contacter l’[administrateur pour activer l’autorisation](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md#assign-roles).
    
2. Si le rôle « Développeur d’applications » ne peut pas être attribué, vérifiez que l’indicateur **Les utilisateurs peuvent inscrire des applications** est défini sur **true** pour que l’utilisateur puisse créer une identité. Pour activer ces autorisations :
    - Connectez-vous au portail Azure.
    - Accédez à **Azure Active Directory** > **Paramètres utilisateur** .
    - Sous **Inscription des applications** , **Les utilisateurs peuvent inscrire des applications** , sélectionnez **Oui** .

      ![Azure AD_application_permission](media/vmware-azure-deploy-configuration-server/AAD_application_permission.png)

> [!NOTE]
> Les services de fédération Active Directory (AD FS) ne *sont pas pris en charge* . Utilisez un compte managé via [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md).

## <a name="download-the-template"></a>Téléchargez le modèle

1. Dans le coffre, allez dans **Préparer l’infrastructure** > **Source** .
2. Dans **Préparer la source** , sélectionnez **+Serveur de configuration** .
3. Dans **Ajouter un serveur** , vérifiez que **Serveur de configuration pour VMware** s’affiche dans **Type de serveur** .
4. Téléchargez le modèle OVA pour le serveur de configuration.

   > [!TIP]
   >Vous pouvez également télécharger directement la dernière version du modèle de serveur de configuration dans le [Centre de téléchargement Microsoft](https://aka.ms/asrconfigurationserver).

> [!NOTE]
> La licence fournie avec un modèle OVA est une licence d’évaluation valide pendant 180 jours. Au-delà de cette période, vous devez acquérir une licence.

## <a name="import-the-template-in-vmware"></a>Importer le modèle dans VMware

1. Connectez-vous au serveur VMware vCenter ou à l’hôte vSphere ESXi en utilisant le client VMware vSphere.
2. Dans le menu **Fichier** , sélectionnez **Déployer le modèle OVF** pour démarrer l’ **Assistant Déploiement du modèle OVF** .

     ![Déployer le modèle OVF](./media/vmware-azure-deploy-configuration-server/vcenter-wizard.png)

3. Dans **Sélectionner une source** , entrez l’emplacement du modèle OVF téléchargé.
4. Dans **Examiner les détails** , sélectionnez **Suivant** .
5. Dans **Sélectionner le nom et le dossier** et **Sélectionner la configuration** , acceptez les paramètres par défaut.
6. Dans **Sélectionner un stockage** , pour des performances optimales, sélectionnez **Remise à zéro rapide d’un approvisionnement important** dans **Sélectionner le format de disque virtuel** . L'utilisation de l'option d'allocation dynamique peut avoir un impact sur les performances du serveur de configuration.
7. Dans le reste des pages de l’Assistant, acceptez les paramètres par défaut.
8. Dans **Prêt à finaliser** :

    * Pour configurer la machine virtuelle avec les paramètres par défaut, sélectionnez **Mettre sous tension après le déploiement** > **Terminer** .
    * Pour ajouter une interface réseau supplémentaire, désactivez **Mettre sous tension après le déploiement** , puis sélectionnez **Terminer** . Par défaut, le modèle de serveur de configuration est déployé avec une seule carte d’interface réseau, mais vous pouvez en ajouter d’autres après le déploiement.

> [!IMPORTANT]
> Veillez à ne pas modifier les configurations de ressources (restriction mémoire/cœurs/UC), ou modifier ou supprimer des services ou fichiers installés sur le serveur de configuration après le déploiement. Ces types de modifications affectent l’inscription du serveur de configuration avec les services Azure et les performances du serveur de configuration.

## <a name="add-an-additional-adapter"></a>Ajouter une carte supplémentaire

> [!NOTE]
> Si vous envisagez de conserver les adresses IP des ordinateurs sources lors d’un basculement et que vous souhaitez par la suite effectuer une restauration automatique localement, vous avez besoin de deux cartes réseau. Une carte réseau est connectée aux ordinateurs sources et l’autre est utilisée pour connectivité Azure.

Si vous souhaitez ajouter une carte d’interface réseau supplémentaire au serveur de configuration, faites-le avant d’inscrire le serveur dans le coffre. L’ajout de cartes supplémentaires n’est pas pris en charge après l’inscription.

1. Dans l’inventaire du client vSphere, faites un clic droit sur la machine virtuelle, puis sélectionnez **Modifier les paramètres** .
2. Dans **Matériel** , sélectionnez **Ajouter** > **Adaptateur Ethernet** . Sélectionnez ensuite **Suivant** .
3. Sélectionnez un type d’adaptateur et un réseau.
4. Pour connecter la carte réseau virtuelle lorsque la machine virtuelle s’allume, sélectionnez **Connecter à la mise sous tension** . Ensuite, sélectionnez **Suivant** > **Terminer** > **OK** .

## <a name="register-the-configuration-server-with-azure-site-recovery-services"></a>Inscrire le serveur de configuration auprès des services Azure Site Recovery

1. À partir de la console du client VMware vSphere, activez la machine virtuelle.
2. La machine virtuelle démarre sur une expérience d’installation de Windows Server 2016. Acceptez le contrat de licence et entrez un mot de passe administrateur.
3. Une fois l’installation terminée, connectez-vous à la machine virtuelle en tant qu’administrateur.
4. Lors de la première connexion, l’outil de configuration d’Azure Site Recovery démarre en quelques secondes.
5. Saisissez un nom utilisé pour inscrire le serveur de configuration sur Site Recovery. Sélectionnez ensuite **Suivant** .
6. L’outil vérifie que la machine virtuelle peut se connecter à Azure. Une fois la connexion établie, sélectionnez **Connecter** pour vous connecter à votre abonnement Azure.</br>
    a. Les informations d’identification doivent avoir accès au coffre dans lequel vous souhaitez inscrire le serveur de configuration.</br>
    b. Assurez-vous que le compte d’utilisateur choisi dispose d’autorisations pour créer une application dans Azure. Pour activer les autorisations requises, suivez les instructions de la section [Exigences relatives aux autorisations Azure Active Directory](#azure-active-directory-permission-requirements).
7. L’outil effectue des tâches de configuration, puis redémarre.
8. Reconnectez-vous à la machine. L’Assistant Gestion du serveur de configuration démarre automatiquement au bout de quelques secondes.

### <a name="configure-settings"></a>Configurer les paramètres

1. Dans l’assistant de gestion de serveur de configuration, sélectionnez **Configurer la connectivité** . Dans les menus déroulants, commencez par sélectionner la carte réseau que le serveur de processus intégré utilise pour l’installation push du service Mobilité sur les machines sources. Sélectionnez ensuite la carte réseau que le serveur de configuration utilise pour la connectivité avec Azure. Sélectionnez **Enregistrer** . Vous ne pouvez pas modifier ce paramètre une fois qu’il a été configuré. Ne modifiez pas l’adresse IP d’un serveur de configuration. Vérifiez que l’adresse IP affectée au serveur de configuration est une adresse IP statique et pas une adresse IP DHCP.
2. Dans **Sélectionner un coffre Recovery Services** , connectez-vous à Microsoft Azure avec les informations d’identification utilisées à l’étape 6 de [Inscrire le serveur de configuration auprès des services Azure Site Recovery](#register-the-configuration-server-with-azure-site-recovery-services).
3. Une fois connecté, sélectionnez votre abonnement Azure ainsi que le groupe de ressources et le coffre appropriés.

    > [!NOTE]
    > Une fois inscrit, le coffre Recovery Services n’est plus modifiable.
    > Le changement de coffre Recovery Services requiert une dissociation du serveur de configuration du coffre actuel, entraînant l’arrêt de la réplication de toutes les machines virtuelles protégées sur le serveur de configuration. Pour en savoir plus, consultez [Gérer le serveur de configuration pour la récupération d’urgence de machines virtuelles VMware](vmware-azure-manage-configuration-server.md#register-a-configuration-server-with-a-different-vault).

4. Dans **Installer des logiciels tiers** :

    |Scénario   |Procédure à suivre  |
    |---------|---------|
    |Peut-on télécharger et installer manuellement MySQL ?     |  Oui. Téléchargez l’application MySQL, placez-la dans le dossier **C:\Temp\ASRSetup** , puis installez-la manuellement. Après avoir accepté les conditions et sélectionné **Télécharger et installer** , le portail indique *Déjà installé* . Vous pourrez passer à l’étape suivante.       |
    |Est-il possible d’éviter de télécharger MySQL en ligne ?     |   Oui. Placez votre application d’installation MySQL dans le dossier **C:\Temp\ASRSetup** . Acceptez les termes du contrat, sélectionnez **Télécharger et installer** , et le portail utilise le programme d’installation que vous avez ajouté pour installer l’application. Une fois l’installation terminée, passez à l’étape suivante.    |
    |Je souhaite télécharger et installer MySQL avec Azure Site Recovery.    |  Acceptez le contrat de licence et sélectionnez **Télécharger et installer** . Une fois l’installation terminée, passez à l’étape suivante.       |

5. Dans **Valider la configuration de l’appliance** , les conditions préalables sont vérifiées avant que vous ne poursuiviez.
6. Dans **Configurer le serveur vCenter Server/vSphere ESXi** , saisissez le nom de domaine complet ou l’adresse IP du serveur vCenter Server, ou l’hôte vSphere, sur lequel sont situées les machines virtuelles que vous souhaitez répliquer. Entrez le numéro de port sur lequel le serveur écoute. Entrez un nom convivial à utiliser pour le serveur VMware dans le coffre.
7. Entrez les informations d’identification à utiliser par le serveur de configuration pour se connecter au serveur VMware. Site Recovery utilise ces informations d’identification pour détecter automatiquement les machines virtuelles VMware disponibles pour la réplication. Sélectionnez **Ajouter** > **Continuer** . Les informations d’identification entrées ici sont enregistrées localement.
8. Dans **Configurer les informations d’identification des machines virtuelles** , entrez le nom d’utilisateur et le mot de passe des machines virtuelles pour installer automatiquement le service Mobilité au cours de la réplication. Dans le cas de machines **Windows** , le compte doit disposer de privilèges d’administrateur local sur les machines à répliquer. Sous **Linux** , fournissez les informations du compte racine.
9. Sélectionnez **Finaliser la configuration** pour terminer l’inscription.
10. Une fois l’inscription terminée, ouvrez le Portail Azure, vérifiez que le serveur de configuration et le serveur VMware sont listés dans **Coffre Recovery Services** > **Gérer** > **Infrastructure Site Recovery** > **Serveurs de configuration** .

## <a name="upgrade-the-configuration-server"></a>Mettre à niveau le serveur de configuration

Pour mettre à niveau le serveur de configuration vers la dernière version, consultez [Gérer le serveur de configuration pour la récupération d’urgence de machines virtuelles VMware](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server). Pour obtenir des instructions sur la mise à niveau de tous les composants Site Recovery, voir [Mises à jour de service dans Site Recovery](service-updates-how-to.md).

## <a name="manage-the-configuration-server"></a>Gérer le serveur de configuration

Pour éviter toute interruption d’une réplication continue, assurez-vous que l’adresse IP du serveur de configuration ne change pas une fois celui-ci inscrit dans un coffre. Pour en savoir plus sur les tâches de gestion communes d’un serveur de configuration, consultez [Gérer le serveur de configuration pour la récupération d’urgence de machines virtuelles VMware](vmware-azure-manage-configuration-server.md).

## <a name="troubleshoot-deployment-issues"></a>Résoudre les problèmes de déploiement

Pour résoudre les problèmes de connectivité et de déploiement, consultez cet [article](vmware-azure-troubleshoot-configuration-server.md).

## <a name="faqs"></a>FAQ

* Pendant combien de temps la licence fournie sur le serveur de configuration déployé via OVF est-elle valide ? Que se passe-t-il si je ne réactive pas la licence ?

    La licence fournie avec un modèle OVA est une licence d’évaluation valide pendant 180 jours. Avant l’expiration, vous devez activer la licence. Sinon, cela peut entraîner un arrêt fréquent du serveur de configuration, et donc constituer une entrave aux activités de réplication. Pour en savoir plus, consultez [Gérer le serveur de configuration pour la récupération d’urgence de machines virtuelles VMware](vmware-azure-manage-configuration-server.md#update-windows-license).

* Puis-je utiliser la machine virtuelle sur laquelle le serveur de configuration est installé à des fins différentes ?

    Non. Utilisez la machine virtuelle uniquement pour le serveur de configuration. Veillez à suivre toutes les spécifications mentionnées dans [Conditions préalables](#prerequisites) pour une gestion efficace de la récupération d’urgence.
* Peut-on remplacer le coffre déjà inscrit dans le serveur de configuration par un nouveau ?

    Non. Une fois le coffre inscrit auprès du serveur de configuration, il n’est plus modifiable.
* Peut-on utiliser le même serveur de configuration pour protéger à la fois des machines physiques et des machines virtuelles ?

    Oui. Un même serveur de configuration peut être utilisé pour répliquer des machines physiques et virtuelles. Toutefois, une machine physique ne peut être restaurée automatiquement que vers une machine virtuelle VMware.
* Quel est le rôle d’un serveur de configuration et où est-il utilisé ?

    Pour plus d’informations sur le serveur de configuration et ses fonctionnalités, consultez [Architecture de la réplication VMware vers Azure](vmware-azure-architecture.md).
* Où peut-on trouver la dernière version du serveur de configuration ?

    Pour savoir comment mettre à niveau le serveur de configuration via le portail, voir [Mettre à niveau le serveur de configuration](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server). Pour obtenir des instructions sur la mise à niveau de tous les composants Site Recovery, voir [Mises à jour de service dans Site Recovery](./service-updates-how-to.md).
* Où puis-je télécharger la phrase secrète du serveur de configuration ?

    Pour télécharger la phrase secrète, consultez [Gérer le serveur de configuration pour la récupération d’urgence de machines virtuelles VMware](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase).
* Puis-je modifier la phrase secrète ?

    Non. Ne modifiez pas la phrase secrète du serveur de configuration. Modifier la phrase secrète arrête la réplication des machines protégées et entraîne un état d’intégrité critique.
* Où puis-je télécharger les clés d’inscription du coffre ?

    Dans le **coffre Recovery Services** , sélectionnez **Gérer** > **Infrastructure Site Recovery** > **Serveurs de configuration** . Dans **Serveurs** , sélectionnez **Télécharger une clé d’inscription** pour télécharger le fichier d’informations d’identification du coffre.
* Puis-je cloner un serveur de configuration existant et l’utiliser pour l’orchestration de la réplication ?

    Non. L’utilisation d’un composant de serveur de configuration cloné n’est pas prise en charge. Le clonage du serveur de traitement de montée en puissance est également un scénario non pris en charge. Le clonage de composants Site Recovery a un impact sur les réplications en cours.

* Puis-je changer l’adresse IP du serveur de configuration ?

    Non. Ne modifiez pas l’adresse IP d’un serveur de configuration. Vérifiez que toutes les adresses IP attribuées au serveur de configuration sont des adresses IP statiques et non des adresses IP DHCP.
* Puis-je configurer le serveur de configuration sur Azure ?

    Il est conseillé de configurer le serveur de configuration sur l’environnement local moyennant une ligne de vue directe avec v-Center, et de réduire les latences de transfert de données. Vous pouvez procéder à des sauvegardes planifiées du serveur de configuration [à des fins de restauration automatique](vmware-azure-manage-configuration-server.md#failback-requirements).

* Puis-je modifier le pilote de cache sur un serveur de configuration ou sur un serveur de traitement avec scale-out ?

    Non, le pilote de cache ne peut pas être modifié une fois la configuration terminée.

Pour plus d’informations sur les serveurs de configuration, consultez [Questions courantes relatives aux serveurs de configuration](vmware-azure-common-questions.md#configuration-server).

## <a name="next-steps"></a>Étapes suivantes

Configurez la récupération d’urgence des [machines virtuelles VMware](vmware-azure-tutorial.md) vers Azure.