---
title: Extension Azure Policy pour Visual Studio Code
description: Découvrez comment utiliser l’extension Azure Policy pour Visual Studio Code afin de rechercher des alias Azure Resource Manager.
ms.date: 10/20/2020
ms.topic: how-to
ms.openlocfilehash: 8c7357d70323fa74ec77cf43bd11f149a2f99154
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906299"
---
# <a name="use-azure-policy-extension-for-visual-studio-code"></a>Utiliser l’extension Azure Policy pour Visual Studio Code

> S’applique à l’extension Azure Policy, versions **0.1.0** et ultérieures

Découvrez comment utiliser l’extension Azure Policy pour Visual Studio Code afin de rechercher des [alias](../concepts/definition-structure.md#aliases), d’examiner des ressources et stratégies, d’exporter des objets et d’évaluer des définitions de stratégie. Tout d’abord, nous décrirons comment installer l’extension Azure Policy dans Visual Studio Code. Ensuite, nous étudierons comment rechercher des alias.

L’extension Azure Policy pour Visual Studio Code peut être installée sur toutes les plateformes prises en charge par Visual Studio Code. Windows, Linux et macOS.

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour effectuer les étapes décrites dans cet article :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- [Visual Studio Code](https://code.visualstudio.com).

## <a name="install-and-configure-the-azure-policy-extension"></a>Installer et configurer l’extension Azure Policy

Une fois tous les prérequis remplis, vous pouvez installer l’extension Azure Policy pour Visual Studio Code en suivant les étapes ci-dessous :

1. Ouvrez Visual Studio Code.
1. À partir de la barre de menus, accédez à **Afficher** > **Extensions**.
1. Dans la zone de recherche, entrez **Azure Policy**.
1. Sélectionnez **Azure Policy** dans les résultats de la recherche, puis sélectionnez **Installer**.
1. Sélectionnez **Recharger** si nécessaire.

Dans le cas d’un utilisateur d’un cloud national, suivez d’abord ces étapes pour configurer l’environnement Azure :

1. Sélectionnez **File\Preferences\Settings**.
1. Recherchez la chaîne suivante : _Azure : Cloud_
1. Sélectionnez le cloud national dans la liste :

   :::image type="content" source="../media/extension-for-vscode/set-default-azure-cloud-sign-in.png" alt-text="Capture d’écran de la sélection de la connexion au cloud Azure national pour Visual Studio Code." border="false":::

## <a name="using-the-policy-extension"></a>Utilisation de l’extension de stratégie

> [!NOTE]
> Les modifications apportées localement aux stratégies qui s’affichent dans l’extension Azure Policy pour Visual Studio Code ne sont pas synchronisées avec Azure.

### <a name="connect-to-an-azure-account"></a>Se connecter à un compte Azure

Pour pouvoir évaluer des ressources et rechercher des alias, vous devez vous connecter à votre compte Azure. Suivez ces étapes pour vous connecter à Azure à partir de Visual Studio Code :

1. Connectez-vous à Azure à partir de l’extension Azure Policy ou de la palette de commandes.

   - Extension Azure Policy

     À partir de l’extension Azure Policy, sélectionnez **Se connecter à Azure**.

     :::image type="content" source="../media/extension-for-vscode/azure-cloud-sign-in-policy-extension.png" alt-text="Capture d’écran de Visual Studio Code et de l’icône de l’extension Azure Policy." border="false":::

   - Palette de commandes

     À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Azure: Se connecter**.

     :::image type="content" source="../media/extension-for-vscode/azure-cloud-sign-in-command-palette.png" alt-text="Capture d’écran des options de connexion au cloud Azure pour Visual Studio Code à partir de la palette de commandes." border="false":::

1. Suivez les instructions de connexion pour vous connecter à Azure. Une fois que vous êtes connecté, le nom de votre compte Azure s’affiche dans la barre d’état, en bas de la fenêtre Visual Studio Code.

### <a name="select-subscriptions"></a>Sélectionner des abonnements

Lors de la première connexion, seules les stratégies et les ressources d’abonnement par défaut sont chargées par l’extension Azure Policy. Pour ajouter des abonnements aux ressources et aux stratégies qui s’affichent ou en supprimer, suivez les étapes ci-dessous :

1. Démarrez la commande d’abonnement à partir de la palette de commandes ou du pied de page de la fenêtre.

   - Palette de commandes :

     À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Azure : Sélectionner des abonnements**.

   - Pied de page de la fenêtre

     Dans le pied de page de la fenêtre en bas de l’écran, sélectionnez le segment correspondant à **Azure : \<your account\>** .

1. Utilisez la zone de filtre pour trouver rapidement des abonnements par leur nom. Ensuite, cochez ou décochez les abonnements pour définir ceux qui seront affichés par l’extension Azure Policy. Une fois que vous avez terminé d’ajouter ou de supprimer des abonnements, sélectionnez **OK**.

### <a name="search-for-and-view-resources"></a>Rechercher et afficher des ressources

L’extension Azure Policy liste les ressources des abonnements sélectionnés par fournisseur de ressources et par groupe de ressources dans le volet **Ressources**. L’arborescence comprend les regroupements de ressources suivants au sein de l’abonnement sélectionné ou au niveau de l’abonnement :

- **Fournisseurs de ressources**
  - Chacun des fournisseurs de ressources inscrits avec des ressources et les ressources enfants associées qui possèdent des alias de stratégie
- **Groupes de ressources**
  - Toutes les ressources suivant le groupe de ressources dans lequel elles se trouvent

Par défaut, l’extension filtre la partie « Fournisseur de ressources » sur les ressources existantes et les ressources qui possèdent des alias de stratégie. Modifiez ce comportement dans **Paramètres** > **Extensions** > **Azure Policy** pour voir tous les fournisseurs de ressources sans filtrage.

Les clients disposant de centaines ou de milliers de ressources dans un même abonnement préféreront probablement un moyen de trouver leurs ressources avec possibilité de recherche. L’extension Azure Policy permet de rechercher une ressource spécifique en suivant les étapes suivantes :

1. Lancez l’interface de recherche à partir de l’extension Azure Policy ou de la palette de commandes.

   - Extension Azure Policy

     Dans l’extension Azure Policy, placez le curseur sur le panneau **Ressources** et sélectionnez les points de suspension, puis **Rechercher des ressources**.

   - Palette de commandes :

     Dans la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Ressources : Rechercher des ressources**.

1. Si plusieurs abonnements sont sélectionnés pour l’affichage, utilisez le filtre pour sélectionner celui dans lequel la recherche devra être effectuée.

1. Utilisez le filtre pour sélectionner le groupe de ressources dans lequel la recherche devra être effectuée, au sein de l’abonnement choisi.

1. Utilisez le filtre pour sélectionner la ressource à afficher. Le filtre fonctionne à la fois pour le nom et le type de ressource.

### <a name="discover-aliases-for-resource-properties"></a>Découvrir les alias des propriétés de ressources

Lorsqu’une ressource est sélectionnée, que ce soit par le biais de l’interface de recherche ou dans l’arborescence, l’extension Azure Policy ouvre le fichier JSON représentant cette ressource et toutes ses valeurs de propriété Azure Resource Manager.

Lorsqu’une ressource est ouverte, pointez sur le nom ou la valeur de la propriété Resource Manager pour afficher l’alias Azure Policy s’il en existe un. Dans cet exemple, la ressource a pour type `Microsoft.Compute/virtualMachines` et le curseur est placé sur la propriété **properties.storageProfile.imageReference.offer**, ce qui affiche les alias correspondants.

:::image type="content" source="../media/extension-for-vscode/extension-hover-shows-property-alias.png" alt-text="Capture d’écran de l’extension Azure Policy pour Visual Studio Code permettant de survoler une propriété afin d’afficher les noms d’alias." border="false":::

> [!NOTE]
> L’extension VS Code prend uniquement en charge l’évaluation des propriétés du mode Resource Manager. Pour plus d’informations sur les modes, consultez les [définitions de mode](../concepts/definition-structure.md#mode).

### <a name="search-for-and-view-policies-and-assignments"></a>Rechercher et afficher des stratégies et des affectations

L’extension Azure Policy liste les types et les affectations de stratégie sous la forme d’une arborescence pour les abonnements à afficher dans le volet **Stratégies**. Les clients disposant de centaines ou de milliers de stratégies ou d’affectations dans un même abonnement préféreront probablement un moyen de trouver leurs stratégies ou leurs affectations avec possibilité de recherche. L’extension Azure Policy permet de rechercher une stratégie ou une affectation spécifique en suivant les étapes suivantes :

1. Lancez l’interface de recherche à partir de l’extension Azure Policy ou de la palette de commandes.

   - Extension Azure Policy

     Dans l’extension Azure Policy, placez le curseur sur le panneau **Stratégies** et sélectionnez les points de suspension, puis **Rechercher des stratégies**.

   - Palette de commandes :

     Dans la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Stratégies : Rechercher des stratégies**.

1. Si plusieurs abonnements sont sélectionnés pour l’affichage, utilisez le filtre pour sélectionner celui dans lequel la recherche devra être effectuée.

1. Utilisez le filtre pour sélectionner l’affectation ou le type de stratégie dans lequel la recherche devra être effectuée, au sein de l’abonnement choisi.

1. Utilisez le filtre pour sélectionner la stratégie ou l’affectation à afficher. Le filtre fonctionne sur _displayName_ pour la définition ou l’affectation de stratégie.

Lorsqu’une stratégie ou une affectation est sélectionnée, que ce soit par le biais de l’interface de recherche ou dans l’arborescence, l’extension Azure Policy ouvre le fichier JSON représentant la stratégie ou l’affectation et toutes ses valeurs de propriété Resource Manager. L’extension peut valider le schéma JSON Azure Policy ouvert.

### <a name="export-objects"></a>Exporter des objets

Vous pouvez exporter les objets de vos abonnements vers un fichier JSON local. Dans le volet **Ressources** ou **Stratégies**, pointez sur un objet exportable ou sélectionnez-en un. À la fin de la ligne en surbrillance, sélectionnez l’icône Enregistrer, puis un dossier pour enregistrer ce fichier JSON de ressources.

Vous pouvez exporter les objets suivants localement :

- Volet Ressources
  - Groupes de ressources
  - Ressources individuelles (dans un groupe de ressources ou sous un fournisseur de ressources)
- Volet Stratégies
  - Attributions de stratégies
  - Définitions de stratégie intégrées
  - Définitions de stratégie personnalisées
  - Initiatives

### <a name="on-demand-evaluation-scan"></a>Analyse d’évaluation à la demande

Vous pouvez démarrer une analyse d’évaluation avec l’extension Azure Policy pour Visual Studio Code. Pour démarrer une évaluation, sélectionnez et épinglez chacun des objets suivants : une ressource, une définition de stratégie et une attribution de stratégie.

1. Pour épingler chaque objet, recherchez-le dans le volet **Ressources** ou **Stratégies**, puis sélectionnez l’icône Épingler à un onglet Modifier. L’épinglage d’un objet a pour effet d’ajouter celui-ci au volet **Évaluation** de l’extension.
1. Dans le volet **Évaluation**, sélectionnez un objet de chaque type et utilisez l’icône Sélectionner pour évaluation pour marquer l’objet comme étant inclus dans l’évaluation.
1. En haut du volet **Évaluation**, sélectionnez l’icône Exécuter l’évaluation. Un nouveau volet dans Visual Studio Code s’ouvre avec les détails de l’évaluation résultants au format JSON.

> [!NOTE]
> Pour les définitions de stratégie [AuditIfNotExists](../concepts/effects.md#auditifnotexists) ou [DeployIfNotExists](../concepts/effects.md#deployifnotexists), utilisez l’icône plus (+) dans le volet **Évaluation** pour sélectionner une ressource _liée_ pour la vérification d’existence.

Les résultats de l’évaluation fournissent des informations sur la définition et l’attribution de stratégie, ainsi que sur la propriété **policyEvaluations.evaluationResult**. La sortie doit ressembler à celle-ci :

```json
{
    "policyEvaluations": [
        {
            "policyInfo": {
                ...
            },
            "evaluationResult": "Compliant",
            "effectDetails": {
                "policyEffect": "Audit",
                "existenceScope": "None"
            }
        }
    ]
}
```

> [!NOTE]
> L’extension VS Code prend uniquement en charge l’évaluation des propriétés du mode Resource Manager. Pour plus d’informations sur les modes, consultez les [définitions de mode](../concepts/definition-structure.md#mode).
>
> La fonctionnalité d’évaluation ne fonctionne pas sur les installations macOS et Linux de l’extension.

### <a name="sign-out"></a>Se déconnecter

À partir de la barre de menus, accédez à **Afficher** > **Palette de commandes**, puis entrez **Azure: Se déconnecter**.

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).
- Découvrez comment [créer des stratégies par programmation](programmatically-create.md).
- Découvrez comment [corriger des ressources non conformes](remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
