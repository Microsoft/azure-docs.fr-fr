---
title: Tailles des machines virtuelles pour Azure Cloud services | Microsoft Docs
description: Répertorie les différentes tailles de machines virtuelles (et les identifiants) des rôles web et de travail des services cloud Azure.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: b617ba8db8f9dbb1c25ac34aa879613c29c723b7
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340286"
---
# <a name="sizes-for-cloud-services"></a>Tailles de services cloud
Cette rubrique décrit les tailles et options disponibles pour les instances de rôle de Cloud Services (rôles web et rôles de travail). Il expose également les points à prendre en considération pour le déploiement quand vous planifiez l'utilisation de ces ressources. Chaque taille a un identifiant que vous placez dans votre [fichier de définition de service](cloud-services-model-and-package.md#csdef). Les prix pour chaque taille sont disponibles sur la page de [tarification des Services Cloud](https://azure.microsoft.com/pricing/details/cloud-services/).

> [!NOTE]
> Pour connaître les limites d'Azure associées, consultez l'article [Abonnement Azure et limites, quotas et contraintes du service](../azure-resource-manager/management/azure-subscription-service-limits.md)
>
>

## <a name="sizes-for-web-and-worker-role-instances"></a>Tailles pour les instances de rôle web et de travail
Azure offre le choix entre plusieurs tailles standard. Voici des considérations quant à certaines de ces tailles :

