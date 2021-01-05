---
title: Images Red Hat Enterprise Linux disponibles dans Azure
description: En savoir plus sur les images Red Hat Enterprise Linux dans Microsoft Azure
author: asinn826
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 04/16/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: ea0e8c148a847ecc23bab5e87a1dc60a626cf0c1
ms.sourcegitcommit: 1d366d72357db47feaea20c54004dc4467391364
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95411612"
---
# <a name="red-hat-enterprise-linux-rhel-images-available-in-azure"></a>Images RHEL (Red Hat Enterprise Linux) disponibles dans Azure
Azure propose une variété d’images RHEL pour différents cas d’usage.

> [!NOTE]
> Toutes les images RHEL sont disponibles dans les clouds Azure publics et Azure Government. Elles ne sont pas disponibles dans les clouds Azure Chine.

## <a name="list-of-rhel-images"></a>Liste des images RHEL
Voici la liste des images RHEL disponibles dans Azure. Sauf indication contraire, toutes les images sont partitionnées et attachées aux dépôts RHEL standard (et non EUS, E4S). Les images suivantes sont actuellement disponibles pour une utilisation générale :

> [!NOTE]
> Les images RAW ne sont plus produites, au profit des images partitionnées au format LVM. Le format LVM offre plusieurs avantages par rapport à l’ancien schéma de partitionnement but (non LVM), dont des options de redimensionnement de partition beaucoup plus flexibles.

