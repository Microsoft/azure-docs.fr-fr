---
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 04/28/2019
ms.author: raynew
ms.openlocfilehash: 088cd5447b1f96dbf172b5918c29e4f3293289a6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008471"
---
1. Établissez une connexion Bureau à distance à la machine exécutant le serveur de traitement. 
2. Exécutez cspsconfigtool.exe pour démarrer l’outil de configuration du serveur de traitement Azure Site Recovery.
    - L’outil est lancé automatiquement la première fois que vous vous connectez au serveur de traitement.
    - S’il ne s’ouvre pas automatiquement, cliquez sur son raccourci sur le bureau.

3. Dans **FQDN ou IP du serveur de Configuration**, spécifiez le nom ou l’adresse IP du serveur de configuration avec laquelle inscrire le serveur de traitement.
4. Vérifiez que **Port du serveur de Configuration** est défini sur 443. Il s’agit du port sur lequel le serveur de configuration écoute les requêtes.
5. Dans **Phrase secrète de connexion**, indiquez la phrase secrète spécifiée lorsque vous configurez le serveur de configuration. Pour rechercher la phrase secrète :
    -  Sur le serveur de configuration, accédez au dossier d’installation de Site Recovery * *\home\svssystems\bin\** :
    ```
    cd %ProgramData%\ASR\home\svsystems\bin
    ```
    - Exécutez la commande ci-dessous pour imprimer la phrase secrète actuelle :
    ```
    genpassphrase.exe -n
    ```

6. Dans **Port de transfert de données**, laissez la valeur par défaut, sauf si vous avez indiqué un port personnalisé.

7. Cliquez sur **Enregistrer** pour enregistrer les paramètres et le serveur de traitement.

    
    ![Enregistrer le serveur de traitement](./media/site-recovery-vmware-register-process-server/register-ps.png)
