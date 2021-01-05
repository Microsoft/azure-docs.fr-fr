---
title: Exemples relatifs au service Stockage Azure avec Python | Microsoft Docs
description: Affichez, téléchargez et exécutez des exemples de code et des applications pour Azure Storage. Découvrez des exemples permettant de bien démarrer avec les objets blob, les files d’attente, les tables et les fichiers à l’aide des bibliothèques de client de stockage Python.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 10/01/2020
ms.service: storage
ms.subservice: common
ms.topic: sample
ms.custom: devx-track-python
ms.openlocfilehash: f57fa1b668cacdaaceb2026f9e5d7fff0cc56d7f
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371356"
---
# <a name="azure-storage-samples-using-v12-python-client-libraries"></a>Exemples de stockage Azure utilisant les bibliothèques clientes Python v12

Les tableaux suivants donnent une vue d’ensemble de notre dépôt d’exemples et des scénarios traités dans chacun d’eux. Cliquez sur les liens pour afficher l’exemple de code correspondant dans GitHub.

> [!NOTE]
> Ces exemples utilisent la bibliothèque .NET v12 du stockage Azure la plus récente. Pour le code v2.1 hérité, consultez [Stockage Azure : Bien démarrer avec Stockage Azure en Python](https://github.com/Azure-Samples/storage-blob-python-getting-started) dans le dépôt GitHub.

## <a name="blob-samples"></a>Exemples d’objets blob

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [Créer un client du service BLOB à l’aide d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L45)
   :::column-end:::
   :::column span="":::
      [Créer un client de conteneur à l’aide d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L50)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client d’objet blob à l’aide d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L56)
   :::column-end:::
   :::column span="":::
      [Créer un client du service BLOB à l’aide d’une clé d’accès partagé](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L66)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client d’objet blob à partir d’une URL](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L75)
   :::column-end:::
   :::column span="":::
      [Créer un client d’objet blob à partir d’une URL SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L80)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client du service BLOB à l’aide de ClientSecretCredential](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L88)
   :::column-end:::
   :::column span="":::
      [Créer un jeton SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L110)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client du service BLOB à l’aide d’une identité Azure](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_authentication.py#L125)
   :::column-end:::
   :::column span="":::
      [Créer un instantané d’objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L56)
   :::column-end:::
:::row-end:::

### <a name="blob-service"></a>Service d'objets blob

:::row:::
   :::column span="":::
      [Obtenir des informations de compte du service BLOB](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L30)
   :::column-end:::
   :::column span="":::
      [Définir les propriétés du service BLOB](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L41)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés du service BLOB](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L61)
   :::column-end:::
   :::column span="":::
      [Obtenir les statistiques du service BLOB](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L71)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un conteneur à l’aide d’un client du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L82)
   :::column-end:::
   :::column span="":::
      [Lister des conteneurs](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L90)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un conteneur à l’aide d’un client du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L103)
   :::column-end:::
   :::column span="":::
      [Obtenir un client de conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L117)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Obtenir un client d’objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_service.py#L130)
   :::column-end:::
:::row-end:::

### <a name="container"></a>Conteneur

