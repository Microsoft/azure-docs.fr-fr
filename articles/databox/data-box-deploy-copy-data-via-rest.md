---
title: 'Tutoriel : Copier dans le Stockage Blob par le biais des API REST'
titleSuffix: Azure Data Box
description: Ce tutoriel montre comment vous connecter à un stockage Blob Azure Data Box à l’aide d’API REST sur http ou https, puis copier des données à partir d’Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 07/02/2020
ms.author: alkohli
ms.openlocfilehash: cb0a90db0595c655191006969071bc5b9cceaa75
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94337591"
---
# <a name="tutorial-use-rest-apis-to-copy-data-to-azure-data-box-blob-storage"></a>Tutoriel : Utiliser les API REST pour copier des données vers le stockage Blob Azure Data Box  

Ce tutoriel explique comment se connecter au stockage Blob Azure Data Box par le biais des API REST sur *http* ou *https*. Une fois connecté, les étapes nécessaires pour copier les données dans le stockage Blob Data Box et préparer la Data Box pour l’expédition sont également décrites.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter au stockage Blob Data Box par le biais de *http* ou *https*
> * Copier des données sur Data Box

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le [Tutoriel : Configurer Azure Data Box](data-box-deploy-set-up.md).
2. Vous avez reçu votre Data Box et que l’état de la commande dans le portail est **Remis**.
3. Vous avez consulté la [configuration système requise pour le stockage Blob Data Box](data-box-system-requirements-rest.md) et vous connaissez les versions prises en charge des API, SDK et outils.
4. Vous avez accès à un ordinateur hôte contenant les données que vous souhaitez copier sur Data Box. Votre ordinateur hôte doit
    * Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
    * Connectez-vous à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, vous pouvez utiliser une liaison de données 1 GbE. Cependant, les vitesses de copie seront affectées.
