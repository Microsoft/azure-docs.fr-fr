---
title: Se connecter à une machine virtuelle Windows avec Azure Bastion
description: Dans cet article, découvrez comment vous connecter à une machine virtuelle Azure exécutant Windows à l’aide d’Azure Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 10/21/2020
ms.author: cherylmc
ms.openlocfilehash: 708bd1f61da2f3973333f8e68cabdceee0717bee
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521530"
---
# <a name="connect-to-a-windows-virtual-machine-using-azure-bastion"></a>Connexion à une machine virtuelle Windows à l’aide d’Azure Bastion

En utilisant Azure Bastion, vous pouvez vous connecter de manière sécurisée et fluide à vos machines virtuelles via SSL directement dans le portail Azure. Quand vous utilisez Azure Bastion, vos machines virtuelles n’ont pas besoin de client, d’agent ni de logiciel supplémentaire. Cet article vous explique comment vous connecter à vos machines virtuelles Windows. Pour plus d’informations sur la connexion à une machine virtuelle Linux, consultez [Connexion à une machine virtuelle Linux](bastion-connect-vm-ssh.md).

Azure Bastion fournit une connectivité sécurisée à toutes les machines virtuelles du réseau virtuel dans lequel il est provisionné. Azure Bastion protège vos machines virtuelles contre l’exposition des ports RDP/SSH au monde extérieur, tout en fournissant un accès sécurisé à l’aide de RDP/SSH. Pour plus d’informations, consultez la page [Présentation d’Azure Bastion](bastion-overview.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous respectez les critères suivants :

* Un réseau virtuel sur lequel l’hôte Bastion est déjà installé.

   Assurez-vous d’avoir configuré un hôte Azure Bastion pour le réseau virtuel dans lequel se trouve la machine virtuelle. Une fois que le service Bastion est provisionné et déployé dans votre réseau virtuel, vous pouvez l’utiliser pour vous connecter à toutes les machines virtuelles dans le réseau virtuel. Pour configurer un hôte Azure Bastion, consultez [Créer un hôte Bastion](tutorial-create-host-portal.md#createhost).
* Une machine virtuelle Windows hébergée sur le réseau virtuel.
* Les rôles requis suivants :
  * Rôle de lecteur sur la machine virtuelle.
  * Rôle de lecteur sur la carte réseau avec adresse IP privée de la machine virtuelle.
  * Rôle de lecteur sur la ressource Azure Bastion.
* Ports : Pour vous connecter à la machine virtuelle Windows, les ports suivants doivent être ouverts sur votre machine virtuelle Windows :
  * Ports d’entrée : RDP (3389)

## <a name="connect"></a><a name="rdp"></a>Connexion

[!INCLUDE [Connect to a Windows VM](../../includes/bastion-vm-rdp.md)]
 
## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).