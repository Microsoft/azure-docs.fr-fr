---
title: Comprendre vos factures Azure Cosmos DB
description: Cet article fournit quelques exemples pour vous aider à mieux comprendre votre facture Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/04/2020
ms.reviewer: sngun
ms.openlocfilehash: b51be165d3d05ae753c7e0e5536a157fbbacf1ab
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938628"
---
# <a name="understand-your-azure-cosmos-db-bill"></a>Compréhension de vos factures Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB est un service de base de données natif Cloud entièrement géré qui simplifie la facturation en facturant uniquement vos opérations de base de données et le stockage consommé. Il ne comporte aucuns frais de licence supplémentaires, ni coûts matériels, frais de fonctionnement ou frais d’équipement, contrairement aux autres solutions IaaS ou hébergées en local. Grâce aux fonctionnalités multi-région d’Azure Cosmos DB, le service de base de données réduit considérablement les coûts par rapport aux solutions IaaS ou locales existantes.

- **Opérations de base de données** : la façon dont vous êtes facturé pour vos opérations de base de données dépend du type de compte Azure Cosmos que vous utilisez.

  - **Débit approvisionné** : vous êtes facturé à l’heure pour le débit maximal approvisionné pour une heure donnée, par incréments de 100 RU/s.
  - **Serverless** : vous êtes facturé à l’heure pour la quantité totale d’unités de requête consommées par vos opérations de base de données.

- **Stockage** : un tarif fixe vous est facturé pour la quantité totale de stockage (exprimée en Go) que vos données et index ont utilisée pendant une heure donnée.

Pour accéder aux informations les plus récentes sur la tarification, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).

Cet article s’appuie sur des exemples pour vous aider à comprendre les détails figurant sur la facture mensuelle. Les chiffres indiqués dans les exemples peuvent être différents si vos conteneurs Azure Cosmos fournissent une autre quantité de débit, s’ils s’étendent sur plusieurs régions ou s’ils sont exécutés sur une période supérieure à un mois. Tous les exemples présentés dans cet article calculent la facturation en fonction des informations tarifaires fournies dans la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).

