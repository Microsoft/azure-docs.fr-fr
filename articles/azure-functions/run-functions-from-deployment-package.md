---
title: Exécuter Azure Functions à partir d’un package
description: Exécutez vos fonctions avec le runtime Azure Functions, en montant un fichier de package de déploiement qui contient les fichiers projet de votre application de fonction.
ms.topic: conceptual
ms.date: 07/15/2019
ms.openlocfilehash: b2d90cf78263b30b4315199cf1c543186a435f17
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88639883"
---
# <a name="run-your-azure-functions-from-a-package-file"></a>Exécuter des fonctions Azure à partir d’un fichier de package

Dans Azure, vous pouvez exécuter vos fonctions directement dans un fichier de package de déploiement de votre application de fonction. L’autre option consiste à déployer vos fichiers dans le répertoire `d:\home\site\wwwroot` de votre application de fonction.

Cet article décrit les avantages que présente l’exécution de vos fonctions à partir d’un package. Il explique également comment activer cette fonctionnalité dans votre application de fonction.

## <a name="benefits-of-running-from-a-package-file"></a>Avantages de l’exécution à partir d’un fichier de package
  
L’exécution à partir d’un fichier de package présente plusieurs avantages :

+ Réduction des risques de verrouillage lors de la copie de fichiers
+ Déploiement possible sur une application de production (après redémarrage)
+ Connaissance des fichiers qui sont exécutés dans votre application
+ Amélioration des performances des [déploiements Azure Resource Manager](functions-infrastructure-as-code.md)
+ Peut réduire les temps de démarrage à froid, en particulier pour les fonctions JavaScript avec des arborescences de package npm de grande taille.

Pour plus d’informations, lisez [cette annonce](https://github.com/Azure/app-service-announcements/issues/84).

## <a name="enabling-functions-to-run-from-a-package"></a>Permettre aux fonctions de s’exécuter à partir d’un package

Pour permettre à votre application de fonction de s’exécuter à partir d’un package, il suffit d’ajouter un paramètre `WEBSITE_RUN_FROM_PACKAGE` à vos paramètres d’application de fonction. Le paramètre `WEBSITE_RUN_FROM_PACKAGE` doit avoir l’une des valeurs suivantes :

| Value  | Description  |
|---------|---------|
| **`1`**  | Recommandé pour les applications de fonction qui s’exécutent sur Windows. Lancez l’exécution à partir d’un fichier de package dans le dossier `d:\home\data\SitePackages` de votre application de fonction. Si vous ne [déployez pas le fichier avec la fonction de déploiement zip](#integration-with-zip-deployment), le dossier doit également contenir un fichier appelé `packagename.txt` pour que cette option puisse être utilisée. Ce fichier contient uniquement le nom du fichier de package du dossier, sans espace. |
|**`<URL>`**  | Emplacement d’un fichier de package que vous souhaitez exécuter. Lorsque vous utilisez le stockage Blob, vous devez utiliser un conteneur privé avec une [signature d’accès partagé (SAP)](../vs-azure-tools-storage-manage-with-storage-explorer.md#generate-a-sas-in-storage-explorer) pour permettre au runtime Functions d’accéder au package. Vous pouvez utiliser [l’Explorateur Stockage Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) pour charger des fichiers de package sur votre compte Stockage Blob. Lorsque vous spécifiez une URL, vous devez également [synchroniser les déclencheurs](functions-deployment-technologies.md#trigger-syncing) après avoir publié un package mis à jour. |

> [!CAUTION]
> Lorsque vous exécutez une application de fonction sur Windows, l’option d’URL externe génère les performances de démarrage à froid les moins élevées. Lorsque vous déployez votre application de fonction sur Windows, vous devez définir `WEBSITE_RUN_FROM_PACKAGE` sur `1` et effectuer la publication avec le déploiement zip.

L’exemple suivant montre une application de fonction configurée pour s’exécuter à partir d’un fichier zip hébergé dans le stockage Blob Azure :

![Paramètre d’application WEBSITE_RUN_FROM_ZIP](./media/run-functions-from-deployment-package/run-from-zip-app-setting-portal.png)

> [!NOTE]
> Actuellement, seuls les fichiers de package zip sont pris en charge.

## <a name="integration-with-zip-deployment"></a>Intégration au déploiement zip

[Déploiement zip][Zip deployment for Azure Functions] est une fonctionnalité Azure App Service qui vous permet de déployer votre projet d’application de fonction dans le répertoire `wwwroot`. Le projet est empaqueté dans un fichier de déploiement zip. Les mêmes API peuvent être utilisées pour déployer votre package dans le dossier `d:\home\data\SitePackages`. Quand le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` est défini sur `1`, les API de déploiement zip copient votre package dans le dossier `d:\home\data\SitePackages` au lieu d’extraire les fichiers dans `d:\home\site\wwwroot`. Le fichier `packagename.txt` est également créé. Après un redémarrage, le package est monté sur `wwwroot` en tant que système de fichiers en lecture seule. Pour plus d’informations sur le déploiement zip, consultez [Déploiement zip pour Azure Functions](deployment-zip-push.md).

> [!NOTE]
> Lorsqu’un déploiement se produit, un redémarrage de l’application de fonction est déclenché. Avant un redémarrage, toutes les exécutions de fonction existantes sont autorisées à se terminer ou à expirer. Pour plus d’informations, consultez [Comportements de déploiement](functions-deployment-technologies.md#deployment-behaviors).

## <a name="adding-the-website_run_from_package-setting"></a>Ajout du paramètre WEBSITE_RUN_FROM_PACKAGE

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]


## <a name="troubleshooting"></a>Dépannage

- L’option d’exécution à partir d’un package rend `wwwroot` accessible en lecture seule. Donc, vous recevez une erreur lors de l’écriture de fichiers dans ce répertoire.
- Les formats TAR et GZIP ne sont pas pris en charge.
- Cette fonctionnalité ne tient pas compte du cache local.
- Pour améliorer les performances de démarrage à froid, utilisez l’option zip locale (`WEBSITE_RUN_FROM_PACKAGE`= 1).
- L’option d’exécution à partir d’un package n’est pas compatible avec l’option de personnalisation du déploiement (`SCM_DO_BUILD_DURING_DEPLOYMENT=true`), l’étape de génération est donc ignorée pendant le déploiement.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déploiement continu pour Azure Functions](functions-continuous-deployment.md)

[Zip deployment for Azure Functions]: deployment-zip-push.md
