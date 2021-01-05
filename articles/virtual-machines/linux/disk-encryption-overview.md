---
title: Activer Azure Disk Encryption pour les machines virtuelles Linux
description: Cet article fournit des instructions pour l’activation de Microsoft Azure Disk Encryption pour les machines virtuelles Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 91ef5ca35cc96aa2028522d370ffbade45ecc2de
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779768"
---
# <a name="azure-disk-encryption-for-linux-vms"></a>Azure Disk Encryption pour les machines virtuelles Linux 

Azure Disk Encryption vous aide à protéger et à préserver vos données de façon à répondre aux engagements de votre entreprise en matière de sécurité et de conformité. Il utilise la fonctionnalité [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux afin de fournir un chiffrement de volume pour le système d’exploitation et les disques de données des machines virtuelles Azure. Il s’intègre à [Azure Key Vault](../../key-vault/index.yml) pour vous aider à contrôler et à gérer les clés de chiffrement de disque et les secrets. 

Si vous utilisez [Azure Security Center](../../security-center/index.yml), vous recevez une alerte dès lors que certaines de vos machines virtuelles ne sont pas chiffrées. Les alertes indiquent un niveau de gravité élevé et recommandent de chiffrer ces machines virtuelles.

![Alerte de chiffrement de disque Azure Security Center](media/disk-encryption/security-center-disk-encryption-fig1.png)

> [!WARNING]
> - Si vous avez déjà utilisé Azure Disk Encryption avec Azure AD pour chiffrer une machine virtuelle, vous devez continuer à utiliser cette option pour chiffrer votre machine virtuelle. Pour plus d’informations, consultez [Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-overview-aad.md). 
> - Certaines recommandations peuvent entraîner une augmentation de l’utilisation des données, des réseaux ou des ressources de calcul débouchant sur des coûts de licence ou d’abonnement supplémentaires. Vous devez disposer d’un abonnement Azure actif valide pour créer des ressources dans Azure dans les régions prises en charge.
> - Actuellement, les machines virtuelles de 2e génération ne prennent pas en charge Azure Disk Encryption. Pour plus d’informations, consultez [Prise en charge des machines virtuelles de 2e génération dans Azure](../generation-2.md).

Vous pouvez découvrir les notions de base d’Azure Disk Encryption pour Linux en quelques minutes avec les guides de démarrage rapide [Créer et chiffrer une machine virtuelle Linux avec Azure CLI](disk-encryption-cli-quickstart.md) et [Créer et chiffrer une machine virtuelle Linux avec Azure PowerShell](disk-encryption-powershell-quickstart.md).

## <a name="supported-vms-and-operating-systems"></a>Machines virtuelles et systèmes d’exploitation pris en charge

### <a name="supported-vms"></a>Machines virtuelles prises en charge

Les machines virtuelles Linux sont disponibles dans une [gamme de tailles](../sizes.md). Azure Disk Encryption n’est pas disponible sur les [machines virtuelles De base et de série A](https://azure.microsoft.com/pricing/details/virtual-machines/series/), ni sur celles qui ne disposent pas de la mémoire minimale requise comme indiqué ci-dessous :

| Machine virtuelle | Mémoire minimale requise |
|--|--|
| Machines virtuelles Linux lors du chiffrement des volumes de données uniquement| 2 Go |
| Machines virtuelles Linux lors du chiffrement des volumes de système d’exploitation et de données, et où l’utilisation du système de fichiers racine (/) est de 4 Go ou moins | 8 Go |
| Machines virtuelles Linux lors du chiffrement des volumes de système d’exploitation et de données, et où l’utilisation du système de fichiers racine (/) est supérieure à 4 Go | L’utilisation du système de fichiers racine * 2. Par exemple, une utilisation du système de fichiers racine de 16 Go nécessite au moins 32 Go de RAM |

Une fois que le processus de chiffrement du disque de système d’exploitation est terminé sur les machines virtuelles Linux, il est possible de configurer la machine virtuelle pour qu’elle s’exécute avec moins de mémoire. 

Azure Disk Encryption est également disponible pour les machines virtuelles avec stockage premium.

Azure Disk Encryption n’est pas disponible sur les [machines virtuelles de 2e génération](../generation-2.md#generation-1-vs-generation-2-capabilities) et les [machines virtuelles de la série Lsv2](../lsv2-series.md). Pour plus d’exceptions, consultez [Azure Disk Encryption : Scénarios non pris en charge](disk-encryption-linux.md#unsupported-scenarios).

Azure Disk Encryption n’est pas disponible sur les images de machine virtuelle sans disque temporaire (Dv4, Dsv4, Ev4 et Esv4).  Voir [Tailles de machine virtuelle Azure sans disque temporaire local](../azure-vms-no-temp-disk.md).

### <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Azure Disk Encryption est pris en charge sur un sous-ensemble de [distributions Linux approuvées par Azure](endorsed-distros.md), qui est lui-même un sous-ensemble de toutes les distributions de serveur Linux possibles.

![Diagramme de Venn de distributions de serveur Linux prenant en charge Azure Disk Encryption](./media/disk-encryption/ade-supported-distros.png)

Les distributions de serveur Linux qui ne sont pas approuvées par Azure ne prennent pas en charge Azure Disk Encryption. Parmi celles qui sont approuvées, seules les distributions et versions suivantes prennent en charge Azure Disk Encryption :

| Serveur de publication | Offre | SKU | URN | Type de volume pris en charge pour le chiffrement |
| --- | --- |--- | --- |
| Canonical | Ubuntu | 18.04-LTS | Canonical:UbuntuServer:18.04-LTS:latest | Disque de système d’exploitation et de données |
| Canonical | Ubuntu 18.04 | 18.04-DAILY-LTS | Canonical:UbuntuServer:18.04-DAILY-LTS:latest | Disque de système d’exploitation et de données |
| Canonical | Ubuntu 16.04 | 16.04-DAILY-LTS | Canonical:UbuntuServer:16.04-DAILY-LTS:latest | Disque de système d’exploitation et de données |
| Canonical | Ubuntu 14.04.5</br>[avec le noyau Azure mis à jour vers la version 4.15 ou ultérieure](disk-encryption-troubleshooting.md) | 14.04.5-LTS | Canonical:UbuntuServer:14.04.5-LTS:latest | Disque de système d’exploitation et de données |
| Canonical | Ubuntu 14.04.5</br>[avec le noyau Azure mis à jour vers la version 4.15 ou ultérieure](disk-encryption-troubleshooting.md) | 14.04.5-DAILY-LTS | Canonical:UbuntuServer:14.04.5-DAILY-LTS:latest | Disque de système d’exploitation et de données |
| Red Hat | RHEL 7.8 | 7.8 | RedHat:RHEL:7.8:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.7 | 7,7 | RedHat:RHEL:7.7:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7-LVM | 7-LVM | RedHat:RHEL:7-LVM: 7.8.2020111201 | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.6 | 7.6 | RedHat:RHEL:7.6:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.5 | 7.5 | RedHat:RHEL:7.5:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.4 | 7.4 | RedHat:RHEL:7.4:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.3 | 7.3 | RedHat:RHEL:7.3:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 7.2 | 7.2 | RedHat:RHEL:7.2:latest | Disque de système d’exploitation et de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 6.8 | 6.8 | RedHat:RHEL:6.8:latest | Disque de données (voir la remarque ci-dessous) |
| Red Hat | RHEL 6.7 | 6.7 | RedHat:RHEL:6.7:latest | Disque de données (voir la remarque ci-dessous) |
| OpenLogic | CentOS 7.8 | 7.8 | OpenLogic:CentOS:7_8:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.7 | 7,7 | OpenLogic:CentOS:7.7:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7-LVM | 7-LVM | OpenLogic:CentOS-LVM:7-LVM:7.8.2020111100 | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.6 | 7.6 | OpenLogic:CentOS:7.6:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.5 | 7.5 | OpenLogic:CentOS:7.5:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.4 | 7.4 | OpenLogic:CentOS:7.4:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.3 | 7.3 | OpenLogic:CentOS:7.3:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.2n | 7.2n | OpenLogic:CentOS:7.2n:latest | Disque de système d’exploitation et de données |
| OpenLogic | CentOS 7.1 | 7.1 | OpenLogic:CentOS:7.1:latest | Disque de données uniquement |
| OpenLogic | CentOS 7.0 | 7.0 | OpenLogic:CentOS:7.0:latest | Disque de données uniquement |
| OpenLogic | CentOS 6.8 | 6.8 | OpenLogic:CentOS:6.8:latest | Disque de données uniquement |
| SUSE | openSUSE 42.3 | 42.3 | SUSE:openSUSE-Leap:42.3:latest | Disque de données uniquement |
| SUSE | SLES 12-SP4 | 12-SP4 | SUSE:SLES:12-SP4:latest | Disque de données uniquement |
| SUSE | SLES HPC 12-SP3 | 12-SP3 | SUSE:SLES-HPC:12-SP3:latest | Disque de données uniquement |

> [!NOTE]
> La nouvelle implémentation d’Azure Disk Encryption est prise en charge pour les disques de système d’exploitation et de données RHEL pour les images avec Paiement à l’utilisation de RHEL7.  
>
> ADE est également pris en charge pour les images Gold Bring-Your-Own-Subscription de RHEL, mais uniquement **après** que l’abonnement ait été inscrit. Pour plus d’informations, consultez [Images Gold BYOS (Apportez votre propre abonnement) de Red Hat Enterprise Linux dans Azure](../workloads/redhat/byos.md#encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images)

## <a name="additional-vm-requirements"></a>Configuration requise supplémentaire pour les machines virtuelles

Azure Disk Encryption nécessite la présence des modules dm-crypt et vfat sur le système. La suppression ou la désactivation de vfat de l’image par défaut empêchera le système de lire le volume de clés et d’obtenir la clé nécessaire pour déverrouiller les disques lors des redémarrages suivants. Les étapes de renforcement du système qui suppriment le module vfat du système ou qui appliquent une extension des points de montage/dossiers du système d’exploitation sur les lecteurs de données ne sont pas compatibles avec Azure Disk Encryption. 

Avant d’activer le chiffrement, vous devez lister correctement les disques de données à chiffrer dans /etc/fstab. Utilisez l’option « nofail » lors de la création d’entrées et choisissez un nom d’appareil de bloc persistant (car les noms d’appareil au format « /dev/sdX » ne peuvent pas être associés au même disque entre les redémarrages, en particulier après le chiffrement. Pour plus d’informations sur ce comportement, consultez : [Résoudre les problèmes liés aux changements de noms d’appareil des machines virtuelles Linux](../troubleshooting/troubleshoot-device-names-problems.md)).

Vérifiez que les paramètres /etc/fstab sont correctement configurés pour le montage. Pour configurer ces paramètres, exécutez la commande mount -a, ou redémarrez la machine virtuelle et déclenchez le remontage de cette façon. Une fois cette opération effectuée, consultez la sortie de la commande lsblk pour vérifier que le lecteur est toujours monté. 

- Si le fichier /etc/fstab ne monte pas correctement le lecteur avant l’activation du chiffrement, Azure Disk Encryption ne pourra pas procéder au montage du lecteur.
- Le processus Azure Disk Encryption déplace les informations du fichier /etc/fstab vers son propre fichier de configuration dans le cadre de l’opération de chiffrement. Par conséquent, ne soyez pas surpris que l’entrée ne figure plus dans le fichier /etc/fstab après le chiffrement du lecteur de données.
- Avant de démarrer le chiffrement, veillez à arrêter tous les services et processus pouvant écrire sur des disques de données montés, et à les désactiver de sorte qu’ils ne redémarrent pas automatiquement après un redémarrage. Ils peuvent maintenir des fichiers ouverts sur ces partitions, empêchant ainsi la procédure de chiffrement de les remonter, d’où une défaillance du chiffrement. 
- Après le redémarrage, le montage des disques nouvellement chiffrés par le processus Azure Disk Encryption nécessite un certain temps. Les disques ne sont donc pas disponibles juste après un redémarrage. Le processus a besoin de temps pour démarrer, déverrouiller, puis monter les lecteurs chiffrés avant que ces derniers ne deviennent accessibles à d’autres processus. Ce processus peut prendre plusieurs minutes après le redémarrage en fonction des caractéristiques du système.

Voici un exemple des commandes utilisées pour monter les disques de données et créer les entrées /etc/fstab nécessaires :

```bash
UUID0="$(blkid -s UUID -o value /dev/sda1)"
UUID1="$(blkid -s UUID -o value /dev/sda2)"
mkdir /data0
mkdir /data1
echo "UUID=$UUID0 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "UUID=$UUID1 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```
## <a name="networking-requirements"></a>Configuration requise du réseau

Pour l’activation de la fonctionnalité Azure Disk Encryption, les machines virtuelles Linux doivent répondre aux exigences de configuration de point de terminaison de réseau suivantes :
  - Pour obtenir un jeton afin de se connecter à votre coffre de clés, la machine virtuelle Linux doit être capable de se connecter au point de terminaison Azure Active Directory \[login.microsoftonline.com\].
  - Pour écrire les clés de chiffrement dans votre coffre de clés, la machine virtuelle Linux doit être capable de se connecter au point de terminaison Key Vault.
  - La machine virtuelle Linux doit être capable de se connecter au point de terminaison de stockage Azure qui héberge le référentiel d’extensions Azure et à un compte de stockage Azure qui héberge les fichiers de disque dur virtuel.
  -  Si votre stratégie de sécurité limite l’accès à Internet à partir des machines virtuelles Azure, vous pouvez résoudre l’URI ci-dessus et configurer une règle spécifique pour autoriser les connexions sortantes vers les adresses IP. Pour plus d’informations, consultez l’article [Azure Key Vault derrière un pare-feu](../../key-vault/general/access-behind-firewall.md).  

## <a name="encryption-key-storage-requirements"></a>Exigences liées au stockage des clés de chiffrement  

Azure Disk Encryption exige Azure Key Vault pour contrôler et gérer les clés et les secrets de chiffrement de disque. Votre coffre de clés et vos machines virtuelles doivent se trouver dans la même région et le même abonnement Azure.

Pour plus d’informations, consultez [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="terminology"></a>Terminologie
Le tableau suivant définit certains termes courants utilisés dans la documentation d’Azure Disk Encryption :

| Terminologie | Définition |
| --- | --- |
| Azure Key Vault | Key Vault est un service de gestion de clés de chiffrement basé sur des modules de sécurité matériels validés FIPS (Federal Information Processing Standard). Ces normes permettent de protéger vos clés de chiffrement et vos secrets sensibles. Pour plus d’informations, consultez la documentation [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) et [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md). |
| Azure CLI | [Azure CLI](/cli/azure/install-azure-cli) est optimisé pour gérer et administrer des ressources Azure en ligne de commande.|
| DM-Crypt |[DM-Crypt](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMCrypt) est le sous-système de chiffrement de disque transparent Linux utilisé pour activer le chiffrement de disque sur les machines virtuelles Linux. |
| Clé de chiffrement principale (KEK) | Clé asymétrique (RSA 2048) que vous pouvez utiliser pour protéger ou encapsuler le secret. Vous pouvez fournir une clé protégée par un module de sécurité matériel ou une clé protégée par logiciel. Pour plus d’informations, consultez la documentation [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) et [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md). |
| Applets de commande PowerShell | Pour plus d’informations, voir [Cmdlets Azure PowerShell](/powershell/azure/). |


## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Créer et chiffrer une machine virtuelle Linux avec Azure CLI](disk-encryption-cli-quickstart.md)
- [Démarrage rapide : Créer et chiffrer une machine virtuelle Linux avec Azure PowerShell](disk-encryption-powershell-quickstart.md)
- [Scénarios Azure Disk Encryption sur les machines virtuelles Linux](disk-encryption-linux.md)
- [Script d’interface CLI des prérequis Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started)
- [Script PowerShell des prérequis Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption](disk-encryption-key-vault.md)
