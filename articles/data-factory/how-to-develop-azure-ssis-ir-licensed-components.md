---
title: Installer des composants sous licence pour le runtime d’intégration Azure-SSIS
description: Découvrez comment un éditeur de logiciels indépendant peut développer et installer des composants personnalisés payants ou sous licence pour le runtime d’intégration Azure-SSIS.
services: data-factory
ms.service: data-factory
ms.workload: data-services
author: swinarko
ms.author: sawinark
manager: mflasko
ms.reviewer: douglasl
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/09/2020
ms.openlocfilehash: 77eedbfc65b54ce128e1adbd93375bc624ef38cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86187611"
---
# <a name="install-paid-or-licensed-custom-components-for-the-azure-ssis-integration-runtime"></a>Installer des composants personnalisés payants, ou sous licence, pour le runtime d’intégration Azure-SSIS

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article explique comment un éditeur de logiciels indépendant peut développer et installer des composants personnalisés payants ou sous licence pour les packages SQL Server Integration Services (SSIS) exécutés dans Azure dans le runtime d’intégration Azure-SSIS.

## <a name="the-problem"></a>Le problème

La nature du runtime d’intégration Azure-SSIS présente plusieurs difficultés, qui rendent les méthodes standard de gestion des licences utilisées pour l’installation en local des composants personnalisés inappropriées. Par conséquent, le runtime d’intégration Azure-SSIS nécessite une approche différente.

-   Les nœuds du runtime d’intégration Azure-SSIS sont volatils et peuvent être alloués ou libérés à tout moment. Par exemple, vous pouvez démarrer ou arrêter des nœuds pour gérer le coût ou procéder à une montée ou une descente en puissance sur différentes tailles de nœuds. Par conséquent, la liaison d’une licence de composants tiers à un nœud donné à l’aide d’informations spécifiques à la machine comme l’adresse MAC ou l’ID du processeur n’est plus viable.

-   Vous pouvez également augmenter ou diminuer la taille des instances du runtime d’intégration Azure-SSIS, ce qui vous permet de réduire ou développer le nombre de nœuds à tout moment.

## <a name="the-solution"></a>La solution

En raison des limitations des méthodes traditionnelles de gestion des licences décrites dans la section précédente, l’IR Azure-SSIS fournit une nouvelle solution. Cette solution utilise les variables d’environnement Windows et du système SSIS dédiés à la liaison et la validation des licences pour les composants tiers. Les éditeurs de logiciels indépendants peuvent utiliser ces variables pour obtenir des données uniques et persistantes associées à un runtime d’intégration Azure-SSIS, comme l’ID de cluster et le nombre de nœuds de cluster. Avec ces informations, les éditeurs de logiciels indépendants peuvent lier la licence de leur composant à un IR Azure-SSIS *en tant que cluster*. Cette liaison utilise un ID qui ne change pas lorsque les clients démarrent ou arrêtent, effectuent un scale-up, un scale-down, un scale-in ou un scale-out, ou reconfigurent l’IR Azure-SSIS.

Le schéma suivant représente l’installation typique, l’activation et la liaison des licences, ainsi que les flux de validation pour les composants tiers qui utilisent ces nouvelles variables :

![Installation des composants sous licence](media/how-to-configure-azure-ssis-ir-licensed-components/licensed-component-installation.png)

## <a name="instructions"></a>Instructions
1. Les éditeurs de logiciels indépendants peuvent proposer leurs composants sous licence sous diverses références ou niveaux (par exemple, nœud unique, jusqu’à 5 nœuds, jusqu’à 10 nœuds, etc.). L’éditeur fournit la clé de produit correspondante lorsque les clients acquièrent un produit. Il peut également octroyer le conteneur d’objets blob Stockage Azure, qui comporte le script d’installation éditeur et les fichiers associés. Les clients peuvent copier ces fichiers dans leur propre conteneur de stockage et les modifier avec leur propre clé de produit (par exemple, en exécutant `IsvSetup.exe -pid xxxx-xxxx-xxxx`). Ils sont ensuite en mesure de provisionner ou de reconfigurer le runtime d’intégration Azure-SSIS avec l’URI SAS de leur conteneur en tant que paramètre. Pour plus d’informations, consultez [Custom setup for the Azure-SSIS integration runtime](how-to-configure-azure-ssis-ir-custom-setup.md) (Configuration personnalisée du runtime d’intégration Azure-SSIS).

2. Lors du provisionnement ou de la reconfiguration du runtime d’intégration Azure-SSIS, le script d’installation éditeur est exécuté sur chaque nœud afin d’interroger les variables d’environnement Windows, `SSIS_CLUSTERID` et `SSIS_CLUSTERNODECOUNT`. Ensuite, le runtime d’intégration Azure-SSIS transmet son ID de cluster et la clé du produit sous licence au serveur d’activation éditeur, afin de générer une clé d’activation.

3. Une fois la clé d’activation reçue, le script d’installation éditeur peut la stocker en local sur chaque nœud (par exemple, dans le Registre).

4. Lorsque les clients exécutent un package utilisant le composant sous licence de l’éditeur sur un nœud du runtime d’intégration Azure-SSIS, le package lit la clé d’activation stockée en local et la valide en la comparant à l’ID de cluster du nœud. Le package peut également, éventuellement, transmettre le nombre de nœuds du cluster au serveur d’activation éditeur.

    Voici un exemple de code qui valide la clé d’activation et transmet le nombre de nœuds du cluster :

    ```csharp
    public override DTSExecResult Validate(Connections, VariableDispenser, IDTSComponentEvents componentEvents, IDTSLogging log) 
                                                                                                                               
    {                                                                                                                             
                                                                                                                               
    Variables vars = null;                                                                                                        
                                                                                                                               
    variableDispenser.LockForRead("System::ClusterID");                                                                           
                                                                                                                               
    variableDispenser.LockForRead("System::ClusterNodeCount");                                                                    
                                                                                                                               
    variableDispenser.GetVariables(ref vars);                                                                                     
                                                                                                                               
    // Validate Activation Key with ClusterID                                                                                     
                                                                                                                               
    // Report on ClusterNodeCount                                                                                                 
                                                                                                                               
    vars.Unlock();                                                                                                                
                                                                                                                               
    return base.Validate(connections, variableDispenser, componentEvents, log);                                                   
                                                                                                                               
    }
    ```

## <a name="isv-partners"></a>Partenaires éditeurs de logiciels indépendants

Vous trouverez une liste des partenaires éditeurs de logiciels indépendants qui ont adapté leurs composants et extensions pour l’IR Azure-SSIS à la fin de ce billet de blog - [Enterprise Edition, installation personnalisée et extensibilité tierce pour SSIS dans ADF](https://techcommunity.microsoft.com/t5/SQL-Server-Integration-Services/Enterprise-Edition-Custom-Setup-and-3rd-Party-Extensibility-for/ba-p/388360).

## <a name="next-steps"></a>Étapes suivantes

-   [Installation personnalisée du runtime d’intégration Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md)

-   [Enterprise Edition du runtime d’intégration Azure-SSIS](how-to-configure-azure-ssis-ir-enterprise-edition.md)
