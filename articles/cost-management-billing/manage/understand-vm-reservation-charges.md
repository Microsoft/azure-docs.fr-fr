---
title: Comprendre la remise sur Azure Reserved VM Instances
description: Découvrez comment la remise d’instance de machine virtuelle réservée Azure est appliquée aux machines virtuelles en cours d’exécution.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: bda93712dd9a7501fbfddf0e75b8ae3c0088ed55
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341238"
---
# <a name="how-the-azure-reservation-discount-is-applied-to-virtual-machines"></a>Comment la remise de réservation Azure est-elle appliquée aux machines virtuelles ?

Quand vous achetez une instance de machine virtuelle réservée Azure, la remise de réservation est automatiquement appliquée aux machines virtuelles qui correspondent aux attributs et à la quantité de la réservation. Une réservation couvre les coûts de calcul de vos machines virtuelles.

Une remise de réservation s’applique aux machines virtuelles de base que vous achetez sur la Place de marché Azure.

Pour la capacité réservée SQL Database, consultez [Comprendre la remise des instances réservées Azure](../reservations/understand-reservation-charges.md).

Le tableau suivant montre les coûts de votre machine virtuelle après l’achat d’une instance de machine virtuelle réservée. Dans tous les cas, les frais de stockage et de mise en réseau vous sont facturés aux tarifs normaux.

| Type de machine virtuelle  | Frais avec instance de machine virtuelle réservée |
|-----------------------|--------------------------------------------|
|Machines virtuelles Linux sans logiciel supplémentaire | La réservation couvre les coûts d’infrastructure des machines virtuelles.|
|Machines virtuelles Linux avec frais de logiciel (par exemple, Red Hat) | La réservation couvre les coûts d’infrastructure. Les logiciels supplémentaires vous sont facturés.|
|Machines virtuelles Windows sans logiciel supplémentaire |La réservation couvre les coûts d’infrastructure. Les logiciels Windows vous sont facturés.|
|Machines virtuelles Windows avec logiciels supplémentaires (par exemple, SQL server) | La réservation couvre les coûts d’infrastructure. Les logiciels Windows et les autres logiciels vous sont facturés.|
|Machines virtuelles Windows avec [Azure Hybrid Benefit](../../virtual-machines/windows/hybrid-use-benefit-licensing.md) | La réservation couvre les coûts d’infrastructure. Les coûts des logiciels Windows sont couverts par Azure Hybrid Benefit. Tout logiciel supplémentaire est facturé séparément.|

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de « *capacité utilisée ou perdue* ». Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n'est trouvée dans l'étendue spécifiée, les heures réservées sont *perdues*.

## <a name="reservation-discount-for-non-windows-vms"></a>Remise de réservation pour les machines virtuelles non Windows

 La remise de réservation Azure est appliquée aux instances de machine virtuelle en cours d’exécution sur une base horaire. Les réservations que vous avez achetées sont mises en correspondance avec l’utilisation émise par les machines virtuelles en cours d’exécution pour appliquer la remise de réservation. Pour les machines virtuelles qui ne peuvent pas s’exécuter pendant une heure entière, la réservation est remplie à partir d’autres machines virtuelles n’utilisant pas de réservation, incluant des machines virtuelles s’exécutant simultanément. À la fin de l’heure, l’application de réservation pour les machines virtuelles dans l’heure est verrouillée. Quand une machine virtuelle ne s’exécute pas pendant une heure ou quand des machines virtuelles s’exécutant simultanément dans l’heure ne remplissent pas l’heure de réservation, la réservation est sous-utilisée pour cette heure. Le graphique suivant illustre l’application d’une réservation à l’utilisation de machines virtuelles facturables. La représentation est basée sur l’achat d’une réservation et sur deux instances de machine virtuelle correspondantes.

![Capture d’écran montrant une réservation appliquée et deux instances de machine virtuelle correspondantes](./media/understand-vm-reservation-charges/billing-reserved-vm-instance-application.png)

1. Toute utilisation supérieure à la ligne de réservation est facturée au tarif standard de paiement à l’utilisation. Toute utilisation sous la ligne de réservation n’est pas facturée, car elle a déjà été payée dans le cadre de l’achat de la réservation.
2. Dans l’heure 1, l’instance 1 s’exécute pendant 0,75 heure, et l’instance 2 pendant 0,5 heure. L’utilisation totale de l’heure 1 est 1,25 heure. Vous payez 0,25 heure restante au tarif du paiement à l’utilisation.
3. Pour les heures 2 et 3, les deux instances se sont exécutées chacune pendant 1 heure. Une instance est couverte par la réservation, et l’autre est facturée au tarif du paiement à l’utilisation.
4. Pour l’heure 4, l’instance 1 s’exécute pendant 0,5 heure tandis que l’instance 2 s’exécute pendant 1 heure. L’instance 1 est entièrement couverte par la réservation, et la durée de 0,5 heure de l’instance 2 est couverte. Vous êtes facturé au tarif du paiement à l’utilisation pour cette durée de 0,5 heure restante.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports d’utilisation pour la facturation, voir [Comprendre l’utilisation des réservations](../reservations/understand-reserved-instance-usage-ea.md).

