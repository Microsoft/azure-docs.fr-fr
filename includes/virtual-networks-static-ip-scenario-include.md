---
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 02/14/2020
ms.author: genli
ms.openlocfilehash: 280943bd965c4799ce294321129d1088be9c0caf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015918"
---
## <a name="scenario"></a>Scénario

Pour mieux illustrer la configuration d’une adresse IP statique pour une machine virtuelle, ce document utilise le scénario suivant :

![Scénario de réseau virtuel : Sous-réseaux frontaux et principaux, avec une adresse IP statique pour le sous-réseau frontal](./media/virtual-networks-static-ip-scenario-include/static-ip-scenario.png)

Dans ce scénario, vous créez une machine virtuelle nommée *DNS01* dans le sous-réseau *FrontEnd*, et la configurez pour utiliser l’adresse IP statique *192.168.1.101*.
