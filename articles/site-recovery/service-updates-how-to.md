---
title: Mises à jour et mises à niveau de composants dans Azure Site Recovery
description: Fournit une vue d’ensemble des mises à jour du service Azure Site Recovery et des mises à niveau des composants.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/31/2019
ms.author: ramamill
ms.openlocfilehash: a1ea8b6fb9800d796670161288be0d86ce6ffc42
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89424937"
---
# <a name="service-updates-in-site-recovery"></a>Mises à jour de service dans Site Recovery

Cet article fournit une vue d'ensemble des mises à jour de [Azure Site Recovery](site-recovery-overview.md) et décrit comment mettre à niveau des composants Site Recovery.

Site Recovery publie régulièrement les mises à jour du service. Les mises à jour incluent les nouvelles fonctionnalités, les améliorations de la prise en charge, les mises à jour des composants et les correctifs de bogues. Pour tirer parti des dernières fonctionnalités et des derniers correctifs, nous vous recommandons d’exécuter les dernières versions de Site Recovery Components. 
 
 
## <a name="updates-support"></a>Prise en charge des mises à jour

### <a name="support-statement-for-azure-site-recovery"></a>Déclaration de support pour Azure Site Recovery

Nous vous recommandons de toujours procéder à une mise à niveau vers les versions les plus récentes des composants :

**À chaque publication d’une nouvelle version « N » d’un composant Azure Site Recovery, toutes les versions inférieures à « N-4 » sont considérées comme non couvertes par le support**. 

> [!IMPORTANT]
> La prise en charge officielle concerne la mise à niveau de > N-4 vers la version N. Par exemple, si vous exécutez sur N-6, vous devez d’abord effectuer une mise à niveau vers N-4, puis effectuer une mise à niveau vers N.


### <a name="links-to-currently-supported-update-rollups"></a>Liens vers les correctifs cumulatifs actuellement pris en charge

 Consultez le correctif cumulatif le plus récent (version N ) dans [cet article](site-recovery-whats-new.md). N’oubliez pas que Site Recovery fournit une prise en charge pour les versions N-4.



## <a name="component-expiry"></a>Expiration du composant

Site Recovery vous avertit des composants expirés (ou arrivant à expiration) par courrier électronique (si vous vous êtes abonné à des notifications par courrier électronique) ou dans le tableau de bord du coffre dans le portail.

- En outre, lorsque des mises à jour sont disponibles, dans la vue d’infrastructure de votre scénario dans le portail , un bouton **Mettre à jour les éléments disponibles** apparaît en regard du composant. Ce bouton vous redirige vers un lien pour télécharger la dernière version du composant.
-  Les notifications de tableau de bord de coffres ne sont pas disponibles si vous répliquez des machines virtuelles Hyper-V. 

Les notifications par courrier électronique sont envoyées comme suit.

**Time** | **Fréquence**
--- | ---
60 jours avant l’expiration du composant | Deux fois par semaine
53 prochains jours | Une fois par semaine
7 derniers jours | 1 par jour
Après expiration | Deux fois par semaine


### <a name="upgrading-outside-official-support"></a>Mise à niveau en dehors du support officiel

Si la différence entre la version de votre composant et la version la plus récente est supérieure à quatre, alors le support est considéré comme non pris en charge. Dans ce cas, procédez à la mise à niveau comme suit : 

1. Mettez à niveau le composant actuellement installé vers votre version actuelle plus quatre. Par exemple, si votre version est 9.16, effectuez la mise à niveau vers 9.20.
2. Ensuite, effectuez une mise à niveau vers la version compatible suivante. Ainsi, dans notre exemple, après la mise à niveau de 9.16 vers 9.20, effectuez la mise à niveau vers 9.24. 

Suivez le même processus pour tous les composants concernés.

### <a name="support-for-latest-operating-systemskernels"></a>Prise en charge des derniers systèmes d’exploitation/noyaux

> [!NOTE]
> Si vous avez une fenêtre de maintenance planifiée et qu'un redémarrage est inclus, nous vous recommandons de commencer par mettre à niveau les composants de Site Recovery, puis de poursuivre le reste des activités planifiées dans la fenêtre de maintenance.

