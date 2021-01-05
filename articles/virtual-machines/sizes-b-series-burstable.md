---
title: Série B modulable - Machines Virtuelles Azure
description: Décrit la série B des tailles de machines virtuelles Azure modulables.
services: virtual-machines
ms.subservice: sizes
author: styli365
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: sttsinar
ms.openlocfilehash: 819654ef88584cb91d6032e46256258aaed524fd
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500306"
---
# <a name="b-series-burstable-virtual-machine-sizes"></a>Tailles de machines virtuelles modulables Série B

Les machines virtuelles de la série B sont idéales pour les charges de travail ne nécessitant pas en permanence les performances complètes du processeur, comme les serveurs web, les preuves de concept, les petites bases de données et les environnements de build pour le développement. Ces charges de travail ont généralement des exigences modulables en termes de performances. La série B vous offre la possibilité d’acheter une taille de machine virtuelle aux performances de base qui génère des crédits lorsqu’elle n’utilise pas la totalité de ses performances. Dès qu’elle a cumulé des crédits, la machine virtuelle peut étendre son niveau de performance en utilisant jusqu’à 100 % du processeur virtuel lorsque l’application réclame des performances de processeur plus élevées.

La série B est disponible dans les tailles de machines virtuelles suivantes :

[Unité compute Azure (ACU)](./acu.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) : Variable*<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>
\* Les machines virtuelles de la série B sont Burstables et, par conséquent, les nombres ACU varient en fonction des charges de travail et de l’utilisation des cœurs.

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Perf. du processeur de base de machine virtuelle | Perf. du processeur max. de machine virtuelle | Crédits initiaux | Crédits cumulés/heure | Crédits cumulés max. | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS/MBps | Débit du disque non mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Standard_B1ls<sup>1</sup> | 1  | 0.5 | 4   | 5 %   | 100 %  | 30  | 3   | 72   | 2  | 200/10    | 160/10    | 2  |
| Standard_B1s              | 1  | 1   | 4   | 10 %  | 100 %  | 30  | 6   | 144  | 2  | 400/10    | 320/10    | 2  |
| Standard_B1ms             | 1  | 2   | 4   | 20%  | 100 %  | 30  | 12  | 288  | 2  | 800/10    | 640/10    | 2  |
| Standard_B2s              | 2  | 4   | 8   | 40%  | 200%  | 60  | 24  | 576  | 4  | 1600/15   | 1280/15   | 3  |
| Standard_B2ms             | 2  | 8   | 16  | 60%  | 200%  | 60  | 36  | 864  | 4  | 2400/22.5 | 1920/22.5 | 3  |
| Standard_B4ms             | 4  | 16  | 32  | 90%  | 400%  | 120 | 54  | 1296 | 8  | 3600/35   | 2880/35   | 4  |
| Standard_B8ms             | 8  | 32  | 64  | 135% | 800%  | 240 | 81  | 1944 | 16 | 4320/50   | 4320/50   | 4  |
| Standard_B12ms            | 12 | 48  | 96  | 202 % | 1200 % | 360 | 121 | 2909 | 16 | 6480/75   | 4320/50   | 6  |
| Standard_B16ms            | 16 | 64  | 128 | 270 % | 1600 % | 480 | 162 | 3888 | 32 | 8640/100  | 4320/50   | 8  |
| Standard_B20ms            | 20 | 80  | 160 | 337 % | 2000 % | 600 | 203 | 4860 | 32 | 10800/125 | 4320/50   | 8  |

<sup>1</sup> B1ls est pris en charge uniquement par Linux

## <a name="workload-example"></a>Exemple de charge de travail

Envisagez d’introduire une application de vérification des arrivées et sorties du bureau. L’application doit pouvoir utiliser l’UC en rafale pendant les heures ouvrables, mais n’aura pas besoin d’une grande puissance de calcul pendant les heures creuses. Dans cet exemple, la charge de travail nécessite une machine virtuelle de 16 processeurs virtuels avec 64 Gio de RAM pour fonctionner efficacement.

Le tableau affiche les données du trafic horaire et le graphique est une représentation visuelle de ce trafic.

Caractéristiques B16 :

Performances max. de l’UC : 16 processeurs virtuels * 100 % = 1 600 %

Performances de base : 270 %

![Graphique des données du trafic horaire](./media/b-series-burstable/office-workload.png)

