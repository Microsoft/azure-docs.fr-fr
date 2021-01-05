---
title: Créer un environnement orienté applications - Azure
description: Cet article explique comment créer un environnement orienté applications avec Cloud Shell Colony et Microsoft Azure.
ms.topic: how-to
ms.date: 11/26/2020
ms.openlocfilehash: 88244f268d5ed038e9bb7082d3d5cc1179e5ec4e
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917981"
---
# <a name="create-an-application-centric-environment"></a>Créer un environnement axé sur les applications

[CloudShell Colony de Quali](https://azuremarketplace.microsoft.com/marketplace/apps/quali_systems.cloudshell_colony?tab=Overview) est une plateforme SaaS qui permet une automatisation d'infrastructure à grande échelle pour les environnements complexes orientés applications sur des technologies cloud, comme Azure et Kubernetes. CloudShell Colony complète Azure DevTest Labs pour aider les équipes de développeurs à déployer des applications complexes tout au long de la chaîne de valeur, jusqu'à la production.

Cet article explique comment créer un environnement orienté applications avec CloudShell Colony et Microsoft Azure.

## <a name="set-up-the-environment-with-cloudshell-colony-and-microsoft-azure"></a>Configurer l'environnement avec CloudShell Colony et Microsoft Azure

1. Inscrivez-vous pour bénéficier d'un essai gratuit de [Colony](https://azuremarketplace.microsoft.com/marketplace/apps/quali_systems.cloudshell_colony?tab=Overview).

    :::image type="content" source="./media/create-application-centric-environment/free-trial.png" alt-text="Inscrivez-vous pour un essai gratuit":::    
1. Liez votre compte Azure ([voir les étapes ici](https://colonysupport.quali.com/hc/articles/360008222234)).

    :::image type="content" source="./media/create-application-centric-environment/welcome.png" alt-text="Bienvenue dans Colony":::     
1. Invitez des utilisateurs dans votre espace.
1. Créez votre premier blueprint à l'aide d'un fichier YAML ([voir les étapes ici](https://colonysupport.quali.com/hc/articles/360001680807-Steps-to-Developing-a-Blueprint)).
    1. Liez votre référentiel de blueprints GitHub ou BitBucket à Colony.
    1. Utilisez un exemple de blueprint Colony comme base et modifiez-le en fonction de vos besoins.

        :::image type="content" source="./media/create-application-centric-environment/performance-stress-tests.png" alt-text="Tests de contrainte":::    
    1. Publiez votre blueprint pour le mettre à la disposition d'autres utilisateurs.
1. Lancez votre environnement d'applications dans un bac à sable à l'aide de Colony.

    :::image type="content" source="./media/create-application-centric-environment/blueprints.png" alt-text="Lancez votre environnement d'applications dans un bac à sable à l'aide de Colony":::    

> [!NOTE]
> Vous pouvez également intégrer votre blueprint dans le cadre d'un workflow CI/CD dans Azure DevOps ([voir les étapes ici](https://colonysupport.quali.com/hc/articles/360008464234)).

:::image type="content" source="./media/create-application-centric-environment/devops-pipeline.png" alt-text="Connectez-vous à votre pipeline Azure DevOps":::    

## <a name="next-steps"></a>Étapes suivantes

[Demander une démo de Colony](https://info.quali.com/cloudshell-colony-demo-request)
