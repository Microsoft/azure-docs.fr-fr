---
title: Stockage blob non modifiable – Stockage Azure
description: Le Stockage Azure assure la prise en charge des disques optiques non réinscriptibles (WORM) pour le Stockage (d’objets) Blob, qui permettent aux utilisateurs de stocker des données dans un état immuable et non effaçable sur une période donnée.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 11/13/2020
ms.author: tamram
ms.reviewer: hux
ms.subservice: blobs
ms.openlocfilehash: acb2ebb0d7ce70c6b5963a8a6c3e392091e4bb1e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010059"
---
# <a name="store-business-critical-blob-data-with-immutable-storage"></a>Stocker des données blob critiques pour l’entreprise avec un stockage immuable

Le stockage immuable pour le Stockage Blob Azure permet aux utilisateurs de stocker des objets de données critiques pour l’entreprise dans un état WORM (disque optique non réinscriptible). Cet état les rend non effaçables et immuables pour une durée spécifiée par l’utilisateur. Au cours de cette période de conservation, il est possible de créer et de lire des blobs, mais pas de les modifier ou de les supprimer. Le stockage immuable est disponible pour les comptes v1 universel, v2 universel, BlobStorage et BlockBlobStorage dans toutes les régions Azure.

Pour plus d’informations sur la définition et la suppression de conservations légales ou sur la création d’une stratégie de rétention basée sur la durée à l’aide du Portail Azure, de PowerShell ou d’Azure CLI, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage blob](storage-blob-immutability-policies-manage.md).

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="about-immutable-blob-storage"></a>À propos du stockage blob immuable

Le stockage immuable permet aux établissements de soins de santé, aux institutions financières et aux secteurs associés &mdash; en particulier les organisations de courtage &mdash; de stocker des données en toute sécurité. Il peut également être utilisé dans n’importe quel scénario pour protéger les données vitales contre la modification ou la suppression.

Les applications typiques incluent :