| Scénario | Temps | Utilisation de l’UC (%) | Crédits accumulés<sup>1</sup> | Crédits disponibles |
| --- | --- | --- | --- | --- |
| Déploiement B16ms | Déploiement | Déploiement  | 480 (Crédits initiaux) | 480 |
| Aucun trafic | 0:00 | 0 | 162 | 642 |
| Aucun trafic | 1:00 | 0 | 162 | 804 |
| Aucun trafic | 2:00 | 0 | 162 | 966 |
| Aucun trafic | 3:00 | 0 | 162 | 1128 |
| Aucun trafic | 4:00 | 0 | 162 | 1290 |
| Aucun trafic | 5:00 | 0 | 162 | 1452 |
| Trafic faible | 6:00 | 270 | 0 | 1452 |
| Les employés arrivent au bureau (l’application a besoin de 80 % de ses processeurs virtuels) | 7:00 | 1 280 | -606 | 846 |
| Les employés continuent d’arriver au bureau (l’application a besoin de 80 % de ses processeurs virtuels) | 8:00 | 1 280 | -606 | 240 |
| Trafic faible | 9:00 | 270 | 0 | 240 |
| Trafic faible | 10:00 | 100 | 102 | 342 |
| Trafic faible | 11:00 | 50 | 132 | 474 |
| Trafic faible | 12:00 | 100 | 102 | 576 |
| Trafic faible | 13:00 | 100 | 102 | 678 |
| Trafic faible | 14:00 | 50 | 132 | 810 |
| Trafic faible | 15:00 | 100 | 102 | 912 |
| Trafic faible | 16:00 | 100 | 102 | 1014 |
| Les employés quittent le bureau (l’application a besoin de 100 % de ses processeurs virtuels) | 17:00 | 1 600 | -798 | 216 |
| Trafic faible | 18:00 | 270 | 0 | 216 |
| Trafic faible | 19:00 | 270 | 0 | 216 |
| Trafic faible | 20:00 | 50 | 132 | 348 |
| Trafic faible | 21:00 | 50 | 132 | 480 |
| Aucun trafic | 22:00 | 0 | 162 | 642 |
| Aucun trafic | 23:00 | 0 | 162 | 804 |

<sup>1</sup> Crédits accumulées/crédits utilisés pendant une heure est égal à : `((Base CPU perf of VM - CPU Usage) / 100) * 60 minutes`.  

Pour un D16s_v3 disposant de 16 processeurs virtuels et de 64 Gio de mémoire, le taux horaire est 0,936 $ par heure (673,92 $ par mois). Pour le B16ms avec 16 processeurs virtuels et de 64 Gio de mémoire, le taux est de 0,794 $ par heure (547,86 $ par mois). <b> Vous réduisez les coûts de 15 % !</b>

## <a name="q--a"></a>Questions et réponses

### <a name="q-what-happens-when-my-credits-run-out"></a>Q : Que se passe-t-il si je n’ai plus de crédits ?
**R** : Une fois les crédits épuisés, la machine virtuelle revient aux performances de ligne de base.

### <a name="q-how-do-you-get-135-baseline-performance-from-a-vm"></a>Q : Comment obtenir 135 % des performances de base d’une machine virtuelle ?

**R** : Ce pourcentage est réparti sur les huit processeurs virtuels qui composent la taille de la machine virtuelle. Par exemple, si votre application utilise 4 des 8 cœurs travaillant au traitement par lots et que chacun de ces 4 processeurs virtuels est utilisé à 30 %, la quantité totale des performances du processeur de la machine virtuelle serait égale à 120 %.  Ce qui signifie que votre machine virtuelle générerait un crédit temps basé sur le delta de 15 % à partir de vos performances de base.  Cela signifie également que lorsque vous disposez de crédits, cette même machine virtuelle peut utiliser la totalité des 8 processeurs virtuels pour obtenir une performance de processeur maximale de 800 %.

### <a name="q-how-can-i-monitor-my-credit-balance-and-consumption"></a>Q : Comment puis-je surveiller mon solde créditeur et ma consommation ?

**R** : La métrique **Credit** vous permet d’afficher le nombre de crédits cumulés par votre machine virtuelle et la métrique **ConsumedCredit** le nombre de crédits de processeur consommés par votre machine virtuelle.    Ces mesures figurent sur le volet des mesures du portail ou sont visibles par programme via les API Azure Monitor.

Pour en savoir plus sur l’accès aux données de mesure pour Azure, consultez [Vue d’ensemble des mesures dans Microsoft Azure](../azure-monitor/platform/data-platform.md).

### <a name="q-how-are-credits-accumulated-and-consumed"></a>Q : Comment les crédits sont-ils accumulés et consommés ?

