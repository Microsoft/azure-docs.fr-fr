---
title: 'Tutoriel : Migrer des données locales vers Stockage Azure avec AzCopy | Microsoft Docs'
description: Dans ce tutoriel, vous utilisez AzCopy pour migrer ou copier des données vers ou depuis un blob, une table ou un fichier. Migrez facilement des données de votre stockage local vers le Stockage Azure.
author: normesta
ms.service: storage
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: eddee58d70cf621bd6c82d54fe75434f4e596d9e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498164"
---
#  <a name="tutorial-migrate-on-premises-data-to-cloud-storage-with-azcopy"></a>Tutoriel : Migrer des données locales vers le stockage cloud avec AzCopy

AzCopy est un outil en ligne de commande qui vous permet de copier des données vers ou depuis le Stockage Blob Azure, Azure Files ou le Stockage Table Azure, à l’aide de commandes simples. Les commandes sont conçues pour offrir des performances optimales. Grâce à AzCopy, vous pouvez copier des données entre un système de fichiers et un compte de stockage, ou entre comptes de stockage. AzCopy peut être utilisé pour copier des données à partir de données locales vers un compte de stockage.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un compte de stockage. 
> * Utilisez AzCopy pour charger toutes vos données.
> * Modifiez les données pour les besoins du test.
> * Créez une tâche planifiée ou un travail Cron pour identifier les nouveaux fichiers à charger.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, téléchargez la dernière version d’AzCopy. Consultez [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md).

Si vous êtes sur Windows, vous aurez besoin de [Schtasks](/windows/win32/taskschd/schtasks) car ce tutoriel l’utilise afin de planifier une tâche. Les utilisateurs Linux utiliseront la commande crontab à la place.

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="create-a-container"></a>Créez un conteneur.

La première étape consiste à créer un conteneur, étant donné que les objets Blob doivent toujours être chargés dans un conteneur. Les conteneurs sont utilisés en tant que méthode d’organisation des groupes d’objets Blob, comme vous le feriez dans des dossiers pour les fichiers sur votre ordinateur.

Pour créer un conteneur, effectuez les étapes suivantes :

1. Sélectionnez le bouton **Comptes de stockage** dans la page principale, puis sélectionnez le compte de stockage que vous avez créé.
2. Sélectionnez **Objets blob** sous **Services**, puis sélectionnez **Conteneur**.

   ![Capture d’écran montrant la création d’un conteneur](media/storage-azcopy-migrate-on-premises-data/CreateContainer.png)
 
