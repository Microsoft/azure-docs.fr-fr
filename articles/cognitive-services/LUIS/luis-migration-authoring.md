---
title: Effectuer une migration vers une clé de ressource de création Azure
titleSuffix: Azure Cognitive Services
description: Cet article explique comment migrer l’authentification de création Language Understanding (LUIS) d’un compte e-mail vers une ressource Azure.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 12/07/2020
ms.openlocfilehash: 243c9834aa256e26d620c00ac0fa7a262919aabd
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96762653"
---
# <a name="migrate-to-an-azure-resource-authoring-key"></a>Effectuer une migration vers une clé de ressource de création Azure

> [!IMPORTANT]
>  À partir du 3 décembre, les utilisateurs qui utilisent actuellement LUIS doivent effectuer le processus de migration pour continuer à créer des applications LUIS.

Pour la création LUIS (Language Understanding), l’authentification ne se fait plus avec un compte e-mail, mais avec une ressource Azure. Dans cet article, découvrez comment migrer votre compte, si vous ne l’avez pas encore fait.  


## <a name="what-is-migration"></a>Qu’est-ce que la migration ?

La migration est le processus consistant à passer d’un compte e-mail à une ressource Azure pour l’authentification de la création. Une fois la migration effectuée, votre compte est lié à un abonnement Azure et à une ressource de création Azure. *Tous les utilisateurs de LUIS (collaborateurs et propriétaires d’applications) doivent effectuer la migration à un moment donné.*

