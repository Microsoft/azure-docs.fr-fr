---
title: Présentation de l’utilisation des machines virtuelles Azure
description: Présentation des détails relatifs à l’utilisation des machines virtuelles
services: virtual-machines
documentationcenter: ''
author: mimckitt
ms.author: mimckitt
ms.service: virtual-machines-linux
ms.topic: how-to
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 07/28/2020
ms.openlocfilehash: b845d547224fb173d2a4b156575778783e0281fa
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488563"
---
# <a name="understanding-azure-virtual-machine-usage"></a>Présentation de l’utilisation des machines virtuelles Azure
L’analyse de vos données d’utilisation Azure vous permet d’en savoir plus sur votre consommation, et ainsi d’optimiser la gestion et l’allocation des coûts au sein de votre organisation. Ce document offre une immersion dans les détails de votre consommation de Calcul Azure. Pour plus d’informations sur l’utilisation générale d’Azure, consultez [Présentation de votre facture](../cost-management-billing/understand/review-individual-bill.md).

## <a name="download-your-usage-details"></a>Télécharger vos détails d’utilisation
Pour commencer, [téléchargez vos détails d’utilisation](../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal). Le tableau ci-dessous fournit la définition et des exemples de valeurs d’utilisation pour des machines virtuelles déployées par le biais d’Azure Resource Manager. Il ne contient pas d’informations détaillées sur les machines virtuelles déployées par le biais de notre modèle classique.


