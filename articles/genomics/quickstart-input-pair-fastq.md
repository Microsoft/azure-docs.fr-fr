---
title: Soumettre un workflow à l’aide d’entrées de fichiers FASTQ
titleSuffix: Microsoft Genomics
description: Cet article montre comment soumettre un workflow dans le service Microsoft Genomics si vos fichiers d’entrée sont une paire unique de fichiers FASTQ.
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: conceptual
ms.date: 12/07/2017
ms.openlocfilehash: 3806b165e5abb661e53c6a315650d025fd42e17f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "72248558"
---
# <a name="submit-a-workflow-using-fastq-file-inputs-in-microsoft-genomics"></a>Soumettre un workflow à l’aide d’entrées de fichiers FASTQ dans Microsoft Genomics

Cet article montre comment soumettre un workflow dans le service Microsoft Genomics si vos fichiers d’entrée sont une paire unique de fichiers FASTQ. Cette rubrique suppose que vous avez déjà installé et exécuté le client `msgen`, et que vous savez comment utiliser Stockage Azure. Si vous avez correctement soumis un workflow à l’aide de l’exemple de données fourni, vous êtes prêt à appliquer la procédure décrite dans cet article. 

## <a name="set-up-upload-your-fastq-files-to-azure-storage"></a>Configurer : Charger vos fichiers FASTQ dans Stockage Azure
Supposons que vous disposiez de 2 fichiers, *reads_1.fq.gz* et *reads_2.fq.gz*, et que vous les ayez chargés dans votre compte de stockage *myaccount* dans Azure sous **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/inputs/reads_1<span></span>.fq<span></span>.gz<span></span>** and **https://<span></span>myaccount.blob.core.<span></span>windows<span></span>.net/<span></span>inputs/<span></span>reads_2.fq<span></span>.gz<span></span>** . Vous disposez de l’URL d’API et de votre clé d’accès. Vos sorties doivent être hébergées sous **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/outputs<span></span>** .


## <a name="submit-your-job-to-the-msgen-client"></a>Envoyer votre tâche au client `msgen` 

Voici le jeu minimal d’arguments qu’il vous faudra fournir au client `msgen` ; des sauts de ligne ont été ajoutés pour plus de clarté :

Pour Windows :

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads_1.fq.gz ^
  --input-blob-name-2 reads_2.fq.gz ^
  --output-storage-account-name myaccount ^
  --output-storage-account-key <storage access key to "myaccount"> ^
  --output-storage-account-container outputs
```

Pour Unix :

```
msgen submit \
  --api-url-base <Genomics API URL> \
  --access-key <Genomics access key> \
  --process-args R=b37m1 \
  --input-storage-account-name myaccount \
  --input-storage-account-key <storage access key to "myaccount"> \
  --input-storage-account-container inputs \
  --input-blob-name-1 reads_1.fq.gz \
  --input-blob-name-2 reads_2.fq.gz \
  --output-storage-account-name myaccount \
  --output-storage-account-key <storage access key to "myaccount"> \
  --output-storage-account-container outputs
```


Si vous préférez utiliser un fichier de configuration, voici à quoi il doit ressembler :

```
api_url_base:                     <Genomics API URL>
access_key:                       <Genomics access key>
process_args:                     R=b37m1
input_storage_account_name:       myaccount
input_storage_account_key:        <storage access key to "myaccount">
input_storage_account_container:  inputs
input_blob_name_1:                reads_1.fq.gz
input_blob_name_2:                reads_2.fq.gz
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

Envoyez le fichier `config.txt` avec cet appel : `msgen submit -f config.txt`

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez chargé une paire de fichiers FASTQ dans Stockage Azure et soumis un workflow dans le service Microsoft Genomics, via le client Python `msgen`. Pour en savoir plus sur la soumission du workflow et d’autres commandes pouvant être utilisées avec le service Microsoft Genomics, voir notre [FAQ](frequently-asked-questions-genomics.md). 
