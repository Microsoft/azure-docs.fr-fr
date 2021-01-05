---
title: Vue d’ensemble des images Red Hat Enterprise Linux dans Azure
description: Découvrez les images Red Hat Enterprise Linux dans Microsoft Azure.
author: asinn826
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 02/10/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: 5cac21046b6b4afdefc4213f6a1fab438d62cca3
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957535"
---
# <a name="overview-of-red-hat-enterprise-linux-images"></a>Vue d’ensemble des images Red Hat Enterprise Linux

Cet article décrit les images Red Hat Enterprise Linux (RHEL) disponibles sur la Place de marché Azure, ainsi que les stratégies relatives à leur nommage et à leur conservation.

Pour obtenir des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL, consultez la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata). Pour plus d’informations sur les tarifs, consultez [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

>[!IMPORTANT]
> Les images RHEL actuellement disponibles dans la Place de marché Azure prennent en charge les modèles de licence BYOS (Apportez votre propre abonnement) ou Paiement à l’utilisation. [Azure Hybrid Use Benefit](../../windows/hybrid-use-benefit-licensing.md) et le changement dynamique entre les licences BYOS et les licences de paiement à l’utilisation ne sont pas pris en charge. Pour changer de mode de licence, vous devez redéployer la machine virtuelle à partir de l’image correspondante.

>[!NOTE]
> Pour tout problème lié aux images RHEL sur la Place de marché Azure, ouvrez un ticket de support auprès de Microsoft.

## <a name="view-images-available-in-azure"></a>Voir les images disponibles dans Azure

Quand vous recherchez « Red Hat » sur la Place de marché, ou que vous créez une ressource dans l’interface utilisateur du portail Azure, vous ne voyez qu’un sous-ensemble des images RHEL disponibles. Vous pouvez toujours obtenir l’ensemble complet des images de machine virtuelle disponibles à l’aide de l’interface Azure CLI, de PowerShell et de l’API.

Pour voir l’ensemble des images Red Hat disponibles dans Azure, exécutez la commande suivante :

```azurecli-interactive
az vm image list --publisher RedHat --all
```

### <a name="naming-convention"></a>Conventions d’affectation de noms

Les images de machine virtuelle dans Azure sont organisées par éditeur, offre, référence SKU et version. La combinaison éditeur:offre:référence SKU:version est l’image URN et identifie de façon unique l’image à utiliser.

Par exemple, `RedHat:RHEL:8-LVM:8.1.20200318` fait référence à une image RHEL 8.1 partitionnée au format LVM, créée le 18 mars 2020.

Un exemple montrant comment créer une machine virtuelle RHEL 8.1 est présenté ici.

```azurecli-interactive
az vm create --name RhelVM --resource-group TestRG --image RedHat:RHEL:8-LVM:8.1.20200318 --no-wait
```

### <a name="the-latest-moniker"></a>Moniker « latest »

L’API REST Azure permet l’utilisation du moniker « latest » comme version au lieu de la version spécifique. Utiliser la version « latest » a pour effet de provisionner la dernière image disponible pour un éditeur, une offre et une référence SKU spécifiés.

Par exemple, `RedHat:RHEL:8-LVM:latest` fait référence à la dernière image partitionnée au format LVM de la famille RHEL 8 disponible.

```azurecli-interactive
az vm create --name RhelVM --resource-group TestRG --image RedHat:RHEL:8-LVM:latest --no-wait
```

>[!NOTE]
> En règle générale, la comparaison des versions pour déterminer la dernière version suit les règles de la [méthode CompareTo](/dotnet/api/system.version.compareto?view=netcore-3.1#system_version_compareto_system_version_).
Pour cette comparaison des versions d’image, les valeurs sont comparées comme objet [Version](/dotnet/api/system.version.-ctor?view=netframework-4.8), et non comme chaîne.

## <a name="rhel-6-image-types"></a>Types d’images RHEL 6

Les types d’images RHEL 6.x sont présentés dans le tableau suivant.

|Serveur de publication | Offre | Valeur de référence SKU | Version | Détails
|----------|-------|-----------|---------|--------
|Red Hat | RHEL | Version mineure (par exemple, 6.9) | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 6.9.2018010506) | Toutes les images RHEL 6.x standard respectent cette convention.
|Red Hat | rhel-byos | rhel-raw69 | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 6.9.20181023) | Cette image est une image RHEL 6.9 BYOS.
|Red Hat | RHEL | RHEL-SAP-APPS | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 6.8.2017053118) | Cette image est une image RHEL 6.8 pour applications SAP. Elle est autorisée à accéder aux référentiels d’applications SAP et aux référentiels RHEL de base.
|Red Hat | RHEL | RHEL-SAP-HANA | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 6.7.2017053121) | Cette image est une image RHEL 6.7 pour SAP HANA. Elle est autorisée à accéder aux référentiels SAP HANA et aux référentiels RHEL de base.