:::row:::
   :::column span="":::
      [Créer un client de conteneur à partir du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L35)
   :::column-end:::
   :::column span="":::
      [Créer un client de conteneur à l’aide d’une URL SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L44)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un conteneur à l’aide d’un client de conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L52)
   :::column-end:::
   :::column span="":::
      [Obtenir les propriétés d’un conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L56)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un conteneur à l’aide d’un client de conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L61)
   :::column-end:::
   :::column span="":::
      [Acquérir le bail sur un conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L80)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Définir les métadonnées d’un conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L101)
   :::column-end:::
   :::column span="":::
      [Définir une stratégie d’accès pour conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L128)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir une stratégie d’accès pour conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L141)
   :::column-end:::
   :::column span="":::
      [Générer un jeton SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L145)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client de conteneur à l’aide d’un jeton SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L158)
   :::column-end:::
   :::column span="":::
      [Charger un objet blob dans un conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L182)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les objets blob dans un conteneur](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L189)
   :::column-end:::
   :::column span="":::
      [Obtenir un client d’objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_containers.py#L213)
   :::column-end:::
:::row-end:::

### <a name="blob"></a>Objet blob

:::row:::
   :::column span="":::
      [Charger un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_hello_world.py#L68)
   :::column-end:::
   :::column span="":::
      [Télécharger un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_hello_world.py#L74)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_hello_world.py#L80)
   :::column-end:::
   :::column span="":::
      [Annuler la suppression d’un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L100)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés d’un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L105)
   :::column-end:::
   :::column span="":::
      [Supprimer plusieurs objets blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L133)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Copier un objet blob à partir d’une URL](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L193)
   :::column-end:::
   :::column span="":::
      [Abandonner la copie d’un blob à partir d’une URL](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L205)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Acquérir le bail sur un objet blob](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/samples/blob_samples_common.py#L167)
   :::column-end:::
:::row-end:::

## <a name="data-lake-storage-gen2-samples"></a>Exemples de Data Lake Storage Gen2

### <a name="data-lake-service"></a>Service Data Lake

:::row:::
   :::column span="2":::
      [Créer un client du service Data Lake](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L64)
   :::column-end:::
:::row-end:::

### <a name="file-system"></a>Système de fichiers

:::row:::
   :::column span="":::
      [Créer un client du système de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L74)
   :::column-end:::
   :::column span="":::
      [Supprimer un système de fichiers](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L81)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Répertoire

:::row:::
   :::column span="":::
      [Créer un client de répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L14)
   :::column-end:::
   :::column span="":::
      [Obtenir les autorisations d’un répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L19)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Définir les autorisations d’un répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L23)
   :::column-end:::
   :::column span="":::
      [Renommer un répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_directory.py#L19)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés d’un répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_directory.py#L25)
   :::column-end:::
   :::column span="":::
      [Supprimer le répertoire](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_directory.py#L29)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Fichier

:::row:::
   :::column span="":::
      [Créer un client de fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_upload_download.py#L13)
   :::column-end:::
   :::column span="":::
      [Créer un fichier](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L45)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir des autorisations de fichier](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L35)
   :::column-end:::
   :::column span="":::
      [Définir les autorisations d’un fichier](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control.py#L38)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Ajouter des données à un fichier](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_upload_download.py#L18)
   :::column-end:::
   :::column span="":::
      [Lire des données à partir d’un fichier](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_upload_download.py#L29)
   :::column-end:::
:::row-end:::

## <a name="azure-files-samples"></a>Exemples Azure Files

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [Créer un client du service de partage à partir de la chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_authentication.py#L42)
   :::column-end:::
   :::column span="":::
      [Créer un client du service de partage à partir d’un compte et d’une clé d’accès](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_authentication.py#L49)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Générer un jeton SAS](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_authentication.py#L59)
   :::column-end:::
:::row-end:::

### <a name="file-service"></a>Service Fichier

:::row:::
   :::column span="":::
      [Définir les propriétés du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L39)
   :::column-end:::
   :::column span="":::
      [Obtenir les propriétés du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L68)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer des partages à l’aide d’un client du service de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L77)
   :::column-end:::
   :::column span="":::
      [Lister des partages à l’aide d’un client du service de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L81)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer des partages à l’aide d’un client du service de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L91)
   :::column-end:::
:::row-end:::

### <a name="file-share"></a>Partage de fichiers

:::row:::
   :::column span="":::
      [Créer un client de partage à partir d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L52)
   :::column-end:::
   :::column span="":::
      [Télécharger un client de partage](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_service.py#L96)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un partage à l’aide du client de partage de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L39)
   :::column-end:::
   :::column span="":::
      [Créer un instantané de partage](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L43)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un partage à l’aide du client de partage de fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L47)
   :::column-end:::
   :::column span="":::
      [Définir le quota de partage](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L61)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Définir les métadonnées d’un partage](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L66)
   :::column-end:::
   :::column span="":::
      [Obtenir les propriétés d’un partage](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_hello_world.py#L47)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Répertoire

:::row:::
   :::column span="":::
      [Créer un répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L45)
   :::column-end:::
   :::column span="":::
      [Charger un fichier dans un répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L49)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un fichier d’un répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L55)
   :::column-end:::
   :::column span="":::
      [Supprimer le répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L60)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un sous-répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L80)
   :::column-end:::
   :::column span="":::
      [Lister des répertoires et des fichiers](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L96)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un sous-répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L104)
   :::column-end:::
   :::column span="":::
      [Obtenir un client de sous-répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_directory.py#L121)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Lister les fichiers dans un répertoire](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_share.py#L87)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Fichier

:::row:::
   :::column span="":::
      [Créer un client de fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_hello_world.py#L65)
   :::column-end:::
   :::column span="":::
      [Créer un fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_client.py#L48)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Charger un fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_client.py#L54)
   :::column-end:::
   :::column span="":::
      [Télécharger un fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_client.py#L60)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un fichier](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_client.py#L67)
   :::column-end:::
   :::column span="":::
      [Copier un fichier à partir d’une URL](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-share/samples/file_samples_client.py#L101)
   :::column-end:::
:::row-end:::

## <a name="queue-samples"></a>Exemples de files d’attente

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [S’authentifier au moyen d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_authentication.py#L50)
   :::column-end:::
   :::column span="":::
      [Créer un jeton du service de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_authentication.py#L69)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un client de file d’attente à partir d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L35)
   :::column-end:::
   :::column span="":::
      [Générer un jeton SAS de client de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L61)
   :::column-end:::
:::row-end:::

### <a name="queue-service"></a>Service File d’attente

:::row:::
   :::column span="":::
      [Créer un client du service de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_authentication.py#L60)
   :::column-end:::
   :::column span="":::
      [Définir les propriétés du service de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L35)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés du service de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L67)
   :::column-end:::
   :::column span="":::
      [Créer une file d’attente à l’aide d’un client du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L76)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer une file d’attente à l’aide d’un client du service](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L94)
   :::column-end:::
:::row-end:::

### <a name="queue"></a>File d'attente

:::row:::
   :::column span="":::
      [Créer un client de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L72)
   :::column-end:::
   :::column span="":::
      [Définir les métadonnées d’une file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L95)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés d’une file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L100)
   :::column-end:::
   :::column span="":::
      [Créer une file d’attente à l’aide d’un client de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_hello_world.py#L45)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer une file d’attente à l’aide d’un client de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_hello_world.py#L62)
   :::column-end:::
   :::column span="":::
      [Lister des files d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L81)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Obtenir un client de file d’attente](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_service.py#L103)
   :::column-end:::
:::row-end:::

### <a name="message"></a>Message

:::row:::
   :::column span="":::
      [Envoyer des messages](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L117)
   :::column-end:::
   :::column span="":::
      [Recevoir des messages](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L125)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Afficher un aperçu d’un message](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L197)
   :::column-end:::
   :::column span="":::
      [Mettre à jour un message](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L222)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un message](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L165)
   :::column-end:::
   :::column span="":::
      [Effacer des messages](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L173)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Définir la stratégie d’accès des messages](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-queue/samples/queue_samples_message.py#L47)
   :::column-end:::
:::row-end:::


## <a name="table-samples-sdk-v21"></a>Exemples de tables (Kit SDK v2.1)

:::row:::
   :::column span="":::
      [Créer une table](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py#L46)
   :::column-end:::
   :::column span="":::
      [Supprimer une entité/une table](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py#L79)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Insérer/fusionner/remplacer une entité](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py#L57)
   :::column-end:::
   :::column span="":::
      [Interroger des entités](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py#L62)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Interroger des tables](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py)
   :::column-end:::
   :::column span="":::
      [Propriétés/liste de contrôle d’accès d’une table](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_advanced_samples.py#L138)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Mettre à jour une entité](https://github.com/Azure-Samples/storage-table-python-getting-started/blob/master/table_basic_samples.py#L68)
   :::column-end:::
:::row-end:::

## <a name="azure-code-sample-libraries"></a>Bibliothèques d’exemples de code Azure

Pour voir l’intégralité des bibliothèques d’exemples Python, accédez à :

* [Exemples de code BLOB Azure](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-blob/samples)
* [Exemples de code Azure Data Lake](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)
* [Exemples de code Azure Files](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-share/samples)
* [Exemples de code de file d’attente Azure](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue/samples)

Vous pouvez parcourir et cloner le dépôt GitHub pour chaque bibliothèque.

## <a name="getting-started-guides"></a>Guides de prise en main

Consultez les guides suivants si vous recherchez des instructions sur l’installation et le démarrage des bibliothèques clientes Stockage Azure.

* [Getting Started with Azure Blob Service in Python](../blobs/storage-quickstart-blobs-python.md)
* [Getting Started with Azure Queue Service in Python](../queues/storage-quickstart-queues-python.md)
* [Getting Started with Azure Table Service in Python](../../cosmos-db/table-storage-how-to-use-python.md)
* [Getting Started with Azure Files Service in Python](../files/storage-python-how-to-use-file-storage.md)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les exemples pour d’autres langages :

* .NET : [Exemples relatifs au service Stockage Azure avec .NET](storage-samples-dotnet.md)
* Java : [Exemples relatifs au service Stockage Azure avec Java](storage-samples-java.md)
* JavaScript/Node.js : [Exemples relatifs au service Stockage Azure en JavaScript](storage-samples-javascript.md)
* C++ : [Exemples relatifs au service Stockage Azure avec C++](storage-samples-c-plus-plus.md)
* Tous les autres langages : [Exemples relatifs à Stockage Azure](storage-samples.md)