Offre| SKU | Partitionnement | Approvisionnement | Notes
:----|:----|:-------------|:-------------|:-----
RHEL          | 6.7      | RAW    | Agent Linux | Support du cycle de vie étendu disponible à partir du 1er décembre. [Vous trouverez plus de détails ici.](redhat-extended-lifecycle-support.md)
|             | 6.8      | RAW    | Agent Linux | Support du cycle de vie étendu disponible à partir du 1er décembre. [Vous trouverez plus de détails ici.](redhat-extended-lifecycle-support.md)
|             | 6.9      | RAW    | Agent Linux | Support du cycle de vie étendu disponible à partir du 1er décembre. [Vous trouverez plus de détails ici.](redhat-extended-lifecycle-support.md)
|             | 6.10     | RAW    | Agent Linux | Support du cycle de vie étendu disponible à partir du 1er décembre. [Vous trouverez plus de détails ici.](redhat-extended-lifecycle-support.md)
|             | 7-RAW    | RAW    | Agent Linux | Famille d’images RHEL 7.x <br> Attachement à des dépôts standard par défaut (et non EUS).
|             | 7-LVM    | LVM    | Agent Linux | Famille d’images RHEL 7.x <br> Attachement à des dépôts standard par défaut (et non EUS). Si vous recherchez une image RHEL standard à déployer, utilisez cet ensemble d’images et/ou son équivalent de 2e génération.
|             | 7lvm-gen2| LVM    | Agent Linux | Famille d’images RHEL 7.x, 2e génération. <br> Attachement à des dépôts standard par défaut (et non EUS). Si vous recherchez une image RHEL standard à déployer, utilisez cet ensemble d’images et/ou son équivalent de 1re génération.
|             | 7-RAW-CI | RAW-CI | Cloud-init  | Famille d’images RHEL 7.x <br> Attachement à des dépôts standard par défaut (et non EUS).
|             | 7.2      | RAW    | Agent Linux |
|             | 7.3      | RAW    | Agent Linux |
|             | 7.4      | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut à compter d’avril 2019
|             | 74-gen2  | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 7.5      | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut à compter de juin 2019
|             | 75-gen2  | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 7.6      | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut à compter de mai 2019
|             | 76-gen2  | RAW    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 7,7      | LVM    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 77-gen2  | LVM    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 7.8      | LVM    | Agent Linux | Attaché aux dépôts ordinaires (EUS non disponible pour RHEL 7.8)
|             | 78-gen2  | LVM    | Agent Linux | Attaché aux dépôts ordinaires (EUS non disponible pour RHEL 7.8)
|             | 7.9      | LVM    | Agent Linux | Attaché aux référentiels ordinaires (EUS non disponible pour RHEL 7.9)
|             | 79-gen2  | LVM    | Agent Linux | Attaché aux référentiels ordinaires (EUS non disponible pour RHEL 7.9)
|             | 8-LVM    | LVM    | Agent Linux | Famille d’images RHEL 8.x. Attachement à des dépôts standard.
|             | 8-lvm-gen2| LVM    | Agent Linux | Hyper-V Génération 2 - Famille d’images RHEL 8.x. Attachement à des dépôts standard.
|             | 8        | LVM    | Agent Linux | Images RHEL 8.0.
|             | 8-gen2   | LVM    | Agent Linux | Hyper-V Génération 2 - Images RHEL 8.0.
|             | 8.1      | LVM    | Agent Linux | Attaché à des référentiels EUS par défaut
|             | 81gen2   | LVM    | Agent Linux | Hyper-V Generation 2 – Attaché à des référentiels EUS à compter de novembre 2020.
|             | 8.1-ci   | LVM    | Agent Linux | Attaché à des référentiels EUS à compter de novembre 2020.
|             | 81-ci-gen2| LVM    | Agent Linux | Hyper-V Generation 2 – Attaché à des référentiels EUS à compter de novembre 2020.
|             | 8,2      | LVM    | Agent Linux | Attaché à des référentiels EUS à compter de novembre 2020.
|             | 82gen2   | LVM    | Agent Linux | Hyper-V Generation 2 – Attaché à des référentiels EUS à compter de novembre 2020.
|             | 8.3   | LVM    | Agent Linux |  Attaché aux référentiels ordinaires (EUS non disponible pour RHEL 8.3)
|             | 83-gen2   | LVM    | Agent Linux |Hyper-V Generation 2 – Attaché aux référentiels ordinaires (EUS non disponible pour RHEL 8.3)
RHEL-SAP      | 7.4      | LVM    | Agent Linux | RHEL 7.4 pour SAP HANA et les applications métiers. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 74sap-gen2| LVM    | Agent Linux | RHEL 7.4 pour SAP HANA et les applications métiers. Image de 2e génération. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 7.5       | LVM    | Agent Linux | RHEL 7.5 pour SAP HANA et les applications métiers. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 75sap-gen2| LVM    | Agent Linux | RHEL 7.5 pour SAP HANA et les applications métiers. Image de 2e génération. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 7.6       | LVM    | Agent Linux | RHEL 7.6 for SAP HANA et Business Applications. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 76sap-gen2| LVM    | Agent Linux | RHEL 7.6 for SAP HANA et Business Applications. Image de 2e génération. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
|             | 7,7       | LVM    | Agent Linux | RHEL 7.7 for SAP HANA et Business Applications. Attachement à des dépôts E4S, facture une prime pour SAP et RHEL, ainsi que des frais de calcul de base.
RHEL-SAP-HANA (sera supprimé en novembre 2020) | 6.7       | RAW    | Agent Linux | RHEL 6.7 pour SAP HANA. Obsolète, privilégier les images RHEL-SAP. Cette image sera supprimée en novembre 2020. Pour plus d’informations sur les offres cloud SAP de Red Hat, [cliquez ici](https://access.redhat.com/articles/3751271).
|             | 7.2       | LVM    | Agent Linux | RHEL 7.2 pour SAP HANA. Obsolète, privilégier les images RHEL-SAP. Cette image sera supprimée en novembre 2020. Pour plus d’informations sur les offres cloud SAP de Red Hat, [cliquez ici](https://access.redhat.com/articles/3751271).
|             | 7.3       | LVM    | Agent Linux | RHEL 7.3 pour SAP HANA. Obsolète, privilégier les images RHEL-SAP. Cette image sera supprimée en novembre 2020. Pour plus d’informations sur les offres cloud SAP de Red Hat, [cliquez ici](https://access.redhat.com/articles/3751271).
RHEL-SAP-APPS | 6.8       | RAW    | Agent Linux | RHEL 6.8 pour applications métiers SAP. Obsolète, privilégier les images RHEL-SAP.
|             | 7.3       | LVM    | Agent Linux | RHEL 7.3 pour applications métiers SAP. Obsolète, privilégier les images RHEL-SAP.
|             | 7.4       | LVM    | Agent Linux | RHEL 7.4 pour applications métiers SAP.
|             | 7.6       | LVM    | Agent Linux | RHEL 7.6 pour applications métiers SAP.
|             | 7,7       | LVM    | Agent Linux | RHEL 7.7 pour applications métiers SAP.
|             | 77-gen2       | LVM    | Agent Linux | RHEL 7.7 pour applications métiers SAP. Image de 2e génération
|             | 8.1       | LVM    | Agent Linux | RHEL 8.1 pour applications métiers SAP.
|             | 81-gen2      | LVM    | Agent Linux | RHEL 8.1 pour applications métiers SAP. Image de 2e génération.
|             | 8,2       | LVM    | Agent Linux | RHEL 8.2 pour applications métiers SAP.
|             | 82-gen2      | LVM    | Agent Linux | RHEL 8.2 pour applications métiers SAP. Image de 2e génération.
RHEL-HA       | 7.4       | LVM    | Agent Linux | RHEL 7.4 avec add-on HA. Facture une prime pour HA et RHEL en plus des frais de calcul de base. Obsolète, privilégier les images RHEL-SAP-HA.
|             | 7.5       | LVM    | Agent Linux | RHEL 7.5 avec add-on HA. Facture une prime pour HA et RHEL en plus des frais de calcul de base. Obsolète, privilégier les images RHEL-SAP-HA.
|             | 7.6       | LVM    | Agent Linux | RHEL 7.6 avec add-on HA. Facture une prime pour HA et RHEL en plus des frais de calcul de base. Obsolète, privilégier les images RHEL-SAP-HA.
RHEL-SAP-HA   | 7.4          | LVM    | Agent Linux | RHEL 7.4 for SAP avec services de haute disponibilité et de mise à jour. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 74sapha-gen2 | LVM    | Agent Linux | RHEL 7.4 for SAP avec services de haute disponibilité et de mise à jour. Image de 2e génération. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 7.5          | LVM    | Agent Linux | RHEL 7.5 for SAP avec services de haute disponibilité et de mise à jour. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 7.6          | LVM    | Agent Linux | RHEL 7.6 for SAP avec services de haute disponibilité et de mise à jour. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 76sapha-gen2 | LVM    | Agent Linux | RHEL 7.6 for SAP avec services de haute disponibilité et de mise à jour. Image de 2e génération. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 7,7          | LVM    | Agent Linux | RHEL 7.7 for SAP avec services de haute disponibilité et de mise à jour. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 77sapha-gen2 | LVM    | Agent Linux | RHEL 7.7 for SAP avec services de haute disponibilité et de mise à jour. Image de 2e génération. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 8.1          | LVM    | Agent Linux | RHEL 8.1 pour SAP avec services de haute disponibilité et de mise à jour. Attachement à des dépôts E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 81sapha-gen2          | LVM    | Agent Linux | RHEL 8.1 pour SAP avec services de haute disponibilité et de mise à jour. Images de 2e génération – Attachées à des référentiels E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 8,2          | LVM    | Agent Linux | RHEL 8.2 pour SAP avec services de haute disponibilité et de mise à jour. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
|             | 82sapha-gen2          | LVM    | Agent Linux | RHEL 8.2 pour SAP avec services de haute disponibilité et de mise à jour. Images de 2e génération – Attachées à des référentiels E4S. Facture une prime pour SAP et HA ainsi que RHEL en plus des frais de calcul de base.
rhel-byos     |rhel-lvm74| LVM    | Agent Linux | Les images BYOS RHEL 7.4, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées pour RHEL Premium.
|             |rhel-lvm75| LVM    | Agent Linux | Les images BYOS RHEL 7.5, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées pour RHEL Premium.
|             |rhel-lvm76| LVM    | Agent Linux | Les images BYOS RHEL 7.6, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées pour RHEL Premium.
|             |rhel-lvm76-gen2| LVM    | Agent Linux | Les images BYOS RHEL 7.6 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm77| LVM    | Agent Linux | Les images BYOS RHEL 7.7, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées pour RHEL Premium.
|             |rhel-lvm77-gen2| LVM    | Agent Linux | Les images BYOS RHEL 7.7 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm78| LVM    | Agent Linux | Les images BYOS RHEL 7.8, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm78-gen2| LVM    | Agent Linux | Les images BYOS RHEL 7.8 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm8 | LVM    | Agent Linux | Les images BYOS RHEL 8.0, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm8-gen2 | LVM    | Agent Linux | Les images BYOS RHEL 8.0 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm81 | LVM    | Agent Linux | Les images BYOS RHEL 8.1, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm81-gen2 | LVM    | Agent Linux | Les images BYOS RHEL 8.1 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm82 | LVM    | Agent Linux | Les images BYOS RHEL 8.2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.
|             |rhel-lvm82-gen2 | LVM    | Agent Linux | Les images BYOS RHEL 8.2 Génération 2, qui ne sont associées à aucune source de mises à jour, ne sont pas facturées comme RHEL Premium.

> [!NOTE]
> L’offre de produits RHEL-SAP-HANA est considérée comme en fin de vie par Red Hat. Les déploiements existants continueront de fonctionner normalement, mais Red Hat recommande aux clients d’effectuer une migration des images RHEL-SAP-HANA vers les images RHEL-SAP-HA, qui incluent les référentiels SAP HANA ainsi que le module complémentaire HA. Pour plus d’informations sur les offres cloud SAP de Red Hat, [cliquez ici](https://access.redhat.com/articles/3751271).

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [images Red Hat dans Azure](./redhat-images.md).
* En savoir plus sur l’[infrastructure RHUI (Red Hat Update Infrastructure)](./redhat-rhui.md).
* En savoir plus sur l’[offre BYOS RHEL](./byos.md).
* Vous trouverez des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL sur la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).
