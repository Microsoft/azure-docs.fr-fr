---
title: Optimiser le débit du réseau des machines virtuelles | Microsoft Docs
description: Optimisez le débit du réseau pour les machines virtuelles Microsoft Azure Windows et Linux, notamment les distributions majeures telles que Ubuntu, CentOS et Red Hat.
services: virtual-network
documentationcenter: na
author: steveesp
manager: Gerald DeGrace
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/06/2020
ms.author: steveesp
ms.openlocfilehash: a9db2bcc0b44dfb6146517de8a139f34cd8584af
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654453"
---
# <a name="optimize-network-throughput-for-azure-virtual-machines"></a>Optimiser le débit du réseau des machines virtuelles Azure

Les machines virtuelles Azure disposent de paramètres réseau par défaut qui peuvent être davantage optimisés pour le débit du réseau. Cet article décrit comment optimiser le débit du réseau pour les machines virtuelles Microsoft Azure Windows et Linux, notamment les distributions majeures telles que Ubuntu, CentOS et Red Hat.

## <a name="windows-vm"></a>Machine virtuelle Windows

Si votre machine virtuelle Windows prend en charge la [mise en réseau accélérée](create-vm-accelerated-networking-powershell.md), l’activation de cette fonctionnalité constitue la configuration optimale pour le débit. Pour toutes les autres machines virtuelles Windows, l’utilisation de la mise à l’échelle côté réception (RSS) peut permettre d’atteindre un débit maximal supérieur à celui d’une machine virtuelle sans RSS. La mise à l’échelle côté réception (RSS) peut être désactivée par défaut sur une machine virtuelle Windows. Pour déterminer si la mise à l’échelle côté réception (RSS) est activée et, si elle ne l’est pas, l’activer, effectuez les étapes suivantes :

1. Utilisez la commande PowerShell `Get-NetAdapterRss` pour savoir si la mise à l’échelle côté réception (RSS) est activée sur une carte réseau. Dans l’exemple de sortie suivant retourné par `Get-NetAdapterRss`, la mise à l’échelle côté réception (RSS) n’est pas activée.

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                 : False
    ```
2. Pour activer la mise à l’échelle côté réception (RSS), entrez la commande suivante :

    ```powershell
    Get-NetAdapter | % {Enable-NetAdapterRss -Name $_.Name}
    ```
    La commande précédente n’a pas de sortie. La commande a modifié les paramètres de la carte réseau, entraînant une perte temporaire de la connectivité pendant environ une minute. Une boîte de dialogue de reconnexion s’affiche lors de la perte de connectivité. En général, la connectivité est rétablie après la troisième tentative.
3. Vérifiez que la mise à l’échelle côté réception (RSS) est activée sur la machine virtuelle en entrant de nouveau la commande `Get-NetAdapterRss`. Si l’opération réussit, l’exemple de sortie suivant est retourné :

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                  : True
    ```

## <a name="linux-vm"></a>Machine virtuelle Linux

La mise à l’échelle côté réception (RSS) est toujours activée par défaut sur une machine virtuelle Azure Linux. Les noyaux Linux publiés depuis octobre 2017 incluent de nouvelles options d’optimisation du réseau qui permettent à une machine virtuelle Linux d’obtenir un débit réseau plus élevé.

### <a name="ubuntu-for-new-deployments"></a>Ubuntu pour les nouveaux déploiements

Le noyau Ubuntu Azure est le plus optimisé pour les performances réseau sur Azure. Pour obtenir les dernières optimisations, installez d’abord la version la plus récente de 18.04-LTS, comme suit :

```json
"Publisher": "Canonical",
"Offer": "UbuntuServer",
"Sku": "18.04-LTS",
"Version": "latest"
```

Une fois la création terminée, entrez les commandes suivantes pour obtenir les mises à jour les plus récentes. Ces étapes fonctionnent aussi pour les machines virtuelles qui s’exécutent actuellement sur le noyau Ubuntu Azure.

```bash
#run as root or preface with sudo
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

L’ensemble de commandes optionnel suivant peut être utile pour les déploiements Ubuntu existants qui disposent déjà du noyau Azure mais qui ont rencontré des erreurs lors de mises à jour.

```bash
#optional steps may be helpful in existing deployments with the Azure kernel
#run as root or preface with sudo
apt-get -f install
apt-get --fix-missing install
apt-get clean
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

#### <a name="ubuntu-azure-kernel-upgrade-for-existing-vms"></a>Mise à niveau du noyau Ubuntu Azure pour les machines virtuelles existantes

Des performances significatives en termes de débit peuvent être atteintes en mettant à jour vers le noyau Azure Linux proposé. Pour vérifier si vous avez ce noyau, vérifiez la version de votre noyau. Elle doit être identique ou ultérieure à l’exemple.

```bash
#Azure kernel name ends with "-azure"
uname -r

#sample output on Azure kernel:
#4.13.0-1007-azure
```

Si votre machine virtuelle ne dispose pas du noyau Azure, le numéro de version commence en général par « 4.4 ». Si la machine virtuelle n’a pas le noyau Azure, exécutez les commandes suivantes à la racine :

```bash
#run as root or preface with sudo
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
apt-get install "linux-azure"
reboot
```

### <a name="centos"></a>CentOS

Pour bénéficier des dernières optimisations, il est préférable de créer une machine virtuelle avec la version la plus récente prise en charge en spécifiant les paramètres suivants :

```json
"Publisher": "OpenLogic",
"Offer": "CentOS",
"Sku": "7.7",
"Version": "latest"
```

Il peut être bénéfique pour les machines virtuelles nouvelles ou existantes d’installer la dernière version de Linux Integration Services (LIS). L’optimisation du débit est incluse dans les LIS, à partir de la version 4.2.2-2, bien que les versions ultérieures contiennent d’autres améliorations. Entrez les commandes suivantes pour installer la dernière version de LIS :

```bash
sudo yum update
sudo reboot
sudo yum install microsoft-hyper-v
```

### <a name="red-hat"></a>Red Hat

Pour bénéficier des optimisations, il est préférable de créer une machine virtuelle avec la version la plus récente prise en charge en spécifiant les paramètres suivants :

```json
"Publisher": "RedHat"
"Offer": "RHEL"
"Sku": "7-RAW"
"Version": "latest"
```

Il peut être bénéfique pour les machines virtuelles nouvelles ou existantes d’installer la dernière version de Linux Integration Services (LIS). L’optimisation du débit est incluse dans les LIS, à partir de la version 4.2. Entrez les commandes suivantes pour télécharger et installer les LIS :

```bash
wget https://aka.ms/lis
tar xvf lis
cd LISISO
sudo ./install.sh #or upgrade.sh if prior LIS was previously installed
```

Apprenez-en plus sur les Services d’intégration Linux version 4.2 pour Hyper-V en consultant la [page de téléchargement](https://www.microsoft.com/download/details.aspx?id=55106).

## <a name="next-steps"></a>Étapes suivantes
* Déployez des machines virtuelles proches les unes des autres pour une faible latence avec un [groupe de placement de proximité](../virtual-machines/windows/co-location.md)
* Découvrez le résultat optimisé avec le [Test de bande passante/débit de machine virtuelle Azure](virtual-network-bandwidth-testing.md) pour votre scénario.
* Découvrez de quelle façon [la bande passante est allouée aux machines virtuelles](virtual-machine-network-throughput.md)
* En savoir plus avec le [FAQ sur les réseaux virtuels Azure](virtual-networks-faq.md)
