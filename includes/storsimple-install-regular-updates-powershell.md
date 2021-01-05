---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: dc50f94ae9b207961a71480c2fc172e88db79cf4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177096"
---
#### <a name="to-install-regular-updates-via-windows-powershell-for-storsimple"></a>Pour installer des mises à jour périodiques via Windows PowerShell pour StorSimple
1. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**. Tapez le mot de passe. Le mot de passe par défaut est *Password1*. 
2. À l’invite de commandes, tapez :
   
     `Get-HcsUpdateAvailability`
   
    Le système vous indique si des mises à jour sont disponibles et si elles risquent ou non de provoquer une interruption de service.
3. À l’invite de commandes, tapez :
   
     `Start-HcsUpdate`
   
    La mise à jour démarre.

> [!IMPORTANT]
> * Cette commande s’applique uniquement aux mises à jour périodiques. Vous exécutez cette commande sur un seul contrôleur, mais les deux contrôleurs sont mis à jour. 
> * Il est possible qu’un basculement de contrôleur se produise pendant la mise à jour. Celui-ci n’a aucune incidence sur la disponibilité ou le fonctionnement du système.
> 
> 

