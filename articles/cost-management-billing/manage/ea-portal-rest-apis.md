---
title: API REST Entreprise Azure
description: Cet article décrit les API REST qui sont disponibles avec votre inscription Entreprise Azure.
author: bandersmsft
ms.author: banders
ms.date: 09/03/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: c4c99142c64278514066efa8925ed8e3f6617235
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132582"
---
# <a name="azure-enterprise-rest-apis"></a>API REST Entreprise Azure

Cet article décrit les API REST qui sont disponibles avec votre inscription Entreprise Azure. Il explique également comment résoudre les problèmes courants liés aux API REST.

## <a name="consumption-and-usage-apis"></a>API de consommation et d’utilisation

Les clients d’Entreprise Microsoft Azure peuvent obtenir des données sur l’utilisation et la facturation par le biais des API REST. Le propriétaire du rôle (Administrateur d’entreprise, Administrateur de service, Propriétaire du compte) doit activer l’accès à l’API en générant une clé à partir du portail Azure EA. Ensuite, toute personne disposant du numéro et de la clé d’inscription peut accéder aux données par l’API.

### <a name="available-apis"></a>API disponibles

**Solde et résumé** : utilisez l’[API Solde et résumé](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) pour obtenir un résumé mensuel des informations sur les soldes, les nouveaux achats, les frais du service Place de marché Azure, les ajustements et les frais de dépassement. Pour plus d’informations, consultez [API de création de rapports pour les clients d’Entreprise : Solde et résumé](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary).

**Détails de l’utilisation** : l’[API Détails de l’utilisation](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail) fournit une analyse quotidienne des quantités consommées et des frais estimés par inscription. Le résultat inclut également des informations sur les instances, les compteurs et les services. Vous pouvez interroger l’API par période de facturation, ou pour une plage donnée en spécifiant les dates de début et de fin. Pour plus d’informations, consultez [API de création de rapports pour les clients d’Entreprise : Détails de l’utilisation](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail).

**Frais du magasin de la Place de marché** : l’[API Frais du magasin de la Place de marché](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) présente l’analyse détaillée des frais de la Place de marché basés sur l’utilisation par jour pour la période de facturation ou la plage entre les dates de début et de fin indiquées. Pour plus d’informations, consultez [API de création de rapports pour les clients d’Entreprise :Frais du magasin de la Place de marché](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge).

**Grille tarifaire** : l’[API Grille tarifaire](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) indique les tarifs applicables pour chaque compteur selon l’inscription et la période de facturation. Pour plus d’informations, consultez [API de création de rapports pour les clients d’Entreprise : Grille tarifaire](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet).

**Périodes de facturation** : l’[API Périodes de facturation](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) affiche, par ordre chronologique inverse, une liste des périodes de facturation comportant les données de consommation pour une inscription. Chaque période contient une propriété qui pointe vers la route de l’API pour les quatre ensembles de données : BalanceSummary (Solde et résumé), UsageDetails (Détails de l’utilisation), Marketplace Charges (Frais de la Place de marché) et PriceSheet (Grille tarifaire). Pour plus d’informations, consultez [API de création de rapports pour les clients d’Entreprise : Périodes de facturation](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods).

### <a name="enable-api-data-access"></a>Activer l’accès aux données d’API

Les propriétaires de rôle peuvent effectuer les étapes suivantes dans le portail Azure EA. Après avoir accédé à **Reports** > **Download Usage** > **API Access Key** (Rapports > Télécharger l’utilisation > Clé d’accès API), ils peuvent :

- Générer les clés d’accès principale et secondaire.
- Désactiver les clés d’accès.
- Consulter les dates de début et de fin des clés d’accès.

### <a name="generate-or-retrieve-the-api-key"></a>Générer ou récupérer la clé API

1. Connectez-vous en tant qu’administrateur d’entreprise.
2. Cliquez sur **Reports** (Rapports) dans la fenêtre de navigation de gauche, puis cliquez sur l’onglet **Download Usage** (Télécharger l’utilisation).
3. Cliquez sur **API Access Key** (Clé d’accès API).
4. Sous **Enrollment Access Keys** (Clés d’accès d’inscription), sélectionnez le symbole de génération de clé pour générer une clé primaire ou secondaire.
5. Sélectionnez **Expand Key** (Développer la clé) pour afficher l’intégralité de la clé d’accès API générée.
6. Sélectionnez **Copier** pour obtenir la clé d’accès API et l’utiliser immédiatement.

