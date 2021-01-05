---
title: Prise en main d'Azure Automation State Configuration
description: Cet article explique comment accomplir les tâches les plus courantes dans Azure Automation configuration de l’État.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.date: 04/15/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 0e98078fe2799d1c524190c8ced36588f35498b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186433"
---
# <a name="get-started-with-azure-automation-state-configuration"></a>Prise en main d'Azure Automation State Configuration

Cet article fournit un guide pas à pas pour l’exécution des tâches les plus courantes avec Azure Automation State Configuration, comme la création, l’importation et la compilation de configurations, l’habilitation d’ordinateurs pour la gestion, et la consultation de rapports. Pour une vue d’ensemble de State Configuration, consultez [Vue d’ensemble de State Configuration](automation-dsc-overview.md). Pour accéder à une documentation de Desired State Configuration (DSC), consultez l’article [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).

Si vous souhaitez obtenir un exemple d’environnement préconfiguré sans avoir à suivre les étapes décrites dans cet article, vous pouvez utiliser le [modèle Azure Automation Managed Node](https://github.com/Azure/azure-quickstart-templates/tree/master/101-automation-configuration). Ce modèle configure un environnement State Configuration (DSC) complet, incluant une machine virtuelle Azure gérée par State Configuration (DSC).

## <a name="prerequisites"></a>Prérequis

Pour exécuter les exemples de cet article, vous devez disposer des éléments suivants :

- Un compte Azure Automation. Pour obtenir des instructions sur la création d’un compte d’identification Azure Automation, consultez [Authentifier des Runbooks avec un compte d’identification Azure](./manage-runas-account.md).
- Une machine virtuelle Azure Resource Manager (non classique) exécutant un [système d’exploitation pris en charge](automation-dsc-overview.md#operating-system-requirements). Pour obtenir des instructions sur la création d’une machine virtuelle, consultez [Créer votre première machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/quick-create-portal.md)

## <a name="create-a-dsc-configuration"></a>Création d’une configuration DSC

Vous créez une [configuration DSC](/powershell/scripting/dsc/configurations/configurations) simple qui garantit la présence ou l’absence de la fonctionnalité Windows **Serveur web** (IIS), selon le mode d’attribution des nœuds.

1. Démarrez [VSCode](https://code.visualstudio.com/docs) (ou n’importe quel éditeur de texte).
1. Saisissez le texte suivant :

    ```powershell
    configuration TestConfig
    {
        Node IsWebServer
        {
            WindowsFeature IIS
            {
                Ensure               = 'Present'
                Name                 = 'Web-Server'
                IncludeAllSubFeature = $true
            }
        }

        Node NotWebServer
        {
            WindowsFeature IIS
            {
                Ensure               = 'Absent'
                Name                 = 'Web-Server'
            }
        }
    }
    ```
1. Enregistrez le fichier sous le nom **TestConfig.ps1**.

Cette configuration appelle une ressource dans chaque bloc de nœuds, la [ressource WindowsFeature](/powershell/scripting/dsc/reference/resources/windows/windowsfeatureresource). Cette ressource garantit la présence ou l'absence de la fonctionnalité **Web-Server**.

## <a name="import-a-configuration-into-azure-automation"></a>Importer une configuration dans Azure Automation

Ensuite, vous importez la configuration dans le compte Automation.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, sélectionnez **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Configurations**, puis sur **Ajouter**.
1. Sur le volet Importer la configuration, recherchez le fichier `TestConfig.ps1` sur votre ordinateur.

   ![Capture d’écran du panneau **Importer la configuration**](./media/automation-dsc-getting-started/AddConfig.png)

1. Cliquez sur **OK**.

## <a name="view-a-configuration-in-azure-automation"></a>Afficher une configuration dans Azure Automation

Après avoir importé une configuration, vous pouvez l’afficher dans le portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, sélectionnez **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Configurations**, puis sur **TestConfig**. Il s'agit du nom de la configuration que vous avez importée dans la procédure précédente.
1. Sur le volet Configuration de TestConfig, cliquez sur **Afficher la source de configuration**.

   ![Capture d’écran du panneau Configuration de TestConfig](./media/automation-dsc-getting-started/ViewConfigSource.png)

   Le volet source Configuration de TestConfig s'ouvre, affichant le code PowerShell de la configuration.

## <a name="compile-a-configuration-in-azure-automation"></a>Compiler une configuration dans Azure Automation

Avant de pouvoir appliquer un état souhaité à un nœud, vous devez compiler une configuration DSC définissant cet état dans une ou plusieurs configurations de nœuds (documents MOF) et la placer sur le serveur Pull Automation DSC. Pour obtenir une description plus détaillée de la compilation des configurations dans State Configuration (DSC), consultez [Compiler des configurations dans Azure Automation State Configuration](automation-dsc-compile.md).
Pour plus d’informations sur la compilation des configurations, consultez [Configurations DSC](/powershell/scripting/dsc/configurations/configurations).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Configurations**, puis sur **TestConfig**. Il s'agit du nom de la configuration précédemment importée.
1. Sur le volet Configuration de TestConfig, cliquez sur **Compiler**, puis sur **Oui**. Une tâche de compilation démarre.

   ![Capture d’écran de la page Configuration de TestConfig mettant en évidence le bouton de compilation](./media/automation-dsc-getting-started/CompileConfig.png)

> [!NOTE]
> Lorsque vous compilez une configuration dans Azure Automation, tous les fichiers MOF de configuration de nœud créés sont automatiquement déployés sur le serveur Pull.

## <a name="view-a-compilation-job"></a>Afficher une tâche de compilation

Après avoir démarré une compilation, vous pouvez l'afficher sur la mosaïque **Travaux de compilation** de la page **Configuration**. La mosaïque **Tâches de compilation** affiche les tâches en cours d’exécution, terminées et en échec. Lorsque vous ouvrez le volet d'une tâche de compilation, vous obtenez des informations sur cette tâche, notamment les erreurs ou les avertissements rencontrés, les paramètres d'entrée utilisés dans la configuration et les journaux de compilation.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Configurations**, puis sur **TestConfig**. Il s'agit du nom de la configuration précédemment importée.
1. Sous **Tâches de compilation**, sélectionnez la tâche de compilation à afficher. Un volet Tâche de compilation s'ouvre en indiquant la date de démarrage de la tâche.

   ![Capture d’écran de la page Tâche de compilation](./media/automation-dsc-getting-started/CompilationJob.png)

1. Cliquez sur n'importe quelle mosaïque du volet Tâche de compilation pour afficher des détails supplémentaires sur la tâche.

## <a name="view-node-configurations"></a>Afficher des configurations de nœud

La réussite d’une tâche de compilation a pour effet de créer une ou plusieurs configurations de nœud. Une configuration de nœud est un document MOF déployé sur le serveur Pull et prêt à être extrait et appliqué par un ou plusieurs nœuds. Vous pouvez afficher les configurations de nœuds de votre compte Automation sur la page State Configuration (DSC). Le nom d'une configuration de nœud se présente sous la forme `ConfigurationName.NodeName`.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Configurations compilées**.

   ![Capture d’écran de l’onglet Configurations compilées](./media/automation-dsc-getting-started/NodeConfigs.png)

## <a name="enable-an-azure-resource-manager-vm-for-management-with-state-configuration"></a>Activer une machine virtuelle Azure Resource Manager pour la gestion avec State Configuration

State Configuration vous permet de gérer vos machines virtuelles Azure (par le biais des modèles de déploiement classique et Resource Manager), vos machines virtuelles locales, vos machines Linux, vos machines virtuelles AWS et vos ordinateurs physiques locaux. Cet article explique comment n’intégrer que des machines virtuelles Azure Resource Manager. Pour plus d’informations sur l’intégration d’autres types de machines, consultez [Activer la gestion des machines par Azure Automation State Configuration](automation-dsc-onboarding.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), sélectionnez l'onglet **Nœuds**, puis cliquez sur **+ Ajouter**.

   ![Capture d’écran de la page Nœuds DSC mettant en évidence le bouton Ajouter des machines virtuelles Azure](./media/automation-dsc-getting-started/OnboardVM.png)

1. Sur le volet Machines virtuelles, sélectionnez votre machine virtuelle.
1. Sur le volet d'informations de la machine virtuelle, cliquez sur **+ Connecter**.

   > [!IMPORTANT]
   > Il doit s'agir d'une machine virtuelle Azure Resource Manager exécutant un [système d'exploitation pris en charge](automation-dsc-overview.md#operating-system-requirements).

2. Sur la page Inscription, sélectionnez le nom de la configuration de nœud à appliquer à la machine virtuelle dans la zone **Nom de la configuration de nœud**. Vous n’êtes pas obligé de fournir un nom à ce stade. Vous pouvez modifier la configuration de nœud assignée une fois le nœud activé.

3. Cochez la case **Redémarrer le nœud si nécessaire**, puis cliquez sur **OK**.

   ![Capture d’écran du panneau Inscription](./media/automation-dsc-getting-started/RegisterVM.png)

   La configuration de nœud que vous avez spécifiée est appliquée à la machine virtuelle aux intervalles spécifiés par la valeur fournie pour le paramètre **Fréquence du mode de configuration**. La machine virtuelle recherche les mises à jour de la configuration de nœud aux intervalles spécifiés par la valeur du paramètre **Fréquence d'actualisation**. Pour plus d’informations sur la façon dont ces valeurs sont utilisées, consultez [Configuration du Gestionnaire de configuration local](/powershell/scripting/dsc/managing-nodes/metaConfig).

Azure lance le processus d’activation de la machine virtuelle. Au terme de ce processus, la machine virtuelle apparaît dans l'onglet **Nœuds** de la page State Configuration (DSC) du compte Automation.

## <a name="view-the-list-of-managed-nodes"></a>Afficher la liste des nœuds gérés

Vous pouvez afficher la liste de tous les ordinateurs habilités à gérer dans votre compte Automation sous l'onglet **Nœuds** de la page State Configuration (DSC).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Nœuds**.

## <a name="view-reports-for-managed-nodes"></a>Afficher des rapports pour des nœuds gérés

Chaque fois que State Configuration effectue une vérification de cohérence sur un nœud géré, le nœud renvoie un rapport d’état au serveur Pull. Vous pouvez consulter ces rapports sur la page correspondant à ce nœud.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Nœuds**. Ici, vous pouvez voir la vue d’ensemble de l’état de la configuration et les détails de chaque nœud.

   ![Capture d’écran de la page des nœuds](./media/automation-dsc-getting-started/NodesTab.png)

1. Sur l’onglet **Nœuds**, cliquez sur l’enregistrement de nœud pour ouvrir les rapports. Cliquez sur le rapport pour lequel vous souhaitez afficher des détails supplémentaires.

   ![Capture d’écran du panneau Rapport](./media/automation-dsc-getting-started/NodeReport.png)

Le panneau d’un rapport vous permet d’accéder aux informations d’état suivantes pour vous permettre d’effectuer la vérification de cohérence correspondante :

- L'état du rapport. Les valeurs possibles sont les suivantes :
    * Conforme : le nœud est conforme à la vérification.
   * Échec : la vérification de la configuration a échoué.
   * Non conforme : le nœud est en mode `ApplyandMonitor` et la machine n'est pas à l'état souhaité.
- L’heure de début de la vérification de cohérence.
- Le temps total d’exécution de la vérification de cohérence.
- Le type de vérification de cohérence.
- Toute erreur rencontrée, avec code d’erreur et message d’erreur.
- Toutes les ressources DSC utilisées dans la configuration, ainsi que l'état de chaque ressource (si le nœud est à l'état souhaité pour cette ressource). Vous pouvez cliquer sur chaque ressource pour obtenir des informations plus détaillées sur celle-ci.
- Le nom, l’adresse IP et le mode de configuration du nœud.

Vous pouvez également cliquer sur **Afficher le rapport brut** pour afficher les données que le nœud envoie effectivement au serveur.
Pour plus d’informations sur l’utilisation de ces données, consultez la page [Utilisation d’un serveur de rapports DSC](/powershell/scripting/dsc/pull-server/reportserver).

Il peut s'écouler un certain temps après l'activation d'un nœud avant que le premier rapport soit disponible. Vous devrez peut-être attendre le premier rapport jusqu'à 30 minutes après l’activation d’un nœud.

## <a name="reassign-a-node-to-a-different-node-configuration"></a>Réaffecter un nœud à une configuration de nœud différente

Vous pouvez attribuer un nœud à une configuration de nœud différente de celle qui lui a été initialement affectée.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Nœuds**.
1. Sous l’onglet **Nœuds**, cliquez sur le nom du nœud que vous souhaitez réaffecter.
1. Sur la page de ce nœud, cliquez sur **Attribuer une configuration de nœuds**.

    ![Capture d’écran de la page de détails des nœuds mettant en évidence le bouton Attribuer une configuration de nœuds](./media/automation-dsc-getting-started/AssignNode.png)

1. Sur la page Attribuer une configuration de nœud, sélectionnez la configuration de nœud à laquelle vous souhaitez attribuer le nœud, puis cliquez sur **OK**.

    ![Capture d’écran de la page Attribuer une configuration de nœuds](./media/automation-dsc-getting-started/AssignNodeConfig.png)

## <a name="unregister-a-node"></a>Désinscrire un nœud

Si vous ne souhaitez plus qu’un nœud soit géré par Azure Automation DSC, vous pouvez le désinscrire.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, cliquez sur **Toutes les ressources**, puis sur le nom de votre compte Automation.
1. Sur la page Compte Automation, cliquez sur **State Configuration (DSC)** sous **Gestion de la configuration**.
1. Sur la page State Configuration (DSC), cliquez sur l'onglet **Nœuds**.
1. Sous l’onglet **Nœuds**, cliquez sur le nom du nœud dont vous voulez annuler l’inscription.
1. Sur le volet de ce nœud, cliquez sur **Annuler l'inscription**.

    ![Capture d’écran de la page de détails des nœuds mettant en évidence le bouton Annuler l’inscription](./media/automation-dsc-getting-started/UnregisterNode.png)

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble, consultez [Vue d’ensemble d’Azure Automation State Configuration](automation-dsc-overview.md).
- Pour activer la fonctionnalité pour les machines virtuelles de votre environnement, consultez [Activer Azure Automation State Configuration](automation-dsc-onboarding.md).
- Pour comprendre DSC PowerShell, consultez [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
