---
title: 'Démarrage rapide : Nouvelle attribution de stratégie avec JavaScript'
description: Dans ce guide de démarrage rapide, vous utilisez JavaScript pour créer une attribution Azure Policy afin d’identifier des ressources non conformes.
ms.date: 09/24/2020
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 7548053e4bd5be214bf7de3eef3dc4c6c95442d4
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91347973"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-javascript"></a>Démarrage rapide : Créer une attribution de stratégie pour identifier les ressources non conformes à l’aide de JavaScript

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources. Dans ce démarrage rapide, vous créerez une affectation de stratégie pour identifier les machines virtuelles qui n’utilisent pas de disques managés. Lorsque vous aurez terminé, vous identifierez les machines virtuelles qui ne sont _pas conformes_.

La bibliothèque JavaScript permet de gérer les ressources Azure à partir de la ligne de commande ou à l’aide de scripts. Ce guide explique comment utiliser une bibliothèque JavaScript pour créer une attribution de stratégie.

## <a name="prerequisites"></a>Prérequis

- **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- **Node.JS** : La version 12 ou supérieure de [Node.js](https://nodejs.org/) est nécessaire.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-libraries"></a>Ajouter les bibliothèques Azure Policy

Pour permettre à JavaScript de fonctionner avec Azure Policy, les bibliothèques doivent être ajoutées. Ces bibliothèques fonctionnent partout où vous pouvez utiliser JavaScript, y compris [bash sur Windows 10](/windows/wsl/install-win10).

1. Configurez un nouveau projet Node.js en exécutant la commande suivante.

   ```bash
   npm init -y
   ```

1. Ajoutez une référence à la bibliothèque yargs.

   ```bash
   npm install yargs
   ```

1. Ajoutez une référence aux bibliothèques Azure Policy.

   ```bash
   # arm-policy is for working with Azure Policy objects such as definitions and assignments
   npm install @azure/arm-policy

   # arm-policyinsights is for working with Azure Policy compliance data such as events and states
   npm install @azure/arm-policyinsights
   ```

1. Ajoutez une référence à la bibliothèque d’authentification Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Vérifiez dans _package.json_ que la version de `@azure/arm-policy` est **3.1.0** ou une version ultérieure, que la version de `@azure/arm-policyinsights` est **3.2.0** ou une version ultérieure, et que la version de `@azure/ms-rest-nodeauth` est **3.0.5** ou une version ultérieure.

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

Dans ce guide de démarrage rapide, vous allez créer une attribution de stratégie et attribuer la définition **Auditer les machines virtuelles qui n'utilisent aucun disque managé** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Cette stratégie identifie les ressources qui ne sont pas conformes aux conditions définies dans sa définition.

1. Créez un fichier nommé _policyAssignment.js_ et entrez le code suivant.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyObjects = require("@azure/arm-policy");
   
   if (argv.subID && argv.name && argv.displayName && argv.policyDefID && argv.scope && argv.description) {
   
       const createAssignment = async () => {
           const credentials = await authenticator.interactiveLogin();
           const client = new policyObjects.PolicyClient(credentials, argv.subID);
           const assignments = new policyObjects.PolicyAssignments(client);
   
           const result = await assignments.create(
               argv.scope,
               argv.name,
               {
                   displayName: argv.displayName,
                   policyDefinitionId: argv.policyDefID,
                   description: argv.description
               }
           );
           console.log(result);
       };
   
       createAssignment();
   }
   ```

1. Entrez la commande suivante dans le terminal :

   ```bash
   node policyAssignment.js `
      --subID "{subscriptionId}" `
      --name "audit-vm-manageddisks" `
      --displayName "Audit VMs without managed disks Assignment" `
      --policyDefID "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      --description "Shows all virtual machines not using managed disks" `
      --scope "{scope}"
   ```

Les commandes précédentes utilisent les informations suivantes :

- **subID** – ID d’abonnement du contexte d’authentification. Veillez à remplacer `{subscriptionId}` par votre abonnement.
- **name** – Nom unique de l’objet d’attribution de stratégie. L’exemple ci-dessus utilise _audit-vm-manageddisks_.
- **displayName** – Nom complet pour l’attribution de stratégie. Dans ce cas, nous allons utiliser l’affectation _Audit VMs without managed disks_ (Auditer les machines virtuelles sans disques managés).
- **policyDefID** – Chemin de la définition de stratégie, sur lequel vous vous basez pour créer l’attribution. Dans ce cas, il s’agit de l’ID de la définition de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_.
- **description** : explication plus détaillée de ce que fait la stratégie ou de la raison pour laquelle elle est attribuée à cette étendue.
- **scope** : une étendue détermine les ressources ou le groupe de ressources sur lesquels l'attribution de stratégie est appliquée. Cette étendue peut aller d'un groupe d'administration à une ressource individuelle. Veillez à remplacer `{scope}` par l'un des modèles suivants :
  - Groupe d'administration : `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Abonnement : `/subscriptions/{subscriptionId}`
  - Groupe de ressources : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Ressource : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

Vous êtes maintenant prêt à identifier les ressources non conformes pour comprendre l’état de conformité de votre environnement.

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Maintenant que votre attribution de stratégie est créée, vous pouvez identifier les ressources qui ne sont pas conformes.

1. Créez un fichier nommé _policyState.js_ et entrez le code suivant.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyInsights = require("@azure/arm-policyinsights");
   
   if (argv.subID && argv.name) {
   
       const getStates = async () => {
   
           const credentials = await authenticator.interactiveLogin();
           const client = new policyInsights.PolicyInsightsClient(credentials);
           const policyStates = new policyInsights.PolicyStates(client);
           const result = await policyStates.listQueryResultsForSubscription(
               "latest",
               argv.subID,
               {
                   queryOptions: {
                       filter: "IsCompliant eq false and PolicyAssignmentId eq '" + argv.name + "'",
                       apply: "groupby((ResourceId))"
                   }
               }
           );
           console.log(result);
       };
   
       getStates();
   }
   ```

1. Entrez la commande suivante dans le terminal :

   ```bash
   node policyState.js --subID "{subscriptionId}" --name "audit-vm-manageddisks"
   ```

Remplacez `{subscriptionId}` par l’abonnement dont vous souhaitez afficher les résultats de conformité pour l’attribution de stratégie nommée « audit-vm-manageddisks » que nous avons créée dans les étapes précédentes. Pour obtenir la liste des autres étendues et méthodes permettant de synthétiser les données, consultez les méthodes [PolicyStates*](/javascript/api/@azure/arm-policyinsights/).

Vos résultats doivent ressembler à l’exemple suivant :

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Les résultats correspondent à ce que vous voyez sous l’onglet **Conformité des ressources** d’une attribution de stratégie dans la vue du portail Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

- Supprimez l’attribution de stratégie _Audit VMs without managed disks Assignment_ (Auditer les machines virtuelles sans attribution de disques managés) via le portail. La définition de stratégie étant intégrée, il n’y a aucune définition à supprimer.

- Si vous souhaitez supprimer les bibliothèques installées de votre application, exécutez la commande suivante.

  ```bash
  npm uninstall @azure/arm-policy @azure/arm-policyinsights @azure/ms-rest-nodeauth yargs
  ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie pour identifier les ressources non conformes de votre environnement Azure.

Pour en savoir plus sur l'attribution de définitions de stratégie visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)