5. [Téléchargez AzCopy 7.1.0](https://aka.ms/azcopyforazurestack20170417) sur votre ordinateur hôte. Vous utiliserez AzCopy pour copier des données vers le stockage Blob Azure Data Box à partir de votre ordinateur hôte.

## <a name="connect-via-http-or-https"></a>Se connecter par le biais de http ou https

Vous pouvez vous connecter au stockage Blob Data Box par le biais de *http* ou *https*.

* *Https* est le protocole recommandé et le plus sûr pour se connecter au stockage Blob Data Box.
* *Http* est utilisé lors de la connexion par le biais de réseaux approuvés.

Les étapes de connexion varient selon que vous vous connectez au stockage Blob Data Box sur *http* ou *https*.

## <a name="connect-via-http"></a>Se connecter par le biais de http

La connexion aux API REST de stockage Blob Data Box par le biais de *http* implique les étapes suivantes :

* Ajouter l’adresse IP de l’appareil et le point de terminaison de service blob à l’hôte distant
* Configurer les logiciels tiers et vérifier la connexion

Chacune de ces étapes est décrite dans les sections suivantes.

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Ajouter l’adresse IP de l’appareil et le point de terminaison du service blob

[!INCLUDE [data-box-add-device-ip](../../includes/data-box-add-device-ip.md)]

### <a name="verify-connection-and-configure-partner-software"></a>Vérifier la connexion et configurer les logiciels partenaires

[!INCLUDE [data-box-configure-partner-software](../../includes/data-box-configure-partner-software.md)]

[!INCLUDE [data-box-verify-connection](../../includes/data-box-verify-connection.md)]

## <a name="connect-via-https"></a>Se connecter par le biais de https

La connexion aux API REST de stockage Blob Azure par le biais de https implique les étapes suivantes :

* Télécharger le certificat à partir du portail Azure
* Importer le certificat sur le client ou l’hôte distant
* Ajouter l’adresse IP de l’appareil et le point de terminaison de service blob au client ou à l’hôte distant
* Configurer les logiciels tiers et vérifier la connexion

Chacune de ces étapes est décrite dans les sections suivantes.

### <a name="download-certificate"></a>Télécharger le certificat

Utilisez le portail Azure pour télécharger le certificat.

1. Connectez-vous au portail Azure.
2. Affichez votre commande Data Box, puis accédez à **Général > Détails de l’appareil**.
3. Sous **informations d’identification de l’appareil**, accédez à **Accès d’API**. Cliquez sur **Télécharger**. Cette action télécharge un fichier de certificat **\<your order name>.cer**. **Enregistrez** ce fichier. Vous installerez ce certificat sur l’ordinateur hôte ou client que vous utiliserez pour vous connecter à l’appareil.

    ![Télécharger le certificat dans le portail Azure](media/data-box-deploy-copy-data-via-rest/download-cert-1.png)

### <a name="import-certificate"></a>Importation d’un certificat

L’accès au stockage Blob Data Box sur HTTPS nécessite un certificat TLS/SSL pour l’appareil. La façon dont ce certificat est mis à la disposition de l’application cliente varie en fonction de l’application, du système d’exploitation et de la distribution. Certaines applications peuvent accéder au certificat après son importation dans le magasin de certificats du système, alors que d’autres n’utilisent pas ce mécanisme.

Des informations spécifiques à certaines applications sont mentionnées dans cette section. Pour plus d’informations sur les autres applications, consultez la documentation propre à chaque application et au système d’exploitation utilisé.

Suivez ces étapes pour importer le fichier `.cer` dans le magasin racine d’un client Windows ou Linux. Sur un système Windows, vous pouvez utiliser Windows PowerShell ou l’interface utilisateur de Windows Server pour importer et installer le certificat sur votre système.

#### <a name="use-windows-powershell"></a>Utiliser Windows PowerShell

1. Démarrez une session Windows PowerShell en tant qu’administrateur.
2. À l’invite de commandes, tapez :

    ```
    Import-Certificate -FilePath C:\temp\localuihttps.cer -CertStoreLocation Cert:\LocalMachine\Root
    ```

#### <a name="use-windows-server-ui"></a>Utiliser l’interface utilisateur de Windows Server

1. Cliquez avec le bouton droit sur le fichier `.cer` et sélectionnez **Installer le certificat**. Cette action démarre l’Assistant Importation de certificat.
2. Pour **Emplacement du magasin**, sélectionnez **Ordinateur local**, puis cliquez sur **Suivant**.

    ![Assistant Importation de certificat, Windows Server](media/data-box-deploy-copy-data-via-rest/import-cert-ws-1.png)

3. Sélectionnez **Placer tous les certificats dans le magasin suivant**, puis cliquez sur **Parcourir**. Accédez au magasin racine de l’hôte distant, puis cliquez sur **Suivant**.

    ![Assistant Importation de certificat, Magasin de certificats](media/data-box-deploy-copy-data-via-rest/import-cert-ws-2.png)

4. Cliquez sur **Terminer**. Un message indiquant que l’importation a réussi s’affiche.

    ![Assistant Importation de certificat, terminer l’importation](media/data-box-deploy-copy-data-via-rest/import-cert-ws-3.png)

#### <a name="use-a-linux-system"></a>Utiliser un système Linux

La méthode d’importation d’un certificat varie selon la distribution.

Plusieurs distributions, notamment Ubuntu et Debian, utilisent la commande `update-ca-certificates`.  

* Renommez le fichier de certificat codé en Base64 avec l’extension `.crt` et copiez-le dans `/usr/local/share/ca-certificates directory`.
* Exécutez la commande `update-ca-certificates`.

Les versions récentes de RHEL, Fedora et CentOS utilisent la commande `update-ca-trust`.

* Copiez le fichier de certificat dans le répertoire `/etc/pki/ca-trust/source/anchors`.
* Exécutez `update-ca-trust`.

Pour plus de détails, consultez la documentation spécifique à votre distribution.

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Ajouter l’adresse IP de l’appareil et le point de terminaison du service blob 

Suivez les mêmes étapes que celles effectuées pour [ajouter l’adresse IP de l’appareil et le point de terminaison du service blob avec une connexion sur *http*](#add-device-ip-address-and-blob-service-endpoint).

### <a name="configure-partner-software-and-verify-connection"></a>Configurer les logiciels partenaires et vérifier la connexion

Suivez les mêmes étapes que celles effectuées pour [configurer des logiciels partenaires avec une connexion sur *http*](#verify-connection-and-configure-partner-software). La seule différence est que vous devez laisser la case *Utiliser HTTP* non cochée.

## <a name="copy-data-to-data-box"></a>Copier des données sur Data Box

Une fois que vous êtes connecté au stockage Blob Data Box, l’étape suivante consiste à copier les données. Avant de copier les données, passez en revue les considérations suivantes :

* Lorsque vous copiez des données, vérifiez que la taille des données est conforme aux limites de taille spécifiées dans l’article [Azure storage and Data Box limits](data-box-limits.md) (Limitations relatives au Stockage Azure et à Data Box).
* Si les données, qui sont en cours de chargement par Data Box, sont chargées simultanément par d’autres applications en dehors de Data Box, cela peut entraîner l’échec du chargement ou des corruptions de données.

> [!IMPORTANT]
> Veillez à conserver une copie des données sources tant que vous n’avez pas la confirmation que le Data Box a transféré vos données dans Stockage Azure.

Dans ce tutoriel, nous utilisons AzCopy pour copier des données vers le stockage Blob Data Box. Vous pouvez également utiliser l’Explorateur Stockage Azure (si vous préférez un outil basé sur une interface graphique utilisateur) ou un logiciel de partenaire pour copier les données.

La procédure de copie nécessite les étapes suivantes :

* Créez un conteneur.
* Charger le contenu d’un dossier dans le Stockage Blob Data Box
* Charger les fichiers modifiés dans le Stockage Blob Data Box

Chacune de ces étapes est décrite en détail dans les sections suivantes.

### <a name="create-a-container"></a>Créez un conteneur.

La première étape consiste à créer un conteneur, car les objets blob sont toujours chargés dans un conteneur. Les conteneurs organisent des groupes d’objets blob comme des fichiers dans des dossiers sur votre ordinateur. Pour créer un conteneur d’objets blob, effectuez les étapes suivantes :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage dans lequel vous souhaitez créer le conteneur d’objets blob.
3. Cliquez avec le bouton droit sur **Conteneurs d’objets blob** puis, dans le menu contextuel, sélectionnez **Créer un conteneur d’objets blob**.

   ![Menu contextuel Conteneurs d’objets blob, Créer un conteneur d’objets blob](media/data-box-deploy-copy-data-via-rest/create-blob-container-1.png)

4. Une zone de texte apparaît sous le dossier **Conteneurs d’objets blob**. Entrez le nom de votre conteneur d’objets blob. Pour obtenir des informations sur les règles et restrictions applicables aux noms de conteneurs d’objets blob, consultez [Créer le conteneur et définir les autorisations](../storage/blobs/storage-quickstart-blobs-dotnet.md).
5. Appuyez sur **Entrée** pour créer le conteneur d’objets blob, ou sur **ÉCHAP** pour annuler. Une fois le conteneur créé, il apparaît sous le dossier **Conteneurs d’objets blob** correspondant au compte de stockage sélectionné.

   ![Conteneur d’objets blob créé](media/data-box-deploy-copy-data-via-rest/create-blob-container-2.png)

### <a name="upload-contents-of-a-folder-to-data-box-blob-storage"></a>Charger le contenu d’un dossier dans le Stockage Blob Data Box

Utilisez AzCopy pour charger tous les fichiers d’un dossier dans le stockage Blob sur Windows ou Linux. Pour charger tous les objets blob d’un dossier, entrez la commande AzCopy suivante :

#### <a name="linux"></a>Linux

```azcopy
azcopy \
    --source /mnt/myfolder \
    --destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
    --dest-key <key> \
    --recursive
```

#### <a name="windows"></a>Windows

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S
```

Remplacez `<key>` par votre clé de compte. Pour obtenir votre clé de compte, dans le portail Azure, accédez à votre compte de stockage. Accédez à **Paramètres > Clés d’accès**, sélectionnez une clé, puis collez-la dans la commande AzCopy.

Si le conteneur de destination spécifié n’existe pas, AzCopy le crée et y charge le fichier. Mettez à jour le chemin source en indiquant votre répertoire de données, puis remplacez `data-box-storage-account-name` dans l’URL de destination par le nom du compte de stockage associé à votre Data Box.

Pour charger le contenu du répertoire spécifié dans le Stockage Blob de manière récursive, spécifiez l’option `--recursive` (Linux) ou `/S` (Windows). Quand vous exécutez AzCopy avec l’une de ces options, tous les sous-dossiers et leurs fichiers sont également chargés.

### <a name="upload-modified-files-to-data-box-blob-storage"></a>Charger les fichiers modifiés dans le Stockage Blob Data Box

Utilisez AzCopy pour charger des fichiers en fonction de l’heure de leur dernière modification. Pour les besoins du test, modifiez des fichiers existants ou créez-en d’autres dans votre répertoire source. Pour charger uniquement les fichiers nouveaux ou mis à jour, ajoutez le paramètre `--exclude-older` (Linux) ou `/XO` (Windows) à la commande AzCopy.

Si vous souhaitez copier uniquement les ressources de code source qui n’existent pas dans la destination, spécifiez les deux paramètres `--exclude-older` et `--exclude-newer` (Linux) ou les deux paramètres `/XO` et `/XN` (Windows) dans la commande AzCopy. AzCopy charge uniquement les données mises à jour, en se basant sur leur horodatage.

#### <a name="linux"></a>Linux

```azcopy
azcopy \
--source /mnt/myfolder \
--destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
--dest-key <key> \
--recursive \
--exclude-older
```

#### <a name="windows"></a>Windows

```azcopy
AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S /XO
```

S’il existe des erreurs pendant l’opération de copie ou de connexion, consultez [Résoudre les problèmes avec le Stockage Blob Data Box](data-box-troubleshoot-rest.md).

L’étape suivante consiste à préparer votre appareil pour l’expédition.

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter au stockage Blob Data Box par le biais de *http* ou *https*
> * Copier des données sur Data Box

Passez au tutoriel suivant pour découvrir comment renvoyer votre Data Box à Microsoft.

> [!div class="nextstepaction"]
> [Ship your Azure Data Box to Microsoft](./data-box-deploy-picked-up.md) (Expédier votre Azure Data Box à Microsoft)
