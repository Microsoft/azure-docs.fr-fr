---
title: Exporter la configuration de provisionnement et restaurer la dernière bonne configuration connue pour la reprise d’activité
description: Découvrez comment exporter votre configuration d’approvisionnement et restaurer un état correct connu pour la récupération d’urgence.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 03/19/2020
ms.author: kenwith
ms.openlocfilehash: e34656d6ce515cabe955c101f7b52ac0f2ade8db
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994806"
---
# <a name="how-to-export-provisioning-configuration-and-roll-back-to-a-known-good-state"></a>Procédure : Exporter la configuration de provisionnement et restaurer la dernière bonne configuration connue

Dans cet article, vous allez apprendre à :

- Exporter et importer votre configuration d’approvisionnement à partir du portail Azure
- Exporter et importer votre configuration d’approvisionnement à l’aide de l’API Microsoft Graph

## <a name="export-and-import-your-provisioning-configuration-from-the-azure-portal"></a>Exporter et importer votre configuration d’approvisionnement à partir du portail Azure

### <a name="export-your-provisioning-configuration"></a>Exporter votre configuration de provisionnement

Pour exporter vos données de configuration :

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com/), sélectionnez **Azure Active Directory**.
1. Dans le volet **Azure Active Directory**, sélectionnez **Applications d’entreprise** et choisissez votre application.
1. Dans le volet de navigation de gauche, sélectionnez **Approvisionnement**. À partir de la page de configuration de l’approvisionnement, cliquez sur **mappages d’attributs**, puis sur **afficher les options avancées**, et enfin sur **passer en revue votre de schéma**. Vous accédez alors à l’éditeur de schéma.
1. Cliquez sur Télécharger dans la barre de commandes en haut de la page pour télécharger votre schéma.

### <a name="disaster-recovery---roll-back-to-a-known-good-state"></a>Récupération d’urgence : restaurer un état correct connu

L’exportation et l’enregistrement de votre configuration vous permettent de restaurer une version précédente de votre configuration. Nous vous recommandons d’exporter votre configuration d’approvisionnement et de l’enregistrer pour une utilisation ultérieure chaque fois que vous apportez une modification à vos mappages d’attributs ou filtres d’étendue. Il vous suffit d’ouvrir le fichier JSON que vous avez téléchargé lors des étapes précédentes, de copier tout le contenu du fichier JSON, de remplacer tout le contenu de la charge utile JSON dans l’éditeur de schéma, puis d’enregistrer. Si un cycle d’approvisionnement est actif, il se terminera et le cycle suivant utilisera le schéma mis à jour. Le cycle suivant sera également un cycle initial, qui réévaluera tous les utilisateurs et groupes en fonction de la nouvelle configuration. Tenez compte de ce qui suit lors de la restauration d’une configuration précédente :

