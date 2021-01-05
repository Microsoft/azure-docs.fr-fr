---
title: Résoudre les problèmes liés aux connexions sortantes dans Azure Load Balancer
description: Résolutions des problèmes courants de connectivité sortante via Azure Load Balancer.
services: load-balancer
author: erichrt
ms.service: load-balancer
ms.topic: troubleshooting
ms.date: 05/7/2020
ms.author: errobin
ms.openlocfilehash: 516576f4e005cc9fe2303945ecb1a13489908a5d
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696351"
---
# <a name="troubleshooting-outbound-connections-failures"></a><a name="obconnecttsg"></a> Résolution des problèmes liés aux défaillances des connexions sortantes

Cet article a pour but de fournir des solutions aux problèmes courants liés aux connexions sortantes d’Azure Load Balancer. La plupart des problèmes de connectivité sortante que les clients rencontrent sont dus à une insuffisance de ports SNAT et à l’expiration des délais de connexion, ce qui entraîne une perte de paquets. Cet article décrit les étapes à suivre pour atténuer chacun de ces problèmes.

## <a name="managing-snat-pat-port-exhaustion"></a><a name="snatexhaust"></a>Gestion de l’insuffisance de ports (PAT) SNAT
Les [ports éphémères](load-balancer-outbound-connections.md) utilisés pour [PAT](load-balancer-outbound-connections.md) sont une ressource épuisable, comme décrit dans [Machine virtuelle autonome sans adresse IP publique](load-balancer-outbound-connections.md) et [Machine virtuelle à charge équilibrée sans adresse IP publique](load-balancer-outbound-connections.md). Vous pouvez superviser l’utilisation des ports éphémères et la comparer à votre allocation actuelle pour déterminer le niveau de risque, ou pour confirmer l’insuffisance des ports SNAT à l’aide de [ce](./load-balancer-standard-diagnostics.md#how-do-i-check-my-snat-port-usage-and-allocation) guide.

Si vous savez que vous lancez de nombreuses connexions TCP ou UDP sortantes vers les mêmes adresse IP et port de destination et constatez que des connexions sortantes échouent, ou si l’équipe de support vous signale que les ports SNAT ([ports éphémères](load-balancer-outbound-connections.md#preallocatedports) préaffectés) utilisés par la [PAT](load-balancer-outbound-connections.md) arrivent à épuisement, plusieurs options d’atténuation générales s’offrent à vous. Passez en revue ces options et choisissez l’option disponible la plus appropriée pour votre scénario. Plusieurs options peuvent être adaptées à votre scénario.

Si vous avez des difficultés à comprendre le comportement des connexions sortantes, vous pouvez utiliser les statistiques de pile IP (netstat). Il peut aussi être utile d’observer les comportements de connexion à travers les captures de paquets. Vous pouvez effectuer ces captures de paquets dans le SE invité de votre instance, ou utiliser le [Network Watcher pour la capture des paquets](../network-watcher/network-watcher-packet-capture-manage-portal.md). 

## <a name="manually-allocate-snat-ports-to-maximize-snat-ports-per-vm"></a><a name ="manualsnat"></a>Allouer manuellement les ports SNAT pour augmenter le nombre de ports SNAT par machine virtuelle
Comme défini dans [ports préaffectés](load-balancer-outbound-connections.md#preallocatedports), l’équilibreur de charge alloue automatiquement les ports en fonction du nombre de machines virtuelles dans le serveur principal. Par défaut, cette opération est effectuée de manière restrictive pour garantir la scalabilité. Si vous connaissez le nombre maximal de machines virtuelles que vous aurez dans le serveur back-end, vous pouvez allouer manuellement des ports SNAT dans chaque règle de trafic sortant. Par exemple, si vous savez qu’il y a un maximum de 10 machines virtuelles, vous pouvez allouer 6 400 ports SNAT par machine virtuelle plutôt que les 1 024 par défaut. 

## <a name="modify-the-application-to-reuse-connections"></a><a name="connectionreuse"></a>Modifier l’application pour réutiliser des connexions 
Vous pouvez réduire la demande pour les ports éphémères utilisés pour la traduction d’adresses réseau sources en réutilisant des connexions dans votre application. La réutilisation des connexions est particulièrement pertinente pour les protocoles tels que HTTP/1.1, où la réutilisation des connexions est la valeur par défaut. D’autres protocoles qui utilisent HTTP comme transport (par exemple REST) peuvent aussi en bénéficier. 

Il est toujours préférable de réutiliser les connexions que de recourir à des connexions TCP individuelles et atomiques pour chaque demande. La réutilisation produit des transactions TCP plus performantes et très efficaces.

## <a name="modify-the-application-to-use-connection-pooling"></a><a name="connection pooling"></a>Modifier l’application pour utiliser un regroupement de connexions
Vous pouvez utiliser un schéma de regroupement de connexions dans votre application, dans lequel les demandes sont distribuées en interne sur un ensemble fixe de connexions (chacune étant réutilisée dans la mesure du possible). Ce schéma limite le nombre de ports éphémères utilisés et crée un environnement plus prévisible. Il peut aussi accroître le débit des demandes en autorisant plusieurs opérations simultanées quand une seule connexion se bloque sur la réponse d’une opération.  

Il est possible que le regroupement de connexions existe déjà dans le framework que vous utilisez pour développer votre application ou dans les paramètres de configuration de votre application. Vous pouvez associer le regroupement de connexions à la réutilisation des connexions. Vos diverses demandes consomment alors un nombre de ports fixe et prévisible vers les mêmes adresse IP et port de destination. Les demandes profitent aussi d’une utilisation efficace des transactions TCP, ce qui réduit la latence et l’utilisation de ressources. Cela peut aussi être bénéfique pour les transactions UDP, car la gestion du nombre de flux UDP peut à son tour éviter des conditions d’épuisement et gérer l’utilisation des ports SNAT.

## <a name="modify-the-application-to-use-less-aggressive-retry-logic"></a><a name="retry logic"></a>Modifier l’application pour utiliser une logique de nouvelle tentative moins agressive
Quand les [ports éphémères préaffectés](load-balancer-outbound-connections.md#preallocatedports) utilisés pour la [traduction d’adresse de port](load-balancer-outbound-connections.md) sont épuisés, ou quand des échecs d’application se produisent, les nouvelles tentatives de connexion agressives ou par force brute sans logique de réduction ou d’interruption entraînent la survenance ou la persistance de l’épuisement. Vous pouvez réduire la demande de ports éphémères en utilisant une logique de nouvelle tentative moins agressive. 

Les ports éphémères ont un délai d’inactivité de 4 minutes (non ajustable). Si les nouvelles tentatives sont trop agressives, le problème d’épuisement ne peut pas se résoudre de lui-même. Par conséquent, il est essentiel de pouvoir évaluer comment et selon quelle fréquence votre application relance les transactions.

## <a name="assign-a-public-ip-to-each-vm"></a><a name="assignilpip"></a>Assigner une adresse IP publique à chaque machine virtuelle
L’attribution d’une adresse IP publique modifie votre scénario sur [Adresse IP publique sur une machine virtuelle](load-balancer-outbound-connections.md). Les ports éphémères de l’adresse IP publique qui sont utilisés pour chaque machine virtuelle sont tous accessibles à la machine virtuelle. (Contrairement aux scénarios où les ports éphémères d’une adresse IP publique sont partagés avec toutes les machines virtuelles associés au pool backend correspondant.) Des compromis sont à prendre en compte, notamment le coût supplémentaire des IP publiques et l’impact possible du filtrage d’un nombre important d’adresses IP individuelles.

>[!NOTE] 
>Cette option n’est pas disponible pour les rôles de travail web.

## <a name="use-multiple-frontends"></a><a name="multifesnat"></a>Utiliser plusieurs serveurs frontaux
Lorsque vous utilisez l’équilibreur de charge standard public, vous assignez [plusieurs adresses IP de serveur frontal pour les connexions sortantes](load-balancer-outbound-connections.md) et [multipliez le nombre de ports SNAT disponibles](load-balancer-outbound-connections.md#preallocatedports).  Créez une configuration IP de serveur frontal, une règle et un pool principal pour déclencher la programmation du SNAT sur l’adresse IP publique du serveur frontal.  La règle n’a pas besoin de fonctionner et une sonde d’intégrité n’a pas besoin d’aboutir.  Si vous utilisez plusieurs frontends pour le trafic entrant également (plutôt que simplement pour le trafic sortant), vous devez correctement utiliser les sondes d’intégrité personnalisées pour garantir la fiabilité.

>[!NOTE]
>Dans la plupart des cas, l’insuffisance des ports SNAT résulte d’une mauvaise conception.  Assurez-vous que vous comprenez la raison de l’insuffisance de ports avant d’utiliser plus de serveurs frontaux pour ajouter des ports SNAT.  Vous pouvez masquer un problème qui peut provoquer une défaillance ultérieure.

## <a name="scale-out"></a><a name="scaleout"></a>Scale-out
Des [ports préalloués](load-balancer-outbound-connections.md#preallocatedports) sont attribués en fonction de la taille du pool backend et regroupés en niveaux afin de réduire les interruptions dans les situations où certains des ports doivent être réalloués pour prendre en charge le prochain niveau immédiatement supérieur de taille de pool backend.  Vous pouvez éventuellement augmenter l’utilisation des ports SNAT d’un serveur front-end donné en effectuant une mise à l’échelle de votre pool de serveurs back-end à la taille maximale d’un niveau donné.  Gardez à l’esprit que l’allocation de port par défaut est nécessaire pour que l’application puisse effectuer un scale-out efficace sans risquer de manquer de ports SNAT.

Par exemple, deux machines virtuelles du pool principal auraient 1024 ports SNAT disponibles par configuration IP, pour un total de 2 048 ports SNAT pris en charge pour le déploiement.  Si le déploiement devait être augmenté de 50 machines virtuelles, même si le nombre de ports préalloués demeure constant par machine virtuelle, un total de 51 200 (50 x 1 024) ports SNAT peut être utilisé par le déploiement.  Si vous souhaitez effectuer un scale-out de votre déploiement, vérifiez le nombre de [ports préalloués](load-balancer-outbound-connections.md#preallocatedports) par niveau pour avoir la certitude de configurer votre scale-out à la valeur maximale du niveau respectif.  Dans l’exemple précédent, si vous choisissez d’effectuer un scale-out à 51 instances et non à 50, vous atteignez le niveau suivant et obtenez moins de ports SNAT par machine virtuelle et au total.

Si vous effectuez un scale-out vers le niveau immédiatement supérieur de taille de pool principal et que l’opération nécessite une réallocation des ports alloués, certaines de vos connexions peuvent expirer.  Si vous utilisez uniquement certains de vos ports SNAT, une telle montée en charge n’affectera pas vos connexions.  La moitié des ports existants seront réaffectés à chaque fois que vous passerez au niveau de pool principal suivant.  Si vous ne voulez pas que cela se produise, vous devez configurer votre déploiement en fonction de la taille de niveau considérée.  Sinon, assurez-vous que votre application peut détecter et effectuer autant de tentatives que nécessaire.  Les conservations de connexion active TCP peuvent contribuer à détecter un dysfonctionnement des ports SNAT suite à une réallocation.

## <a name="use-keepalives-to-reset-the-outbound-idle-timeout"></a><a name="idletimeout"></a>Utiliser des conservations de connexion active pour réinitialiser le délai d’inactivité en sortie
Les connexions sortantes ont un délai d’inactivité de 4 minutes. Vous pouvez ajuster ce délai d’expiration à l’aide de [règles de trafic sortant](outbound-rules.md). Vous pouvez également utiliser un transport (par exemple, des conservations de connexion active TCP) ou des conservations de connexion active de couche Application pour actualiser un flux inactif et réinitialiser ce délai d’inactivité, si nécessaire.  

Lorsque vous utilisez des conservations de connexion active TCP, il suffit de les activer sur un côté de la connexion. Par exemple, il suffit de les activer sur le côté serveur uniquement pour réinitialiser la minuterie d’inactivité ; il est inutile que les 2 côtés lancent des conservations de connexion active.  Des concepts similaires existent pour la couche d’application, notamment les configurations client-serveur de base de données.  Examinez côté serveur les options de persistance de connexion spécifiques aux applications.

## <a name="next-steps"></a>Étapes suivantes
Nous cherchons toujours à améliorer l’expérience utilisateur de nos clients. Si vous rencontrez des problèmes de connectivité sortante qui ne sont pas listés ou résolus par cet article, envoyez vos commentaires via GitHub au bas de cette page, et nous les traiterons le plus rapidement possible.