La migration doit être effectuée à partir du [portail LUIS](https://www.luis.ai). Si vous créez les clés de création à l’aide de l’interface de ligne de commande LUIS, vous devrez effectuer le processus de migration dans le portail LUIS. Après la migration, vous pouvez toujours avoir des coauteurs associés à vos applications, mais ceux-ci sont ajoutés au niveau de la ressource Azure plutôt qu’au niveau de l’application.

> [!Note]
> Avant la migration, les cocréateurs sont appelés _collaborateurs_ au niveau de l’application LUIS. Après la migration, le rôle Azure de _contributeur_ est utilisé pour la même fonctionnalité au niveau de la ressource Azure.

## <a name="notes-before-you-migrate"></a>Remarques avant la migration

* La migration ne peut pas être inversée.
* Si vous vous êtes connecté à plusieurs [portails régionaux LUIS](./luis-reference-regions.md#luis-authoring-regions), vous êtes invité à migrer dans plusieurs régions à la fois.
* Les applications migrent automatiquement avec leur propriétaire.
* Le propriétaire ne peut pas effectuer la migration d’un sous-ensemble d’applications et ce processus n’est pas réversible.
* Après que le propriétaire a opéré la migration, les applications disparaissent du compte du collaborateur.
* Les propriétaires sont invités à envoyer des e-mails aux collaborateurs pour les informer de la migration.
* Si vous êtes un collaborateur associé à une application, celle-ci ne migre pas avec vous. Toutefois, les collaborateurs sont invités à exporter les applications dont ils ont besoin.
* Il n’existe aucun moyen pour un propriétaire de savoir si les collaborateurs ont migré.
* La migration ne déplace ni n’ajoute automatiquement les collaborateurs à la ressource de création Azure. Il incombe au propriétaire de l’application d’effectuer cette étape après la migration. Cette étape nécessite de disposer d’autorisations [sur la ressource de création Azure](./luis-how-to-collaborate.md).
* Une fois affectés à la ressource Azure, les collaborateurs devront effectuer la migration avant de pouvoir accéder aux applications. Autrement, ils n’auront pas accès à la création des applications.
* Il n’est pas possible d’ajouter un utilisateur migré en tant que collaborateur de l’application.


> [!Note]
> Si vous avez besoin de créer un runtime de prédiction, [un processus distinct](luis-how-to-azure-subscription.md#create-resources-in-the-azure-portal) permet de le faire.

## <a name="migration-prerequisites"></a>Prérequis pour la migration

* Vous devez être associé à un abonnement Azure valide. Demandez à votre administrateur de locataire de vous ajouter à l’abonnement, ou [inscrivez-vous pour en obtenir un gratuitement](https://azure.microsoft.com/free/cognitive-services).
* Vous devez créer une ressource de création LUIS Azure à partir du portail LUIS ou du [portail Azure](https://portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne). La création d’une ressource de création à partir du portail LUIS fait partie du processus de migration décrit dans la section suivante.
* Si vous êtes un collaborateur associé à des applications, celles-ci ne migreront pas automatiquement. Vous serez invité à exporter ces applications pendant que vous suivez le flux de migration. Vous pouvez également utiliser l’[API d’exportation](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40). Vous pouvez les réimporter dans LUIS après la migration. Le processus d’importation crée une application (avec un nouvel ID d’application) dont vous êtes le propriétaire.
* Si vous êtes le propriétaire de l’application, vous n’aurez pas besoin d’exporter vos applications car elles migreront automatiquement. Un modèle d’e-mail contenant une liste de tous les collaborateurs de chaque application est fourni, afin que ces derniers puissent être informés du processus de migration.

> [!Note]
> La création de votre application LUIS est gratuite, comme indiqué par le niveau F0. Découvrez [plus d’informations sur les niveaux tarifaires](luis-limits.md#key-limits).


## <a name="migration-steps"></a>Étapes de la migration

1. Lorsque vous vous connectez au [portail LUIS](https://www.luis.ai), une fenêtre de migration Azure s’ouvre avec les étapes de la migration. Si vous la fermez, vous ne pourrez pas procéder à la création de vos applications LUIS et la seule action affichée sera de poursuivre la migration.

    > [!div class="mx-imgBorder"]
    > ![Présentation de la fenêtre de migration](./media/migrate-authoring-key/notify-azure-migration.png)

2. Si vous avez des collaborateurs sur une de vos applications, une liste de noms d’application dont vous êtes propriétaire s’affiche, ainsi que la région de création et les e-mails des collaborateurs de chaque application. Nous vous recommandons d’envoyer à vos collaborateurs un e-mail les informant de la migration en cliquant sur le bouton **Envoyer** à gauche du nom de l’application.
Un symbole `*` s’affiche en regard du nom de l’application si un collaborateur a une ressource de prédiction affectée à votre application. Après la migration, ces applications continuent d’affecter ces ressources de prédiction même si les collaborateurs n’ont pas accès à la création de vos applications. Toutefois, cette affectation est rompue si le propriétaire de la ressource de prédiction [a régénéré les clés](./luis-how-to-azure-subscription.md#regenerate-an-azure-key) à partir du Portail Azure.  

   > [!div class="mx-imgBorder"]
   > ![Notifier les collaborateurs](./media/migrate-authoring-key/notify-azure-migration-collabs.png)


   Pour chaque collaborateur et application, l’application de messagerie par défaut s’ouvre avec un e-mail légèrement formaté. Vous pouvez modifier l'e-mail avant de l'envoyer. Le modèle d'e-mail inclut l'ID exact de l'application et le nom de l'application.

   ```html
   Dear Sir/Madam,

   I will be migrating my LUIS account to Azure. Consequently, you will no longer have access to the following app:

   App Id: <app-ID-omitted>
   App name: Human Resources

   Thank you
   ```
   > [!Note]
   > Une fois votre compte migré vers Azure, vos applications ne sont plus disponibles pour les collaborateurs.

3. Si vous êtes un collaborateur d’une application, une liste de noms d’application partagée avec vous s’affiche avec la région de création et les e-mails de propriétaires de chaque application. Il est recommandé d’exporter une copie des applications en cliquant sur le bouton Exporter à gauche du nom de l’application. Vous pouvez réimporter celles-ci après la migration, car elles ne migreront pas automatiquement avec vous.
Un symbole `*` s’affiche en regard du nom de l’application si une ressource de prédiction est affectée à une application. Après la migration, votre ressource de prédiction est toujours affectée à ces applications, même si vous n’avez plus accès à la création de ces applications. Si vous souhaitez rompre l’affectation entre votre ressource de prédiction et l’application, vous devez accéder au Portail Azure et [régénérer les clés](./luis-how-to-azure-subscription.md#regenerate-an-azure-key).

   > [!div class="mx-imgBorder"]
   > ![Exporter vos applications](./media/migrate-authoring-key/migration-export-apps.png).


4. Dans la fenêtre de migration des régions, vous êtes invité à migrer vos applications vers une ressource Azure dans la même région que celle où elles ont été créées. LUIS a trois régions de création [et portails](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions#luis-authoring-regions). La fenêtre affiche les régions où vos applications détenues ont été créées. Les régions de migration affichées peuvent être différentes selon le portail régional que vous utilisez et les applications que vous avez créées. 

   > [!div class="mx-imgBorder"]
   > ![Migration de plusieurs régions.](./media/migrate-authoring-key/migration-regional-flow.png)

5. Pour chaque région, choisissez de créer une ressource de création LUIS ou de migrer vers une ressource existante en utilisant les boutons.

   > [!div class="mx-imgBorder"]
   > ![Choisir de créer ou de migrer vers une ressource de création existante](./media/migrate-authoring-key/migration-multiregional-resource.png)

   Fournissez les informations suivantes :

   * **Nom du locataire** : locataire auquel votre abonnement Azure est associé. Par défaut, ce paramètre est défini sur le locataire que vous utilisez actuellement. Vous pouvez changer de locataire en fermant cette fenêtre et en sélectionnant l’avatar contenant vos initiales dans le coin supérieur droit de l’écran. Cliquez sur **Migrer vers Azure** pour rouvrir la fenêtre.
   * **Nom de l’abonnement Azure** : abonnement à associer à la ressource. Si votre locataire possède plusieurs abonnements, sélectionnez celui que vous souhaitez dans la liste déroulante.
   * **Nom de la ressource de création** : nom personnalisé que vous choisissez. Il est utilisé comme élément de l’URL pour vos requêtes de création et de point de terminaison de prédiction. Si vous créez une ressource de création, notez que le nom associé peut uniquement contenir des caractères alphanumériques (`-`) et ne doit pas commencer ni se terminer par `-`. Si d’autres symboles sont inclus dans le nom, la création et la migration des ressources échouent.
   * **Nom du groupe de ressources Azure** : nom de groupe de ressources personnalisé que vous choisissez dans la liste déroulante. Les groupes de ressources vous permettent de regrouper des ressources Azure pour l’accès et la gestion. Si vous n’avez pas de groupe de ressources dans votre abonnement, vous ne serez pas autorisé à en créer un dans le portail LUIS. Accédez au [Portail Azure](https://ms.portal.azure.com/#create/Microsoft.ResourceGroup) pour en créer un, puis cliquez sur LUIS pour continuer le processus de connexion.

6. Une fois que vous avez effectué la migration dans toutes les régions, cliquez sur Terminer. Vous avez maintenant accès à vos applications. Vous pouvez continuer à créer et à gérer toutes vos applications dans toutes les régions dans le portail.


## <a name="using-apps-after-migration"></a>Utilisation des applications après la migration

Après le processus de migration, toutes les applications LUIS dont vous êtes propriétaire sont maintenant affectées à une seule ressource de création LUIS.
La liste **Mes applications** affiche les applications qui ont fait l’objet d’une migration vers la nouvelle ressource de création. Avant d’accéder à vos applications, sélectionnez **Choisir une autre ressource de création** afin de sélectionner l’abonnement et la ressource de création pour afficher les applications qui peuvent être créées.

> [!div class="mx-imgBorder"]
> ![Sélectionner un abonnement et une ressource de création](./media/migrate-authoring-key/select-sub-and-resource.png)


Si vous prévoyez de modifier vos applications par programmation, vous aurez besoin des valeurs de la clé de création. Pour afficher ces valeurs, cliquez sur **Gérer** en haut de l’écran dans le portail LUIS, puis sélectionnez **Ressources Azure**. Elles sont également disponibles sur la page **Clés et points de terminaison** de la ressource dans le portail Azure. Vous pouvez également créer des ressources de création supplémentaires et les affecter à partir de la même page.

## <a name="adding-contributors-to-authoring-resources"></a>Ajout de contributeurs à des ressources de création

[!INCLUDE [Manage contributors for the Azure authoring resource for language understanding](./includes/manage-contributors-authoring-resource.md)]

Découvrez [comment ajouter des contributeurs](luis-how-to-collaborate.md) à votre ressource de création. Les contributeurs ont accès à toutes les applications associées à cette ressource.

Vous pouvez ajouter des contributeurs à la ressource de création à partir du portail Azure, dans la page **Contrôle d’accès (IAM)** de cette ressource. Pour plus d’informations, consultez [Ajouter des contributeurs à votre application](luis-how-to-collaborate.md).

> [!Note]
> Si le propriétaire de l’application LUIS a migré et ajouté le collaborateur en tant que contributeur sur la ressource Azure, le collaborateur n’aura toujours pas accès à l’application, sauf s’il migre également.


## <a name="troubleshooting-the-migration-process"></a>Résolution des problèmes liés au processus de migration

Si vous ne trouvez pas votre abonnement Azure dans la liste déroulante :
* Vérifiez que vous disposez d’un abonnement Azure valide autorisé à créer des ressources Cognitive Services. Accédez au [portail Azure](https://ms.portal.azure.com) et vérifiez l’état de l’abonnement. Si vous n’en avez pas, [créez un compte Azure gratuit](https://azure.microsoft.com/free/cognitive-services/).
* Assurez-vous que vous êtes dans le locataire approprié associé à votre abonnement valide. Vous pouvez changer de locataire en sélectionnant l’avatar contenant vos initiales dans le coin supérieur droit de l’écran.

  > [!div class="mx-imgBorder"]
  > ![Page de changement de répertoires](./media/migrate-authoring-key/switch-directories.png)

Si vous disposez d’une ressource de création existante, mais ne la trouvez pas lors de la sélection de l’option **Use Existing Authoring resource** (Utiliser une ressource de création existante) :
* Votre ressource a probablement été créée dans une autre région que celle dans laquelle vous essayez de migrer.
* Créez plutôt une ressource à partir du portail LUIS.

Si vous sélectionnez l’option **Create New Authoring Resource** (Créer une ressource de création) et que la migration a échoué avec le message d’erreur « Failed retrieving user’s Azure information, retry again later » (Échec de la récupération des informations Azure de l’utilisateur, réessayez plus tard) :
* Il se peut que votre abonnement compte au moins 10 ressources de création par région et par abonnement. Si tel est le cas, vous ne pourrez pas créer de ressource de création.
* Effectuez la migration en sélectionnant l’option **Use Existing Authoring resource** (Utiliser une ressource de création existante), puis en choisissant l’une des ressources disponibles dans votre abonnement.

## <a name="create-new-support-request"></a>Créer une demande de support

Si vous rencontrez des problèmes de migration qui ne sont pas traités dans la section de résolution des problèmes, veuillez [créer une rubrique de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) et fournir les informations ci-dessous avec les champs suivants :

   * **Type de problème** : Techniques
   * **Abonnement**: choisissez un abonnement dans la liste déroulante.
   * **Service** : effectuez une recherche et sélectionnez Cognitive Services.
   * **Ressource** : choisissez une ressource LUIS si elle existe déjà. Sinon, sélectionnez Question générale.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les [concepts relatifs aux clés de création et de runtime](luis-how-to-azure-subscription.md).
* Découvrez comment [affecter des clés](luis-how-to-azure-subscription.md) et ajouter des [contributeurs](luis-how-to-collaborate.md).
