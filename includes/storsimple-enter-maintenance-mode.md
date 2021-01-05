---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: d35f0ef783a2c48f8211657bc8829635c19495aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177090"
---
#### <a name="to-enter-maintenance-mode"></a>Pour passer en mode Maintenance
1. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**.
2. Tapez le mot de passe. Le mot de passe par défaut est **Password1**.
3. À l'invite de commandes, tapez
   
     `Enter-HcsMaintenanceMode`
4. Un message d’avertissement s’affiche pour vous indiquer que le mode Maintenance va perturber toutes les demandes d’E/S et annuler la connexion au portail Azure Classic. Vous êtes invité à confirmer l’opération. Tapez **O** pour passer en mode Maintenance.
   
    Les deux contrôleurs redémarrent. Une fois le redémarrage terminé, un autre message s’affiche pour indiquer que l’appareil est en mode Maintenance.

