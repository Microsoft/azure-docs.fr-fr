---
title: Déployer des environnements de modèles imbriqués dans Azure DevTest Labs
description: Découvrez comment déployer des modèles Azure Resource Manager imbriqués pour fournir des environnements avec Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 39002e286fafd4f813333a14ed86256a517897e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85481338"
---
# <a name="deploy-nested-azure-resource-manager-templates-for-testing-environments"></a>Déployer des modèles Azure Resource Manager imbriqués pour les environnements de test
Un déploiement imbriqué vous permet d’exécuter d’autres modèles Azure Resource Manager à partir d’un modèle Resource Manager principal. Il vous permet de décomposer votre déploiement en un ensemble de modèles ciblés et spécifiques. Il offre des avantages en termes de test, de réutilisation et de lisibilité. L’article [Utilisation de modèles liés lors du déploiement des ressources Azure](../azure-resource-manager/templates/linked-templates.md) fournit une bonne vue d’ensemble de cette solution avec plusieurs exemples de code. Cet article fournit un exemple spécifique d’Azure DevTest Labs. 

## <a name="key-parameters"></a>Paramètres clés
Lorsque vous créez votre propre modèle Resource Manager à partir de zéro, nous vous recommandons d’utiliser le [projet Groupe de ressources Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) dans Visual Studio, ce qui facilite le développement et le débogage des modèles. Lorsque vous ajoutez une ressource de déploiement imbriquée dans azuredeploy.json, Visual Studio ajoute plusieurs éléments pour rendre le modèle plus flexible. Ces éléments incluent le sous-dossier avec le modèle secondaire et le fichier de paramètres, des noms de variables dans le fichier de modèle principal, et deux paramètres pour l’emplacement de stockage des nouveaux fichiers. Les paramètres **_artifactsLocation** et **_artifactsLocationSasToken** sont les paramètres clés utilisés par DevTest Labs. 

Si vous ne savez pas comment DevTest Labs fonctionne avec les environnements, consultez [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md). Vos modèles sont stockés dans le référentiel lié au laboratoire dans DevTest Labs. Lorsque vous créez un environnement avec ces modèles, les fichiers sont déplacés dans un conteneur Stockage Azure dans le laboratoire. Pour être en mesure d’identifier et de copier les fichiers imbriqués, DevTest Labs identifie les paramètres _artifactsLocation et _artifactsLocationSasToken et copie les sous-dossiers dans le conteneur de stockage. Ensuite, il insère automatiquement l’emplacement et le jeton de signature d’accès partagé (SAP) dans les paramètres. 

## <a name="nested-deployment-example"></a>Exemple de déploiement imbriqué
Voici un exemple simple de déploiement imbriqué :

```json

"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": {
    "_artifactsLocation": {
        "type": "string"
    },
    "_artifactsLocationSasToken": {
        "type": "securestring"
    }},
"variables": {
    "NestOneTemplateFolder": "nestedtemplates",
    "NestOneTemplateFileName": "NestOne.json",
    "NestOneTemplateParametersFileName": "NestOne.parameters.json"},
    "resources": [
    {
        "name": "NestOne",
        "type": "Microsoft.Resources/deployments",
        "apiVersion": "2016-09-01",
        "dependsOn": [ ],
        "properties": {
            "mode": "Incremental",
            "templateLink": {
                "uri": "[concat(parameters('_artifactsLocation'), '/', variables('NestOneTemplateFolder'), '/', variables('NestOneTemplateFileName'), parameters('_artifactsLocationSasToken'))]",
                "contentVersion": "1.0.0.0"
            },
            "parametersLink": {
                "uri": "[concat(parameters('_artifactsLocation'), '/', variables('NestOneTemplateFolder'), '/', variables('NestOneTemplateParametersFileName'), parameters('_artifactsLocationSasToken'))]",
                "contentVersion": "1.0.0.0"
            }
        }    
    }],
"outputs": {}
```

Le dossier dans le référentiel contenant ce modèle comporte un sous-dossier `nestedtemplates` avec les fichiers **NestOne.json** et **NestOne.parameters.json**. Dans le fichier **azuredeploy.json**, l’URI du modèle est généré au moyen de l’emplacement des artefacts, du dossier du modèle imbriqué, et du nom du fichier du modèle imbriqué. De même, l’URI des paramètres est généré au moyen de l’emplacement des artefacts, du dossier du modèle imbriqué et du fichier de paramètres pour le modèle imbriqué. 

Voici l’image de la même structure de projet dans Visual Studio : 

![Structure de projet dans Visual Studio](./media/deploy-nested-template-environments/visual-studio-project-structure.png)

Vous pouvez ajouter des dossiers dans le dossier principal, mais à un seul niveau maximum. 

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur les environnements, consultez les articles suivants : 

- [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md)
- [Configurer et utiliser des environnements publics dans Azure DevTest Labs](devtest-lab-configure-use-public-environments.md)
- [Connecter un environnement au réseau virtuel de votre lab dans Azure DevTest Labs](connect-environment-lab-virtual-network.md)