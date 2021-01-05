---
title: Guide de gestion des coûts pour Azure Lab Services
description: Découvrez les différentes façons de visualiser les coûts des services Lab.
author: rbest
ms.author: rbest
ms.date: 08/16/2020
ms.topic: article
ms.openlocfilehash: 29f6be5319c5a142ad3ea0d73deb2f95d8cb0d7a
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659724"
---
# <a name="cost-management-for-azure-lab-services"></a>Gestion des coûts pour Azure Lab Services

Pour Azure Lab Services, la gestion des coûts peut être divisée en deux domaines distincts : l’estimation des coûts et l’analyse des coûts. L’estimation des coûts se produit lors de la configuration du labo pour s’assurer que la structure initiale du labo sera adaptée au budget attendu. L’analyse des coûts se produit généralement à la fin du mois pour déterminer les actions nécessaires pour le mois suivant.

## <a name="estimate-the-lab-costs"></a>Estimer les coûts de laboratoire

Chaque tableau de bord de laboratoire dispose d’une section **Coûts et facturation** qui présente une estimation sommaire du coût que peut représenter le laboratoire pour le mois. L’estimation des coûts résume les heures d’utilisation avec le nombre maximal d’utilisateurs par le coût horaire estimé. Pour connaître l’estimation la plus précise, configurez le labo, y compris la [planification](how-to-create-schedules.md). Le tableau de bord reflète le coût estimé. 

Cette estimation peut ne pas afficher tous les coûts possibles. Certaines ressources ne sont pas incluses :

- Coût de préparation du modèle. Le temps nécessaire à la création du modèle peut varier considérablement. Le coût d’exécution du modèle est le même que le coût global par heure du labo. 
- Toute [galerie d’images partagées](how-to-use-shared-image-gallery.md) coûte, car une galerie peut être partagée entre plusieurs labos. 
- Heures de démarrage d’une machine virtuelle par le créateur de labo.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant le tableau de bord d’estimation des coûts.](./media/cost-management-guide/dashboard-cost-estimation.png)

## <a name="analyze-the-previous-months-usage"></a>Analyser l’utilisation du mois précédent

L’analyse des coûts permet d’examiner l’utilisation des mois précédents pour vous aider à déterminer les ajustements à apporter au labo. Vous trouverez la répartition des coûts passés dans l’[Analyse des coûts d’abonnement](../cost-management-billing/costs/quick-acm-cost-analysis.md). Dans le Portail Azure, vous pouvez saisir **Abonnements** dans le champ de recherche, puis sélectionner l’option **Abonnements**. 

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant la zone de recherche et l’option Abonnements.](./media/cost-management-guide/subscription-search.png)

Sélectionnez l’abonnement spécifique à évaluer.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant une sélection d’abonnements.](./media/cost-management-guide/subscription-select.png)

Sélectionnez **Analyse du coût** dans le volet gauche sous **Cost Management**.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant une analyse des coûts d’abonnement sur un graphique.](./media/cost-management-guide/subscription-cost-analysis.png)

Ce tableau de bord permet une analyse détaillée des coûts, y compris la capacité d’exportation vers différents types de fichiers sur une planification. Pour plus d’informations, consultez [Présentation Cost Management + Facturation](../cost-management-billing/cost-management-billing-overview.md).

Vous pouvez filtrer par type de ressource. L’utilisation de `microsoft.labservices/labaccounts` affiche uniquement le coût associé aux services Lab.

## <a name="understand-the-usage"></a>Comprendre l’utilisation

La capture d'écran suivante est un exemple d’analyse du coût.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant un exemple d’analyse du coût d’abonnement.](./media/cost-management-guide/cost-analysis.png)

Par défaut, Il y a six colonnes : **Ressource**, **Type de ressources**, **Emplacement**, **Nom de groupe de ressources**, **Balises** et **Coût**. La colonne **Ressource** contient les informations sur le compte lab, le nom du labo et la machine virtuelle. Les lignes qui affichent le compte lab, le nom du labo et la valeur par défaut (deuxièmes et troisièmes lignes) représentent le coût du labo. Les machines virtuelles utilisées ont un coût que vous pouvez voir pour les lignes qui affichent le compte lab, le nom du labo, la valeur par défaut et le nom de la machine virtuelle. 

Dans cet exemple, en ajoutant les premières et deuxièmes lignes (commençant toutes deux par **aaalab/dockerlab**), vous obtenez le coût total du labo « dockerlab » dans le compte lab « aaalab ».

Pour obtenir le coût global de la galerie d’images, modifiez le type de ressource en `Microsoft.Compute/Galleries`. La galerie d’images partagée peut ne pas apparaître dans les coûts en fonction de l’emplacement de stockage de la galerie.

