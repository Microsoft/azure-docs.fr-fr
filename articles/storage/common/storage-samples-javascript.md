---
title: Exemples relatifs au service Stockage Azure en JavaScript | Microsoft Docs
description: Affichez, téléchargez et exécutez des exemples de code et des applications pour Azure Storage. Découvrez des exemples permettant de bien démarrer avec les objets blob, les files d’attente, les tables et les fichiers à l’aide des bibliothèques de clients de stockage JavaScript/Node.js.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 10/01/2020
ms.service: storage
ms.subservice: common
ms.topic: sample
ms.custom: devx-track-js
ms.openlocfilehash: d21c92d6595d2db3f9fb01ee5ddc3669a0564bab
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92359745"
---
# <a name="azure-storage-samples-using-v12-javascript-client-libraries"></a>Exemples de stockage Azure utilisant les bibliothèques clientes JavaScript v12

Les tableaux suivants donnent une vue d’ensemble de notre dépôt d’exemples et des scénarios traités dans chacun d’eux. Cliquez sur les liens pour afficher l’exemple de code correspondant dans GitHub.

> [!NOTE]
> Ces exemples utilisent la bibliothèque JavaScript v12 du stockage Azure la plus récente. Pour le code v11 hérité, consultez [Bien démarrer avec le service Azure Blob en Node.js](https://github.com/Azure-Samples/storage-blob-node-getting-started) dans le dépôt GitHub.

## <a name="blob-samples"></a>Exemples d’objets blob

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [S’authentifier au moyen d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/withConnString.js#L14)
   :::column-end:::
   :::column span="":::
      [S’authentifier au moyen d’une chaîne de connexion SAS](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/withConnString.js#L14)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [S’authentifier à l’aide d’informations d’identification de clé partagée](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/sharedKeyCred.js#L5)
   :::column-end:::
   :::column span="":::
      [S’authentifier au moyen de AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/anonymousCred.js#L18)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [S’authentifier à l’aide d’Azure Active Directory](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/azureAdAuth.js#L47)
   :::column-end:::
   :::column span="":::
      [S’authentifier au moyen d’un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/proxyAuth.js#L28)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Se connecter à l’aide d’un pipeline personnalisé](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/customPipeline.js#L26)
   :::column-end:::
:::row-end:::

### <a name="blob-service"></a>Service d'objets blob

:::row:::
   :::column span="2":::
      [Créer un client du service BLOB à l’aide d’une URL SAS](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L39)
   :::column-end:::
:::row-end:::

### <a name="container"></a>Conteneur

:::row:::
   :::column span="":::
      [Créer un conteneur](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L53)
   :::column-end:::
   :::column span="":::
      [Créer un conteneur à l’aide d’informations d’identification de clé partagée](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L38)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des conteneurs](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L48)
   :::column-end:::
   :::column span="":::
      [Lister des conteneurs à l’aide d’un itérateur](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-containers.js#L28)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des conteneurs par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-containers.js#L53)
   :::column-end:::
   :::column span="":::
      [Supprimer un conteneur](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L82)
   :::column-end:::
:::row-end:::

### <a name="blob"></a>Objet blob

:::row:::
   :::column span="":::
      [Créer un objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L60)
   :::column-end:::
   :::column span="":::
      [Lister des objets blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L67)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Télécharger un objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L73)
   :::column-end:::
   :::column span="":::
      [Lister des objets blob à l’aide d’un itérateur](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs.js#L41)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les objets blob par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs.js#L66)
   :::column-end:::
   :::column span="":::
      [Lister les objets blob par hiérarchie](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs-hierarchy.js#L70)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les objets blob sans utiliser await](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-without-await.js#L42)
   :::column-end:::
   :::column span="":::
      [Créer un instantané d’objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L56)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Télécharger un instantané d’objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L61)
   :::column-end:::
   :::column span="":::
      [Charger en parallèle un flux sur un objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L74)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Télécharger en parallèle un objet blob de blocs](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L99)
   :::column-end:::
   :::column span="":::
      [Définir le niveau d’accès sur un objet blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L118)
   :::column-end:::
:::row-end:::

### <a name="troubleshooting"></a>Dépannage

:::row:::
   :::column span="2":::
      [Déclencher une erreur récupérable à l’aide d’un client de conteneur](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/errorsAndResponses.js#L33)
   :::column-end:::
:::row-end:::


## <a name="data-lake-storage-gen2-samples"></a>Exemples de Data Lake Storage Gen2

:::row:::
   :::column span="":::
      [Créer un client du service Data Lake](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L36)
   :::column-end:::
   :::column span="":::
      [Créer un système de fichiers](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L47)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des systèmes de fichiers](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L42)
   :::column-end:::
   :::column span="":::
      [Créer un fichier](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L54)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les chemins dans un système de fichiers](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L63)
   :::column-end:::
   :::column span="":::
      [Télécharger un fichier](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L69)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Supprimer un système de fichiers](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L78)
   :::column-end:::
:::row-end:::

## <a name="azure-files-samples"></a>Exemples Azure Files

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [S’authentifier au moyen d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/withConnString.js)
   :::column-end:::
   :::column span="":::
      [S’authentifier à l’aide d’informations d’identification de clé partagée](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/sharedKeyCred.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [S’authentifier au moyen de AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/anonymousCred.js)
   :::column-end:::
   :::column span="":::
      [Se connecter à l’aide d’un pipeline personnalisé](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/customPipeline.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Se connecter à l’aide d’un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/proxyAuth.js)
   :::column-end:::
:::row-end:::

### <a name="share"></a>Partager

:::row:::
   :::column span="":::
      [Créer un partage](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L48)
   :::column-end:::
   :::column span="":::
      [Lister des partages](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-shares.js#L27)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les partages par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-shares.js#L51)
   :::column-end:::
   :::column span="":::
      [Supprimer un partage](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L104)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Répertoire

:::row:::
   :::column span="":::
      [Créer un répertoire](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L54)
   :::column-end:::
   :::column span="":::
      [Lister des fichiers et des répertoires](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-files-and-directories.js#L56)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Lister les fichiers et les répertoires par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-files-and-directories.js#L90)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Fichier

:::row:::
   :::column span="":::
      [Charger en parallèle un fichier](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L65)
   :::column-end:::
   :::column span="":::
      [Charger en parallèle un flux lisible](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L74)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Télécharger en parallèle un fichier](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L93)
   :::column-end:::
   :::column span="":::
      [Lister des descripteurs de fichiers](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-handles.js#L43)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Lister les descripteurs de fichiers par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-handles.js#L79)
   :::column-end:::
:::row-end:::

## <a name="queue-samples"></a>Exemples de files d’attente

### <a name="authentication"></a>Authentification

:::row:::
   :::column span="":::
      [S’authentifier au moyen d’une chaîne de connexion](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/withConnString.js)
   :::column-end:::
   :::column span="":::
      [S’authentifier à l’aide d’informations d’identification de clé partagée](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/sharedKeyCred.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [S’authentifier au moyen de AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/anonymousCred.js)
   :::column-end:::
   :::column span="":::
      [Se connecter à l’aide d’un pipeline personnalisé](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/customPipeline.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Se connecter à l’aide d’un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/proxyAuth.js)
   :::column-end:::
   :::column span="":::
      [S’authentifier à l’aide d’Azure Active Directory](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/azureAdAuth.js)
   :::column-end:::
:::row-end:::

### <a name="queue-service"></a>Service File d’attente

:::row:::
   :::column span="2":::
      [Créer un client du service de file d’attente](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L42)
   :::column-end:::
:::row-end:::

### <a name="queue"></a>File d'attente

:::row:::
   :::column span="":::
      [Créer une file d’attente](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L54)
   :::column-end:::
   :::column span="":::
      [Lister des files d’attente](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/iterators.js#L27)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister les files d’attente par page](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/iterators.js#L51)
   :::column-end:::
   :::column span="":::
      [Supprimer une file d’attente](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L89)
   :::column-end:::
:::row-end:::

### <a name="message"></a>Message

:::row:::
   :::column span="":::
      [Envoyer un message dans une file d’attente](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L62)
   :::column-end:::
   :::column span="":::
      [Afficher un aperçu des messages](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L68)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Recevoir des messages](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L76)
   :::column-end:::
   :::column span="":::
      [Supprimer des messages](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L80)
   :::column-end:::
:::row-end:::

## <a name="table-samples-v11"></a>Exemples de tables (v11)

:::row:::
   :::column span="":::
      [Traiter par lots des entités](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L87)
   :::column-end:::
   :::column span="":::
      [Créer une table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L41)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Supprimer une entité/une table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L67)
   :::column-end:::
   :::column span="":::
      [Insérer/fusionner/remplacer une entité](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L49)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Lister des tables](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L63)
   :::column-end:::
   :::column span="":::
      [Interroger des entités](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L59)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Interroger des tables](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L140)
   :::column-end:::
   :::column span="":::
      [Requête de plage](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L102)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Signature d’accès partagé (SAP)](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L87)
   :::column-end:::
   :::column span="":::
      [Liste de contrôle d’accès d’une table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L255)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Règles CORS (Cross-Origin Resource Sharing) de table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L149)
   :::column-end:::
   :::column span="":::
      [Propriétés de table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L188)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Statistiques de table](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L243)
   :::column-end:::
   :::column span="":::
      [Mettre à jour une entité](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L49)
   :::column-end:::
