---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 10/16/2019
ms.author: mimart
ms.openlocfilehash: 3ebe1ec4c0292a530e5ef2c754e9b002e931300e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994306"
---
#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/) 

1. Sélectionnez **Inscriptions d’applications**.
1. Sélectionnez l’application *webapi1* pour ouvrir sa page **Vue d’ensemble**.
1. Sous **Gérer**, sélectionnez **Exposer une API**.
1. A côté d’**URI d’ID d’application**, sélectionnez le lien **Définir**.
1. Remplacez la valeur par défaut (un GUID) par `api`, puis sélectionnez **Enregistrer**. L’URI complet est affiché et doit être au format `https://your-tenant-name.onmicrosoft.com/api`. Lorsque votre application web demande un jeton d’accès pour l’API, elle doit ajouter cet URI en tant que préfixe de chaque étendue que vous définissez pour l’API.
1. Sous **Étendues définies par cette API**, sélectionnez **Ajouter une étendue**.
1. Entrez les valeurs suivantes pour créer une étendue qui définit l’accès en lecture à l’API, puis sélectionnez **Ajouter une étendue** :
    1. **Nom de l’étendue** : `demo.read`
    1. **Nom d’affichage du consentement administrateur** : `Read access to demo API`
    1. **Description du consentement de l’administrateur** : `Allows read access to the demo API`
1. Sélectionnez **Ajouter une étendue**, entrez les valeurs suivantes pour ajouter une étendue qui définit l’accès en écriture à l’API, puis sélectionnez **Ajouter une étendue** :
    1. **Nom de l’étendue** : `demo.write`
    1. **Nom d’affichage du consentement administrateur** : `Write access to demo API`
    1. **Description du consentement de l’administrateur** : `Allows write access to the demo API`

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Sélectionnez **Applications (héritées)** .
1. Sélectionnez l’application *webapi1* pour ouvrir sa page **Propriétés**.
1. Sélectionnez **Étendues publiées**. Les étendues publiées peuvent être utilisées pour accorder à une application cliente certaines autorisations d’accès à l’API web.
1. Pour **ÉTENDUE**, entrez `demo.read` puis, pour **DESCRIPTION**, entrez `Read access to the web API`.
1. Pour **ÉTENDUE**, entrez `demo.write` puis, pour **DESCRIPTION**, entrez `Write access to the web API`.
1. Sélectionnez **Enregistrer**.