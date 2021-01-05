---
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: include
ms.date: 04/28/2019
ms.author: ramamill
ms.openlocfilehash: bff9457220efbb6a6b318550a5b2dd9af4ae230f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019182"
---
Suivez les étapes pour votre situation spécifique.

### <a name="unregister-a-connected-process-server"></a>Annuler l’inscription d’un serveur de traitement connecté

1. Établissez une connexion à distance avec le serveur de traitement en tant qu’Administrateur.
2. Dans le **Panneau de configuration**, ouvrez **Programmes > Désinstaller un programme**.
3. Désinstallez le programme **Service Mobilité/Serveur cible maître Microsoft Azure Site Recovery**.
4. Désinstallez le programme **Serveur de configuration/traitement Microsoft Azure Site Recovery**.
5. Une fois les programmes des étapes 3 et 4 désinstallés, vous pouvez désinstaller **Dépendances du serveur de configuration/traitement Microsoft Azure Site Recovery**.

### <a name="unregister-a-disconnected-process-server"></a>Annuler l’inscription d’un serveur de traitement déconnecté

N’utilisez ces étapes que s’il n’existe aucun autre moyen de réactiver l’ordinateur sur lequel est installé le serveur de traitement.

1. Connectez-vous au serveur de configuration en tant qu’administrateur.
2. Ouvrez une invite de commande d’administration et accédez à `%ProgramData%\ASR\home\svsystems\bin`.
3. Exécutez cette commande pour obtenir une liste d’un ou plusieurs serveurs de traitement.

    ```
    perl Unregister-ASRComponent.pl -IPAddress <IP_of_Process_Server> -Component PS
    ```
    - S. Numéro : le numéro de série du serveur de traitement.
    - Adresse IP/Nom : L’adresse IP et le nom de la machine exécutant le serveur de traitement.
    - Pulsation : Dernière pulsation de la machine du serveur de traitement.
    ![La capture d’écran montre un affichage en texte clair des informations sur vos serveurs de traitement et le texte Choisissez l’un des serveurs ci-dessus à désinscrire.(media/site-recovery-vmware-unregister-process-server/Unregister-cmd.PNG)

4. Spécifiez le numéro de série du serveur de traitement dont vous souhaitez annuler l’inscription.
5. L’annulation de l’enregistrement d’un serveur de traitement supprime tous les détails du système et affiche le message : **Désinscription de nom-serveur réussie> (adresse-IP-serveur)**

