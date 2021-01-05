---
title: Ignorer la suppression des utilisateurs qui sortent de l’étendue
description: Découvrez comment remplacer le comportement par défaut de déprovisionnement des utilisateurs hors étendue.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 12/10/2019
ms.author: kenwith
ms.reviewer: celested
ms.openlocfilehash: f459a804b4c375eea17cbc22ded2f41f808c1b82
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995367"
---
# <a name="skip-deletion-of-user-accounts-that-go-out-of-scope"></a>Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue

Par défaut, le moteur de provisionnement Azure AD supprime de manière réversible ou désactive les utilisateurs qui sortent de l’étendue. Cependant, dans certains scénarios, comme le provisionnement entrant d’utilisateurs de Workday vers AD, ce comportement par défaut peut ne pas convenir et vous pouvez souhaiter le remplacer.  

Cet article explique comment utiliser l’API Microsoft Graph et son Afficheur pour définir l’indicateur ***SkipOutOfScopeDeletions** _ qui contrôle le traitement des comptes qui sortent de l’étendue. _ Si ***SkipOutOfScopeDeletions** _ est défini sur 0 (false), les comptes en dehors de l’étendue seront désactivés dans la cible.
_ Si ***SkipOutOfScopeDeletions** _ est défini sur 1 (true), les comptes en dehors de l’étendue ne seront pas désactivés dans la cible. Cet indicateur est défini au niveau de l’_application d’approvisionnement* et peut être configuré à l’aide de l’API Graph. 

Du fait que cette configuration est largement utilisée avec l’application de *provisionnement d’utilisateurs de Workday vers Active Directory*, les étapes suivantes incluent des captures d’écran de l’application Workday. Cependant, la configuration peut aussi être utilisée avec *toutes les autres applications*, comme ServiceNow, Salesforce et Dropbox.

## <a name="step-1-retrieve-your-provisioning-app-service-principal-id-object-id"></a>Étape 1 : Récupérer l’ID de principal du service de l’application de provisionnement (ID d’objet)

1. Lancez le [portail Azure](https://portal.azure.com) et accédez à la section Propriétés de votre application de provisionnement. Par exemple, si vous souhaitez exporter votre mappage d’*application de provisionnement de Workday vers AD*, accédez à la section Propriétés de cette application. 
1. Dans la section Propriétés de votre application d'approvisionnement, copiez la valeur GUID associée au champ *ID de l'objet*. Cette valeur, également appelée **ServicePrincipalId** de votre application, sera utilisée dans les opérations de l'Afficheur Graph.

   ![ID du principal de service de l'application Workday](./media/skip-out-of-scope-deletions/wd_export_01.png)

## <a name="step-2-sign-into-microsoft-graph-explorer"></a>Étape 2 : Se connecter à l'Afficheur Microsoft Graph

1. Lancez l'[Afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
1. Cliquez sur le bouton « Se connecter avec Microsoft » et connectez-vous à l'aide des informations d'identification d'administrateur de l'application ou d'administrateur global d'Azure AD.

    ![Connexion à Graph](./media/skip-out-of-scope-deletions/wd_export_02.png)

1. Une fois connecté, les détails du compte d'utilisateur apparaissent dans le volet de gauche.

## <a name="step-3-get-existing-app-credentials-and-connectivity-details"></a>Étape 3 : Obtenir les informations d’identification et les détails de connectivité de l’application existante

Dans l'Afficheur Microsoft Graph, exécutez la requête GET suivante en remplaçant [servicePrincipalId] par la valeur **ServicePrincipalId** extraite à l'[étape 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/secrets
```

   ![Requête de tâche GET](./media/skip-out-of-scope-deletions/skip-03.png)

Copiez la réponse dans un fichier texte. Elle ressemble au texte JSON présenté ci-dessous, à la différence près que les valeurs surlignées en jaune sont remplacées par celles propres à votre déploiement. Ajoutez les lignes surlignées en vert situées à la fin et mettez à jour le mot de passe de connexion Workday surligné en bleu. 

   ![Réponse de tâche GET](./media/skip-out-of-scope-deletions/skip-04.png)

Voici le bloc JSON à ajouter au mappage. 

```json
        {
            "key": "SkipOutOfScopeDeletions",
            "value": "True"
        }
```

## <a name="step-4-update-the-secrets-endpoint-with-the-skipoutofscopedeletions-flag"></a>Étape 4 : Mettre à jour le point de terminaison secrets avec l’indicateur SkipOutOfScopeDeletions

Dans l’Afficheur Graph, exécutez la commande ci-dessous pour mettre à jour le point de terminaison secrets avec l’indicateur **_SkipOutOfScopeDeletions_* _. 

Dans l’URL ci-dessous, remplacez [servicePrincipalId] par la valeur _ *ServicePrincipalId** extraite à l’[étape 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id). 

```http
   PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/secrets
```
Copiez le texte mis à jour de l’étape 3 dans « Corps de la demande » et définissez l’en-tête « Content-Type » sur « application/json » dans « En-têtes de demande ». 

   ![Demande PUT](./media/skip-out-of-scope-deletions/skip-05.png)

Cliquer sur « Exécuter la requête ». 

Vous devez obtenir la sortie « Réussite – Code d’État 204 ». 

   ![Réponse PUT](./media/skip-out-of-scope-deletions/skip-06.png)

## <a name="step-5-verify-that-out-of-scope-users-dont-get-disabled"></a>Étape 5 : Vérifier que les utilisateurs hors étendue ne sont pas désactivés

Vous pouvez vérifier que cet indicateur produit un comportement attendu en mettant à jour vos règles d’étendue pour ignorer un utilisateur spécifique. Dans l’exemple ci-dessous, nous excluons l’employé dont l’ID est 21173 (qui se trouvait auparavant dans l’étendue) en ajoutant une nouvelle règle d’étendue : 

   ![Capture d’écran montrant la section « Ajouter un filtre d’étendue » avec un exemple d’utilisateur mis en évidence.](./media/skip-out-of-scope-deletions/skip-07.png)

Dans le prochain cycle de provisionnement, le service de provisionnement d’Azure AD déterminera que l’utilisateur 21173 est sorti de l’étendue si la propriété SkipOutOfScopeDeletions est activée, et la règle de synchronisation pour cet utilisateur affichera un message comme indiqué ci-dessous : 

   ![Exemple d’étendue](./media/skip-out-of-scope-deletions/skip-08.png)


