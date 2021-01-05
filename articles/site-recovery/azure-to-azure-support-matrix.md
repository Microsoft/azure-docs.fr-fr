---
title: Prendre en charge la matrice de la récupération d’urgence des machines virtuelles Azure à l’aide d’Azure Site Recovery
description: Résume la prise en charge de la récupération d’urgence des machines virtuelles Azure vers une région secondaire à l’aide d’Azure Site Recovery.
ms.topic: article
ms.date: 11/29/2020
ms.author: raynew
ms.openlocfilehash: 8123dbc18ad2fc162b9b92ae40920417e6b874ce
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763279"
---
# <a name="support-matrix-for-azure-vm-disaster-recovery-between-azure-regions"></a>Prendre en charge la matrice de la récupération d’urgence de machines virtuelles Azure entre les régions Azure

Cet article récapitule la prise en charge et les conditions préalables pour la récupération d’urgence de machines virtuelles Azure d’une région Azure vers une autre, avec le service [Azure Site Recovery](site-recovery-overview.md).


## <a name="deployment-method-support"></a>Prise en charge de la méthode de déploiement

**Déploiement** |  **Support**
--- | ---
**Azure portal** | Pris en charge.
**PowerShell** | Pris en charge. [En savoir plus](azure-to-azure-powershell.md)
**REST API** | Pris en charge.
**INTERFACE DE LIGNE DE COMMANDE** | Non prise en charge pour le moment


## <a name="resource-support"></a>Prise en charge des ressources

**Action de ressource** | **Détails**
--- | ---
**Déplacer des coffre entre plusieurs groupes de ressources** | Non pris en charge
**Déplacer le calcul/le stockage/les ressources réseau entre plusieurs groupes de ressources** | Non pris en charge.<br/><br/> Si vous déplacez une machine virtuelle ou des composants associés tels que le stockage/réseau après la réplication de la machine virtuelle, vous devez désactiver et réactiver la réplication pour la machine virtuelle.
**Répliquer des machines virtuelles Azure d’un abonnement à un autre pour la reprise d’activité** | Pris en charge à l’intérieur du même locataire Azure Active Directory.
**Migrer des machines virtuelles entre des régions dans les clusters géographiques pris en charge (dans un même abonnement et entre plusieurs abonnements)** | Pris en charge à l’intérieur du même locataire Azure Active Directory.
**Migrer des machines virtuelles au sein de la même région** | Non pris en charge.

## <a name="region-support"></a>Prise en charge de la région

Vous pouvez répliquer et restaurer des machines virtuelles entre deux régions appartenant au même cluster géographique. Les clusters géographiques sont définis en gardant à l’esprit la en souveraineté et la latence des données.


**Cluster géographique** | **Régions Azure**
-- | --
America | Canada , Canada Centre, USA Centre Sud, Ouest du USA Centre, USA Est, USA Est 2, USA Ouest, USA Ouest 2, USA Centre, USA Centre Nord
Europe | Royaume-Uni Ouest, Royaume-Uni Sud, Europe Nord, Europe Ouest, Afrique du Sud Ouest, Afrique du Sud Nord, Norvège Est, Norvège Ouest, France Centre, Suisse Nord
Asia | Inde Sud, Inde Centre, Inde Ouest, Asie Sud-Est, Asie Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud
Australie    | Australie Est, Australie Sud-Est, Australie Centre, Australie Centre 2
Azure Government    | US Gov Virginie, US Gov Iowa, US Gov Arizona, US Gov Texas, US DoD Est, US DoD Centre
Allemagne    | Centre de l’Allemagne, Nord-Est de l’Allemagne
Chine | Chine Est, Chine Nord, Chine Nord 2, Chine Est 2
Régions restreintes réservées pour la reprise d’activité après sinistre dans leur pays |Allemagne Nord réservée pour les clients Allemagne Centre-Ouest, Suisse Ouest réservée pour les clients Suisse Nord, France Sud réservée pour les clients France Centre, Émirats arabes unis Centre réservée pour les clients Émirats arabes unis Nord

>[!NOTE]
>
> - Pour **Brésil Sud**, vous pouvez répliquer et basculer vers ces régions : USA Centre Sud, USA Centre-Ouest, USA Est, USA Est 2, USA Ouest, USA Ouest 2 et USA Centre Nord.
> - Brésil Sud fait uniquement office de région source à partir de laquelle les machines virtuelles peuvent répliquer à l’aide de Site Recovery. Elle ne peut pas faire office de région cible et ce, en raison de problèmes de latence dus aux distances géographiques. Notez que si vous basculez du Brésil Sud en tant que région source vers une cible, la restauration automatique vers le Brésil Sud à partir de la région cible est prise en charge.
> - Vous pouvez travailler dans des régions pour lesquelles vous disposez d'un accès approprié.
> - Si la région où vous souhaitez créer un coffre ne s'affiche pas, assurez-vous que votre abonnement dispose d'un accès lui permettant de créer des ressources dans cette région.
> - Si vous ne parvenez pas à voir une région au sein d’un cluster géographique lorsque vous activez la réplication, assurez-vous que votre abonnement dispose des autorisations nécessaires pour créer des machines virtuelles dans cette région.



## <a name="cache-storage"></a>Stockage du cache

Le tableau suivant récapitule la prise en charge du compte de stockage du cache utilisé par Site Recovery lors de la réplication.