> [!NOTE]
> La galerie d’image partagée est connectée au compte lab. Cela signifie que plusieurs labos peuvent utiliser la même image.

## <a name="separate-the-costs"></a>Séparer les coûts

Certaines universités ont utilisé le compte lab et le groupe de ressources pour séparer les différentes classes. Chaque classe a son propre compte lab et son propre groupe de ressources. 

Dans le volet analyse du coût, ajoutez un filtre basé sur le nom du groupe de ressources avec le nom de groupe de ressources correspondant à la classe. Ensuite, seuls les coûts de cette classe seront visibles. Cela permet une délimitation plus claire entre les différentes classes lors de la visualisation des coûts. Vous pouvez utiliser la fonctionnalité [d’exportation planifiée](../cost-management-billing/costs/tutorial-export-acm-data.md) de l’analyse du coût pour télécharger les coûts de chaque classe dans des fichiers distincts.

## <a name="manage-costs"></a>Gérer les coûts

Selon le type de classe, il existe des moyens de gérer les coûts pour réduire les instances de machines virtuelles qui s’exécutent sans qu’un étudiant ne les utilise.

### <a name="automatic-shutdown-settings-for-cost-control"></a>Paramètres d’arrêt automatique pour le contrôle du coût

Les fonctionnalités d’arrêt automatique vous permettent d’éviter de gaspiller les heures d’utilisation de machines virtuelles dans les labos. Les paramètres suivants interceptent la plupart des cas où les utilisateurs laissent accidentellement leurs machines virtuelles en cours d’exécution :

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant les trois paramètres d’arrêt automatique.](./media/cost-management-guide/auto-shutdown-disconnect.png)

Vous pouvez configurer ces paramètres au niveau du compte lab ainsi qu’au niveau du labo. Si vous les activez au niveau du compte lab, ils sont appliqués à tous les labos au sein du compte lab. Pour tous les nouveaux comptes labo, ces paramètres sont activés par défaut. 

#### <a name="automatically-disconnect-users-from-virtual-machines-that-the-os-deems-idle"></a>Déconnecter automatiquement les utilisateurs des machines virtuelles que le système d’exploitation juge inactives

> [!NOTE]
> Ce paramètre est disponible uniquement pour les machines virtuelles Windows.

Lorsque le paramètre **Déconnecter les utilisateurs lorsque des machines virtuelles sont inactives** est activé, l’utilisateur est déconnecté de toutes les machines du labo lorsque le système d’exploitation Windows considère que la session est inactive (y compris les modèles de machines virtuelles). La [définition de l’inactivité du système d’exploitation Windows](/windows/win32/taskschd/task-idle-conditions#detecting-the-idle-state) utilise deux critères : 

* Absence de l’utilisateur : aucune entrée au clavier ou à la souris.
* Absence de consommation des ressources : tous les processeurs et tous les disques sont restés inactifs pendant un certain pourcentage de temps.

Les utilisateurs verront un message similaire à celui-ci dans la machine virtuelle avant d’être déconnectés : 

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant un message d’avertissement indiquant qu’une session a été inactive pendant sa limite de temps et sera déconnectée.](./media/cost-management-guide/idle-timer-expired.png)
 
La machine virtuelle est toujours en cours d’exécution lorsque l’utilisateur est déconnecté. Si l’utilisateur se reconnecte à la machine virtuelle en se connectant, les fenêtres ou les fichiers qui étaient ouverts ou le travail non sauvegardé avant la déconnexion seront toujours là. Dans cet état, parce que la machine virtuelle fonctionne, elle est toujours comptabilisée comme active et accumule les coûts. 
 
Pour arrêter automatiquement les machines virtuelles Windows inactives déconnectées, utilisez la combinaison des paramètres **Déconnecter les utilisateurs quand des machines virtuelles sont inactives** et **Arrêter les machines virtuelles quand les utilisateurs se déconnectent**.

Par exemple, si vous configurez les paramètres comme suit :
 
* **Déconnecter les utilisateurs quand les machines virtuelles sont inactives** : 15 minutes après la détection de l'état d'inactivité.
* **Arrêter des machines virtuelles quand les utilisateurs se déconnectent** : 5 minutes après la déconnexion de l'utilisateur.
 
Les machines virtuelles Windows s’arrêtent automatiquement 20 minutes après que l’utilisateur cesse de les utiliser. 
 
> [!div class="mx-imgBorder"]
> ![Diagramme illustrant la combinaison de paramètres entraînant l’arrêt automatique de la machine virtuelle.](./media/cost-management-guide/vm-idle-diagram.png)

#### <a name="automatically-shut-down-virtual-machines-when-users-disconnect"></a>Arrêt automatique des machines virtuelles quand les utilisateurs se déconnectent
 