> [!NOTE]
> La facturation concerne une partie d’une heure d’horloge, et non pas une durée de 60 minutes. Tous les exemples présentés dans ce document se basent sur le prix d’un compte Azure Cosmos déployé dans une région non gouvernementale aux États-Unis. Le tarif et le calcul varient en fonction de la région. Pour connaître les dernières informations tarifaires, consultez la [page des tarifs Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

## <a name="billing-examples"></a>Exemples de facturation

### <a name="billing-example---provisioned-throughput-on-a-container-full-month"></a>Exemple de facturation : débit approvisionné sur un conteneur (mois complet)

* Supposons que vous configuriez un débit de 1 000 RU/seconde sur un conteneur et que ce débit est maintenu pendant 24 heures x 30 jours sur l’ensemble du mois, soit 720 heures au total.  

* 1 000 RU/s représentent 10 unités de 100 RU/seconde par heure pour chaque heure d’existence des conteneurs (autrement dit, 1 000/100 = 10). 

* En multipliant ces 10 unités par heure par un coût de 0,008 $ (pour 100 RU/s par heure), on obtient 0,08 $ par heure. 

* En multipliant ces 0,08 $ par heure par le nombre d’heures au cours du mois, on obtient 0,08 $ x 24 heures x 30 jours = 57,60 $ pour le mois.  

* La facture totale mensuelle indique 7 200 unités (de 100 RU), qui vous coûteront 57,60 $.

### <a name="billing-example---provisioned-throughput-on-a-container-partial-month"></a>Exemple de facturation : débit approvisionné sur un conteneur (mois partiel)

* Supposons que vous créez un conteneur avec un débit approvisionné de 2 500 RU/s. Le conteneur est utilisé pendant 24 heures au cours du mois (par exemple, vous le supprimez 24 heures après l’avoir créé).  

* La facture indiquera 600 unités (2 500 RU/s / 100 RU/s/unité x 24 heures). Le coût sera de 4,80 $ (600 unités x 0,008 $/unité).

* La facture totale pour le mois sera de 4,80 $.

### <a name="billing-example---serverless-container"></a>Exemple de facturation : conteneur serverless

* Supposons que nous créons un conteneur serverless. 

* En un mois, nous émettons des demandes de base de données qui consomment un total de 500 000 unités de requête. Le coût sera de 0,125 USD (500 000 * 0,25 USD/million).

* La facture mensuelle totale sera de 0,125 USD.

### <a name="billing-rate-if-storage-size-changes"></a>Taux de facturation en cas de changement de taille de stockage

La capacité de stockage est facturée en unités de la quantité horaire maximale de données stockées (exprimée en Go) sur une période d’un mois. Par exemple, si vous avez utilisé 100 Go de stockage au cours de la première moitié du mois, et 50 Go lors de la seconde moitié, vous êtes facturé pour l’équivalent de 75 Go de stockage durant ce mois.

### <a name="billing-rate-when-container-or-a-set-of-containers-are-active-for-less-than-an-hour"></a>Taux de facturation appliqué lorsqu’un conteneur ou un ensemble de conteneurs est actif pendant moins d’une heure

Vous êtes facturé à un taux fixe pour chaque heure d’existence du conteneur ou de la base de données, indépendamment de l’utilisation, et même si le conteneur ou la base de données sont actifs pendant moins d’une heure. Par exemple, si vous créez un conteneur ou une base de données, puis les supprimez au bout de 5 minutes, vous serez facturé pour 1 heure.

### <a name="billing-rate-when-provisioned-throughput-on-a-container-or-database-scales-updown"></a>Taux de facturation appliqué en cas d’augmentation ou de diminution du débit approvisionné sur un conteneur ou une base de données

Si vous augmentez le débit provisionné à 9h30 en le faisant passer de 400 unités de requête/seconde à 1 000 unités de requête/seconde, puis que vous réduisez ensuite le débit à 10h45 à 400 unités de requête/seconde, vous payez deux heures de 1 000 unités de requête/seconde. 

Si vous augmentez le débit provisionné pour un conteneur ou un ensemble de conteneurs à 9h30 en le faisant passer de 100 000 unités de requête/seconde à 200 000 unités de requête/seconde, puis que vous réduisez ensuite le débit à 10h45 à 100 000 unités de requête/seconde, vous payez deux heures de 200 000 unités de requête/seconde.

### <a name="billing-example-multiple-containers-each-with-dedicated-provisioned-throughput-mode"></a>Exemple de facturation : plusieurs conteneurs, chacun avec un mode de débit approvisionné dédié

* Si vous créez un compte Azure Cosmos dans la région USA Est 2 avec deux conteneurs dotés de débits approvisionnés respectifs 500 RU/s et de 700 RU/s, vous obtenez un débit approvisionné total de 1 200 RU/s.  

* Vous serez facturé 1 200/100 x 0,008 $ = 0,096 $/ heure. 

* Si vos besoins en débit ont changé et que vous avez augmenté la capacité de chaque conteneur de 500 RU/s tout en créant un conteneur illimité utilisant 20 000 RU/s, votre capacité approvisionnée globale est de 22 200 RU/s (1 000 RU/s + 1 200 RU/s + 20 000 RU/s).  

* Vous êtes alors facturé comme suit :  0,008 $ x 222 = 1,776 $/heure. 

* Au cours d’un mois comptant 720 heures (24 heures x 30 jours), si le débit provisionné pendant 500 heures est de 1 200 unités de requête/seconde et que pour les 220 heures restantes, le débit provisionné est de 22 200 unités de requête/seconde, votre facture mensuelle est la suivante : 500 x 0,096 $/heure + 220 x 1,776 $/heure = 438,72 /mois.

:::image type="content" source="./media/understand-your-bill/bill-example1.png" alt-text="Exemple de facturation d’un débit dédié":::

### <a name="billing-example-containers-with-shared-provisioned-throughput-mode"></a>Exemple de facturation : conteneurs en mode de débit (approvisionné) partagé

* Si vous créez un compte Azure Cosmos dans la région USA Est 2 avec deux bases de données Azure Cosmos (comprenant un ensemble de conteneurs qui partage le débit au niveau de la base de données) et un débit approvisionné de 50 000 RU/s et 70 000 RU/s, respectivement, le débit provisionné serait de 120 000 RU/s au total.  

* Vous serez alors facturé 1 200 x 0,008 $ = 9,60 $/ heure. 

* Si vos besoins en débit ont changé et que vous avez augmenté le débit approvisionné de chaque base de données de 10 000 RU/s pour chaque base de données, puis que vous ajoutez un nouveau conteneur à la première base de données en utilisant un mode de débit dédié de 15 000 RU/s sur votre base de données à débit partagé, votre capacité approvisionnée globale serait de 155 000 RU/s (60 000 RU/s + 80 000 RU/s + 15 000 RU/s).  

* Vous serez alors facturé comme suit : 1 550 x 0,008 $ = 12,40 $/heure.  

* Au cours d’un mois comptant 720 heures, si le débit provisionné pendant 300 heures est de 120 000 unités de requête/seconde et que pour les 420 heures restantes, le débit provisionné est de 155 000 unités de requête/seconde, votre facture mensuelle est la suivante : 300 x 9,60 $/heure + 420 x 12,40 $/heure = 2 880 $ + 5 208 $ = 8 088 $ /mois. 

:::image type="content" source="./media/understand-your-bill/bill-example2.png" alt-text="Exemple de facturation d’un débit partagé":::

## <a name="billing-examples-with-geo-replication"></a>Exemples de facturation avec géoréplication  

Vous pouvez ajouter des régions Azure à votre compte de base de données Azure Cosmos n’importe où dans le monde, ou en supprimer, et ceci à tout moment. La réservation du débit que vous avez configuré pour les différentes bases de données et conteneurs Azure Cosmos est garantie dans chacune des régions Azure associées à votre compte de base de données Azure Cosmos. Si la somme de débit approvisionné (RU/s) configuré sur l’ensemble des bases de données et des conteneurs de votre compte Azure Cosmos Database (approvisionné par heure) est T et que le nombre de régions Azure associées à votre compte de base de données est N, alors le débit total approvisionné pour une heure donnée et pour votre compte Azure Cosmos Database est égal à T x N RU/s. Le débit approvisionné (région d’écriture unique) coûte 0,008 $/heure par 100 RU/s et le débit approvisionné avec plusieurs régions accessibles en écriture (configuration d’écritures multirégions) coûte 0,016 $/par heure par 100 RU/s (voir la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/)). Azure Cosmos DB vous permet de lire vos données de n’importe quelle région, que vous utilisiez une seule ou plusieurs région(s) d’écriture.

