---
title: Exemples de stockage Azure avec Java| Microsoft Docs
description: Affichez, téléchargez et exécutez des exemples de code et des applications pour Azure Storage. Découvrez des exemples de mise en route d’objets blob, de files d’attente, de tables et de fichiers à l’aide des bibliothèques clientes de stockage Java.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 10/01/2020
ms.service: storage
ms.subservice: common
ms.topic: sample
ms.openlocfilehash: 9e3cd6be0da7c6d9401945293283de874c7cd866
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92359762"
---
# <a name="azure-storage-samples-using-v12-java-client-libraries"></a>Exemples de stockage Azure utilisant les bibliothèques clientes Java v12

Le tableau suivant fournit une vue d’ensemble de notre référentiel d’exemples et des scénarios traités dans chaque exemple. Cliquez sur les liens pour afficher l’exemple de code correspondant dans GitHub.

> [!NOTE]
> Ces exemples utilisent la bibliothèque Java v12 du stockage Azure la plus récente. Pour le code v8 hérité, consultez [Bien démarrer avec le service Azure Blob en Java](https://github.com/Azure-Samples/storage-blob-java-getting-started) dans le dépôt GitHub.

## <a name="blob-samples"></a>Exemples d’objets blob

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [S’authentifier à l’aide d’informations d’identification de clé partagée](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L38)
   :::column-end:::
   :::column span="":::
      [S’authentifier à l’aide de l’identité Azure](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/AzureIdentityExample.java#L10)
   :::column-end:::
:::row-end:::

### <a name="blob-service"></a>Service d'objets blob

:::row:::
   :::column span="":::
      [Créer un client du service BLOB](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L49)
   :::column-end:::
   :::column span="":::
      [Lister des conteneurs](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/ListContainersExample.java#L10)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer des conteneurs](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/ListContainersExample.java#L52)
   :::column-end:::
:::row-end:::

### <a name="batching"></a>Traitement par lot

:::row:::
   :::column span="":::
      [Créer un client de lot d’objets blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob-batch/src/samples/java/com/azure/storage/blob/batch/ReadmeSamples.java#L41)
   :::column-end:::
   :::column span="":::
      [Supprimer en bloc des objets blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob-batch/src/samples/java/com/azure/storage/blob/batch/ReadmeSamples.java#L45)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Définir le niveau d’accès sur un lot d’objets blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob-batch/src/samples/java/com/azure/storage/blob/batch/ReadmeSamples.java#L51)
   :::column-end:::
:::row-end:::

### <a name="container"></a>Conteneur

:::row:::
   :::column span="":::
      [Créer un client de conteneur](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L57)
   :::column-end:::
   :::column span="":::
      [Créer un conteneur](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L64)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des objets blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L112)
   :::column-end:::
   :::column span="":::
      [Supprimer un conteneur](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L123)
   :::column-end:::
:::row-end:::

### <a name="blob"></a>Objet blob

:::row:::
   :::column span="":::
      [Charger un objet blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L79)
   :::column-end:::
   :::column span="":::
      [Télécharger un objet blob](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L86)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un blob.](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/BasicExample.java#L118)
   :::column-end:::
   :::column span="":::
      [Charger un objet blob à partir d’un fichier volumineux](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/FileTransferExample.java#L95)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Télécharger un objet blob volumineux dans un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/FileTransferExample.java#L100)
   :::column-end:::
:::row-end:::

### <a name="troubleshooting"></a>Dépannage
:::row:::
   :::column span="2":::
      [Déclencher une erreur récupérable à l’aide d’un client de conteneur](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob/StorageErrorHandlingExample.java#L11)
   :::column-end:::
:::row-end:::

## <a name="data-lake-storage-gen2-samples"></a>Exemples de Data Lake Storage Gen2

### <a name="data-lake-service"></a>Service Data Lake
:::row:::
   :::column span="":::
      [Créer un client du service Data Lake](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L48)
   :::column-end:::
   :::column span="":::
      [Créer un client du système de fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L57)
   :::column-end:::
:::row-end:::

### <a name="file-system"></a>Système de fichiers
:::row:::
   :::column span="":::
      [Créer un système de fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L64)
   :::column-end:::
   :::column span="":::
      [Créer un répertoire](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L68)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un fichier et un sous-répertoire](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L73)
   :::column-end:::
   :::column span="":::
      [Créer un client de fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L83)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les chemins dans un système de fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L131)
   :::column-end:::
   :::column span="":::
      [Supprimer un système de fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L142)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Lister les systèmes de fichiers dans un compte de stockage Azure](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/ListFileSystemsExample.java#L10)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Répertoire

:::row:::
   :::column span="":::
      [Créer un client de répertoire](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L31)
   :::column-end:::
   :::column span="":::
      [Créer un répertoire parent](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L37)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un répertoire enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L44)
   :::column-end:::
   :::column span="":::
      [Créer un fichier dans un répertoire enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L52)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés d’un répertoire](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L68)
   :::column-end:::
   :::column span="":::
      [Supprimer un répertoire enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L83)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer un dossier parent](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/DirectoryExample.java#L90)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Fichier

:::row:::
   :::column span="":::
      [Créer un fichier à l’aide d’un client de fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L93)
   :::column-end:::
   :::column span="":::
      [Supprimer un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/BasicExample.java#L137)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Définir les contrôles d’accès sur un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/GetSetAccessControlExample.java#L82)
   :::column-end:::
   :::column span="":::
      [Obtenir les contrôles d’accès sur un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake/GetSetAccessControlExample.java#L104)
   :::column-end:::
:::row-end:::


## <a name="azure-file-samples"></a>Exemples de fichiers Azure

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="2":::
      [S’authentifier au moyen d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareServiceSample.java#L27)
   :::column-end:::
:::row-end:::

### <a name="file-service"></a>Service Fichier

:::row:::
   :::column span="":::
      [Créer des partages de fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareServiceSample.java#L31)
   :::column-end:::
   :::column span="":::
      [Obtenir des propriétés](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareServiceSample.java#L40)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des partages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareServiceSample.java#L49)
   :::column-end:::
   :::column span="":::
      [Supprimer des partages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareServiceSample.java#L49)
   :::column-end:::
:::row-end:::

### <a name="file-share"></a>Partage de fichiers

:::row:::
   :::column span="":::
      [Créer un client de partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L29)
   :::column-end:::
   :::column span="":::
      [Créer un partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L40)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un instantané de partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L55)
   :::column-end:::
   :::column span="":::
      [Créer un répertoire à l’aide d’un client de partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L63)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Obtenir les propriétés d’un partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L72)
   :::column-end:::
   :::column span="":::
      [Obtenir un répertoire racine et lister les répertoires](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L100)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer un partage](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/ShareSample.java#L151)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Répertoire

:::row:::
   :::column span="":::
      [Créer un répertoire parent](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L35)
   :::column-end:::
   :::column span="":::
      [Créer un répertoire enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L42)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Créer un fichier dans un répertoire enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L50)
   :::column-end:::
   :::column span="":::
      [Lister des répertoires et des fichiers](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L66)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un dossier enfant](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L90)
   :::column-end:::
   :::column span="":::
      [Supprimer un dossier parent](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/DirectorySample.java#L97)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Fichier

:::row:::
   :::column span="":::
      [Créer un client de fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/FileSample.java#L45)
   :::column-end:::
   :::column span="":::
      [Charger un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/FileSample.java#L90)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Télécharger un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/FileSample.java#L100)
   :::column-end:::
   :::column span="":::
      [Obtenir les propriétés d’un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/FileSample.java#L120)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer un fichier](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share/FileSample.java#L128)
   :::column-end:::
:::row-end:::


## <a name="queue-samples"></a>Exemples de files d’attente

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="2":::
      [S’authentifier au moyen d’un jeton SAS](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/QueueServiceSamples.java#L17)
   :::column-end:::
:::row-end:::

### <a name="queue-service"></a>Service File d’attente

:::row:::
   :::column span="":::
      [Créer une file d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/QueueServiceSamples.java#L20)
   :::column-end:::
   :::column span="":::
      [Lister des files d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/QueueServiceSamples.java#L22)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer des files d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/QueueServiceSamples.java#L27)
   :::column-end:::
:::row-end:::

### <a name="queue"></a>File d'attente

:::row:::
   :::column span="":::
      [Créer un client de file d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L25)
   :::column-end:::
   :::column span="":::
      [Ajouter des messages à une file d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L27)
   :::column-end:::
:::row-end:::

### <a name="message"></a>Message

:::row:::
   :::column span="":::
      [Obtenir le nombre de messages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L34)
   :::column-end:::
   :::column span="":::
      [Afficher un aperçu des messages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L37)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Recevoir des messages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L41)
   :::column-end:::
   :::column span="":::
      [Mettre à jour un message](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L45)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer le premier message](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L50)
   :::column-end:::
   :::column span="":::
      [Effacer tous les messages](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L59)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer une file d’attente](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue/MessageSamples.java#L64)
   :::column-end:::
:::row-end:::

## <a name="table-samples-v11"></a>Exemples de tables (v11)

:::row:::
   :::column span="":::
      [Créer une table](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L50)
   :::column-end:::
   :::column span="":::
      [Supprimer une entité/une table](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L109)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Insérer/fusionner/remplacer une entité](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L195)
   :::column-end:::
   :::column span="":::
      [Interroger des entités](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L234)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Interroger des tables](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L262)
   :::column-end:::
   :::column span="":::
      [Propriétés/liste de contrôle d’accès d’une table](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableAdvanced.java#L49)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Mettre à jour une entité](https://github.com/Azure-Samples/storage-table-java-getting-started/blob/master/src/main/java/com/microsoft/azure/cosmosdb/tablesample/TableBasics.java#L76)
   :::column-end:::
:::row-end:::

## <a name="azure-code-sample-libraries"></a>Bibliothèques d’exemples de code Azure

Pour afficher l’intégralité des bibliothèques d’exemples Java, accédez à :

* [Exemples de code d’objet blob Azure](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob)
* [Exemples de code Azure Data Lake](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/src/samples/java/com/azure/storage/file/datalake)
* [Exemples de code Azure Files](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-share/src/samples/java/com/azure/storage/file/share)
* [Exemples de code de file d’attente Azure](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue)

Vous pouvez parcourir et cloner le dépôt GitHub pour chaque bibliothèque.

## <a name="getting-started-guides"></a>Guides de prise en main

Consultez les guides suivants si vous recherchez des instructions sur l’installation et la prise en main des bibliothèques clientes de stockage Azure.

* [Getting Started with Azure Blob Service in Java (Prise en main du service Azure Blob en Java)](../blobs/storage-quickstart-blobs-java.md)
* [Getting Started with Azure Queue Service in Java (Prise en main du service de File d’attente Azure en Java)](../queues/storage-quickstart-queues-java.md)
* [Getting Started with Azure Table Service in Java (Prise en main du service de Table Azure en Java)](../../cosmos-db/table-storage-how-to-use-java.md)
* [Getting Started with Azure File Service in Java (Prise en main du service de fichiers Azure en Java)](../files/storage-java-how-to-use-file-storage.md)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les exemples pour d’autres langages :

* .NET : [Exemples relatifs au service Stockage Azure avec .NET](storage-samples-dotnet.md)
* Python : [Exemples relatifs au service Stockage Azure avec Python](storage-samples-python.md)
* JavaScript/Node.js : [Exemples relatifs au service Stockage Azure en JavaScript](storage-samples-javascript.md)
* C++ : [Exemples relatifs au service Stockage Azure avec C++](storage-samples-c-plus-plus.md)
* Tous les autres langages : [Exemples relatifs à Stockage Azure](storage-samples.md)
