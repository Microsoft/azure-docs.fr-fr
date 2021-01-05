---
title: Convertir des configurations en ressources composites pour Azure Automation State Configuration
description: Cet article explique comment convertir des configurations en ressources composites pour Azure Automation State Configuration.
keywords: dsc,powershell,configuration,installation
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8c834caa2285135b7d39c440489b42c366418042
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186467"
---
# <a name="convert-configurations-to-composite-resources"></a>Convertir des configurations en ressources composites

> S’applique à : Windows PowerShell 5.1

Une fois que vous avez commencé à créer des configurations, vous pouvez créer rapidement des « scénarios » qui gèrent les groupes de paramètres.
Voici quelques exemples :

- création d’un serveur web
- création d’un serveur DNS
- création d’un serveur SharePoint
- configuration d’un cluster SQL
- gestion des paramètres de pare-feu
- gestion des paramètres de mot de passe

Si vous souhaitez partager ce travail avec d’autres personnes, la meilleure solution consiste à empaqueter la configuration sous forme de [ressource composite](/powershell/scripting/dsc/resources/authoringresourcecomposite).
Il peut être déroutant de créer des ressources composites pour la première fois.

> [!NOTE]
> Cet article fait référence à une solution tenue à jour par la communauté Open Source.
> La prise en charge est disponible uniquement sous forme de collaboration GitHub, et non auprès de Microsoft.

## <a name="community-project-compositeresource"></a>Projet communautaire : CompositeResource

Une solution tenue à jour par la communauté nommée [CompositeResource](https://github.com/microsoft/compositeresource) a été créée pour relever ce défi.

CompositeResource automatise la création d’un module à partir de votre configuration.
Commencez par faire un appel de source de type [dot sourcing](https://devblogs.microsoft.com/scripting/how-to-reuse-windows-powershell-functions-in-scripts/) pour le script de configuration sur votre station de travail (ou serveur de builds) afin de le charger en mémoire.
Ensuite, au lieu d’exécuter la configuration pour générer un fichier MOF, utilisez la fonction fournie par le module CompositeResource pour automatiser une conversion.
L’applet de commande charge le contenu de votre configuration, obtient la liste de paramètres et génère un nouveau module avec tout ce dont vous avez besoin.

Une fois que vous avez généré un module, vous pouvez incrémenter la version et ajouter des notes de publication chaque fois que vous apportez des modifications et que vous les publiez dans votre propre [référentiel PowerShellGet](https://powershellexplained.com/2018-03-03-Powershell-Using-a-NuGet-server-for-a-PSRepository/?utm_source=blog&utm_medium=blog&utm_content=psscriptrepo).

Après avoir créé un module de ressources composites contenant votre configuration (ou plusieurs configurations), vous pouvez les utiliser dans l’[expérience de création composable](./compose-configurationwithcompositeresources.md) d’Azure ou les ajouter à des [scripts de configuration DSC](/powershell/scripting/dsc/configurations/configurations) pour générer des fichiers MOF et [charger ces mêmes fichiers MOF dans Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Inscrivez ensuite vos serveurs [localement](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) ou [dans Azure](./automation-dsc-onboarding.md#enable-azure-vms) pour extraire les configurations.
À l’occasion de la dernière mise à jour du projet, des [runbooks](https://www.powershellgallery.com/packages?q=DscGallerySamples) ont aussi été publiés pour Azure Automation afin d’automatiser l’importation de configurations à partir de PowerShell Gallery.

Pour tester la création automatique de ressources composites pour DSC, accédez à [PowerShell Gallery](https://www.powershellgallery.com/packages/compositeresource/) et téléchargez la solution ou cliquez sur « Project Site » pour afficher la [documentation](https://github.com/microsoft/compositeresource).

## <a name="next-steps"></a>Étapes suivantes

- Pour comprendre DSC PowerShell, consultez [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Découvrez les ressources DSC PowerShell dans [Ressources DSC](/powershell/scripting/dsc/resources/resources).
- Pour plus d’informations sur la configuration du Configuration Manager local, consultez [Configuration du Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig).
