---
title: CI/CD pour Gestion des API Azure à l’aide de modèles Resource Manager
description: Introduction à l’API DevOps avec gestion des API Azure, à l’aide de modèles de Azure Resource Manager pour gérer les déploiements d’API dans un pipeline CI/CD
services: api-management
author: miaojiang
ms.service: api-management
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: apimpm
ms.openlocfilehash: 62f163b9ce649cd5ddb52b4325682570633dfb92
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183156"
---
# <a name="cicd-for-api-management-using-azure-resource-manager-templates"></a>CI/CD pour la gestion des API à l’aide de modèles de Azure Resource Manager

Cet article explique comment utiliser l’API DevOps avec Gestion des API Azure, à l’aide de modèles Azure Resource Manager. Avec la valeur stratégique des API, un pipeline d’intégration continue et de déploiement continu (CI/CD) est devenu un aspect important du développement des API. Elle permet aux organisations d’automatiser le déploiement des modifications d’API sans étapes manuelles sujettes aux erreurs, de détecter les problèmes plus tôt et de fournir aux utilisateurs une valeur plus rapide. 

Pour plus d’informations, d’outils et d’exemples de code pour implémenter l’approche DevOps décrite dans cet article, consultez le[Kit de ressources Azure API Management DevOps](https://github.com/Azure/azure-api-management-devops-resource-kit) Open source dans GitHub. Étant donné que les clients apportent un large éventail de cultures d’ingénierie et de solutions d’automatisation existantes, l’approche n’est pas une solution adaptée à une seule taille.

## <a name="the-problem"></a>La problématique

Aujourd’hui, les organisations disposent généralement de plusieurs environnements de déploiement (tels que le développement, les tests et la production) et utilisent des instances Gestion des API distinctes pour chaque environnement. Certaines instances sont partagées par plusieurs équipes de développement, qui sont responsables de différentes API avec des cadences de version différentes.

Par conséquent, les clients sont confrontés aux défis suivants :

* Comment automatiser le déploiement d’API dans Gestion des API
* Comment migrer des configurations d’un environnement à un autre
* Comment éviter les interférences entre les différentes équipes de développement qui partagent la même instance Gestion des API

## <a name="manage-configurations-in-resource-manager-templates"></a>Gérer les configurations dans les modèles Resource Manager

L’approche proposée est illustrée dans l’image suivante. 

:::image type="content" source="media/devops-api-development-templates/apim-devops.png" alt-text="Diagramme illustrant DevOps avec Gestion des API.":::

Dans cet exemple, il existe deux environnements de déploiement : *Développement* et *Production*. Chacun possède sa propre instance gestion des API. 

* Les développeurs d’API ont accès à l’instance de développement et peuvent l’utiliser pour développer et tester leurs API. 
* Une équipe désignée appelée *Éditeurs d’API* gère l’instance de production.

La clé de cette approche proposée consiste à conserver toutes les configurations de gestion des API dans les [modèles de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). L’organisation doit conserver ces modèles dans un système de contrôle de code source tel que Git. Comme illustré dans l’image, un référentiel de publication contient toutes les configurations de l’instance Gestion des API de production dans une collection de modèles :

|Modèle  |Description  |
|---------|---------|
|Modèle de service     | Configurations au niveau du service de l’instance Gestion des API, telles que le niveau tarifaire et les domaines personnalisés.         |
|Modèles partagés     |  Ressources partagées dans une instance Gestion des API, telles que des groupes, des produits et des enregistreurs d’événements.    |
|Modèles d’API     |  Configurations des API et de leurs sous-ressources : opérations, stratégies, paramètres de diagnostic.        |
|Modèle principal     |   Lie tout ensemble en [faisant le lien](../azure-resource-manager/templates/linked-templates.md) entre tous les modèles et leur déploiement dans l’ordre. Pour déployer toutes les configurations sur une instance de gestion des API, déployez le modèle principal. Vous pouvez également déployer chaque modèle individuellement.       |

Les développeurs d’API vont répliquer le référentiel de l’éditeur vers un référentiel de développement et travailler sur les modifications pour leurs API. Dans la plupart des cas, ils se concentrent sur les modèles d’API pour leurs API et n’ont pas besoin de modifier les modèles de service ou partagés.

## <a name="migrate-configurations-to-templates"></a>Migrer des configurations vers des modèles
Les développeurs d’API rencontrent des difficultés lors de l’utilisation de modèles Resource Manager :

* Les développeurs d’API travaillent souvent avec la [spécification OpenAPI](https://github.com/OAI/OpenAPI-Specification) et peuvent ne pas connaître les schémas Resource Manager. La création manuelle de modèles peut être sujette aux erreurs. 

   Un outil appelé [Creator](https://github.com/Azure/azure-api-management-devops-resource-kit/blob/master/src/APIM_ARMTemplate/README.md#Creator) dans le kit de ressources peut aider à automatiser la création de modèles d’API basés sur un fichier de spécification OpenAPI. En outre, les développeurs peuvent fournir des stratégies de gestion des API pour une API au format XML. 

* Pour les clients qui utilisent déjà la gestion des API, une autre difficulté consiste à extraire les configurations existantes dans des modèles Resource Manager. Pour ces clients, un outil appelé [Extracteur](https://github.com/Azure/azure-api-management-devops-resource-kit/blob/master/src/APIM_ARMTemplate/README.md#extractor) dans le kit de ressources peut aider à générer des modèles en extrayant des configurations de leurs instances de gestion des API.  

## <a name="workflow"></a>Workflow

* Une fois que les développeurs d’API ont terminé le développement et le test d’une API et qu’ils ont généré les modèles d’API, ils peuvent envoyer une demande de tirage (pull request) pour fusionner les modifications dans le référentiel de l’éditeur. 

* Les éditeurs d’API peuvent valider la demande de tirage (pull request) et vérifier que les modifications sont sûres et conformes. Par exemple, ils peuvent vérifier si seul le protocole HTTPs est autorisé à communiquer avec l’API. La plupart des validations peuvent être automatisées comme une étape dans le pipeline CI/CD.

* Une fois les modifications approuvées et fusionnées avec succès, les éditeurs d’API peuvent choisir de les déployer sur l’instance de production selon la planification ou à la demande. Le déploiement des modèles peut être automatisé à l’aide de [GitHub Actions](https://github.com/Azure/apimanagement-devops-samples), [Azure Pipelines](/azure/devops/pipelines), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md), [Azure CLI](../azure-resource-manager/templates/deploy-cli.md)ou d’autres outils.


Avec cette approche, une organisation peut automatiser le déploiement des modifications d’API dans les instances Gestion des API, et il est facile de promouvoir les modifications d’un environnement à un autre. Étant donné que différentes équipes de développement d’API travaillent sur différents ensembles de modèles et de fichiers d’API, cela empêche les interférences entre les différentes équipes.

## <a name="video"></a>Vidéo

> [!VIDEO https://www.youtube.com/embed/4Sp2Qvmg6j8]

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des informations, des outils et des exemples de modèles, consultez [ le kit de ressourcesAzure API Management DevOps](https://github.com/Azure/azure-api-management-devops-resource-kit) Open source.