Le paramètre **Arrêter des machines virtuelles lorsque les utilisateurs se déconnectent** prend en charge les machines virtuelles Windows et Linux. Lorsque ce paramètre est activé, l’arrêt automatique se produit dans les cas suivants :
 
* Pour Windows, une connexion Bureau à distance (RDP) est déconnectée.
* Pour Linux, une connexion SSH est déconnectée.
 
> [!NOTE]
> Seules [des distributions et versions spécifiques de Linux](../virtual-machines/extensions/diagnostics-linux.md#supported-linux-distributions) sont prises en charge.
 
Vous pouvez spécifier la durée pendant laquelle les machines virtuelles doivent attendre que l’utilisateur se reconnecte avant de s’arrêter automatiquement. 

#### <a name="automatically-shut-down-virtual-machines-that-are-started-but-users-dont-connect"></a>Arrêt automatique des machines virtuelles démarrées, mais auxquelles les utilisateurs ne se connectent pas.
 
Dans un labo, un utilisateur peut démarrer une machine virtuelle sans jamais s’y connecter. Par exemple :
 
* Une planification dans le labo permet de démarrer toutes les machines virtuelles pour une session de classe, mais certains étudiants ne se présentent pas et ne se connectent pas à leurs machines. 
* Un utilisateur démarre une machine virtuelle, mais oublie de se connecter. 
 
Le paramètre **Arrêter des machines virtuelles quand les utilisateurs ne se connectent pas** détecte ces cas et arrête automatiquement les machines virtuelles. 
 
Pour plus d’informations sur la configuration et l’activation de l’arrêt automatique des machines virtuelles lors de la déconnexion, consultez les articles suivants :

* [Configurer l’arrêt automatique des machines virtuelles pour un compte labo](how-to-configure-lab-accounts.md)
* [Configurer l’arrêt automatique des machines virtuelles pour un labo](how-to-enable-shutdown-disconnect.md)

### <a name="scheduled-time-vs-quota-time"></a>Heure planifiée contre heure du quota

Comprendre la différence entre [heure planifiée](classroom-labs-concepts.md#schedules) et [heure du quota](classroom-labs-concepts.md#quota) vous permettra de configurer un labo pour qu’il réponde mieux aux besoins du professeur et des étudiants. 

L’heure planifiée est une heure définie à laquelle toutes les machines virtuelles des étudiants ont été démarrées et sont disponibles pour la connexion. L’heure planifiée est couramment utilisée lorsque tous les étudiants ont leurs propres machines virtuelles et suivent les instructions du professeur à une heure définie de la journée (comme les heures de classe). L’inconvénient est que toutes les machines virtuelles des étudiants sont démarrées et qu’elles engendrent des coûts, même si un étudiant ne s’y connecte pas. 

L’heure de quota correspond au temps alloué à chaque étudiant pour une utilisation à sa discrétion et sert souvent aux sessions d’étude autonome. Les machines virtuelles ne sont pas démarrées tant que l’étudiant ne les démarre pas. 

Un labo peut utiliser l’heure de quota, l’heure planifiée ou une combinaison des deux. Si une classe n’a pas besoin de l’heure planifiée, utilisez uniquement l’heure de quota pour une utilisation optimale des machines virtuelles.

### <a name="scheduled-event-stop-only"></a>Événement planifié : arrêt uniquement

Dans la planification, vous pouvez ajouter un type d’événement d’arrêt uniquement, qui arrêtera toutes les machines à un moment donné. Certains propriétaires de labo ont défini un événement d’arrêt uniquement tous les jours à minuit pour réduire le coût et l’utilisation du quota lorsqu’un étudiant oublie d’éteindre la machine virtuelle qu’il utilise. L’inconvénient de ce type d’événement est que toutes les machines virtuelles seront arrêtées même si l’étudiant en utilise une.

### <a name="other-costs-related-to-labs"></a>Autres coûts liés aux labos 

Il existe des coûts qui ne sont pas intégrés à Lab Services, mais qui peuvent être liés à un service lab. Vous pouvez vous connecter à une galerie d’images partagées sur un labo, mais il ne s’affiche pas sous les coûts de Lab Services et n’engendre pas de coûts. Pour réduire les coûts globaux, vous devez supprimer toutes les images inutilisées de la galerie, car elles représentent un coût de stockage hérité. 

Les labos peuvent avoir des connexions à d’autres ressources Azure via un réseau virtuel. En cas de suppression d’un labo, vous devez supprimer le réseau virtuel et les autres ressources.

## <a name="conclusion"></a>Conclusion

Nous espérons que les informations contenues dans cet article vous ont donné une meilleure compréhension des outils qui peuvent vous aider à réduire les coûts d’utilisation.