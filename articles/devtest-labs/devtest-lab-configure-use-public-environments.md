---
title: Configurer et utiliser des environnements publics dans Azure DevTest Labs | Microsoft Docs
description: Cet article explique comment configurer et utiliser des environnements publics (modèles Azure Resource Manager dans un référentiel Git) dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 61cabdb296c3fff75137c7ce7e87652241fd2926
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85482664"
---
# <a name="configure-and-use-public-environments-in-azure-devtest-labs"></a>Configurer et utiliser des environnements publics dans Azure DevTest Labs
Azure DevTest Labs a un [dépôt public de modèles Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments), que vous pouvez utiliser pour créer des environnements sans devoir vous connecter à une source GitHub externe par vous-même. Ce dépôt inclut des modèles fréquemment utilisés, comme Azure Web Apps, Cluster Service Fabric et l’environnement de batterie de serveurs SharePoint de développement. Cette fonctionnalité est similaire au dépôt public d’artefacts qui est inclus pour chaque lab que vous créez. Le dépôt d’environnements vous permet de vous familiariser rapidement avec des modèles d’environnement préalablement créés avec un nombre minimal de paramètres d’entrée, pour un démarrage en douceur avec les ressources PaaS au sein des labs. 

## <a name="configuring-public-environments"></a>Configuration des environnements publics
En tant que propriétaire d’un lab, vous pouvez activer le dépôt public d’environnements pour votre lab lors de la création du lab. Pour activer des environnements publics pour votre lab, sélectionnez **Activé** pour le champ **Environnements publics** lors de la création d’un lab. 

![Activer un environnement public pour un nouveau lab](media/devtest-lab-configure-use-public-environments/enable-public-environment-new-lab.png)


Pour les labs existants, le dépôt d’environnements publics n’est pas activé. Activez-le manuellement pour utiliser des modèles du dépôt. Pour les labs créés avec des modèles Resource Manager, le dépôt est également désactivé par défaut.

Vous pouvez activer/désactiver des environnements publics pour votre lab, ainsi que mettre seulement certains environnements spécifiques à la disposition des utilisateurs d’un lab en effectuant les étapes suivantes : 

1. Sélectionnez **Configuration et stratégies** pour votre lab. 
2. Dans la section **BASES DE MACHINE VIRTUELLE**, sélectionnez **Environnements publics**.
3. Pour activer les environnements publics pour le lab, sélectionnez **Oui**. Sinon, sélectionnez **Non**. 
4. Si vous avez activé les environnements publics, tous les environnements du dépôt sont activés par défaut. Vous pouvez désélectionner un environnement afin que celui-ci ne soit plus disponible pour les utilisateurs de votre lab. 

![Page Environnements publics](media/devtest-lab-configure-use-public-environments/public-environments-page.png)

## <a name="use-environment-templates-as-a-lab-user"></a>Utiliser des modèles d’environnement en tant qu’utilisateur de lab
En tant qu’utilisateur de lab, vous pouvez créer un environnement à partir de la liste activée des modèles d’environnement en sélectionnant simplement **+Ajouter** dans la barre d’outils de la page du lab. La liste des bases inclut les modèles d’environnements publics activés par l’administrateur de votre lab en haut de la liste.

![Modèles d’environnement public](media/devtest-lab-configure-use-public-environments/public-environment-templates.png)

## <a name="next-steps"></a>Étapes suivantes
Ce dépôt est un dépôt open source auquel vous pouvez contribuer en ajoutant des modèles Resource Manager fréquemment utilisés et pratiques que vous avez créés vous-même. Pour contribuer, envoyez simplement une demande de tirage (pull request) sur le dépôt.  
