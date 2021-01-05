---
title: Tutoriel - Copier des données via SMB sur Azure Data Box Heavy | Microsoft Docs
description: Dans ce tutoriel, découvrez comment établir une connexion à votre ordinateur hôte et copier des données de cet ordinateur vers Azure Data Box Heavy à l’aide de SMB avec l’interface utilisateur web locale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 08/29/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 77dc64d9660f9a0bf66559c4a5a976362cf1acd0
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91951636"
---
::: zone target = "docs"

# <a name="tutorial-copy-data-to-azure-data-box-heavy-via-smb"></a>Tutoriel : Copier des données sur Azure Data Box Heavy par le biais de SMB

::: zone-end

::: zone target = "chromeless"

## <a name="copy-data-to-azure-data-box-heavy"></a>Copier des données sur Azure Data Box Heavy

::: zone-end

::: zone target = "docs"

Ce tutoriel explique comment vous connecter à votre ordinateur hôte et copier des données à partir de cet ordinateur à l’aide de l’interface utilisateur web locale.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Se connecter à Data Box Heavy
> * Copier des données sur Data Box Heavy

::: zone-end

::: zone target = "chromeless"

Vous pouvez copier des données de votre serveur source vers votre Data Box en utilisant SMB, NFS, REST ou le service de copie de données. Vous pouvez également effectuer la copie sur des disques managés.

Dans tous les cas, vérifiez que les noms des partages et des dossiers ainsi que la taille des données respectent les instructions décrites dans la section [Limites des services Stockage Azure et Data Box Heavy](data-box-heavy-limits.md).

::: zone-end

::: zone target = "docs"

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le [Tutoriel : Configurer Azure Data Box Heavy](data-box-deploy-set-up.md).
2. Vous avez reçu votre Data Box Heavy et l’état de la commande dans le portail est **Livré**.
3. Vous disposez d’un ordinateur hôte contenant les données que vous souhaitez copier sur Data Box Heavy. Votre ordinateur hôte doit
    - Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
    - Connectez-vous à un réseau haut débit. Pour obtenir des vitesses de copie plus rapides, il est possible d’utiliser deux connexions 40 GbE (une par nœud) en parallèle. Si aucune connexion 40-GbE n’est disponible, nous vous recommandons d’avoir au moins deux connexions 10 GbE (une par nœud).
   

## <a name="connect-to-data-box-heavy-shares"></a>Se connecter à des partages Data Box Heavy

En fonction du compte de stockage sélectionné, Data Box Heavy crée jusqu’à :
- Trois partages pour chaque compte de stockage associé pour GPv1 et GPv2.
- Un partage pour le stockage Premium.
- Un partage pour le compte de stockage Blob.

Ces partages sont créés sur les deux nœuds de l’appareil.

Sous les partages d’objet blob de blocs et d’objet blob de pages :
- Les entités de premier niveau sont des conteneurs.
- Les entités de second niveau sont des objets blob.

Sous les partages Azure Files :
- Les entités de premier niveau sont des partages.
- Les entités de second niveau sont des fichiers.

Le tableau suivant montre le chemin UNC des partages sur votre Data Box Heavy et l’URL du chemin du Stockage Azure où les données sont chargées. La dernière URL de chemin de Stockage Azure peut être dérivée à partir du chemin de partage UNC.
 
