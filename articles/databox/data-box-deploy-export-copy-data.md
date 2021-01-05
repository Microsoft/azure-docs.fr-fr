---
title: Tutoriel pour copier des données via SMB depuis votre Azure Data Box | Microsoft Docs
description: Découvrez comment copier des données sur votre Azure Data Box par le biais de SMB.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 07/10/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: ecc6e1e1a543f3190e9f73512ca0b9ae45cc3fe9
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335211"
---
# <a name="tutorial-copy-data-from-azure-data-box-via-smb-preview"></a>Tutoriel : Copier des données depuis Azure Data Box via SMB (préversion)

Ce tutoriel explique comment se connecter à votre appareil Data Box et copier des données entre celui-ci et un serveur local par le biais de l’interface utilisateur web locale. L’appareil Data Box contient les données exportées à partir de votre compte de stockage Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter à Data Box
> * Copier des données à partir de la Data Box

[!INCLUDE [Data Box feature is in preview](../../includes/data-box-feature-is-preview-info.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez passé la commande pour Azure Data Box.
    - Pour une commande d’importation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-ordered.md).
    - Pour une commande d’exportation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-export-ordered.md).
2. Vous avez reçu votre Data Box et que l’état de la commande dans le portail est **Remis**.
3. Vous disposez d’un ordinateur hôte sur lequel vous voulez copier les données de votre Data Box. Votre ordinateur hôte doit
   * Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
   * Connectez-vous à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, utilisez une liaison de données 1 GbE. Cependant, les vitesses de copie seront affectées.

## <a name="connect-to-data-box"></a>Se connecter à Data Box

[!INCLUDE [data-box-shares](../../includes/data-box-shares.md)]

Si vous utilisez un ordinateur hôte Windows Server, effectuez les étapes suivantes pour vous connecter à la Data Box.

1. La première étape consiste à vous authentifier et à démarrer une session. Accédez à la page **Connect and copy** (Connexion et copie). Sélectionnez **Obtenir les informations d’identification** pour obtenir les informations d’identification qui sont nécessaires pour accéder aux partages associés à votre compte de stockage. 

    ![Obtenir les informations d’identification de partage](media/data-box-deploy-export-copy-data/get-share-credentials-1.png)

2. Dans la boîte de dialogue Access share and copy data (Accéder au partage et copier les données), copiez les valeurs **Nom d’utilisateur** et **Mot de passe** correspondant au partage. Sélectionnez **OK**.
    
    ![Obtenir les informations d’identification de partage, accéder au partage et copier les données](media/data-box-deploy-export-copy-data/get-share-credentials-2.png)

3. Pour accéder aux partages associés à votre compte de stockage (*exportbvtdataset2* dans l’exemple suivant) à partir de votre ordinateur hôte, ouvrez une fenêtre de commande. À l’invite de commandes, tapez :

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    En fonction du format de vos données, les chemins de partage sont les suivants :
    - Objet blob de blocs Azure - `\\169.254.143.85\exportbvtdataset2_BlockBlob`
    - Objet blob de pages Azure - `\\169.254.143.85\exportbvtdataset2_PageBlob`
    - Azure Files - `\\169.254.143.85\exportbvtdataset2_AzFile`

4. Lorsque vous y êtes invité, entrez le mot de passe du partage. L’exemple suivant illustre la connexion à un partage à l’aide de la commande précédente.

    ```
    C:\Users\Databoxuser>net use \\169.254.143.85\exportbvtdataset2_BlockBlob /u:exportbvtdataset2
    Enter the password for 'exportbvtdataset2' to connect to '169.254.143.85':
    The command completed successfully.
    ```

5. Appuyez sur Windows + R. Dans la fenêtre **Exécuter**, spécifiez `\\<device IP address>`. Sélectionnez **OK** pour ouvrir l’Explorateur de fichiers.
    
    ![Se connecter au partage via l’Explorateur de fichiers, entrer l’adresse IP de l’appareil](media/data-box-deploy-export-copy-data/connect-shares-file-explorer-1.png)

    À présent, vous devriez voir les partages sous forme de dossiers.
    
    ![Se connecter au partage via l’Explorateur de fichiers, afficher les partages](media/data-box-deploy-export-copy-data/connect-shares-file-explorer-2.png)

    
Si vous utilisez un client Linux, utilisez la commande suivante pour monter le partage SMB. Le paramètre « vers » ci-dessous est la version de SMB que votre hôte Linux prend en charge. Utilisez la version appropriée dans la commande ci-dessous. Pour connaître les versions de SMB que le service the Data Box prend en charge, consultez [Systèmes de fichiers pris en charge pour les clients Linux](./data-box-system-requirements.md#supported-file-transfer-protocols-for-clients). 

```console
sudo mount -t nfs -o vers=2.1 169.254.143.85:/exportbvtdataset2_BlockBlob /home/databoxubuntuhost/databox
```

## <a name="copy-data-from-data-box"></a>Copier des données à partir de la Data Box

Une fois que vous êtes connecté aux partages Data Box, l’étape suivante consiste à copier les données.

[!INCLUDE [data-box-export-review-logs](../../includes/data-box-export-review-logs.md)]


 Après vous être connecté au partage SMB, commencez la copie des données. Vous pouvez utiliser n’importe quel outil de copie de fichier SMB, comme Robocopy, pour copier vos données. Plusieurs travaux de copie peuvent être lancés simultanément à l’aide de Robocopy. 


Pour plus d’informations sur la commande Robocopy, consultez [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy et quelques exemples).

Une fois la copie terminée, accédez au **Tableau de bord**, puis vérifiez l’espace utilisé et l’espace libre sur votre appareil.

Vous pouvez maintenant procéder à l’expédition de votre Data Box à Microsoft.


## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter à Data Box
> * Copier des données à partir de la Data Box

Passez au tutoriel suivant pour découvrir comment renvoyer votre Data Box à Microsoft.

> [!div class="nextstepaction"]
> [Ship your Azure Data Box to Microsoft](./data-box-deploy-export-picked-up.md) (Expédier votre Azure Data Box à Microsoft)