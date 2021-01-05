---
title: Intégration en tant que partenaire Azure Event Grid à l’aide du portail Azure
description: Utilisez le portail Azure pour intégrer un partenaire Azure Event Grid.
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: 8344fcd822bfcdc67e2c27b200d97fa70dfefdf2
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349384"
---
# <a name="onboard-as-an-azure-event-grid-partner-using-the-azure-portal"></a>Intégration en tant que partenaire Azure Event Grid à l’aide du portail Azure
Cet article décrit la façon dont les fournisseurs SaaS tiers, également appelés éditeurs ou partenaires d’événements, sont intégrés à Event Grid pour être en mesure de publier des événements à partir de leurs services et de déterminer la façon dont ces événements sont utilisés par les clients finaux.

> [!IMPORTANT]
> Si vous ne connaissez pas les Événements partenaires, consultez [Vue d’ensemble des événements partenaires](partner-events-overview.md) pour obtenir une présentation détaillée des concepts clés qui sont essentiels pour comprendre et suivre les étapes décrites dans cet article.

## <a name="onboarding-process-for-event-publishers-partners"></a>Processus d’intégration pour les éditeurs d’événements (partenaires)
Pour résumer, l’activation des événements liés à votre service par les utilisateurs implique généralement le processus suivant :

1. **Manifestez votre intérêt** à l’idée de devenir partenaire de l’équipe de service Event Grid avant de passer aux étapes suivantes.
1. Créez un type de rubrique partenaire en créant une **inscription**. 
1. Créez un **espace de noms**.
1. Créer un **canal d’événements** et une **rubrique de partenaire** (étape unique).
1. Testez la fonctionnalité des événements partenaire de bout en bout.

Pour l’étape 4, vous devez déterminer le type d’expérience utilisateur que vous souhaitez proposer. Les options suivantes s’offrent à vous :
- Fournissez votre propre solution, en général une expérience de l’interface graphique utilisateur (GUI) web, hébergée par votre domaine à l’aide de notre kit de développement logiciel (SDK) et/ou de l’API REST pour créer un canal d’événements et sa rubrique partenaire correspondante. Avec cette option, vous pouvez demander à l’utilisateur l’abonnement et le groupe de ressources pour lesquels vous allez créer une rubrique de partenaire.
- Utilisez le portail Azure ou l’interface de ligne de commande pour créer le canal d’événements et la rubrique de partenaire associée. Avec cette option, vous devez avoir accès à l’abonnement et au groupe de ressources pour lesquels vous allez créer une rubrique de partenaire. 

Cet article vous montre comment procéder à l’Intégration en tant que partenaire Azure Event Grid à l’aide du portail Azure. 