### <a name="billing-example-multi-region-azure-cosmos-account-single-region-writes"></a>Exemple de facturation : compte Azure Cosmos avec plusieurs régions et une seule région d’écriture

Supposons que vous disposez d’un conteneur Azure Cosmos dans la région USA Ouest. Le conteneur est créé avec un débit de 10 000 RU/s et vous stockez 1 To de données ce mois-ci. Supposons que vous ajoutez à votre compte Azure Cosmos trois régions (USA Est, Europe Nord et Asie Est) et que chaque région dispose d’un stockage et d’un débit identiques. Votre facture mensuelle totale est (en supposant un mois de 30 jours). Votre facture se présente comme suit : 

|**Item** |**Utilisation (mois)** |**Tarif** |**Coût mensuel** |
|---------|---------|---------|-------|
|Facture de débit pour le conteneur de la région USA Ouest      | 10 000 RU/s x 24 x 30    |0,008 $ pour 100 RU/s par heure   |576 $|
|Facture de débit pour 3 régions supplémentaires (USA Est, Europe Nord et Asie Est)       | 3 x 10 000 RU/s x 24 x 30    |0,008 $ pour 100 RU/s par heure  |1 728 $|
|Facture de stockage pour le conteneur de la région USA Ouest      | 250 Go    |0,25 $/Go  |62,50 $|
|Facture de stockage pour 3 régions supplémentaires (USA Est, Europe Nord et Asie Est)      | 3 x 250 Go    |0,25 $/Go  |187,50 $|
|**Total**     |     |  |**2 554**|

