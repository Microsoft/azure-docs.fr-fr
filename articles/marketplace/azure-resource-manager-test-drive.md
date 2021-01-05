---
title: Types de versions d’évaluation, Place de marché commerciale Microsoft
description: Types de versions d’évaluation sur la Place de marché commerciale
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 06/19/2020
ms.author: trkeya
author: trkeya
ms.openlocfilehash: fd3fa04d2d7e868476838788dd9cf0e27c07aeca
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461757"
---
# <a name="azure-resource-manager-test-drive"></a>Version d’évaluation Azure Resource Manager

Utilisez ce type si vous disposez d’une offre sur la Place de marché Azure ou AppSource, mais que vous souhaitez créer une version d’évaluation uniquement avec des ressources Azure. Un modèle ARM (Azure Resource Manager) est un conteneur codé de ressources Azure que vous concevez pour représenter votre solution de la meilleure façon. La version d’évaluation prend le modèle ARM fourni et déploie toutes les ressources nécessaires sur un groupe de ressources. Il s’agit de la seule option de version d’évaluation pour les machines virtuelles ou Azure App.

Si vous ne savez pas très bien ce qu’est un modèle ARM, lisez [Qu’est-ce qu’Azure Resource Manager ?](../azure-resource-manager/management/overview.md) et [Comprendre la structure et la syntaxe des modèles ARM](../azure-resource-manager/templates/template-syntax.md) pour mieux comprendre comment créer et tester vos propres modèles.

Pour plus d’informations sur ce qu’est une version d’évaluation **hébergée** ou d’**application logique**, consultez [Qu’est-ce qu’une version d’évaluation ?](what-is-test-drive.md)

## <a name="technical-configuration"></a>Configuration technique

Un modèle de déploiement contient toutes les ressources Azure constituant votre solution. Les produits adaptés à ce scénario utilisent uniquement des ressources Azure. Définissez les propriétés suivantes dans l’Espace partenaires :

- **Régions** (obligatoire) : à l’heure actuelle, vous pouvez mettre votre version d’évaluation à disposition dans 26 régions Azure. En règle générale, vous devez rendre votre version d’évaluation disponible dans les régions où vous prévoyez le plus grand nombre de clients, de sorte qu’ils puissent sélectionner la région la plus proche pour des performances optimales. Vous devez vous assurer que votre abonnement a l’autorisation de déployer toutes les ressources nécessaires dans chacune des régions sélectionnées.

- **Instances** : sélectionnez le type (chaud ou froid) et le nombre d’instances disponibles, qui seront multipliés par le nombre de régions où votre offre sera disponible.

  - **Chaud** : ce type d’instance est déployé et en attente de l’accès par région sélectionnée. Les clients peuvent accéder instantanément aux instances *à chaud* d’une version d’évaluation au lieu d’attendre un déploiement. L’inconvénient est que ces instances sont toujours en cours d’exécution sur votre abonnement Azure, entraînant des coûts de fonctionnement plus importants. Il est vivement recommandé d’avoir au moins une instance *à chaud*, étant donné que la plupart de vos clients ne souhaitent pas attendre la fin des déploiements complets, et il y a donc une chute dans l’utilisation du client si aucune instance *à chaud* n’est disponible.

  - **Froid** : ce type d’instance représente le nombre total d’instances pouvant éventuellement être déployées par région. Les instances à froid nécessitent le modèle Resource Manager entier de la version d’évaluation pour être déployées lorsqu’un client demande la version d’évaluation, les instances *à froid* sont donc plus lentes à charger que les instances *à chaud*. L’inconvénient est que vous avez uniquement à payer pour la durée de la version d’évaluation, elle n’est *pas* toujours en cours d’exécution sur votre abonnement Azure comme c’est le cas avec une instance *à chaud*.

