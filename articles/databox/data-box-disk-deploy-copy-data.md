---
title: Tutoriel pour copier des données sur Azure Data Box Disk | Microsoft Docs
description: Dans ce tutoriel, découvrez comment copier des données de votre ordinateur hôte vers Azure Data Box Disk, puis générer les sommes de contrôle pour vérifier l’intégrité des données.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 9186c742f34f1c060b57ffdfbf1bf688607b4c03
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92125335"
---
::: zone target="docs"

# <a name="tutorial-copy-data-to-azure-data-box-disk-and-verify"></a>Tutoriel : Copier des données sur Azure Data Box Disk et procéder à une vérification

::: zone-end

::: zone target="chromeless"

## <a name="copy-data-to-azure-data-box-disk-and-validate"></a>Copier des données sur un disque Azure Data Box et procéder à une validation

Une fois les disques connectés et déverrouillés, vous pouvez copier des données de votre serveur de données source sur vos disques. Une fois la copie des données terminée, vous devez valider les données pour garantir qu’elles seront correctement chargées dans Azure.

::: zone-end

::: zone target="docs"

Ce didacticiel explique comment copier des données à partir de votre ordinateur hôte, puis générer les sommes de contrôle pour vérifier l’intégrité des données.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Copier des données sur un disque Data Box
> * Vérifier les données

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :
- Vous avez terminé le [Tutoriel : Installer et configurer votre offre Azure Data Box Disk](data-box-disk-deploy-set-up.md).
- Vos disques sont déverrouillés et connectés à un ordinateur client.
- Votre ordinateur client utilisé pour copier des données sur les disques doit exécuter un [système d’exploitation pris en charge](data-box-disk-system-requirements.md#supported-operating-systems-for-clients).
- Vérifiez que le type de stockage prévu pour vos données correspond aux [types de stockage pris en charge](data-box-disk-system-requirements.md#supported-storage-types-for-upload).
- Vérifiez les [limites applicables aux disques managés dans la section sur les limites de taille des objets Azure](data-box-disk-limits.md#azure-object-size-limits).


## <a name="copy-data-to-disks"></a>Copier des données sur des disques

Passez en revue les considérations suivantes avant de commencer la copie des données sur les disques :

- C’est à vous de vous assurer que les données sont copiées vers des dossiers compatibles avec le format des données. Par exemple, les données d’objet blob de blocs doivent être copiées dans le dossier des objets blob de blocs. Si le format des données ne correspond pas au dossier (type de stockage), les données ne pourront pas être chargées dans Azure.
- Lorsque vous copiez des données, vérifiez que la taille des données est conforme aux limites de taille spécifiées dans l’article [Azure storage and Data Box Disk limits](data-box-disk-limits.md) (Limitations relatives au stockage Azure et aux disques Data Box).
- Si les données, qui sont en cours de chargement via Data Box Disk, sont chargées simultanément par d’autres applications en dehors de Data Box Disk, cela pourrait entraîner l’échec du téléchargement ou des corruptions de données.

   > [!IMPORTANT]
   >  Si vous avez spécifié des disques managés en tant que destination de stockage pendant la création de commandes, la section suivante s’applique.

- Vous pouvez avoir un seul disque managé du même nom dans un groupe de ressources sur l’ensemble des dossiers précréés et de Data Box Disk. Les disques durs virtuels chargés dans les dossiers pré-créés doivent donc avoir des noms uniques. Assurez-vous que le nom donné n’est pas déjà attribué à un autre disque managé existant dans un groupe de ressources. Si plusieurs disques durs virtuels ont le même nom, un seul de ces disques est converti en disque managé avec ce nom. Les autres disques durs virtuels sont chargés comme objets blob de pages dans le compte de stockage de préproduction.
- Vous devez toujours copier les disques durs virtuels dans l’un des dossiers pré-créés. Si vous copiez les disques durs virtuels ailleurs que dans ces dossiers ou dans un dossier que vous avez créé vous-même, les disques durs virtuels sont chargés dans le compte de stockage Azure comme objets blob de pages au lieu de disques non managés.
- Seuls les disques durs virtuels fixes peuvent être chargés pour créer des disques managés. Les disques durs virtuels dynamiques, les disques durs virtuels de différenciation ou les fichiers VHDX ne sont pas pris en charge.


Procédez comme suit pour vous connecter et copier des données à partir de votre ordinateur vers le disque Data Box.

1. Affichez le contenu du disque déverrouillé. La liste des dossiers et sous-dossiers précréés sur le disque varie selon les options sélectionnées au moment de la commande Data Box Disk. S’il n’existe pas de dossier précréé, ne le créez pas, car la copie vers un dossier créé par l’utilisateur ne pourra pas être chargée sur Azure.

    |Destination de stockage sélectionnée  |Type de compte de stockage|Type de compte de stockage de préproduction |Dossiers et sous-dossiers  |
    |---------|---------|---------|------------------|
    |Compte de stockage     |GPv1 ou GPv2                 | N/D | BlockBlob <br> PageBlob <br> AzureFile        |
    |Compte de stockage     |Compte de stockage d’objets blob         | N/D | BlockBlob        |
    |Disques managés     |N/D | GPv1 ou GPv2         | ManagedDisk<ul> <li>PremiumSSD</li><li>StandardSSD</li><li>StandardHDD</li></ul>        |
    |Compte de stockage <br> Disques managés     |GPv1 ou GPv2 | GPv1 ou GPv2         |BlockBlob <br> PageBlob <br> AzureFile <br> ManagedDisk<ul> <li> PremiumSSD </li><li>StandardSSD</li><li>StandardHDD</li></ul>         |
    |Compte de stockage <br> Disques managés    |Compte de stockage d’objets blob | GPv1 ou GPv2         |BlockBlob <br> ManagedDisk<ul> <li>PremiumSSD</li><li>StandardSSD</li><li>StandardHDD</li></ul>         |

    La capture d’écran ci-dessous montre un exemple de commande où figure un compte de stockage GPv2 :

    ![Contenu du lecteur de disque](media/data-box-disk-deploy-copy-data/data-box-disk-content.png)
 
2. Copiez les données devant être importées comme objets blob de blocs dans le dossier *BlockBlob*. De même, copiez les données telles que VHD/VHDX dans le dossier *PageBlob* et les données dans le dossier *AzureFile*.

    Un conteneur est créé dans le compte de stockage Azure de chaque sous-dossier sous les dossiers BlockBlob et PageBlob. Tous les fichiers sous les dossiers BlockBlob et PageBlob sont copiés dans un conteneur par défaut `$root` dans le compte de stockage Azure. Tous les fichiers présents dans le conteneur `$root` sont systématiquement chargés en tant qu’objets blob de blocs.

   Copiez les fichiers dans un sous-dossier du dossier *AzureFile*. Un sous-dossier dans le dossier *AzureFile* crée un partage de fichiers. Les fichiers copiés directement dans le dossier *AzureFile* échouent et sont chargés en tant qu’objets blob de blocs.

    Si les fichiers et les dossiers existent dans le répertoire racine, vous devez les déplacer vers un autre dossier avant de commencer à copier des données.

    > [!IMPORTANT]
    > Tous les conteneurs, objets blob et fichiers doivent respecter les [conventions de nommage Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Si ces règles ne sont pas respectées, le chargement des données vers Azure échoue.

3. Lorsque vous copiez des fichiers, assurez-vous qu’ils ne dépassent pas environ 4,7 Tio pour les objets blob de blocs, environ 8 Tio pour les objets blob de pages et environ 1 Tio pour Azure Files. 
4. Vous pouvez utiliser la fonction de glisser-déplacer dans l’Explorateur de fichiers pour copier les données. Vous pouvez également utiliser n’importe quel outil de copie de fichier SMB, comme Robocopy, pour copier vos données. Plusieurs travaux de copie peuvent être lancés simultanément à l’aide de la commande Robocopy suivante :

    `Robocopy <source> <destination>  * /MT:64 /E /R:1 /W:1 /NFL /NDL /FFT /Log:c:\RobocopyLog.txt` 
    
    Les paramètres et les options de la commande sont présentés sous forme de tableau tel que suit :
    
    |Paramètres/Options  |Description |
    |--------------------|------------|
    |Source            | Spécifie le chemin d’accès au répertoire source.        |
    |Destination       | Spécifie le chemin d’accès au répertoire de destination.        |
    |/E                  | Copie les sous-répertoires, y compris les répertoires vides. |
    |/MT[:N]             | Crée des copies multithread avec N threads où N est un nombre entier compris entre 1 et 128. <br>La valeur par défaut de N est 8.        |
    |/R: \<N>             | Spécifie le nombre de nouvelles tentatives en cas d’échec de la copie. La valeur par défaut de N est 1 000 000 (un million de tentatives).        |
    |/W: \<N>             | Spécifie le délai d’attente entre les tentatives, en secondes. La valeur par défaut de N est 30 (temps d’attente de 30 secondes).        |
    |/NFL                | Indique que les noms de fichier ne doivent pas être consignés.        |
    |/NDL                | Indique que les noms de répertoire ne doivent pas être consignés.        |
    |/FFT                | Calcule l’heure des fichiers FAT (à 2 secondes près).        |
    |/Log:\<Log File>     | Écrit la sortie d’état dans le fichier journal (remplace le fichier journal existant).         |

    Plusieurs disques peuvent être utilisés en parallèle, avec plusieurs travaux s’exécutant sur chaque disque.

6. Vérifiez l’état de l’opération de copie pendant l’exécution du travail. L’exemple suivant affiche la sortie de la commande robocopy pour copier les fichiers sur le disque Data Box.

    ```
    C:\Users>robocopy
        -------------------------------------------------------------------------------
       ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
      Started : Thursday, March 8, 2018 2:34:53 PM
           Simple Usage :: ROBOCOPY source destination /MIR
    
                 source :: Source Directory (drive:\path or \\server\share\path).
            destination :: Destination Dir  (drive:\path or \\server\share\path).
                   /MIR :: Mirror a complete directory tree.
    
        For more usage information run ROBOCOPY /?    
    
    ****  /MIR can DELETE files as well as copy them !
    
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.126.76.172\devicemanagertest1_AzFile\templates /MT:64 /E /R:1 /W:1 /FFT 
    -------------------------------------------------------------------------------
       ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
      Started : Thursday, March 8, 2018 2:34:58 PM
       Source : C:\Git\azure-docs-pr\contributor-guide\
         Dest : \\10.126.76.172\devicemanagertest1_AzFile\templates\
    
        Files : *.*
    
      Options : *.* /DCOPY:DA /COPY:DAT /MT:8 /R:1000000 /W:30
    
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
       Times :   0:00:05   0:00:00                       0:00:00   0:00:00
        
       Speed :                5620 Bytes/sec.
       Speed :               0.321 MegaBytes/min.
       Ended : Thursday, March 8, 2018 2:34:59 PM
        
    C:\Users>
    ```
 
    Pour optimiser les performances, utilisez les paramètres robocopy suivants lors de la copie des données.

    |    Plateforme    |    Principalement des fichiers de petite taille (< 512 Ko)                           |    Principalement des fichiers de taille moyenne (entre 512 Ko et 1 Mo)                      |    Principalement des fichiers volumineux (> 1 Mo)                             |   
    |----------------|--------------------------------------------------------|--------------------------------------------------------|--------------------------------------------------------|
    |    Data Box Disk        |    4 sessions Robocopy* <br> 16 threads par session    |    2 sessions Robocopy* <br> 16 threads par session    |    2 sessions Robocopy* <br> 16 threads par session    |
    
    **Chaque session Robocopy peut avoir un maximum de 7 000 répertoires et 150 millions de fichiers.*
    
    >[!NOTE]
    > Les paramètres suggérés ci-dessus s’appuient sur l’environnement utilisé lors des tests internes.
    
    Pour plus d’informations sur la commande Robocopy, consultez [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy et quelques exemples).

6. Ouvrez le dossier cible pour afficher et vérifier les fichiers copiés. Si vous rencontrez des erreurs au cours du processus de copie, téléchargez les fichiers journaux pour résoudre les problèmes. L’emplacement des fichiers journaux est spécifié dans la commande robocopy.
 
### <a name="split-and-copy-data-to-disks"></a>Fractionner et copier des données sur plusieurs disques

Vous pouvez exécuter cette procédure facultative lorsque vous utilisez plusieurs disques et que vous disposez d’un jeu de données volumineux qui doit être fractionné et copié sur la totalité des disques. L’outil Data Box Split Copy vous permet de fractionner et copier les données sur un ordinateur Windows.

>[!IMPORTANT]
> L’outil Data Box Split Copy valide également vos données. Si vous utilisez l’outil Data Box Split Copy pour copier des données, vous pouvez ignorer l’[étape de validation](#validate-data).
> L’outil Split Copy n’est pas pris en charge avec des disques managés.

1. L’outil Data Box Split Copy doit avoir été téléchargé et extrait dans un dossier local de votre ordinateur Windows. Cet outil a été téléchargé lorsque vous avez téléchargé l’ensemble d’outils Data Box Disk pour Windows.
2. Ouvrez l’Explorateur de fichiers. Notez la lettre de lecteur de la source de données et les lettres de lecteur attribuées à Data Box Disk. 

     ![Fractionnement des données de copie](media/data-box-disk-deploy-copy-data/split-copy-1.png)
 
3. Identifiez les données sources à copier. Par exemple, dans ce cas précis :
    - Les données d’objet blob de blocs ci-après ont été identifiées.

         ![Fractionnement des données de copie 2](media/data-box-disk-deploy-copy-data/split-copy-2.png)    

    - Les données d’objet blob de pages ci-après ont été identifiées.

         ![Fractionnement des données de copie 3](media/data-box-disk-deploy-copy-data/split-copy-3.png)
 
4. Accédez au dossier dans lequel le logiciel a été extrait. Recherchez le fichier `SampleConfig.json` dans ce dossier. Il s’agit d’un fichier en lecture seule que vous pouvez modifier et enregistrer.

   ![Fractionnement des données de copie 4](media/data-box-disk-deploy-copy-data/split-copy-4.png)
 
5. Modifiez le fichier `SampleConfig.json`.
 
   - Fournissez un nom de travail. Cette opération crée un dossier dans Data Box Disk qui devient le conteneur dans le compte de stockage Azure associé à ces disques. Le nom du travail doit respecter les conventions d’affectation de noms de conteneur Azure. 
   - Fournissez un chemin source en notant le format du chemin dans le fichier `SampleConfigFile.json`. 
   - Entrez les lettres de lecteur correspondant aux disques cibles. Les données sont copiées sur les différents disques à partir du chemin d’accès source.
   - Fournissez un chemin d’accès pour les fichiers journaux. Par défaut, ces fichiers sont envoyés au répertoire dans lequel se trouve le fichier `.exe`.

     ![Fractionnement des données de copie 5](media/data-box-disk-deploy-copy-data/split-copy-5.png)

6. Pour valider le format de fichier, accédez à `JSONlint`. Enregistrez le fichier sous le nom `ConfigFile.json`. 

     ![Fractionnement des données de copie 6](media/data-box-disk-deploy-copy-data/split-copy-6.png)
 
7. Ouvrez une fenêtre d’invite de commandes. 

8. Exécutez `DataBoxDiskSplitCopy.exe`. Type

    `DataBoxDiskSplitCopy.exe PrepImport /config:<Your-config-file-name.json>`

     ![Fractionnement des données de copie 7](media/data-box-disk-deploy-copy-data/split-copy-7.png)
 
9. Appuyez sur Entrée pour poursuivre le script.

    ![Fractionnement des données de copie 8](media/data-box-disk-deploy-copy-data/split-copy-8.png)
  
10. Une fois le jeu de données fractionné et copié, le résumé de l’outil Split Copy pour la session de copie vous est présenté. Voici un exemple de sortie obtenue.

    ![Fractionnement des données de copie 9](media/data-box-disk-deploy-copy-data/split-copy-9.png)
 
11. Vérifiez que les données ont été fractionnées entre les disques cibles. 
 
    ![Fractionnement des données de copie 10](media/data-box-disk-deploy-copy-data/split-copy-10.png)
    ![Fractionnement des données de copie 11](media/data-box-disk-deploy-copy-data/split-copy-11.png)
     
    Si vous examinez de plus près le contenu du lecteur `n:`, vous pouvez constater que deux sous-dossiers ont été créés et correspondent aux données d’objet blob de blocs et d’objet blob de pages.
    
     ![Fractionnement des données de copie 12](media/data-box-disk-deploy-copy-data/split-copy-12.png)

12. Si la session de copie échoue, vous pouvez récupérer et reprendre l’opération en utilisant la commande suivante :

    `DataBoxDiskSplitCopy.exe PrepImport /config:<configFile.json> /ResumeSession`

Si vous constatez des erreurs avec l’outil de fractionnement des données de copie, accédez au guide de [Résolution des erreurs de l’outil de fractionnement des données de copie](data-box-disk-troubleshoot-data-copy.md).

Une fois la copie des données terminée, vous pouvez passer à leur validation. Si vous avez utilisé l’outil Split Copy, ignorez la validation (car l’outil s’en charge aussi) et passez au tutoriel suivant.


## <a name="validate-data"></a>Valider des données

Si vous n’avez pas utilisé l’outil Split Copy pour copier les données, vous devez les valider. Pour vérifier les données, procédez comme suit.

1. Exécutez le fichier `DataBoxDiskValidation.cmd` pour la validation des sommes de contrôle dans le dossier *DataBoxDiskImport* de votre lecteur. Cette option est disponible uniquement pour les environnements Windows. Les utilisateurs Linux doivent vérifier que les données sources copiées sur le disque répondent aux [prérequis](./data-box-disk-limits.md).
    
    ![Sortie de l’outil de validation Data Box Disk](media/data-box-disk-deploy-copy-data/data-box-disk-validation-tool-output.png)

2. Choisissez l’option appropriée. **Nous vous recommandons de toujours valider les fichiers et générer des sommes de contrôle en sélectionnant l’option 2**. Selon la taille de vos données, cette étape peut prendre un certain temps. Une fois le script exécuté, quittez la fenêtre de commandes. Si des erreurs se produisent pendant la validation et la génération des sommes de contrôle, vous en êtes averti, et un lien d’accès aux journaux d’activité des erreurs vous est également fourni.

    ![Sortie de la somme de contrôle](media/data-box-disk-deploy-copy-data/data-box-disk-checksum-output.png)

    > [!TIP]
    > - Réinitialisez l’outil entre deux exécutions.
    > - Utiliser l’option 1 si vous traitez un grand jeu de données qui contient des petits fichiers (~ Ko). Cette option valide seulement les fichiers, car la génération de la somme de contrôle peut prendre beaucoup de temps et les performances peuvent ralentir considérablement.

3. Si vous utilisez plusieurs disques, exécutez la commande pour chaque disque.

Si vous constatez des erreurs lors de la validation, consultez [Résoudre les erreurs de validation](data-box-disk-troubleshoot.md).

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a apporté des connaissances sur les disques Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
> * Copier des données sur un disque Data Box
> * Vérifier l’intégrité des données

Passez au didacticiel suivant pour savoir comment renvoyer le disque Data Box et vérifier le chargement des données dans Azure.

> [!div class="nextstepaction"]
> [Envoyer votre Azure Data Box à Microsoft](./data-box-disk-deploy-picked-up.md)

::: zone-end

::: zone target="chromeless"

### <a name="copy-data-to-disks"></a>Copier des données sur des disques

Effectuez les étapes suivantes pour vous connecter et copier des données à partir de votre ordinateur vers le Data Box Disk.

1. Affichez le contenu du disque déverrouillé. La liste des dossiers et sous-dossiers précréés sur le disque varie selon les options sélectionnées au moment de la commande Data Box Disk.
2. Copiez les données dans des dossiers qui correspondent au format de données approprié. Par exemple, copiez les données non structurées dans le dossier *BlockBlob*, les données VHD ou VHDX dans le dossier *PageBlob* et les fichiers dans *AzureFile*. Si le format des données ne correspond pas au dossier (type de stockage), les données ne pourront pas être chargées vers Azure.

    - Assurez-vous que tous les conteneurs, objets blob et fichiers respectent les [conventions de nommage Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions) et les [limites de taille d’objet Azure](data-box-disk-limits.md#azure-object-size-limits). Si ces règles ou limites ne sont pas respectées, le chargement des données vers Azure échoue.     
    - Si votre commande comporte Disques managés comme l’une des destinations de stockage, consultez les conventions de nommage pour les [disques managés](data-box-disk-limits.md#managed-disk-naming-conventions).
    - Un conteneur est créé dans le compte de stockage Azure de chaque sous-dossier sous les dossiers BlockBlob et PageBlob. Tous les fichiers sous les dossiers *BlockBlob* et *PageBlob* sont copiés dans un conteneur par défaut $root dans le compte Stockage Azure. Tous les fichiers présents dans le conteneur $root sont systématiquement chargés en tant qu’objets blob de blocs.
    - Créez un sous-dossier dans le dossier *AzureFile*. Ce sous-dossier est mappé à un partage de fichiers dans le cloud. Copiez les fichiers dans le sous-dossier. Les fichiers copiés directement dans le dossier *AzureFile* échouent et sont chargés en tant qu’objets blob de blocs.
    - Si les fichiers et les dossiers existent dans le répertoire racine, vous devez les déplacer vers un autre dossier avant de commencer à copier des données.

3. Utilisez la fonction glisser-déplacer avec l’Explorateur de fichiers ou n’importe quel outil de copie de fichier compatible SMB, comme Robocopy, pour copier vos données. Plusieurs travaux de copie peuvent être lancés simultanément à l’aide de la commande suivante :

    ```
    Robocopy <source> <destination>  * /MT:64 /E /R:1 /W:1 /NFL /NDL /FFT /Log:c:\RobocopyLog.txt
    ```
4. Ouvrez le dossier cible pour afficher et vérifier les fichiers copiés. Si vous rencontrez des erreurs au cours du processus de copie, téléchargez les fichiers journaux pour résoudre les problèmes. L’emplacement des fichiers journaux est spécifié dans la commande robocopy.

Appliquez la procédure facultative de [fractionnement et copie](data-box-disk-deploy-copy-data.md#split-and-copy-data-to-disks) quand vous utilisez plusieurs disques et que vous disposez d’un jeu de données volumineux qui doit être fractionné et copié sur la totalité des disques.

### <a name="validate-data"></a>Valider des données

Effectuez les étapes suivantes pour vérifier vos données.

1. Exécutez le fichier `DataBoxDiskValidation.cmd` pour la validation des sommes de contrôle dans le dossier *DataBoxDiskImport* de votre lecteur.
2. Utilisez l’option 2 pour valider vos fichiers et générer des sommes de contrôle. Selon la taille de vos données, cette étape peut prendre un certain temps. Si des erreurs se produisent pendant la validation et la génération des sommes de contrôle, vous en êtes averti, et un lien d’accès aux journaux d’activité des erreurs vous est également fourni.

    Pour plus d’informations sur la validation de données, consultez [Valider les données](#validate-data). Si des erreurs se produisent lors de la validation, consultez [Résoudre les erreurs de validation](data-box-disk-troubleshoot.md).

::: zone-end