![Exemple de page API Access Key (Clé d’accès API)](./media/ea-portal-rest-apis/ea-create-generate-or-retrieve-the-api-key.png)

Si vous souhaitez attribuer les clés d’accès API à des utilisateurs qui ne sont pas administrateurs d’entreprise dans votre inscription, effectuez les étapes suivantes :

1. Dans la fenêtre de navigation de gauche, cliquez sur **Manage** (Gérer).
2. Cliquez sur le symbole de crayon à côté de **DA view charges** (Afficher les frais pour l’administrateur de service).
3. Sélectionnez **Enabled** (Activé), puis cliquez sur **Save** (Enregistrer).
4. Cliquez sur le symbole de crayon à côté de **AO view charges** (Afficher les frais pour le propriétaire du compte).
5. Sélectionnez **Enabled** (Activé), puis cliquez sur **Save** (Enregistrer).

![Exemple d’activation de l’affichage des frais pour l’administrateur de service et le propriétaire de compte](./media/ea-portal-rest-apis/create-ea-generate-or-retrieve-api-key-enable-ao-do-view.png). Les étapes précédentes donnent aux détenteurs des clés d’accès API l’accès aux informations sur les coûts et les tarifs dans les rapports d’utilisation.

### <a name="pass-keys-in-the-api"></a>Passer les clés dans l’API

Passez la clé API pour chaque appel en vue de l’authentification et l’autorisation. Passez la propriété suivante aux en-têtes HTTP :

| Clé d’en-tête de demande | Valeur |
| --- | --- |
| Autorisation | Indiquez la valeur dans ce format : **bearer {API\_KEY}**
Exemple : bearer \&lt;APIKey\&gt; |

### <a name="swagger"></a>Fichier Swagger

