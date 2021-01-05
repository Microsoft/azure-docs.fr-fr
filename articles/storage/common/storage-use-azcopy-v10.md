---
title: Copier ou déplacer des données vers Stockage Azure avec AzCopy v10 | Microsoft Docs
description: AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des données vers, depuis ou entre des comptes de stockage. Cet article vous aide à télécharger AzCopy, connectez-vous à votre compte de stockage et transférez des fichiers.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.custom: contperf-fy21q2
ms.openlocfilehash: 6232b315abf8840d6fd77d02e3e928c0004746a0
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029326"
---
# <a name="get-started-with-azcopy"></a>Bien démarrer avec AzCopy

AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des blobs ou des fichiers vers ou depuis un compte de stockage. Cet article vous aide à télécharger AzCopy, connectez-vous à votre compte de stockage et transférez des fichiers.

> [!NOTE]
> La version actuellement prise en charge d’AzCopy est la **version 10**.
>
> Si vous avez besoin d’utiliser une version antérieure d’AzCopy, consultez la section [Utiliser la version précédente d’AzCopy](#previous-version) de cet article.

<a id="download-and-install-azcopy"></a>

## <a name="download-azcopy"></a>Télécharger AzCopy

Commencez par télécharger le fichier exécutable AzCopy V10 dans un répertoire sur votre ordinateur. AzCopy v10 est un fichier exécutable, et il n'y a donc rien à installer.

- [Windows 64 bits](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Windows 32 bits](https://aka.ms/downloadazcopy-v10-windows-32bit) (zip)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

Ces fichiers sont compressés dans un fichier zip (Windows et Mac) ou un fichier tar (Linux). Pour télécharger et décompresser le fichier tar sur Linux, consultez la documentation de votre distribution Linux.

> [!NOTE]
> Si vous voulez copier des données depuis et vers le service [Table de stockage Azure](../tables/table-storage-overview.md), installez [AzCopy version 7.3](https://aka.ms/downloadazcopynet).


## <a name="run-azcopy"></a>Exécuter AzCopy

Pour plus de commodité, vous pouvez envisager d’ajouter l’emplacement du répertoire de l’exécutable AzCopy dans votre chemin d’accès du système. De cette façon, vous pouvez saisir `azcopy` à partir de n’importe quel répertoire sur votre système.

Si vous choisissez de ne pas ajouter le répertoire AzCopy à votre chemin d’accès, vous devrez remplacer les répertoires par l’emplacement de votre fichier exécutable AzCopy et saisir `azcopy` ou `.\azcopy` dans les invites de commandes Windows PowerShell.

Pour afficher une liste de commandes, saisissez `azcopy -h` et appuyez sur la touche ENTRÉE.

Pour en savoir plus sur une commande spécifique, incluez simplement le nom de la commande (par exemple : `azcopy list -h`).

> [!div class="mx-imgBorder"]
> ![Aide en ligne](media/storage-use-azcopy-v10/azcopy-inline-help.png)


Pour obtenir une documentation de référence détaillée pour chaque commande et paramètre de commande, consultez [azcopy](storage-ref-azcopy.md)

> [!NOTE] 
> En tant que propriétaire de votre compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée. Avant d’effectuer une opération pertinente avec AzCopy, vous devez choisir comment fournir les informations d’identification d’autorisation du service de stockage. 

<a id="choose-how-youll-provide-authorization-credentials"></a>

## <a name="authorize-azcopy"></a>Autoriser AzCopy

Vous pouvez fournir des informations d’identification d’autorisation à l’aide d’Azure Active Directory (AD), ou en utilisant un jeton de signature d’accès partagé (SAP).

Utilisez ce tableau pour vous guider :

| Type de stockage | Méthode actuellement prise en charge pour l’autorisation |
|--|--|
|**Stockage Blob** | Azure AD et SAS |
|**Stockage de blobs (espace de noms hiérarchique)** | Azure AD et SAS |
|**Stockage Fichier** | SAS uniquement |

#### <a name="option-1-use-azure-active-directory"></a>Option 1 : Utiliser Azure Active Directory

Cette option est disponible uniquement pour le stockage blob. En utilisant Azure Active Directory, vous pouvez fournir des informations d’identification en une seule fois au lieu d’ajouter un jeton SAS à chaque commande.  

> [!NOTE]
> Dans la version actuelle, si vous envisagez de copier des blobs entre des comptes de stockage, vous devez ajouter un jeton SAS à chaque URL source. Vous pouvez omettre le jeton SAS uniquement à partir de l’URL de destination. Pour des exemples, consultez [Copier des objets blob entre des comptes de stockage](storage-use-azcopy-blobs.md).

Pour autoriser l’accès à l’aide d’Azure AD, consultez [Autoriser l’accès aux objets blob avec AzCopy et Azure Active Directory (Azure AD)](storage-use-azcopy-authorize-azure-active-directory.md).

#### <a name="option-2-use-a-sas-token"></a>Option n°2 : Utiliser un jeton SAS

Vous pouvez ajouter un jeton SAP à chaque URL source ou de destination utilisée dans vos commandes AzCopy.

Cet exemple de commande copie de manière récursive les données d’un répertoire local dans un conteneur de blobs. Un jeton SAP fictif est ajouté à la fin de l’URL du conteneur.

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

Pour en savoir plus sur les jetons SAS et savoir comment en obtenir un, voir [Utilisation des signatures d’accès partagé (SAP)](./storage-sas-overview.md).

<a id="transfer-data"></a>

## <a name="transfer-data"></a>Transfert de données

Une fois que vous avez autorisé votre identité ou obtenu un jeton SAP, vous pouvez commencer le transfert des données.

Pour obtenir des exemples de commandes, consultez l’un de ces articles.

| Service | Article |
|--------|-----------|
|Stockage Blob Azure |[Charger des fichiers dans Stockage Blob Azure](storage-use-azcopy-blobs-upload.md)<br><br>[Télécharger des blobs depuis Stockage Blob Azure](storage-use-azcopy-blobs-download.md)<br><br>[Copier des blobs entre des comptes de stockage Azure](storage-use-azcopy-blobs-download.md)<br><br>[Synchroniser avec Stockage Blob Azure](storage-use-azcopy-blobs-download.md)|
|Azure Files |[Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)|
|Amazon S3|[Transférer des données avec AzCopy et des compartiments Amazon S3](storage-use-azcopy-s3.md)|
|Stockage Azure Stack|[Transfert de données avec AzCopy et le stockage Azure Stack](/azure-stack/user/azure-stack-storage-transfer#azcopy)|

## <a name="use-in-a-script"></a>Utiliser dans un script

#### <a name="obtain-a-static-download-link"></a>Obtenir un lien de téléchargement statique

Au fil du temps, le [lien de téléchargement](#download-and-install-azcopy) AzCopy pointera vers les nouvelles versions d’AzCopy. Si votre script télécharge AzCopy, il se peut qu’il cesse de fonctionner si une version plus récente d’AzCopy modifie les fonctionnalités dont votre script dépend.

Pour éviter ces problèmes, obtenez un lien statique (non modifiable) vers la version actuelle d’AzCopy. De cette façon, votre script télécharge la même version exacte d’AzCopy à chaque fois qu’il s’exécute.

Pour obtenir le lien, exécutez cette commande :

| Système d’exploitation  | Commande |
|--------|-----------|
| **Linux** | `curl -s -D- https://aka.ms/downloadazcopy-v10-linux | grep ^Location` |
| **Windows** | `(curl https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction silentlycontinue).headers.location` |

> [!NOTE]
> Pour Linux, `--strip-components=1` sur la commande `tar` supprime le dossier de niveau supérieur qui contient le nom de version, et extrait le binaire directement dans le dossier courant. Ceci permet de mettre à jour le script avec une nouvelle version de `azcopy` en mettant à jour uniquement l’URL `wget`.

L’URL apparaît dans la sortie de cette commande. Votre script peut alors télécharger AzCopy en utilisant cette URL.

| Système d’exploitation  | Commande |
|--------|-----------|
| **Linux** | `wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1` |
| **Windows** | `Invoke-WebRequest https://azcopyvnext.azureedge.net/release20190517/azcopy_windows_amd64_10.1.2.zip -OutFile azcopyv10.zip <<Unzip here>>` |

#### <a name="escape-special-characters-in-sas-tokens"></a>Caractères spéciaux d’échappement dans les jetons SAS

Dans les fichiers de lot qui portent l’extension `.cmd`, vous devez placer dans une séquence d’échappement les caractères `%` qui apparaissent dans les jetons SAS. Pour cela, ajoutez un caractère `%` supplémentaire à côté des caractères `%` existants dans la chaîne de jeton SAS.

#### <a name="run-scripts-by-using-jenkins"></a>Exécuter des scripts à l’aide de Jenkins

Si vous envisagez d’utiliser [Jenkins](https://jenkins.io/) pour exécuter des scripts, veillez à placer la commande suivante au début du script.

```
/usr/bin/keyctl new_session
```

## <a name="use-in-azure-storage-explorer"></a>Utiliser dans l'Explorateur Stockage Azure

L’[Explorateur de stockage](https://azure.microsoft.com/features/storage-explorer/) utilise AzCopy pour effectuer toutes les opérations de transfert de données. Vous pouvez utiliser l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) si vous souhaitez tirer parti des avantages d’AzCopy en termes de performances, mais préférez utiliser l’interface graphique utilisateur plutôt que la ligne de commande pour interagir avec vos fichiers.

L’Explorateur Stockage utilise votre clé de compte pour effectuer des opérations, donc après vous être connecté à Storage Explorer, vous n’aurez pas besoin de fournir des informations d’autorisation supplémentaires.

<a id="previous-version"></a>

## <a name="configure-optimize-and-fix"></a>Configurer, optimiser et corriger

Voir [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)

## <a name="use-a-previous-version"></a>Utiliser une version précédente

Si vous devez utiliser la version précédente d’AzCopy, consultez l’un des liens suivants :

- [AzCopy sur Windows (v8)](/previous-versions/azure/storage/storage-use-azcopy)

- [AzCopy sur Linux (v7)](/previous-versions/azure/storage/storage-use-azcopy-linux)

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des questions, des problèmes ou des commentaires d’ordre général, saisissez-les sur la page [GitHub](https://github.com/Azure/azure-storage-azcopy).