---
title: API de facturation Azure Enterprise
description: En savoir plus sur les API de création de rapports qui permettent aux clients d’Azure Enterprise d’extraire leurs données de consommation par programme.
author: mumami
tags: billing
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: reference
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: b78b046e2991fa4ebfb575e4f6a1cc900df0d298
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686902"
---
# <a name="overview-of-reporting-apis-for-enterprise-customers"></a>Vue d’ensemble des API de création de rapports pour les clients Enterprise

> [!Note]
> Microsoft ne met plus à jour les API Création de rapports – Facturation Azure. Utilisez à la place les API [Consommation Azure](/rest/api/consumption).

Les API de création de rapports permettent aux clients Azure Enterprise d’extraire leurs données de consommation et de facturation par programme pour les transférer vers les outils d’analyse de données de leur choix. Les clients Entreprise ont signé un [contrat Entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/) avec Azure pour fixer des prix négociés et bénéficier de tarifs personnalisés pour les ressources Azure.

Tous les paramètres de date et d’heure nécessaires aux API doivent être représentés sous forme de valeurs UTC (temps universel coordonné) combinées. Les valeurs retournées par les API s’affichent au format UTC.

## <a name="enabling-data-access-to-the-api"></a>Autoriser l’API à accéder aux données
* **Générer ou récupérer la clé API** : connectez-vous au portail Entreprise et accédez à Rapports > Télécharger l’utilisation > Clé d’accès API pour générer ou récupérer la clé API.
* **Transmission des clés dans l’API** : vous devez transmettre la clé API pour chaque appel à des fins d’authentification et d’autorisation. Les en-têtes HTTP doivent contenir la propriété suivante

|Clé d’en-tête de demande | Valeur|
|-|-|
|Autorisation| Indiquez la valeur dans ce format : **porteur {API_CLÉ}** <br/> Exemple : porteur eyr... 09|

## <a name="consumption-based-apis"></a>API basées sur la consommation
Un point de terminaison Swagger est disponible [ici](https://consumption.azure.com/swagger/ui/index) pour les API décrites ci-dessous. Il doit faciliter l’introspection de l’API et la capacité à générer des Kits de développement logiciel (SDK) clients en utilisant [AutoRest](https://github.com/Azure/AutoRest) ou [CodeGen Swagger](https://swagger.io/swagger-codegen/). À compter du 1er mai 2014, ces données sont disponibles via cette API.

* **Balance and Summary** (Solde et résumé) : l’[API Balance and Summary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) permet un résumé mensuel des informations sur les soldes, nouveaux achats, frais de service de la Place de marché Azure, ajustements et dépassement des frais.

* **Usage Details** (Détails d’utilisation) : l’[API Usage Details](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail) offre une analyse quotidienne des quantités consommées et des frais estimés pour un abonnement. Le résultat inclut également des informations sur les instances, les compteurs et les services. L’API peut être interrogée par période de facturation ou en indiquant une date de début et de fin.

* **Marketplace Store Charge** (Frais de stockage de la Place de marché) : l’[API Marketplace Store Charge](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) renvoie l’analyse détaillée des frais de Place de marché basée sur l’utilisation par jour pour la période de facturation ou les dates de début et de fin indiquées (les frais ponctuels ne sont pas inclus).

* **Price Sheet** (Grille tarifaire) : l’[API Price Sheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) fournit les tarifs applicables pour chaque compteur selon l’abonnement et la période de facturation donnés.

* **Informations détaillées sur les instances réservées** - L'[API Utilisation des instances réservées](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) renvoie l'utilisation des instances réservées achetées. L'[API Frais des instances réservées](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) affiche les transactions de facturation effectuées.

## <a name="data-freshness"></a>Actualisation des données
Les Etags s’afficheront dans la réponse de toutes les API ci-dessus. Une modification de l’ETag indique que les données ont été actualisées.  Dans les appels suivants à la même API utilisant les mêmes paramètres, passez l’ETag capturée avec la clé « If-None-Match » dans l’en-tête de requête http. Le code d’état de réponse est « NotModified » si les données n’a plus été actualisées et qu’aucune donnée n’est retournée. L’API retourne le jeu de données complet pour la période demandée chaque fois qu’il y a une modification de l’ETag.

## <a name="helper-apis"></a>API d’assistance
 **List Billing Periods** (Liste des périodes de facturation) : l’[API List Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) renvoie une liste des périodes de facturation contenant les données de consommation pour l’abonnement indiqué, par ordre chronologique inverse. Chaque période contient une propriété qui pointe vers l’itinéraire d’API pour les quatre ensembles de données : BalanceSummary (Solde et résumé), UsageDetails (Détails d’utilisation), Marketplace Charges (Frais de la Place de marché) et PriceSheet (Grille tarifaire).


## <a name="api-response-codes"></a>Codes de réponse HTTP   
|Code du statut de réponse|Message|Description|
|-|-|-|
|200| OK|Aucune erreur|
|400| Demande incorrecte| Paramètres non valides : plages de dates, nombres de Contrats Entreprise (EA), etc.|
|401| Non autorisé| Clé API introuvable, non valide, expirée, etc.|
|404| Non disponible| Point de terminaison de rapport introuvable|
|429 | TooManyRequests | La demande a été limitée. Réessayez après avoir attendu le temps spécifié dans l’en-tête <code>x-ms-ratelimit-microsoft.consumption-retry-after</code>.|
|500| Erreur de serveur| Erreur inattendue lors du traitement de la requête|
| 503 | ServiceUnavailable | Le service est temporairement indisponible. Réessayez après avoir attendu le temps spécifié dans l’en-tête <code>Retry-After</code>.|
