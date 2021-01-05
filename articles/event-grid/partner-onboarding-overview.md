---
title: Vue d’ensemble de l’intégration de partenaires (Azure Event Grid)
description: Fournit une vue d’ensemble de l’intégration en tant que partenaire Event Grid.
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: 2a2e33395cabd368d5d5d870dd0461e4cbd37e0d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93081193"
---
# <a name="partner-onboarding-overview-azure-event-grid"></a>Vue d’ensemble de l’intégration de partenaires (Azure Event Grid)

Cet article explique comment utiliser les ressources de partenaire Azure Event Grid de manière privée et comment effectuer la conversion en type de rubrique de partenaire disponible publiquement.

Vous n’avez pas besoin d’une autorisation spéciale pour commencer à utiliser les types de ressources Event Grid associés à la publication d’événements en tant que partenaire Event Grid. En fait, vous pouvez les utiliser dès aujourd’hui pour publier des événements de manière privée sur vos propres abonnements Azure et tester le modèle de ressource si vous envisagez de devenir partenaire.

> [!NOTE]
> Pour obtenir des instructions pas à pas sur la façon d’intégrer en tant que partenaire Event Grid à l’aide du portail Azure, consultez [Comment intégrer en tant que partenaire Event Grid (Portail Azure)](onboard-partner.md). 

## <a name="how-partner-events-work"></a>Fonctionnement d’Événements partenaires
La fonctionnalité Événements partenaires exploite l’architecture déjà utilisée dans Event Grid pour publier des événements à partir de ressources Azure, telles que Stockage Azure et Azure IoT Hub, et rend ces outils accessibles publiquement à tous les utilisateurs. Par défaut, l’utilisation de ces outils est privée pour votre abonnement Azure uniquement. Pour rendre vos événements disponibles publiquement, renseignez le formulaire et [contactez l’équipe Event Grid](mailto:gridpartner@microsoft.com).

La fonctionnalité Événements partenaires vous permet de publier des événements dans Azure Event Grid à des fins de consommation mutualisée.

## <a name="onboarding-and-event-publishing-overview"></a>Présentation de l’intégration et de la publication d’événements

### <a name="partner-flow"></a>Flux du partenaire

1. Créez un locataire Azure, le cas échéant.
1. Utilisez l’interface de ligne de commande Azure pour créer une instance Event Grid `partnerRegistration`. Cette ressource contient des informations telles que le nom d’affichage, la description, l’URI d’installation, etc.

    ![Créer une rubrique de partenaire](./media/partner-onboarding-how-to/create-partner-registration.png)

1. Créez un ou plusieurs espaces de noms de partenaire dans chaque région dans laquelle vous souhaitez publier des événements. Le service Event Grid provisionne un point de terminaison de publication (par exemple, `https://contoso.westus-1.eventgrid.azure.net/api/events`) ainsi que des clés d’accès.

    ![Créer un espace de noms de partenaire](./media/partner-onboarding-how-to/create-partner-namespace.png)

1. Permet aux clients d’indiquer à votre système qu’ils veulent bénéficier d’une rubrique de partenaire.
1. Contactez l’équipe Event Grid pour lui indiquer que vous souhaitez que votre type de rubrique de partenaire devienne public.

### <a name="customer-flow"></a>Flux du client

1. Votre client se rend sur le portail Azure pour noter l’ID d’abonnement Azure et le groupe de ressources au sein desquels créer la rubrique de partenaire.
1. Le client demande une rubrique de partenaire par le biais de votre système. En réponse, vous créez un tunnel d’événements pour votre espace de noms de partenaire.
1. Event Grid crée une rubrique de partenaire **en attente** dans le groupe de ressources et l’abonnement Azure du client.

    ![Créer un canal d’événements](./media/partner-onboarding-how-to/create-event-tunnel-partner-topic.png)

1. Le client active la rubrique de partenaire dans le portail Azure. Les événements peuvent maintenant passer de votre service à l’abonnement Azure du client.

    ![Activer une rubrique de partenaire](./media/partner-onboarding-how-to/activate-partner-topic.png)