**Paramètre** | **Support** | **Détails**
--- | --- | ---
Comptes de stockage V2 à usage général (niveaux chaud et froid) | Prise en charge | L'utilisation de GPv2 est déconseillée, car les coûts de transaction pour V2 sont sensiblement plus élevés que pour les comptes de stockage V1.
Stockage Premium | Non pris en charge | Les comptes de stockage standard sont utilisés pour le stockage du cache, afin d’optimiser les coûts.
Pare-feux du Stockage Azure pour réseaux virtuels  | Prise en charge | Si vous utilisez un compte de stockage de cache ou cible autorisé par le pare-feu, vérifiez que l’option « [Autoriser les services Microsoft approuvés](../storage/common/storage-network-security.md#exceptions) » est activée.<br></br>Vérifiez également que vous autorisez l’accès à au moins un sous-réseau de réseau virtuel source.


## <a name="replicated-machine-operating-systems"></a>Systèmes d’exploitation de machine répliquée

Site Recovery prend en charge la réplication de machines virtuelles Azure exécutant les systèmes d’exploitation répertoriés dans cette section. Notez que si une machine déjà répliquée est ensuite mise à niveau (ou rétrogradée) vers un autre noyau principal, vous devez désactiver la réplication et la réactiver après la mise à niveau.

### <a name="windows"></a>Windows


**Système d’exploitation** | **Détails**
--- | ---
Windows Server 2019 | Pris en charge pour Server Core, Server avec Expérience utilisateur.
Windows Server 2016  | Prise en charge de Server Core, Server avec Expérience utilisateur.
Windows Server 2012 R2 | Pris en charge.
Windows Server 2012 | Pris en charge.
Windows Server 2008 R2 avec SP1/SP2 | Pris en charge.<br/><br/> À partir de la version [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de l’extension du service Mobility pour les machines virtuelles Azure, vous devez installer une [mise à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) Windows et une[ mise à jour SHA-2](https://support.microsoft.com/help/4474419) sur les machines exécutant Windows Server 2008 R2 SP1/SP2.  SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Pris en charge.
Windows 8.1 (x64) | Pris en charge.
Windows 8 (x64) | Pris en charge.
Windows 7 (x64) avec SP1 et versions ultérieures | À partir de la version [9.30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) de l’extension du service Mobility pour les machines virtuelles Azure, vous devez installer une [mise à jour de la pile de maintenance (SSU)](https://support.microsoft.com/help/4490628) Windows et une[ mise à jour SHA-2](https://support.microsoft.com/help/4474419) sur les machines exécutant Windows 7 avec SP1.  SHA-1 n’est pas pris en charge à partir de septembre 2019, et si la signature de code SHA-2 n’est pas activée, l’extension de l’agent ne sera pas installée/mise à niveau comme prévu. En savoir plus sur la [mise à niveau et la configuration requise pour SHA-2](https://aka.ms/SHA-2KB).



#### <a name="linux"></a>Linux

**Système d’exploitation** | **Détails**
--- | ---
Red Hat Enterprise Linux | 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6,[7.7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4564347/), [7.9](https://support.microsoft.com/help/4578241/), [8.0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8.1, [8.2](https://support.microsoft.com/help/4570609/), [8.3](https://support.microsoft.com/help/4597409/)
CentOS | 6.5, 6.6, 6.7, 6.8, 6.9, 6.10 </br> 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, [7.8](https://support.microsoft.com/help/4564347/), [version pré-GA 7.9](https://support.microsoft.com/help/4578241/), la version GA 7.9 est prise en charge à partir du correctif logiciel à chaud** 9.37 </br> 8.0, 8.1, [8.2](https://support.microsoft.com/en-us/help/4570609), [8.3](https://support.microsoft.com/help/4597409/)
Serveur LTS Ubuntu 14.04 | Prend en charge toutes les versions 14.04.*x* ; [Versions de noyau prises en charge](#supported-ubuntu-kernel-versions-for-azure-virtual-machines) ; 
Serveur LTS Ubuntu 16.04 | Prend en charge toutes les versions 16.04.*x* ; [Versions de noyau prises en charge](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Sur les serveurs Ubuntu utilisant l’authentification et la connexion basées sur un mot de passe, et le package cloud-init pour configurer des machines virtuelles cloud, la connexion basée sur un mot de passe peut être désactivée lors du basculement (en fonction de la configuration de cloudinit). La connexion basée sur un mot de passe peut être réactivée sur la machine virtuelle en réinitialisant le mot de passe dans le menu Support > Résolution des problèmes > Paramètres (de la machine virtuelle basculée sur le portail Azure).
Serveur Ubuntu 18.04 LTS | Prend en charge toutes les versions 18.04.*x* ; [Versions de noyau prises en charge](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Sur les serveurs Ubuntu utilisant l’authentification et la connexion basées sur un mot de passe, et le package cloud-init pour configurer des machines virtuelles cloud, la connexion basée sur un mot de passe peut être désactivée lors du basculement (en fonction de la configuration de cloudinit). La connexion basée sur un mot de passe peut être réactivée sur la machine virtuelle en réinitialisant le mot de passe dans le menu Support > Résolution des problèmes > Paramètres (de la machine virtuelle basculée sur le portail Azure).
Serveur Ubuntu 20.04 LTS | Prend en charge toutes les versions 20.04.*x* ; [Versions de noyau prises en charge](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Debian 7 | Inclut la prise en charge pour toutes les versions 7. *x* [Versions du noyau prises en charge](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | Inclut la prise en charge pour toutes les versions 8. *x* [Versions du noyau prises en charge](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 9 | Inclut la prise en charge pour les versions 9.1 à 9.13. Debian 9.0 n’est pas pris en charge. [Versions du noyau prises en charge](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4, SP5 [(Versions du noyau prises en charge)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 15 | 15, SP1, SP2[(versions du noyau prises en charge)](#supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | SP3<br/><br/> La mise à niveau des machines de réplication SP3 vers SP4 n’est pas prise en charge. Si une machine répliquée a été mise à niveau, vous devez désactiver la réplication et la réactiver après la mise à niveau.
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, [7.7](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery), [7.8](https://support.microsoft.com/help/4573888/), [7.9](https://support.microsoft.com/help/4597409), [8.0](https://support.microsoft.com/help/4573888/), [8.1](https://support.microsoft.com/help/4573888/) (exécutant le noyau compatible Red Hat ou le noyau Unbreakable Enterprise Kernel Release 3, 4 et 5 (UEK3, UEK4, UEK5)<br/><br/>8.1 (l’exécution sur tous les noyaux UEK et le noyau RedHat <= 3.10.0-1062.* est prise en charge dans [9.35](https://support.microsoft.com/help/4573888/). La prise en charge des autres noyaux RedHat est disponible dans [9.36](https://support.microsoft.com/help/4578241/))

> [!NOTE]
> Pour les versions de Linux, Azure Site Recovery ne prend pas en charge les images de système d’exploitation personnalisées. Seuls les noyaux de stockage qui font partie de la version/mise à jour mineure de distribution sont pris en charge.

**Remarque : Pour prendre en charge les noyaux Linux les plus récents dans un délai de 15 jours à partir de la publication, Azure Site Recovery déploie le correctif logiciel à chaud en plus de la dernière version de l’agent de mobilité. Ce correctif est déployé dans les deux versions majeures de la publication. Pour effectuer une mise à jour vers la dernière version de l’agent de mobilité (y compris le correctif logiciel à chaud), suivez les étapes mentionnées dans [cet article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). Ce correctif est actuellement déployé pour les agents de mobilité utilisés dans le scénario de reprise d’activité Azure vers Azure.

#### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Versions du noyau Ubuntu prises en charge pour les machines virtuelles Azure

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
14.04 LTS | [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/), [9.39](https://support.microsoft.com/help/4597409/)| 3.13.0-24-generic à 3.13.0-170-generic,<br/>3.16.0-25-generic à 3.16.0-77-generic,<br/>3.19.0-18-generic à 3.19.0-80-generic,<br/>4.2.0-18-generic à 4.2.0-42-generic,<br/>4.4.0-21-generic à 4.4.0-148-generic,<br/>4.15.0-1023-azure à 4.15.0-1045-azure |
|||
LTS 16.04 | [9.39](https://support.microsoft.com/help/4597409/) | 4.4.0-21-generic à 4.4.0-194-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-123-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1098-azure|
LTS 16.04 | [9.38](https://support.microsoft.com/help/4590304/) | 4.4.0-21-generic à 4.4.0-190-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-118-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1096-azure </br> 4.4.0-193-generic, 4.15.0-120-generic, 4.15.0-122-generic, 4.15.0-1098-azure à 9.38 (correctif logiciel à chaud)**|
LTS 16.04 | [9.37](https://support.microsoft.com/help/4582666/) | 4.4.0-21-generic à 4.4.0-189-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-115-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1093-azure </br> 4.4.0-190-generic, 4.15.0-117-generic, 4.15.0-118-generic, 4.15.0-1095-azure et 4.15.0-1096-azure à 9.37 (correctif logiciel à chaud)**|
LTS 16.04 | [9.36](https://support.microsoft.com/help/4578241/)| 4.4.0-21-generic à 4.4.0-187-generic,<br/>4.8.0-34-generic à 4.8.0-58-generic,<br/>4.10.0-14-generic à 4.10.0-42-generic,<br/>4.11.0-13-generic à 4.11.0-14-generic,<br/>4.13.0-16-generic à 4.13.0-45-generic,<br/>4.15.0-13-generic à 4.15.0-112-generic<br/>4.11.0-1009-azure à 4.11.0-1016-azure,<br/>4.13.0-1005-azure à 4.13.0-1018-azure <br/>4.15.0-1012-azure à 4.15.0-1092-azure |
|||
18.04 LTS | [9.39](https://support.microsoft.com/help/4597409/) | 4.15.0-20-generic à 4.15.0-123-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-63-generic </br> 5.3.0-19-generic à 5.3.0-69-generic </br> 5.4.0-37-generic à 5.4.0-53-generic</br> 4.15.0-1009-azure à 4.15.0-1099-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1031-azure|
18.04 LTS | [9.38](https://support.microsoft.com/help/4590304/) | 4.15.0-20-generic à 4.15.0-118-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-61-generic </br> 5.3.0-19-generic à 5.3.0-67-generic </br> 5.4.0-37-generic à 5.4.0-48-generic</br> 4.15.0-1009-azure à 4.15.0-1096-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1026-azure </br> 4.15.0-121-generic, 4.15.0-122-generic, 5.0.0-62-generic, 5.3.0-68-generic, 5.4.0-51-generic, 5.4.0-52-generic, 4.15.0-1099-azure,  5.4.0-1031-azure à 9.38 (correctif logiciel à chaud)**|
18.04 LTS | [9.37](https://support.microsoft.com/help/4582666/) | 4.15.0-20-generic à 4.15.0-115-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-60-generic </br> 5.3.0-19-generic à 5.3.0-66-generic </br> 5.4.0-37-generic à 5.4.0-45-generic</br> 4.15.0-1009-azure à 4.15.0-1093-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1035-azure </br> 5.4.0-1020-azure à 5.4.0-1023-azure</br> 4.15.0-117-generic, 4.15.0-118-generic, 5.0.0-61-generic, 5.3.0-67-generic, 5.4.0-47-generic, 5.4.0-48-generic, 4.15.0-1095-azure, 4.15.0-1096-azure, 5.4.0-1025-azure, 5.4.0-1026-azure avec correctif logiciel à chaud** 9.37|
18.04 LTS | [9.36](https://support.microsoft.com/help/4578241/) | 4.15.0-20-generic à 4.15.0-112-generic </br> 4.18.0-13-générique à 4.18.0-25-générique </br> 5.0.0-15-generic à 5.0.0-58-generic </br> 5.3.0-19-generic à 5.3.0-65-generic </br> 5.4.0-37-generic à 5.4.0-42-generic</br> 4.15.0-1009-azure à 4.15.0-1092-azure </br> 4.18.0-1006-azure à 4.18.0-1025-azure </br> 5.0.0-1012-azure à 5.0.0-1036-azure </br> 5.3.0-1007-azure à 5.3.0-1032-azure </br> 5.4.0-1020-azure à 5.4.0-1022-azure </br> 5.0.0-60-generic et 5.3.0-1035-azure via le correctif logiciel à chaud 9.36**|
|||
20.04 LTS |[9.39](https://support.microsoft.com/help/4597409/) | 5.4.0-26-generic à 5.4.0-53 </br> -generic 5.4.0-1010-azure à 5.4.0-1031-azure
20.04 LTS |[9.38](https://support.microsoft.com/help/4590304/) | 5.4.0-26-generic à 5.4.0-48 </br> -generic 5.4.0-1010-azure à 5.4.0-1026-azure </br> 5.4.0-51-generic, 5.4.0-52-generic, 5.8.0-23-generic, 5.8.0-25-generic, 5.4.0-1031-azure via le correctif logiciel à chaud 9.38**
20.04 LTS |[9.37](https://support.microsoft.com/help/4582666/) | 5.4.0-26-generic à 5.4.0-45 </br> -generic 5.4.0-1010-azure à 5.4.0-1023-azure </br> 5.4.0-47-generic, 5.4.0-48-generic, 5.4.0-1025-azure, 5.4.0-1026-azure via le correctif logiciel à chaud** 9.37
20.04 LTS |[9.36](https://support.microsoft.com/help/4578241/) | 5.4.0-26-generic à 5.4.0-42 </br> -generic 5.4.0-1010-azure à 5.4.0-1022-azure

**Remarque : Pour prendre en charge les noyaux Linux les plus récents dans un délai de 15 jours à partir de la publication, Azure Site Recovery déploie le correctif logiciel à chaud en plus de la dernière version de l’agent de mobilité. Ce correctif est déployé dans les deux versions majeures de la publication. Pour effectuer une mise à jour vers la dernière version de l’agent de mobilité (y compris le correctif logiciel à chaud), suivez les étapes mentionnées dans [cet article](service-updates-how-to.md#azure-vm-disaster-recovery-to-azure). Ce correctif est actuellement déployé pour les agents de mobilité utilisés dans le scénario de reprise d’activité Azure vers Azure.

#### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Versions du noyau Debian prises en charge pour les machines virtuelles Azure

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
Debian 7 | [9.35](https://support.microsoft.com/help/4573888/), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/), [9.39](https://support.microsoft.com/help/4597409/) | 3.2.0-4-amd64 à 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | [9.35](https://support.microsoft.com/help/4573888/, ), [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/), [9.38](https://support.microsoft.com/help/4590304/), [9.39](https://support.microsoft.com/help/4597409/) | 3.16.0-4-amd64 à 3.16.0-11-amd64, 4.9.0-0.bpo.4-amd64 à 4.9.0-0.bpo.11-amd64 |
Debian 8 | [9.34](https://support.microsoft.com/help/4570609) | 3.16.0-4-amd64 à 3.16.0-10-amd64, 4.9.0-0.bpo.4-amd64 à 4.9.0-0.bpo.11-amd64 |
|||
Debian 9.1 | [9.39](https://support.microsoft.com/help/4597409/) | 4.9.0-1-amd64 à 4.9.0-14-amd64 </br> 4.19.0-0.bpo.1-amd64 à 4.19.0-0.bpo.12-amd64 </br> 4.19.0-0.bpo.1-cloud-amd64 à 4.19.0-0.bpo.12-cloud-amd64 </br> 
Debian 9.1 | [9.38](https://support.microsoft.com/help/4590304/) | 4.9.0-1-amd64 à 4.9.0-13-amd64 </br> 4.19.0-0.bpo.1-amd64 à 4.19.0-0.bpo.11-amd64 </br> 4.19.0-0.bpo.1-cloud-amd64 à 4.19.0-0.bpo.11-cloud-amd64 </br> 4.9.0-14-amd64, 4.19.0-0.bpo.12-amd64, 4.19.0-0.bpo.12-cloud-amd64 via le correctif logiciel à chaud 9.38**
Debian 9.1 | [9.37](https://support.microsoft.com/help/4582666/) | 4.9.0-3-amd64 to 4.9.0-13-amd64, 4.19.0-0.bpo.6-amd64 à 4.19.0-0.bpo.10-amd64, 4.19.0-0.bpo.6-cloud-amd64 à 4.19.0-0.bpo.10-cloud-amd64

#### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Versions du noyau SUSE Linux Enterprise Server 12 prises en charge pour les machines virtuelles Azure

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.39](https://support.microsoft.com/help/4597409/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.34-azure |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.38](https://support.microsoft.com/help/4590304/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.28-azure |
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.22-azure </br> 4.12.14-16.25-azure, 4.12.14-16.28-azure via le correctif logiciel à chaud 9.37**|
SUSE Linux Enterprise Server 12 (SP1,SP2,SP3,SP4, SP5) | [9.35](https://support.microsoft.com/help/4573888/) | Tous les noyaux de [stock SUSE 12 SP1, SP2, SP3, SP4](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.4.138-4.7-azure à 4.4.180-4.31-azure,</br>4.12.14-6.3-azure à 4.12.14-6.43-azure </br> 4.12.14-16.7-azure à 4.12.14-16.19-azure  |

#### <a name="supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines"></a>Versions du noyau SUSE Linux Enterprise Server 15 prises en charge pour les machines virtuelles Azure

**Version release** | **Version du service Mobilité** | **Version du noyau** |
--- | --- | --- |
SUSE Linux Enterprise Server 15, SP1, SP2 | [9.39](https://support.microsoft.com/help/4597409/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.47-azure </br> 5.3.18-16-azure </br> 5.3.18-18.5-azure à 5.3.18-18.21-azure
SUSE Linux Enterprise Server 15, SP1, SP2 | [9.38](https://support.microsoft.com/help/4590304/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.44-azure </br> 5.3.18-16-azure </br> 5.3.18-18.5-azure à 5.3.18-18.18-azure </br> 4.12.14-8.47-azure, 5.3.18-18.21-azure à 9.38 (correctif logiciel à chaud)**
SUSE Linux Enterprise Server 15 et 15 SP1 | [9.36](https://support.microsoft.com/help/4578241/), [9.37](https://support.microsoft.com/help/4582666/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.38-azure </br> 4.12.14-8.41-azure, 4.12.14-8.44-azure via le correctif logiciel à chaud** 9.37
SUSE Linux Enterprise Server 15 et 15 SP1 | [9.35](https://support.microsoft.com/help/4573888/)  | Par défaut, tous les [noyaux de stock SUSE 15 et 15](https://www.suse.com/support/kb/doc/?id=000019587) sont pris en charge.</br></br> 4.12.14-5.5-azure à 4.12.14-5.47-azure </br></br> 4.12.14-8.5-azure à 4.12.14-8.33-azure 


## <a name="replicated-machines---linux-file-systemguest-storage"></a>Machines répliquées - Stockage invité/système de fichiers Linux

* Systèmes de fichiers : ext3, ext4, XFS, BTRFS
* Gestionnaire de volume : LVM2

> [!NOTE]
> Le logiciel multichemin n’est pas pris en charge.


## <a name="replicated-machines---compute-settings"></a>Machines répliquées - Paramètres de calcul

**Paramètre** | **Support** | **Détails**
--- | --- | ---
Taille | N’importe quelle taille de machine virtuelle Azure avec au moins 2 cœurs d’UC et 1 Go de RAM | Consultez [Tailles de machine virtuelle Azure](../virtual-machines/sizes.md).
Groupes à haute disponibilité | Prise en charge | Si vous activez la réplication pour une machine virtuelle Azure avec les options par défaut, un groupe à haute disponibilité est créé automatiquement, selon les paramètres de la région source. Vous pouvez modifier ces paramètres.
Zones de disponibilité | Prise en charge |
HUB (Hybrid Use Benefit) | Prise en charge | Si la machine virtuelle source a une licence HUB activée, une machine virtuelle de basculement ou de test de basculement utilise également la licence HUB.
Groupes identiques de machines virtuelles | Non pris en charge |
Images de la galerie Azure - Publiées par Microsoft | Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Images de la galerie Azure - Publiées par un tiers | Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Images personnalisées - Publiées par un tiers | Prise en charge | Prises en charge si la machine virtuelle s’exécute sur un système d’exploitation pris en charge.
Machines virtuelles migrées à l’aide de Site Recovery | Prise en charge | Si un ordinateur physique une machine virtuelle VMware a été migré(e) vers Azure à l’aide de Site Recovery, vous devez désinstaller l’ancienne version du service Mobilité sur l’ordinateur et redémarrer celui-ci avant de le répliquer vers une autre région Azure.
Stratégies Azure RBAC | Non pris en charge | Les stratégies de contrôle d’accès en fonction du rôle Azure (Azure RBAC) sur les machines virtuelles ne sont pas répliquées sur la machine virtuelle de basculement dans la région cible.
Extensions | Non pris en charge | Les extensions ne sont pas répliquées sur la machine virtuelle de basculement dans la région cible. Elles doivent être installées manuellement après le basculement.
Groupes de placement de proximité | Prise en charge | Les machines virtuelles situées à l’intérieur d’un groupe de placement de proximité peuvent être protégées avec Site Recovery.


## <a name="replicated-machines---disk-actions"></a>Machines répliquées - Actions de disque

**Action** | **Détails**
-- | ---
Redimensionner le disque sur la machine virtuelle répliquée | Pris en charge sur la machine virtuelle source avant le basculement. Vous n’avez pas besoin de désactiver/réactiver la réplication.<br/><br/> Si vous modifiez la machine virtuelle source après le basculement, les modifications ne sont pas capturées.<br/><br/> Si vous modifiez la taille du disque sur la machine virtuelle Azure après le basculement, les modifications ne sont pas capturées par Site Recovery et la restauration automatique sera à la taille de la machine virtuelle d’origine.
Ajouter un disque à une machine virtuelle répliquée | Prise en charge
Modifications hors connexion apportées aux disques protégés | La déconnexion des disques et leur modification hors connexion nécessitent le déclenchement d’une resynchronisation complète.

## <a name="replicated-machines---storage"></a>Machines répliquées - Stockage

Ce tableau récapitule la prise en charge du disque du système d’exploitation, du disque de données et du disque temporaire de la machine virtuelle Azure.

- Il est important d’observer les limites des disques de machine virtuelle et les cibles des machines virtuelles [Linux](../virtual-machines/linux/disk-scalability-targets.md) et [Windows](../virtual-machines/windows/disk-scalability-targets.md) pour éviter tout problème de performances.
- Si vous déployez avec les paramètres par défaut, Site Recovery crée automatiquement les disques et les comptes de stockage en fonction des paramètres de la source.
- Si vous les personnalisez, veillez à respecter les instructions.

**Composant** | **Support** | **Détails**
--- | --- | ---
Taille maximale du disque du système d’exploitation | 2048 GB | [En savoir plus](../virtual-machines/managed-disks-overview.md) sur les disques de machines virtuelles.
Disque temporaire | Non pris en charge | Le disque temporaire est toujours exclu de la réplication.<br/><br/> Ne conservez pas de données persistantes sur le disque temporaire. [Plus d’informations](../virtual-machines/managed-disks-overview.md)
Taille maximale du disque de données | 32 To pour les disques managés<br></br>4 To pour les disques non managés|
Taille minimale du disque de données | Aucune restriction pour les disques non managés. 2 Go pour les disques managés |
Nombre maximal de disques de données | Jusqu’à 64, en adéquation avec la prise en charge pour une taille spécifique de machine virtuelle Azure | [En savoir plus](../virtual-machines/sizes.md) sur les tailles de machines virtuelles.
Taux de modification du disque de données | 20 Mbit/s maximum par disque pour le Stockage Premium. 2 Mbits/s maximum par disque pour le stockage Standard. | Si le taux moyen de modification des données sur le disque est en permanence supérieur à la valeur maximale, la réplication ne pourra pas suivre.<br/><br/>  Toutefois, si la valeur maximale est dépassée de manière sporadique, la réplication peut suivre, mais les points de récupération pourraient être légèrement différés.
Disque de données - Compte de stockage Standard | Prise en charge |
Disque de données - Compte de stockage Premium | Prise en charge | Si une machine virtuelle a des disques répartis sur des comptes de stockage Standard et Premium, vous pouvez sélectionner un compte de stockage cible différent pour chaque disque afin d’être sûr d’avoir la même configuration de stockage dans la région cible.
Disque managé - Standard | Pris en charge dans les régions Azure dans lesquelles Azure Site Recovery est pris en charge. |
Disque managé - Premium | Pris en charge dans les régions Azure dans lesquelles Azure Site Recovery est pris en charge. |
SSD Standard | Prise en charge |
Redondance | LRS et GRS sont pris en charge.<br/><br/> ZRS n’est pas pris en charge.
Stockage à froid et à chaud | Non pris en charge | Les disques de machine virtuelle ne sont pas pris en charge sur le stockage à froid et à chaud
Espaces de stockage | Prise en charge |
Chiffrement au repos (SSE) | Prise en charge | SSE est le paramètre par défaut sur les comptes de stockage.
Chiffrement au repos (CMK) | Prise en charge | Les clés HSM et logicielles sont prises en charge pour les disques managés
Double chiffrement au repos | Prise en charge | En savoir plus sur les régions prises en charge pour [Windows](../virtual-machines/disk-encryption.md) et [Linux](../virtual-machines/disk-encryption.md).
Azure Disk Encryption (ADE) pour système d’exploitation Windows | Pris en charge pour les machines virtuelles avec des disques managés. | Les machines virtuelles utilisant des disques non managés ne sont pas prises en charge. <br/><br/> Les clés protégées par HSM ne sont pas prises en charge. <br/><br/> Le chiffrement de volumes individuels sur un seul disque n’est pas pris en charge. |
Azure Disk Encryption (ADE) pour système d’exploitation Linux | Pris en charge pour les machines virtuelles avec des disques managés. | Les machines virtuelles utilisant des disques non managés ne sont pas prises en charge. <br/><br/> Les clés protégées par HSM ne sont pas prises en charge. <br/><br/> Le chiffrement de volumes individuels sur un seul disque n’est pas pris en charge. <br><br> Problème connu avec l’activation de la réplication. [En savoir plus.](./azure-to-azure-troubleshoot-errors.md#enable-protection-failed-as-the-installer-is-unable-to-find-the-root-disk-error-code-151137) |
Rotation de clé SAS | Non pris en charge | Si la clé SAS pour les comptes de stockage fait l’objet d’une rotation, le client doit désactiver et réactiver la réplication. |
Ajout à chaud    | Prise en charge | L'activation de la réplication pour un disque de données que vous ajoutez à une machine virtuelle Azure répliquée est prise en charge pour les machines virtuelles utilisant des disques managés. <br/><br/> Un seul disque peut être ajouté à chaud à une machine virtuelle Azure à la fois. L’ajout parallèle de plusieurs disques n’est pas pris en charge. |
Retrait de disque à chaud    | Non pris en charge | Si vous retirez un disque de données de la machine virtuelle, vous devez désactiver la réplication puis la réactiver pour la machine virtuelle.
Exclure le disque | Pris en charge. Vous devez utiliser [PowerShell](azure-to-azure-exclude-disks.md) pour configurer. |    Les disques temporaires sont exclus par défaut.
Espaces de stockage direct  | Pris en charge pour les points de récupération cohérents d’incident. Les points de récupération cohérents d’incident ne sont pas pris en charge. |
Serveur de fichiers avec montée en puissance parallèle  | Pris en charge pour les points de récupération cohérents d’incident. Les points de récupération cohérents d’incident ne sont pas pris en charge. |
DRBD | Les disques qui font partie d’une installation DRBD ne sont pas pris en charge. |
LRS | Prise en charge |
GRS | Prise en charge |
RA-GRS | Prise en charge |
ZRS | Non pris en charge |
Stockage à froid et à chaud | Non pris en charge | Les disques de machine virtuelle ne sont pas pris en charge sur le stockage à froid et à chaud.
Pare-feux du Stockage Azure pour réseaux virtuels  | Prise en charge | Si vous limitez l’accès au réseau virtuel aux comptes de stockage, activez l'option [Autoriser les services Microsoft de confiance](../storage/common/storage-network-security.md#exceptions).
Comptes de stockage V2 à usage général (niveaux chaud et froid) | Prise en charge | Augmentation significative des coûts de transaction par rapport aux comptes de stockage V1 à usage général
Génération 2 (démarrage UEFI) | Prise en charge
Disques NVMe | Non pris en charge
Disques partagés Azure | Non pris en charge
Option de transfert sécurisé | Prise en charge
Disques avec accélérateur d’écriture | Non pris en charge

>[!IMPORTANT]
> Pour éviter les problèmes de performances, assurez-vous de vous respecter les valeurs d'évolutivité et de performances des disques VM pour les machines virtuelles [Linux](../virtual-machines/linux/disk-scalability-targets.md) ou [Windows](../virtual-machines/windows/disk-scalability-targets.md). Si vous utilisez les paramètres par défaut, Site Recovery crée les disques et comptes de stockage requis en fonction de la configuration de la source. Si vous personnalisez et sélectionnez vos propres paramètres, suivez les cibles de scalabilité et de performances de disque de vos machines virtuelles sources.

## <a name="limits-and-data-change-rates"></a>Limites et taux de changement des données

Le tableau suivant récapitule les limites de Site Recovery.

- Ces limites sont basées sur nos tests, mais ne couvrent pas toutes les combinaisons d’E/S d’application possibles.
- Les résultats réels varient en fonction de la combinaison d’E/S de votre application.
- Il existe deux limites à prendre en compte, le taux d’activité de données par disque et le taux d’activité de données par machine virtuelle.
- La limite actuelle de l’activité des données par machine virtuelle est de 54 Mo/s, quelle que soit la taille.

**Cible de stockage** | **E/S moyennes de disque source** |**Activité des données moyenne de disque source** | **Total de l’activité des données de disque source par jour**
---|---|---|---
Stockage Standard | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 8 Ko    | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 16 Ko | 4 Mo/s |    336 Go par disque
Disque Premium P10 ou P15 | 32 Ko ou plus | 8 Mo/s | 672 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 8 Ko    | 5 Mo/s | 421 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 16 Ko ou plus |20 Mo/s | 1 684 Go par disque

## <a name="replicated-machines---networking"></a>Machines répliquées - Mise en réseau
**Paramètre** | **Support** | **Détails**
--- | --- | ---
Carte d’interface réseau | Nombre maximal pris en charge pour une taille de machine virtuelle Azure spécifique | Les cartes réseau sont créées lors de la création de la machine virtuelle pendant le basculement.<br/><br/> Le nombre de cartes réseau sur la machine virtuelle de basculement dépend du nombre de cartes réseau que possède la machine virtuelle source au moment de l’activation de la réplication. Si vous ajoutez ou supprimez une carte réseau après l’activation de la réplication, cela n’affecte pas le nombre de cartes réseau sur la machine virtuelle répliquée après le basculement. <br/><br/> Il n’est pas garanti que l’ordre des cartes réseau après basculement soit le même que l’ordre d’origine. <br/><br/> Vous pouvez renommer des cartes réseau dans la région cible conformément aux conventions de nommage de votre organisation. Le renommage des cartes réseau est pris en charge en utilisant PowerShell.
Équilibreur de charge Internet | Non pris en charge | Vous pouvez configurer des équilibreurs de charge publics ou Internet dans la région primaire. Toutefois, les équilibreurs de charge publics/Internet ne sont pas pris en charge par Azure Site Recovery dans la région de récupération d’urgence.
Équilibreur de charge interne | Prise en charge | Associez l’équilibreur de charge préconfiguré à l’aide d’un script Azure Automation dans un plan de récupération.
Adresse IP publique | Prise en charge | Associez une adresse IP publique existante à la carte réseau. Ou, créez une adresse IP publique et associez-la à la carte réseau à l’aide d’un script Azure Automation dans un plan de récupération.
Groupe de sécurité réseau (NSG) sur la carte réseau | Prise en charge | Associez le groupe de sécurité réseau à la carte réseau à l’aide d’un script Azure Automation dans un plan de récupération.
Groupe de sécurité réseau (NSG) sur le sous-réseau | Prise en charge | Associez le groupe de sécurité réseau au sous-réseau à l’aide d’un script Azure Automation dans un plan de récupération.
Adresse IP (statique) réservée | Prise en charge | Si la carte réseau sur la machine virtuelle source a une adresse IP statique et que le sous-réseau cible a la même adresse IP disponible, celle-ci est affectée à la machine virtuelle de basculement.<br/><br/> Si le sous-réseau cible n’a pas la même adresse IP disponible, l’une des adresses IP disponibles sur le sous-réseau est réservée à la machine virtuelle.<br/><br/> Vous pouvez également spécifier une adresse IP fixe et un sous-réseau dans **Éléments répliqués** > **Paramètres** > **Calcul et réseau** > **Interfaces réseau**.
Adresse IP dynamique | Prise en charge | Si la carte réseau sur la machine virtuelle source a l’adressage IP dynamique, la carte réseau sur la machine virtuelle de basculement est également dynamique par défaut.<br/><br/> Vous pouvez remplacer cela par une adresse IP fixe si nécessaire.
Plusieurs adresses IP | Non pris en charge | Lorsque vous basculez une machine virtuelle équipée d'une carte réseau avec plusieurs adresses IP, seule l’adresse IP principale de la carte réseau de la région source est conservée. Pour attribuer plusieurs adresses IP, vous pouvez ajouter des machines virtuelles à un [plan de récupération](recovery-plan-overview.md) et joindre un script pour attribuer des adresses IP supplémentaires à ce plan. Vous pouvez également procéder à la modification manuellement ou à l'aide d'un script après basculement.
Traffic Manager     | Prise en charge | Vous pouvez préconfigurer Traffic Manager pour que le trafic soit acheminé vers le point de terminaison dans la région source de manière régulière et vers le point de terminaison dans la région cible en cas de basculement.
Azure DNS | Prise en charge |
Système DNS personnalisé    | Prise en charge |
Proxy non authentifié | Prise en charge | [En savoir plus](./azure-to-azure-about-networking.md)
Proxy authentifié | Non pris en charge | Si la machine virtuelle utilise un proxy authentifié pour la connectivité sortante, elle ne peut pas être répliquée à l’aide d’Azure Site Recovery.
Connexion VPN de site à site à un environnement local<br/><br/>(avec ou sans ExpressRoute)| Prise en charge | Vérifiez que les itinéraires définis par l’utilisateur et les groupes de sécurité réseau sont configurés de telle sorte que le trafic Site Recovery ne soit pas acheminé vers l’infrastructure locale. [En savoir plus](./azure-to-azure-about-networking.md)
Connexion de réseau virtuel à réseau virtuel    | Prise en charge | [En savoir plus](./azure-to-azure-about-networking.md)
Points de terminaison de service de réseau virtuel | Prise en charge | Si vous limitez l’accès au réseau virtuel aux comptes de stockage, assurez-vous que les services Microsoft de confiance sont autorisés à accéder au compte de stockage.
Mise en réseau accélérée | Prise en charge | L’accélération réseau doit être activée sur la machine virtuelle source. [Plus d’informations](azure-vm-disaster-recovery-with-accelerated-networking.md)
Appliance Palo Alto Network | Non pris en charge | Concernant les appliances tierces, il existe souvent des restrictions imposées par le fournisseur à l’intérieur de la machine virtuelle. Azure Site Recovery nécessite une connectivité sortante, d’agent et d’extensions pour être disponible. Toutefois, l’appliance ne permet pas de configurer une activité sortante à l’intérieur de la machine virtuelle.
IPv6  | Non pris en charge | Les configurations mixtes qui incluent à la fois IPv4 et IPv6 ne sont pas non plus prises en charge. Libérez le sous-réseau de la plage IPv6 avant toute opération de Site Recovery.
Lien privé d’accès au service Site Recovery | Prise en charge | [En savoir plus](azure-to-azure-how-to-enable-replication-private-endpoints.md)



## <a name="next-steps"></a>Étapes suivantes

- Lisez la [mise en réseau pour la réplication des machines virtuelles Azure](./azure-to-azure-about-networking.md).
- Déployez la récupération d’urgence en [répliquant des machines virtuelles Azure](./azure-to-azure-quickstart.md).