- **Conformité réglementaire** : Le stockage immuable pour le Stockage Blob Azure permet aux organisations de respecter entre autres les réglementations SEC 17a-4(f), CFTC 1.31(d) et FINRA. Un livre blanc technique de Cohasset Associates qui décrit en détail la manière dont le stockage immuable répond à ces exigences réglementaires est maintenant téléchargeable via le [Portail d’approbation de services Microsoft](https://aka.ms/AzureWormStorage). Le [Centre de confidentialité Azure](https://www.microsoft.com/trustcenter/compliance/compliance-overview) contient des informations détaillées sur nos certifications de conformité.

- **Conservation sécurisée des documents** : avec le stockage immuable pour le Stockage Blob, les données ne sont ni modifiables ni supprimables par les utilisateurs, même s’ils disposent d’un compte avec privilèges administratifs.

- **Conservation légale** : le stockage immuable pour le Stockage Blob Azure permet aux utilisateurs de stocker dans un état de protection inviolable des informations sensibles critiques dans le cadre d’une utilisation professionnelle ou portant sur un litige, pour la durée souhaitée jusqu’à ce que la conservation prenne fin. Cette fonctionnalité n’est pas limitée aux seuls cas d’utilisation légale, mais peut également être considérée comme une conservation basée sur un événement ou un verrou de l’entreprise, lorsqu’il est nécessaire de protéger les données sur la base de déclencheurs d’événements ou d’une stratégie d’entreprise.

Le stockage immuable prend en charge les fonctionnalités suivantes :

- **[Prise en charge de la stratégie de conservation limitée dans le temps](#time-based-retention-policies)**  : les utilisateurs peuvent définir des stratégies pour stocker les données pendant une période donnée. Lorsqu’une stratégie de conservation limitée dans le temps est définie, des objets Blob peuvent être créés et lus, mais ils ne peuvent pas être modifiés ni supprimés. Après l’expiration de la période de conservation, les tâches peuvent être effacées, mais pas remplacées.

- **[Prise en charge de la stratégie de conservation légale](#legal-holds)**  : Si la période de conservation est inconnue, les utilisateurs peuvent définir des conservations légales afin de stocker les données non modifiables jusqu’à ce que la conservation légales soit désactivée.  Lorsqu’une stratégie de conservation légale est définie, des objets Blob peuvent être créés et lus, mais ils ne peuvent pas être modifiés ni supprimés. Chaque archivage juridique est associé à une balise alphanumérique définie par l’utilisateur (comme pour l’ID d’un cas, un nom d’événement), utilisée comme chaîne d’identificateur (comme pour l’ID d’un cas). 

- **Prise en charge de tous les niveaux d’objets blob** : Les stratégies WORM sont indépendantes du niveau de Stockage Blob Azure et s’appliquent à tous les niveaux (chaud, froid et archive). Les clients peuvent transférer les données au niveau ayant le coût le plus adapté à leurs charges de travail tout en conservant l’immuabilité des données.

- **Configuration au niveau du conteneur** : Les utilisateurs peuvent configurer des stratégies de conservation limitée dans le temps et des balises de conservation légale au niveau du conteneur. Grâce à de simples paramètres au niveau du conteneur, ils ont la possibilité de créer et verrouiller des stratégies de conservation limitée dans le temps, d’étendre les périodes de conservation, de définir et supprimer des archivages juridiques, etc. Ces stratégies s’appliquent à tous les objets blob du conteneur, anciens ou nouveaux.

- **Prise en charge des enregistrements d’audit** : chaque conteneur comprend un journal d’audit de stratégie. Il présente jusqu’à sept commandes pour les stratégies de conservation limitée dans le temps verrouillées, et contient l’ID utilisateur, le type commande, les valeurs d’horodatage et l’intervalle de rétention. En ce qui concerne les stratégies d’archivage juridique, il comporte l’identifiant utilisateur, le type de commande, les timestamps et les balises d’archivage juridique. Ce journal est conservé pendant toute la durée de vie de la stratégie, conformément aux instructions réglementaires SEC 17a-4(f). Le [Journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md) est un journal plus complet de toutes les activités du plan de contrôle tandis que les [journaux de ressources Azure](../../azure-monitor/platform/platform-logs-overview.md) mémorisent et affichent les opérations du plan de données. Il est de la responsabilité de l’utilisateur de stocker ces journaux d’activité de manière persistante, car ceux-ci peuvent être nécessaires à des fins réglementaires ou autres.

## <a name="how-it-works"></a>Fonctionnement

Le stockage immuable du Stockage Blob Azure prend en charge deux types de stratégies WORM ou immuables : la conservation limitée dans le temps et l’archivage juridique. Lorsqu’une stratégie de conservation limitée dans le temps ou d’archivage juridique est appliquée sur un conteneur, tous les objets blob existants prennent l’état WORM immuable en moins de 30 secondes. Tous les nouveaux objets blob chargés dans ce conteneur protégé par une stratégie ont également cet état. Une fois que tous les objets blob sont à l’état immuable, la stratégie immuable est confirmée et aucune opération de remplacement ou de suppression n’est autorisée dans le conteneur immuable.

La suppression du conteneur et du compte de stockage n’est pas non plus autorisée si le conteneur contient des objets blob protégés par une conservation légale ou une stratégie limitée dans le temps verrouillée. Une stratégie de conservation légale vous protège contre la suppression des objets blob, des conteneurs et des comptes de stockage. Les stratégies limitées dans le temps verrouillées et déverrouillées assurent une protection contre la suppression des objets blob pendant la durée spécifiée. Les stratégies limitées dans le temps verrouillées et déverrouillées assurent une protection contre la suppression des conteneurs à condition qu’ils contiennent au moins un objet blob. Seul un conteneur assorti d’une stratégie limitée dans le temps *verrouillée* assure une protection contre la suppression des comptes de stockage ; les conteneurs assortis de stratégies limitées dans le temps déverrouillées n’offrent pas de conformité ni de protection contre la suppression des comptes de stockage.

Pour plus d’informations sur la façon de définir et de verrouiller des stratégies de rétention limitée dans le temps, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage blob](storage-blob-immutability-policies-manage.md).

## <a name="time-based-retention-policies"></a>Stratégies de rétention limitée dans le temps

> [!IMPORTANT]
> Une stratégie de rétention limitée dans le temps doit être *verrouillée* pour que l’objet blob ait un état immuable conforme (protégé contre l’écriture et la suppression) conformément entre autres à la réglementation SEC 17a-4(f). Il est recommandé de verrouiller la stratégie au bout d’un délai raisonnable, en général en moins de 24 heures. L’état initial de la stratégie de rétention définie dans le temps est *déverrouillé*, ce qui vous permet de tester la fonctionnalité et d’apporter des modifications à la stratégie avant de la verrouiller. Tant que l’état *déverrouillé* protège l’immuabilité du conteneur, nous conseillons de ne pas utiliser l’état *déverrouillé* pour d’autres raisons que des évaluations de fonctionnalités à court terme. 

Lorsqu’une stratégie de conservation limitée dans le temps est appliquée à un conteneur, tous les objets blob de ce conteneur conserveront l’état immuable pendant la période de conservation *effective*. La durée de rétention effective pour les objets blob est égale à la différence entre l’**heure de création** de l’objet blob et l’intervalle de rétention spécifié par l’utilisateur. Dans la mesure où les utilisateurs peuvent étendre la période de conservation, le stockage immuable utilise la valeur la plus récente de la période de conservation spécifiée par l’utilisateur pour calculer la durée de conservation effective.

Par exemple, supposons qu’un utilisateur crée une stratégie de rétention limitée dans le temps avec une période de conservation de cinq ans. Un objet blob existant dans ce conteneur, _testblob1_, a été créé il y a un an. Autrement dit, la durée de rétention effective de _testblob1_ est de quatre ans. Quand un nouvel objet blob, _testblob2_, est chargé dans le conteneur, la durée de rétention effective de _testblob2_ est de cinq ans à partir du moment où il est créé.

Une stratégie de rétention limitée dans le temps et déverrouillée est recommandée à des fins de test uniquement et une stratégie doit être verrouillée conformément à la réglementation SEC 17a-4(f) et à d’autres réglementations. Une fois verrouillée, une stratégie de rétention limitée dans le temps ne peut plus être supprimée ; seul un maximum de cinq prolongations de la période de rétention effective est autorisé.

Les limites suivantes s’appliquent aux stratégies de rétention :

- Pour un compte de stockage, le nombre maximal de conteneurs avec stratégies immuables limitées dans le temps verrouillées est de 10 000.
- La période de rétention minimale est de 1 jour. Le maximum est de 146 000 jours (400 ans).
- Pour un conteneur, il est possible de modifier jusqu’à cinq fois la période de rétention définie dans des stratégies immuables verrouillées limitées dans le temps.
- Pour un conteneur, un maximum de sept journaux d’audit de stratégie de rétention limitée dans le temps est conservé pour une stratégie verrouillée.

### <a name="allow-protected-append-blobs-writes"></a>Autoriser les écritures protégées d’objets blob d’ajout

Les objets blob d’ajout sont constitués de blocs de données et sont optimisés pour les opérations d’ajout de données nécessaires dans les scénarios d’audit et de journalisation. Du fait de la conception des objets blob d’ajout, les nouveaux blocs ne peuvent être ajoutés qu’à la fin des objets blob. Indépendamment de l’immuabilité, la modification ou la suppression de blocs existants dans un objet blob d’ajout ne sont absolument pas autorisées. Pour en savoir plus sur les objets blob d’ajout, consultez [À propos des objets blob d’ajout](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs).

Seules les stratégies de rétention limitées dans le temps ont un paramètre `allowProtectedAppendWrites` qui autorise l’écriture de nouveaux blocs dans un objet blob d’ajout tout en conservant la protection et la conformité de l’immuabilité. Si ce paramètre est activé, vous êtes autorisé à créer un objet blob d’ajout directement dans le conteneur protégé par une stratégie et à continuer à ajouter de nouveaux blocs de données à la fin des objets blob d’ajout existants à l’aide de l’API *AppendBlock*. Seuls de nouveaux blocs peuvent être ajoutés et les blocs existants ne peuvent pas être modifiés ou supprimés. La protection de l’immuabilité de la durée de rétention continue de s’appliquer, ce qui empêche la suppression de l’objet blob d’ajout tant que la période de rétention effective n’est pas passée. L’activation de ce paramètre n’affecte pas le comportement d’immuabilité des objets blob de blocs ou de pages.

Sachant que ce paramètre fait partie d’une stratégie de rétention limitée dans le temps, les objets blob d’ajout restent toujours à l’état immuable pendant la période de rétention *effective*. Comme il est possible d’ajouter de nouvelles données au-delà de la création initiale de l’objet blob d’ajout, il existe une légère différence quant à la façon dont la durée de rétention est déterminée. La rétention effective est égale à la différence entre le moment où l’objet blob d’ajout est **modifié pour la dernier fois** et l’intervalle de rétention spécifié par l’utilisateur. De la même manière, quand l’intervalle de rétention est allongé, le stockage immuable utilise la valeur la plus récente de l’intervalle de rétention spécifié par l’utilisateur pour calculer la durée de conservation effective.

Par exemple, supposez qu’un utilisateur crée une stratégie de rétention limitée dans le temps avec le paramètre `allowProtectedAppendWrites` activé et un intervalle de rétention de 90 jours. Un objet blob d’ajout, _logblob1_, est créé aujourd’hui dans le conteneur et de nouveaux journaux continuent d’y être ajoutés les 10 jours suivants. Ainsi, la période de rétention effective de _logblob1_ est de 100 jours à partir d’aujourd’hui (dernière fois qu’il ajouté + 90 jours).

Les stratégies de rétention limitées dans le temps déverrouillées permettent d’activer et de désactiver à tout moment le paramètre `allowProtectedAppendWrites`. Une fois que la stratégie de rétention limitée dans le temps est verrouillée, le paramètre `allowProtectedAppendWrites` ne peut pas être modifié.

Les stratégies de conservation légale ne peuvent pas activer `allowProtectedAppendWrites`, et toute conservation légale annule la propriété « allowProtectedAppendWrites ». Si une conservation légale est appliquée à une stratégie de rétention limitée dans le temps avec le paramètre `allowProtectedAppendWrites` activé, l’API *AppendBlock* échoue tant que la conservation légale n’est pas levée.

## <a name="legal-holds"></a>Conservation juridique

Les conservations légales sont des conservations temporaires qui peuvent être utilisées à des fins d’enquête judiciaire ou de stratégies de protection générales. Chaque stratégie de conservation légale doit être associée à une ou plusieurs balises. Les balises servent d’identificateur nommé, comme un ID de cas ou un événement, pour classer et décrire l’objet de la conservation.

Un conteneur peut avoir à la fois une stratégie d’archivage juridique et une stratégie de conservation limitée dans le temps. Tous les objets blob de ce conteneur conservent l’état immuable jusqu’à ce que toutes les stratégies d’archivage juridique aient été levées, même si leur période de conservation effective est écoulée. À l’inverse, un objet blob demeure dans un état immuable jusqu’à expiration de la période de conservation effective, même si toutes les stratégies d’archivage juridique ont été levées.

Les limites suivantes s’appliquent aux conservations légales :

- Pour un compte de stockage, le nombre maximal de conteneurs avec le paramètre de conservation légale est de 10 000.
- Le nombre maximal par conteneur de balises d’archivage juridique est de 10.
- La longueur minimale d’une balise de conservation légale est de trois caractères alphanumériques. La longueur maximale est de 23 caractères alphanumériques.
- Pour un conteneur, 10 journaux d’activité de stratégie de rétention légale au maximum sont conservés pour la durée de la stratégie.

## <a name="scenarios"></a>Scénarios

Le tableau suivant montre les types d’opérations de stockage blob qui sont désactivés dans chaque scénario immuable. Pour plus d’informations, consultez la documentation [API REST du service Blob Azure](/rest/api/storageservices/blob-service-rest-api).

| Scénario | État des objets blob | Opérations Blob refusées | Protection de conteneur et de compte |
|--|--|--|--|
| L’intervalle de rétention effective sur l’objet Blob n’a pas encore expiré et/ou une conservation juridique est définie | Immuable : non modifiable et non supprimable | Put Blob<sup>1</sup>, Put Block<sup>1</sup>, Put Block List<sup>1</sup>, Delete Container, Delete Blob, t Blob Metadata, Put Page, Set Blob Properties, Snapshot Blob, Incremental Copy Blob, Append Block<sup>2</sup> | Suppression de conteneur refusée ; Suppression de compte de stockage refusée |
| L’intervalle de rétention effective sur l’objet blob a expiré et aucune conservation légale n’est définie | Non modifiable seulement (suppressions autorisées) | Put Blob<sup>1</sup>, Put Block<sup>1</sup>, Put Block List<sup>1</sup>, Set Blob Metadata, Put Page, Set Blob Properties,  Snapshot Blob, Incremental Copy Blob, Append Block<sup>2</sup> | Suppression de conteneur refusée s’il existe au moins 1 objet blob dans le conteneur protégé ; suppression de compte de stockage refusée uniquement pour les stratégies limitées dans le temps *verrouillées* |
| Aucune stratégie WORM appliquée (aucune rétention limitée dans le temps et aucune balise de conservation légale) | Mutable | None | None |

<sup>1</sup> Le service Blob permet à ces opérations de créer un seul objet blob. Toutes les opérations d’écrasement ultérieures sur un chemin d’objets blob existant dans un conteneur immuable ne sont pas autorisées.

<sup>2</sup> Append Block n’est autorisé que pour les stratégies de rétention limitée dans le temps avec la propriété `allowProtectedAppendWrites` activée. Pour plus d’informations, consultez la section [Autoriser les écritures protégées d’objets blob d’ajout](#allow-protected-append-blobs-writes).

> [!IMPORTANT]
> Certaines charges de travail, telles que [Sauvegarde SQL vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url), créent un blob et l’ajoutent ensuite. Si le conteneur est dotée d’une stratégie active de rétention basée sur le temps ou d’une conservation légale en place, ce modèle échouera.

## <a name="pricing"></a>Tarifs

L’utilisation de cette fonctionnalité n’entraîne aucun coût supplémentaire. Les données immuables sont facturées de la même façon que les données modifiables classiques. Pour plus d’informations sur les prix du stockage Blob Azure, consultez la [page de tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="faq"></a>Questions fréquentes (FAQ)

**Pouvez-vous fournir de la documentation sur la conformité WORM ?**

Oui. Pour documenter la conformité, Microsoft a choisi les services de Cohasset Associates, une importante société d’évaluation indépendante spécialisée dans la gestion des dossiers et la gouvernance de l’information, afin d’évaluer le stockage Blob immuable et sa conformité aux exigences propres au secteur des services financiers. Cohasset a confirmé que le stockage Blob immuable, lorsqu’il est utilisé pour conserver les blobs temporels dans un état WORM, répond aux exigences de stockage appropriées décrites par la réglementation FINRA 4511, le paragraphe 1.31(c)-(d) de la réglementation CFTC et le paragraphe 17a-4 de la réglementation SEC. Microsoft a opté pour cet ensemble de règles, car elles représentent les recommandations les plus normatives au monde en matière de rétention des enregistrements pour les institutions financières. Le rapport Cohasset est disponible dans le [Centre de gestion de la confidentialité des services Microsoft](https://aka.ms/AzureWormStorage). Pour demander une lettre d’attestation de Microsoft concernant la conformité à l’immuabilité WORM, contactez le support Azure.

**Cette fonctionnalité s’applique-t-elle uniquement aux objets blob de blocs et aux objets blob d’ajout ou aussi aux objets blob de pages ?**

Si le stockage immuable peut être utilisé avec tout type d’objet blob, puisqu’il est défini au niveau du conteneur, nous vous recommandons cependant d’utiliser WORM pour les conteneurs qui stockent principalement des objets blob de blocs et des objets blob d’ajout. Les objets blob existants d’un conteneur sont protégés par une stratégie WORM nouvellement définie. Mais les nouveaux objets blob de pages doivent être créés en dehors d’un conteneur WORM avant d’y être copiés. Une fois copiés dans un conteneur WORM, aucune autre modification ne peut être apportée à un objet blob de pages. La définition d’une stratégie WORM sur un conteneur qui stocke des disques durs virtuels (objets blob de pages) pour toute machine virtuelle active est déconseillée, car elle verrouille le disque de la machine virtuelle. Avant de verrouiller des stratégies limitées dans le temps, nous vous recommandons de consulter attentivement la documentation et de tester vos scénarios.

**Dois-je créer un compte de stockage pour utiliser cette fonctionnalité ?**

Non. Vous pouvez utiliser le stockage immuable avec des comptes v1 universel, v2 universel, BlobStorage ou BlockBlobStorage existants ou nouvellement créés. Les comptes de stockage v1 universel sont pris en charge, mais nous vous recommandons de les mettre à niveau vers la version v2 universel de façon à bénéficier de fonctionnalités supplémentaires. Pour plus d’informations sur la mise à niveau d’un compte de stockage v1 universel, consultez [Mettre à niveau un compte de stockage](../common/storage-account-upgrade.md).

**Puis-je appliquer à la fois une conservation légale et une stratégie de rétention à durée définie ?**

Oui, un conteneur peut avoir à la fois une stratégie de conservation légale et une stratégie de conservation basée sur la durée. Toutefois, le paramètre « allowProtectedAppendWrites » ne s’applique pas tant que la conservation légale n’est pas désactivée. Tous les objets blob de ce conteneur conservent l’état immuable jusqu’à ce que toutes les stratégies d’archivage juridique aient été levées, même si leur période de conservation effective est écoulée. À l’inverse, un objet blob demeure dans un état immuable jusqu’à expiration de la période de conservation effective, même si toutes les stratégies d’archivage juridique ont été levées. 

**Les stratégies de conservation légale s’appliquent-elles uniquement aux procédures judiciaires ou existe-t-il d’autres scénarios d’utilisation ?**

Non, « conservation légale » est l’expression générale désignant une stratégie de rétention limitée dans le temps. Il n’est pas nécessaire de l’utiliser uniquement dans le cadre de procédures contentieuses. Les stratégies de conservation légale sont utiles pour désactiver le remplacement et les suppressions pour protéger les données WORM importantes de l’entreprise, lorsque la période de conservation est inconnue. Vous pouvez utiliser la période de conservation en tant que stratégie d’entreprise pour protéger vos charges de travail WORM stratégiques ou en tant que stratégie intermédiaire avant qu’un événement personnalisé ne nécessite une stratégie de rétention limitée dans le temps. 

**Puis-je supprimer une stratégie de rétention à durée définie _verrouillée_ ou une conservation légale ?**

Seules les stratégies de rétention à durée définie non verrouillées peuvent être supprimées d’un conteneur. Une fois verrouillée, une stratégie de rétention à durée définie ne peut plus être supprimée ; seules les prolongations effectives de la période de conservation sont autorisées. Les balises de conservation légale peuvent être supprimées. Lorsque toutes les balises légales sont supprimées, la conservation légale l’est également.

**Que se passe-t-il si j’essaie de supprimer un conteneur assorti d’une stratégie de rétention limitée dans le temps ou d’une conservation légale ?**

L’opération Delete Container échoue s’il existe au moins un objet blob dans le conteneur assorti d’une stratégie de conservation limitée dans le temps verrouillée ou déverrouillée ou si le conteneur contient une conservation légale. L’opération Delete Container réussit uniquement s’il n’existe aucun objet blob dans le conteneur et qu’il n’y a pas de conservations légales. 

**Que se passe-t-il si j’essaie de supprimer un compte de stockage avec un conteneur assorti d’une stratégie de rétention limitée dans le temps ou d’une conservation légale ?**

La suppression du compte de stockage échoue s’il existe au moins un conteneur assorti d’une conservation légale définie ou d’une stratégie limitée dans le temps **verrouillée**. Un conteneur assorti d’une stratégie limitée dans le temps déverrouillée ne protège pas contre une suppression de compte de stockage. Vous devez supprimer toutes les conservations légales ainsi que tous les conteneurs **verrouillés** pour pouvoir supprimer le compte de stockage. Pour plus d’informations sur la suppression des conteneurs, voir la question précédente. Vous pouvez aussi appliquer d’autres protections contre la suppression de votre compte de stockage avec des [verrous Azure Resource Manager](../../azure-resource-manager/management/lock-resources.md).

**Puis-je déplacer les données entre différents niveaux d’objets blob (chaud, froid, archive) quand l’objet blob est à l’état immuable ?**

Oui, vous pouvez utiliser la commande Set Blob Tier pour déplacer des données sur d’autres niveaux d’objets blob tant en conservant leur état immuable conforme. Le stockage immuable est pris en charge sur les niveaux d’objets blob chaud, froid et archive.

**Que se passe-t-il si je ne paie pas et que mon intervalle de rétention n’a pas expiré ?**

En cas de défaut de paiement, les stratégies de conservation des données normales s’appliquent, comme le stipulent les conditions générales de votre contrat avec Microsoft. Pour obtenir des informations générales, consultez [Gestion des données chez Microsoft](https://www.microsoft.com/en-us/trust-center/privacy/data-management). 

**Proposez-vous une période d’essai ou de grâce pour essayer la fonctionnalité ?**

Oui. Lors de la définition initiale d’une stratégie de conservation limitée dans le temps, celle-ci a pour état *déverrouillé*. Dans cet état, vous pouvez apporter les modifications souhaitées à la période de conservation, par exemple, l’augmenter, la diminuer ou même supprimer la stratégie. Une fois que la stratégie est verrouillée, elle reste verrouillée jusqu’à l’expiration de l’intervalle de rétention. Cette stratégie verrouillée empêche la suppression et la modification de l’intervalle de rétention. Il est vivement recommandé de n’utiliser l’état *déverrouillé* qu’à des fins d’évaluation et de verrouiller la stratégie dans les 24 heures. Ces pratiques permettent de respecter entre autres la réglementation SEC 17a-4(f).

**Puis-je utiliser la suppression réversible en même temps que les stratégies de blob immuable ?**

Oui, si vos exigences de conformité autorisent l’activation de la suppression réversible. [La suppression réversible pour le Stockage Blob Azure](./soft-delete-blob-overview.md) s’applique à tous les conteneurs d’un compte de stockage, indépendamment de la conservation légale ou d’une stratégie de rétention limitée dans le temps. Nous recommandons d’activer la suppression réversible pour une protection supplémentaire avant d’appliquer et de confirmer toutes les stratégies WORM immuables.

## <a name="next-steps"></a>Étapes suivantes

- [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](storage-blob-immutability-policies-manage.md)
- [Définir des règles pour hiérarchiser et supprimer automatiquement les données Blob avec la gestion du cycle de vie](storage-lifecycle-management-concepts.md)
- [Suppression réversible pour les objets blob de Stockage Azure](./soft-delete-blob-overview.md)
- [Protéger des abonnements, des groupes de ressources et des ressources avec des verrous Azure Resource Manager](../../azure-resource-manager/management/lock-resources.md).