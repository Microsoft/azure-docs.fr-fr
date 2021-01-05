---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/10/2019
ms.author: danlep
ms.openlocfilehash: 35f506235f698fbcf42308e6f0b0f400e925df29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68361265"
---
### <a name="bastion-tier"></a>Niveau bastion

L’hôte bastion est un composant facultatif que vous pouvez utiliser comme serveur de renvoi pour accéder aux instances de l’application et de la base de données. Une adresse IP publique peut être affectée à la machine virtuelle hôte bastion, bien qu’il soit recommandé de configurer une connexion ExpressRoute ou un VPN de site à site avec votre réseau local pour un accès sécurisé. En outre, seul le protocole SSH (port 22, Linux) ou RDP (port 3389, Windows Server) doit être ouvert au trafic entrant. Pour la haute disponibilité, déployez un hôte bastion dans deux zones de disponibilité ou dans un seul groupe de disponibilité.

Vous pouvez également activer le transfert de l’agent SSH sur vos machines virtuelles, ce qui vous permet d’accéder à d’autres machines virtuelles dans le réseau virtuel en transférant les informations d’identification de votre hôte bastion. Ou utilisez le tunneling SSH pour accéder à d’autres instances.

Voici un exemple de transfert d’agent :

```
ssh -A -t user@BASTION_SERVER_IP ssh -A root@TARGET_SERVER_IP`
```

Cette commande se connecte au bastion, puis réexécute immédiatement `ssh`. Vous recevez donc un terminal sur l’instance cible. Vous devrez peut-être spécifier un autre utilisateur que la racine sur l’instance cible si votre cluster est configuré différemment. L'argument `-A` transfère la connexion de l’agent de telle sorte que votre clé privée sur votre ordinateur local est utilisée automatiquement. Notez que le transfert de l’agent est une chaîne. Par conséquent, la deuxième commande `ssh` inclut également `-A` afin que toutes les connexions SSH ultérieures lancées à partir de l’instance cible utilisent également votre clé privée locale.