---
title: Configurer des données à grande échelle pour Azure Automation State Configuration
description: Cet article explique comment configurer des données à grande échelle pour Azure Automation State Configuration.
keywords: dsc,powershell,configuration,installation
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: bc55a4c5ab20cac041a00a0f924b207eb256ae8b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186518"
---
# <a name="configure-data-at-scale-for-azure-automation-state-configuration"></a>Configurer des données à grande échelle pour Azure Automation State Configuration

> S’applique à : Windows PowerShell 5.1

Il peut être très compliqué de gérer des centaines voire des milliers de serveurs.
Aux dires des clients, le plus difficile en réalité est de gérer les [données de configuration](/powershell/scripting/dsc/configurations/configdata).
Organiser les informations dans les différentes constructions logiques comme l’emplacement, le type et l’environnement.

> [!NOTE]
> Cet article fait référence à une solution tenue à jour par la communauté Open Source.
> La prise en charge est disponible uniquement sous forme de collaboration GitHub, et non auprès de Microsoft.

## <a name="community-project-datum"></a>Projet communautaire : Datum

Une solution tenue à jour par la communauté nommée [Datum](https://github.com/gaelcolas/Datum) a été créée pour relever ce défi.
Datum s’appuie sur les idées ingénieuses d’autres plateformes de gestion de configuration et implémente le même type de solution pour PowerShell DSC.
Les informations sont [organisées dans des fichiers texte](https://github.com/gaelcolas/Datum#3-intended-usage) selon des idées logiques.
Voici quelques exemples :

- Paramètres devant s’appliquer globalement
- Paramètres devant s’appliquer à tous les serveurs d’un site
- Paramètres devant s’appliquer à tous les serveurs de base de données
- Paramètres d’un serveur individuel

Ces informations sont organisées dans le format de fichier de votre choix (JSON, YAML ou PSD1).
Des applets de commande sont ensuite fournies pour générer des fichiers de données de configuration en [regroupant les informations](https://github.com/gaelcolas/Datum#datum-tree) de chaque fichier dans une même vue d’un serveur ou rôle de serveur.

Une fois que les fichiers de données ont été générés, vous pouvez les utiliser avec des [scripts de configuration DSC](/powershell/scripting/dsc/configurations/write-compile-apply-configuration) pour générer des fichiers MOF et [charger ces mêmes fichiers MOF dans Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Inscrivez ensuite vos serveurs [localement](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) ou [dans Azure](./automation-dsc-onboarding.md#enable-azure-vms) pour extraire les configurations.

Pour tester Datum, accédez à [PowerShell Gallery](https://www.powershellgallery.com/packages/datum/) et téléchargez la solution ou cliquez sur « Project Site » pour afficher la [documentation](https://github.com/gaelcolas/Datum#2-getting-started--concepts).

## <a name="next-steps"></a>Étapes suivantes

- Pour comprendre DSC PowerShell, consultez [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Découvrez les ressources DSC PowerShell dans [Ressources DSC](/powershell/scripting/dsc/resources/resources).
- Pour plus d’informations sur la configuration du Configuration Manager local, consultez [configuration du Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig).