Les noms de conteneur doivent commencer par une lettre ou un chiffre. Ils peuvent contenir uniquement des lettres, des chiffres et des traits d’union (-). Pour prendre connaissance des autres règles de nommage des objets blob et des conteneurs, consultez [Nommage et référencement des conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

## <a name="download-azcopy"></a>Télécharger AzCopy

Téléchargez le fichier exécutable AzCopy V10.

- [Windows](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Linux](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

Placez le fichier AzCopy sur votre ordinateur à l’emplacement de votre choix. Ajoutez l’emplacement du fichier à votre variable de chemin système pour pouvoir faire référence à ce fichier exécutable depuis n’importe quel dossier sur votre ordinateur.

## <a name="authenticate-with-azure-ad"></a>S’authentifier avec Azure AD

Tout d’abord, attribuez le rôle [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-queue-data-contributor) à votre identité. Consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](./storage-auth-aad-rbac-portal.md).

Ensuite, ouvrez une invite de commandes, tapez la commande suivante, puis appuyez sur la touche Entrée.

```azcopy
azcopy login
```

Cette commande retourne un code d’authentification et l’URL d’un site web. Ouvrez le site web, indiquez le code, puis choisissez le bouton **Suivant**.

![Capture d’écran montrant l’invite de connexion](media/storage-use-azcopy-v10/azcopy-login.png)

Une fenêtre de connexion s’affiche. Dans cette fenêtre, connectez-vous à votre compte Azure à l’aide de vos informations d’identification de compte Azure. Une fois connecté, vous pouvez fermer la fenêtre du navigateur et commencer à utiliser AzCopy.

## <a name="upload-contents-of-a-folder-to-blob-storage"></a>Charger les contenus d’un dossier dans le Stockage Blob

Vous pouvez utiliser AzCopy pour charger tous les fichiers d’un dossier dans le Stockage Blob sur [Windows](./storage-use-azcopy-v10.md) ou [Linux](./storage-use-azcopy-v10.md). Pour charger tous les objets blob d’un dossier, entrez la commande AzCopy suivante :

```AzCopy
azcopy copy "<local-folder-path>" "https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>" --recursive=true
```

* Remplacez l’espace réservé `<local-folder-path>` par le chemin d’un dossier qui contient des fichiers (par exemple : `C:\myFolder` ou `/mnt/myFolder`).

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

* Remplacez l’espace réservé `<container-name>` par le nom du conteneur que vous avez créé.

Pour charger le contenu du répertoire spécifié dans le stockage Blob de manière récursive, spécifiez l’option `--recursive`. Quand vous exécutez AzCopy avec cette option, tous les sous-dossiers et leurs fichiers sont également chargés.

## <a name="upload-modified-files-to-blob-storage"></a>Charger les fichiers modifiés dans le Stockage Blob

Vous pouvez utiliser AzCopy pour charger des fichiers en fonction de l’heure de leur dernière modification. 

Pour les besoins du test, modifiez des fichiers existants ou créez-en d’autres dans votre répertoire source. Ensuite, utilisez la commande AzCopy `sync`.

```AzCopy
azcopy sync "<local-folder-path>" "https://<storage-account-name>.blob.core.windows.net/<container-name>" --recursive=true
```

* Remplacez l’espace réservé `<local-folder-path>` par le chemin d’un dossier qui contient des fichiers (par exemple : `C:\myFolder` ou `/mnt/myFolder`).

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

* Remplacez l’espace réservé `<container-name>` par le nom du conteneur que vous avez créé.

Pour en savoir plus sur la commande `sync`, consultez [Synchroniser des fichiers](storage-use-azcopy-blobs.md#synchronize-files).

## <a name="create-a-scheduled-task"></a>Créer une tâche planifiée

Vous pouvez créer une tâche planifiée ou un travail Cron qui exécute un script de commande AzCopy. Le script identifie et charge les nouvelles données locales dans le stockage cloud selon un intervalle de temps spécifique.

Copiez la commande AzCopy dans un éditeur de texte. Mettez à jour les valeurs de paramètre de la commande AzCopy avec les valeurs appropriées. Enregistrez le fichier sous `script.sh` (Linux) ou `script.bat` (Windows) pour AzCopy. 

Ces exemples supposent que le dossier est nommé `myFolder`, que le nom de votre compte de stockage est `mystorageaccount` et celui de votre conteneur est `mycontainer`.

> [!NOTE]
> L’exemple Linux ajoute un jeton SAS. Vous devez en fournir un dans votre commande. La version actuelle de AzCopy V10 ne prend pas en charge les autorisations Azure AD dans les travaux Cron.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
azcopy sync "/mnt/myfiles" "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-05-30T06:57:40Z&st=2019-05-29T22:57:40Z&spr=https&sig=BXHippZxxx54hQn%2F4tBY%2BE2JHGCTRv52445rtoyqgFBUo%3D" --recursive=true
```

# <a name="windows"></a>[Windows](#tab/windows)

```bash
azcopy sync "C:\myFolder" "https://mystorageaccount.blob.core.windows.net/mycontainer" --recursive=true
```

---

Dans ce didacticiel, [Schtasks](/windows/win32/taskschd/schtasks) est utilisé pour créer une tâche planifiée sur Windows. La commande [Crontab](http://crontab.org/) est utilisée pour créer un travail Cron sur Linux.

 Avec **Schtasks**, un administrateur peut créer, supprimer, interroger, modifier, exécuter et terminer des tâches planifiées sur un ordinateur local ou distant. Avec **Cron**, les utilisateurs sur Linux et Unix peuvent exécuter des commandes ou des scripts à la date et à l’heure qu’ils ont spécifiées à l’aide [d’expressions Cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="linux"></a>[Linux](#tab/linux)

Pour créer un travail Cron sur Linux, entrez la commande suivante sur un terminal :

```bash
crontab -e
*/5 * * * * sh /path/to/script.sh
```

La spécification de l’expression Cron `*/5 * * * *` dans la commande indique que le script d’interpréteur de commandes `script.sh` doit s’exécuter toutes les cinq minutes. Vous pouvez planifier le script pour qu’il s’exécute à une heure précise tous les jours, tous les mois ou tous les ans. Pour en savoir plus sur la définition de la date et l’heure d’exécution d’un travail, consultez la page sur les [expressions Cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="windows"></a>[Windows](#tab/windows)

Pour créer une tâche planifiée sur Windows, entrez la commande suivante à une invite de commandes ou dans PowerShell :

Cet exemple suppose que votre script se trouve dans le lecteur racine de votre ordinateur, mais il peut se trouver à n’importe quel emplacement de votre choix.

```cmd
schtasks /CREATE /SC minute /MO 5 /TN "AzCopy Script" /TR C:\script.bat
```

La commande utilise les paramètres suivants :
- `/SC` : spécifie une planification en minutes.
- `/MO` : spécifie un intervalle de temps de cinq minutes.
- `/TN` : spécifie le nom de la tâche.
- `/TR` : spécifie le chemin du fichier `script.bat`.

Pour en savoir plus sur la création d’une tâche planifiée sur Windows, consultez [Schtasks](/previous-versions/orphan-topics/ws.10/cc772785(v=ws.10)#BKMK_minutes).

---

Pour vérifier que la tâche planifiée ou le travail Cron s’exécute correctement, créez des fichiers de test dans votre répertoire `myFolder`. Attendez cinq minutes pour laisser le temps aux nouveaux fichiers de se charger dans votre compte de stockage. Accédez à votre répertoire des journaux d’activité pour afficher les journaux d’activité de sortie de la tâche planifiée ou du travail Cron.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les façons de déplacer des données locales vers le Stockage Azure, et inversement, suivez ce lien :

* [Déplacer des données vers et à partir de Stockage Azure](./storage-choose-data-transfer-solution.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).  

Pour plus d’informations sur AzCopy, consultez les articles suivants :

* [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)

* [Transférer des données avec AzCopy et le Stockage Blob](storage-use-azcopy-blobs.md)

* [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)

* [Transférer des données avec AzCopy et des compartiments Amazon S3](storage-use-azcopy-s3.md)
 
* [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)