Un point de terminaison Swagger est disponible sur [Enterprise Reporting v3 APIs](https://consumption.azure.com/swagger/ui/index) pour les API suivantes. Swagger est un outil qui facilite l’inspection de l’API. Utilisez Swagger pour générer des SDK clients avec [AutoRest](https://github.com/Azure/AutoRest) ou [Swagger CodeGen](https://swagger.io/swagger-codegen/). Les données disponibles après le 1er mai 2014 sont accessibles par le biais de l’API.

### <a name="api-response-codes"></a>Codes de réponse de l’API

Quand vous utilisez une API, plusieurs codes d’état de réponse peuvent s’afficher. Le tableau suivant les décrit.

| Code d’état de la réponse | Message | Description |
| --- | --- | --- |
| 200 | OK | Aucune erreur |
| 401 | Non autorisé | Clé API introuvable, non valide, expirée, etc. |
| 404 | Non disponible | Point de terminaison de rapport introuvable |
| 400 | Demande incorrecte | Paramètres non valides : plages de dates, nombres de Contrats Entreprise (EA), etc. |
| 500 | Erreur de serveur | Erreur inattendue lors du traitement de la requête |

### <a name="usage-and-billing-data-update-frequency"></a>Fréquence de mise à jour des données d’utilisation et de facturation

Les fichiers de données d’utilisation et de facturation sont mis à jour toutes les 24 heures pour le mois de facturation actuel. Toutefois, la latence des données peut atteindre trois jours. Par exemple, si l’utilisation a eu lieu le lundi, vous devrez peut-être attendre le jeudi pour voir les données reflétées dans le fichier de données.

### <a name="test-enrollment-for-development"></a>Inscription Test pour le développement

Si vous êtes un partenaire ou un développeur sans inscription Entreprise Azure et que vous souhaitez accéder à l’API, utilisez l’inscription Test. Le nom d’inscription est _EnrollmentNumber 100_. Vous pouvez rechercher et tester les informations d’utilisation jusqu’en juin 2018. Vous pouvez ensuite utiliser la clé suivante pour appeler l’API et voir des exemples de données.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImpoeXA2UU9DWlZmY1pmdmhDVGR1OFdxeTJ5byJ9.eyJFbnJvbGxtZW50TnVtYmVyIjoiMTAwIiwiSWQiOiI1ZTc2ZmNiMy0xN2I4LTQ5ZDItYjdkOC0zMDU0YjUwOWY0MWYiLCJSZXBvcnRWaWV3IjoiU3lzdGVtIiwiUGFydG5lcklkIjoiIiwiRGVwYXJ0bWVudElkIjoiIiwiQWNjb3VudElkIjoiIiwiaXNzIjoiZWEubWljcm9zb2Z0YXp1cmUuY29tIiwiYXVkIjoiY2xpZW50LmVhLm1pY3Jvc29mdGF6dXJlLmNvbSIsImV4cCI6MTU4NjM5MDA2OSwibmJmIjoxNTcwNTc4ODY5fQ.lENR5pCBph6iZCVexUlN1b-j7StaILCyBewVHoILD-_fn8S2o2bHY1qUseGOkBwNlaFQfk2OZIo-jQYvnf3eP3UNrNVTCINT0APbc1RqgwSjZSxugVVHH9jnSzEjONkJaSKmi4tlidk6zkF1-uY-TPJkKxYN_9ar7BgLshF9JGXk7t8OZhxSCxDZc-smntu6ORFDl4gRZZVBKXhqOGjOAdYX5tPiGDF2Bxb68RSzh9Xyr5PXxKLx5yivZzUdo0-GFHo13V9w6a5VQM4R1w4_ro8jF8WAo3mpGZ_ovx_U5IY6zMNmi_AoA1mUyvTGotgcu94RragutoJRxAGHbNJZ0Q
```

### <a name="azure-service-catalog"></a>Catalogue des services Azure

Tous les services Azure sont proposés dans un catalogue au format CSV, publié dans un blog Azure Storage. Ce catalogue est utile si vous devez créer un catalogue organisé de tous les services Azure disponibles pour votre système. Le catalogue actuel est consultable sur [https://azurecatalog.blob.core.windows.net/catalog/AzureCatalog.csv](https://azurecatalog.blob.core.windows.net/catalog/AzureCatalog.csv).

### <a name="csv-data-file-details"></a>Détails du fichier de données CSV

Les informations suivantes décrivent les propriétés des rapports d’API.

#### <a name="usage-summary"></a>Récapitulatif de l’utilisation

Le format JSON est généré à partir du rapport CSV. Le format est donc le même que le format CSV du récapitulatif. Le nom de la colonne étant complexe, vous devez le désérialiser dans une table de données quand vous consommez les données du récapitulatif JSON.

| Nom de colonne CSV | Nom de colonne JSON | Nouvelle colonne JSON | Commentaire |
| --- | --- | --- | --- |
| AccountOwnerId | AccountOwnerLiveId | AccountOwnerLiveId |   |
| Nom du compte | AccountName | AccountName |   |
| ServiceAdministratorId | ServiceAdministratorLiveId | ServiceAdministratorLiveId |   |
| SubscriptionId | SubscriptionId | SubscriptionId |   |
| SubscriptionGuid | MOCPSubscriptionGuid | SubscriptionGuid |   |
| Nom d'abonnement | SubscriptionName | SubscriptionName |   |
| Date | Date | Date | Affiche la date à laquelle le rapport du catalogue de services a été généré. Le format est une chaîne de date sans horodatage. |
| Month | Month | Month |   |
| jour | jour | jour |   |
| Year | Year | Year |   |
| Produit | BillableItemName | Produit |   |
| ID du compteur | ResourceGUID | ID du compteur |   |
| Catégorie du compteur | Service | MeterCategory | Facilite la recherche des services. Utile pour les services qui ont plusieurs ServiceType. Par exemple, Machines virtuelles. |
| Sous-catégorie du compteur | ServiceType | MeterSubCategory | Fournit un deuxième niveau de détails pour un service. Par exemple, A1 VM (Non-Windows).  |
| Région du compteur | ServiceRegion | MeterRegion | Troisième niveau de détails requis pour un service. Utile pour trouver le contexte de la région du ResourceGUID. |
| Nom du compteur | ServiceResource | MeterName | Nom du service. |
| Quantité consommée | ResourceQtyConsumed | ConsumedQuantity |   |
| ResourceRate | ResourceRate | ResourceRate |   |
| ExtendedCost | ExtendedCost | ExtendedCost |   |
| Emplacement de la ressource | ServiceSubRegion | ResourceLocation |   |
| Service consommé | ServiceInfo | ConsumedService |   |
| ID de l’instance | Composant | InstanceId |   |
| ServiceInfo1 | ServiceInfo1 | ServiceInfo1 |   |
| ServiceInfo2 | ServiceInfo2 | ServiceInfo2 |   |
| AdditionalInfo | AdditionalInfo | AdditionalInfo |   |
| Balises | Balises | Balises |   |
| Store Service Identifier   | OrderNumber | StoreServiceIdentifier   |   |
| Department Name | DepartmentName | DepartmentName |   |
| Cost Center | CostCenter | CostCenter |   |
| Unité de mesure | UnitOfMeasure | UnitOfMeasure | Exemples de valeurs : Hours (Heures), GB (Go), Events (Événements), Pushes (Push), Unit (Unité), Unit Hours (Heures d’unité), MB (Mo), Daily Units (Unités journalières) |
| ResourceGroup | ResourceGroup | ResourceGroup |   |

#### <a name="azure-marketplace-report"></a>Rapport de la Place de marché Azure

| Nom de colonne CSV | Nom de colonne JSON | Nouvelle colonne JSON |
| --- | --- | --- |
| AccountOwnerId | AccountOwnerId | AccountOwnerId |
| Nom du compte | AccountName | AccountName |
| SubscriptionId | SubscriptionId | SubscriptionId |
| SubscriptionGuid | SubscriptionGuid | SubscriptionGuid |
| Nom d'abonnement | SubscriptionName |  SubscriptionName |
| Date | BillingCycle |  Date (chaîne de date uniquement, pas d’horodatage)
| Month | Month |  Month |
| jour | jour |  jour |
| Year | Year |  Year |
| ID du compteur | MeterResourceId |  ID du compteur |
| Nom de l’éditeur | PublisherFriendlyName |  PublisherName |
| Nom de l’offre | OfferFriendlyName |  OfferName |
| Nom du plan | PlanFriendlyName |  PlanName |
| Quantité consommée | BilledQty |  ConsumedQuantity |
| ResourceRate | ResourceRate | ResourceRate |
| ExtendedCost | ExtendedCost | ExtendedCost |
| Unité de mesure | UnitOfMeasure | UnitOfMeasure |
| ID de l’instance | InstanceId | InstanceId |
| Informations supplémentaires | AdditionalInfo | AdditionalInfo |
| Balises | Balises | Balises |
| Order Number | OrderNumber | OrderNumber |
| Department Name | DepartmentNames | DepartmentName |
| Cost Center | CostCenters |  CostCenter |
| Groupe de ressources | ResourceGroup |  ResourceGroup |

#### <a name="price-sheet"></a>Price sheet

| Nom de colonne CSV | Nom de colonne JSON | Commentaire |
| --- | --- | --- |
| Service | Service |  Pas de changement de prix |
| Unité de mesure | UnitOfMeasure |   |
| Overage Part Number | ConsumptionPartNumber |   |
| Overage Unit Price | ConsumptionPrice |   |
| Code devise | CurrencyCode |     |

### <a name="common-api-issues"></a>Problèmes courants liés aux API

Quand vous utilisez les API REST Entreprise Azure, vous pouvez rencontrer les problèmes courants suivants.

Vous essayez d’utiliser une clé API qui n’a pas le type d’autorisation approprié. Les clés API sont générées par ces rôles :

- Administrateur d’entreprise
- Administrateur de service (DA)
- Propriétaire du compte (AO)

Une clé générée par l’administrateur d’entreprise (EA) Azure donne accès à toutes les informations relatives à cette inscription. Un administrateur EA qui a des droits en lecture seule ne peut pas générer de clé API.

Une clé générée par un administrateur DA ou AO ne permet pas d’accéder aux informations relatives aux soldes, aux frais et à la grille tarifaire.

Une clé API expire au bout de six mois. Quand la clé a expiré, vous devez la regénérer.

Si vous recevez une erreur de délai d’expiration, vous pouvez la résoudre en augmentant la limite du délai d’expiration.

Vous pouvez recevoir une erreur d’expiration 401 (non autorisée). L’erreur est généralement due à une clé expirée. Si la clé a expiré, vous devez la regénérer.

Vous pouvez recevoir les erreurs 400 et 404 (non disponibles) retournées par un appel d’API quand aucune donnée n’est disponible pour la plage de dates sélectionnée. Par exemple, cette erreur peut se produire si un transfert d’inscription a été lancé récemment. Les données pour une plage de dates spécifique se trouvent maintenant dans une nouvelle inscription. L’erreur peut aussi se produire si vous utilisez un nouveau numéro d’inscription pour récupérer des données contenues dans une ancienne inscription.

## <a name="next-steps"></a>Étapes suivantes

- Les administrateurs du portail Azure EA doivent lire la [documentation consacrée à l’administration du portail Azure EA](ea-portal-administration.md) pour découvrir les tâches d’administration courantes.
- Si vous avez besoin d’aide pour résoudre des problèmes rencontrés avec le portail Azure EA, consultez [Résoudre les problèmes d’accès au portail Azure EA](ea-portal-troubleshoot.md).