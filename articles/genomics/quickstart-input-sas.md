---
title: Workflow utilisant des signatures d’accès partagé (SAP)
titleSuffix: Microsoft Genomics
description: Cet article montre comment envoyer un workflow au service Microsoft Genomics à l’aide de signatures d’accès partagé (SAP) au lieu de clés de compte de stockage.
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: conceptual
ms.date: 03/02/2018
ms.openlocfilehash: 82f5e8b4a0c06517381857f0d914bcb65ba41d35
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394609"
---
# <a name="submit-a-workflow-to-microsoft-genomics-using-a-sas-instead-of-a-storage-account-key"></a>Envoyer un workflow à Microsoft Genomics à l’aide d’un SAS plutôt que d’une clé de compte de stockage 

Cet article montre comment envoyer un workflow au service Microsoft Genomics à l’aide d’un fichier config.txt contenant des [signatures d’accès partagé (SAP)](../storage/common/storage-sas-overview.md) au lieu de clés de compte de stockage. Cette fonctionnalité peut être utile si avoir la clé de compte de stockage visible dans le fichier config.txt pose des problèmes de sécurité. 

Cette article suppose que vous avez déjà installé et exécuté le client `msgen`, et que vous savez comment utiliser Stockage Azure. Si vous avez correctement soumis un workflow à l’aide de l’exemple de données fourni, vous êtes prêt à appliquer la procédure décrite dans cet article. 

## <a name="what-is-a-sas"></a>Qu’est ce qu’une SAP ?
Une [signature d’accès partagé (SAP)](../storage/common/storage-sas-overview.md) fournit un accès délégué aux ressources de votre compte de stockage. Avec une signature d’accès partagé, vous pouvez accorder l’accès aux ressources dans votre compte de stockage sans partager les clés de votre compte. C’est tout l’intérêt d’utiliser des signatures d’accès partagé dans vos applications : une SAP est un moyen sécurisé de partager vos ressources de stockage sans compromettre vos clés de compte.

La SAP envoyée à Microsoft Genomics doit être une [SAP Service](/rest/api/storageservices/Constructing-a-Service-SAS) qui délègue l’accès uniquement au blob ou au conteneur où sont stockés les fichiers d’entrée et de sortie. 

L’URI d’un jeton de signature d’accès partagé (SAP) au niveau du service se compose de l’URI vers la ressource à laquelle la SAP va déléguer l’accès, suivi du jeton SAP. Le jeton SAP est la chaîne de requête qui inclut toutes les informations nécessaires pour authentifier la SAP, et qui précise également la ressource, les autorisations disponibles pour l’accès, l’intervalle de temps pendant lequel la signature est valide, l’adresse ou la plage d’adresses IP prise en charge à partir de laquelle les requêtes peuvent provenir, le protocole pris en charge permettant d’effectuer une requête, un identificateur de stratégie d’accès facultatif associé à la demande et la signature proprement dite. 

## <a name="sas-needed-for-submitting-a-workflow-to-the-microsoft-genomics-service"></a>SAP nécessaires pour l’envoi d’un flux de travail au service Microsoft Genomics
Deux ou plusieurs jetons SAP sont requis pour chaque flux de travail envoyé au service Microsoft Genomics, un pour chaque fichier d’entrée et un pour le conteneur de sortie.

Les SAP pour les fichiers d’entrée doivent avoir les propriétés suivantes :
 - Portée (compte, conteneur, blob) : blob
 - Expiration : 48 heures à partir de maintenant
 - Autorisations : lecture

Les SAP pour le conteneur de sortie doivent avoir les propriétés suivantes :
 - Portée (compte, conteneur, blob) : conteneur
 - Expiration : 48 heures à partir de maintenant
 - Autorisations : lecture, écriture, suppression


## <a name="create-a-sas-for-the-input-files-and-the-output-container"></a>Créer une SAP pour les fichiers d’entrée et le conteneur de sortie
Il existe deux façons de créer un jeton SAP, à l’aide de l’Explorateur Stockage Azure ou par programme.  Si vous écrivez du code, vous pouvez construire la SAP vous-même, ou utiliser le kit de développement logiciel (SDK) Stockage Azure dans le langage de votre choix.