**R** : Les taux de cumul et d’utilisation de la machine virtuelle sont définis pour qu’une machine virtuelle s’exécutant exactement à son niveau de performances de base ne génère aucun cumul net ou n’utilise aucun crédit.  Une machine virtuelle connaît une augmentation nette de ses crédits chaque fois qu’elle s’exécute sous son niveau de performances de base, et une diminution nette de ses crédits chaque fois qu’elle utilise le processeur à un niveau plus élevé de performances.

**Exemple** :  Je déploie une machine virtuelle à l’aide de la taille B1ms pour ma petite application de base de données de pointage des présences. Cette taille permet à mon application d’utiliser jusqu’à 20 % d’un processeur virtuel qui est considéré comme étant ma base, soit 0,2 crédit par minute utilisable ou cumulable.

Mon application est occupée en début et fin de journée de travail de mes employés, soit entre 7 h 00 et 9 h 00 et 16 h 00 et 18 h 00. Pendant les 20 heures restantes de la journée, mon application est en général en veille et n’utilise que 10 % du processeur virtuel. Pendant les heures creuses, je cumule 0,2 crédit par minute et utilise uniquement 0,1 crédit par minute. Ainsi, ma machine virtuelle cumule 0,1 x 60, soit 6 crédits par heure.  Pendant les 20 heures creuses, je cumule 120 crédits.  

Pendant les heures de pointe, mon application utilise en moyenne 60 % du processeur virtuel. Je cumule toujours 0,2 crédit par minute, mais j’utilise 0,6 crédit par minute pour un coût net de 0,4 crédit par minute ou 0,4 x 60 = 24 crédits par heure. Je dispose de 4 heures d’utilisation de pointe par jour, ce qui coûte 4 x 24 = 96 crédits.

Si je prends les 120 crédits cumulés lors des heures creuses et que je soustrais les 96 crédits utilisés pendant mes heures de pointe, je cumule 24 crédits supplémentaires par jour utilisables pour d’autres pics d’activité.

### <a name="q-how-can-i-calculate-credits-accumulated-and-used"></a>Q : Comment puis-je calculer les crédits accumulés et utilisés ?

**R** : Vous pouvez utiliser les formules suivantes :

(Perf. de base de l’UC de la machine virtuelle - utilisation de l’UC) / 100 = accumulation ou utilisation de crédits par minute

Par exemple, dans l’exemple ci-dessus, vos performances de base sont de 20 %. Si vous utilisez 10 % de l’UC, vous accumulez (20 % - 10 %) / 100 = 0,1 crédit par minute.

### <a name="q-does-the-b-series-support-premium-storage-data-disks"></a>Q : La série B prend-elle en charge les disques de données de stockage Premium ?

**R** : Oui, toutes les tailles de série B prennent en charge les disques de données de stockage Premium.

### <a name="q-why-is-my-remaining-credit-set-to-0-after-a-redeploy-or-a-stopstart"></a>Q : Pourquoi mon crédit restant est-il défini sur 0 après un redéploiement ou un arrêt/démarrage ?

**R** : Lorsqu’une machine virtuelle est « REDÉPLOYÉE » et qu’elle est déplacée vers un autre nœud, le crédit cumulé est perdu. Si la machine virtuelle est arrêtée/démarrée, mais reste sur le même nœud, la machine virtuelle conserve le crédit cumulé. Lorsque la machine virtuelle redémarre à zéro sur un nœud, elle obtient un crédit initial (240 pour Standard_B8ms).

### <a name="q-what-happens-if-i-deploy-an-unsupported-os-image-on-b1ls"></a>Q : Que se passe-t-il si je déploie une image de système d’exploitation non prise en charge sur B1ls ?

**R** : B1ls prend uniquement en charge les images Linux. Si vous déployez une autre image de système d’exploitation, vous ne bénéficierez peut-être pas de la meilleure expérience utilisateur.

## <a name="other-sizes-and-information"></a>Autres tailles et informations

- [Usage général](sizes-general.md)
- [Optimisé pour le calcul](sizes-compute.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le stockage](sizes-storage.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Calcul haute performance](sizes-hpc.md)

Calculatrice de prix : [Calculatrice de prix](https://azure.microsoft.com/pricing/calculator/)

Pour plus d’informations sur les types de disques : [Types de disques](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Étapes suivantes

Lisez-en davantage sur les [Unités de calcul Azure (ACU)](acu.md) pour découvrir comment comparer les performances de calcul entre les références Azure.
