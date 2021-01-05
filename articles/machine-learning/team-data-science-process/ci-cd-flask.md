---
title: Créer un pipeline CI/CD avec Azure Pipelines - Processus TDSP
description: Créez un pipeline d’intégration continue et de livraison continue pour les applications d’intelligence artificielle (IA) à l’aide de Docker et Kubernetes.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=jainr, previous-ms.author=jainr
ms.openlocfilehash: af7900faff18d526686b80f23b9de3b3bce5ec5c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93309601"
---
# <a name="create-cicd-pipelines-for-ai-apps-using-azure-pipelines-docker-and-kubernetes"></a>Créer des pipelines CI/CD pour les applications IA à l’aide d’Azure Pipelines, de Docker et de Kubernetes

Une application d’intelligence artificielle (IA) est un code d’application incorporé avec un modèle de Machine Learning (ML) préformé. Il existe toujours deux flux de travail pour une application IA : Les scientifiques des données génèrent le modèle ML, tandis que les développeurs d’applications génèrent l’application et l’exposent aux utilisateurs finaux pour qu’ils les utilisent. Cet article explique comment implémenter un pipeline d’intégration continue et de livraison continue (CI/CD) pour une application IA qui incorpore le modèle ML dans le code source de l’application. L’exemple de code et le tutoriel utilisent une application web Python Flask et extraient un modèle préentraîné d’un compte de stockage de blobs Azure privé. Vous pouvez également utiliser un compte de stockage AWS S3.

> [!NOTE]
> Le processus suivant est l’un des nombreux moyens d’effectuer des opérations CI/CD. Il existe des alternatives à ces outils et aux prérequis.

## <a name="source-code-tutorial-and-prerequisites"></a>Code source, tutoriel et prérequis

Vous pouvez télécharger le [code source](https://github.com/Azure/DevOps-For-AI-Apps) et un [tutoriel détaillé](https://github.com/Azure/DevOps-For-AI-Apps/blob/master/Tutorial.md) à partir de GitHub. Suivez les étapes du tutoriel afin d’implémenter un pipeline CI/CD pour votre propre application.

Pour utiliser le code source et le tutoriel téléchargés, vous avez besoin de ce qui suit : 

- Le [dépôt du code source](https://github.com/Azure/DevOps-For-AI-Apps) dupliqué sur votre compte GitHub
- Une [organisation Azure DevOps](/azure/devops/organizations/accounts/create-organization-msa-or-work-student)
- [Azure CLI](/cli/azure/install-azure-cli)
- Un [cluster Azure Container Service pour Kubernetes (AKS)](/previous-versions/azure/container-service/kubernetes/container-service-tutorial-kubernetes-deploy-cluster)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/), pour exécuter des commandes et récupérer la configuration du cluster AKS 
- Un [compte Azure Container Registry (ACR)](../../container-registry/container-registry-get-started-portal.md)

## <a name="cicd-pipeline-summary"></a>Le pipeline CI/CD en quelques mots

Chaque nouvelle validation Git démarre le pipeline de build. La génération tire (pull) de manière sécurisée le dernier modèle ML d’un compte de stockage d’objets blob et l’empaquette avec le code d’application dans un seul conteneur. Ce découplage des flux de travail de la science des données et du développement d’applications garantit que l’application de production exécute toujours le code le plus récent avec le dernier modèle ML. Si l’application réussit le test, le pipeline stocke de manière sécurisée l’image de la génération dans un conteneur Docker au sein d’ACR. Le pipeline de mise en production déploie ensuite le conteneur à l’aide d’AKS. 

## <a name="cicd-pipeline-steps"></a>Étapes du pipeline CI/CD

Le diagramme et les étapes ci-dessous décrivent l’architecture du pipeline CI/CD :

![Architecture du pipeline CI/CD](./media/ci-cd-flask/architecture.png)

1. Les développeurs travaillent sur le code de l’application dans l’IDE de leur choix.
2. Les développeurs valident le code sur Azure Repos, GitHub ou un autre fournisseur de contrôle de code source Git. 
3. À part, les scientifiques des données travaillent sur le développement de leur modèle ML.
4. Les scientifiques des données publient le modèle terminé dans un dépôt de modèle, en l’occurrence un compte de stockage d’objets blob. 
5. Azure Pipelines lance une génération basée sur la validation Git.
6. Le pipeline de build tire (pull) le dernier modèle ML du stockage d’objets blob et crée un conteneur.
7. Le pipeline envoie (push) l’image de la génération vers le dépôt d’images privé dans ACR.
8. Le pipeline de mise en production démarre en fonction de la réussite de la génération.
9. Le pipeline tire la dernière image d’ACR et la déploie sur le cluster Kubernetes sur AKS.
10. Les demandes d’utilisateur pour l’application passent par le serveur DNS.
11. Le serveur DNS transfère les demandes à l’équilibreur de charge et renvoie les réponses aux utilisateurs.

## <a name="see-also"></a>Voir aussi

- [TDSP (Team Data Science Process)](./index.yml)
- [Azure Machine Learning (AML)](../index.yml)
- [Azure DevOps](https://azure.microsoft.com/services/devops/)
- [Azure Kubernetes Services (AKS)](../../aks/intro-kubernetes.md)