1. Avant de mettre à niveau les versions du système d’exploitation/noyau, vérifiez si la version cible est prise en charge Site Recovery. 

    - Support des [machines virtuelles Azure](azure-to-azure-support-matrix.md#replicated-machine-operating-systems).
    - Support [VMware/serveur physique](vmware-physical-azure-support-matrix.md#replicated-machines)
    - Support [Hyper-V](hyper-v-azure-support-matrix.md#replicated-vms).
2. Passez en revue les [mises à jour disponibles](site-recovery-whats-new.md) pour découvrir ce que vous souhaitez mettre à niveau.
3. Effectuez une mise à niveau vers la dernière version de Site Recovery.
4. Mettez à niveau le système d’exploitation/noyau vers les versions requises.
5. Redémarrage.


Ce processus garantit que le système d’exploitation/noyau de l’ordinateur est mis à niveau vers la dernière version, et que les dernières modifications de Site Recovery nécessaires pour prendre en charge la nouvelle version sont chargées sur l’ordinateur.

## <a name="azure-vm-disaster-recovery-to-azure"></a>Récupération d’urgence de machine virtuelle Azure vers Azure

Dans ce scénario, nous vous recommandons vivement d’[activer les mises à jour automatiques](azure-to-azure-autoupdate.md). Vous pouvez autoriser Site Recovery à gérer les mises à jour comme suit :

- Pendant le processus d’activation de la réplication.
- En définissant les paramètres de mise à jour de l’extension dans le coffre.

Si vous souhaitez gérer manuellement les mises à jour, vous pouvez choisir l’une des options suivantes :

1. Quand une nouvelle mise à jour de l’agent est disponible, Site Recovery fournit une notification dans le coffre vers le haut de la page. Dans le coffre > **Éléments répliqués**, cliquez sur cette notification en haut de l'écran : 
    
    **Une nouvelle mise à jour de l’agent de réplication Site Recovery est disponible. Cliquez pour installer ->** <br/><br/>Sélectionnez les machines virtuelles auxquelles vous souhaitez appliquer la mise à jour, puis cliquez sur **OK**.

2. Sur la page vue d’ensemble de la récupération d’urgence de la machine virtuelle, vous trouverez le champ « État de l’agent » qui indique « Mise à niveau critique » si l’agent est en attente d’expiration. Cliquez dessus et suivez les instructions suivantes pour mettre à niveau manuellement la machine virtuelle.

## <a name="vmware-vmphysical-server-disaster-recovery-to-azure"></a>Récupération d’urgence des machines virtuelles VMware/serveurs physiques sur Azure

1. En fonction de votre version actuelle et de la [déclaration de support](#support-statement-for-azure-site-recovery), installez d’abord la mise à jour sur le serveur de configuration local, à l’aide de [ces instructions](vmware-azure-deploy-configuration-server.md#upgrade-the-configuration-server). 
2. Si vous avez des serveurs de processus scale-out, mettez-les à jour à l’aide de [ces instructions](vmware-azure-manage-process-server.md#upgrade-a-process-server).
3. Pour mettre à jour l’agent Mobility Service sur chaque machine protégée, reportez-vous à [cet](vmware-physical-manage-mobility-service.md#update-mobility-service-from-azure-portal) article.

### <a name="reboot-after-mobility-service-upgrade"></a>Redémarrage après la mise à niveau du service mobilité

Un redémarrage est recommandé après chaque mise à niveau du service mobilité, afin de garantir que toutes les dernières modifications sont chargées sur la machine source.

Un redémarrage n’est pas obligatoire, sauf si la différence entre la version de l’agent au cours du dernier redémarrage et la version actuelle est supérieure à quatre.

L’exemple du tableau montre comment cela fonctionne.

|**Version de l’agent ( dernier redémarrage)** | **Mise à niveau vers** | **Redémarrage obligatoire ?**|
|---------|---------|---------|
|9.16 |  9.18 | Facultatif|
|9.16 | 9.19 | Facultatif|
| 9.16 | 9.20 | Facultatif
 | 9.16 | 9.21 | Mandatory.<br/><br/> Effectuez une mise à niveau vers 9.20, puis redémarrez avant de procéder à la mise à niveau vers 9.21.

## <a name="hyper-v-vm-disaster-recovery-to-azure"></a>Récupération d’urgence de machine virtuelle Hyper-V vers Azure

### <a name="between-a-hyper-v-site-and-azure"></a>Entre un site Hyper-V et Azure

1. Téléchargez la mise à jour du fournisseur Microsoft Azure Site Recovery.
2. Installez le fournisseur sur chaque serveur Hyper-V inscrit dans Site Recovery. Si vous exécutez un cluster, effectuez une mise à niveau sur tous les nœuds du cluster.


## <a name="between-an-on-premises-vmm-site-and-azure"></a>Entre un site VMM local et Azure
1. Téléchargez la mise à jour du fournisseur Microsoft Azure Site Recovery.
2. Installer le fournisseur sur le serveur VMM. Si VMM est déployé dans un cluster, installez le fournisseur sur tous les nœuds du cluster.
3. Installez le dernier agent de Microsoft Azure Recovery Services sur tous les hôtes Hyper-V ou nœuds de cluster.


## <a name="between-two-on-premises-vmm-sites"></a>Entre deux sites VMM locaux
1. Téléchargez la dernière mise à jour du fournisseur Microsoft Azure Site Recovery.
2. Installez le dernier fournisseur sur le serveur VMM qui gère le site de récupération secondaire. Si VMM est déployé dans un cluster, installez le fournisseur sur tous les nœuds du cluster.
3. Une fois le site de récupération mis à jour, installez le fournisseur sur le serveur VMM qui gère le site principal.

## <a name="next-steps"></a>Étapes suivantes

Suivez notre page des [mises à jour Azure](https://azure.microsoft.com/updates/?product=site-recovery) pour suivre les nouvelles mises à jour et mises en production.