*Supposons également que vous faites sortir 100 Go de données chaque mois du conteneur dans la région USA Ouest afin de répliquer les données dans les régions USA Est, Europe Nord et Asie Est. Vous êtes facturé pour les sorties en fonction du tarif des transferts de données.*

### <a name="billing-example-multi-region-azure-cosmos-account-multi-region-writes"></a>Exemple de facturation : compte Azure Cosmos avec plusieurs régions et plusieurs régions d’écriture

Supposons que vous créez un conteneur Azure Cosmos dans la région USA Ouest. Le conteneur est créé avec un débit de 10 000 RU/s et vous stockez 1 To de données ce mois-ci. Supposons que vous ajoutez trois régions (USA Est, Europe Nord et Asie Est), que chaque région dispose d’un stockage et d’un débit identiques et que vous souhaitez pouvoir écrire dans les conteneurs de toutes les régions associées à votre compte Azure Cosmos. Votre facture mensuelle totale est (en supposant un mois de 30 jours) :

|**Item** |**Utilisation (mois)**|**Tarif** |**Coût mensuel** |
|---------|---------|---------|-------|
|Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)       | 10 000 RU/s x 24 x 30    |0,016 $ pour 100 RU/s par heure    |1 152 $ |
|Facture de débit pour 3 régions supplémentaires : USA Est, Europe Nord et Asie Est (écriture dans toutes les régions)        | 3 x 10 000 RU/s x 24 x 30    |0,016 $ pour 100 RU/s par heure   |3 456 $ |
|Facture de stockage pour le conteneur de la région USA Ouest      | 250 Go    |0,25 $/Go  |62,50 $|
|Facture de stockage pour 3 régions supplémentaires (USA Est, Europe Nord et Asie Est)      | 3 x 250 Go    |0,25 $/Go  |187,50 $|
|**Total**     |     |  |**6 010 $**|

*Supposons également que vous faites sortir 100 Go de données chaque mois du conteneur dans la région USA Ouest afin de répliquer les données dans les régions USA Est, Europe Nord et Asie Est. Vous êtes facturé pour les sorties en fonction du tarif des transferts de données.*

### <a name="billing-example-azure-cosmos-account-with-multi-region-writes-database-level-throughput-including-dedicated-throughput-mode-for-some-containers"></a>Exemple de facturation : Compte Azure Cosmos avec un débit d’écritures multirégions au niveau de la base de données et mode de débit dédié pour certains conteneurs

Prenons l’exemple suivant, où nous avons un compte Azure Cosmos multirégion où toutes les régions sont accessibles en écriture (configuration de plusieurs régions d’écriture). Par souci de simplicité, supposons que la taille du stockage reste constante et qu’elle n’évolue pas. Nous allons ici l’omettre pour simplifier notre exemple. Le débit approvisionné au cours du mois évolue comme suit (en supposant 30 jours ou 720 heures) : 

[0-100 heures] :  

* Nous avons créé un compte Azure Cosmos avec trois régions (USA Ouest, USA Est et Europe Nord), où toutes les régions sont accessibles en écriture 

* Nous avons créé une base de données (D1) avec un débit partagé de 10 000 RU/s 

* Nous avons créé une base de données (D2) avec un débit partagé de 30 000 RU/s et  

* Nous avons créé un conteneur (C1) avec un débit partagé de 20 000 RU/s 

[101-200 heures] :  

