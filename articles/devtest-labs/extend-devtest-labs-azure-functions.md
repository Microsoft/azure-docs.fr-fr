---
title: Étendre Azure DevTest Labs à l’aide d’Azure Functions | Microsoft Docs
description: Découvrez comment étendre Azure DevTest Labs à l’aide d’Azure Functions.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: bcd12d77065d231198e992fa5c459f0fc210855a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476306"
---
# <a name="use-azure-functions-to-extend-devtest-labs"></a>Utiliser Azure Functions pour étendre DevTest Labs
Vous pouvez utiliser Azure Functions pour prendre en charge des scénarios en plus de ceux qui sont déjà pris en charge par DevTest Labs. Azure Functions permet d’étendre les fonctionnalités intégrées du service pour répondre à des besoins spécifiques de votre entreprise. La liste suivante présente certains des scénarios possibles. Cet article montre comment implémenter l’un de ces exemples de scénarios.

- Fournir une synthèse générale des machines virtuelles dans le labo
- [Configurer un labo pour utiliser une passerelle des services Bureau à distance](configure-lab-remote-desktop-gateway.md)
- Rapports de conformité sur la page Support interne
- Permettre aux utilisateurs d’effectuer des opérations qui nécessitent des autorisations accrues dans l’abonnement
- [Démarrage de workflows basés sur des événements DevTest Labs](https://github.com/RogerBestMsft/DTL-SecureArtifactData)

## <a name="overview"></a>Vue d’ensemble
[Azure Functions](../azure-functions/functions-overview.md) est une plateforme calcul Serverless dans Azure. L’utilisation d’Azure Functions dans une solution avec DevTest Labs nous permet d’augmenter les fonctionnalités existantes avec notre propre code personnalisé. Pour plus d’informations sur Azure Functions, voir la [Documentation Azure Functions](../azure-functions/functions-overview.md). Pour illustrer la façon dont Azure Functions peut vous aider à répondre à vos besoins ou à exécuter des scénarios complets dans DevTest Labs, cet article présente un exemple de fourniture de synthèse générale des machines virtuelles dans le labo comme suit :

**Exemple d’exigence/de scénario** : Les utilisateurs peuvent afficher des détails sur toutes les machines virtuelles d’un labo, y compris le système d’exploitation, le propriétaire et tous les artefacts appliqués.  De plus, si l’artefact **Appliquer les mises à jour Windows** n’a pas été appliqué récemment, il existe un moyen simple de l’appliquer.

Pour exécuter le scénario, vous allez utiliser deux fonctions indiquées dans le diagramme suivant :  

![Flux général](./media/extend-devtest-labs-azure-functions/flow.png)

Le code source de ces exemples de fonctions se trouve dans le [dépôt GitHub DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/AzureFunctions) (les implémentations C# et PowerShell sont disponibles).

- **UpdateInternalSupportPage** : Cette fonction interroge DevTest Labs et met à jour la page Support interne directement avec des détails sur les machines virtuelles.
- **ApplyWindowsUpdateArtifact** : Pour une machine virtuelle dans un labo, cette fonction applique l’artefact **Windows Update**.

## <a name="how-it-works"></a>Fonctionnement
Quand des utilisateurs sélectionnent la page de **Support interne** dans DevTest Labs, ils disposent d’une page pré-renseignée contenant des informations sur les machines virtuelles, les propriétaires de labos et les contacts du support.  

Lorsque vous actionnez le bouton **Cliquer ici pour actualiser**, la page appelle la première fonction Azure : **UpdateInternalSupportPage**. La fonction interroge DevTest Labs pour obtenir des informations, puis réécrit la page **Support interne** avec les nouvelles informations.

Pour exécuter une autre action sur toutes les machines virtuelles auxquelles les artefacts Windows Update n’ont pas été appliqués récemment, un bouton permet d’appliquer des mises à jour Windows à la machine virtuelle. Lorsque vous sélectionnez le bouton ***Exécuter Windows Update** pour une machine virtuelle, la page appelle la deuxième fonction Azure : **ApplyWindowsUpdateArtifact**. Cette fonction vérifie si la machine virtuelle est en cours d’exécution et, si c’est le cas, applique directement l’artefact [Windows Update](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-install-windows-updates).

## <a name="step-by-step-walkthrough"></a>Procédure pas à pas
Cette section fournit des instructions pas à pas pour la configuration des ressources Azure nécessaires pour mettre à jour la page **Support interne**. Cette procédure pas à pas fournit un exemple d’extension de DevTest Labs. Vous pouvez utiliser ce modèle pour d’autres scénarios.

### <a name="step-1-create-a-service-principal"></a>Étape 1 : Créer un principal du service 
La première étape consiste à obtenir un principal de service avec l’autorisation d’accès à l’abonnement contenant le labo. Le principal de service doit utiliser l’authentification par mot de passe. Cela est possible avec [Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest), [Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps?view=azps-2.5.0) ou le [portail Azure](../active-directory/develop/howto-create-service-principal-portal.md). Si vous avez déjà un principal de service à utiliser, vous pouvez ignorer cette étape.

Notez l’**ID d’application**, la **clé** et l’**ID de locataire** pour le principal de service. Vous en aurez besoin plus loin dans le cadre de cette procédure pas à pas. 

### <a name="step-2-download-the-sample-and-open-in-visual-studio-2019"></a>Étape 2 : Télécharger et ouvrir l’exemple dans Visual Studio 2019
Téléchargez une copie de l’[Exemple Azure Functions C#](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/AzureFunctions/CSharp) localement (en clonant le dépôt ou en le téléchargeant [à partir d’ici](https://github.com/Azure/azure-devtestlab/archive/master.zip)).  

1. Ouvrez l’exemple de solution avec Visual Studio 2019.  
1. Installez la charge de travail **Développement Azure** pour Visual Studio si vous ne l’avez pas encore fait. Vous pouvez l’installer via l’élément de menu **Outils** -> **Obtenir les outils et fonctionnalités**.

    ![Charge de travail de développement Azure](./media/extend-devtest-labs-azure-functions/azure-development-workload-vs.png)
1. Générez la solution. Cliquez sur **Build**, puis sur l’élément de menu **Générer la solution**.

### <a name="step-3-deploy-the-sample-to-azure"></a>Étape 3 : Déployer l’exemple sur Azure
Dans Visual Studio, dans la fenêtre **Explorateur de solutions**, cliquez avec le bouton droit sur le projet **AzureFunctions**, puis sélectionnez **Publier**. Suivez les instructions de l’Assistant pour publier les fonctions dans une Application de fonction Azure nouvelle ou existante. Pour plus d’informations sur le développement et le déploiement de fonctions Azure à l’aide de Visual Studio, voir [Développer Azure Functions à l’aide de Visual Studio](../azure-functions/functions-develop-vs.md).

![Boîte de dialogue Publier](./media/extend-devtest-labs-azure-functions/publish-dialog.png)


### <a name="step-4--gather-application-settings"></a>Étape 4 :  Collecter les paramètres d’application
Une fois les fonctions publiées, vous devez obtenir leurs URL à partir du portail Azure. 

1. Accédez au [portail Azure](https://portal.azure.com). 
1. Recherchez l’application de fonction.
1. Dans la page **Applications de fonctions**, sélectionnez la fonction. 
1. Sélectionnez **Obtenir l’URL de la fonction** comme illustré dans l’image suivante. 

    ![URL de fonctions Azure](./media/extend-devtest-labs-azure-functions/function-url.png)
4. Copiez et enregistrez l’URL. Répétez ces étapes pour l’autre fonction Azure. 

Vous aurez également besoin d’informations supplémentaires sur le principal de service, telles que l’ID d’application, la clé et l’ID de locataire.


### <a name="step-5--update-application-settings"></a>Étape 5 :  Mettre à jour les paramètres d’application
Dans Visual Studio, après publication de la fonction Azure, sous **Actions**, sélectionnez l’option **Modifier les paramètres d’Azure App Service**. Mettez à jour les paramètres d’application suivants (à distance) :

- AzureFunctionUrl_ApplyUpdates
- AzureFunctionUrl_UpdateSupportPage
- WindowsUpdateAllowedDays (7 par défaut)
- ServicePrincipal_AppId
- ServicePrincipal_Key
- ServicePrincipal_Tenant

    ![Paramètres de l’application](./media/extend-devtest-labs-azure-functions/application-settings.png)

### <a name="step-6-test-the-azure-function"></a>Étape 6 : Tester la fonction Azure
La dernière étape de cette procédure pas à pas consiste à tester la fonction Azure.  

1. Accédez à la fonction **UpdateInternalSupportPage** dans l’application de fonction créée à l’étape 3. 
1. Sélectionnez **Tester** du côté droit de la page. 
1. Entrez les propriétés de l’itinéraire (LABNAME, RESOURCEGROUPNAME et SUBSCRIPTIONID).
1. Sélectionnez **Exécuter** pour exécuter la fonction.  

    Cette fonction met à jour la page Support interne du labo spécifié. Elle inclut également un bouton permettant aux utilisateurs d’appeler directement la fonction une prochaine fois.

    ![Tester la fonction](./media/extend-devtest-labs-azure-functions/test-function.png)

## <a name="next-steps"></a>Étapes suivantes
Azure Functions peut vous aider à étendre les fonctionnalités de DevTest Labs au-delà de ce qui est déjà intégré, ainsi qu’aider des clients à répondre aux besoins uniques de leurs équipes. Ce modèle peut être étendu et développé davantage pour en couvrir encore davantage de fonctions.  Pour plus d’informations sur DevTest Labs, voir les articles suivants : 

- [Architecture de référence d’entreprise pour Azure DevTest Labs](devtest-lab-reference-architecture.md)
- [Questions fréquentes (FAQ)](devtest-lab-faq.md)
- [Montée en puissance de DevTest Labs](devtest-lab-guidance-scale.md)
- [Automatisation de DevTest Labs avec PowerShell](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Modules/Library/Tests)








