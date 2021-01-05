---
title: Déployer des applications avec des modèles
description: Trouvez des conseils pour créer des modèles Azure Resource Manager en vue de provisionner et déployer des applications App Service.
author: tfitzmac
ms.topic: article
ms.date: 01/03/2019
ms.author: tomfitz
ms.custom: seodec18
ms.openlocfilehash: 1146b5979d81b91c6c6894aa54b2e0ca50c896c1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88961616"
---
# <a name="guidance-on-deploying-web-apps-by-using-azure-resource-manager-templates"></a>Aide au déploiement d’applications web avec des modèles Azure Resource Manager

Cet article propose des recommandations sur la création de modèles Azure Resource Manager dans le but de déployer des solutions Azure App Service. Ces recommandations peuvent vous aider à éviter les problèmes courants.

## <a name="define-dependencies"></a>Définir des dépendances

La définition de dépendances pour des applications web implique de comprendre comment les ressources interagissent au sein d’une application web. Si vous ne spécifiez pas les dépendances dans le bon ordre, vous risquez de provoquer des erreurs de déploiement ou de créer une condition de concurrence qui entrave le déploiement.

> [!WARNING]
> Si vous intégrez une extension de site MSDeploy à votre modèle, vous devez définir toutes les ressources de configuration comme dépendantes de la ressource MSDeploy. Les modifications de configuration entraînent le redémarrage asynchrone du site. En rendant les ressources de configuration dépendantes de MSDeploy, vous permettez à MSDeploy de se terminer avant que le site ne redémarre. Sans ces dépendances, le site risquerait de redémarrer pendant le processus de déploiement de MSDeploy. Vous trouverez un exemple de modèle sur la page [Modèle WordPress avec une dépendance Web Deploy](https://github.com/davidebbo/AzureWebsitesSamples/blob/master/ARMTemplates/WordpressTemplateWebDeployDependency.json).

L’image suivante montre l’ordre de dépendance de différentes ressources App Service :

![Dépendances des applications web](media/web-sites-rm-template-guidance/web-dependencies.png)

Déployez des ressources dans l’ordre suivant :

**Niveau 1**
* Plan App Service.
* Toutes les autres ressources associées, comme les bases de données ou les comptes de stockage.

**Niveau 2**
* Application web (dépend du plan App Service).
* Instance Azure Application Insights ciblant la batterie de serveurs (dépend du plan App Service).

**Niveau 3**
* Contrôle de code source (dépend de l’application web).
* Extension de site MSDeploy (dépend de l’application web).
* Instance Azure Application Insights ciblant l’application web (dépend de l’application web).

**Niveau 4**
* App Service Certificate (dépend du contrôle de code source ou de MSDeploy si l’un ou l’autre est présent ; sinon, dépend de l’application web).
* Paramètres de configuration, comme les chaînes de connexion, les valeurs web.config et les paramètres d’application (dépend du contrôle de code source ou de MSDeploy si l’un ou l’autre est présent ; sinon, dépend de l’application web).

**Niveau 5**
* Liaisons du nom d’hôte (dépend du certificat s’il est présent ; sinon, dépend d’une ressource de niveau supérieur).
* Extensions de site (dépend des paramètres de configuration s’ils sont présents ; sinon, dépend d’une ressource de niveau supérieur).

En règle générale, votre solution inclut uniquement une partie de ces ressources et niveaux. Pour les niveaux manquants, mappez les ressources de niveau inférieur sur le niveau immédiatement supérieur.

L’exemple suivant montre une partie d’un modèle. La valeur de configuration de la chaîne de connexion dépend de l’extension MSDeploy. L’extension MSDeploy dépend de l’application web et de la base de données. 

```json
{
    "name": "[parameters('appName')]",
    "type": "Microsoft.Web/Sites",
    ...
    "resources": [
      {
          "name": "MSDeploy",
          "type": "Extensions",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', parameters('appName'))]",
            "[concat('Microsoft.Sql/servers/', parameters('dbServerName'), '/databases/', parameters('dbName'))]",
          ],
          ...
      },
      {
          "name": "connectionstrings",
          "type": "config",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', parameters('appName'), '/Extensions/MSDeploy')]"
          ],
          ...
      }
    ]
}
```

Pour un exemple prêt à exécuter incluant le code ci-dessus, consultez [Modèle : Créer une application web Umbraco simple](https://github.com/Azure/azure-quickstart-templates/tree/master/umbraco-webapp-simple).

## <a name="find-information-about-msdeploy-errors"></a>Rechercher des informations sur les erreurs MSDeploy

Si votre modèle Resource Manager utilise MSDeploy, les messages d’erreur de déploiement peuvent être difficiles à comprendre. Pour obtenir plus d’informations suite à un échec de déploiement, essayez les étapes suivantes :

1. Accédez au site de la [console Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console).
2. Accédez au dossier à D:\home\LogFiles\SiteExtensions\MSDeploy.
3. Recherchez les fichiers appManagerStatus.xml et appManagerLog.xml. Le premier fichier enregistre l’état. Le second fichier enregistre des informations sur l’erreur. Si vous ne comprenez pas l’erreur, vous pouvez l’inclure dans votre demande d’aide sur le [forum](/answers/topics/azure-webapps.html).

## <a name="choose-a-unique-web-app-name"></a>Choisir un nom unique pour l’application web

Le nom de votre application web doit être globalement unique. Vous pouvez utiliser une convention d’affectation de noms qui favorise les noms uniques ou la [fonction uniqueString](../azure-resource-manager/templates/template-functions-string.md#uniquestring) pour générer un nom unique.

```json
{
  "apiVersion": "2016-08-01",
  "name": "[concat(parameters('siteNamePrefix'), uniqueString(resourceGroup().id))]",
  "type": "Microsoft.Web/sites",
  ...
}
```

## <a name="deploy-web-app-certificate-from-key-vault"></a>Déployer un certificat d’application web à partir de Key Vault

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si votre modèle inclut une ressource [Microsoft.Web/certificates](/azure/templates/microsoft.web/certificates) pour la liaison TLS/SSL et que le certificat est stocké dans une instance Key Vault, vous devez vous assurer que l’identité App Service peut accéder au certificat.

Dans Azure, le principal du service App Service est doté de l’ID **abfa0a7c-a6b6-4736-8310-5855508787cd**. Pour accorder l’accès à Key Vault au principal du service App Service, utilisez :

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy `
  -VaultName KEY_VAULT_NAME `
  -ServicePrincipalName abfa0a7c-a6b6-4736-8310-5855508787cd `
  -PermissionsToSecrets get `
  -PermissionsToCertificates get
```

Dans Azure Government, le principal du service App Service est doté de l’ID **6a02c803-dafd-4136-b4c3-5a6f318b4714**. Utilisez cet ID dans l’exemple précédent.

Dans votre instance Key Vault, sélectionnez **Certificats** et **Générer/Importer** pour charger le certificat.

![Importation d’un certificat](media/web-sites-rm-template-guidance/import-certificate.png)

Dans votre modèle, indiquez le nom du certificat pour le `keyVaultSecretName`.

Pour un exemple de modèle, consultez [Deploy a Web App certificate from Key Vault secret and use it for creating SSL binding](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault) (Déployer un certificat d’application web à partir d’un secret Key Vault et l’utiliser pour créer une liaison SSL).

## <a name="next-steps"></a>Étapes suivantes

* Pour suivre un tutoriel sur le déploiement d’applications web avec un modèle, consultez [Provisionner et déployer des microservices de manière prévisible dans Azure](deploy-complex-application-predictably.md).
* Pour découvrir la syntaxe et les propriétés JSON pour les types de ressources dans les modèles, voir la [Documentation de référence concernant le modèle Azure Resource Manager](/azure/templates/).