> [!NOTE]
> L’inscription d’un type de rubrique de partenaire est une étape facultative. Pour vous aider à déterminer si vous devez créer un type de rubrique de partenaire, consultez [Ressources gérées par l’éditeur d’événements](partner-events-overview.md#resources-managed-by-event-publishers).

## <a name="communicate-your-interest-in-becoming-a-partner"></a>Manifestez votre intérêt de devenir partenaire
Renseignez [ce formulaire](https://aka.ms/gridpartnerform) et contactez l’équipe Event Grid au [GridPartner@microsoft.com](mailto:GridPartner@microsoft.com). Nous discuterons avec vous en vous fournissant des informations détaillées sur les scénarios d’utilisation des événements des partenaires, les personnages, le processus d’intégration, les fonctionnalités, la tarification et bien plus encore.

## <a name="prerequisites"></a>Prérequis
Pour effectuer les étapes restantes, assurez vous de disposer des éléments suivants :

- Un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Un [locataire](../active-directory/develop/quickstart-create-new-tenant.md) Azure.

## <a name="register-a-partner-topic-type-optional"></a>Inscrire un type de rubrique de partenaire (facultatif)
1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Tous les services** dans le volet de navigation de gauche, puis entrez **Inscriptions des partenaires Event Grid** dans la barre de recherche, puis sélectionnez-le. 
1. Dans la page **Inscriptions des partenaires Event Grid**, sélectionnez **+ Ajouter** dans la barre d’outils. 

    :::image type="content" source="./media/onboard-partner/add-partner-registration-link.png" alt-text="Ajoute un lien d’inscription de partenaire":::
1. Sur la page **Créer les inscriptions de type de rubrique de partenaire - De base**, entrez les informations suivantes : 
    1. Dans la section **Détails du projet**, effectuez les étapes suivantes :
        1. Sélectionnez votre **abonnement** Azure. 
        1. Sélectionnez un **groupe de ressources** Azure existant ou créez-en un. 
    1. Dans la section **Détails de l’inscription**, effectuez les étapes suivantes :
        1. Pour **Nom d’inscription**, entrez un nom pour l’inscription. 
        1. Pour **Nom de l’organisation**, entrez le nom de votre organisation. 
    1. Dans la section **Type de ressource de partenaire**, entrez les détails sur votre type de ressource qui sera affiché sur la page **de création d’une rubrique de partenaire** : 
        1. Pour **Nom du type de ressource de partenaire**, entrez le nom du type de ressource. Il s’agit du type de rubrique de partenaire qui sera créé dans votre abonnement Azure. 
        2. Pour **Nom d’affichage**, entrez un nom d’affichage convivial pour le type de rubrique de partenaire (ressource). 
        3. Entrez une **description pour le type de ressource**. 
        4. Entrez une **description pour le scénario**. Il doit expliquer les procédures ou les scénarios selon lesquels les rubriques de partenaires de vos ressources peuvent être utilisées.  

            :::image type="content" source="./media/onboard-partner/create-partner-registration-page.png" alt-text="Créer une inscription de partenaire":::            
1. Sélectionnez **Suivant : Service personnalisé** en bas de la page. Sous l’onglet **Service clientèle** de la page **Créer une inscription de partenaire**, entrez les informations que les utilisateurs de l’abonné utiliseront pour vous contacter en cas de problème avec la source de l’événement :
    1. Obtenez le **numéro de téléphone**.
    1. Entrez l’**extension** du numéro de téléphone.
    1. Entrez l’**URL** d’un site web de support. 
    
        :::image type="content" source="./media/onboard-partner/create-partner-registration-customer-service.png" alt-text="Créer une inscription de partenaire - service client":::        
1. Sélectionnez **Suivant : Étiquettes** au bas de la page. 
1. Sur la page **Étiquettes**, configurez les valeurs suivantes. 
    1. Entrez un **nom** et une **valeur** pour l'étiquette que vous souhaitez ajouter. Cette étape est **facultative**. 
    1. Sélectionnez **Vérifier + créer** au bas de la page pour créer l’inscription (type de rubrique de partenaire).

## <a name="create-a-partner-namespace"></a>Créer un espace de noms de partenaire

1. Sur le portail Azure, sélectionnez **Tous les services** dans le menu de navigation de gauche, puis entrez **Espaces de noms de partenaires Event Grid** dans la barre de recherche, puis sélectionnez-le dans la liste. 
1. Sur la page **Espaces de noms des partenaires Event Grid**, sélectionnez **+ Ajouter** dans la barre d’outils. 
    
    :::image type="content" source="./media/onboard-partner/add-partner-namespace-link.png" alt-text="Espaces de noms de partenaires - Ajouter un lien":::
1. Sur la page **Créer un espace de noms de partenaire - De base**, renseignez les informations suivantes.
    1. Dans la section **Détails du projet**, effectuez les étapes suivantes : 
        1. Sélectionnez un **abonnement** Azure.
        1. Sélectionnez un **groupe de ressources** existant ou créez-en un. 
    1. Dans la section **Détails de l’espace de noms**, effectuez les étapes suivantes :
        1. Entrez un **nom** pour l’espace de noms. 
        1. Sélectionnez un **emplacement** pour l’espace de noms. 
    1. Dans la section **Détails de l’inscription**, procédez comme suit pour associer l’espace de noms à une inscription de partenaire. 
        1. Sélectionnez l’**abonnement** dans lequel l’inscription du partenaire existe. 
        1. Sélectionnez le **groupe de ressources** qui contient l’inscription du partenaire. 
        1. Sélectionnez l’**inscription de partenaire** dans la liste déroulante.
    1. Sélectionnez **Suivant : Étiquettes** au bas de la page.

        :::image type="content" source="./media/onboard-partner/create-partner-namespace-basics-page.png" alt-text="Page Créer un espace de noms de partenaire - De base":::
1. Sur la page **Étiquettes**, ajoutez des étiquettes (facultatif).
    1. Entrez un **nom** et une **valeur** pour l'étiquette que vous souhaitez ajouter. Cette étape est **facultative**.
    1. Au bas de la page, sélectionnez **Examiner et créer**.         
1. Sur la page **Vérifier + créer**, vérifiez les détails, puis sélectionnez **Créer**. 

## <a name="create-an-event-channel"></a>Créer un canal d’événements
> [!IMPORTANT]
> Vous devez demander à votre utilisateur un abonnement Azure, un groupe de ressources, un emplacement et un nom de rubrique partenaire pour créer une rubrique de partenaire que votre utilisateur détiendra et gèrera.

1. Accédez à la page **Vue d’ensemble** de l’espace de noms que vous avez créé. 

    :::image type="content" source="./media/onboard-partner/partner-namespace-overview.png" alt-text="Page Espace de noms de partenaire - Vue d’ensemble":::
    partner-namespace-overview.png
1. Sélectionnez **+ Canal d’événements** dans la barre d’outils. 
1. Sur la page **Créer un canal d’événements - De base**, renseignez les informations suivantes. 
    1. Dans la section **Détails du canal**, procédez comme suit :
        1. Pour **Nom du canal d’événements**, entrez un nom pour le canal d’événements. 
        1. Entrez la **source**. Consultez [Spécifications des événements cloud 1.0](https://github.com/cloudevents/spec/blob/v1.0/spec.md#source-1) pour avoir une idée de la valeur appropriée pour la source. Consultez également [cet exemple de schéma d’événements cloud](cloud-event-schema.md#sample-event-using-cloudevents-schema).
        1. Entrez la source (DE QUOI S’AGIT-IL ?).
    1. Dans la section **Détails de la destination**, entrez les détails de la rubrique du partenaire de destination qui sera créée pour ce canal d’événements. 
        1. Entrez l’**ID de l’abonnement** dans lequel la rubrique partenaire sera créée. 
        1. Entrez le **nom du groupe de ressources** dans lequel la ressource de la rubrique de partenaire sera créée. 
        1. Entrez un **nom pour la rubrique de partenaire**. 
    1. Sélectionnez **Suivant : Filtres** au bas de la page. 
    
        :::image type="content" source="./media/onboard-partner/create-event-channel-basics-page.png" alt-text="Page Créer un canal d’événements - De base":::
1. Sur la page **Filtres**, ajoutez des filtres. Procédez comme suit :
    1. Filtrez les attributs de chaque événement. Seuls les événements qui correspondent à tous les filtres sont affichés. Jusqu'à 25 filtres peuvent être spécifiés. Les comparaisons ne respectent pas la casse. Les clés valides utilisées pour les filtres varient en fonction du schéma d’événement. Dans l’exemple suivant, `eventid`, `source`, `eventtype` et `eventtypeversioin` peuvent être utilisés comme clés. Vous pouvez également utiliser des propriétés personnalisées dans la charge utile de données, en utilisant `.` comme opérateur d’imbrication. Par exemple : `data`, `data.key`, `data.key1.key2`.
    1. Sélectionnez **Suivant : Fonctionnalités supplémentaires** en bas de la page. 
    
        :::image type="content" source="./media/onboard-partner/create-event-channel-filters-page.png" alt-text="Page Créer un canal d’événements - Filtres":::
        create-event-channel-filters-page.png
1. Sur la page **Fonctionnalités supplémentaires**, vous pouvez définir une **heure d’expiration** et une **description pour la rubrique de partenaire**. 
    1. L’**heure d’expiration** est l’heure à laquelle la rubrique et son canal d’événements associé seront automatiquement supprimés s’ils ne sont pas activés par le client. Une valeur par défaut de sept jours est utilisée dans le cas où aucune heure n’est définie. Activez la case à cocher pour spécifier votre propre heure d’expiration. 
    1. Comme cette rubrique est une ressource qui n’est pas créée par l’utilisateur, une **description** peut aider l’utilisateur à comprendre la nature de cette rubrique. Une description générique sera renseignée si aucune n’est définie. Activez la case à cocher pour définir votre propre description de rubrique de partenaire. 
    1. Sélectionnez **Suivant : Vérifier + créer**. 
    
        :::image type="content" source="./media/onboard-partner/create-event-channel-additional-features-page.png" alt-text="Page Créer un canal d’événements - fonctionnalités supplémentaires":::
1. Dans la page **Vérifier + créer**, passez en revue tous les paramètres et sélectionnez **Créer** pour créer le canal d’événements. 

## <a name="next-steps"></a>Étapes suivantes
- [Vue d’ensemble des rubriques de partenaire](./partner-events-overview.md)
- [Formulaire d’intégration des rubriques de partenaire](https://aka.ms/gridpartnerform)
- [Rubrique de partenaire Auth0](auth0-overview.md)
- [Comment utiliser la rubrique de partenaire Auth0](auth0-how-to.md)