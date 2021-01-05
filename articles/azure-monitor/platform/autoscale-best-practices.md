---
title: Meilleures pratiques pour la mise à l’échelle automatique
description: Modèles de mise à l’échelle automatique d’Azure pour Web Apps, Virtual Machine Scale Sets et Cloud Services
ms.topic: conceptual
ms.date: 07/07/2017
ms.subservice: autoscale
ms.openlocfilehash: 7fdb3588833dd9bcf989e020cd1dd861c6e28f37
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95745314"
---
# <a name="best-practices-for-autoscale"></a>Meilleures pratiques pour la mise à l’échelle automatique
La mise à l’échelle automatique Azure Monitor s’applique uniquement aux [groupes de machines virtuelles identiques](https://azure.microsoft.com/services/virtual-machine-scale-sets/), aux [services cloud](https://azure.microsoft.com/services/cloud-services/), à [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) et aux [services de gestion des API](../../api-management/api-management-key-concepts.md).

## <a name="autoscale-concepts"></a>Concepts de la mise à l’échelle automatique

* Une ressource ne peut avoir qu’ *un* paramètre de mise à l’échelle automatique
* Un paramètre de mise à l’échelle automatique peut comporter un ou plusieurs profils et chaque profil peut avoir une ou plusieurs règles de mise à l’échelle automatique.
* Un paramètre de mise à l’échelle automatique met à l’échelle les instances horizontalement, c’est-à-dire vers *l’extérieur* en augmentant la taille des instances et vers *l’intérieur* en diminuant la taille des instances.
  Un paramètre de mise à l’échelle automatique a une valeur d’instances maximum, minimum et par défaut.
* Une tâche de mise à l’échelle automatique lit la mesure associée à mettre à l’échelle, en vérification qu’elle a dépassé le seuil configuré pour l’augmentation ou la diminution de taille d’instance. Vous pouvez afficher une liste des mesures pour la mise à l’échelle automatique dans la rubrique [Mesures courantes de mise à l’échelle automatique Azure Monitor](autoscale-common-metrics.md).
* Tous les seuils sont calculés au niveau de l’instance. Par exemple, « effectuer un scale-out d’une instance lorsque l’UC moyenne est supérieure à 80 % quand le nombre d’instances est égal à 2 » signifie effectuer un scale-out lorsque l’UC moyenne sur toutes les instances est supérieure à 80 %.
* Tous les échecs de mise à l’échelle automatique sont enregistrés dans le journal d’activité. Vous pouvez ensuite configurer une [alerte de journal d’activité](./activity-log-alerts.md) pour être informé par e-mail, SMS ou webhooks à chaque fois qu’un échec de mise à l’échelle automatique se produit.
* De même, toutes les opérations de mise à l’échelle réussies sont consignées dans le journal d’activité. Vous pouvez ensuite configurer une alerte de journal d’activité pour être informé par e-mail, SMS ou webhooks à chaque fois qu’une opération de mise à l’échelle automatique se termine avec succès. Vous pouvez également configurer des notifications par e-mail ou webhook pour être averti en cas d’action de mise à l’échelle réussie via l’onglet Notifications du paramètre de mise à l’échelle automatique.

## <a name="autoscale-best-practices"></a>Meilleures pratiques relatives à la mise à l’échelle automatique

Utilisez les meilleures pratiques suivantes lorsque vous utilisez la mise à l’échelle automatique.

### <a name="ensure-the-maximum-and-minimum-values-are-different-and-have-an-adequate-margin-between-them"></a>Vérifiez que les valeurs minimales et maximales sont différentes et séparées par une marge suffisante.

Si le paramètre a une valeur minimum = 2, une valeur maximum = 2 et que le nombre d’instances actuel est égal à 2, aucune action de mise à l’échelle ne peut se produire. Conservez une marge suffisante entre les nombres d’instances minimum et maximum, qui sont inclusifs. La mise à l’échelle agit toujours entre ces limites.

### <a name="manual-scaling-is-reset-by-autoscale-min-and-max"></a>La mise à l’échelle manuelle est réinitialisée par les valeurs min et max de mise à l’échelle

Si vous mettez à jour manuellement le nombre d’instances avec une valeur inférieure au minimum ou supérieure au maximum, le moteur de mise à l’échelle s’ajuste automatiquement à la valeur minimale (si elle est inférieure) ou à la valeur maximale (le cas ci-dessus). Par exemple, vous définissez la plage entre 3 et 6. Si vous avez une seule instance en cours d’exécution, le moteur de mise à l’échelle automatique effectue la mise à l’échelle sur trois instances lors de sa prochaine exécution. De même, si vous définissez manuellement la mise à l’échelle sur huit instances, la mise à l’échelle sera redéfinie sur six instances lors de la prochaine exécution.  La mise à l’échelle manuelle est temporaire, sauf si vous réinitialisez aussi les règles de mise à l’échelle.

### <a name="always-use-a-scale-out-and-scale-in-rule-combination-that-performs-an-increase-and-decrease"></a>Utilisez toujours une combinaison de règle d’augmentation et de diminution de la taille des instances qui exécute une augmentation et une diminution
Si vous utilisez uniquement une partie de la combinaison, la mise à l’échelle automatique n’effectue l’action que dans une direction (scale-out ou scale-in) jusqu’à ce qu’elle atteigne le nombre d’instances minimum ou maximum défini dans le profil. Cette configuration n’est pas optimale ; dans l’idéal, vous souhaiteriez que votre ressource puisse monter en puissance lors des périodes de forte utilisation afin d’assurer la disponibilité. De même, lors des périodes de faible utilisation, vous souhaitez que votre ressource descende en puissance pour vous permettre de réduire vos coûts.

### <a name="choose-the-appropriate-statistic-for-your-diagnostics-metric"></a>Sélection de la statistique appropriée pour votre mesure de diagnostic
Pour les mesures de diagnostics, vous pouvez choisir entre *Moyen*, *Minimum*, *Maximum* et *Total* comme mesure de mise à l’échelle. La statistique la plus courante est *Moyen*.

### <a name="choose-the-thresholds-carefully-for-all-metric-types"></a>Sélection des seuils pour tous les types de mesure
Nous vous recommandons de choisir avec soin des seuils différents pour l’augmentation de la taille des instances et la diminution de la taille des instances en fonction des pratiques.

Nous vous *déconseillons* de choisir des paramètres de mise à l’échelle tels que les exemples ci-dessous, avec des valeurs de seuil similaires ou identiques pour l’augmentation et la diminution de la taille des instances :

* Augmenter les instances de 1 lorsque le nombre de threads >= 600
* Diminuer les instances de 1 lorsque le nombre de threads <= 600

Examinons un exemple de ce qui peut entraîner un comportement qui peut sembler déroutant. Examinez la séquence suivante.

1. Supposons qu’il existe deux instances pour commencer, puis que le nombre moyen de threads par instance atteigne 625.
2. La mise à l’échelle automatique augmente la taille des instances en ajoutant une troisième instance.
3. Ensuite, supposons que le nombre de threads moyen entre les instances diminue pour atteindre 575.
4. Avant la descente en puissance, la mise à l’échelle automatique essaye d’estimer quel sera l’état final en cas de diminution de la taille des instances. Par exemple, 575 x 3 (nombre d’instances actuel) = 1 725 / 2 (nombre final d’instances lors de la descente en puissance) = 862,5 threads. Cela signifie que la mise à l’échelle automatique effectuerait immédiatement un scale-out même après le scale-in, si le nombre moyen de threads reste le même ou même baisse d’une petite quantité. Toutefois, en cas de nouvelle montée en puissance, l’ensemble du processus se répète, menant à une boucle infinie.
5. Pour éviter ce problème , la mise à l’échelle automatique ne descend pas du tout en puissance. Au lieu de cela, elle ignore et réévalue la condition lors de la prochaine exécution de la tâche du service. Le bagottement de l’état peut perturber de nombreuses personnes, car la mise à l’échelle automatique semble ne pas fonctionner lorsque le nombre moyen de threads est de 575.

Lors d’une mise à l’échelle, l’estimation permet d’éviter les situations de « bagottement », où la taille des instances est continuellement modifiée (diminuée puis augmentée, et inversement). Gardez ce comportement à l’esprit lorsque vous choisissez les mêmes seuils pour la diminution et l’augmentation de la taille des instances.

Nous vous recommandons de choisir une marge suffisante entre les seuils de diminution et d’augmentation de la taille des instances. Par exemple, examinez la combinaison de règles suivante bien plus adaptée.

* Augmenter les instances de 1 lorsque le % du processeur >= 80
* Diminuer les instances de 1 lorsque le % du processeur <= 60

Dans ce cas  

1. Supposons qu’il existe 2 instances pour commencer.
2. Si le pourcentage moyen du processeur entre les instances atteint 80, la mise à l’échelle automatique augmente la taille des instances en ajoutant une troisième instance.
3. Supposons maintenant qu’au fil du temps, le pourcentage du processeur baisse à 60.
4. La règle de diminution de la taille des instances de la mise à l’échelle automatique évalue l’état final en cas de diminution de la taille des instances. Par exemple, 60 x 3 (nombre d’instances actuel) = 180 / 2 (nombre final d’instances lors de la descente en puissance) = 90. Par conséquent, la mise à l’échelle automatique ne diminue pas la taille des instances, car il lui faudrait de nouveau monter en puissance immédiatement. Au lieu de cela, elle ignore la descente en puissance.
5. Lors de la vérification de mise à l’échelle de temps suivante, l’UC continue sa diminution à 50. Une nouvelle estimation est effectuée : 3 x 50 instances = 150 / 2 instances = 75, ce qui est inférieur au seuil d’augmentation de la taille des instances qui est de 80, la diminution de la taille des instances s’exécute pour obtenir 2 instances.

> [!NOTE]
> Si le moteur de mise à l’échelle automatique détecte un risque de bagottement en raison de la mise à l’échelle vers le nombre cible d’instances, il essaie également d’effectuer une mise à l’échelle avec un nombre différent d’instances entre le nombre actuel et le nombre cible. Si le bagottement ne se produit pas dans cette plage, la mise à l’échelle automatique continue l’opération de mise à l’échelle avec la nouvelle cible.

### <a name="considerations-for-scaling-threshold-values-for-special-metrics"></a>Considérations relatives aux valeurs de seuil de la mise à l’échelle pour les mesures spéciales
 Pour les mesures spéciales, telles que les mesures de longueur de file d’attente Service Bus ou de stockage, le seuil correspond au nombre moyen de messages disponibles en fonction du nombre actuel d’instances. Choisissez soigneusement la valeur de seuil pour ce métrique.

Examinons un exemple qui vous permettra de mieux comprendre ce comportement.

* Augmenter les instances de 1 lorsque le nombre de messages de file d’attente de stockage >= 50
* Diminuer les instances de 1 lorsque le nombre de messages de file d’attente de stockage <= 10

Examinez la séquence suivante :

1. Il existe deux instances de file d’attente de stockage.
2. Les messages continuent d’arriver et lorsque vous examinez la file d’attente de stockage, le nombre total est de 50. Vous pourriez supposer que la mise à l’échelle automatique devrait démarrer une action de montée en charge. Toutefois, notez que le nombre de messages par instance est de 50/2 = 25 messages. Par conséquent, la montée en charge ne se produit pas. Pour que la montée en charge se produise, le nombre total de messages dans la file d’attente de stockage doit être égal à 100.
3. Ensuite, supposons que le nombre total de messages atteigne 100.
4. Une troisième instance de file d'attente de stockage est ajoutée en raison d’une action de montée en charge.  La prochaine action de montée en charge ne se produira que lorsque le nombre total de messages dans la file d’attente atteindra 150, car 150/3 = 50.
5. Maintenant, le nombre de messages dans la file d’attente diminue. Avec trois instances, la première action de diminution de la taille des instances se produit lorsque le nombre total de messages dans toutes les files d’attente atteint 30, car 30/3 donne 10 messages par instance, ce qui correspond au seuil de diminution de la taille des instances.

### <a name="considerations-for-scaling-when-multiple-profiles-are-configured-in-an-autoscale-setting"></a>Considérations relatives à la mise à l’échelle lorsque plusieurs profils sont configurés dans un paramètre de mise à l’échelle automatique
Dans un paramètre de mise à l’échelle automatique, vous pouvez choisir un profil par défaut, qui est toujours appliqué indépendamment de toute planification ou de l’heure, ou vous pouvez choisir un profil récurrent ou un profil pour une durée fixe avec une plage de dates et d’heures.

Lorsque le service de mise à l’échelle automatique traite ces éléments, il vérifie toujours ce qui suit dans l’ordre :

1. Profil de date fixe
2. Profil récurrent
3. Profil par défaut (« Toujours »)

Si une condition de profil est remplie, la mise à l’échelle automatique ne vérifie pas la condition de profil suivante. La mise à l’échelle automatique ne traite qu’un seul profil à la fois. Cela signifie que si vous souhaitez également inclure une condition de traitement d’un profil de niveau inférieur, vous devez également inclure ces règles dans le profil actuel.

Examinons cela à l’aide d’un exemple :

L’image ci-dessous illustre un paramètre de mise à l’échelle automatique avec un profil par défaut d’instances minimum = 2 et d’instances maximum = 10. Dans cet exemple, les règles sont configurées pour la montée en charge lorsque le nombre de messages dans la file d’attente est supérieur à 10 et la diminution de la taille des instances lorsque le nombre de messages dans la file d’attente est inférieur à trois. À présent, la ressource peut être mise à l’échelle entre deux et dix instances.

En outre, il existe un profil récurrent défini pour Lundi. Il est défini pour des instances minimum = 3 et des instances maximum = 10. Cela signifie que le lundi, la première fois que la mise à l’échelle automatique vérifie cette condition, si le nombre d’instances est égal à deux, il est mis à l’échelle pour correspondre au nouveau niveau minimum de trois. Tant que la mise à l’échelle automatique rencontre cette condition de profil respectée (lundi), elle ne traite que les règles de montée/descente en puissance basées sur le processeur configurées pour ce profil. À ce stade, elle ne vérifie pas la longueur de la file d’attente. Toutefois, si vous souhaitez également que la condition de longueur de la file d’attente soit vérifiée, vous devez inclure les règles du profil par défaut dans votre profil de Lundi.

De même, lorsque la mise à l’échelle automatique bascule vers le profil par défaut, elle vérifie d’abord si les conditions minimales et maximales sont remplies. Si le nombre d’instances à ce moment-là est égal à 12, la taille des instances diminue jusqu’à 10, le maximum autorisé pour le profil par défaut.

![paramètres de mise à l’échelle automatique](./media/autoscale-best-practices/insights-autoscale-best-practices-2.png)

### <a name="considerations-for-scaling-when-multiple-rules-are-configured-in-a-profile"></a>Considérations relatives à la mise à l’échelle lorsque plusieurs règles sont configurées dans un profil

Il existe des cas où vous devrez définir plusieurs règles dans un profil. Les règles de mise à l’échelle automatique suivantes sont utilisées par le moteur de mise à l’échelle automatique quand plusieurs règles sont définies.

Pour *l’augmentation de la taille des instances*, la mise à l’échelle automatique s’exécute si une règle est respectée.
Pour la *diminution de la taille des instances*, la mise à l’échelle automatique nécessite que toutes les règles soient respectées.

Pour illustrer cela, supposons que vous disposez des quatre règles de mise à l’échelle automatique suivantes :

* Si UC < 30 %, diminuer la taille des instances de 1
* Si Mémoire < 50 %, diminuer la taille des instances de 1
* Si UC > 75 %, augmenter la taille des instances de 1
* Si Mémoire > 75 %, augmenter la taille des instances de 1

Alors, ce qui suit se produit :

* Si le processeur est de 76 % et la mémoire est de 50 %, une augmentation de la taille des instances se produit.
* Si le processeur est de 50 % et la mémoire est de 76 %, une augmentation de la taille des instances se produit.

En revanche, si le processeur est de 25 % et la mémoire est de 51 %, la mise à l’échelle automatique ne diminue **pas** la taille des instances. Pour diminuer la taille des instances, le processeur doit être de 29 % et la mémoire de 49 %.

### <a name="always-select-a-safe-default-instance-count"></a>Sélectionnez toujours un nombre d’instances par défaut sans échec
Le nombre d’instances par défaut est important, car la mise à l’échelle de vos services s’effectue en fonction de ce nombre quand les métriques ne sont pas disponibles. Par conséquent, sélectionnez un nombre d’instances par défaut qui est sécurisé pour vos charges de travail.

### <a name="configure-autoscale-notifications"></a>Configuration des notifications de mise à l’échelle automatique
Les événements de mise à l’échelle automatique sont enregistrés dans le journal d’activité dans les cas suivants :

* La mise à l’échelle automatique déclenche une opération de mise à l’échelle.
* Le service de mise à l’échelle automatique termine une opération de mise à l’échelle avec succès.
* Le service de mise à l’échelle automatique ne parvient pas à terminer une opération de mise à l’échelle avec succès
* Les mesures ne sont pas disponibles pour que le service de mise à l’échelle automatique prenne une décision de mise à l’échelle.
* Les mesures sont de nouveau disponibles (récupération) pour prendre une décision de mise à l’échelle.
* La mise à l’échelle automatique détecte un bagottement et abandonne la tentative de mise à l’échelle. Vous verrez un type de journal `Flapping` dans cette situation. Si vous voyez cela, déterminez si vos seuils sont trop rapprochés.
* La mise à l’échelle automatique détecte un bagottement mais reste en mesure d’effectuer correctement la mise à l’échelle. Vous verrez un type de journal `FlappingOccurred` dans cette situation. Si vous voyez cela, le moteur de mise à l’échelle automatique a tenté une mise à l’échelle (par exemple, de 4 instances à 2), mais a déterminé que cela entraînerait un bagottement. Au lieu de cela, le moteur de mise à l’échelle automatique a pris en compte une mise à l’échelle vers un nombre d’instances différent (par exemple, en utilisant 3 instances au lieu de 2), ce qui ne provoque plus de bagottement, et a donc effectué la mise à l’échelle vers ce nombre d’instances.

Vous pouvez également utiliser une alerte de journal d’activité pour surveiller l’intégrité du moteur de mise à l’échelle automatique. Voici des exemples pour [créer une alerte de journal d’activité pour surveiller toutes les opérations du moteur de mise à l’échelle automatique dans votre abonnement](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert) ou [créer une alerte de journal d’activité pour surveiller tous les échecs d’opérations de mise à l’échelle automatique dans votre abonnement](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert).

Outre l’activation des alertes de journal d’activité, vous pouvez configurer des notifications par e-mail ou webhook pour être averti en cas d’action de mise à l’échelle réussie via l’onglet Notifications du paramètre de mise à l’échelle automatique.

## <a name="next-steps"></a>Étapes suivantes
- [Créez une alerte de journal d’activité pour surveiller toutes les opérations du moteur de mise à l’échelle automatique dans votre abonnement.](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)
- [Créer une alerte de journal d’activité pour surveiller tous les échecs d’opérations de mise à l’échelle automatique dans votre abonnement](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)

