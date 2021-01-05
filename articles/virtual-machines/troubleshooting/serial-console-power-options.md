---
title: Options d’alimentation de la console série Azure | Microsoft Docs
description: Options d’alimentation des machines virtuelles disponibles dans la console série Azure
services: virtual-machines
documentationcenter: ''
author: asinn826
manager: borisb
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 8/6/2019
ms.author: alsin
ms.openlocfilehash: 11c2549d7282bae5654ede1ac34d63a43cd0f059
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75451196"
---
# <a name="power-options-available-from-the-azure-serial-console"></a>Options d’alimentation disponibles dans la console série Azure

La console série Azure fournit plusieurs outils puissants pour la gestion de l’alimentation sur votre machine virtuelle ou groupe de machines virtuelles identiques. Ces options de gestion de l’alimentation peuvent être déroutantes pour certains, ce document constitue donc une vue d’ensemble de chaque outil et du cas d’utilisation prévu.

Fonctionnalité de la console série | Description | Cas d’usage
:----------------------|:------------|:---------
Redémarrer la machine virtuelle | Redémarrage normal de votre machine virtuelle ou votre instance de groupe de machines virtuelles identiques. Cette opération est identique à l’appel de la fonctionnalité de redémarrage disponible dans la page Vue d’ensemble. | Dans la plupart des cas, cette option doit être votre premier outil de tentative de redémarrage de votre machine virtuelle. La connexion à votre console série subira une courte interruption et reprendra automatiquement dès que la machine virtuelle aura redémarré.
Réinitialiser une machine virtuelle | Redémarrage forcé de votre machine virtuelle ou de votre groupe de machines virtuelles identiques sur la plateforme Azure. | Cette option permet de redémarrer immédiatement votre système d’exploitation, quel que soit son état actuel. Étant donné que cette opération n’est pas normale, il existe un risque de perte ou d’endommagement des données. Il n’y a aucune interruption de la connexion à la console série, ce qui peut être utile pour envoyer des commandes tôt au moment du démarrage (par exemple, pour accéder à GRUB sur une machine virtuelle Linux ou au mode sans échec sur une machine virtuelle Windows).
SysRq - Reboot (b) | Demande système pour forcer un redémarrage de l’invité. | Cette fonctionnalité s’applique uniquement aux systèmes d’exploitation Linux et requiert [l’activation de SysRq](./serial-console-nmi-sysrq.md#system-request-sysrq) dans le système d’exploitation. Si le système d’exploitation est correctement configuré pour SysRq, cette commande entraîne le redémarrage du système d’exploitation.
Interruption non masquable (NMI) | Commande d’interruption, qui sera remise au système d’exploitation | Cette opération est disponible pour les machines virtuelles [Windows](./serial-console-windows.md#use-the-serial-console-for-nmi-calls) et [Linux](./serial-console-nmi-sysrq.md#non-maskable-interrupt-nmi) et requiert l’activation de NMI. L’envoi d’une NMI entraîne généralement le blocage de votre système d’exploitation. Vous pouvez configurer votre système d’exploitation pour créer un fichier de vidage, puis redémarrer lors de la réception de la NMI, ce qui peut être utile lors du débogage de bas niveau.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [console série Azure pour les machines virtuelles Linux](./serial-console-linux.md)
* En savoir plus sur la [console série Azure pour les machines virtuelles Windows](./serial-console-windows.md)