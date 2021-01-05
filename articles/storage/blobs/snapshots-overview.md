---
title: Instantanés d’objet blob
titleSuffix: Azure Storage
description: Comprendre comment des captures instantanées fonctionnent et comment elles sont facturées.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/27/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: ab4c152f30ab96fe5e221a605a2339c773e32547
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295396"
---
# <a name="blob-snapshots"></a>Instantanés d’objet blob

Un instantané est une version en lecture seule d'un objet blob capturé à un instant donné.

> [!NOTE]
> Le contrôle de version des objets blob est une meilleure manière de conserver les versions précédentes d’un objet blob. Pour plus d’informations, consultez [Contrôle de version des objets blob](versioning-overview.md).

## <a name="about-blob-snapshots"></a>À propos des instantanés blob

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

Un instantané d’un objet blob est identique à l’objet blob de base, à la différence que l’URI de l’objet blob a une valeur **DateTime** à la fin qui indique l’heure à laquelle l’instantané a été pris. Par exemple, si l’URI de l’objet blob de pages est `http://storagesample.core.blob.windows.net/mydrives/myvhd`, l’URI de l’instantané est du type `http://storagesample.core.blob.windows.net/mydrives/myvhd?snapshot=2011-03-09T01:42:34.9360000Z`.

> [!NOTE]
> Tous les instantanés partagent l’URI de l’objet blob de base. La seule distinction entre l’objet blob de base et l’instantané est la valeur **DateTime** ajoutée à la fin.

Un objet blob peut avoir plusieurs instantanés. Les instantanés persistent jusqu’à ce qu’ils soient explicitement supprimés, indépendamment ou dans le cadre de l’opération de [suppression d’objets blob](/rest/api/storageservices/delete-blob) pour l’objet blob de base. Vous pouvez énumérer les instantanés associés à l’objet blob de base pour effectuer le suivi de vos instantanés actuels.

Lorsque vous créez un instantané d’un objet blob, les propriétés système de l’objet blob sont copiées dans l’instantané avec les mêmes valeurs. Les métadonnées de l’objet blob de base sont également copiées dans l’instantané, sauf si vous spécifiez des métadonnées distinctes pour l’instantané lorsque vous le créez. Une fois créé, un instantané peut être lu, copié ou supprimé, mais pas modifié.

Aucun bail associé à l’objet blob de base n’a d’incidence sur l’instantané. Vous ne pouvez pas acquérir de bail pour un instantané.

Un fichier de disque dur virtuel est utilisé pour stocker les informations et l’état actuels d’un disque de machine virtuelle. Vous pouvez détacher un disque de la machine virtuelle ou arrêter la machine virtuelle, puis prendre une capture instantanée de son fichier de disque dur virtuel. Vous pourrez par la suite utiliser ce fichier de capture instantanée pour récupérer le fichier de disque dur virtuel à ce moment précis et recréer la machine virtuelle.

## <a name="understand-how-snapshots-accrue-charges"></a>Présentation des frais liés aux instantanés

### <a name="important-billing-considerations"></a>Considérations importantes relatives à la facturation

La liste suivante contient des points clés à prendre en compte lors de la création d’un instantané.

- Des frais s’appliquent à votre compte de stockage pour des pages ou des blocs uniques, qu’ils soient dans l’objet blob ou dans l’instantané. Aucuns frais supplémentaires ne sont imputés à votre compte pour les instantanés associés à un objet blob tant que vous n'avez pas mis à jour l’objet blob sur lequel ils sont basés. Une fois que vous avez mis à jour l’objet blob de base, il diffère de ses instantanés. Dans ce cas, vous êtes facturé pour les blocs ou pages uniques de chaque objet blob ou instantané.
- Quand vous remplacez un bloc au sein d’un objet blob de bloc, ce bloc est ensuite facturé comme un bloc unique. Cela est vrai même si le bloc a le même ID de bloc et les mêmes données que dans l'instantané. Une fois le bloc revalidé, il diffère de son homologue dans tous les instantanés et vous serez facturé pour ses données. Il en va de même pour une page dans un objet blob de pages qui est mise à jour avec des données identiques.
- La mise à jour d’un objet blob de blocs par l’appel d’une méthode qui écrase tout le contenu du blob remplacera tous les objet blob de blocs. Si vous avez une capture instantanée associée à cet objet blob, tous les blocs dans l’objet blob de base et la capture instantanée seront désormais différents et vous serez facturé pour tous les blocs des deux objets blob. Cela est vrai même si les données de l’objet blob de base et l’instantané restent identiques.
- Le service blob Azure ne dispose d'aucun moyen pour déterminer si deux blocs contiennent des données identiques. Chaque bloc qui est téléchargé et validé est traité comme étant unique, même s’il a les mêmes données et le même ID de bloc. Comme des frais sont applicables aux blocs uniques, il est important de savoir que la mise à jour d’un objet blob qui a une capture instantanée entraîne la création de blocs uniques supplémentaires et donc des frais supplémentaires.