:::row-end:::

## <a name="azure-code-sample-libraries"></a>Bibliothèques d’exemples de code Azure

Pour afficher l’intégralité des bibliothèques d’exemples JavaScript, accédez à :

* [Exemples de code BLOB Azure](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-blob/samples/javascript)
* [Exemples de code Azure Data Lake](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples/javascript)
* [Exemples de code Azure Files](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-share/samples/javascript)
* [Exemples de code de file d’attente Azure](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue/samples/javascript)

Vous pouvez parcourir et cloner le dépôt GitHub pour chaque bibliothèque.

## <a name="getting-started-guides"></a>Guides de prise en main

Consultez les guides suivants si vous recherchez des instructions sur l’installation et la prise en main des bibliothèques clientes de stockage Azure.

* [Bien démarrer avec le service Blob Azure en JavaScript](../blobs/storage-quickstart-blobs-nodejs.md)
* [Bien démarrer avec le service File d’attente Azure en JavaScript](../queues/storage-quickstart-queues-nodejs.md)
* [Bien démarrer avec le service Table Azure en JavaScript](../../cosmos-db/table-storage-how-to-use-nodejs.md)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les exemples pour d’autres langages :

* .NET : [Exemples relatifs au service Stockage Azure avec .NET](storage-samples-dotnet.md)
* Java : [Exemples relatifs au service Stockage Azure avec Java](storage-samples-java.md)
* Python : [Exemples relatifs au service Stockage Azure avec Python](storage-samples-python.md)
* C++ : [Exemples relatifs au service Stockage Azure avec C++](storage-samples-c-plus-plus.md)
* Tous les autres langages : [Exemples relatifs à Stockage Azure](storage-samples.md)
