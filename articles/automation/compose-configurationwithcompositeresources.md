---
title: Composer des configurations DSC
description: Cet article explique comment composer des configurations à l’aide de ressources composites dans Azure Automation State Configuration.
keywords: powershell dsc, desired state configuration, configuration d’état souhaité, powershell dsc azure, ressources composites
services: automation
ms.subservice: dsc
ms.date: 08/21/2018
ms.topic: conceptual
ms.openlocfilehash: 9c5524d2847f59dda1f2c24f67e1e18f18d49b1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86185566"
---
# <a name="compose-dsc-configurations"></a>Composer des configurations DSC

Lorsqu'il vous faut gérer une ressource avec plusieurs configurations DSC, la meilleure solution consiste à utiliser des [ressources composites](/powershell/scripting/dsc/resources/authoringresourcecomposite). Une ressource composite est une configuration de paramètres imbriqués utilisée comme ressource DSC au sein d’une autre configuration. Les ressources composites vous permettent de créer des configurations complexes tout en autorisant la création de ressources composites sous-jacentes et leur gestion à titre individuel.

Azure Automation permet [d’importer et de compiler des ressources composites](automation-dsc-compile.md). Après avoir importé des ressources composites dans votre compte Automation, vous pouvez utiliser Azure Automation State Configuration via la fonctionnalité **State Configuration (DSC)** du portail Azure.

## <a name="compose-a-configuration"></a>Composer une configuration

Avant de pouvoir attribuer une configuration effectuée à partir de ressources composites dans le portail Microsoft Azure, vous devez composer cette configuration. La composition utilise **Composer la configuration** sur la page State Configuration (DSC), à partir des onglets **Configurations** ou **Configurations compilées**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, sélectionnez **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State configuration (DSC), cliquez sur l’onglet **Configurations** ou **Configurations compilées**, puis sur **Composer la configuration** dans le menu en haut de la page.
1. Au moment de l’étape relative aux **fonctions de base**, indiquez le nom de la nouvelle configuration (obligatoire), cliquez n’importe où sur la ligne de chaque ressource composite que vous souhaitez inclure dans la nouvelle configuration, puis cliquez sur **Suivant** ou sur l’étape **Code source**. Pour les étapes suivantes, nous avons sélectionné les ressources composites `PSExecutionPolicy` et `RenameAndDomainJoin`.
   ![Capture d’écran de l’étape des fonctions de base sur la page Composer la configuration](./media/compose-configurationwithcompositeresources/compose-configuration-basics.png)
1. L’étape **Code source** montre à quoi ressemble la configuration composée des ressources composites sélectionnées. Vous pouvez voir la fusion de tous les paramètres et la façon dont ils sont transmis à la ressource composite. Après avoir examiné le nouveau code source, cliquez sur **Suivant** ou sur l’étape **Paramètres**.
   ![Capture d’écran de l’étape du code source sur la page Composer la configuration](./media/compose-configurationwithcompositeresources/compose-configuration-sourcecode.png)
1. Au niveau de l’étape **Paramètres**, le paramètre de chaque ressource composite est exposé afin de fournir les valeurs. Si un paramètre possède une description, celle-ci s’affiche à côté du champ de paramètre. Si un paramètre est de type `PSCredential`, la liste déroulante fournit une liste d'objets **Informations d’identification** dans le compte Automation actuel. Une option **+ Add a credential** (+ Ajouter une information d’identification) est également disponible. Une fois que tous les paramètres requis ont été fournis, cliquez sur **Save and compile** (Enregistrer et compiler).
   ![Capture d’écran de l’étape des paramètres sur la page Composer la configuration](./media/compose-configurationwithcompositeresources/compose-configuration-parameters.png)

## <a name="submit-the-configuration-for-compilation"></a>Envoyer la configuration pour compilation

Une fois la nouvelle configuration enregistrée, elle est envoyée en compilation. Vous pouvez afficher l’état de la tâche de compilation comme vous le feriez avec n’importe quelle configuration importée. Pour plus d’informations, consultez [Afficher une tâche de compilation](automation-dsc-getting-started.md#view-a-compilation-job).

Si la compilation aboutit, la nouvelle configuration s’affiche dans l’onglet **Configurations compilées**. Vous pouvez ensuite affecter la configuration à un nœud géré à l’aide de la procédure décrite dans [Réaffectation d’un nœud à une configuration de nœud différente](automation-dsc-getting-started.md#reassign-a-node-to-a-different-node-configuration).

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment activer des nœuds, consultez [Activer State Configuration](automation-dsc-onboarding.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, consultez [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour voir un exemple d’utilisation d’Azure Automation State Configuration dans un pipeline de déploiement continu, consultez [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