## <a name="resource-model"></a>Modèle de ressource
Le modèle de ressource suivant est destiné à Événements partenaires.

### <a name="partner-registrations"></a>Inscriptions de partenaires
* Ressource : `partnerRegistrations`
* Utilisée par : Partenaires
* Description : Capturent les métadonnées globales du partenaire SaaS (logiciel en tant que service), par exemple, le nom, le nom d’affichage, la description et l’URI d’installation.
    
    La création ou la mise à jour d’une inscription de partenaire est une opération que les partenaires peuvent effectuer en libre-service. Ainsi, ils peuvent créer et tester le processus complet de bout en bout.
    
    Seules les inscriptions de partenaires approuvées par Microsoft peuvent être découvertes par les clients.
* Étendue : Créées dans l’abonnement Azure du partenaire. Les métadonnées sont visibles pour les clients dès lors qu’elles sont rendues publiques.

### <a name="partner-namespaces"></a>Espaces de noms de partenaires
* Ressource : `partnerNamespaces`
* Utilisée par : Partenaires
* Description : Fournit une ressource régionale dans laquelle publier des événements clients. Chaque espace de noms de partenaire possède un point de terminaison de publication et des clés d’authentification. C’est également par l’espace de noms que le partenaire demande une rubrique de partenaire pour un client donné et liste les clients actifs.
* Étendue : Existent au sein de l’abonnement du partenaire.

### <a name="event-channel"></a>Canal d’événements
* Ressource : `partnerNamespaces/eventChannels`
* Utilisée par : Partenaires
* Description : Les canaux d’événements sont un miroir de la rubrique de partenaire du client. En créant un canal d’événements et en spécifiant l’abonnement Azure et le groupe de ressources du client dans les métadonnées, vous indiquez à Event Grid de créer une rubrique de partenaire pour le client. Event Grid émet un appel Azure Resource Manager pour créer une rubrique de partenaire correspondante dans l’abonnement du client. La rubrique de partenaire est créée à l’état « En attente ». Il existe un lien un-à-un entre chaque canal d’événements et chaque rubrique de partenaire.
* Étendue : Existe au sein de l’abonnement du partenaire.

### <a name="partner-topics"></a>Rubriques de partenaire
* Ressource : `partnerTopics`
* Utilisée par : Clients
* Description : Les rubriques de partenaires sont similaires aux rubriques personnalisées et système disponibles dans Event Grid. Chaque rubrique de partenaire est associée à une source spécifique (par exemple, `Contoso:myaccount`) et à un type de rubrique de partenaire spécifique (par exemple, Contoso). Les clients créent des abonnements d’événements dans la rubrique de partenaire pour router les événements vers différents gestionnaires d’événements.

    Les clients ne peuvent pas créer cette ressource directement. La seule façon de créer une rubrique de partenaire consiste à effectuer une opération de partenaire pour créer un canal d’événements.
* Étendue : Existent au sein de l’abonnement du client.

### <a name="partner-topic-types"></a>Types de rubrique de partenaire
* Ressource : `partnerTopicTypes`
* Utilisée par : Clients
* Description : Les types de rubriques de partenaire sont des types de ressources à l’échelle du locataire qui permettent aux clients de découvrir la liste des types de rubriques de partenaire approuvés. L’URL est de type https://management.azure.com/providers/Microsoft.EventGrid/partnerTopicTypes).
* Étendue : Global

## <a name="publish-events-to-event-grid"></a>Publier des événements sur Event Grid
Quand vous créez un espace de noms de partenaire dans une région Azure, vous obtenez un point de terminaison régional et les clés d’authentification correspondantes. Publiez des lots d’événements sur ce point de terminaison pour tous les canaux d’événements clients dans cet espace de noms. En fonction du champ source de l’événement, Azure Event Grid mappe chaque événement avec les rubriques de partenaire correspondantes.

