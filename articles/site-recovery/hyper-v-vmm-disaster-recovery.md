---
title: Configurer la reprise d’activité après sinistre pour Hyper-V sur un site secondaire à l’aide d’Azure Site Recovery
description: Découvrez comment configurer la récupération d’urgence pour les machines virtuelles Hyper-V entre vos sites locaux et Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: how-to
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: b2164f8927e5c3224f8b07c30d057f48fb7bbc32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87495967"
---
# <a name="set-up-disaster-recovery-for-hyper-v-vms-to-a-secondary-on-premises-site"></a>Configurer la récupération d’urgence des machines virtuelles Hyper-V vers un site local secondaire

Le service [Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de récupération d’urgence en gérant et en coordonnant la réplication, le basculement et la restauration automatique des machines locales et des machines virtuelles Azure.

Cet article vous indique comment configurer la récupération d’urgence vers un site secondaire, pour les machines virtuelles Hyper-V locales managées dans des clouds VMM (System Center Virtual Machine Manager). Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Préparer les serveurs VMM sur site et les hôtes Hyper-V
> * Créer un coffre Recovery Services pour Site Recovery 
> * Configurer les environnements de réplication source et cible 
> * Configurer le mappage réseau 
> * Créer une stratégie de réplication
> * Activer la réplication pour une machine virtuelle


## <a name="prerequisites"></a>Prérequis

Pour suivre ce scénario :

- Examinez [l’architecture et les composants du scénario](hyper-v-vmm-architecture.md).
- Assurez-vous que les serveurs VMM et les hôtes Hyper-V sont conformes aux [exigences de prise en charge](hyper-v-vmm-secondary-support-matrix.md).
- Vérifiez que les machines virtuelles à répliquer sont conformes aux [exigences de prise en charge de la machine répliquée](hyper-v-vmm-secondary-support-matrix.md#replicated-vm-support).
- Préparez les serveurs VMM pour le mappage réseau.

### <a name="prepare-for-network-mapping"></a>Préparer le mappage réseau

Le [mappage réseau](hyper-v-vmm-network-mapping.md) effectue un mappage entre les réseaux de machines virtuelles VMM sur site dans les clouds source et cible. Il effectue les opérations suivantes :

- Connecte les machines virtuelles aux réseaux de machines virtuelles cibles appropriés après le basculement. 
- Place de manière optimale les machines virtuelles de réplication sur les serveurs hôtes Hyper-V cibles. 
- Si vous ne configurez pas le mappage réseau, les machines virtuelles réplica ne seront connectées à aucun réseau de machines virtuelles après le basculement.

Préparez les serveurs VMM comme suit :

1. Assurez-vous que vous avez des [réseaux logiques VMM](/system-center/vmm/network-logical) sur les serveurs VMM source et cible.
    - Le réseau logique sur le serveur source doit être associé au cloud source dans lequel se trouvent les hôtes Hyper-V.
    - Le réseau logique sur le serveur cible doit être associé au cloud cible.
1. Assurez-vous que vous avez des [réseaux de machines virtuelles](/system-center/vmm/network-virtual) sur les serveurs VMM source et cible. Les réseaux de machines virtuelles doivent être associés au réseau logique dans chaque emplacement.
2. Connectez les machines virtuelles sur les hôtes Hyper-V sources au réseau de machines virtuelles source. 


## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]


## <a name="choose-a-protection-goal"></a>Choisir un objectif en matière de protection

Sélectionnez les éléments à répliquer et l’emplacement de la réplication.

1. Cliquez sur **Site Recovery** > **Étape 1 : préparer l’infrastructure** > **Objectif de protection**.
2. Sélectionnez **Vers le site de récupération**, puis **Oui, avec Hyper-V**.
3. Sélectionnez **Oui** pour indiquer que vous utilisez VMM pour gérer les hôtes Hyper-V.
4. Sélectionnez **Oui** si vous avez un serveur VMM secondaire. Si vous déployez la réplication entre des clouds sur un seul serveur VMM, cliquez sur **Non**. Cliquez ensuite sur **OK**.


## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

Installez le fournisseur Azure Site Recovery sur les serveurs VMM puis découvrez et inscrivez ces derniers dans le coffre.

1. Cliquez sur **Préparer l’infrastructure** > **Source**.
2. Dans **Préparer la source**, cliquez sur **+ VMM** pour ajouter un serveur VMM.
3. Dans **Ajouter un serveur**, vérifiez que **Serveur System Center VMM** s’affiche dans **Type de serveur**.
4. Téléchargez le fichier d’installation du fournisseur Azure Site Recovery.
5. Téléchargez la clé d’inscription. Cette clé est nécessaire lorsque vous installez le fournisseur. Une fois générée, la clé reste valide pendant 5 jours.

    ![Capture d’écran des options permettant de télécharger le fournisseur et la clé d’inscription.](./media/hyper-v-vmm-disaster-recovery/source-settings.png)

6. Installez le fournisseur sur chaque serveur VMM. Vous n’avez besoin de rien installer explicitement sur les hôtes Hyper-V.

### <a name="install-the-azure-site-recovery-provider"></a>installation du fournisseur Azure Site Recovery

1. Exécutez le fichier d’installation du fournisseur sur chaque serveur VMM. Si VMM est déployé dans un cluster, procédez comme suit lors de la première installation :
    -  Installez le fournisseur sur un nœud actif et terminez l’installation pour inscrire ce serveur VMM dans le coffre.
    - Ensuite, installez le fournisseur sur les autres nœuds. Les nœuds de cluster doivent exécuter la même version du fournisseur.
2. Le programme d’installation exécute quelques vérifications préalables et demande l’autorisation d’arrêter le service VMM. Le service VMM redémarre automatiquement une fois l’installation terminée. Si vous installez sur un cluster VMM, vous êtes invité à arrêter le rôle de cluster.
3. Dans **Microsoft Update**, vous pouvez choisir des mises à jour, pour que celles du fournisseur soient installées conformément à votre stratégie Microsoft Update.
4. Dans le champ **Installation**, acceptez ou modifiez l’emplacement d’installation par défaut et cliquez sur **Installer**.
5. Une fois l’installation terminée, cliquez sur **Inscrire** pour inscrire le serveur dans le coffre.

    ![Capture d’écran de l’écran d’installation du fournisseur, y compris l’emplacement de l’installation.](./media/hyper-v-vmm-disaster-recovery/provider-register.png)
6. Dans **Vault name**, vérifiez le nom du coffre dans lequel le serveur est enregistré. Cliquez sur **Suivant**.
7. Sur la page **Connexion proxy**, indiquez la façon dont le fournisseur exécuté sur le serveur VMM se connecte à Azure.
   - Vous pouvez définir la connexion directe, ou via un proxy, du fournisseur à Internet. Spécifiez les paramètres de proxy, si nécessaire.
   - Si vous utilisez un proxy, un compte RunAs VMM (DRAProxyAccount) est créé automatiquement avec les informations d’identification du proxy spécifiées. Configurez le serveur proxy pour que ce compte puisse s'authentifier correctement. Vous pouvez modifier les paramètres du compte RunAs dans la console VMM (**Paramètres** > **Sécurité** > **Comptes d’identification**).
   - Redémarrez le service VMM afin de mettre à jour les modifications.
8. Dans **Clé d’inscription**, sélectionnez la clé que vous avez téléchargée et copiez-la sur le serveur VMM.
9. Le paramètre de chiffrement ne s’applique pas à ce scénario. 
10. Dans **Server name**, entrez un nom convivial pour identifier le serveur VMM dans le coffre. Dans un cluster, spécifiez le nom de rôle de cluster VMM.
11. Dans **Synchroniser les métadonnées du cloud** , indiquez si vous voulez synchroniser les métadonnées de tous les clouds sur le serveur VMM. Cette action se produit une seule fois sur chaque serveur. Si vous ne souhaitez pas synchroniser tous les clouds, laisser cette option désactivée. Vous pouvez synchroniser chaque cloud individuellement dans les propriétés du cloud de la console VMM.
12. Cliquez sur **Suivant** pour terminer le processus. Une fois l'inscription terminée, Azure Site Recovery extrait les métadonnées du serveur VMM. Le serveur s’affiche dans **Serveurs** > **Serveurs VMM** dans le coffre.
13. Une fois le serveur présent dans le coffre, dans **Source** > **Préparer la source**, sélectionnez le serveur VMM, puis sélectionnez le cloud dans lequel se trouve l’hôte Hyper-V. Cliquez ensuite sur **OK**.


## <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

Sélectionnez le cloud et le serveur VMM cible :

1. Cliquez sur **Préparer l’infrastructure** > **Cible** et sélectionnez le serveur VMM cible.
2. Les clouds VMM synchronisés avec Site Recovery s’affichent. Sélectionnez le cloud cible.

   ![Capture d’écran du serveur VMM cible et des sélections du cloud.](./media/hyper-v-vmm-disaster-recovery/target-vmm.png)


## <a name="set-up-a-replication-policy"></a>Configurer une stratégie de réplication

Avant de commencer, assurez-vous que tous les hôtes qui utilisent la stratégie possèdent le même système d’exploitation. Si les hôtes exécutent différentes versions de Windows Server, vous avez besoin de plusieurs stratégies de réplication.

1. Pour créer une stratégie de réplication, cliquez sur **Préparer l’infrastructure** > **Paramètres de réplication** >  **+Créer et associer**.
2. Dans **Créer et associer une stratégie**, indiquez le nom de la stratégie. Le type source et cible doit être **Hyper-V**.
3. Dans **Version de l’hôte Hyper-V**, sélectionnez le système d’exploitation sous lequel fonctionne l’hôte.
4. Dans **Type d’authentification** et **Port d’authentification**, indiquez le mode d’authentification du trafic entre les serveurs hôtes Hyper-V principaux et de récupération.
    - Sélectionnez **Certificat** , sauf si vous avez un environnement Kerberos opérationnel. Azure Site Recovery configurera automatiquement des certificats pour l'authentification HTTPS. Vous n’avez pas besoin de faire quoi que ce soit manuellement.
    - Par défaut, les ports 8083 et 8084 (pour les certificats) seront ouverts dans le Pare-feu Windows sur les serveurs hôtes Hyper-V.
    - Si vous sélectionnez **Kerberos**, un ticket Kerberos est utilisé pour l'authentification mutuelle des serveurs hôtes. Kerberos ne s’applique que pour les serveurs hôtes Hyper-V s'exécutant sur Windows Server 2012 R2 ou version ultérieure.
1. Dans **Fréquence de copie**, spécifiez la fréquence selon laquelle répliquer les données delta après la réplication initiale (toutes les 30 secondes ou toutes les 5 ou 15 minutes).
2. Dans **Rétention des points de récupération**, spécifiez la durée (en heures) de la fenêtre de rétention pour chaque point de récupération. Les machines répliquées peuvent être récupérées à tout moment pendant cette fenêtre temporelle.
3. Dans **Fréquence des captures instantanées cohérentes de l’application**, spécifiez la fréquence de création des points de récupération contenant des instantanés cohérents au niveau des applications (entre 1 et 12 heures). Hyper-V utilise deux types de captures instantanées :
    - **Instantané standard** : fournit un instantané incrémentiel de toute la machine virtuelle.
    - **Instantané de cohérence des applications** : prend un instantané à un point dans le temps des données d’application à l’intérieur de la machine virtuelle. Le service VSS (Volume Shadow Copy Service) s'assure que les applications sont dans un état cohérent au moment de la prise des captures instantanées. L’activation d’instantanés cohérents au niveau applicatif affecte la performance de l’application sur les machines virtuelles sources. Définissez une valeur inférieure au nombre de points de récupération supplémentaires que vous configurez.
4. Dans **Compression du transfert de données**, indiquez si les données de réplication transférées doivent être compressées.
5. Sélectionnez **Supprimer une machine virtuelle de réplica** pour spécifier que la machine virtuelle doit être supprimée si vous désactivez la protection de la machine virtuelle source. Si vous activez ce paramètre et que vous désactivez la protection de la machine virtuelle source, elle est supprimée de la console Site Recovery, les paramètres Site Recovery de VMM sont supprimés de la console VMM et le réplica est supprimé.
6. Dans **Méthode de réplication initiale**, si vous effectuez une réplication sur le réseau, indiquez si vous souhaitez lancer la réplication initiale ou la planifier. Pour économiser de la bande passante réseau, il peut être intéressant de la planifier en dehors des heures de pointe. Cliquez ensuite sur **OK**.

     ![Capture d’écran des options de stratégie de réplication.](./media/hyper-v-vmm-disaster-recovery/replication-policy.png)
     
7. La nouvelle stratégie est automatiquement associée au cloud VMM. Dans **Stratégie de réplication**, cliquez sur **OK**. 


## <a name="enable-replication"></a>Activer la réplication

1. Cliquez sur **Répliquer l’application** > **Source**. 
2. Dans **Source**, sélectionnez le serveur VMM et le cloud hébergeant les hôtes Hyper-V à répliquer. Cliquez ensuite sur **OK**.
3. Dans **Cible**, vérifiez le serveur VMM secondaire et le cloud.
4. Dans **Machines virtuelles**, sélectionnez les machines virtuelles que vous souhaitez protéger dans la liste.


Vous pouvez suivre la progression de l’action **Activer la protection** dans **Travaux** > **Travaux Site Recovery**. Lorsque le travail de **finalisation de la protection** est terminé, la réplication initiale est également terminée et la machine virtuelle est prête à être basculée.

## <a name="next-steps"></a>Étapes suivantes

[Effectuer un test de récupération d’urgence](hyper-v-vmm-test-failover.md)
