---
title: Fichier Include
description: Fichier Include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/04/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 4604616cd4f2d6c75c272586df1331fc405061cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70737489"
---
## <a name="error-conditionheadersnotsupported-from-a-web-application-using-azure-files-from-browser"></a>Erreur ConditionHeadersNotSupported sur une application web utilisant Azure Files à partir du navigateur

L’erreur ConditionHeadersNotSupported se produit quand vous accédez à du contenu hébergé dans Azure Files à partir d’une application utilisant des en-têtes conditionnels (par exemple, un navigateur web) et que l’accès échoue. Le message d’erreur indique que les en-têtes conditionnels ne sont pas pris en charge.

![Erreur relative aux en-têtes conditionnels avec Azure Files](media/storage-files-condition-headers/conditionalerror.png)

### <a name="cause"></a>Cause :

Les en-têtes conditionnels ne sont pas encore pris en charge. Les applications qui utilisent ces en-têtes doivent demander le fichier complet pour chaque accès au fichier.

### <a name="workaround"></a>Solution de contournement

Quand un nouveau fichier est chargé, la propriété cache-control est définie sur « no-cache » par défaut. Pour forcer l’application à demander le fichier à chaque fois, la valeur « no-cache » de la propriété cache-control du fichier doit être remplacée par « no-cache, no-store, must-revalidate ». Vous pouvez le faire en utilisant [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/).

![Modification du cache de contenu de l’Explorateur Stockage pour les en-têtes conditionnels Azure Files](media/storage-files-condition-headers/storage-explorer-cache.png)