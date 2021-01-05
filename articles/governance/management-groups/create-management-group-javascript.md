---
title: 'Démarrage rapide : Créer un groupe d’administration avec JavaScript'
description: Dans ce démarrage rapide, vous allez utiliser JavaScript pour créer un groupe d’administration afin de hiérarchiser vos ressources.
ms.date: 11/18/2020
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: c1b800f150cebacf8d17785feb1bded5189f1574
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886598"
---
# <a name="quickstart-create-a-management-group-with-javascript"></a>Démarrage rapide : Créer un groupe d’administration avec JavaScript

Les groupes d’administration sont des conteneurs qui vous aident à gérer l’accès, la stratégie et la conformité dans plusieurs abonnements. Créez ces conteneurs pour construire une hiérarchie efficace utilisable avec [Azure Policy](../policy/overview.md) et les [contrôles d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md). Pour plus d’informations sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](overview.md).

La création du premier groupe d’administration dans l’annuaire peut nécessiter jusqu’à 15 minutes. En effet, des processus s’exécutent la première fois pour configurer le service des groupes d’administration dans Azure pour votre annuaire. Vous recevez une notification quand le processus est terminé. Pour plus d’informations, consultez la [configuration initiale des groupes d’administration](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- Avant de commencer, assurez-vous que la dernière version (12) de [Node.js](https://nodejs.org/) est installée.

- Tout utilisateur Azure AD dans le locataire peut créer un groupe d’administration sans l’autorisation d’écriture du groupe d’administration attribuée à cet utilisateur si la [protection de la hiérarchie](./how-to/protect-resource-hierarchy.md#setting---require-authorization) n’est pas activée. Ce nouveau groupe d’administration devient alors un enfant du groupe d’administration racine ou le [groupe d’administration par défaut](./how-to/protect-resource-hierarchy.md#setting---default-management-group), et le rôle « Propriétaire » est attribué au créateur. Le service de groupe d’administration offre cette possibilité afin que les attributions de rôles ne soient pas nécessaires au niveau de la racine. Aucun utilisateur n’a accès au groupe d’administration racine lors de sa création. Pour éviter l’obstacle que constitue la recherche des administrateurs généraux Azure AD pour commencer à utiliser des groupes d’administration, nous autorisons la création des groupes d’administration initiaux au niveau racine.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Configuration des applications

Pour permettre à JavaScript d’interroger Azure Resource Graph, l’environnement doit être configuré. Cette configuration fonctionne partout où vous pouvez utiliser JavaScript, y compris [bash sur Windows 10](/windows/wsl/install-win10).

1. Configurez un nouveau projet Node.js en exécutant la commande suivante.

   ```bash
   npm init -y
   ```

1. Ajoutez une référence au module yargs.

   ```bash
   npm install yargs
   ```

1. Ajoutez une référence au module Azure Resource Graph.

   ```bash
   npm install @azure/arm-managementgroups
   ```

1. Ajoutez une référence à la bibliothèque d’authentification Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Vérifiez dans _package.json_ que la version de `@azure/arm-managementgroups` est **1.1.0** ou une version ultérieure, et que la version de `@azure/ms-rest-nodeauth` est **3.0.5** ou une version ultérieure.

## <a name="create-the-management-group"></a>Créer le groupe d’administration

1. Créez un fichier nommé _index.js_ et entrez le code suivant.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const managementGroups = require("@azure/arm-managementgroups");

   if (argv.groupID && argv.displayName) {
       const createMG = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new managementGroups.ManagementGroupsAPI(credentials);
          const result = await client.managementGroups.createOrUpdate(
             groupId: argv.groupID,
             {
                 displayName: argv.displayName
             }
          );
          console.log(result);
       };

       createMG();
   }
   ```

1. Entrez la commande suivante dans le terminal :

   ```bash
   node index.js --groupID "<NEW_MG_GROUP_ID>" --displayName "<NEW_MG_FRIENDLY_NAME>"
   ```

   Veillez à remplacer l’espace réservé `<>` de chaque jeton respectivement par votre _ID de groupe d’administration_ et votre _nom convivial de groupe d’administration_.

   Quand le script tente de s’authentifier, un message similaire au message suivant s’affiche dans le terminal :

   > Pour vous connecter, utilisez un navigateur web afin d’ouvrir la page https://microsoft.com/devicelogin et entrez le code FGB56WJUGK pour vous authentifier.

   Une fois que vous vous authentifiez dans le navigateur, le script continue à s’exécuter.

Le résultat de la création du groupe d’administration s’affiche sur la console.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer les bibliothèques installées de votre application, exécutez la commande suivante.

```bash
npm uninstall @azure/arm-managementgroups @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un groupe d’administration pour organiser votre hiérarchie de ressources. Le groupe d’administration peut contenir des abonnements ou d’autres groupes d’administration.

Pour en savoir plus sur les groupes d’administration et sur la gestion de votre hiérarchie de ressources, consultez :

> [!div class="nextstepaction"]
> [Gérer vos ressources avec des groupes d’administration](./manage.md)