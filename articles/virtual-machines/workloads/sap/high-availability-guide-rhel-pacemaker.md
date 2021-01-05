---
title: Configuration de Pacemaker sur RHEL dans Azure | Microsoft Docs
description: Configuration de Pacemaker sur Red Hat Entreprise Linux dans Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 12/01/2020
ms.author: radeltch
ms.openlocfilehash: b111dae035e7a055628642fe7c460734199ff608
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486340"
---
# <a name="setting-up-pacemaker-on-red-hat-enterprise-linux-in-azure"></a>Configuration de Pacemaker sur Red Hat Entreprise Linux dans Azure

[planning-guide]:planning-guide.md
[deployment-guide]:deployment-guide.md
[dbms-guide]:dbms-guide.md
[sap-hana-ha]:sap-hana-high-availability.md
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351

[virtual-machines-linux-maintenance]:../../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot


Commencez par lire les notes et publications SAP suivantes :

* Note SAP [1928533], qui contient :
  * La liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP.
  * Des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure.
  * Les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données.
  * La version du noyau SAP requise pour Windows et Linux sur Microsoft Azure.
* La note SAP [2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2002167] recommande des paramètres de système d’exploitation pour Red Hat Enterprise Linux
* La note SAP [2009879] conseille sur SAP HANA pour Red Hat Enterprise Linux
* La note SAP [2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1999351] contient des informations de dépannage supplémentaires pour l’extension d’analyse Azure améliorée pour SAP.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* [Planification et implémentation de machines virtuelles Azure pour SAP sur Linux][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP sur Linux (cet article)][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP sur Linux][dbms-guide]
* [Réplication de système SAP HANA dans un cluster Pacemaker](https://access.redhat.com/articles/3004101)
* Documentation RHEL générale
  * [Vue d’ensemble des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administration des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Référence des modules complémentaires de haute disponibilité](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Stratégies de prise en charge pour les clusters haute disponibilité RHEL – sbd et fence_sbd](https://access.redhat.com/articles/2800691)
* Documentation RHEL spécifique à Azure :
  * [Stratégies de prise en charge des clusters à haute disponibilité RHEL - Machines virtuelles Microsoft Azure en tant que membres du cluster](https://access.redhat.com/articles/3131341)
  * [Installation et configuration d’un cluster à haute disponibilité Red Hat Enterprise Linux 7.4 (et versions ultérieures) sur Microsoft Azure](https://access.redhat.com/articles/3252491)
  * [Considérations relatives à l’adoption de RHEL 8 - Haute disponibilité et clusters](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/considerations_in_adopting_rhel_8/high-availability-and-clusters_considerations-in-adopting-rhel-8)
  * [Configure SAP S/4HANA ASCS/ERS with Standalone Enqueue Server 2 (ENSA2) in Pacemaker on RHEL 7.6](https://access.redhat.com/articles/3974941) (Configurer SAP S/4HANA ASC/ERS avec le serveur de file d’attente autonome 2 (ENSA2) dans Pacemaker sur RHEL 7.6)
  * [Offres RHEL pour SAP sur Azure](https://access.redhat.com/articles/5456301)

## <a name="cluster-installation"></a>Installation du cluster

![Présentation de Pacemaker sur RHEL](./media/high-availability-guide-rhel-pacemaker/pacemaker-rhel.png)

> [!NOTE]
> Red Hat ne prend pas en charge la surveillance émulée par logiciel. Red Hat ne prend pas en charge SBD sur des plateformes cloud. Pour plus d’informations, voir [Stratégies de prise en charge pour les clusters haute disponibilité RHEL – sbd et fence_sbd](https://access.redhat.com/articles/2800691).
> Le seul mécanisme de délimitation pris en charge pour des clusters Pacemaker Red Hat Enterprise Linux sur Azure est l’agent de délimitation Azure.  

Les éléments suivants sont précédés de **[A]** (applicable à tous les nœuds), de **[1]** (applicable uniquement au nœud 1) ou de **[2]** (applicable uniquement au nœud 2).

1. **[A]** Inscrire. Cette étape n’est pas requise si vous utilisez des images RHEL SAP compatibles avec la haute disponibilité.  

   Inscrivez vos machines virtuelles et les attacher à un pool qui contient des référentiels pour RHEL 7.

   <pre><code>sudo subscription-manager register
   # List the available pools
   sudo subscription-manager list --available --matches '*SAP*'
   sudo subscription-manager attach --pool=&lt;pool id&gt;
   </code></pre>

   En attachant un pool à une image PAYG RHEL de la Place de marché Azure, vous serez doublement facturé pour votre utilisation de RHEL : une fois pour l’image PAYG et une fois pour le droit d’utilisation de RHEL dans le pool que vous attachez. Pour limiter ceci, Azure fournit désormais des images RHEL BYOS. Des informations supplémentaires sont disponibles [ici](../redhat/byos.md).  

1. **[A]** Activer RHEL pour les référentiels SAP. Cette étape n’est pas requise si vous utilisez des images RHEL SAP compatibles avec la haute disponibilité.  

   Pour installer les packages requis, activez les référentiels suivants.

   <pre><code>sudo subscription-manager repos --disable "*"
   sudo subscription-manager repos --enable=rhel-7-server-rpms
   sudo subscription-manager repos --enable=rhel-ha-for-rhel-7-server-rpms
   sudo subscription-manager repos --enable=rhel-sap-for-rhel-7-server-rpms
   sudo subscription-manager repos --enable=rhel-ha-for-rhel-7-server-eus-rpms
   </code></pre>

1. **[A]** Installer le module complémentaire RHEL HA

   <pre><code>sudo yum install -y pcs pacemaker fence-agents-azure-arm nmap-ncat
   </code></pre>

   > [!IMPORTANT]
   > Nous vous recommandons d’utiliser les versions suivantes de l’agent de clôture Azure (ou versions ultérieures) pour que les clients bénéficient d’un délai de basculement plus rapide, en cas d’échec d’un arrêt de ressource ou si les nœuds de cluster ne peuvent plus communiquer les uns avec les autres :  
   > RHEL 7.7 et version ultérieure utilisent la dernière version disponible du package fence-agents  
   > RHEL 7.6 : fence-agents-4.2.1-11.el7_6.8  
   > RHEL 7.5 : fence-agents-4.0.11-86.el7_5.8  
   > RHEL 7.4 : fence-agents-4.0.11-66.el7_4.12  
   > Pour plus d’informations, consultez [Azure VM running as a RHEL High Availability cluster member take a very long time to be fenced, or fencing fails / times-out before the VM shuts down](https://access.redhat.com/solutions/3408711).

   Vérifiez la version de l’agent de clôture Azure. Si nécessaire, mettez-le à jour vers une version ultérieure ou égale à celle indiquée ci-dessus.

   <pre><code># Check the version of the Azure Fence Agent
    sudo yum info fence-agents-azure-arm
   </code></pre>

   > [!IMPORTANT]
   > Si vous devez mettre à jour l’agent de clôture Azure et si vous utilisez un rôle personnalisé, mettez à jour ce rôle personnalisé en y ajoutant l’action **powerOff**. Pour plus d’informations, consultez [Créer un rôle personnalisé pour l’agent de clôture](#1-create-a-custom-role-for-the-fence-agent).  

1. **[A]** Configurer la résolution de nom d’hôte

   Vous pouvez utiliser un serveur DNS ou modifier le fichier /etc/hosts sur tous les nœuds. Cet exemple montre comment utiliser le fichier /etc/hosts.
   Remplacez l’adresse IP et le nom d’hôte dans les commandes suivantes.  

   >[!IMPORTANT]
   > Si vous utilisez des noms d’hôte dans la configuration du cluster, il est essentiel de disposer d’une résolution de nom d’hôte fiable. Les communications de cluster échouent, si les noms ne sont pas disponibles et peuvent entraîner des retards de basculement de cluster.
   > L’avantage d’utiliser/etc/hosts réside dans le fait que votre cluster devient indépendant du serveur DNS, ce qui peut aussi être un point de défaillance unique.  

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Insérez les lignes suivantes dans le fichier /etc/hosts. Modifiez l’adresse IP et le nom d’hôte en fonction de votre environnement

   <pre><code># IP address of the first cluster node
   <b>10.0.0.6 prod-cl1-0</b>
   # IP address of the second cluster node
   <b>10.0.0.7 prod-cl1-1</b>
   </code></pre>

1. **[A]** Changer le mot de passe hacluster pour utiliser le même mot de passe

   <pre><code>sudo passwd hacluster
   </code></pre>

1. **[A]** Ajouter des règles de pare-feu pour Pacemaker

   Ajoutez les règles de pare-feu suivantes à toutes les communications de cluster entre les nœuds de cluster.

   <pre><code>sudo firewall-cmd --add-service=high-availability --permanent
   sudo firewall-cmd --add-service=high-availability
   </code></pre>

1. **[A]** Activer les services de cluster de base

   Exécutez les commandes suivantes pour activer le service Pacemaker et le démarrer.

   <pre><code>sudo systemctl start pcsd.service
   sudo systemctl enable pcsd.service
   </code></pre>

1. **[1]** Créer le cluster Pacemaker

   Exécutez les commandes suivantes pour authentifier les nœuds et créer le cluster. Définissez le jeton sur 30000 pour autoriser la maintenance avec préservation de la mémoire. Pour plus d’informations, consultez [cet article pour Linux][virtual-machines-linux-maintenance].  
   
   Si vous créez un cluster sur **RHEL 7.x**, utilisez les commandes suivantes :  
   <pre><code>sudo pcs cluster auth <b>prod-cl1-0</b> <b>prod-cl1-1</b> -u hacluster
   sudo pcs cluster setup --name <b>nw1-azr</b> <b>prod-cl1-0</b> <b>prod-cl1-1</b> --token 30000
   sudo pcs cluster start --all
   </code></pre>

   Si vous créez un cluster sur **RHEL 8.x**, utilisez les commandes suivantes :  
   <pre><code>sudo pcs host auth <b>prod-cl1-0</b> <b>prod-cl1-1</b> -u hacluster
   sudo pcs cluster setup <b>nw1-azr</b> <b>prod-cl1-0</b> <b>prod-cl1-1</b> totem token=30000
   sudo pcs cluster start --all
   </code></pre>

   Surveillez l’état du cluster en exécutant la commande suivante :  
   <pre><code> # Run the following command until the status of both nodes is online
   sudo pcs status
   # Cluster name: nw1-azr
   # WARNING: no stonith devices and stonith-enabled is not false
   # Stack: corosync
   # Current DC: <b>prod-cl1-1</b> (version 1.1.18-11.el7_5.3-2b07d5c5a9) - partition with quorum
   # Last updated: Fri Aug 17 09:18:24 2018
   # Last change: Fri Aug 17 09:17:46 2018 by hacluster via crmd on <b>prod-cl1-1</b>
   #
   # 2 nodes configured
   # 0 resources configured
   #
   # Online: [ <b>prod-cl1-0</b> <b>prod-cl1-1</b> ]
   #
   # No resources
   #
   # Daemon Status:
   #   corosync: active/disabled
   #   pacemaker: active/disabled
   #   pcsd: active/enabled
   </code></pre>

1. **[A]** Définir les votes attendus. 
   
   <pre><code># Check the quorum votes 
    pcs quorum status
    # If the quorum votes are not set to 2, execute the next command
    sudo pcs quorum expected-votes 2
   </code></pre>

   >[!TIP]
   > Si vous créez un cluster à plusieurs nœuds qui est un cluster avec plus de deux nœuds, ne définissez pas la valeur 2 aux votes.    

1. **[1]** Autoriser les actions de clôture simultanées

   <pre><code>sudo pcs property set concurrent-fencing=true
   </code></pre>

## <a name="create-stonith-device"></a>Créer l’appareil STONITH

L’appareil STONITH utilise un principal de service pour l’autorisation sur Microsoft Azure. Pour créer un principal de service, effectuez les étapes suivantes.

1. Accédez à <https://portal.azure.com>
1. Ouvrez le panneau Azure Active Directory  
   Accédez aux propriétés et notez l’ID du répertoire. Il s’agit de **l’ID du locataire**.
1. Cliquez sur Inscriptions d’applications
1. Cliquez sur Nouvelle inscription
1. Entrez un nom, sélectionnez « Comptes dans cet annuaire d’organisation uniquement » 
2. Sélectionnez le type d’application « Web », entrez une URL de connexion (par exemple http:\//localhost), puis cliquez sur Ajouter  
   L’URL de connexion n’est pas utilisée et peut être une URL valide
1. Sélectionnez Certificats et secrets, puis sélectionnez Nouveau secret client
1. Entrez une description pour la nouvelle clé, sélectionnez « N’expire jamais » et cliquez sur Ajouter
1. Définissez la valeur d’un nœud. Cette valeur est utilisée comme **mot de passe** pour le principal de service
1. Sélectionnez Vue d’ensemble. Notez l’ID de l’application. Cet identifiant est utilisé comme nom d’utilisateur (**ID de connexion** dans la procédure ci-dessous) du principal de service

### <a name="1-create-a-custom-role-for-the-fence-agent"></a>**[1]**  Créer un rôle personnalisé pour l’agent d’isolation

Par défaut, le principal de service ne possède pas les autorisations d’accéder à vos ressources Azure. Vous devez accorder au principal du service les autorisations de démarrer et d’arrêter (mettre hors tension) toutes les machines virtuelles du cluster. Si vous n’avez pas encore créé le rôle personnalisé, vous pouvez le créer à l’aide de [PowerShell](../../../role-based-access-control/role-assignments-powershell.md) ou de l’[interface de ligne de commande Azure](../../../role-based-access-control/role-assignments-cli.md).

Utilisez le contenu suivant pour le fichier d’entrée. Vous devez adapter le contenu à vos abonnements, c’est-à-dire remplacer c276fc76-9cd4-44c9-99a7-4fd71546436e et e91d47c4-76f3-4271-a796-21b4ecfe3624 par les ID de vos abonnements. Si vous n’avez qu’un seul abonnement, supprimez la deuxième entrée dans AssignableScopes.

```json
{
    "properties": {
        "roleName": "Linux Fence Agent Role",
        "description": "Allows to power-off and start virtual machines",
        "assignableScopes": [
            "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
            "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"
        ],
        "permissions": [
            {
                "actions": [
                    "Microsoft.Compute/*/read",
                    "Microsoft.Compute/virtualMachines/powerOff/action",
                    "Microsoft.Compute/virtualMachines/start/action"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

### <a name="a-assign-the-custom-role-to-the-service-principal"></a>**[A]** Affecter le rôle personnalisé au principal de service

Affectez au principal de service le rôle personnalisé Linux Fence Agent Role (Rôle d’agent d’isolation Linux) créé dans la section précédente. N’utilisez plus le rôle Propriétaire !

1. Accédez à https://portal.azure.com
1. Ouvrez le panneau Toutes les ressources
1. Sélectionnez la machine virtuelle du premier nœud de cluster.
1. Cliquez sur Contrôle d’accès (IAM)
1. Cliquez sur Ajouter une attribution de rôle.
1. Sélectionnez le rôle Linux Fence Agent Role (Rôle d’agent d’isolation Linux).
1. Entrez le nom de l’application que vous avez créée ci-dessus
1. Cliquez sur Enregistrer.

Répétez les étapes ci-dessus pour le deuxième nœud de cluster.

### <a name="1-create-the-stonith-devices"></a>**[1]** Créer les appareils STONITH

Une fois que vous avez modifié les autorisations pour les machines virtuelles, vous pouvez configurer les appareils STONITH dans le cluster.

<pre><code>
sudo pcs property set stonith-timeout=900
</code></pre>

> [!NOTE]
> L’option ’pcmk_host_map’ est requise dans la commande UNIQUEMENT si les noms d’hôte RHEL et les noms de nœud Azure ne sont PAS identiques. Consultez la section en gras dans les commandes.

Pour RHEL **7.x**, utilisez la commande suivante pour configurer l’appareil de clôture :    
<pre><code>sudo pcs stonith create rsc_st_azure fence_azure_arm login="<b>login ID</b>" passwd="<b>password</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" subscriptionId="<b>subscription id</b>" <b>pcmk_host_map="prod-cl1-0:10.0.0.6;prod-cl1-1:10.0.0.7"</b> \
power_timeout=240 pcmk_reboot_timeout=900 pcmk_monitor_timeout=120 pcmk_monitor_retries=4 pcmk_action_limit=3 \
op monitor interval=3600
</code></pre>

Pour RHEL **8.x**, utilisez la commande suivante pour configurer l’appareil de clôture :  
<pre><code>sudo pcs stonith create rsc_st_azure fence_azure_arm username="<b>login ID</b>" password="<b>password</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" subscriptionId="<b>subscription id</b>" <b>pcmk_host_map="prod-cl1-0:10.0.0.6;prod-cl1-1:10.0.0.7"</b> \
power_timeout=240 pcmk_reboot_timeout=900 pcmk_monitor_timeout=120 pcmk_monitor_retries=4 pcmk_action_limit=3 \
op monitor interval=3600
</code></pre>

> [!IMPORTANT]
> Les opérations de surveillance et de clôture sont désérialisées. Par conséquent, s’il y a une opération de surveillance plus longue et un événement de clôture simultanés, il n’y a aucun délai pour le basculement du cluster en raison de l’opération de surveillance en cours d’exécution.  

### <a name="1-enable-the-use-of-a-stonith-device"></a>**[1]** Activer l’utilisation d’un appareil STONITH

<pre><code>sudo pcs property set stonith-enabled=true
</code></pre>

> [!TIP]
>L’agent d’isolation Azure requiert une connectivité sortante vers les points de terminaison publics comme indiqué, ainsi que des solutions possibles évoquées dans [Connectivité de point de terminaison public pour les machines virtuelles utilisant un équilibreur de charge interne standard](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

## <a name="next-steps"></a>Étapes suivantes

* [Planification et implémentation de machines virtuelles Azure pour SAP][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP][dbms-guide]
* Pour savoir comment établir une haute disponibilité et planifier la récupération d’urgence de SAP HANA sur des machines virtuelles Azure, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha]