* Nous avons augmenté le débit de la base de données (D1) à 50 000 RU/s 

* Nous avons augmenté le débit de la base de données (D2) à 70 000 RU/s  

* Nous avons supprimé le conteneur (C1)  

[201-300 heures] :  

* Nous avons de nouveau créé un conteneur (C1) avec un débit partagé de 20 000 RU/s 

[301-400 heures] :  

* Nous avons supprimé une des régions du compte Azure Cosmos (le nombre de régions accessibles en écriture est maintenant de 2) 

* Nous avons réduit le débit de la base de données (D1) à 10 000 RU/s 

* Nous avons augmenté le débit de la base de données (D2) à 80 000 RU/s  

* Nous avons de nouveau supprimé le conteneur (C1) 

[401-500 heures] :  

* Nous avons réduit le débit de la base de données (D2) à 10 000 RU/s  

* Nous avons de nouveau créé un conteneur (C1) avec un débit partagé de 20 000 RU/s 

[501-700 heures] :  

* Nous avons augmenté le débit de la base de données (D1) à 20 000 RU/s  

* Nous avons augmenté le débit de la base de données (D2) à 100 000 RU/s  

* Nous avons de nouveau supprimé le conteneur (C1)  

[701-720 heures] :  

* Nous avons diminué le débit de la base de données (D2) à 50 000 RU/s  

Les modifications apportées au débit total approvisionné pendant 720 heures pour le mois sont représentées visuellement dans la figure ci-dessous : 

:::image type="content" source="./media/understand-your-bill/bill-example3.png" alt-text="Exemple concret":::

La facture totale mensuelle sera (en supposant une durée de 30 jours/720 heures par mois) sera calculée comme suit :

