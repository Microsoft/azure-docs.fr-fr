---
title: Options de groupe et de filtre dans Azure Cost Management
description: Cet article explique comment utiliser les options de groupe et de filtre dans Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 06/08/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: ac9828ca61009eb3ee39412169b2b454b9ecbd00
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131868"
---
# <a name="group-and-filter-options-in-cost-analysis"></a>Options de groupe et de filtre dans l’analyse des coûts

L’analyse des coûts dispose de nombreuses options de regroupement et de filtrage. Cet article vous aide à comprendre à quel moment les utiliser.

Pour visionner une vidéo sur les options de regroupement et de filtrage, regardez la vidéo [Cost Management - Génération de rapports en fonction des dimensions et des balises](https://www.youtube.com/watch?v=2Vx7V17zbmk). Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/2Vx7V17zbmk]

## <a name="group-and-filter-properties"></a>Propriétés de groupe et de filtre

Le tableau suivant répertorie quelques options courantes de regroupement et de filtrage disponibles dans l’analyse des coûts, et indique à quel moment les utiliser.

| Propriété | Quand l’utiliser | Notes |
| --- | --- | --- |
| **Zones de disponibilité** | Décompose les coûts AWS par zone de disponibilité. | S’applique uniquement aux étendues et aux groupes d’administration AWS. Les données Azure n’incluent pas la zone de disponibilité et s’affichent avec le libellé **Non applicable**. |
| **Période de facturation** | Décompose les coûts de paiement à l’utilisation selon le mois où ils ont été (ou seront) facturés. | Utilisez **Période de facturation** pour avoir une représentation précise des frais PAYG facturés. Incluez 2 jours supplémentaires avant et après la période de facturation si vous filtrez sur une plage de dates personnalisée. La limitation aux dates exactes de la période de facturation ne correspondra pas à la facture. Affiche les coûts de toutes les factures de la période de facturation. Utilisez **ID de facture** pour filtrer sur une facture spécifique. S’applique uniquement aux abonnements PAYG, car EA et MCA sont facturés par mois calendaires. Les comptes EA/MCA peuvent utiliser les mois du calendrier dans le sélecteur de dates ou la granularité mensuelle pour atteindre le même objectif. |
| **Type de dépense** | Pour décomposer selon le type de dépense : utilisation, achat, remboursement ou réservation non utilisée. | Les achats de réservation et les remboursements sont disponibles seulement pour les coûts réels, mais pas pour les coûts amortis. Les coûts liés aux réservations non utilisées s’affichent uniquement lors e la consultation des coûts amortis. |
| **Département** | Décompose les coûts par département EA. | Disponible uniquement pour EA et les groupes d’administration. Les abonnements avec paiement à l’utilisation n’ont pas de département et apparaissent avec le libellé **Non applicable** ou **Non affecté**. |
| **Compte d’inscription** | Pour décomposer les coûts par propriétaire de compte EA. | Disponible seulement pour les comptes, les départements et groupes d’administration EA. Les abonnements avec paiement à l’utilisation n’ont pas d’inscription EA et apparaissent avec le libellé **Non applicable** ou **Non affecté**. |
| **Fréquence** | Pour décomposer les coûts selon qu’ils sont uniques, récurrents ou basés sur l’utilisation. | |
| **ID de facture** | Décompose les coûts par facture émise. | Les frais non facturés n’ont pas encore d’ID de facture, et les coûts EA n’incluent pas les détails de facturation et s’affichent avec le libellé **Non applicable**.  |
| **Compteur** | Décompose les coûts par compteur d’utilisation. | Les achats et l’utilisation de la Place de marché s’affichent avec le libellé **Non applicable**. Reportez-vous à **Type de frais** pour identifier les achats et à **Type d’éditeur** pour identifier les frais de la Place de marché. |
| **opération** | Décompose les coûts AWS par opération. | S’applique uniquement aux étendues et aux groupes d’administration AWS. Les données Azure n’incluent pas les opérations et s’affichent avec le libellé **Non applicable** ; utilisez **Compteur** à la place. |
| **Modèle de tarification** | Décompose les coûts en fonction de l’utilisation à la demande, à la réservation ou ponctuelle. | Les achats apparaissent avec le libellé **OnDemand**. Si vous voyez **Non applicable**, regroupez par **Réservation** pour déterminer si l’utilisation est « Réservation » ou « À la demande », par **Type de frais** pour identifier les achats.
| **Fournisseur** | Pour décomposer les coûts liés à AWS et liés à Azure. | Disponible uniquement pour les groupes d’administration. |
| **Type d’éditeur** | Pour décomposer les coûts selon qu’ils proviennent d’AWS, d’Azure ou de la Place de marché. |  |
| **Réservation** | Pour décomposer les coûts par réservation. | Les utilisations ou les achats qui ne sont pas associés à une réservation s’affichent avec le libellé **Non applicable**. Regroupez par **Type d’éditeur** pour identifier les autres achats Azure, AWS ou Place de marché. |
| **Ressource** | Pour décomposer les coûts par ressource. | Les achats s’affichent avec le libellé **Non applicable**, car ils s’appliquent au niveau d’un compte de facturation EA/PAYG ou au niveau du profil de facturation MCA, et ne sont pas associés à une ressource spécifique. Regroupez par **Type d’éditeur** pour identifier les autres achats Azure, AWS ou Place de marché. |
| **Groupe de ressources** | Pour décomposer les coûts par groupe de ressources. | Les achats, les ressources de locataire non associées à des abonnements, les ressources d’abonnement non déployées sur un groupe de ressources et les ressources classiques n’ont pas de groupe de ressources, et ils apparaissent avec le libellé **Autres**, **Services classiques**, **$system** ou **Non applicable**. |
| **Type de ressource** | Pour décomposer les coûts par type de ressource. | Les achats et les services classiques n’ont pas de type de ressource Azure Resource Manager et apparaissent avec le libellé **Autres**, **Services classiques** ou **Non applicable**. |
| **Emplacement de la ressource** | Décompose les coûts par emplacement ou par région. | Les achats et l’utilisation de la Place de marché peuvent apparaître avec le libellé **Non affecté**, **Inconnu**, **Non mappé** ou **Non applicable**. |
| **Nom du service** ou **Catégorie du compteur** | Pour décomposer les coûts par service Azure. | Les achats et l’utilisation de la Place de marché apparaissent avec le libellé **Non applicable** ou **Non affecté**. |
| **Niveau de service** ou **Sous-catégorie de compteur** | Pour décomposer les coûts par sous-classification du compteur d’utilisation Azure. | Les achats et l’utilisation de la Place de marché apparaissent avec le libellé **Non applicable** ou **Non affecté**. |
| **Abonnement** | Décompose les coûts par abonnement Azure et par compte AWS lié. | Les achats et les ressources des locataires peuvent apparaître avec le libellé **Non applicable**. |
| **Tag** | Pour décomposer les coûts par valeur d’étiquette pour une clé d’étiquette spécifique. | Les étiquettes ne sont pas disponibles pour les achats, les ressources des locataires qui ne sont pas associées à des abonnements, les ressources d’abonnement non déployées sur un groupe de ressources et les ressources classiques. Certains services n’incluent pas d’étiquettes dans les données d’utilisation. Pour plus d’informations, consultez [Prise en charge des étiquettes pour chaque type de ressource](../../azure-resource-manager/management/tag-support.md). |

Pour plus d’informations sur ces termes, consultez [Comprendre les termes utilisés dans le fichier Utilisation et frais Azure](../understand/understand-usage.md).

## <a name="next-steps"></a>Étapes suivantes

- [Démarrer l’analyse des coûts](./quick-acm-cost-analysis.md)