## <a name="rhel-7-image-types"></a>Types d’images RHEL 7

Il existe différents types d’images RHEL 7.x. Le tableau suivant présente les différents jeux d’images que nous proposons. Pour afficher une liste complète, utilisez la commande Azure CLI `az vm image list --publisher redhat --all`.

>[!NOTE]
> Sauf indication contraire, toutes les images sont partitionnées au format LVM et se connectent à des référentiels RHEL standard. Autrement dit, les référentiels ne sont pas des référentiels EUS (Extended Update Support) ni des services de mise à jour pour SAP (E4S). À partir de maintenant, nous ne publierons que des images partitionnées au format LVM, mais nous sommes ouverts à tout commentaire concernant cette décision. Pour plus d’informations sur le support EUS (Extended Update Support) et les services de mise à jour pour SAP, consultez [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

|Serveur de publication | Offre | Valeur de référence SKU | Version | Détails
|----------|-------|------------|---------|--------
|Red Hat | RHEL | Version mineure (par exemple, 7.6) | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019102813) | Les images publiées avant avril 2019 sont attachées à des référentiels RHEL standard. Celles publiées après avril 2019 sont associées aux référentiels EUS de Red Hat pour permettre le verrouillage d’une version mineure spécifique. Les clients qui veulent des référentiels standard doivent utiliser les images dont la valeur de référence SKU comporte 7-LVM ou 7-RAW (plus d’informations ci-dessous). Les images RHEL 7.7 et versions ultérieures sont partitionnées au format LVM. Toutes les autres images de cette catégorie sont à partitions brutes.
|Red Hat | RHEL | 7-RAW | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019102813) | Ces images sont à partitions brutes (par exemple, aucun volume logique n’a été ajouté).
|Red Hat | RHEL | 7-RAW-CI | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019072418) | Ces images sont à partitions brutes (par exemple, aucun volume logique n’a été ajouté) et utilisent cloud-init pour le provisionnement.
|Red Hat | RHEL | 7-LVM | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019062414) | Ces images sont partitionnées au format LVM.
|Red Hat | rhel-byos | rhel-{lvm,raw} | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.7.20190819) | Il s’agit des images RHEL 7 BYOS. Elles ne sont attachées à aucun référentiel et ne sont pas soumises aux frais de la prime RHEL. Si les images RHEL BYOS vous intéressent, [faites une demande d’accès](https://aka.ms/rhel-byos). Les valeurs SKU se terminent par la version mineure et indiquent si l’image est à partitions brutes ou partitionnée au format LVM. Par exemple, la valeur SKU rhel-lvm77 indique une image RHEL 7.7 partitionnée au format LVM.
|Red Hat | RHEL | RHEL-SAP | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019071300) | Il s’agit d’images RHEL for SAP. Elles sont autorisées à accéder aux référentiels SAP HANA et aux référentiels d’applications, ainsi qu’aux référentiels RHEL E4S. La facturation comprend la prime RHEL et la prime SAP en plus des frais de calcul de base.
|Red Hat | RHEL | RHEL-SAP-HA | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019062320) | Il s’agit d’images RHEL for SAP avec des services de haute disponibilité et de mise à jour. Elles sont autorisées à accéder aux référentiels SAP HANA, aux référentiels d’applications et aux référentiels à haute disponibilité, ainsi qu’aux référentiels RHEL E4S. La facturation comprend la prime RHEL, la prime SAP et la prime de haute disponibilité en plus des frais de calcul de base.
|Red Hat | RHEL | RHEL-HA | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.6.2019062019) | Ces images sont des images RHEL également autorisées à accéder au module complémentaire à haute disponibilité. Elles sont soumises à quelques frais supplémentaires par rapport aux frais de calcul RHEL et de base en raison de la prime du module complémentaire à haute disponibilité.
|Red Hat | RHEL | RHEL-SAP-APPS | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.3.2017053118) | Ces images sont obsolètes car les référentiels d’applications SAP et les référentiels SAP HANA ont été regroupés en référentiels SAP. Ces images sont des images RHEL for SAP Applications. Elles sont autorisées à accéder aux référentiels d’applications SAP et aux référentiels RHEL de base.
|Red Hat | RHEL | RHEL-SAP-HANA | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 7.3.2018051421) | Ces images sont obsolètes car les référentiels d’applications SAP et les référentiels SAP HANA ont été regroupés en référentiels SAP. Ces images sont des images RHEL for SAP HANA. Elles sont autorisées à accéder aux référentiels SAP HANA et aux référentiels RHEL de base.