### <a name="event-schema-cloudevents-v10"></a>Schéma d’événement : CloudEvents v1.0
Publiez des événements sur Azure Event Grid à l’aide du schéma CloudEvents 1.0. Event Grid prend en charge le mode structuré et le mode par lot. CloudEvents 1.0 est le seul schéma d’événement pris en charge pour les espaces de noms de partenaire.

### <a name="example-flow"></a>Exemple de flux

1.  Le service de publication effectue une publication HTTP sur `https://contoso.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01`.
1.  Dans la requête, incluez une valeur d’en-tête nommée aeg-sas-key, qui contient une clé pour l’authentification. Cette clé est provisionnée pendant la création de l’espace de noms de partenaire. Par exemple, une valeur d’en-tête valide est aeg-sas-key: VXbGWce53249Mt8wuotr0GPmyJ/nDT4hgdEj9DpBeRr38arnnm5OFg = =.
1.  Définissez l’en-tête Content-Type sur « application/cloudevents-batch+json; charset=UTF-8a ».
1.  Exécutez une requête HTTP POST sur l’URL de publication avec un lot d’événements correspondant à cette région. Par exemple :

``` json
[
{
    "specversion" : "1.0-rc1",
    "type" : "com.contoso.ticketcreated",
    "source" : " com.contoso.account1",
    "subject" : "tickets/123",
    "id" : "A234-1234-1234",
    "time" : "2019-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
          object-unique-to-each-publisher
    }
},
{
    "specversion" : "1.0-rc1",
    "type" : "com.contoso.ticketclosed",
    "source" : "https://contoso.com/account2",
    "subject" : "tickets/456",
    "id" : "A234-1234-1234",
    "time" : "2019-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
          object-unique-to-each-publisher
    }
}
]
```

Après la publication sur le point de terminaison de l’espace de noms du partenaire, vous recevez une réponse. La réponse est un code de réponse HTTP standard. Certaines réponses courantes sont :

| Résultats                             | response              |
|------------------------------------|-----------------------|
| Succès                            | 200 OK                |
| Les données d’événement ont un format incorrect    | 400 Demande incorrecte       |
| Clé d’accès non valide                 | 401 Non autorisé      |
| Point de terminaison incorrecte                 | 404 Introuvable         |
| La taille du tableau ou de l’événement dépasse la limite autorisée | 413 charge utile maximale dépassée |

## <a name="references"></a>References

  * [Swagger](https://github.com/ahamad-MS/azure-rest-api-specs/blob/master/specification/eventgrid/resource-manager/Microsoft.EventGrid/preview/2020-04-01-preview/EventGrid.json)
  * [Modèle ARM](/azure/templates/microsoft.eventgrid/allversions)
  * [Schéma de modèle ARM](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2020-04-01-preview/Microsoft.EventGrid.json)
  * [API REST](/rest/api/eventgrid/version2020-04-01-preview/partnernamespaces)
  * [Extension CLI](/cli/azure/ext/eventgrid/)

### <a name="sdks"></a>Kits SDK
  * [.NET](https://www.nuget.org/packages/Microsoft.Azure.Management.EventGrid/5.3.1-preview)
  * [Python](https://pypi.org/project/azure-mgmt-eventgrid/3.0.0rc6/)
  * [Java](https://search.maven.org/artifact/com.microsoft.azure.eventgrid.v2020_04_01_preview/azure-mgmt-eventgrid/1.0.0-beta-3/jar)
  * [Ruby](https://rubygems.org/gems/azure_mgmt_event_grid/versions/0.19.0)
  * [JS](https://www.npmjs.com/package/@azure/arm-eventgrid/v/7.0.0)
  * [Go](https://github.com/Azure/azure-sdk-for-go)


## <a name="next-steps"></a>Étapes suivantes
- [Vue d’ensemble des rubriques de partenaire](partner-events-overview.md)
- [Formulaire d’intégration des rubriques de partenaire](https://aka.ms/gridpartnerform)
- [Rubrique de partenaire Auth0](auth0-overview.md)
- [Comment utiliser la rubrique de partenaire Auth0](auth0-how-to.md)