### <a name="minimize-costs-with-snapshot-management"></a>Réduire les coûts grâce à la gestion des captures instantanées

Nous vous recommandons de gérer vos captures instantanées avec soin pour éviter des frais supplémentaires. Vous pouvez suivre ces meilleures pratiques pour réduire les coûts liés au stockage de vos captures instantanées :

- Supprimez et recréez les instantanés associés à un objet blob chaque fois que vous mettez à jour l'objet blob, même si vous le mettez à jour avec des données identiques, sauf si la conception de votre application exige de tenir à jour des instantanés. En supprimant et en recréant les captures instantanées de l’objet blob, vous pouvez vous assurer que l’objet blob et les captures instantanées ne diffèrent pas.
- Si vous conservez des instantanés pour un objet blob, évitez d’appeler des méthodes qui remplacent la totalité de l’objet blob lorsque vous mettez à jour celui-ci. Au lieu de cela, mettez à jour le nombre de blocs le moins élevé possible pour réduire les coûts.

### <a name="snapshot-billing-scenarios"></a>Scénarios de facturation d’instantanés

Les scénarios suivants illustrent l’accumulation des coûts pour un objet blob de blocs et ses instantanés.

## <a name="pricing-and-billing"></a>Tarification et facturation

La création d’un instantané, c’est-à-dire d’une copie en lecture seule d’un objet blob, peut entraîner des frais de stockage de données supplémentaires pour votre compte. Lors de la conception de votre application, il est important de savoir comment ces frais peuvent s’accumuler pour pouvoir réduire les coûts.

Les captures instantanées d’objets blob comme les versions d’objets blob sont facturées au même tarif que les données actives. La façon dont les captures instantanées sont facturées varie selon que vous avez défini explicitement le niveau pour l’objet blob de base ou pour l’une de ses captures instantanées (ou versions). Pour plus d’informations sur les niveaux d’accès, consultez [Stockage d’objets blob Azure : niveaux d’accès chaud, froid et archive](storage-blob-storage-tiers.md).