|**Heures**  |**RU/s** |**Item** |**Utilisation (horaire)** |**Coût** |
|---------|---------|---------|-------|-------|
|[0-100] |D1 : 10 000 <br/>D2 : 30 000 <br/>C1 : 20 000 |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  | `D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br/>`D2: 30 K RU/sec/100 * $0.016 * 100 hours = $480` <br/>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |960 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(2 + 1) * (60 K RU/sec /100 * $0.016) * 100 hours = $2,880`  |2 880 $  |
|[101-200] |D1 : 50 000 <br/>D2 : 70 000 <br/>C1 : -- |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 50 K RU/sec/100 * $0.016 * 100 hours = $800` <br/>`D2: 70 K RU/sec/100 * $0.016 * 100 hours = $1,120` |1 920 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(2 + 1) * (120 K RU/sec /100 * $0.016) * 100 hours = $5,760`  |5 760 $  |
|[201-300]  |D1 : 50 000 <br/>D2 : 70 000 <br/>C1 : 20 000 |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 50 K RU/sec/100 * $0.016 * 100 hours = $800` <br/>`D2: 70 K RU/sec/100 * $0.016 * 100 hours = $1,120` <br/>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |2 240 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(2 + 1) * (140 K RU/sec /100 * $0.016-) * 100 hours = $6,720` |6 720 |
|[301-400] |D1 : 10 000 <br/>D2 : 80 000 <br/>C1 : -- |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br/>`D2: 80 K RU/sec/100 * $0.016 * 100 hours = $1,280`  |1 440 $   |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(1 + 1) * (90 K RU/sec /100 * $0.016) * 100 hours = $2,880`  |2 880 $  |
|[401-500] |D1 : 10 000 <br>D2 : 10 000 <br>C1 : 20 000 |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br>`D2: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |640 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(1 + 1) * (40 K RU/sec /100 * $0.016) * 100 hours = $1,280`  |1 280 $  |
|[501-700] |D1 : 20 000 <br>D2 : 100 000 <br>C1 : -- |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 20 K RU/sec/100 * $0.016 * 200 hours = $640` <br>`D2: 100 K RU/sec/100 * $0.016 * 200 hours = $3,200` |3 840 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(1 + 1) * (120 K RU/sec /100 * $0.016) * 200 hours = $1,280`  |7 680 $  |
|[701-720] |D1 : 20 000 <br/>D2 : 50 000 <br/>C1 : -- |Facture de débit pour le conteneur de la région USA Ouest (écriture dans toutes les régions)  |`D1: 20 K RU/sec/100 *$0.016 * 20 hours = $64` <br/>`D2: 50 K RU/sec/100 *$0.016 * 20 hours = $160` |224 $  |
| | |Facture de débit pour 2 régions supplémentaires : USA Est, Europe Nord (toutes les régions sont accessibles en écriture)  |`(1 + 1) * (70 K RU/sec /100 * $0.016) * 20 hours = $448`  |224 $  |
|| |**Coût mensuel total**  | |**38 688 $**   |

## <a name="billing-examples-with-free-tier-accounts"></a>Exemples de facturation avec des comptes de niveau gratuit
Avec le niveau gratuit Azure Cosmos DB, vous recevez gratuitement les premiers 400 RU/s et 5 Go de stockage dans votre compte, appliqués au niveau du compte. Les RU/s et le stockage au-delà de 400 RU/s et de 5 Go sont facturés selon les tarifs standard indiqués dans la page des tarifs. Sur la facture, vous ne voyez pas de coût ni de ligne pour les 400 RU/s et les 5 Go gratuits, mais seulement les RU/s et le stockage au-delà de ce qui est couvert par le niveau gratuit. Les 400 RU/s s’appliquent à tous les types de RU/s – débit provisionné, mise à l’échelle automatique et écritures multirégions.  

### <a name="billing-example---container-or-database-with-provisioned-throughput"></a>Exemple de facturation – conteneur ou base de données avec débit provisionné
- Supposons que nous créons une base de données ou un conteneur dans un compte de niveau gratuit avec 400 RU/s et 5 Go de stockage.
- Votre facture n’affiche aucuns frais pour cette ressource. Votre coût horaire et mensuel est de 0 USD.
- À présent, supposons que nous ajoutons dans le même compte une autre base de données ou un autre conteneur avec 1 000 RU/s et 10 Go de stockage.
- Votre facture affiche maintenant des frais pour les 1 000 RU/s et 10 Go de stockage. 

### <a name="billing-example---container-with-autoscale-throughput"></a>Exemple de facturation : conteneur avec débit en mode de mise à l’échelle automatique
- Supposons que nous créons dans un compte de niveau gratuit un conteneur avec la mise à l’échelle automatique activée, avec un maximum de 4 000 RU/s. Cette ressource effectue automatiquement une mise à l’échelle entre 400 RU/s et 4 000 RU/s. 
- Supposons que pendant les heures 1 à 10 la ressource est au minimum de 400 RU/s. Pendant l’heure 11, la ressource effectue un scale-up jusqu’à 1 000 RU/s, puis revient à 400 RU/s dans l’heure.
- Pour les heures 1 à 10, il vous est facturé 0 USD pour le débit, car les 400 RU/s sont couvertes par le niveau gratuit. 
- Pour l’heure 11, il vous est facturé 1 000 RU/s – 400 RU/s = 600 RU/s, car il s’agit du débit le plus élevé dans l’heure. Cela représente 6 unités de 100 RU/s pour l’heure, de sorte que le coût total du débit pour l’heure est de 6 unités * 0,012 USD = 0,072 USD. 
- Tout stockage au-delà des 5 premiers Go est facturé au tarif de stockage normal. 

### <a name="billing-example---multi-region-single-write-region-account"></a>Exemple de facturation – compte multirégion, avec une seule région d’écriture
- Supposons que nous créons une base de données ou un conteneur dans un compte de niveau gratuit avec 1 200 RU/s et 10 Go de stockage. Nous répliquons le compte dans 3 régions et nous disposons d’un compte monorégion d’écriture.
- Au total, sans niveau gratuit, il nous serait facturé 3 * 1 200 RU/s = 3 600 RU/s et 3 * 10 Go = 30 Go de stockage.
- Avec la remise du niveau gratuit, après avoir enlevé 400 RU/s et 5 Go de stockage, il nous est facturé un débit provisionné de 3 200 RU/s (32 unités) au tarif de la région d’écriture unique et 25 Go de stockage.
- Le coût mensuel pour les RU/s est le suivant : 32 unités * 0,008 USD * 24 heures * 31 jours = 190,46 USD. Le coût mensuel du stockage est le suivant : 25 Go * 0,25 USD/Go = 6,25 USD. Le coût total est de 190,46 USD + 6,25 USD = 196,71 USD.
- Remarque : Si le prix unitaire pour les RU/s ou le stockage diffère dans les régions, les 400 RU/s et 5 Go du niveau gratuit reflètent les tarifs de la région dans laquelle le compte a été créé.

### <a name="billing-example---multi-region-account-with-multiple-write-regions"></a>Exemple de facturation – multirégion, compte avec plusieurs régions d’écriture

Cet exemple reflète les [tarifs pour écritures multirégions](https://azure.microsoft.com/pricing/details/cosmos-db/) pour les comptes créés après le 1er décembre 2019. 
- Supposons que nous créons une base de données ou un conteneur dans un compte de niveau gratuit avec 1 200 RU/s et 10 Go de stockage. Nous répliquons le compte dans 3 régions et nous disposons d’un compte multirégion d’écriture. 
- Au total, sans niveau gratuit, il nous serait facturé 3 * 1 200 RU/s = 3 600 RU/s et 3 * 10 Go = 30 Go de stockage.
- Avec la remise du niveau gratuit, après avoir enlevé 400 RU/s et 5 Go de stockage, il nous est facturé un débit provisionné de 3200 RU/s (32 unités) au tarif des multiples régions d’écriture et 25 Go de stockage.
- Le coût mensuel pour les RU/s est le suivant : 32 unités * 0,016 USD * 24 heures * 31 jours = 380,93 USD. Le coût mensuel du stockage est le suivant : 25 Go * 0,25 USD/Go = 6,25 USD. Le coût total est de 380,93 USD + 6,25 USD = 387,18 USD.

## <a name="proactively-estimating-your-monthly-bill"></a>Estimation proactive de votre facture mensuelle  

Prenons un autre exemple, dans lequel vous souhaitez anticiper le montant de votre facture avant la fin du mois. Vous pouvez estimer votre facture de la manière suivante :

**Coût de stockage**

* Taille moyenne d’enregistrement (en Ko) = 1 
* Nombre d’enregistrements = 100 000 000 
* Stockage total (en Go) = 100 
* Coût mensuel par Go = 0,25 $ 
* Coût mensuel de stockage attendu = 25,00 $ 

**Coût du débit**

|Type d'opération| Requêtes/s| Avg. RU/requête| RU nécessaires|
|----|----|----|----|
|Write| 100 | 5 | 500|
|Lire| 400| 1| 400|

RU total/s : 500 + 400 = 900 ; coût horaire : 900/100 x 0,008 $ = coût mensuel attendu de 0,072 $ pour le débit (en supposant une durée de 31 jours) : 0,072 $ x 24 x 31 = 53,57 $

**Coût mensuel total**

Coût mensuel total = coût mensuel de stockage + coût mensuel du débit pour un coût mensuel total du débit = 25,00 $ + 53,57 $ = 78,57 $

*La tarification peut varier selon la région. Pour connaître les tarifs applicables, consultez la [Page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).*

## <a name="billing-with-azure-cosmos-db-reserved-capacity"></a>Facturation avec une capacité réservée Azure Cosmos DB

La capacité réservée Azure Cosmos DB vous permet d’acheter à l’avance du débit approvisionné (une capacité réservée ou une réservation) qui peut être appliqué à tous les conteneurs et toutes les bases de données Azure Cosmos (pour n’importe quelle API ou n’importe quel modèle de données) de toutes les régions Azure. Comme le prix du débit approvisionné varie d’une région à l’autre, il est utile de considérer la capacité de réserve comme un crédit monétaire que vous avez acheté à prix réduit et qui peut être déduit pour le débit approvisionné du prix respectif de chaque région. Par exemple, supposons que vous disposez d’un compte Azure Cosmos avec un seul conteneur approvisionné avec 50 000 RU/s et deux régions répliquées à l’échelle mondiale : USA Est et Japon Est. Si vous choisissez l’option de paiement à l’utilisation, vous devrez alors payer :  

* dans la région USA Est : 50 000 RU/s au tarif de 0,008 $ pour 100 RU/s dans cette région 

* dans la région Japon Est : 50 000 RU/s au tarif de 0,009 $ pour 100 RU/s dans cette région

Votre facture totale (sans capacité réservée) serait (en supposant une durée de 30 jours ou 720 heures) : 

|**Région**| **Tarif horaire pour 100 RU/s**|**Unités (RU/s)**|**Montant facturé (horaire)**| **Montant facturé (mensuel)**|
|----|----|----|----|----|
|USA Est|0,008 $ |50 000|4 $|2 880 $ |
|Japon Est|0,009 $ |50 000| 4,50 $ |3 240 $ |
|Total|||8,50 $|6 120 $ |

Imaginons que vous avez acheté une capacité de réserve à la place. Vous pouvez acheter une capacité réservée pour 100 000 RU/s au prix de 56,064 $ par an (avec une remise de 20 %) ou de 6,40 $ par heure. Pour connaître les tarifs appliqués pour la capacité réservée, consultez la [Page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/)).  

* Coût du débit (paiement à l’utilisation) : 100 000 RU/s/100 x 0,008 $/heure x 8 760 heures par an = 70 080 $ 

* Coût du débit (avec une capacité réservée) de 70 080 $ avec une remise de 20 % = 56 064 $ 

Vous avez en réalité acheté un crédit valant 8 $ par heure pour 100 000 RU/s, selon le prix catalogue pour la région USA Est, au prix de 6,40 $ par heure. Vous pouvez ensuite prélever de cette réservation de débit prépayée, sur une base horaire pour la capacité de débit disponible dans n’importe quelle région Azure mondiale aux prix de liste régionaux respectifs fixés pour votre abonnement. Dans cet exemple, où vous prévoyez 50 000 RU/s chacun dans la région USA Est et dans la région Japon Est, vous pourrez tirer 8,00 $ de débit approvisionné par heure, et vous serez sur-facturé de 0,50 $ par heure (ou 360 $ par mois). 

|**Région**| **Tarif horaire pour 100 RU/s**|**Unités (RU/s)**| **Montant facturé (horaire)**| **Montant facturé (mensuel)**|
|----|----|----|----|----|
|USA Est|0,008 $ |50 000|4 $|2 880 $ |
|Japon Est|0,009 $ |50 000| 4,50 $ |3 240 $ |
|||Paiement à l’utilisation|8,50 $|6 120 $|
|Capacité de réserve achetée|0,0064 $ (remise de 20 %) |100 RU/s ou 8 $ de capacité de réserve pré-achetée |-8 $|-5 760 $ |
|Facture nette|||0,50 $ |360 $ |

## <a name="next-steps"></a>Étapes suivantes

Pour approfondir vos connaissances sur l’optimisation des coûts dans Azure Cosmos DB, consultez les articles suivants :

* Pour plus d’informations, consultez [En quoi le modèle de tarification Cosmos DB est rentable pour les clients](total-cost-ownership.md)
* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur l’[optimisation du coût du débit](optimize-cost-throughput.md)
* En savoir plus sur l’[optimisation du coût de stockage](optimize-cost-storage.md)
* En savoir plus sur l’[optimisation du coût des lectures et écritures](optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des requêtes](./optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des comptes Azure Cosmos sur plusieurs régions](optimize-cost-regions.md)