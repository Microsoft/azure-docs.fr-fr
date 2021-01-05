---
title: Configurer une instance et l’authentification (procédure scriptée)
titleSuffix: Azure Digital Twins
description: Découvrez comment configurer une instance du service Azure Digital Twins, en exécutant un script de déploiement automatisé
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5806ea094abd3431cd7e22064c6acd8ad150726a
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92495020"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-scripted"></a>Configurer une instance Azure Digital Twins et l’authentification (procédure scriptée)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

Cet article explique comment **configurer une nouvelle instance Azure Digital Twins** , notamment la création de l’instance et la configuration de l’authentification. À l’issue de cet article, vous aurez une instance Azure Digital Twins prête pour la programmation.

Cette version de cet article effectue ces étapes en exécutant un [exemple de **script de déploiement automatisé**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) qui simplifie le processus. 
* Pour afficher les étapes manuelles d’interface CLI que le script exécute en arrière-plan, consultez la version CLI de cet article : [*Guide pratique : Configurer une instance et l’authentification (CLI)*](how-to-set-up-instance-cli.md).
* Pour afficher les étapes manuelles utilisant le portail Azure, consultez la version de cet article relative au portail : [*Guide pratique : Configurer une instance et l’authentification (portail)*](how-to-set-up-instance-portal.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

## <a name="prerequisites-download-the-script"></a>Configuration requise : Télécharger le script

L’exemple de script est écrit dans PowerShell. Il fait partie des [**exemples Azure Digital Twins de bout en bout**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/), que vous pouvez télécharger sur votre ordinateur en suivant ce lien et en sélectionnant le bouton *Télécharger le fichier ZIP* sous le titre.

L’exemple de projet sera téléchargé sur votre ordinateur sous le nom _**Azure_Digital_Twins_end_to_end_samples.zip**_ . Accédez au dossier sur votre ordinateur et décompressez-le pour extraire les fichiers.

Dans le dossier décompressé, le script de déploiement se trouve à l’emplacement suivant : _Azure_Digital_Twins_end_to_end_samples > scripts > **deploy.ps1**_ .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="run-the-deployment-script"></a>Exécuter le script de déploiement

Cet article utilise un exemple de code Azure Digital Twins pour déployer de façon semi-automatique une instance Azure Digital Twins et l’authentification requise. Il peut également être utilisé comme point de départ pour écrire vos propres interactions scriptées.

Voici les étapes à suivre pour exécuter le script de déploiement dans Cloud Shell.
1. Accédez à une fenêtre [Azure Cloud Shell](https://shell.azure.com/) dans votre navigateur. Connectez-vous à l’aide de cette commande :
    ```azurecli-interactive
    az login
    ```
    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion. Sinon, ouvrez une page de navigateur à l’adresse *https://aka.ms/devicelogin* et entrez le code d’autorisation affiché dans votre terminal.
 
2. Dans la barre d’icônes Cloud Shell, vérifiez que Cloud Shell est configuré pour exécuter la version PowerShell.

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-powershell.png" alt-text="Fenêtre Cloud Shell présentant la sélection de la version PowerShell":::

1. Sélectionnez l’icône « Charger/Télécharger des fichiers » et choisissez « Charger ».

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Fenêtre Cloud Shell présentant la sélection de la version PowerShell":::

    Accédez au fichier _**deploy.ps1**_ sur votre ordinateur (à l’emplacement suivant : _Azure_Digital_Twins_end_to_end_samples > scripts > **deploy.ps1**_ ) et appuyez sur « Ouvrir ». Cette opération charge le fichier dans Cloud Shell pour que vous puissiez l’exécuter dans la fenêtre Cloud Shell.

4. Exécutez le script en envoyant la commande `./deploy.ps1` dans la fenêtre Cloud Shell. Vous pouvez copier la commande ci-dessous. (Pour rappel, vous pouvez utiliser **Ctrl+Maj+V** sur Windows et Linux ou **Cmd+Maj+V** sur macOS pour coller dans Cloud Shell. Vous avez également la possibilité de vous servir du menu contextuel.)

    ```azurecli-interactive
    ./deploy.ps1
    ```

    Le script créera une instance Azure Digital Twins et attribuera à votre utilisateur Azure le rôle *Propriétaire de données Azure Digital Twins* sur l’instance.

    Lorsque le script exécute les étapes de configuration automatisée, vous êtes invité à transmettre les valeurs suivantes :
    * Pour l’instance : l’ *ID d’abonnement* de votre abonnement Azure à utiliser
    * Pour l’instance : un *emplacement* où vous souhaitez déployer l’instance. Pour voir quelles régions prennent en charge Azure Digital Twins, visitez [*Disponibilité des produits Azure par région*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
    * Pour l’instance : un nom de *groupe de ressources* . Vous pouvez utiliser un groupe de ressources existant ou entrer un nouveau nom pour en créer un.
    * Pour l’instance : un *nom* de votre instance Azure Digital Twins. Le nom de la nouvelle instance doit être unique dans la région pour votre abonnement (ce qui signifie que si votre abonnement a une autre instance Azure Digital Twins dans cette région, qui utilise déjà le nom que vous choisissez, vous devrez choisir un autre nom).

Voici un extrait du journal de sortie du script :

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png" alt-text="Fenêtre Cloud Shell présentant la sélection de la version PowerShell" lightbox="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png":::

Si le script s’exécute correctement, l’affichage final indique `Deployment completed successfully`. Sinon, traitez le message d’erreur et réexécutez le script. Cela permet de contourner les étapes que vous avez déjà effectuées et de demander une nouvelle entrée au stade où vous vous étiez arrêté.

> [!NOTE]
> Le script attribue actuellement le rôle de gestion requis dans Azure Digital Twins ( *Propriétaire de données Azure Digital Twins* ) au même utilisateur qui exécute ce script à partir de Cloud Shell. Si vous devez attribuer ce rôle à une autre personne chargée de gérer cette instance, vous pouvez le faire via le portail Azure ([instructions](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) ou l’interface CLI ([instructions](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

>[!NOTE]
>Il existe actuellement un **problème connu** avec l’installation par script, qui a pour effet que certains utilisateurs (en particulier, des utilisateurs de [comptes Microsoft (MSA)](https://account.microsoft.com/account) personnel) peuvent découvrir que l’ **attribution de rôle au _Propriétaire de données Azure Digital Twins_ n’a pas été créée** .
>
>Vous pourrez vérifier l’attribution de rôle à l’aide de la section [*Vérifier l’attribution du rôle utilisateur*](#verify-user-role-assignment) plus loin dans cet article et, si nécessaire, configurer manuellement l’attribution de rôle à l’aide du [portail Azure](how-to-set-up-instance-portal.md#set-up-user-access-permissions) ou de [CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).
>
>Pour plus de détails sur ce problème, consultez [*Résolution des problèmes : Problèmes connus dans Azure Digital Twins*](troubleshoot-known-issues.md#missing-role-assignment-after-scripted-setup).

## <a name="verify-success-and-collect-important-values"></a>Vérifier la réussite de l’exécution et collecter les valeurs importantes

Pour vérifier la création de vos ressources et autorisations configurées par le script, vous pouvez les consulter dans le [portail Azure](https://portal.azure.com) en suivant les instructions ci-dessous. Si vous ne parvenez pas à vérifier la réussite d’une étape, recommencez l’opération. Pour appliquer les étapes individuellement, suivez les instructions du [Portail Azure](how-to-set-up-instance-portal.md) ou de [l’interface CLI](how-to-set-up-instance-cli.md) .

Cette section vous montre également comment rechercher des valeurs importantes parmi les ressources configurées par le script dont vous pourriez avoir besoin lorsque vous continuez à travailler avec votre instance Azure Digital Twins. Vous devez enregistrer ces valeurs dans un emplacement sûr ou revenir à cette section pour les retrouver plus tard lorsque vous en aurez besoin. Si d’autres utilisateurs doivent programmer pour l’instance, vous devez également partager ces valeurs avec eux.

### <a name="verify-instance"></a>Vérifier l’instance

Pour vérifier que votre instance a été créée, accédez à la [page Azure Digital Twins](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) dans le portail Azure. Vous pouvez accéder à cette page vous-même en recherchant *Azure Digital Twins* dans la barre de recherche du portail.

Cette page liste toutes vos instances Azure Digital Twins. Recherchez le nom de votre instance nouvellement créée dans la liste.

Si la vérification a échoué, vous pouvez réessayer de créer une instance à l’aide du [Portail](how-to-set-up-instance-portal.md#create-the-azure-digital-twins-instance) ou de [l’interface CLI](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance).

### <a name="collect-instance-values"></a>Collecter les valeurs d’instance

Sélectionnez le nom de votre instance dans la [page Azure Digital Twins](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) pour ouvrir la page *Vue d’ensemble* de l’instance. Notez son *nom* , son *groupe de ressources* et son *nom d’hôte* . Vous en aurez peut-être besoin plus tard pour identifier votre instance et vous y connecter.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Fenêtre Cloud Shell présentant la sélection de la version PowerShell":::

### <a name="verify-user-role-assignment"></a>Vérifier l’attribution du rôle utilisateur

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

> [!NOTE]
> Rappelez-vous que le script affecte actuellement ce rôle nécessaire au même utilisateur qui exécute le script à partir de Cloud Shell. Si vous devez attribuer ce rôle à une autre personne chargée de gérer cette instance, vous pouvez le faire via le portail Azure ([instructions](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) ou l’interface CLI ([instructions](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

Si la vérification a échoué, vous pouvez également refaire votre propre attribution de rôle à l’aide du [Portail](how-to-set-up-instance-portal.md#set-up-user-access-permissions) ou de [l’interface CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).

## <a name="next-steps"></a>Étapes suivantes

Testez les appels d’API REST individuels sur votre instance à l’aide des commandes CLI d’Azure Digital Twins : 
* [az dt reference](/cli/azure/ext/azure-iot/dt?preserve-view=true&view=azure-cli-latest)
* [*Guide pratique : Utiliser l’interface CLI d’Azure Digital Twins*](how-to-use-cli.md)

Vous pouvez également découvrir comment connecter une application cliente à votre instance avec un code d’authentification :
* [*Guide pratique : Écrire le code d’authentification de l’application*](how-to-authenticate-client.md)