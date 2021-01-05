---
title: Tutoriel pour créer, générer et déployer des contrats intelligents - Azure Blockchain Service
description: Tutoriel expliquant comment utiliser le kit de développement Azure Blockchain pour l’extension Ethereum dans Visual Studio Code pour créer, générer et déployer un contrat intelligent dans Azure Blockchain Service.
ms.date: 11/30/2020
ms.topic: tutorial
ms.reviewer: caleteet
ms.openlocfilehash: f7605a0c118a40e52210582d2411569795fb25ee
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763687"
---
# <a name="tutorial-create-build-and-deploy-smart-contracts-on-azure-blockchain-service"></a>Tutoriel : Créer, générer et déployer des contrats intelligents sur Azure Blockchain Service

Dans ce tutoriel, vous allez utiliser le kit de développement Azure Blockchain pour l’extension Ethereum dans Visual Studio Code pour créer, générer et déployer un contrat intelligent dans Azure Blockchain Service. Vous allez aussi utiliser le kit de développement pour exécuter une fonction de contrat intelligent via une transaction.

Vous utilisez Azure Blockchain Development Kit pour Ethereum afin de :

> [!div class="checklist"]
> * Créer un contrat intelligent
> * Déployer un contrat intelligent
> * Exécuter une fonction de contrat intelligent par le biais d’une transaction

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Effectuer l’étape [Démarrage rapide : Utiliser Visual Studio Code pour se connecter à un réseau de consortium Azure Blockchain Service](connect-vscode.md)
* [Visual Studio Code](https://code.visualstudio.com/Download)
* [Azure Blockchain Development Kit pour l’extension Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain)
* [Node.js 10.15.x ou ultérieur](https://nodejs.org/download)
* [Git 2.10.x ou ultérieur](https://git-scm.com)
* [Truffle 5.0.0](https://www.trufflesuite.com/docs/truffle/getting-started/installation)
* [Ganache CLI 6.0.0](https://github.com/trufflesuite/ganache-cli)

Sur Windows, un compilateur C++ doit être installé pour le module node-gyp. Vous pouvez utiliser les outils MSBuild :

* Si Visual Studio 2017 est installé, configurez npm pour utiliser les outils MSBuild avec la commande `npm config set msvs_version 2017 -g`
* Si Visual Studio 2019 est installé, définissez le chemin des outils de build MS pour npm. Par exemple : `npm config set msbuild_path "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe"`
* Dans le cas contraire, installez les outils de build Visual Studio autonomes à l’aide de `npm install --global windows-build-tools` dans une interface de commande *Exécuter en tant qu’administrateur* avec élévation de privilèges.

Pour plus d’informations sur node-gyp, consultez le [dépôt node-gyp sur GitHub](https://github.com/nodejs/node-gyp).

## <a name="create-a-smart-contract"></a>Créer un contrat intelligent

Azure Blockchain Development Kit pour Ethereum utilise des modèles de projet et des outils Truffle pour aider à structurer, générer et déployer des contrats. Avant de commencer, respectez les prérequis [Démarrage rapide : Utilisez Visual Studio Code pour vous connecter à un réseau de consortium Azure Blockchain Service](connect-vscode.md). Le guide de démarrage rapide vous aide à installer et à configurer pas à pas le kit de développement Azure Blockchain pour Ethereum.

1. À partir de la palette de commandes VS Code, choisissez **Blockchain : Nouveau projet Solidity**.
1. Choisissez **Créer un projet de base**.
1. Créez un dossier nommé `HelloBlockchain` et **Sélectionnez un nouveau chemin de projet**.

Azure Blockchain Development Kit crée et initialise un nouveau projet Solidity pour vous. Le projet de base comprend un exemple de contrat intelligent **HelloBlockchain** et tous les fichiers nécessaires à la création et au déploiement sur votre membre de consortium dans Azure Blockchain Service. La création du projet peut prendre plusieurs minutes. Vous pouvez superviser la progression dans le panneau Terminal de VS Code en sélectionnant la sortie pour Azure Blockchain.

La structure du projet ressemble à l’exemple suivant :

   ![Projet Solidity](./media/send-transaction/solidity-project.png)

## <a name="build-a-smart-contract"></a>Générer un contrat intelligent

Les contrats intelligents se trouvent dans le répertoire **contracts** du projet. Vous compilez les contrats intelligents avant de les déployer sur un blockchain. Utilisez la commande **Générer les contrats** pour compiler tous les contrats intelligents dans votre projet.

1. Dans la barre latérale de l’Explorateur de VS Code, développez le dossier **contracts** dans votre projet.
1. Cliquez avec le bouton droit sur **HelloBlockchain.sol**, puis choisissez **Générer les contrats** dans le menu.

    ![Choix de l’option permettant de générer les contrats ](./media/send-transaction/build-contracts.png)

Azure Blockchain Development Kit utilise Truffle pour compiler les contrats intelligents.

![Sortie du compilateur Truffle](./media/send-transaction/compile-output.png)

## <a name="deploy-a-smart-contract"></a>Déployer un contrat intelligent

Truffle utilise des scripts de migration pour déployer vos contrats sur un réseau Ethereum. Les migrations sont des fichiers JavaScript situés dans le répertoire **migrations** du projet.

1. Pour déployer votre contrat intelligent, cliquez avec le bouton droit sur **HelloBlockchain.sol** et choisissez **Déployer les contrats** dans le menu.
1. Choisissez votre réseau de consortium Azure Blockchain dans la palette de commandes. Le réseau blockchain de consortium a été ajouté au fichier de configuration Truffle du projet lorsque vous avez créé le projet.
1. Choisissez **Générer le mnémonique**. Choisissez un nom de fichier et enregistrez le fichier mnémonique dans le dossier du projet. Par exemple : `myblockchainmember.env`. Le fichier mnémonique est utilisé pour générer une clé privée Ethereum pour votre membre blockchain.

Azure Blockchain Development Kit utilise Truffle pour exécuter le script de migration et déployer les contrats dans le blockchain.

![Contrat correctement déployé](./media/send-transaction/deploy-contract.png)

## <a name="call-a-contract-function"></a>Appeler une fonction de contrat

La fonction **SendRequest** du contrat **HelloBlockchain** modifie la variable d’état **RequestMessage**. La modification de l’état d’un réseau blockchain s’effectue par le biais d’une transaction. Vous pouvez utiliser la page d’interaction des contrats intelligents du kit de développement Azure Blockchain pour appeler la fonction **SendRequest** via une transaction.

1. Pour interagir avec votre contrat intelligent, cliquez avec le bouton droit sur **HelloBlockchain.sol** et choisissez **Show Smart Contract Interaction Page** (Afficher la page d’interaction du contrat intelligent) dans le menu.

    ![Choix de l’option permettant d’afficher la page d’interaction du contrat intelligent](./media/send-transaction/contract-interaction.png)

1. La page d’interaction vous permet de choisir une version de contrat déployée, d’appeler des fonctions, de voir l’état actuel et de voir les métadonnées.

    ![Exemple de page d’interaction de contrat intelligent](./media/send-transaction/interaction-page.png)

1. Pour appeler une fonction de contrat intelligent, sélectionnez l’action de contrat et passez vos arguments. Choisissez l’action de contrat **SendRequest** et entrez **Hello, Blockchain!** pour le paramètre **requestMessage**. Sélectionnez **Execute** (Exécuter) pour appeler la fonction **SendRequest** par le biais d’une transaction.

    ![Exécuter l’action SendRequest](./media/send-transaction/sendrequest-action.png)

Une fois la transaction traitée, la section d’interaction reflète les changements d’état.

![Changements de l’état du contrat](./media/send-transaction/contract-state.png)

La fonction SendRequest définit les champs **RequestMessage** et **State**. L’état actuel de **RequestMessage** correspond à l’argument que vous avez passé (**Hello, Blockchain**). La valeur du champ **State** reste **Request**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez supprimer les ressources en supprimant le groupe de ressources `myResourceGroup` que vous avez créé à l’étape du démarrage rapide *Créer un membre blockchain* effectuée en tant que prérequis.

Pour supprimer le groupe de ressources :

1. Dans le portail Azure, accédez à **Groupe de ressources** dans le volet de navigation de gauche et sélectionnez le groupe de ressources que vous souhaitez supprimer.
1. Sélectionnez **Supprimer le groupe de ressources**. Validez la suppression en entrant le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un exemple de projet Solidity à l’aide d’Azure Blockchain Development Kit. Vous avez créé et déployé un contrat intelligent, puis appelé une fonction au moyen d’une transaction sur un réseau blockchain de consortium hébergé sur Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Développement d’applications de blockchain à l’aide du service Azure Blockchain](develop.md)