| Stockage           | Chemin UNC                                                                       |
|-------------------|--------------------------------------------------------------------------------|
| Objets blob de blocs Azure | <li>Chemin UNC aux partages : `\\<DeviceIPAddress>\<StorageAccountName_BlockBlob>\<ContainerName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Objets blob de pages Azure  | <li>Chemin UNC aux partages : `\\<DeviceIPAddres>\<StorageAccountName_PageBlob>\<ContainerName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Chemin UNC aux partages : `\\<DeviceIPAddres>\<StorageAccountName_AzFile>\<ShareName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |      

Les étapes permettant de se connecter à l’aide d’un client Windows ou Linux sont différentes.

> [!NOTE]
> Suivez les mêmes étapes pour vous connecter aux deux nœuds de l’appareil en parallèle.

### <a name="connect-on-a-windows-system"></a>Se connecter sur un système Windows

Si vous utilisez un ordinateur hôte Windows Server, effectuez les étapes suivantes pour vous connecter au Data Box Heavy.

1. La première étape consiste à vous authentifier et à démarrer une session. Accédez à la page **Connect and copy** (Connexion et copie). Cliquez sur **Get credentials** (Obtenir les informations d’identification) pour obtenir les informations d’identification requises pour accéder aux partages associés à votre compte de stockage.

    ![Obtenir les informations d’identification de partage](media/data-box-heavy-deploy-copy-data/get-share-credentials-1.png)

2. Dans la boîte de dialogue Access share and copy data (Accéder au partage et copier les données), copiez les valeurs **Nom d’utilisateur** et **Mot de passe** correspondant au partage. Cliquez sur **OK**.
    
    ![Obtenir les informations d’identification du partage 2](media/data-box-heavy-deploy-copy-data/get-share-credentials-2.png)

3. Pour accéder aux partages associés à votre compte de stockage (*databoxe2etest* dans l’exemple suivant) à partir de votre ordinateur hôte, ouvrez une fenêtre de commande. À l’invite de commandes, tapez :

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    En fonction du format de vos données, les chemins de partage sont les suivants :
    - Objet blob de blocs Azure - `\\10.100.10.100\databoxe2etest_BlockBlob`
    - Objet blob de pages Azure - `\\10.100.10.100\databoxe2etest_PageBlob`
    - Azure Files - `\\10.100.10.100\databoxe2etest_AzFile`
    
4. Lorsque vous y êtes invité, entrez le mot de passe du partage. L’exemple suivant illustre la connexion à un partage à l’aide de la commande précédente.

    ```
    C:\Users\Databoxuser>net use \\10.100.10.100\databoxe2etest_BlockBlob /u:databoxe2etest
    Enter the password for 'databoxe2etest' to connect to '10.100.10.100':
    The command completed successfully.
    ```

4. Appuyez sur Windows + R. Dans la fenêtre **Exécuter**, spécifiez `\\<device IP address>`. Cliquez sur **OK** pour ouvrir l’Explorateur de fichiers.
    
    ![Se connecter au partage via l’Explorateur de fichiers](media/data-box-heavy-deploy-copy-data/connect-shares-file-explorer-1.png)

    À présent, vous devriez voir les partages sous forme de dossiers.
    
    ![Se connecter au partage à l’aide de l’Explorateur de fichiers 2](media/data-box-heavy-deploy-copy-data/connect-shares-file-explorer-2.png)

    **Toujours créer un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copier les fichiers dans ce dossier**. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.
    
### <a name="connect-on-a-linux-system"></a>Se connecter sur un système Linux

Si vous utilisez un client Linux, utilisez la commande suivante pour monter le partage SMB.

```
sudo mount -t nfs -o vers=2.1 10.126.76.172:/databoxe2etest_BlockBlob /home/databoxubuntuhost/databox
```

Le paramètre `vers` est la version de SMB que votre hôte Linux prend en charge. Utilisez la version appropriée dans la commande ci-dessus.

Pour connaître les versions de SMB que Data Box Heavy prend en charge, consultez [Systèmes de fichiers pris en charge pour les clients Linux](data-box-heavy-system-requirements.md#supported-file-systems-for-linux-clients).

## <a name="copy-data-to-data-box-heavy"></a>Copier des données sur Data Box Heavy

Une fois que vous êtes connecté aux partages Data Box Heavy, l’étape suivante consiste à copier des données.

### <a name="copy-considerations"></a>Considérations relatives à la copie

Avant de commencer la copie des données, passez en revue les considérations suivantes :

- Vérifiez que les données sont copiées vers des partages compatibles avec le format des données. Par exemple, les données d’objet blob de blocs doivent être copiées dans le partage des objets blob de blocs. Copiez les disques durs virtuels dans un objet blob de pages.

    Si le format des données ne correspond pas au type de partage, les données ne pourront pas être chargées dans Azure.
-  Quand vous copiez des données, vérifiez que leur taille est conforme aux limites de taille spécifiées dans l’article [Limitations relatives au Stockage Azure et à Data Box Heavy](data-box-heavy-limits.md).
- Si les données qui sont en cours de chargement par Data Box Heavy sont chargées simultanément par d’autres applications en dehors de Data Box Heavy, cela peut entraîner l’échec du chargement ou une altération des données.
- Nous vous recommandons :
    - N’utilisez pas SMB et NFS en même temps.
    - Copiez les mêmes données à la même destination finale sur Azure.
     
  En effet, le résultat final ne pourrait pas être déterminé.
- Créez toujours un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copiez les fichiers dans ce dossier. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.

Après vous être connecté au partage SMB, commencez la copie des données.

1. Vous pouvez utiliser n’importe quel outil de copie de fichier SMB, comme Robocopy, pour copier vos données. Plusieurs travaux de copie peuvent être lancés simultanément à l’aide de Robocopy. Utilisez la commande suivante :
    
    ```
    robocopy <Source> <Target> * /e /r:3 /w:60 /is /nfl /ndl /np /MT:32 or 64 /fft /Log+:<LogFile>
    ```
    Les attributs sont décrits dans le tableau suivant.
    
    |Attribut  |Description  |
    |---------|---------|
    |/e      |Copie les sous-répertoires, y compris les répertoires vides.         |
    |/r:     |Spécifie le nombre de nouvelles tentatives en cas d’échec de la copie.         |
    |/w:     |Spécifie le délai d’attente entre les tentatives, en secondes.         |
    |/is     |Inclut les mêmes fichiers.         |
    |/nfl    |Indique que les noms de fichiers ne sont pas enregistrés dans le journal.         |
    |/ndl    |Indique que les noms de répertoires ne sont pas enregistrés dans le journal.        |
    |/np     |Spécifie que la progression de l’opération de copie (le nombre de fichiers ou répertoires copiés jusqu’ici) n’est pas affichée. L’affichage de la progression réduit considérablement les performances.         |
    |/MT     | Utilise le multithreading (32 ou 64 threads recommandés). Cette option n’est pas utilisée avec les fichiers chiffrés. Vous devrez peut-être séparer les fichiers chiffrés et non chiffrés. Toutefois, une copie monothread réduit considérablement les performances.           |
    |/fft    | Utilisé pour réduire la granularité de l’horodatage pour n’importe quel système de fichiers.        |
    |/b      | Copie les fichiers en mode de sauvegarde.        |
    |/z      | Copie les fichiers en mode de redémarrage. Utilisez cette option si l’environnement est instable. La journalisation supplémentaire effectuée par cette option réduit le débit.      |
    | /zb    | Utilise le mode de redémarrage. Si l’accès est refusé, cette option utilise le mode de sauvegarde. La création de points de contrôle par cette option réduit le débit.         |
    |/efsraw | Copie tous les fichiers chiffrés en mode EFS brut. Utilisez cette option avec des fichiers chiffrés uniquement.         |
    |log+:\<LogFile>| Ajoute la sortie au fichier journal existant.|
    
 
    L’exemple suivant affiche la sortie de la commande robocopy destinée à copier les fichiers sur le Data Box Heavy.

    ```   
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.100.10.100\devicemanagertest1_AzFile\templates /MT:24
    -------------------------------------------------------------------------------
        ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
        Started : Thursday, April 4, 2019 2:34:58 PM
        Source : C:\Git\azure-docs-pr\contributor-guide\
        Dest : \\10.100.10.100\devicemanagertest1_AzFile\templates\
        Files : *.*
        Options : *.* /DCOPY:DA /COPY:DAT /MT:24 /R:5 /W:60
    ------------------------------------------------------------------------------
    
    100%        New File                 206        C:\Git\azure-docs-pr\contributor-guide\article-metadata.md
    100%        New File                 209        C:\Git\azure-docs-pr\contributor-guide\content-channel-guidance.md
    100%        New File                 732        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-index.md
    100%        New File                 199        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pr-criteria.md
                New File                 178        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-co100%  .md
                New File                 250        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-et100%  e.md
    100%        New File                 174        C:\Git\azure-docs-pr\contributor-guide\create-images-markdown.md
    100%        New File                 197        C:\Git\azure-docs-pr\contributor-guide\create-links-markdown.md
    100%        New File                 184        C:\Git\azure-docs-pr\contributor-guide\create-tables-markdown.md
    100%        New File                 208        C:\Git\azure-docs-pr\contributor-guide\custom-markdown-extensions.md
    100%        New File                 210        C:\Git\azure-docs-pr\contributor-guide\file-names-and-locations.md
    100%        New File                 234        C:\Git\azure-docs-pr\contributor-guide\git-commands-for-master.md
    100%        New File                 186        C:\Git\azure-docs-pr\contributor-guide\release-branches.md
    100%        New File                 240        C:\Git\azure-docs-pr\contributor-guide\retire-or-rename-an-article.md
    100%        New File                 215        C:\Git\azure-docs-pr\contributor-guide\style-and-voice.md
    100%        New File                 212        C:\Git\azure-docs-pr\contributor-guide\syntax-highlighting-markdown.md
    100%        New File                 207        C:\Git\azure-docs-pr\contributor-guide\tools-and-setup.md
    ------------------------------------------------------------------------------
    
                    Total    Copied   Skipped  Mismatch    FAILED    Extras
        Dirs :         1         1         1         0         0         0
        Files :        17        17         0         0         0         0
        Bytes :     3.9 k     3.9 k         0         0         0         0          
    C:\Users>
    ```       

2. Pour optimiser les performances, utilisez les paramètres robocopy suivants lors de la copie des données. (Les valeurs ci-dessous représentent les scénarios optimaux.)

    | Plateforme    | Principalement des fichiers de petite taille (< 512 Ko)    | Principalement des fichiers de taille moyenne (entre 512 Ko et 1 Mo)  | Principalement des fichiers volumineux (> 1 Mo)                             |
    |-------------|--------------------------------|----------------------------|----------------------------|
    | Data Box Heavy | 6 sessions Robocopy <br> 24 threads par session | 6 sessions Robocopy <br> 16 threads par session | 6 sessions Robocopy <br> 16 threads par session |


    Pour plus d’informations sur la commande Robocopy, consultez [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy et quelques exemples).

3. Ouvrez le dossier cible pour afficher et vérifier les fichiers copiés.

    ![Afficher les fichiers copiés](media/data-box-heavy-deploy-copy-data/view-copied-files-1.png)


4. Lors de la copie des données :

    - Les noms, tailles et formats de fichiers sont validés pour garantir qu’ils respectent les limites des objets et du stockage Azure ainsi que les conventions de nommage des conteneurs et des fichiers Azure.
    - Pour vérifier l’intégrité des données, la somme de contrôle est également calculée par le biais d’une fonction inline.

    Si vous rencontrez des erreurs au cours du processus de copie, téléchargez les fichiers d’erreurs pour résoudre les problèmes. Sélectionnez l’icône en forme de flèche pour télécharger les fichiers d’erreurs.

    ![Télécharger les fichiers d’erreurs](media/data-box-heavy-deploy-copy-data/download-error-files.png)

    Pour plus d’informations, consultez [Afficher les journaux d’erreurs pendant la copie de données vers Data Box Heavy](data-box-logs.md#view-error-log-during-data-copy). Pour obtenir une liste détaillée des erreurs lors de la copie de données, consultez [Résolution les problèmes liés à Data Box Heavy](data-box-troubleshoot.md).

5. Ouvrez le fichier d’erreur dans le Bloc-notes. Le fichier d’erreur suivant indique que les données ne sont pas correctement alignées.

    ![Ouvrir le fichier d’erreur](media/data-box-heavy-deploy-copy-data/open-error-file.png)
    
    Pour un objet blob de pages, les données doivent être alignées par éléments de 512 octets. Après la suppression de ces données, l’erreur est résolue, comme le montre la capture d’écran suivante.

    ![Erreur résolue](media/data-box-heavy-deploy-copy-data/error-resolved.png)

6. Une fois la copie terminée, accédez à la page **Afficher le tableau de bord**. Vérifiez l’espace utilisé et l’espace libre sur votre appareil.
    
    ![Vérifier l’espace libre et l’espace utilisé sur le tableau de bord](media/data-box-heavy-deploy-copy-data/verify-used-space-dashboard.png)

Répétez les étapes ci-dessus pour copier les données sur le deuxième nœud de l’appareil.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Data Box Heavy, notamment concernant les points suivants :

> [!div class="checklist"]
> * Se connecter à Data Box Heavy
> * Copier des données sur Data Box Heavy


Passez au tutoriel suivant pour découvrir comment réexpédier votre Data Box Heavy à Microsoft.

> [!div class="nextstepaction"]
> [Expédier votre Azure Data Box Heavy à Microsoft](./data-box-heavy-deploy-picked-up.md)

::: zone-end

::: zone target = "chromeless"

### <a name="copy-data-via-smb"></a>Copier des données via SMB

1. Si vous utilisez un hôte Windows, utilisez la commande suivante pour vous connecter aux partages SMB :

    `\\<IP address of your device>\ShareName`

2. Pour obtenir les informations d'identification d'accès aux partages, accédez à la page **Connexion et copie** de l'interface utilisateur web locale de Data Box.

3. Utilisez un outil de copie de fichier SMB comme Robocopy pour copier les données vers des partages.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Copier des données sur Azure Data Box Disk par le biais de SMB](data-box-heavy-deploy-copy-data.md).

### <a name="copy-data-via-nfs"></a>Copier des données via NFS

1. Si vous utilisez un hôte NFS, utilisez la commande suivante pour monter les partages NFS :

    `sudo mount <Data Box device IP>:/<NFS share on Data Box device> <Path to the folder on local Linux computer>`

2. Pour obtenir les informations d’identification d’accès aux partages, accédez à la page **Connecter et copie** de l’interface utilisateur web locale de Data Box Heavy.
3. Utilisez la commande `cp` ou `rsync` pour copier les données. 
4. Répétez ces étapes pour vous connecter et copier des données vers le deuxième nœud de votre Data Box Heavy.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Copier des données sur Azure Data Box Disk par le biais de NFS](data-box-heavy-deploy-copy-data-via-nfs.md).

### <a name="copy-data-via-rest"></a>Copier des données en utilisant REST

1. Pour copier des données à l’aide du stockage Blob Data Box en utilisant des API REST, vous pouvez vous connecter sur *http* ou *https*.
2. Pour copier des données vers le stockage Blob Data Box, vous pouvez utiliser AzCopy.
3. Répétez ces étapes pour vous connecter et copier des données vers le deuxième nœud de votre Data Box Heavy.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Copier des données vers le stockage Blob Azure Data Box par le biais des API REST](data-box-heavy-deploy-copy-data-via-rest.md).

### <a name="copy-data-via-data-copy-service"></a>Copier des données en utilisant le service de copie de données

1. Pour copier des données par l’intermédiaire du service de copie de données, vous devez créer un travail. Dans l’interface utilisateur web locale de votre Data Box Heavy, accédez à **Gérer > Copier des données > Créer**.
2. Renseignez les paramètres et créez un travail.
3. Répétez ces étapes pour vous connecter et copier des données vers le deuxième nœud de votre Data Box Heavy.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Utiliser le service de copie des données pour copier des données dans Azure Data Box Heavy](data-box-heavy-deploy-copy-data-via-copy-service.md).

### <a name="copy-data-to-managed-disks"></a>Copier des données sur des disques managés

1. Quand vous commandez l’appareil Data Box Heavy, vous devez avoir sélectionné des disques managés comme destination de stockage.
2. Vous pouvez vous connecter à Data Box Heavy par le biais de partages SMB ou NFS.
3. Vous pouvez ensuite copier des données à l’aide d’outils SMB ou NFS.
4. Répétez ces étapes pour vous connecter et copier des données vers le deuxième nœud de votre Data Box Heavy.

Pour obtenir des instructions pas à pas, consultez le [Tutoriel : Utiliser Data Box Heavy pour importer des données en tant que disques managés dans Azure](data-box-heavy-deploy-copy-data-from-vhds.md).

::: zone-end