- Les utilisateurs sont réévalués pour déterminer s’ils doivent être dans l’étendue. Si les filtres d’étendue ont changé et qu’un utilisateur n’est plus dans la portée. il est alors désactivé. Même si ce comportement est souhaité dans la plupart des cas, il est possible que vous souhaitiez l’empêcher cela et utiliser la fonctionnalité [ignorer les suppressions d’étendue](./skip-out-of-scope-deletions.md). 
- La modification de votre configuration d’approvisionnement redémarre le service et déclenche un [cycle initial](./how-provisioning-works.md#provisioning-cycles-initial-and-incremental).

## <a name="export-and-import-your-provisioning-configuration-by-using-the-microsoft-graph-api"></a>Exporter et importer votre configuration d’approvisionnement à l’aide de l’API Microsoft Graph

Vous pouvez utiliser l’API Microsoft Graph et Microsoft Graph Explorer pour exporter vos mappages d’attributs et votre schéma Attribution d’utilisateurs dans un fichier JSON et l’importer dans Azure AD. Vous pouvez aussi utiliser les étapes capturées ici pour créer une sauvegarde de votre configuration de provisionnement.

### <a name="step-1-retrieve-your-provisioning-app-service-principal-id-object-id"></a>Étape 1 : Récupérer l’ID de principal du service de l’application de provisionnement (ID d’objet)

1. Lancez le [portail Azure](https://portal.azure.com) et accédez à la section Propriétés de votre application de provisionnement. Par exemple, si vous souhaitez exporter votre mappage d’*application d’approvisionnement Workday vers l’utilisateur AD*, accédez à la section Propriétés de cette application.
1. Dans la section Propriétés de votre application d'approvisionnement, copiez la valeur GUID associée au champ *ID de l'objet*. Cette valeur, également appelée **ServicePrincipalId** de votre application, sera utilisée dans les opérations de Microsoft Graph Explorer.

   ![ID du principal de service de l'application Workday](./media/export-import-provisioning-configuration/wd_export_01.png)

### <a name="step-2-sign-into-microsoft-graph-explorer"></a>Étape 2 : Se connecter à l'Afficheur Microsoft Graph

1. Lancez l'[Afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
1. Cliquez sur le bouton « Se connecter avec Microsoft » et connectez-vous à l'aide des informations d'identification d'administrateur de l'application ou d'administrateur global d'Azure AD.

    ![Connexion Microsoft Graph](./media/export-import-provisioning-configuration/wd_export_02.png)

1. Une fois connecté, les détails du compte d'utilisateur apparaissent dans le volet de gauche.

### <a name="step-3-retrieve-the-provisioning-job-id-of-the-provisioning-app"></a>Étape 3 : Récupérer l’ID de travail de provisionnement de l’application de provisionnement

Dans l'Afficheur Microsoft Graph, exécutez la requête GET suivante en remplaçant [servicePrincipalId] par la valeur **ServicePrincipalId** extraite à l'[étape 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs
```

Vous obtiendrez une réponse semblable à l'exemple ci-dessous. Copier l'attribut « id » présent dans la réponse. Il s'agit de la valeur **ProvisioningJobId** qui sera utilisée pour extraire les métadonnées du schéma sous-jacent.

   [![ID du travail d’approvisionnement](./media/export-import-provisioning-configuration/wd_export_03.png)](./media/export-import-provisioning-configuration/wd_export_03.png#lightbox)

### <a name="step-4-download-the-provisioning-schema"></a>Étape 4 : Télécharger le schéma d'approvisionnement

Dans l'Afficheur Microsoft Graph, exécutez la requête GET suivante, en remplaçant [servicePrincipalId] et [ProvisioningJobId] par les valeurs ServicePrincipalId et ProvisioningJobId extraites lors des étapes précédentes.

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Copiez l'objet JSON à partir de la réponse et enregistrez-le dans un fichier pour créer une sauvegarde du schéma.

### <a name="step-5-import-the-provisioning-schema"></a>Étape 5 : Importer le schéma d'approvisionnement

> [!CAUTION]
> Ne suivez cette étape que si vous devez modifier le schéma pour une configuration non modifiable à l'aide du portail Azure ou si vous devez restaurer la configuration à partir d'un fichier précédemment sauvegardé et contenant un schéma valide et fonctionnel.

Dans l'Afficheur Microsoft Graph, configurez la requête PUT suivante en remplaçant [servicePrincipalId] et [ProvisioningJobId] par les valeurs ServicePrincipalId et ProvisioningJobId extraites lors des étapes précédentes.

```http
    PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Dans l'onglet « Corps de la demande », copiez le contenu du fichier de schéma JSON.

   [![Corps de la demande](./media/export-import-provisioning-configuration/wd_export_04.png)](./media/export-import-provisioning-configuration/wd_export_04.png#lightbox)

Sous l'onglet « En-têtes des demandes », ajoutez l'attribut d'en-tête Content-Type avec la valeur « application/json ».

   [![En-têtes des demandes](./media/export-import-provisioning-configuration/wd_export_05.png)](./media/export-import-provisioning-configuration/wd_export_05.png#lightbox)

Sélectionnez **Exécuter la requête** pour importer le nouveau schéma.