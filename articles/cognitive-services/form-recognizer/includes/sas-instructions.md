---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 12/11/2020
ms.author: pafarley
ms.openlocfilehash: 3f4272f5445c8530e97726bd4fdca6bf662719c2
ms.sourcegitcommit: 02ed9acd4390b86c8432cad29075e2204f6b1bc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97807805"
---
Si vous souhaitez récupérer l’URL SAS pour vos données d’entraînement de modèle personnalisé, accédez à votre ressource de stockage dans le portail Azure, puis sélectionnez l’onglet **Explorateur Stockage**. Accédez à votre conteneur, cliquez avec le bouton droit, puis sélectionnez **Obtenir une signature d’accès partagé**. Il est important d’obtenir la signature d’accès partagé de votre conteneur, et non celle du compte de stockage. Assurez-vous que les autorisations de **Lecture** et **Écriture** sont cochées, puis cliquez sur **Créer**. Copiez ensuite la valeur de la section **URL** dans un emplacement temporaire. Il doit avoir le format : `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.