## <a name="rhel-8-image-types"></a>Types d’images RHEL 8

>[!NOTE]
> Red Hat recommande l’utilisation de Grubby pour configurer les paramètres de ligne de commande du noyau dans RHEL 8 et versions ultérieures. Des informations supplémentaires sont disponibles [ici](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/configuring-kernel-command-line-parameters_managing-monitoring-and-updating-the-kernel).

Voici les détails des types d’images RHEL 8.

|Serveur de publication | Offre | Valeur de référence SKU | Version | Détails
|----------|-------|------------|---------|--------
|Red Hat | RHEL | 8 | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 8.0.20191023) | Il s’agit d’images RHEL 8 partitionnées au format LVM connectées à des référentiels Red Hat standard.
|Red Hat | RHEL | 8-gen2 | Valeurs concaténées de la version mineure RHEL et de la date de publication (par exemple, 8.0.20191024) | Il s’agit d’images RHEL 8 partitionnées au format LVM Hyper-V deuxième génération connectées à des référentiels Red Hat standard. Pour plus d’informations sur les machines virtuelles de 2e génération dans Azure, consultez [Prise en charge des machines virtuelles de 2e génération dans Azure](../../generation-2.md).

## <a name="rhel-longer-support-add-ons"></a>Modules complémentaires RHEL à prise en charge prolongée

### <a name="extended-update-support"></a>Extended Update Support

