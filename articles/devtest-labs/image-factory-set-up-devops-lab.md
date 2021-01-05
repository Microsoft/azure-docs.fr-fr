---
title: Exécuter une fabrique d’images dans Azure DevTest Labs à partir d’Azure DevOps
description: Cet article décrit toutes les tâches de préparation nécessaires pour exécuter la fabrique d’images à partir d’Azure DevOps (anciennement Visual Studio Team Services).
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: fa7050bae1ff8681e04b6ab38220be9eaf38a64a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476136"
---
# <a name="run-an-image-factory-from-azure-devops"></a>Exécuter une fabrique d’images depuis Azure DevOps
Cet article décrit toutes les tâches de préparation nécessaires pour exécuter la fabrique d’images à partir d’Azure DevOps (anciennement Visual Studio Team Services).

> [!NOTE]
> Vous pouvez utiliser le moteur d’orchestration de votre choix. Azure DevOps n’est pas obligatoire. La fabrique d’images s’exécute avec des scripts Azure PowerShell. Vous pouvez donc l’exécuter manuellement à l’aide du Planificateur de tâches Windows ou d’autres systèmes CI/CD, par exemple.

## <a name="create-a-lab-for-the-image-factory"></a>Créer un labo pour la fabrique d’images
La première étape de configuration de la fabrique d’images consiste à créer un labo dans Azure DevTest Labs. Il s’agit du labo de la fabrique d’images où nous créons les machines virtuelles et enregistrons les images personnalisées. Ce labo fait partie du processus global de la fabrique d’images. Quand vous créez un labo, prenez note de son nom, car vous en aurez besoin plus tard.

## <a name="scripts-and-templates"></a>Scripts et modèles
L’étape suivante dans la mise en place de la fabrique d’images pour votre équipe est de comprendre quels éléments sont disponibles. Les scripts et les modèles de la fabrique d’images sont disponibles publiquement dans le [dépôt DevTest Labs sur GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Scripts/ImageFactory). Voici une brève présentation des éléments :

- Image Factory. C’est le dossier racine.
    - Configuration. Entrées de la fabrique d’images.
        - GoldenImages. Ce dossier contient les fichiers JSON représentant les définitions des images personnalisées.
        - Labs.json. Fichier dans lequel les équipes s’inscrivent pour recevoir des images personnalisées spécifiques.
- Scripts. Moteur de la fabrique d’images.

Les articles dans cette section fournissent plus d’informations sur ces scripts et ces modèles.

## <a name="create-an-azure-devops-team-project"></a>Créer un projet d’équipe Azure DevOps
Avec Azure DevOps, vous pouvez stocker le code source et exécuter Azure PowerShell à un seul et même endroit. Vous pouvez planifier des exécutions périodiques pour maintenir les images à jour. Vous disposez aussi de fonctionnalités de journalisation des résultats utiles pour diagnostiquer les problèmes.  Vous n’êtes toutefois pas obligé d’utiliser Azure DevOps. Vous pouvez choisir n’importe quel moteur/dispositif capable de se connecter à Azure et d’exécuter Azure PowerShell.

Si vous avez déjà un projet ou un compte DevOps que vous souhaitez utiliser à la place, ignorez cette étape.

Pour commencer, créez un compte gratuit dans Azure DevOps. Accédez à https://www.visualstudio.com/ et sélectionnez **Démarrez gratuitement** sous **Azure DevOps** (anciennement VSTS). Choisissez un nom de compte unique et choisissez l’option de gestion du code avec Git. Une fois que le compte est créé, enregistrez l’URL dans votre projet d’équipe. Voici un exemple d’URL : `https://<accountname>.visualstudio.com/MyFirstProject`.

## <a name="check-in-the-image-factory-to-git"></a>Enregistrer la fabrique d’images dans Git
Tous les éléments PowerShell, de modèle et de configuration pour la fabrique d’images se trouvent dans le [dépôt public DevTest Labs sur GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Scripts/ImageFactory). Le moyen le plus rapide de transférer le code dans votre nouveau projet d’équipe consiste à importer un dépôt. Cette action tire (pull) tout le contenu du dépôt DevTest Labs (y compris la documentation supplémentaire et les exemples).