### <a name="set-up-create-a-sas-using-azure-storage-explorer"></a>Configuration : Créer une SAP avec l’Explorateur Stockage Azure

L’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) est un outil pour gérer les ressources que vous avez stockées dans le stockage Azure.  Vous pouvez obtenir plus d’informations sur l’utilisation de l’Explorateur Stockage Azure [ici](../vs-azure-tools-storage-manage-with-storage-explorer.md).

Les SAP pour les fichiers d’entrée doivent être déterminées en fonction du fichier d’entrée particulier (blob). Pour créer un jeton SAP, suivez [ces instructions](../storage/blobs/storage-quickstart-blobs-storage-explorer.md). Une fois que vous avez créé la SAP, l’URL complète avec la chaîne de requête ainsi que la chaîne de requête elle-même sont fournies et peuvent être copiées à partir de l’écran.

 ![Explorateur Stockage, envoi d’une signature d’accès partagé à Genomics](./media/quickstart-input-sas/genomics-sas-storageexplorer.png "Explorateur Stockage, envoi d’une signature d’accès partagé à Genomics")


### <a name="set-up-create-a-sas-programmatically"></a>Configuration : créer une SAP par programmation

Pour créer une SAP à l’aide du kit de développement logiciel (SDK) Stockage Azure, consultez la documentation existante dans plusieurs langages, notamment [.NET](../storage/common/storage-sas-overview.md), [Python](../storage/blobs/storage-quickstart-blobs-python.md), and [Node.js](../storage/blobs/storage-quickstart-blobs-nodejs.md). 

Pour créer une SAP sans kit de développement logiciel (SDK), la chaîne de requête SAP peut être directement construite, y compris toutes les informations requises pour authentifier la SAP. Ces [instructions](/rest/api/storageservices/constructing-a-service-sas) décrivent en détail les composants de la chaîne de requête SAP et comment la générer. La signature SAP requise est créée en générant un HMAC à l’aide des informations d’authentification du blob/conteneur, comme décrit dans ces [instructions](/rest/api/storageservices/service-sas-examples).


## <a name="add-the-sas-to-the-configtxt-file"></a>Ajouter la SAP au fichier config.txt
Pour exécuter un flux de travail via le service Microsoft Genomics à l’aide d’une chaîne de requête SAP, modifiez le fichier config.txt pour supprimer les clés du fichier config.txt. Ensuite, ajoutez la chaîne de requête SAP (qui commence par un `?`) au nom du conteneur de sortie, comme indiqué. 

![Configuration d’une signature d’accès partagé pour Genomics](./media/quickstart-input-sas/genomics-sas-config.png "Configuration d’une signature d’accès partagé pour Genomics")

Utilisez le client Microsoft Genomics Python pour soumettre votre flux de travail avec la commande suivante, en ajoutant la chaîne de requête SAP correspondante à chacun des noms d’objet blob d’entrée :

```python
msgen submit -f [full path to your config file] -b1 [name of your first paired end read file, SAS query string appended] -b2 [name of your second paired end read file, SAS query string appended]
```

### <a name="if-adding-the-input-file-names-to-the-configtxt-file"></a>Si vous ajoutez les noms de fichiers d’entrée au fichier config.txt
Vous pouvez également ajouter directement les noms des fichiers de séquences appariées au fichier config.txt, en ajoutant les jetons de requête SAP comme indiqué :

![Configuration de noms d’objet blob avec signature d’accès partagé pour Genomics](./media/quickstart-input-sas/genomics-sas-config-blobnames.png "Configuration de noms d’objet blob avec signature d’accès partagé pour Genomics")

Utilisez le client Microsoft Genomics Python pour soumettre votre flux de travail à l’aide de la commande suivante, en omettant les commandes `-b1` et `-b2` :

```python
msgen submit -f [full path to your config file] 
```

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez utilisé des jetons SAP au lieu de clés de compte pour soumettre un flux de travail au service Microsoft Genomics via le client Python `msgen`. Pour plus d’informations sur la soumission du flux de travail et les autres commandes pouvant être utilisées avec le service Microsoft Genomics, consultez notre [FAQ](frequently-asked-questions-genomics.md).