Si vous n’avez pas modifié le niveau d’un objet blob ou d’une capture instantanée, vous êtes facturé pour des blocs de données uniques sur cet objet blob, ses captures instantanées et ses éventuelles versions. Pour plus d’informations, consultez [Facturation quand le niveau d’objet blob n’a pas été défini explicitement](#billing-when-the-blob-tier-has-not-been-explicitly-set).

Si vous avez modifié le niveau d’un objet blob ou d’une capture instantanée, vous êtes facturé pour l’intégralité de l’objet, que l’objet blob et sa capture instantanée se retrouvent ou non dans le même niveau. Pour plus d’informations, consultez [Facturation quand le niveau de l’objet blob a été défini explicitement](#billing-when-the-blob-tier-has-been-explicitly-set).

Pour plus d’informations sur les détails de facturation des versions d’objets blob, consultez [Contrôle de version d’objets blob](versioning-overview.md).

### <a name="billing-when-the-blob-tier-has-not-been-explicitly-set"></a>Facturation quand le niveau de l’objet blob n’a pas été défini explicitement

Si vous n’avez pas défini explicitement le niveau d’objet blob pour un objet blob de base ou l’une de ses captures instantanées, vous êtes facturé pour les blocs ou pages uniques de l’objet blob, ses captures instantanées et ses éventuelles versions. Les données partagées dans un objet blob et ses captures instantanées ne sont facturées qu’une seule fois. Lors de la mise à jour d’un objet blob, les données dans l’objet blob de base divergent des données stockées dans ses captures instantanées, et les données uniques sont facturées par bloc ou page.

Quand vous remplacez un bloc au sein d’un objet blob de bloc, ce bloc est ensuite facturé comme un bloc unique. Cela est vrai même si le bloc a le même ID de bloc et les mêmes données que dans l'instantané. Une fois le bloc revalidé, il diffère de son homologue dans la capture instantanée et vous serez facturé pour ses données. Il en va de même pour une page dans un objet blob de pages qui est mise à jour avec des données identiques.

Le stockage d’objets blob ne dispose d’aucun moyen pour déterminer si deux blocs contiennent des données identiques. Chaque bloc qui est téléchargé et validé est traité comme étant unique, même s’il a les mêmes données et le même ID de bloc. Les blocs uniques accumulant les frais, il est important de garder à l’esprit que la mise à jour d’un objet blob lorsque l’objet blob a des captures instantanées ou des versions générera des blocs uniques supplémentaires, et donc une augmentation des frais.

Lorsqu’un objet blob a des captures instantanées, appelez des opérations de mise à jour des objets blob de blocs de façon à ce qu’elles mettent à jour le plus petit nombre possible de blocs. Les opérations d’écriture qui permettent un contrôle plus précis des blocs sont [Put Block](/rest/api/storageservices/put-block) et [Put Block List](/rest/api/storageservices/put-block-list). L’opération [Put Blob](/rest/api/storageservices/put-blob), en revanche, remplace la totalité du contenu d’un objet blob et peut entraîner des frais supplémentaires.

Les scénarios suivants montrent comment les frais s’accumulent pour un objet blob de blocs et ses captures instantanées lorsque le niveau d’objet blob n’a pas été défini explicitement.

#### <a name="scenario-1"></a>Scénario 1

Dans le scénario 1, l'objet blob de base n'a pas été mis à jour depuis la capture de l'instantané. Des frais sont donc applicables uniquement pour les blocs uniques 1, 2 et 3.

![Diagramme 1 montrant la facturation des blocs uniques dans l’objet blob de base et la capture instantanée.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-1.png)

#### <a name="scenario-2"></a>Scénario 2

Dans le scénario 2, l'objet blob de base a été mis à jour, mais pas l'instantané. Le bloc 3 a été mis à jour, et bien qu’il contienne les mêmes données et le même ID, il est différent du bloc 3 de l’instantané. Par conséquent, des frais pour quatre blocs sont facturés au compte.

![Diagramme 2 montrant la facturation des blocs uniques dans l’objet blob de base et la capture instantanée.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-2.png)

#### <a name="scenario-3"></a>Scénario 3

Dans le scénario 3, l'objet blob de base a été mis à jour, mais pas l'instantané. Le bloc 3 a été remplacé par le bloc 4 dans l’objet blob de base, mais l’instantané reflète toujours le bloc 3. Par conséquent, des frais pour quatre blocs sont facturés au compte.

![Diagramme 3 montrant la facturation des blocs uniques dans l’objet blob de base et la capture instantanée.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-3.png)

#### <a name="scenario-4"></a>Scénario 4

Dans le scénario 4, l'objet blob de base a été complètement mis à jour et ne contient aucun de ses blocs d'origine. Par conséquent, des frais pour les huit blocs uniques sont facturés au compte.

![Diagramme 4 montrant la facturation des blocs uniques dans l’objet blob de base et la capture instantanée.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-4.png)

> [!TIP]
> Évitez d’appeler des méthodes qui remplacent la totalité de l’objet blob et mettez à jour des blocs individuels pour réduire les coûts.

### <a name="billing-when-the-blob-tier-has-been-explicitly-set"></a>Facturation quand le niveau de l’objet blob a été défini explicitement

Si vous avez défini explicitement le niveau d’objet blob pour un objet blob ou une capture instantanée (ou une version), vous êtes facturé pour la longueur totale du contenu de l’objet dans le nouveau niveau, qu’il partage ou non des blocs avec un objet dans le niveau d’origine. Vous êtes également facturé pour la longueur totale du contenu de la version la plus ancienne dans le niveau d’origine. Toutes les versions ou captures instantanées précédentes qui restent dans le niveau d’origine sont facturées pour les blocs uniques qu’elles peuvent partager, comme décrit dans [Facturation quand le niveau de l’objet blob n’a pas été défini explicitement](#billing-when-the-blob-tier-has-not-been-explicitly-set).

#### <a name="moving-a-blob-to-a-new-tier"></a>Déplacement d’un objet blob vers un nouveau niveau

Le tableau suivant décrit le comportement de facturation d’un objet blob ou d’une capture instantanée lors de leur déplacement vers un nouveau niveau.

| Quand le niveau d’objet blob est défini explicitement sur… | Vous êtes facturé pour... |
|-|-|
| Un objet blob de base avec une capture instantanée | L’objet blob de base dans le nouveau niveau et la capture instantanée la plus ancienne dans le niveau d’origine, ainsi que tous les blocs uniques dans d’autres captures instantanées.<sup>1</sup> |
| Un objet blob de base avec une version précédente et une capture instantanée | L’objet blob de base dans le nouveau niveau, la version la plus ancienne dans le niveau d’origine et la capture instantanée la plus ancienne dans le niveau d’origine, ainsi que tous les blocs uniques dans d’autres versions ou captures instantanées<sup>1</sup>. |
| Une capture instantanée | La capture instantanée dans le nouveau niveau et l’objet blob de base dans le niveau d’origine, ainsi que tous les blocs uniques dans d’autres captures instantanées.<sup>1</sup> |

<sup>1</sup>s’il existe d’autres versions ou captures instantanées précédentes qui n’ont pas été déplacées à partir de leur niveau d’origine, ces versions ou captures instantanées sont facturées en fonction du nombre de blocs uniques qu’elles contiennent, comme décrit dans [Facturation quand le niveau de l’objet blob n’a pas été défini explicitement](#billing-when-the-blob-tier-has-not-been-explicitly-set).

Le diagramme suivant illustre la façon dont les objets sont facturés quand un objet blob avec captures instantanées est déplacé vers un autre niveau.

:::image type="content" source="media/snapshots-overview/snapshot-billing-tiers.png" alt-text="Diagramme montrant comment les objets sont facturés quand un objet blob avec contrôle version est explicitement hiérarchisé.":::

La définition explicite du niveau pour un objet blob, une version ou une capture instantanée ne peut pas être annulée. Si vous déplacez un objet blob vers un nouveau niveau, puis le replacez à son niveau d’origine, vous êtes facturé pour la longueur totale du contenu de l’objet, même s’il partage des blocs avec d’autres objets dans le niveau d’origine.

Les opérations qui définissent explicitement le niveau d’un objet blob, d’une version ou d’une capture instantanée sont les suivantes :

- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Placer l’objet blob](/rest/api/storageservices/put-blob) avec le niveau spécifié
- [Placer la liste de blocs](/rest/api/storageservices/put-block-list) avec le niveau spécifié
- [Copier l’objet blob](/rest/api/storageservices/copy-blob) avec le niveau spécifié

#### <a name="deleting-a-blob-when-soft-delete-is-enabled"></a>Suppression d’un objet blob quand la suppression réversible est activée

Lorsque la suppression réversible d’objet blob est activée, si vous supprimez ou remplacez un objet blob de base dont le niveau est explicitement défini, toutes les versions précédentes ou captures instantanées de l’objet blob supprimé de manière réversible sont facturées pour la longueur totale du contenu. Pour plus d’informations sur la manière dont le contrôle de version d’objet blob et de la suppression réversible fonctionnent ensemble, consultez [Contrôle de version des objets blob et suppression réversible](versioning-overview.md#blob-versioning-and-soft-delete).

Le tableau suivant décrit le comportement de facturation d’un objet blob supprimé de manière réversible, selon que le contrôle de version est activé ou désactivé. Quand le contrôle de version est activé, une nouvelle version est créée lors de la suppression d’un objet blob de manière réversible. Quand le contrôle de version est désactivé, la suppression réversible d’un objet blob crée une capture instantanée de suppression réversible.

| Quand vous remplacez un objet blob de base par son niveau explicitement défini… | Vous êtes facturé pour... |
|-|-|
| Si la suppression réversible et le contrôle de version d’objet blob sont tous deux activés | Toutes les versions existantes pour la longueur totale du contenu, quel que soit le niveau. |
| Si la suppression réversible d’objet blob est activée alors que le contrôle de version est désactivé | Tous les captures instantanées de suppression réversible existantes pour la longueur totale du contenu, quel que soit le niveau. |

## <a name="next-steps"></a>Étapes suivantes

- [Contrôle de version des blobs](versioning-overview.md)
- [Créer et gérer un instantané blob dans .NET](snapshots-manage-dotnet.md)
- [Sauvegarder les disques de machines virtuelles Azure non gérés avec des captures instantanées incrémentielles](../../virtual-machines/windows/incremental-snapshots.md)