1. Accédez au projet Azure DevOps que vous avez créé à l’étape précédente (l’URL se présente ainsi : **https:\//\<accountname>.visualstudio.com/MyFirstProjet**).
2. Sélectionnez **Importer un dépôt**.
3. Entrez l’**URL clone** du dépôt DevTest Labs : `https://github.com/Azure/azure-devtestlab`.
4. Sélectionnez **Importer**.

    ![Importer un dépôt Git](./media/set-up-devops-lab/import-git-repo.png)

Si vous préférez enregistrer uniquement les éléments dont vous avez besoin (les fichiers de la fabrique d’images), suivez les étapes décrites [ici](https://www.visualstudio.com/en-us/docs/git/share-your-code-in-git-vs) pour cloner le dépôt Git et envoyer (push) seulement les fichiers du répertoire **scripts/ImageFactory**.

## <a name="create-a-build-and-connect-to-azure"></a>Créer une build et se connecter à Azure
À ce stade, vos fichiers sources sont stockés dans un dépôt Git dans Azure DevOps. Maintenant, vous devez configurer un pipeline pour exécuter Azure PowerShell. Beaucoup d’options sont possibles pour effectuer ces étapes. Dans cet article, vous utilisez une définition de build pour rester simple, mais vous pourriez aussi utiliser une build DevOps, une mise en production DevOps (sur un seul ou plusieurs environnements), un autre moteur d’exécution comme le Planificateur de tâches Windows ou n’importe quel autre dispositif pouvant exécuter Azure PowerShell.

> [!NOTE]
> Gardez à l’esprit que l’exécution de certains fichiers PowerShell prend beaucoup de temps quand il y a plus de 10 images personnalisées à créer. Les agents de build/mise en production DevOps hébergés gratuits ont un délai d’attente de 30 minutes. Ils ne sont donc pas appropriés si vous devez créer beaucoup d’images. Ce problème de délai d’attente concerne tous les dispositifs. Vérifiez au préalable que vous pouvez augmenter les délais d’attente par défaut pour les scripts Azure PowerShell durables. Dans le cas d’Azure DevOps, vous pouvez utiliser soit des agents hébergés payants, soit votre propre agent de build.

1. Pour commencer, sélectionnez **Configurer la build** dans la page d’accueil de votre projet DevOps :

    ![Bouton Configurer la build](./media/set-up-devops-lab/setup-build-button.png)
2. Spécifiez un **nom** pour la build (par exemple : Créer et distribuer des images dans DevTest Labs).
3. Sélectionnez une définition de build **vide**, puis sélectionnez **Appliquer** pour créer votre build.
4. Ici, choisissez **Hébergé** pour l’agent de build.
5. **Enregistrez** la définition de build.

    ![Définition de build](./media/set-up-devops-lab/build-definition.png)

## <a name="configure-the-build-variables"></a>Configurer des variables de build
Pour simplifier les paramètres de ligne de commande, encapsulez les valeurs clés de la fabrique d’images dans un ensemble de variables de build. Sélectionnez l’onglet **Variables**. Une liste de variables par défaut s’affiche. Voici la liste des variables à entrer dans Azure DevOps :


| Nom de la variable | Valeur | Notes |
| ------------- | ----- | ----- |
| ConfigurationLocation | /Scripts/ImageFactory/Configuration | Chemin complet du dossier **Configuration** dans le dépôt. Si vous avez importé le dépôt entier précédemment, la valeur à gauche est correcte. Sinon, mettez-la à jour pour qu’elle pointe vers l’emplacement du dossier Configuration. |
| DevTestLabName | MyImageFactory | Nom du labo dans Azure DevTest Labs utilisé comme fabrique d’images. Si vous n’en avez pas, créez-en un. Le labo doit être dans le même abonnement que celui auquel a accès le point de terminaison du service. |
| ImageRetention | 1 | Nombre d’images de chaque type que vous souhaitez enregistrer. Définissez la valeur par défaut à 1. |
| MachinePassword | ******* | Mot de passe du compte administrateur intégré pour les machines virtuelles. Il s’agit d’un compte temporaire, qui doit être sécurisé. Sélectionnez la petite icône de verrou sur la droite pour vérifier que la chaîne est sécurisée. |
| MachineUserName | ImageFactoryUser | Nom d’utilisateur du compte administrateur intégré pour les machines virtuelles. Il s’agit d’un compte temporaire. |
| StandardTimeoutMinutes | 30 | Délai d’attente à observer pour les opérations Azure standard. |
| SubscriptionId |  0000000000-0000-0000-0000-0000000000000 | ID de l’abonnement dont fait partie le labo et auquel le point de terminaison du service a accès. |
| VMSize | Standard_A3 | Taille de la machine virtuelle à utiliser à l’étape **Créer**. Les machines virtuelles créées sont temporaires. La taille doit être la taille qui est [autorisée pour le lab](devtest-lab-set-lab-policy.md). Vérifiez que le [quota de cœurs d’abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md) est suffisant.

![Variables de build](./media/set-up-devops-lab/configure-build-variables.png)

## <a name="connect-to-azure"></a>Connexion à Azure
L’étape suivante consiste à configurer le principal de service. Il s’agit d’une identité dans Azure Active Directory qui permet à l’agent de build DevOps de faire des opérations dans Azure pour le compte de l’utilisateur. Pour configurer le principal de service, commencez par ajouter la première étape de build Azure PowerShell.

1. Sélectionnez **Ajouter une tâche**.
2. Faites une recherche sur **Azure PowerShell**.
3. Quand vous l’avez trouvée, sélectionnez **Ajouter** pour ajouter la tâche à la build. La tâche ajoutée apparaît sur le côté gauche.

![Configurer l’étape PowerShell](./media/set-up-devops-lab/set-up-powershell-step.png)

Le moyen le plus rapide de configurer un principal de service est de laisser Azure DevOps le faire pour nous.

1. Sélectionnez la **tâche** que vous venez d’ajouter.
2. Comme **Type de connexion Azure**, choisissez **Azure Resource Manager**.
3. Sélectionnez le lien **Gérer** pour configurer le principal de service.

Pour plus d’informations, consultez ce [billet de blog](https://devblogs.microsoft.com/devops/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/). Quand vous sélectionnez le lien **Gérer**, vous arrivez directement à l’endroit dans DevOps (deuxième capture d’écran dans le billet de blog) où vous pouvez configurer la connexion à Azure. Choisissez **Point de terminaison de service Azure Resource Manager** quand vous le configurez.

## <a name="complete-the-build-task"></a>Terminer la tâche de build
Si vous sélectionnez la tâche de build, vous voyez tous les détails à renseigner dans le volet droit.

1. Tout d’abord, nommez la tâche de build : **Créer des machines virtuelles**.
2. Sélectionnez le **principal de service** que vous avez créé en choisissant **Azure Resource Manager**
3. Sélectionnez le **point de terminaison de service**.
4. Dans **Chemin du script**, sélectionnez **... (points de suspension)**  sur la droite.
5. Accédez au script **MakeGoldenImageVMs.ps1**.
6. Les paramètres de script doivent ressembler à ceci : `-ConfigurationLocation $(System.DefaultWorkingDirectory)$(ConfigurationLocation) -DevTestLabName $(DevTestLabName) -vmSize $(VMSize) -machineUserName $(MachineUserName) -machinePassword (ConvertTo-SecureString -string '$(MachinePassword)' -AsPlainText -Force) -StandardTimeoutMinutes $(StandardTimeoutMinutes)`

    ![Terminer la définition de build](./media/set-up-devops-lab/complete-build-definition.png)


## <a name="queue-the-build"></a>Mettre la build en file d’attente
Vérifions maintenant que vous avez tout configuré correctement en mettant une nouvelle build en file d’attente. Durant l’exécution de la build, basculez vers le [portail Azure](https://portal.azure.com) et sélectionnez **Toutes les machines virtuelles** dans votre labo de fabrique d’images pour vérifier que tout fonctionne correctement. Vous devez normalement voir trois machines virtuelles créées dans le labo.

![Machines virtuelles dans le labo](./media/set-up-devops-lab/vms-in-lab.png)

## <a name="next-steps"></a>Étapes suivantes
La première étape de configuration de la fabrique d’images dans Azure DevTest Labs est terminée. Dans l’article suivant de la série, vous créez et enregistrez des images personnalisées généralisées à partir des machines virtuelles de ce labo. Ensuite, vous les distribuez à vos autres labos. Consultez l’article suivant de la série : [Enregistrer des images personnalisées et les distribuer à plusieurs labs](image-factory-save-distribute-custom-images.md).