À compter d’avril 2019, des images RHEL attachées aux référentiels EUS sont disponibles par défaut. Vous trouverez plus d’informations sur EUS RHEL dans la [documentation de Red Hat](https://access.redhat.com/articles/rhel-eus).

Le basculement vers des référentiels EUS est possible et pris en charge. Pour obtenir des instructions sur la façon de basculer votre machine virtuelle vers EUS et des informations supplémentaires sur les dates de fin de vie de la prise en charge d’EUS, consultez [EUS RHEL et machines virtuelles RHEL avec verrouillage de version](./redhat-rhui.md#rhel-eus-and-version-locking-rhel-vms).

>[!NOTE]
> La technologie EUS n’est pas prise en charge sur RHEL Extras. Cela signifie que vous ne pourrez pas, sur EUS, installer un package généralement disponible à partir du canal RHEL Extras. Pour plus d’informations sur le cycle de vie des produits Red Hat Extras, consultez [Cycle de vie des Extras Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/extras/).

#### <a name="differentiate-between-regular-and-eus-images"></a>Différencier les images standard et les images EUS

Les clients qui veulent utiliser des images qui sont attachées à des référentiels EUS doivent utiliser l’image RHEL dont la référence SKU comporte un numéro de version mineure RHEL.

Par exemple, vous pouvez voir les deux images RHEL 7.4 disponibles suivantes.

```bash
RedHat:RHEL:7-LVM:7.6.2019062414
RedHat:RHEL:7.6:7.6.2019102813
```

Dans le cas présent, `RedHat:RHEL:7.6:7.6.2019102813` est attaché par défaut à des référentiels EUS. La valeur de référence SKU est 7.4. Et `RedHat:RHEL:7-LVM:7.6.2019062414` est attaché par défaut à des référentiels non EUS. La valeur de référence SKU est 7-LVM.

Pour utiliser des référentiels standard (non EUS), utilisez une image dont la référence SKU ne comporte pas de numéro de version mineure.

#### <a name="rhel-images-with-eus"></a>Images RHEL avec EUS

Les informations figurant dans le tableau suivant s’appliquent aux images RHEL connectées à des référentiels EUS.

>[!NOTE]
> Au moment de la rédaction, seules les versions mineures RHEL 7.4 et ultérieures sont compatibles EUS. La technologie EUS n’est plus prise en charge pour de RHEL version 7.3 et les versions antérieures.
>
> Pour plus d’informations sur la disponibilité de RHEL EUS, consultez [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

Version secondaire |Exemple d’image EUS              |État EUS                                                   |
:-------------|:------------------------------|:------------------------------------------------------------|
RHEL 7.4      |RedHat:RHEL:7.4:7.4.2019041718 | Les images publiées en avril 2019 et après cette date sont compatibles EUS par défaut.|
RHEL 7.5      |RedHat:RHEL:7.5:7.5.2019060305 | Les images publiées en juin 2019 et après cette date sont compatibles EUS par défaut. |
RHEL 7.6      |RedHat:RHEL:7.6:7.6.2019052206 | Les images publiées en mai 2019 et après cette date sont compatibles EUS par défaut. |
RHEL 8.0      |N/A                            | Aucune image EUS n’est disponible à partir de Red Hat.                               |

### <a name="update-services-for-sap"></a>Services de mise à jour pour SAP

Les dernières images RHEL for SAP seront connectées aux abonnements aux services de mise à jour pour les solutions SAP (E4S). Pour plus d’informations sur E4S, consultez la [documentation](https://access.redhat.com/support/policy/updates/errata#Update_Services_for_SAP_Solutions) de Red Hat.

#### <a name="rhel-images-with-e4s"></a>Images RHEL avec E4S

Les images des offres suivantes créées après décembre 2019 sont connectées aux référentiels E4S :

* RHEL-SAP (RHEL for SAP)
* RHEL-SAP-HA (RHEL for SAP avec des services à haute disponibilité et de mise à jour)

## <a name="other-available-offers-and-skus"></a>Autres offres et références SKU disponibles

La liste complète des offres et références SKU disponibles peut inclure d’autres images en plus de ce qui est indiqué dans le tableau précédent. par exemple `RedHat:rhel-ocp-marketplace:rhel74:7.4.1`. Ces offres peuvent être utilisées pour fournir une prise en charge de solutions spécifiques de la Place de marché. Elles peuvent aussi être publiées pour accéder à des préversions et à des fins de test. Elles peuvent être changées ou supprimées à tout moment sans avertissement. Ne les utilisez pas, sauf si leur présence est documentée publiquement par Microsoft ou Red Hat.

## <a name="publishing-policy"></a>Politique de publication

Microsoft et Red Hat mettent à jour les images à mesure que de nouvelles versions mineures sont publiées, afin de résoudre des vulnérabilités et menaces courantes (CVE) spécifiques, ou pour des changements ou mises à jour de configuration occasionnels. Nous tentons de fournir des images mises à jour dès que possible, dans les trois jours ouvrables suivant la publication d’une nouvelle version ou la disponibilité d’un correctif CVE.

Nous ne mettons à jour que la version mineure actuelle dans une famille d’images donnée. Avec la publication d’une version mineure plus récente, nous arrêtons la mise à jour de l’ancienne version mineure. Par exemple, avec la publication de RHEL 7.6, les images RHEL 7.5 ne sont plus mises à jour.

>[!NOTE]
> Les machines virtuelles Azure actives provisionnées à partir d’images RHEL avec paiement à l’utilisation sont connectées à Azure RHUI et peuvent recevoir des mises à jour et des correctifs dès leur publication par Red Hat et leur réplication vers Azure RHUI. Cela demande généralement moins de 24 heures après la publication officielle par Red Hat. Ces machines virtuelles ne nécessitent pas la publication d’une nouvelle image pour obtenir les mises à jour. Les clients ont un contrôle total sur le moment où lancer la mise à jour.

## <a name="image-retention-policy"></a>Stratégie de rétention d’image

La stratégie actuelle consiste à conserver toutes les images précédemment publiées. Nous nous réservons le droit de supprimer des images qui sont connues pour provoquer des problèmes de toutes sortes. Par exemple, les images avec des configurations incorrectes en raison de mises à jour de composant ou de plateforme ultérieures peuvent être supprimées. Les images susceptibles d’être supprimées respectent la politique de la Place de marché Azure actuelles qui est de fournir des notifications jusqu’à 30 jours avant la suppression de l’image.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir la liste complète des images RHEL dans Azure, consultez [Images RHEL (Red Hat Enterprise Linux) disponibles dans Azure](./redhat-imagelist.md).
* Pour en savoir plus sur Azure Red Hat Update Infrastructure, consultez [Red Hat Update Infrastructure pour les machines virtuelles RHEL à la demande dans Azure](./redhat-rhui.md).
* Pour en savoir plus sur l’offre RHEL BYOS, consultez [Images Gold BYOS (Apportez votre propre abonnement) de Red Hat Enterprise Linux dans Azure](./byos.md).
* Pour obtenir des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL, consultez la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).