## <a name="reservation-discount-for-windows-vms"></a>Remise de réservation pour les machines virtuelles Windows

Quand vous exécutez des instances de machine virtuelle Windows, la réservation est appliquée pour couvrir les coûts d’infrastructure. L’application de la réservation aux coûts d’infrastructure de machine virtuelle pour les machines virtuelles Windows est identique à celle qui est appliquée pour les machines virtuelles non Windows. Les logiciels Windows vous sont facturés séparément, par processeur virtuel. Vous [Coûts des logiciels Windows avec les réservations](../reservations/reserved-instance-windows-software-costs.md). Vous pouvez supporter les coûts de licence Windows avec [Azure Hybrid Benefit pour Windows Server](../../virtual-machines/windows/hybrid-use-benefit-licensing.md).

## <a name="discount-can-apply-to-different-sizes"></a>La remise peut s'appliquer à différentes tailles

Lorsque vous achetez une instance de machine virtuelle réservée et sélectionnez **Optimisé pour la flexibilité de la taille d'instance**, la remise s'applique à la taille de machine virtuelle que vous sélectionnez. Elle peut également s'appliquer à d'autres tailles de machines virtuelles appartenant au même groupe de flexibilité de taille d'instance de série. Pour plus d’informations, consultez [Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées](../../virtual-machines/reserved-vm-instance-size-flexibility.md).

## <a name="premium-storage-vms-dont-get-non-premium-discounts"></a>Les machines virtuelles de stockage Premium ne bénéficient pas des remises non Premium.

Voici un exemple. Supposons que vous ayez acheté une réservation pour cinq machines virtuelles Standard_D1. La remise de réservation s'applique uniquement aux machines virtuelles Standard_D1 ou à d'autres machines virtuelles de la même famille d'instances. La remise ne s'applique pas aux machines virtuelles Standard_DS1 ou à d'autres tailles du groupe de flexibilité de taille d'instance DS1.

L'application de la remise de réservation ne tient pas compte du compteur utilisé pour les machines virtuelles et évalue uniquement ServiceType. Examinez la valeur `ServiceType` de `AdditionalInfo` pour déterminer les informations de série/groupe de flexibilité d'instance pour vos machines virtuelles. Les valeurs sont disponibles dans votre fichier CSV d'utilisation.

Vous ne pouvez pas modifier directement le groupe ou la série de flexibilité d'instance de la réservation après l'achat. Cependant, vous pouvez *échanger* une réservation de machine virtuelle entre un groupe ou une série de flexibilité d'instance et un(e) autre.

## <a name="services-that-get-vm-reservation-discounts"></a>Services qui obtiennent des remises de réservation de machines virtuelles

Vos réservations de machine virtuelle peuvent s’appliquer à l’utilisation de machines virtuelles émise à partir de plusieurs services, pas seulement pour vos déploiements de machine virtuelle. Les ressources qui obtiennent des remises de réservation varient en fonction du paramètre de flexibilité de la taille d’instance.

### <a name="instance-size-flexibility-setting"></a>Paramètre de flexibilité de la taille d’instance

Le paramètre de flexibilité de la taille d’instance détermine les services qui obtiennent les remises d’instance réservée.

Que le paramètre soit activé ou désactivé, les remises de réservation s’appliquent automatiquement à toute utilisation de machine virtuelle correspondante lorsque la valeur *ConsumedService* est `Microsoft.Compute`. Par conséquent, vérifiez vos données d’utilisation pour la valeur *ConsumedService*. Voici quelques exemples :

- Machines virtuelles
- Groupes identiques de machines virtuelles
- Service de conteneur
- Déploiements Azure Batch (en mode Abonnement utilisateur)
- Azure Kubernetes Service (AKS)
- Service Fabric

Lorsque le paramètre est activé, les remises de réservation s’appliquent automatiquement à l’utilisation de la machine virtuelle correspondante lorsque la valeur *ConsumedService* est l’un des éléments suivants :

- Microsoft.Compute
- Microsoft.ClassicCompute
- Microsoft.Batch
- Microsoft.MachineLearningServices
- Microsoft.Kusto

Vérifiez la valeur *ConsumedService* dans vos données d’utilisation pour déterminer si l’utilisation est éligible pour les remises de réservation.

Pour plus d’informations à propos de la flexibilité de la taille d’instance, consultez [Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées](../../virtual-machines/reserved-vm-instance-size-flexibility.md).


## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](../reservations/save-compute-costs-reservations.md)
- [Prépayer des machines virtuelles avec des instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
- [Prépayer des ressources de calcul SQL Database avec une capacité réservée Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Gérer les réservations pour Azure](../reservations/manage-reserved-vm-instance.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../reservations/understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../reservations/understand-reserved-instance-usage-ea.md)
- [Comprendre l’utilisation d’une réservations pour les abonnements CSP](/partner-center/azure-reservations)
- [Coûts des logiciels Windows non inclus dans les réservations](../reservations/reserved-instance-windows-software-costs.md)