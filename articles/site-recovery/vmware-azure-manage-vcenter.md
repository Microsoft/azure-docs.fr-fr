---
title: Gérer des serveurs VMware vCenter dans Azure Site Recovery
description: Cet article décrit comment ajouter et gérer VMware vCenter pour la récupération d’urgence de machines virtuelles VMware sur Azure avec Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 12/24/2019
ms.author: ramamill
ms.openlocfilehash: 01aef3aca4f6967b1681bff9598c7dd7a24739cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84692517"
---
# <a name="manage-vmware-vcenter-server"></a>Gérer un serveur VMware vCenter

Cet article résume les actions de gestion sur un serveur VMware vCenter dans [Azure Site Recovery](site-recovery-overview.md).

## <a name="verify-prerequisites-for-vcenter-server"></a>Vérifier les prérequis pour le serveur vCenter

Les prérequis pour les serveurs vCenter et les machines virtuelles lors de la reprise d’activité des machines virtuelles VMware sur Azure sont listés dans le [tableau de prise en charge](vmware-physical-azure-support-matrix.md#replicated-machines).

## <a name="set-up-an-account-for-automatic-discovery"></a>Configurer un compte pour la détection automatique

Quand vous configurez la récupération d’urgence pour des machines virtuelles VMware locales, Site Recovery doit accéder à au serveur vCenter Server ou à l’hôte vSphere. Le serveur de processus Site Recovery peut ensuite découvrir automatiquement les machines virtuelles et les basculer en fonction des besoins. Par défaut, le serveur de processus s’exécute sur le serveur de configuration Site Recovery. Ajoutez un compte pour le serveur de configuration afin de vous connecter au serveur vCenter ou à l’hôte vSphere comme suit :

1. Connectez-vous au serveur de configuration.
1. Ouvrez l’outil de serveur de configuration (_cspsconfigtool.exe_) à l’aide du raccourci sur le Bureau.
1. Sous l’onglet **Gérer le compte**, cliquez sur **Ajouter un compte**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Fournissez les détails du compte, puis cliquez sur **OK** pour l’ajouter. Le compte doit disposer des privilèges répertoriés dans le tableau d’autorisations suivant.

   > [!NOTE]
   > La synchronisation des informations de compte avec Site Recovery prend environ 15 minutes.

### <a name="account-permissions"></a>Autorisations de compte

|**Tâche** | **Compte** | **Autorisations** | **Détails**|
|--- | --- | --- | ---|
|**Détection/migration des machines virtuelles (sans restauration automatique)** | Au moins un compte d’utilisateur en lecture seule. | Objet de centre de données -> Propager vers l’objet enfant, rôle = lecture seule | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec l’objet **Propager vers enfant** aux objets enfants (hôtes vSphere, banques de données, machines virtuelles et réseaux).|
|**Réplication/basculement** | Au moins un compte d’utilisateur en lecture seule. | Objet de centre de données -> Propager vers l’objet enfant, rôle = lecture seule | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec l’objet **Propager vers enfant** aux objets enfants (hôtes vSphere, banques de données, machines virtuelles et réseaux).<br/><br/> Utile à des fins de migration, mais pas pour la réplication complète, le basculement et la restauration automatique.|
|**Réplication/basculement/restauration automatique** | Nous vous suggérons de créer un rôle (AzureSiteRecoveryRole) avec les autorisations nécessaires, puis d’attribuer le rôle à un utilisateur ou à un groupe d’utilisateurs VMware. | Objet de centre de données -> Propager vers l’objet enfant, rôle = AzureSiteRecoveryRole<br/><br/> Banque de données -> Allouer de l’espace, parcourir la banque de données, opérations de fichier de bas niveau, supprimer le fichier, mettre à jour les fichiers de machine virtuelle<br/><br/> Réseau -> Attribution de réseau<br/><br/> Ressource -> Affecter les machines virtuelles au pool de ressources, migrer des machines virtuelles hors tension, migrer des machines virtuelles sous tension<br/><br/> Tâches -> Créer une tâche, Mettre à jour une tâche<br/><br/> Machine virtuelle -> Configuration<br/><br/> Machine virtuelle -> Interagir -> répondre à la question, connexion d’appareil, configurer un support de CD, configurer une disquette, mettre hors tension, mettre sous tension, installation des outils VMware<br/><br/> Machine virtuelle -> Stock -> Créer, inscrire, désinscrire<br/><br/> Machine virtuelle -> Approvisionnement -> Autoriser le téléchargement de machines virtuelles, autoriser le chargement de fichiers de machine virtuelle<br/><br/> Machine virtuelle -> Instantanés -> Supprimer les instantanés | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec l’objet **Propager vers enfant** aux objets enfants (hôtes vSphere, banques de données, machines virtuelles et réseaux).|

## <a name="add-vmware-server-to-the-vault"></a>Ajouter un serveur VMware dans le coffre

Quand vous configurez la reprise d’activité pour des machines virtuelles VMware locales, vous ajoutez le serveur vCenter/l’hôte vSphere sur lequel vous détectez des machines virtuelles dans le coffre Site Recovery, comme suit :

1. Dans le coffre > **Infrastructure Site Recovery** > **Serveurs de configuration**, ouvrez le serveur de configuration.
1. Dans la page **Détails**, cliquez sur **vCenter**.
1. Dans **Ajouter un serveur vCenter**, spécifiez un nom convivial pour l’hôte vSphere ou le serveur vCenter.
1. Spécifiez l’adresse IP ou le nom de domaine complet du serveur.
1. Conservez le port 443, sauf si vos serveurs VMware sont configurés pour écouter les demandes sur un port différent.
1. Sélectionnez le compte utilisé pour se connecter au serveur VMware vCenter ou vSphere ESXi. Cliquez ensuite sur **OK**.

## <a name="modify-credentials"></a>Modifier les informations d’identification

Si nécessaire, vous pouvez modifier les informations d’identification utilisées pour se connecter au serveur vCenter ou à l’hôte vSphere comme suit :

1. Connectez-vous au serveur de configuration.
1. Ouvrez l’outil de serveur de configuration (_cspsconfigtool.exe_) à l’aide du raccourci sur le Bureau.
1. Cliquez sur **Ajouter un compte** dans l’onglet **Gérer les comptes**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Fournissez les détails du nouveau compte, puis cliquez sur **OK**. Le compte a besoin des autorisations répertoriées dans le tableau d’[autorisations du compte](#account-permissions).
1. Dans le coffre > **Infrastructure Site Recovery** > **Serveurs de configuration**, ouvrez le serveur de configuration.
1. Dans **Détails**, cliquez sur **Actualiser le serveur**.
1. Une fois la tâche d’actualisation du serveur terminée, sélectionnez le serveur vCenter.
1. Dans **Résumé**, sélectionnez le compte qui vient d’être ajouté dans le **compte du serveur vCenter ou de l’hôte vSphere**, puis cliquez sur **Enregistrer**.

   ![modify-account](./media/vmware-azure-manage-vcenter/modify-vcente-creds.png)

## <a name="delete-a-vcenter-server"></a>Supprimer un serveur vCenter

1. Dans le coffre > **Infrastructure Site Recovery** > **Serveurs de configuration**, ouvrez le serveur de configuration.
1. Sur la page **Détails**, sélectionnez le serveur vCenter.
1. Cliquez sur le bouton **Supprimer**.

   ![delete-account](./media/vmware-azure-manage-vcenter/delete-vcenter.png)

## <a name="modify-the-ip-address-and-port"></a>Modifier l’adresse IP et le port

Vous pouvez modifier l’adresse IP du serveur vCenter ou les ports utilisés pour la communication entre le serveur et Site Recovery. Par défaut, Site Recovery accède aux informations du serveur vCenter ou de l’hôte vSphere via le port 443.

1. Dans le coffre > **Infrastructure Site Recovery** > **Serveurs de configuration**, cliquez sur le serveur de configurations auquel le serveur vCenter est ajouté.
1. Dans **Serveurs vCenter**, cliquez sur le serveur vCenter que vous souhaitez modifier.
1. Dans **Résumé**, mettez à jour l’adresse IP et le port, puis enregistrez les modifications.

   ![add_ip_new_vcenter](media/vmware-azure-manage-vcenter/add-ip.png)

1. Afin que les modifications soient prises en compte, attendez 15 minutes ou [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="migrate-all-vms-to-a-new-server"></a>Migrer toutes les machines virtuelles vers un nouveau serveur

Si vous souhaitez migrer toutes les machines virtuelles afin d’utiliser un nouveau serveur vCenter, il vous suffit de mettre à jour l’adresse IP affectée au serveur vCenter. N’ajoutez pas d’autre compte VMware, car cela peut entraîner des entrées en double. Mettez à jour l’adresse comme suit :

1. Dans le coffre > **Infrastructure Site Recovery** > **Serveurs de configuration**, cliquez sur le serveur de configurations auquel le serveur vCenter est ajouté.
1. Dans la section **Serveurs vCenter**, cliquez sur le serveur vCenter à partir duquel vous prévoyez de procéder à la migration.
1. Dans **Résumé**, mettez à jour l’adresse IP vers celle du nouveau serveur vCenter, puis enregistrez les modifications.
1. Aussitôt l’adresse IP mise à jour, Site Recovery commence à recevoir les informations de découverte de machines virtuelles du nouveau serveur vCenter. Les activités de réplication en cours ne s’en trouvent pas affectées.

## <a name="migrate-a-few-vms-to-a-new-server"></a>Migrer quelques machines virtuelles vers un nouveau serveur

Si vous souhaitez uniquement migrer certaines de vos machines virtuelles de réplication vers un nouveau serveur vCenter, procédez comme suit :

1. [Ajoutez](#add-vmware-server-to-the-vault) le nouveau serveur vCenter au serveur de configuration.
1. [Désactivez la réplication](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) pour les machines virtuelles qui seront déplacées vers le nouveau serveur.
1. Dans VMware, migrez les machines virtuelles vers le nouveau serveur vCenter.
1. [Activez de nouveau la réplication](vmware-azure-tutorial.md#enable-replication) pour les machines virtuelles migrées, en sélectionnant le nouveau serveur vCenter.

## <a name="migrate-most-vms-to-a-new-server"></a>Migrer la plupart des machines virtuelles vers un nouveau serveur

Si le nombre de machines virtuelles que vous souhaitez migrer vers un nouveau serveur vCenter est supérieur au nombre de machines virtuelles qui seront conservées sur le serveur vCenter d’origine, procédez comme suit :

1. [Mettez à jour l’adresse IP](#modify-the-ip-address-and-port) affectée au serveur vCenter dans les paramètres du serveur de configuration vers l’adresse du nouveau serveur vCenter.
1. [Désactivez la réplication](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) pour les machines virtuelles qui restent sur l’ancien serveur.
1. [Ajoutez l’ancien serveur vCenter](#add-vmware-server-to-the-vault) et son adresse IP au serveur de configuration.
1. [Réactivez la réplication](vmware-azure-tutorial.md#enable-replication) pour les machines virtuelles qui restent sur l’ancien serveur.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes, voir [Résoudre les problèmes d’échec de découverte de vCenter Server](vmware-azure-troubleshoot-vcenter-discovery-failures.md).
