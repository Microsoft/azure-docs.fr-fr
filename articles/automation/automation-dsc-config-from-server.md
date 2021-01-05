---
title: Créer des configurations à partir de serveurs existants pour Azure Automation State Configuration
description: Cet article explique comment créer des configurations à partir de serveurs existants pour Azure Automation State Configuration.
keywords: dsc,powershell,configuration,installation
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8f376fb383e50a39f0f12d45cf9b5ae47ad6fcbb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186501"
---
# <a name="create-configurations-from-existing-servers"></a>Créer des configurations à partir de serveurs existants

> S’applique à : Windows PowerShell 5.1

La création de configurations à partir de serveurs existants peut être une tâche délicate.
Vous ne voudrez peut-être pas y faire figurer *tous* les paramètres, mais simplement ceux qui vous intéressent.
Même dans ce cas, vous devrez savoir dans quel ordre les paramètres doivent être appliqués pour permettre une bonne application de la configuration.

> [!NOTE]
> Cet article fait référence à une solution tenue à jour par la communauté Open Source.
> La prise en charge est disponible uniquement sous forme de collaboration GitHub, et non auprès de Microsoft.

## <a name="community-project-reversedsc"></a>Projet communautaire : ReverseDSC

Une solution tenue à jour par la communauté nommée [ReverseDSC](https://github.com/microsoft/reversedsc) a été créée pour fonctionner dans la partie démarrant SharePoint.

La solution s’appuie sur la [ressource SharePointDSC](https://github.com/powershell/sharepointdsc) et l’étend pour orchestrer la [collecte d’informations](https://github.com/Microsoft/sharepointDSC.reverse#how-to-use) auprès de serveurs SharePoint existants.
La dernière version propose plusieurs [modes d’extraction](https://github.com/Microsoft/SharePointDSC.Reverse/wiki/Extraction-Modes) pour déterminer le niveau d’information à inclure.

L’utilisation de la solution a pour effet de générer les [données de configuration](https://github.com/Microsoft/sharepointDSC.reverse#configuration-data) à utiliser avec des scripts de configuration SharePointDSC.

Une fois que les fichiers de données ont été générés, vous pouvez les utiliser avec des [scripts de configuration DSC](/powershell/scripting/dsc/overview/overview) pour générer des fichiers MOF et [charger ces mêmes fichiers MOF dans Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Inscrivez ensuite vos serveurs [localement](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) ou [dans Azure](./automation-dsc-onboarding.md#enable-azure-vms) pour extraire les configurations.

Pour tester ReverseDSC, accédez à [PowerShell Gallery](https://www.powershellgallery.com/packages/ReverseDSC/) et téléchargez la solution ou cliquez sur « Project Site » pour afficher la [documentation](https://github.com/Microsoft/sharepointDSC.reverse).

## <a name="next-steps"></a>Étapes suivantes

- Pour comprendre DSC PowerShell, consultez [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Découvrez les ressources DSC PowerShell dans [Ressources DSC](/powershell/scripting/dsc/resources/resources).
- Pour plus d’informations sur la configuration du Configuration Manager local, consultez [configuration du Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig).