| Champ | Signification | Exemples de valeurs | 
|---|---|---|
| Date d'utilisation | Date à laquelle la ressource a été utilisée. | `11/23/2017` |
| ID du compteur | Service de niveau supérieur dont cette utilisation relève.| `Virtual Machines`|
| Sous-catégorie du compteur | Identificateur du compteur facturé. <br><br> Pour l’utilisation du service Heures de calcul, il existe un compteur pour chaque taille de machine virtuelle + système d’exploitation (Windows, non Windows) + région. <br><br> Pour l’utilisation des logiciels Premium, il existe un compteur pour chaque type de logiciel. La plupart des images de logiciels Premium ont des paramètres différents pour chaque taille élémentaire. Pour plus d’informations, consultez la page [Tarifs du calcul](https://azure.microsoft.com/pricing/details/virtual-machines/).</li></ul>| `2005544f-659d-49c9-9094-8e0aea1be3a5`|
| Nom du compteur| Il est propre à chaque service dans Azure. Pour le service de calcul, il s’agit toujours de « Heures de calcul ».| `Compute Hours`|
| Région du compteur| Identifie l’emplacement du centre de données pour certains services dont le prix est basé sur cet emplacement.|  `JA East`|
| Unité| Identifie l’unité dans laquelle le service est facturé. Les ressources de calcul sont facturées par heure.| `Hours`|
| Consommé| Quantité de la ressource consommée ce jour-là. Pour le calcul, nous facturons chaque minute d’exécution de la machine virtuelle pour une heure donnée (jusqu’à six décimales de précision).| `1, 0.5`|
| Emplacement de la ressource  | Identifie le centre de données dans lequel la ressource est en cours d’exécution.| `JA East`|
| Service consommé | Service de plateforme Azure que vous avez utilisé.| `Microsoft.Compute`|
| Groupe de ressources | Groupe de ressources dans lequel la ressource déployée est en cours d’exécution. Pour plus d’informations, consultez [Présentation d’Azure Resource Manager.](../azure-resource-manager/management/overview.md)|`MyRG`|
| ID de l’instance | Identificateur pour la ressource. L’identificateur contient le nom que vous avez spécifié pour la ressource lors de sa création. Pour les machines virtuelles, l’ID de l’instance contient l’ID d’abonnement, le nom du groupe de ressources et le nom de la machine virtuelle (ou le nom du groupe identique en cas d’utilisation d’un groupe de machines virtuelles identiques).| `/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM1`<br><br>or<br><br>`/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachineScaleSets/MyVMSS1`|
| Balises| Balise que vous affectez à la ressource. Utilisez des balises pour regrouper les enregistrements de facturation. Découvrez comment baliser vos machines virtuelles à l’aide de [CLI](./tag-cli.md) ou de [PowerShell](./tag-portal.md). Disponible uniquement pour les machines virtuelles Resource Manager.| `{"myDepartment":"RD","myUser":"myName"}`|
| Informations supplémentaires | Métadonnées relatives au service. Pour les machines virtuelles, nous renseignons les données suivantes dans le champ d’informations supplémentaires : <br><br> Type d’image : image spécifique que vous avez exécutée. Vous trouverez la liste complète des chaînes prises en charge dans la section Type d’image ci-dessous.<br><br> Type de service : taille que vous avez déployée.<br><br> VMName : nom de votre machine virtuelle. Ce champ est renseigné uniquement pour des machines virtuelles appartenant à un groupe identique. Si vous avez besoin du nom d’une machine virtuelle appartenant à un groupe identique, vous le trouverez dans la chaîne ID de l’instance ci-dessus.<br><br> UsageType : spécifie le type d’utilisation représenté.<br><br> ComputeHR est l’utilisation des Heures de calcul pour la machine virtuelle sous-jacente, comme Standard_D1_v2.<br><br> ComputeHR_SW correspond aux frais logiciels Premium si la machine virtuelle utilisent des logiciels Premium, comme Microsoft R Server. | Machines Virtuelles<br>`{"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR"}`<br><br>Virtual Machine Scale Sets<br> `{"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"myVM1", "UsageType":"ComputeHR"}`<br><br>Logiciel Premium<br> `{"ImageType":"","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR_SW"}` |

## <a name="image-type"></a>Type d’image
Pour certaines images de la galerie Azure, le type d’image est renseigné dans le champ Informations supplémentaires. Cela permet aux utilisateurs de comprendre et de suivre ce qu’ils ont déployé sur leur machine virtuelle. Les valeurs suivantes sont renseignées dans ce champ en fonction de l’image que vous avez déployée :
- BitRock 
- Canonical FreeBSD 
- Open Logic 
- Oracle 
- SLES pour SAP 
- SQL Server 14 Preview sur Windows Server 2012 R2 Preview 
- SUSE
- SUSE Premium
- StorSimple Cloud Appliance 
- Red Hat
- Red Hat for SAP Business Applications     
- Red Hat for SAP HANA 
- Windows Client BYOL 
- Windows Server BYOL 
- Windows Server Preview 

## <a name="service-type"></a>Type de service
Le champ Type de service dans le champ Informations supplémentaires correspond à la taille exacte de machine virtuelle que vous avez déployée. Les machines virtuelles à stockage Premium (basées sur SSD) et celles à stockage non-Premium (basées sur disque dur) sont facturées de manière identique. Si vous déployez une taille de type SSD, par exemple Standard\_DS2\_v2, la taille sans SSD (`Standard\_D2\_v2 VM`) apparaît dans la colonne Sous-catégorie du compteur et la taille avec SSD (`Standard\_DS2\_v2`) dans le champ Informations complémentaires.

## <a name="region-names"></a>Noms de régions
Le nom de la région renseigné dans le champ Emplacement de la ressource dans les détails d’utilisation diffère du nom de la région utilisé dans Azure Resource Manager. Voici un mappage entre les valeurs de région :

| **Nom de région dans Resource Manager** | **Emplacement de la ressource dans les détails d’utilisation** |
|---|---|
| australiaeast |Australie Est|
| australiasoutheast | Australie Sud-Est|
| brazilsouth | Brésil Sud|
| CanadaCentral | Canada Centre|
| CanadaEast | Canada Est|
| CentralIndia | Inde Centre|
| centralus | USA Centre|
| chinaeast | Chine orientale|
| chinanorth | Chine du Nord|
| eastasia | Asie Est|
| eastus | USA Est|
| eastus2 | USA Est 2|
| GermanyCentral | Allemagne - Centre|
| GermanyNortheast | Allemagne - Nord-Est|
| japaneast | Japon Est|
| japanwest | Japon Ouest|
| KoreaCentral | Corée Centre|
| KoreaSouth | Corée Sud|
| northcentralus | Centre-Nord des États-Unis|
| northeurope | Europe Nord|
| southcentralus | États-Unis - partie centrale méridionale|
| southeastasia | Asie Sud-Est|
| SouthIndia | Inde Sud|
| UKNorth | Nord du Royaume-Uni|
| uksouth | Sud du Royaume-Uni|
| UKSouth2 | Sud du Royaume-Uni 2|
| ukwest | Ouest du Royaume-Uni|
| USDoDCentral | Centre des États-Unis – US DoD|
| USDoDEast | Est des États-Unis – US DoD|
| USGovArizona | Gouvernement des États-Unis - Arizona|
| usgoviowa | USGov Iowa|
| USGovTexas | Gouvernement des États-Unis - Texas|
| usgovvirginia | USGov Virginia|
| westcentralus | USA Centre-Ouest|
| westeurope | Europe Ouest|
| WestIndia | Inde Ouest|
| westus | USA Ouest|
| westus2 | USA Ouest 2|


## <a name="virtual-machine-usage-faq"></a>Forum aux questions sur l’utilisation des machines virtuelles
### <a name="what-resources-are-charged-when-deploying-a-vm"></a>Quelles sont les ressources facturées lors du déploiement d’une machine virtuelle ?    
Les machines virtuelles entraînent des coûts pour la machine virtuelle elle-même, tout logiciel Premium en cours d’exécution sur la machine virtuelle, le compte de stockage/disque managé associé à la machine virtuelle, et les transferts de bande passante réseau à partir de la machine virtuelle.
### <a name="how-can-i-tell-if-a-vm-is-using-azure-hybrid-benefit-in-the-usage-csv"></a>Comment savoir si une machine virtuelle utilise Azure Hybrid Benefit dans le fichier CSV d’utilisation ?
Si vous déployez à l’aide [d’Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/), le taux de machine virtuelle non-Windows vous est facturé, car vous apportez votre propre licence dans le cloud. Sur votre facture, vous pouvez distinguer les machines virtuelles Resource Manager qui exécutent Azure Hybrid Benefit, car elles comportent la mention « Windows\_Server BYOL » ou « Windows\_Client BYOL » dans la colonne ImageType.
### <a name="how-are-basic-vs-standard-vm-types-differentiated-in-the-usage-csv"></a>Comment les types de machines virtuelles De base et Standard sont-ils différenciés dans le fichier CSV d’utilisation ?
Les machines virtuelles de série A De base et Standard sont toutes deux proposées. Si vous déployez une machine virtuelle De base, Sous-catégorie du compteur contient la chaîne « De base ». Si vous déployez une machine virtuelle Standard de série A, la taille de machine virtuelle affichée est « A1 VM », car Standard est la valeur par défaut. Pour en savoir plus sur les différences entre les machines virtuelles De base et Standard, consultez la [Page de tarification](https://azure.microsoft.com/pricing/details/virtual-machines/).
### <a name="what-are-extrasmall-small-medium-large-and-extralarge-sizes"></a>À quoi correspondent les tailles ExtraSmall, Small, Medium, Large et ExtraLarge ?
Les tailles ExtraSmall - ExtraLarge sont les noms hérités pour Standard\_A0 – Standard\_A4. Dans les enregistrements d’utilisation de machines virtuelles classiques, cette convention peut être utilisée si vous avez déployé ces tailles.
### <a name="what-is-the-difference-between-meter-region-and-resource-location"></a>Quelle est la différence entre Région du compteur et Emplacement de la ressource ?
La Région du compteur est associée au compteur. Pour certains services Azure qui utilisent un seul prix pour toutes les régions, le champ Région du compteur peut être vide. Toutefois, étant donné que les machines virtuelles ont des prix dédiés par région pour Machines virtuelles, ce champ est renseigné. De même, l’Emplacement de la ressource pour Machines virtuelles est l’emplacement où la machine virtuelle est déployée. Les régions Azure dans les deux champs sont identiques, même si leurs noms peuvent être régis par des conventions de chaîne différentes.
### <a name="why-is-the-imagetype-value-blank-in-the-additional-info-field"></a>Pourquoi le champ ImageType est-il vide dans Informations supplémentaires ?
Le champ ImageType est renseigné uniquement pour un sous-ensemble d’images. Si vous n’avez pas déployé l’une des images ci-dessus, ImageType est vide.
### <a name="why-is-the-vmname-blank-in-the-additional-info"></a>Pourquoi le champ VMName est-il vide dans Informations supplémentaires ?
Le champ VMName est renseigné dans le champ Informations supplémentaires uniquement pour les machines virtuelles appartenant à un groupe identique. Le champ ID de l’instance contient le nom de la machine virtuelle pour les machines virtuelles qui ne font pas partie d’un groupe identique.
### <a name="what-does-computehr-mean-in-the-usagetype-field-in-the-additional-info"></a>Que signifie ComputeHR dans le champ UsageType dans Informations supplémentaires ?
ComputeHR signifie Heures de calcul, et représente l’événement d’utilisation pour le coût d’infrastructure sous-jacent. Si UsageType est ComputeHR\_SW, l’événement d’utilisation représente les frais de logiciels Premium pour la machine virtuelle.
### <a name="how-do-i-know-if-i-am-charged-for-premium-software"></a>Comment savoir si des logiciels Premium me sont facturés ?
Quand vous recherchez l’image de machine virtuelle qui convient le mieux à vos besoins, n’oubliez pas de consulter la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute). Pour chaque image, le taux du plan logiciel applicable est indiqué. Si le taux porte la mention « Gratuit », cela signifie qu’il n’y a aucun coût supplémentaire lié au logiciel. 
### <a name="what-is-the-difference-between-microsoftclassiccompute-and-microsoftcompute-in-the-consumed-service"></a>Quelle est la différence entre Microsoft.ClassicCompute et Microsoft.Compute dans le service Consommé ?
Microsoft.ClassicCompute représente les ressources classiques déployées par le biais d’Azure Service Manager. Si vous déployez par le biais de Resource Manager, Microsoft.Compute est renseigné dans le service Consommé. Pour en savoir plus sur les modèles de déploiement Azure, consultez [cet article](../azure-resource-manager/management/deployment-models.md).
### <a name="why-is-the-instanceid-field-blank-for-my-virtual-machine-usage"></a>Pourquoi le champ ID de l’instance est-il vide pour l’utilisation de ma machine virtuelle ?
Si vous déployez par le biais du modèle de déploiement classique, la chaîne ID de l’instance n’est pas disponible.
### <a name="why-are-the-tags-for-my-vms-not-flowing-to-the-usage-details"></a>Pourquoi les balises pour mes machines virtuelles ne figurent-elles pas dans les détails d’utilisation ?
Les balises figurent uniquement dans le fichier CSV d’utilisation pour les machines virtuelles Resource Manager. Les balises de ressources classiques ne sont pas disponibles dans les détails d’utilisation.
### <a name="how-can-the-consumed-quantity-be-more-than-24-hours-one-day"></a>Comment la quantité consommée peut-elle être supérieure à 24 heures en une journée ?
Dans le modèle classique, la facturation des ressources est agrégée au niveau du service cloud. Si vous avez plusieurs machines virtuelles dans un service cloud qui utilise le même compteur de facturation, l’utilisation est agrégée. Les machines virtuelles déployées par le biais de Resource Manager étant facturées au niveau de la machine virtuelle, cette agrégation ne s’appliquera pas.
### <a name="why-is-pricing-not-available-for-dsfsgsls-sizes-on-the-pricing-page"></a>Pourquoi la tarification n’est-elle pas disponible pour les tailles DS/ADFS/GS/LS dans la page de facturation ?
Les machines virtuelles compatibles avec le stockage Premium sont facturées au même taux que les machines virtuelles compatibles avec le stockage non-Premium. Seuls vos coûts de stockage diffèrent. Pour plus d’informations, consultez la [page de tarification du stockage](https://azure.microsoft.com/pricing/details/storage/unmanaged-disks/).

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les détails d’utilisation, consultez [Comprendre votre facture Microsoft Azure.](../cost-management-billing/understand/review-individual-bill.md)