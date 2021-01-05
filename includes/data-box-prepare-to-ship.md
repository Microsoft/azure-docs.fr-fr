---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 07/01/2020
ms.author: alkohli
ms.openlocfilehash: a78bce9f6e714aa5564791f6bb63996554beabf1
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93375858"
---
La dernière étape consiste à préparer l’appareil pour l’expédition. Pour cela, tous les partages de l’appareil sont mis hors connexion. Une fois que ce processus aura débuté, vous ne pourrez plus accéder aux partages.

> [!IMPORTANT]
> La préparation de l’expédition est nécessaire, car elle signale les données qui ne sont pas conformes aux conventions de nommage Azure. Cette étape empêche les échecs de chargement de données liés à la non-conformité de celles-ci.

1. Accédez à la page **Prepare to ship** (Préparer l’expédition) et cliquez sur **Démarrer la préparation**. Par défaut, les sommes de contrôle sont calculées pendant la copie des données. La préparation de l’expédition procède au calcul des sommes de contrôle et crée la liste des fichiers ( *fichiers -BOM* ). Le calcul des sommes de contrôle peut prendre plusieurs heures, selon la taille de vos données. 
   
    ![Préparer l’expédition 1](media/data-box-prepare-to-ship/prepare-to-ship1.png)

    Si pour une raison ou une autre, vous souhaitez arrêter la préparation de l’appareil, cliquez sur **Arrêter la préparation**. Vous pourrez reprendre la préparation à l’expédition plus tard.
        
    ![Préparer l’expédition 2](media/data-box-prepare-to-ship/prepare-to-ship2.png)
    
2. La préparation à l’expédition démarre et les partages de l’appareil sont mis hors ligne. <!--You see a reminder to download the shipping label once the device is ready.--> Une fois la préparation terminée, l’appareil passe à l’état *Ready to ship* (Prêt pour expédition), puis est verrouillé.
        
    ![Préparer l’expédition 3](media/data-box-prepare-to-ship/prepare-to-ship3.png)

    Si vous souhaitez copier davantage de données sur l’appareil, vous pouvez déverrouiller l’appareil, copier d’autres données, puis exécuter une nouvelle préparation à l’expédition.

    S’il existe des erreurs dans cette étape, l’état est mis à jour en *Analyse terminée avec des erreurs*. Déverrouillez l’appareil, accédez à la page **Connexion et copie** , téléchargez la liste des problèmes, puis résolvez les erreurs.

    ![Préparer l’expédition 4](media/data-box-prepare-to-ship/prepare-to-ship4.png)

    Une fois les erreurs résolues, exécutez **Préparer l’expédition**.

4. Une fois la préparation de l’expédition terminée avec succès (sans erreur), procédez comme suit :

    1. Prenez note du numéro de référence de fin d’exécution. En fonction du pays dans lequel vous vous trouvez, ce numéro peut être nécessaire pour différentes opérations.
    2. Téléchargez la liste des fichiers (également appelés manifeste) copiés dans le cadre de ce processus. Vous pourrez utiliser cette liste ultérieurement pour vérifier les fichiers téléchargés vers Azure. Pour plus d’informations, consultez [Inspecter les fichiers BOM pendant la préparation de l’expédition](../articles/databox/data-box-logs.md#inspect-bom-during-prepare-to-ship).
        
        ![Préparer l’expédition 5](media/data-box-prepare-to-ship/prepare-to-ship5.png)
    3. Sélectionnez et téléchargez les instructions d’expédition pour l’appareil. Les instructions d’expédition diffèrent selon le pays dans lequel vous vous trouvez.
    4. Si l’écran E-ink n’affiche pas l’étiquette d’expédition, vous pouvez télécharger l’étiquette d’expédition inverse ici. 

5. Arrêtez l’appareil. Accédez à la page **Shut down or restart** (Arrêter ou redémarrer) et cliquez sur **Arrêter**. Lorsque vous êtes invité à confirmer l’opération, cliquez sur **OK** pour continuer.

6. Débranchez les câbles. L’étape suivante consiste à expédier l’appareil à Microsoft.