- **Version d’évaluation du modèle Azure Resource Manager** – Chargez le fichier .zip contenant votre modèle Azure Resource Manager. Apprenez-en plus sur la création d’un modèle Azure Resource Manager dans l’article de démarrage rapide [Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

    > [!note]
    > Pour une publication réussie, il est important de valider la mise en forme du modèle ARM. Pour ce faire, il existe deux méthodes : (1) en utilisant un [outil API en ligne](/rest/api/resources/deployments/validate) ou (2) avec un [déploiement de test](../azure-resource-manager/templates/deploy-portal.md).

- **Durée de la version d’évaluation** (obligatoire) – Entrez le nombre d’heures durant lesquelles la version d’évaluation restera active. La version d’évaluation se termine automatiquement à la fin de cette période. Utilisez uniquement des nombres entiers (par exemple, « 2 » heures est valide, « 1,5 » ne l’est pas).

## <a name="write-the-test-drive-template"></a>Écrire le modèle de version d’évaluation

Une fois que vous avez conçu le package de ressources souhaité, écrivez et générez le modèle ARM de version d’évaluation. La version d’évaluation exécutant des déploiements dans un mode entièrement automatisé, les modèles de version d’évaluation ont certaines restrictions :

### <a name="parameters"></a>Paramètres

La plupart des modèles ont un ensemble de paramètres qui définissent les noms de ressources, les tailles de ressources (telles que les types de comptes de stockage ou les tailles de machine virtuelle), les noms d’utilisateur et les mots de passe, les noms DNS et ainsi de suite. Lorsque vous déployez des solutions à l’aide du portail Azure, vous pouvez manuellement remplir tous ces paramètres, choisir des noms DNS disponibles ou des noms de compte de stockage et ainsi de suite.

![Liste des paramètres dans Azure Resource Manager](media/test-drive/resource-manager-parameters.png)

Toutefois, la version d’évaluation fonctionne automatiquement, sans intervention humaine. Elle prend donc en charge un ensemble limité de catégories de paramètres. Si un paramètre dans le modèle ARM de version d’évaluation n’entre pas dans l’une des catégories prises en charge, vous devez le remplacer par une variable ou une valeur constante.

Vous pouvez utiliser n’importe quel nom valide pour vos paramètres ; la version d’évaluation reconnaît la catégorie d’un paramètre à l’aide d’une valeur metadata-type. Spécifiez metadata-type pour chaque paramètre de modèle, sinon votre modèle ne passera pas l’étape de validation :

```JSON
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username"
    }
  },
  ...
}
```

> [!NOTE]
> Tous les paramètres sont facultatifs. Vous n’êtes pas obligé d’en utiliser si vous ne le souhaitez pas.

### <a name="accepted-parameter-metadata-types"></a>Types de métadonnées de paramètres acceptés

| Type de métadonnées   | Type de paramètre  | Description     | Exemple de valeur    |
|---|---|---|---|
| **baseuri**     | string          | URI de base de votre package de déploiement| `https:\//\<\..\>.blob.core.windows.net/\<\..\>` |
| **username**    | string          | Nouveau nom d’utilisateur aléatoire.| admin68876      |
| **mot de passe**    | chaîne sécurisée    | Nouveau mot de passe aléatoire | Lp!ACS\^2kh     |
| **ID de la session**   | string          | ID de session unique de la version d’évaluation (GUID)    | b8c8693e-5673-449c-badd-257a405a6dee |

#### <a name="baseuri"></a>baseuri

La version d’évaluation initialise ce paramètre avec un **URI de base** de votre package de déploiement ; vous pouvez donc utiliser ce paramètre pour construire un URI de tous les fichiers inclus dans votre package.

```JSON
"parameters": {
  ...
  "baseuri": {
    "type": "string",
    "metadata": {
      "type": "baseuri",
      "description": "Base Uri of the deployment package."
    }
  },
  ...
}
```

Utilisez ce paramètre à l’intérieur de votre modèle pour construire un URI de n’importe quel fichier à partir du package de déploiement de votre version d’évaluation. L’exemple suivant montre comment construire un URI du modèle lié :

```JSON
"templateLink": {
  "uri": "[concat(parameters('baseuri'),'templates/solution.json')]",
  "contentVersion": "1.0.0.0"
}
```

#### <a name="username"></a>username

La version d’évaluation initialise ce paramètre avec un nouveau nom d’utilisateur aléatoire :

```JSON
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username",
      "description": "Solution admin name."
    }
  },
  ...
}
```

Exemple de valeur : `admin68876`

Vous pouvez utiliser des noms d’utilisateur aléatoires ou constants pour votre solution.

#### <a name="password"></a>mot de passe

La version d’évaluation initialise ce paramètre avec un nouveau mot de passe aléatoire :

```JSON
"parameters": {
  ...
  "password": {
    "type": "securestring",
    "metadata": {
      "type": "password",
      "description": "Solution admin password."
    }
  },
  ...
}
```

Exemple de valeur :  `Lp!ACS^2kh`

Vous pouvez utiliser des mots de passe aléatoires ou constants pour votre solution.

#### <a name="session-id"></a>ID de la session

La version d’évaluation initialise ce paramètre avec un GUID unique représentant l’ID de session de la version d’évaluation :

```JSON
"parameters": {
  ...
  "sessionid": {
    "type": "string",
    "metadata": {
      "type": "sessionid",
      "description": "Unique test drive session id."
    }
  },
  ...
}
```

Exemple de valeur : `b8c8693e-5673-449c-badd-257a405a6dee`

Vous pouvez utiliser ce paramètre pour identifier de façon unique la session de version d’évaluation, si nécessaire.

### <a name="unique-names"></a>Noms uniques

Certaines ressources Azure, comme les comptes de stockage ou les noms DNS, exigent des noms globalement uniques. Autrement dit, chaque fois que la version d’évaluation déploie le modèle ARM, elle crée un groupe de ressources avec un nom unique pour toutes ses ressources. Par conséquent, vous devez utiliser la fonction [uniquestring](../azure-resource-manager/templates/template-functions.md) concaténée avec vos noms de variables sur les ID de groupes de ressources pour générer des valeurs uniques aléatoires :

```JSON
"variables": {
  ...
  "domainNameLabel": "[concat('contosovm',uniquestring(resourceGroup().id))]",
  "storageAccountName": "[concat('contosodisk',uniquestring(resourceGroup().id))]",
  ...
}
```

Veillez à concaténer les chaînes de votre paramètre/variable (`contosovm`) avec une sortie de chaîne unique (`resourceGroup().id`), car cela garantit l’unicité et la fiabilité de chaque variable.

Par exemple, la plupart des noms de ressources ne peuvent pas commencer par un chiffre, mais la fonction de chaîne unique peut retourner une chaîne commençant par un chiffre. Par conséquent, si vous utilisez une sortie de chaîne unique brute, vos déploiements échoueront.

Vous trouverez des informations supplémentaires sur les règles et les restrictions d’affectation de noms de ressources dans [cet article](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).

### <a name="deployment-location"></a>Emplacement de déploiement

Vous pouvez rendre votre version d’évaluation disponible dans plusieurs régions Azure. L’idée est de permettre à un utilisateur de choisir la région la plus proche, afin de fournir la meilleure expérience utilisateur possible.

Quand la version d’évaluation crée une instance du laboratoire, elle crée toujours un groupe de ressources dans la région choisie par un utilisateur et elle exécute ensuite votre modèle de déploiement dans le contexte de ce groupe. Par conséquent, votre modèle doit choisir l’emplacement de déploiement à partir du groupe de ressources :

```JSON
"variables": {
  ...
  "location": "[resourceGroup().location]",
  ...
}
```

Utilisez ensuite cet emplacement pour chaque ressource d’une instance de laboratoire spécifique :

```JSON
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/publicIPAddresses",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/virtualNetworks",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/networkInterfaces",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Compute/virtualMachines",
    "location": "[variables('location')]",
    ...
  }
]
```

Vérifiez que votre abonnement est autorisé à déployer toutes les ressources souhaitées dans chacune des régions sélectionnées. Vérifiez aussi que vos images de machine virtuelle sont disponibles dans toutes les régions que vous activerez, sinon votre modèle de déploiement ne fonctionnera pas dans certaines régions.

### <a name="outputs"></a>Sorties

Normalement, avec des modèles Resource Manager, vous pouvez effectuer un déploiement sans produire de sortie. La raison est que vous connaissez toutes les valeurs que vous utilisez pour remplir les paramètres de modèle et vous pouvez toujours inspecter les propriétés de n’importe quelle ressource manuellement.

Toutefois, pour les modèles Resource Manager de version d’évaluation, il est important de retourner toutes les informations à la version d’évaluation ; elles sont exigées pour obtenir l’accès au laboratoire (URI de site web, noms d’hôte de machine virtuelle, noms d’utilisateur et mots de passe). Vérifiez que vos noms de sortie sont lisibles, car ces variables sont présentées au client.

Il n’existe aucune restriction liée aux sorties de modèle. La version d’évaluation convertit toutes les valeurs de sortie en chaînes. Par conséquent, si vous envoyez un objet vers la sortie, un utilisateur verra une chaîne JSON.

Exemple :

```JSON
"outputs": {
  "Host Name": {
    "type": "string",
    "value": "[reference(variables('pubIpId')).dnsSettings.fqdn]"
  },
  "User Name": {
    "type": "string",
    "value": "[parameters('adminName')]"
  },
  "Password": {
    "type": "string",
    "value": "[parameters('adminPassword')]"
  }
}
```

### <a name="subscription-limits"></a>Limites d’abonnement

N’oubliez pas les limites du service et de l’abonnement. Par exemple, si vous souhaitez déployer jusqu’à dix machines virtuelles à quatre cœurs, vous devez vous assurer que l’abonnement que vous utilisez pour votre laboratoire vous permet d’utiliser 40 cœurs. Pour plus d’informations sur les limites applicables aux services et aux abonnements Azure, consultez [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md). Étant donné que plusieurs versions d’évaluation peuvent être faites en même temps, vérifiez que votre abonnement peut gérer le nombre de cœurs multiplié par le nombre total de versions d’évaluation simultanées pouvant être faites.

### <a name="what-to-upload"></a>Les éléments à télécharger

Le modèle ARM de version d’évaluation est chargé sous la forme d’un fichier zip, pouvant inclure différents artefacts de déploiement, mais il doit inclure un fichier nommé `main-template.json`. Il s’agit du modèle de déploiement ARM utilisé par la version d’évaluation pour instancier un laboratoire. Si vous avez des ressources supplémentaires en plus de ce fichier, vous pouvez les référencer en tant que ressources externes à l’intérieur du modèle, ou les inclure dans le fichier zip.

Au cours de la certification de publication, la version d’évaluation décompresse votre package de déploiement et place son contenu dans un conteneur d’objets blob interne de la version d’évaluation. La structure du conteneur reflète la structure de votre package de déploiement :

| package.zip                       | Conteneur d’objets blob de la version d’évaluation         |
|---|---|
| `main-template.json`                | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/main-template.json`  |
| `templates/solution.json`           | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/templates/solution.json` |
| `scripts/warmup.ps1`                | `https:\//\<\...\>.blob.core.windows.net/\<\...\>/scripts/warmup.ps1`  |
|||

Un URI de ce conteneur d’objets blob est appelé URI de Base. Chaque révision de votre laboratoire ayant son propre conteneur d’objets blob, elle a son propre URI de base. La version d’évaluation peut passer un URI de base de votre package de déploiement décompressé dans votre modèle par le biais des paramètres de modèle.

### <a name="transform-template-examples-for-test-drive"></a>Exemples de transformation de modèle pour version d’évaluation

Le processus de transformation d’une architecture de ressources en modèle Resource Manager de version d’évaluation peut être décourageant. Pour obtenir une aide supplémentaire, consultez les exemples illustrant comment transformer au mieux les modèles de déploiement actuels dans l’article [Qu’est-ce qu’une version d’évaluation ?](what-is-test-drive.md#transforming-examples).

## <a name="test-drive-deployment-subscription-details"></a>Détails de l’abonnement de déploiement de la version d’évaluation

La dernière section à effectuer vise à permettre le déploiement automatique de versions d’évaluation en connectant votre abonnement Azure et Azure Active Directory (AD).

![Détails de l’abonnement de déploiement de la version d’évaluation](media/test-drive/deployment-subscription-details.png)

1. Obtenez un **ID d’abonnement Azure**. Ce paramètre accorde l’accès aux services Azure et au portail Azure. C’est dans l’abonnement que l’utilisation des ressources est signalée et que les services sont facturés. Si vous n’avez pas encore d’abonnement Azure dédié aux versions d’évaluation, créez-en un. Vous pouvez rechercher des ID d’abonnements Azure (par exemple `1a83645ac-1234-5ab6-6789-1h234g764ghty1`) en vous connectant au portail Azure et en sélectionnant **Abonnements** dans le menu de navigation de gauche.

   ![Abonnements Azure](media/test-drive/azure-subscriptions.png)

2. Obtenez un **ID de locataire Azure AD**. Si vous disposez déjà d’un ID de locataire, vous pouvez le trouver dans **Azure Active Directory** > **Propriétés** > **ID de répertoire** :

   ![Propriétés Azure Active Directory](media/test-drive/azure-active-directory-properties.png)

   Si vous n’avez pas d’ID de locataire, créez-en un dans Azure Active Directory. Pour obtenir de l’aide sur la configuration d’un locataire, consultez [Démarrage rapide : Configurer un locataire](../active-directory/develop/quickstart-create-new-tenant.md).

3. **ID application Azure AD** : créez et inscrivez une nouvelle application. Nous utiliserons cette application pour effectuer des opérations sur l’instance de votre version d’évaluation.

   1. Accédez au répertoire que vous venez de créer ou au répertoire existant, et sélectionnez Azure Active Directory dans le volet de filtre.
   2. Recherchez **Inscriptions d’applications** et sélectionnez **Ajouter**.
   3. Entrez un nom d’application.
   4. Sélectionnez le **Type** **Application/API web**.
   5. Spécifiez une valeur quelconque dans URL de connexion. Ce champ n’est pas utilisé.
   6. Sélectionnez **Create** (Créer).
   7. Une fois l’application créée, sélectionnez **Propriétés** > **Définir l’application comme étant multilocataire**, puis **Enregistrer**.

4. Sélectionnez **Enregistrer**.

5. Copiez l’ID de cette application inscrite et collez-le dans le champ de la version d’évaluation.

   ![Détails de l’ID d’application Azure AD](media/test-drive/azure-ad-application-id-detail.png)

6. Étant donné que nous utilisons l’application pour le déploiement sur l’abonnement, nous devons ajouter cette application en tant que contributeur dans l’abonnement :

   1. Sélectionnez le type d’**Abonnement** que vous utilisez pour la version d’évaluation.
   1. Sélectionnez **Contrôle d’accès (IAM)** .
   1. Sélectionnez l’onglet **Attributions de rôle**, puis **Ajouter une attribution de rôle**.

      ![Ajouter un nouveau principal de contrôle d’accès](media/test-drive/access-control-principal.jpg)

   1. Définissez **Rôle** et **Attribuer l’accès à** comme indiqué. Dans le champ **Sélectionner**, entrez le nom de l’application Azure AD. Sélectionnez l’application à laquelle vous souhaitez affecter le rôle de **Contributeur**.

      ![Ajouter les autorisations](media/test-drive/access-control-permissions.jpg)

   1. Sélectionnez **Enregistrer**.

7. Générez une clé d’authentification **Azure AD App**. Sous **Clés**, ajoutez une **Description de clé**, définissez la durée sur **N’expire jamais** (une clé expirée interrompt votre version d’évaluation en production), puis sélectionnez **Enregistrer**. Copiez cette valeur et collez-la dans le champ obligatoire de votre version d’évaluation.

![Affiche les clés de l’application Azure AD](media/test-drive/azure-ad-app-keys.png)

## <a name="republish"></a>Republier

Maintenant que tous les champs de version d’évaluation sont renseignés, **republiez** votre offre. Une fois que votre version d’évaluation a réussi la certification, testez l’expérience du client dans la préversion de votre offre :

1. Démarrez une version d’évaluation dans l’interface utilisateur.
1. Ouvrez votre abonnement Azure dans le portail Azure.
1. Vérifiez que votre version d’évaluation est correctement déployée.

   ![Portail Azure](media/test-drive/azure-portal.png)

Ne supprimez aucune instance de version d’évaluation provisionnée pour vos clients ; le service de version d’évaluation nettoiera automatiquement ces groupes de ressources une fois qu’un client aura terminé de les utiliser.

Une fois que vous vous sentez à l’aise avec votre offre en préversion, il est temps de la **mettre en service**. Un processus de vérification final permet de revérifier l’intégralité de l’expérience de bout en bout. Si nous rejetons l’offre, nous enverrons un e-mail au contact technique de votre offre expliquant ce qui doit être corrigé.

## <a name="next-steps"></a>Étapes suivantes

- Si vous suiviez les instructions pour créer votre offre dans l’Espace partenaires, utilisez la flèche Précédent pour revenir à cette rubrique.
- Apprenez-en davantage sur les autres types de versions d’évaluation dans [Qu’est-ce qu’une version d’évaluation ?](what-is-test-drive.md).