* Les machines virtuelles de la série D sont conçues pour exécuter des applications qui nécessitent une puissance de calcul et des performances de disque temporaire supérieures. Ces machines virtuelles se caractérisent par des processeurs plus rapides, un rapport mémoire-cœur plus élevé et un disque SSD pour le disque temporaire. Pour plus d’informations, voir l’annonce suivante sur le blog Azure : [Nouvelles tailles de machines virtuelles de la série D](https://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes/)(en anglais).
* Les séries Dv3 et Dv2, suites de la série D d’origine, comprennent un processeur plus puissant. Le processeur de la série Dv2 est environ 35 % plus rapide que le processeur de la série D. Il est basé sur la dernière génération de processeur 2,4 GHz Intel Xeon® E5-2673 v3 (Haswell) et comporte la technologie 2.0 Intel Turbo Boost, et peut atteindre 3,1 GHz. La série Dv2 a les mêmes configurations de disque et de mémoire que la série D.
* Les machines virtuelles de la série G offrent le plus de mémoire et s’exécutent sur des hôtes équipés de processeurs de la famille Intel Xeon E5 V3.
* Les machines virtuelles de la série A peuvent être déployées sur différents types de matériels et processeurs. La taille est limitée en fonction du matériel, pour offrir des performances de processeur cohérentes pour l’instance en cours d’exécution, quel que soit le matériel sur lequel elle est déployée. Pour déterminer le matériel physique sur lequel cette taille est déployée, interrogez le matériel virtuel à partir de la machine virtuelle.
* La taille A0 est trop sollicitée sur le matériel physique. Pour cette taille spécifique uniquement, les autres déploiements de clients peuvent affecter les performances de la charge de travail en cours d’exécution. Les performances relatives sont décrites ci-dessous comme référence attendue, soumises à une variation approximative de 15 pour cent.

La taille de la machine virtuelle a une incidence sur la tarification. La taille influe également sur les capacités de traitement, de mémoire et de stockage de la machine virtuelle. Les coûts de stockage sont calculés séparément en fonction des pages utilisées dans le compte de stockage. Pour plus d’informations, voir les pages [Tarification - Services cloud](https://azure.microsoft.com/pricing/details/cloud-services/) et [Tarification - Stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

Les considérations ci-dessous peuvent vous aider à choisir une taille :

* Les tailles A8 à A11 et celles de la série H sont également appelées *instances nécessitant beaucoup de ressources système*. Le matériel qui exécute ces tailles a été conçu et optimisé pour les applications nécessitant beaucoup de ressources système et réseau, notamment les applications en cluster pour des calculs complexes, la modélisation et les simulations. La série A8-A11 utilise un processeur Intel Xeon E5-2670 cadencé à 2,6 GHZ, et la série H un processeur Intel Xeon E5-2667 v3 cadencé à 3,2 GHz. Pour plus d’informations et pour connaître les éléments à prendre en considération sur l’utilisation de ces tailles, consultez [Tailles de machines virtuelles de calcul haute performance](../virtual-machines/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Les séries Dv3, Dv2, D et G sont idéales pour les applications qui exigent des processeurs plus rapides, de meilleures performances de disque local, ou qui ont des exigences de mémoire plus élevées. Elles offrent une combinaison puissante pour de nombreuses applications professionnelles.
* Certains hôtes physiques des centres de données Azure ne prennent pas en charge les tailles de machines virtuelles élevées, comme A5 à A11. Ainsi, vous pouvez obtenir le message d’erreur **Échec de la configuration de la machine virtuelle {nom de la machine}** ou **Échec de la création de la machine virtuelle {nom de la machine}** pendant le redimensionnement d’une machine virtuelle existante, la création d’une machine virtuelle dans un réseau virtuel créé avant le 16 avril 2013 ou l’ajout d’une nouvelle machine virtuelle à un service cloud existant. Consultez [Erreur : « Échec de la configuration de la machine virtuelle »](https://social.msdn.microsoft.com/Forums/9693f56c-fcd3-4d42-850e-5e3b56c7d6be/error-failed-to-configure-virtual-machine-with-a5-a6-or-a7-vm-size?forum=WAVirtualMachinesforWindows) (en anglais) sur le forum d’assistance pour découvrir les solutions de contournement pour chaque scénario de déploiement.
* Il se peut également que votre abonnement limite le nombre de cœurs que vous pouvez déployer dans certaines familles de taille. Pour augmenter un quota, contactez le support technique Azure.

## <a name="performance-considerations"></a>Considérations relatives aux performances
Nous avons créé le concept d’unité de calcul Azure (ACU) pour pouvoir comparer les performances de calcul (UC) des références Azure et identifier la référence qui pourra satisfaire au mieux vos besoins en matière de performances.  L’unité ACU est actuellement normalisée sur une machine virtuelle de petite taille (Standard_A1) correspondant à 100, et toutes les autres références représentent ensuite approximativement la rapidité avec laquelle la référence en question peut exécuter un test d’évaluation.

> [!IMPORTANT]
> Cette unité ACU est fournie uniquement à titre indicatif. Les résultats de votre charge de travail peuvent varier.
>
>

<br>

| Famille de références | ACU/Cœur |
| --- | --- |
| [Très petite](#a-series) |50 |
| [Petite à très grande](#a-series) |100 |
| [A5-7](#a-series) |100 |
| [A8-A11](#a-series) |225* |
| [A v2](#av2-series) |100 |
| [D](#d-series) |160 |
| [D v2](#dv2-series) |160 - 190* |
| [D v3](#dv3-series) |160 - 190* |
| [E v3](#ev3-series) |160 - 190* |
| [G](#g-series) |180 - 240* |
| [H](#h-series) |290 - 300* |

Les unités ACU signalées par un astérisque (*) utilisent la technologie Intel ® Turbo pour augmenter la fréquence du processeur et améliorer les performances. Cette amélioration des performances peut varier en fonction de la taille et de la charge de travail de la machine virtuelle, et des autres charges de travail en cours d’exécution sur le même hôte.

## <a name="size-tables"></a>Tableaux des tailles
Les tableaux ci-après indiquent les tailles et les capacités qu’elles offrent.

* La capacité de stockage est indiquée en unités de Gio ou 1 024^3 octets. Lors de la comparaison de disques mesurés en Go (1 000^3 octets) à des disques mesurés en Gio (1 024^3) n’oubliez pas que les indications de capacité en Gio peuvent sembler plus petites. Par exemple, 1 023 Gio = 1 098,4 Go
* Le débit de disque est mesuré en opérations d’entrée/sortie par seconde (IOPS) et Mbit/s  où Mbit/s  = 10^6 octets par seconde.
* Les disques de données peuvent fonctionner en mode avec ou sans mise en cache. En cas de fonctionnement du disque de données avec mise en cache, le mode de mise en cache hôte est défini sur **ReadOnly** ou **ReadWrite**. En cas de fonctionnement du disque de données sans mise en cache, le mode de mise en cache hôte est défini sur **Aucun**.
* La bande passante réseau maximale est la bande passante maximale agrégée allouée et affectée par type de machine virtuelle. La bande passante maximale fournit des recommandations pour la sélection du bon type de machine virtuelle afin de garantir une capacité réseau adéquate. Lorsque vous évoluez entre Faible, Modéré, Élevé et Très élevé, le débit augmente en conséquence. Les performances réseau réelles dépendent de nombreux facteurs, notamment les charges du réseau et de l’application, ainsi que les paramètres réseau de l’application.

## <a name="a-series"></a>Série A
| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Très petite      | 1         | 0,768        | 20                   | 1 / Faible |
| Petite           | 1         | 1,75         | 225                  | 1 / Modérée |
| Moyenne          | 2         | 3,5          | 490                  | 1 / Modérée |
| grand           | 4         | 7            | 1 000                 | 2 / Élevée |
| Très grande      | 8         | 14           | 2040                 | 4 / Élevée |
| A5              | 2         | 14           | 490                  | 1 / Modérée |
| A6              | 4         | 28           | 1 000                 | 2 / Élevée |
| A7              | 8         | 56           | 2040                 | 4 / Élevée |

## <a name="a-series---compute-intensive-instances"></a>Série A - Instances de calcul intensif
Pour plus d’informations et pour connaître les éléments à prendre en considération sur l’utilisation de ces tailles, consultez [Tailles de machines virtuelles de calcul haute performance](../virtual-machines/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| A8*             |8          | 56           | 1817                 | 2 / Élevée |
| A9*             |16         | 112          | 1817                 | 4 / Très élevée |
| A10             |8          | 56           | 1817                 | 2 / Élevée |
| A11             |16         | 112          | 1817                 | 4 / Très élevée |

\*Compatible RDMA

## <a name="av2-series"></a>Série Av2

| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_A1_v2  | 1         | 2            | 10                   | 1 / Modérée                 |
| Standard_A2_v2  | 2         | 4            | 20                   | 2 / Modérée                 |
| Standard_A4_v2  | 4         | 8            | 40                   | 4 / Élevée                     |
| Standard_A8_v2  | 8         | 16           | 80                   | 8 / Élevée                     |
| Standard_A2m_v2 | 2         | 16           | 20                   | 2 / Modérée                 |
| Standard_A4m_v2 | 4         | 32           | 40                   | 4 / Élevée                     |
| Standard_A8m_v2 | 8         | 64           | 80                   | 8 / Élevée                     |


## <a name="d-series"></a>Série D
| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| D1 standard     | 1         | 3,5          | 50                   | 1 / Modérée |
| D2 standard     | 2         | 7            | 100                  | 2 / Élevée |
| D3 standard     | 4         | 14           | 200                  | 4 / Élevée |
| D4 standard     | 8         | 28           | 400                  | 8 / Élevée |
| D11 standard    | 2         | 14           | 100                  | 2 / Élevée |
| D12 standard    | 4         | 28           | 200                  | 4 / Élevée |
| D13 standard    | 8         | 56           | 400                  | 8 / Élevée |
| D14 standard    | 16        | 112          | 800                  | 8 / Très élevée |

## <a name="dv2-series"></a>Série Dv2
| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_D1_v2  | 1         | 3,5          | 50                   | 1 / Modérée |
| Standard_D2_v2  | 2         | 7            | 100                  | 2 / Élevée |
| Standard_D3_v2  | 4         | 14           | 200                  | 4 / Élevée |
| Standard_D4_v2  | 8         | 28           | 400                  | 8 / Élevée |
| Standard_D5_v2  | 16        | 56           | 800                  | 8 / Extrêmement élevée |
| Standard_D11_v2 | 2         | 14           | 100                  | 2 / Élevée |
| Standard_D12_v2 | 4         | 28           | 200                  | 4 / Élevée |
| Standard_D13_v2 | 8         | 56           | 400                  | 8 / Élevée |
| Standard_D14_v2 | 16        | 112          | 800                  | 8 / Extrêmement élevée |
| Standard_D15_v2 | 20        | 140          | 1 000                | 8 / Extrêmement élevée |

## <a name="dv3-series"></a>Série Dv3

| Taille            | Cœurs d’unité centrale | Mémoire : Gio   | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------- | -------------------- | ---------------------------- |
| Standard_D2_v3  | 2         | 8             | 50                   | 2 / Modérée |
| Standard_D4_v3  | 4         | 16            | 100                  | 2 / Élevée |
| Standard_D8_v3  | 8         | 32            | 200                  | 4 / Élevée |
| Standard_D16_v3 | 16        | 64            | 400                  | 8 / Extrêmement élevée |
| Standard_D32_v3 | 32        | 128           | 800                  | 8 / Extrêmement élevée |
| Standard_D48_v3 | 48        | 192           | 1200                 | 8 / Extrêmement élevée |
| Standard_D64_v3 | 64        | 256           | 1 600                 | 8 / Extrêmement élevée |

## <a name="ev3-series"></a>Série Ev3

| Taille            | Cœurs d’unité centrale | Mémoire : Gio   | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------- | -------------------- | ---------------------------- |
| Standard_E2_v3  | 2         | 16            | 50                   | 2 / Modérée |
| Standard_E4_v3  | 4         | 32            | 100                  | 2 / Élevée |
| Standard_E8_v3  | 8         | 64            | 200                  | 4 / Élevée |
| Standard_E16_v3 | 16        | 128           | 400                  | 8 / Extrêmement élevée |
| Standard_E32_v3 | 32        | 256           | 800                  | 8 / Extrêmement élevée |
| Standard_E48_v3 | 48        | 384           | 1200                 | 8 / Extrêmement élevée |
| Standard_E64_v3 | 64        | 432           | 1 600                 | 8 / Extrêmement élevée |


## <a name="g-series"></a>Série G
| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_G1     | 2         | 28           | 384                  |1 / Élevée |
| Standard_G2     | 4         | 56           | 768                  |2 / Élevée |
| Standard_G3     | 8         | 112          | 1 536                |4 / Très élevée |
| Standard_G4     | 16        | 224          | 3 072                |8 / Extrêmement élevée |
| Standard_G5     | 32        | 448          | 6 144                |8 / Extrêmement élevée |

## <a name="h-series"></a>Série H
Les machines virtuelles de la série H sont des machines virtuelles de calcul haute performance de nouvelles génération, destinées à répondre à des besoins de calcul de haut niveau, par exemple en relation avec la modélisation moléculaire et la dynamique des fluides. Les machines virtuelles à 8 et 16 cœurs reposent sur la technologie de processeur Intel Haswell E5-2667 V3 avec mémoire DDR4 et stockage SSD local.

En plus de la puissance substantielle du processeur, la série H offre différentes options pour des réseaux RDMA à faible latence, en utilisant FDR InfiniBand et plusieurs configurations de mémoire pouvant satisfaire des exigences de calcul nécessitant une mémoire conséquente.

| Taille            | Cœurs d’unité centrale | Mémoire : Gio  | Stockage temporaire (SSD) : Gio       | Cartes réseau (max)/Bande passante réseau |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard_H8     | 8         | 56           | 1 000                 | 8 / Élevée |
| Standard_H16    | 16        | 112          | 2000                 | 8 / Très élevée |
| Standard_H8m    | 8         | 112          | 1 000                 | 8 / Élevée |
| Standard_H16m   | 16        | 224          | 2000                 | 8 / Très élevée |
| Standard_H16r*  | 16        | 112          | 2000                 | 8 / Très élevée |
| Standard_H16mr* | 16        | 224          | 2000                 | 8 / Très élevée |

\*Compatible RDMA

## <a name="configure-sizes-for-cloud-services"></a>Configurer les tailles pour les Cloud Services
Vous pouvez spécifier la taille de l’ordinateur virtuel d’une instance de rôle dans le cadre du modèle de service décrit par le [fichier de définition de service](cloud-services-model-and-package.md#csdef). La taille du rôle détermine le nombre de cœurs du processeur, la capacité de mémoire et la taille du système de fichiers local qui lui est allouée. Choisissez la taille du rôle en fonction des besoins en ressources de votre application.

Voici un exemple qui montre comment configurer un rôle avec la taille Standard_D2 pour une instance de rôle web :

```xml
<WorkerRole name="Worker1" vmsize="Standard_D2">
...
</WorkerRole>
```

## <a name="changing-the-size-of-an-existing-role"></a>Modification de la taille d’un rôle existant

À mesure que la nature des modifications de votre charge de travail évolue, ou si de nouvelles tailles de machine virtuelle sont disponibles, vous pouvez souhaiter modifier la taille de votre rôle. Pour ce faire, vous devez modifier la taille de machine virtuelle dans votre fichier de définition de service (comme indiqué ci-dessus), ré-empaqueter votre Service Cloud et le déployer.

>[!TIP]
> Vous devrez peut-être utiliser différentes tailles de machine virtuelle pour votre rôle dans différents environnements (par exemple : test ou production). Une manière de procéder consiste à créer plusieurs fichiers de définition de service (.csdef) dans votre projet, puis à créer différents packages de service cloud par environnement au cours de la génération automatisée à l’aide de l’outil CSPack. Pour en savoir plus sur les éléments d’un package de services cloud et comment les créer, consultez la rubrique [Qu’est-ce que le modèle Cloud Service, et comment en créer un package ?](cloud-services-model-and-package.md)
>
>

## <a name="get-a-list-of-sizes"></a>Obtenir une liste des tailles
PowerShell ou l’API REST permet d’obtenir la liste des tailles. L’API REST est présentée [ici](/previous-versions/azure/reference/dn469422(v=azure.100)). Le code suivant est une commande PowerShell qui répertorie toutes les tailles disponibles pour Cloud Services. 

```powershell
Get-AzureRoleSize | where SupportedByWebWorkerRoles -eq $true | select InstanceSize, RoleSizeLabel
```

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’ [abonnement Azure et les limites, quotas et contraintes des services](../azure-resource-manager/management/azure-subscription-service-limits.md).
* En savoir plus sur les [Tailles de machines virtuelles de calcul haute performance](../virtual-machines/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) pour les charges de travail HPC.