---
title: Versions de Kubernetes prises en charge dans Azure Kubernetes Service
description: Comprendre la stratégie de prise en charge des versions de Kubernetes et le cycle de vie des clusters dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 09/08/2020
author: palma21
ms.author: jpalma
ms.openlocfilehash: ecea9dcd28aede92ef255bf002b9195ded43c68b
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296112"
---
# <a name="supported-kubernetes-versions-in-azure-kubernetes-service-aks"></a>Versions de Kubernetes prises en charge dans Azure Kubernetes Service (AKS)

La communauté Kubernetes publie des versions mineures à peu près tous les trois mois. Récemment, la communauté Kubernetes a [prolongé la fenêtre de prise en charge de chaque version de 9 mois à 12 mois](https://kubernetes.io/blog/2020/08/31/kubernetes-1-19-feature-one-year-support/), à compter de la version 1.19. Ces versions contiennent de nouvelles fonctionnalités et des améliorations. Les publications de correctifs sont plus fréquentes (parfois hebdomadaires) et sont destinées aux correctifs de bogues critiques dans une version mineure. Ces versions comportent des correctifs pour les failles de sécurité et les bogues majeurs.

## <a name="kubernetes-versions"></a>Version de Kubernetes

Kubernetes applique le schéma de contrôle de version [Gestion sémantique de version](https://semver.org/) standard, ce qui signifie que chaque version de Kubernetes suit ce schéma de numérotation :

```
[major].[minor].[patch]

Example:
  1.17.7
  1.17.8
```

Chaque chiffre de la version indique la compatibilité générale avec la version précédente :

* Les versions principales changent lorsque des modifications incompatibles de l’API ou la rétrocompatibilité peuvent être rompues.
* Les versions mineures changent lorsque des modifications de fonctionnalités rétrocompatibles avec les autres versions mineures sont apportées.
* Les versions des correctifs changent lorsque des corrections de bogues rétrocompatibles sont apportées.

Les utilisateurs doivent s’efforcer d’exécuter la dernière version du correctif de la version mineure qu’ils utilisent. Par exemple, si votre cluster de production fonctionne sur **`1.17.7`** et que la dernière version corrective disponible pour la série *1.17* est **`1.17.8`** , vous devez passer à **`1.17.8`** dès que possible pour garantir la prise en charge et le bon état de votre cluster.

## <a name="kubernetes-version-support-policy"></a>Stratégie de prise en charge des versions de Kubernetes

AKS définit une version en disponibilité générale (GA) comme étant activée dans toutes les mesures SLO ou SLA et disponible dans toutes les régions. Il prend en charge trois versions mineures GA de Kubernetes :

* La dernière version mineure GA publiée dans AKS (que nous appellerons N).
* Deux versions mineures précédentes.
* Chaque version mineure prise en charge gère également deux (2) correctifs stables au maximum.
* AKS peut également prendre en charge des préversions, qui sont explicitement étiquetées et soumises aux [Conditions générales des préversions][preview-terms].

> [!NOTE]
> AKS applique des pratiques de déploiement sécurisé qui impliquent un déploiement graduel des régions. Par conséquent, la mise à disposition d’une nouvelle version dans toutes les régions peut prendre jusqu’à 10 jours ouvrés.

La fenêtre de prise en charge des versions de Kubernetes sur AKS est appelée « N-2 » : (N (dernière publication) - 2 (versions mineures)).

Par exemple, si AKS introduit *1.17.a* aujourd’hui, une prise en charge est assurée pour les versions suivantes :

Nouvelle version mineure    |    Liste des versions prises en charge
-----------------    |    ----------------------
1.17.a               |    1.17.a, 1.17.b, 1.16.c, 1.16.d, 1.15.e, 1.15.f

où « .lettre » est représentatif des versions correctives.

Quand une nouvelle version mineure est introduite, la version mineure et les publications des correctifs les plus anciennes prises en charge sont déconseillées et mises hors service. Par exemple, si la liste des versions actuellement prises en charge est :

```
1.17.a
1.17.b
1.16.c
1.16.d
1.15.e
1.15.f
```

Si AKS publie la version 1.18.\*, cela signifie que toutes les versions 1.15.\* seront supprimées et ne seront plus prises en charge dans 30 jours.

> [!NOTE]
> Veuillez noter que si les clients utilisent une version de Kubernetes non supportée, il leur sera demandé de mettre à jour lors de leur demande de support pour le cluster. Les clusters exécutant des versions de Kubernetes non supportées ne sont pas couverts par les politiques de support [AKS](./support-policies.md).

En plus de ce qui précède, AKS prend en charge au maximum deux versions **correctives** d’une version mineure donnée. Prenons les versions prise en charge suivantes :

```
Current Supported Version List
------------------------------
1.17.8, 1.17.7, 1.16.10, 1.16.9
```

Si AKS publie les versions `1.17.9` et `1.16.11`, les versions correctives les plus anciennes sont dépréciées et supprimées, et la liste des versions prises en charge devient :

```
New Supported Version List
----------------------
1.17.*9*, 1.17.*8*, 1.16.*11*, 1.16.*10*
```

### <a name="supported-kubectl-versions"></a>Versions de `kubectl` prises en charge

Vous pouvez utiliser une version mineure de `kubectl` plus ancienne ou plus récente que votre version de *kube-apiserver*, conforme à la [stratégie de prise en charge de Kubernetes pour kubectl](https://kubernetes.io/docs/setup/release/version-skew-policy/#kubectl).

Par exemple, si vous avez la version *1.17* de *kube-apiserver*, vous pouvez utiliser les versions *1.16* à *1.18* de `kubectl`.

Pour installer ou mettre à jour votre version de `kubectl`, exécutez `az aks install-cli`.

## <a name="release-and-deprecation-process"></a>Processus de publication et de dépréciation

Pour consulter la référence sur les versions à venir et les dépréciations, consultez le [Calendrier de publication AKS Kubernetes](#aks-kubernetes-release-calendar).

Pour les nouvelles versions **mineures** de Kubernetes
1. AKS publie une annonce préalable avec la date prévisionnelle d’une nouvelle version et la dépréciation de l’ancienne version correspondante dans les [Notes de publication AKS](https://aka.ms/aks/releasenotes) au moins 30 jours avant la suppression.
2. AKS publie une [notification d’intégrité des services](../service-health/service-health-overview.md) accessible à tous les utilisateurs disposant d’un accès à AKS et au portail, et envoie un e-mail aux administrateurs des abonnements avec les dates prévisionnelles de suppression des versions.
````
To find out who is your subscription administrators or to change it, please refer to [manage Azure subscriptions](https://docs.microsoft.com/azure/cost-management-billing/manage/add-change-subscription-administrator#assign-a-subscription-administrator).
````
3. Les utilisateurs ont **30 jours** à compter de la suppression d’une version pour effectuer une mise à niveau vers une version mineure prise en charge afin de continuer à bénéficier du support.

Pour les nouvelles versions de **correctif** de Kubernetes
  * En raison de leur nature urgente, les versions correctives peuvent être introduites dans le service dès qu’elles sont disponibles.
  * En général, AKS ne communique pas beaucoup lors de la publication de nouvelles versions correctives. Toutefois, il surveille et valide constamment les correctifs CVE disponibles pour les prendre en charge en temps utile. Si un correctif critique est trouvé ou qu’une action des utilisateurs est requise, AKS avertit ces derniers qu’ils peuvent effectuer une mise à niveau vers le nouveau correctif.
  * Les utilisateurs ont **30 jours** à compter de la suppression d’une version corrective d’AKS pour effectuer une mise à niveau vers un correctif pris en charge et ainsi continuer de bénéficier du support.

### <a name="supported-versions-policy-exceptions"></a>Exceptions de la stratégie de version prise en charge

AKS se réserve le droit d’ajouter ou de supprimer de nouvelles versions ou des versions existantes qui ont été identifiées comme ayant une ou plusieurs productions critiques ayant un impact sur des bogues ou des problèmes de sécurité sans préavis.

Certaines versions de correctifs spécifiques peuvent être ignorées, ou le déploiement peut être accéléré en fonction de la gravité du bogue ou du problème de sécurité.

## <a name="azure-portal-and-cli-versions"></a>Versions du Portail Azure et de l’interface CLI

Quand vous déployez un cluster AKS dans le portail ou avec Azure CLI, le cluster est toujours défini par défaut sur la version mineure n-1 et le dernier correctif. Par exemple, si AKS prend en charge *1.17.a*, *1.17.b*, *1.16.c*, *1.16.d*, *1.15.e* et *1.15.f*, la version sélectionnée par défaut est *1.16.c*.

Pour savoir quelles sont les versions disponibles pour vos abonnement et région, utilisez la commande [az aks get-versions][az-aks-get-versions]. L’exemple suivant répertorie les versions Kubernetes disponibles pour la région *EastUS* :

```azurecli-interactive
az aks get-versions --location eastus --output table
```

## <a name="aks-kubernetes-release-calendar"></a>Calendrier des versions d’AKS Kubernetes

Pour connaître l’historique des versions antérieures, cliquez sur [Kubernetes](https://en.wikipedia.org/wiki/Kubernetes#History).

|  Version de K8s | Sortie en amont  | Préversion d’AKS  | Version GA d’AKS  | Fin de vie |
|--------------|-------------------|--------------|---------|-------------|
| 1.16  | 19 septembre 2019  | janvier 2019   | Mars 2020  | Jan. 2021* | 
| 1.17  | 9 déc. 19  | janvier 2019   | Juil. 2020  | 1.20 GA | 
| 1.18  | 23 mars 20  | Mai 2020   | Août 2020  | 1.21 GA | 
| 1,19  | 4 août 20  | Septembre 2020   | Nov. 2020  | 1.22 GA | 
| 1.20  | 8 décembre2020  | Janvier 2021   | Mars 2021  | 1.23 GA |

\* En raison des fêtes de fin d’année, AKS reporte la fin de vie de la version 1.16 de novembre 2020 à janvier 2021. [En savoir plus](https://github.com/Azure/AKS/releases/tag/2020-10-12).


## <a name="faq"></a>Questions fréquentes (FAQ)

**À quelle fréquence dois-je prévoir de mettre à niveau les versions de Kubernetes pour continuer à bénéficier de la prise en charge ?**

Avec Kubernetes 1.19, la [communauté open source a étendu la durée de prise en charge à une année](https://kubernetes.io/blog/2020/08/31/kubernetes-1-19-feature-one-year-support/). AKS s’engage au minimum à activer les correctifs et à prendre en charge le respect des engagements en amont. Cela signifie qu’à partir des clusters AKS sur 1.19, vous pourrez effectuer une mise à niveau au moins une fois par an pour rester sur une version prise en charge. Pour les versions sur 1.18 ou version antérieure, la fenêtre de prise en charge reste à 9 mois, ce qui nécessite une mise à niveau tous les 9 mois afin de rester sur une version prise en charge. Nous vous recommandons vivement de vérifier régulièrement l’existence de nouvelles versions et de vous préparer à la mise à niveau vers des versions plus récentes afin de capturer les dernières améliorations stables dans Kubernetes.

**Que se passe-t-il quand un utilisateur met à niveau un cluster Kubernetes avec une version mineure non prise en charge ?**

Si vous utilisez la version *n-3* ou une version antérieure, vous êtes hors support et il vous sera demandé d’effectuer une mise à niveau. Une fois votre mise à niveau de la version n-3 à la version n-2 réussie, vous bénéficierez à nouveau de nos stratégies de support. Par exemple :

- Si la plus ancienne version d’AKS prise en charge est *1.15.a* et que vous utilisez la version *1.14.b* ou une version antérieure, vous êtes hors support.
- Une fois la mise à niveau de la version *1.14.b* à la version *1.15.a* ou à une version ultérieure réussie, vous bénéficierez à nouveau de nos stratégies de support.

Les passages à une version antérieure ne sont pas pris en charge.

**Que signifie « Ne plus disposer du support technique » ?**

Si vous êtes « hors support », cela signifie que la version que vous utilisez ne figure pas dans la liste des versions prises en charge et qu’il vous sera demandé de mettre à niveau le cluster vers une version prise en charge en cas de demande de support, à moins que vous ne vous trouviez dans la période de grâce de 30 jours suivant la dépréciation de la version. Par ailleurs, AKS n’offre aucune garantie d’exécution ou autre pour les clusters qui ne figurent pas dans la liste des versions prises en charge.

**Que se passe-t-il quand un utilisateur fait évoluer (scaling) un cluster Kubernetes avec une version mineure non prise en charge ?**

Pour les versions mineures non prises en charge par AKS, le scale-in et le scale-out devraient continuer de fonctionner. Cependant, dans la mesure où il n’y a aucune garantie de qualité de service, il est fortement recommandé d’effectuer la mise à niveau pour que le cluster bénéficie à nouveau du support.

**Un utilisateur peut-il rester éternellement sur une même version de Kubernetes ?**

Si un cluster ne bénéficie plus du support depuis plus de trois (3) versions mineures et se révèle présenter des risques de sécurité, Azure vous contacte pour mettre à niveau votre cluster de manière proactive. À défaut d’action supplémentaire de votre part, Azure se réserve le droit d’effectuer automatiquement la mise à niveau de votre cluster à votre place.

**Quelle version le plan de contrôle prend-il en charge si le pool de nœuds n’est pas dans une des versions d’AKS prises en charge ?**

Le plan de contrôle doit se trouver dans une fenêtre de versions pour tous les pools de nœuds. Pour plus d’informations sur la mise à niveau du plan de contrôle ou des pools de nœuds, consultez la documentation sur la [mise à niveau des pools de nœuds](use-multiple-node-pools.md#upgrade-a-cluster-control-plane-with-multiple-node-pools).

**Puis-je ignorer plusieurs versions d’AKS durant la mise à niveau d’un cluster ?**

Quand vous mettez à niveau un cluster AKS pris en charge, les versions mineures de Kubernetes ne peuvent pas être ignorées. Par exemple, les mises à niveau *1.12.x* -> *1.13.x* ou *1.13.x* -> *1.14.x* sont autorisées, mais pas *1.12.x* -> *1.14.x*.

Pour opérer une mise à niveau *1.12.x* -> *1.14.x*, commencez par une mise à niveau *1.12.x* -> *1.13.x*, puis effectuez la mise à niveau *1.13.x* -> *1.14.x*.

Vous pouvez ignorer plusieurs versions uniquement pour une mise à niveau d’une version non prise en charge vers une version prise en charge. Par exemple, la mise à niveau à partir d’une version *1.10.x* non prise en charge vers une version *1.15.x* prise en charge peut être effectuée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la mise à niveau de votre cluster, consultez [Mettre à niveau un cluster Azure Kubernetes Service (AKS)][aks-upgrade].

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[azure-update-channel]: https://azure.microsoft.com/updates/?product=kubernetes-service

<!-- LINKS - Internal -->
[aks-upgrade]: upgrade-cluster.md
[az-aks-get-versions]: /cli/azure/aks#az-aks-get-versions
[preview-terms]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
