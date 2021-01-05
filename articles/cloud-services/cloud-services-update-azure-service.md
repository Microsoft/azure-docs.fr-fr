---
title: Mise à jour d’un service cloud | Microsoft Docs
description: Découvrez comment mettre à jour des services cloud dans Azure. Découvrez comment mettre à jour un service cloud se poursuit pour garantir la disponibilité.
services: cloud-services
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 04/19/2017
ms.author: tagore
ms.openlocfilehash: f12e5b6b0b2902d69936b9cf2695b7ee21db88e2
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075040"
---
# <a name="how-to-update-a-cloud-service"></a>Mettre à jour un service cloud

La mise à jour d’un service cloud, et notamment de ses rôles et du système d’exploitation invité, est un processus qui se déroule en trois étapes. Tout d’abord, les fichiers binaires et de configuration du nouveau Service Cloud et de la version de système d’exploitation doivent être téléchargés. Dans un deuxième temps, Azure réserve des ressources de calcul et réseau au Service Cloud selon les spécifications de la nouvelle version de Service Cloud. Enfin, Azure effectue une mise à niveau propagée pour effectuer une mise à jour incrémentielle du client vers la nouvelle version ou le nouveau système d’exploitation invité tout en préservant votre disponibilité. Cet article discute des détails de cette dernière étape, la mise à jour propagée.

## <a name="update-an-azure-service"></a>Mettre à jour un Service Azure
Azure organise vos instances de rôle en regroupements logiques appelés domaines de mise à niveau (UD). Les domaines de mise à niveau (UD) sont des ensembles logiques d’instances de rôle qui sont mis à jour en tant que groupe.  Azure met à jour un domaine de mise à niveau de Service Cloud à la fois, ce qui permet aux instances présentes sur les autres domaines de mise à niveau de maintenir le trafic.

Le nombre de domaines de mise à niveau par défaut est de 5. Vous pouvez spécifier un nombre différent de domaines de mise à niveau en incluant l’attribut upgradeDomainCount dans le fichier de définition du service (.csdef). Pour plus d’informations sur l’attribut upgradeDomainCount, consultez [Schéma de définition Microsoft Azure Cloud Services (fichier .csdef)](./schema-csdef-file.md).

Lorsque vous effectuez la mise à jour sur place d’un ou de plusieurs rôles dans votre service, Azure met à jour les ensembles d’instances de rôle en fonction du domaine de mise à niveau auquel ils appartiennent. Azure met à jour toutes les instances dans un domaine de mise à niveau donné (les arrête, les met à jour, les remet en ligne) puis passe au domaine suivant. En arrêtant uniquement les instances en cours d’exécution dans le domaine de mise à niveau en cours, Azure garantit que l’opération aura un impact minimal sur le service en cours d’exécution. Pour plus d’informations, consultez [Déroulement de la mise à niveau](#howanupgradeproceeds) plus loin dans cet article.

> [!NOTE]
> Bien que les termes **mise à jour** et **mise à niveau** aient une signification légèrement différente dans le contexte Azure, ils peuvent être utilisés indifféremment pour les processus et les descriptions des fonctionnalités du présent document.
>
>

Votre service doit définir au moins deux instances d’un rôle pour que le rôle soit mis à jour sur place, sans interruption de service. Si le service se compose d’une seule instance de rôle, votre service sera indisponible jusqu’à la fin de la mise à jour sur place.

Cette rubrique décrit les informations suivantes sur les mises à jour Azure :

* [Modifications de service autorisées pendant une mise à jour](#AllowedChanges)
* [Déroulement d’une mise à niveau](#howanupgradeproceeds)
* [Restauration d’une mise à jour](#RollbackofanUpdate)
* [Lancement de plusieurs opérations de mutation sur un déploiement en cours](#multiplemutatingoperations)
* [Distribution des rôles entre domaines de mise à niveau](#distributiondfroles)

<a name="AllowedChanges"></a>

## <a name="allowed-service-changes-during-an-update"></a>Modifications de service autorisées pendant une mise à jour
Le tableau suivant présente les modifications de service autorisées au cours d’une mise à jour :

| Modifications autorisées de l’hébergement, des services et des rôles | Mise à jour sur place | Intermédiaire (échange d’adresses IP virtuelles) | Supprimer et redéployer |
| --- | --- | --- | --- |
| Version du système d'exploitation |Oui |Oui |Oui |
| Niveau de confiance .NET |Oui |Oui |Oui |
| Taille de la machine virtuelle<sup>1</sup> |Oui<sup>2</sup> |Oui |Oui |
| Paramètres de stockage locaux |Augmentation uniquement<sup>2</sup> |Oui |Oui |
| Ajouter et supprimer les rôles dans un service |Oui |Oui |Oui |
| Nombre d’instances d’un rôle particulier |Oui |Oui |Oui |
| Nombre ou type de points de terminaison pour un service |Oui<sup>2</sup> |Non |Oui |
| Noms et valeurs de paramètres de configuration |Oui |Oui |Oui |
| Valeurs (et non noms) des paramètres de configuration |Oui |Oui |Oui |
| Ajouter de nouveau certificats |Oui |Oui |Oui |
| Modifier les certificats existants |Oui |Oui |Oui |
| Déployer un nouveau code |Oui |Oui |Oui |

<sup>1</sup> Modification de la taille limitée au sous-ensemble des tailles disponibles pour le service cloud.

<sup>2</sup> Nécessite le kit de développement logiciel (SDK) Azure 1.5 ou versions ultérieures.

> [!WARNING]
> La modification de la taille de machine virtuelle détruira les données locales.
>
>

Les éléments suivants ne sont pas pris en charge pendant une mise à jour :

* Modification du nom d’un rôle. Supprimez, puis ajoutez le rôle avec le nouveau nom.
* Modification du nombre de domaines de mise à niveau.
* Réduction de la taille des ressources locales.

Si vous exécutez d’autres mises à jour de votre définition de service, comme la réduction des ressources de taille locale, vous devez exécuter une mise à jour de l’échange d’adresses IP virtuelles. Pour plus d’informations, consultez [Déploiement d’échange](/previous-versions/azure/reference/ee460814(v=azure.100)).

<a name="howanupgradeproceeds"></a>

## <a name="how-an-upgrade-proceeds"></a>Déroulement d’une mise à niveau
Vous pouvez décider si vous souhaitez mettre à jour tous les rôles de votre service ou un seul rôle de ce service. Dans les deux cas, toutes les instances de chaque rôle mis à niveau qui appartiennent au premier domaine de mise à niveau sont arrêtées, mises à niveau et remises en ligne. Une fois de retour en ligne, les instances du deuxième domaine de mise à niveau sont arrêtées, mises à niveau et remises en ligne. Un service cloud peut avoir au plus une mise à niveau active à la fois. La mise à niveau s’effectue toujours sur la dernière version du service cloud.

Le diagramme suivant montre comment la mise à niveau se poursuit si vous mettez à niveau tous les rôles dans le service :

![Mettre à niveau le service](media/cloud-services-update-azure-service/IC345879.png "Mettre à niveau le service")

Le diagramme suivant montre comment la mise à jour se déroule en cas de mise à jour d’un seul rôle :

![Mettre à niveau le rôle](media/cloud-services-update-azure-service/IC345880.png "Mettre à niveau le rôle")  

Pendant une mise à jour automatique, le contrôleur de structure Azure évalue périodiquement l’intégrité du service cloud pour déterminer le moment opportun pour passer au domaine de mise à niveau suivant. Cette évaluation de l’état est effectuée sur une base par rôle et considère uniquement les instances de la dernière version (c’est-à-dire les instances de domaines de mise à niveau qui ont déjà été examinées). Il vérifie qu’un nombre minimum d’instances de rôle, pour chaque rôle, a atteint un état terminal satisfaisant.

### <a name="role-instance-start-timeout"></a>Délai de démarrage de l’instance de rôle
Le contrôleur de structure attend 30 minutes pour que chaque instance de rôle atteigne un état démarré. Si la durée d'expiration est écoulée, le contrôleur de structure continuera à remonter jusqu’à l'instance de rôle suivante.

### <a name="impact-to-drive-data-during-cloud-service-upgrades"></a>Impact sur les données de lecteur lors de la mise à jour des services cloud

Lors de la mise à niveau d’un service d’instance unique vers plusieurs instances, votre service est réduit pendant que la mise à niveau est exécutée, en fonction de la façon dont Azure met à niveau les services. Le contrat de niveau de service garantissant la disponibilité du service ne s’applique qu’aux services déployés avec plusieurs instances. La liste suivante décrit la façon dont les données de chaque lecteur sont affectées à chaque scénario de mise à niveau de service Azure :

|Scénario|Lecteur C|Lecteur D|Lecteur E|
|--------|-------|-------|-------|
|Redémarrage de la machine virtuelle|Préservé|Préservé|Préservé|
|Redémarrage du portail|Préservé|Préservé|Détruit|
|Réinitialisation du portail|Préservé|Détruit|Détruit|
|Mise à niveau sur place|Préservé|Préservé|Détruit|
|Migration des nœuds|Détruit|Détruit|Détruit|

Notez que, dans la liste ci-dessus, le lecteur E: représente le lecteur racine du rôle et qu’il ne doit pas être codées en dur. Utilisez plutôt la variable d’environnement **%RoleRoot%** pour représenter le lecteur.

Pour réduire les temps d’arrêt en cas de mise à niveau d’un service à instance unique, déployez un nouveau service à instances multiples sur le serveur intermédiaire et effectuez un échange d’adresses IP virtuelles.

<a name="RollbackofanUpdate"></a>

## <a name="rollback-of-an-update"></a>Restauration d’une mise à jour
Azure offre une flexibilité dans la gestion des services pendant la mise à jour en vous permettant de lancer des opérations supplémentaires sur un service, une fois la demande de mise à jour initiale acceptée par le contrôleur d’architecture Azure. Une restauration ne peut être effectuée que lorsqu’une mise à jour (modification de configuration) ou une mise à niveau se trouve dans l’état **en cours** du déploiement. Une mise à jour ou mise à niveau est considérée comme en cours tant qu’il existe au moins une instance du service qui n’a pas encore été mise à jour vers la nouvelle version. Pour vérifier si une restauration est autorisée, assurez-vous que la valeur de l’indicateur RollbackAllowed retournée par les opérations [Obtention du déploiement](/previous-versions/azure/reference/ee460804(v=azure.100)) et [Obtention des propriétés de service cloud](/previous-versions/azure/reference/ee460806(v=azure.100)) est bien définie sur true.

> [!NOTE]
> Il convient d’appeler la restauration uniquement sur une mise à jour ou une mise à niveau **sur place** , parce que les mises à niveau d’échange d’adresse virtuelle impliquent le remplacement d’une instance complète s’exécutant sur votre service avec une autre.
>
>

Le rétablissement d’une mise à jour en cours a les effets suivants sur le déploiement :

* Toutes les instances de rôle n’ayant pas encore été mises à jour ou à niveau vers la nouvelle version ne sont ni mises à jour ni mises à niveau, car les instances s’exécutent déjà sur la version cible du service.
* Toutes les instances de rôle déjà mises à jour ou à niveau vers la nouvelle version du fichier de package de service (\*.cspkg) ou le fichier de configuration (\*.cscfg) (ou les deux fichiers) sont rétablies vers la version précédant la mise à niveau de ces fichiers.

Cette fonction est assurée par les fonctionnalités suivantes :

* La [restauration de mise à jour ou de mise à niveau](/previous-versions/azure/reference/hh403977(v=azure.100)), qui peut être appelée sur une mise à jour de configuration (déclenchée par un appel [Modifier la configuration du déploiement](/previous-versions/azure/reference/ee460809(v=azure.100))) ou une mise à niveau (déclenchée par l’appel [Mettre à niveau un déploiement](/previous-versions/azure/reference/ee460793(v=azure.100))) tant qu’il reste au moins une instance du service qui n’a pas encore été mise à jour vers la nouvelle version.
* L’élément Verrouillé et l’élément RollbackAllowed, qui sont retournés comme parties intégrantes du corps de réponse des opérations [Obtention du déploiement](/previous-versions/azure/reference/ee460804(v=azure.100)) et [Obtention des propriétés de service cloud](/previous-versions/azure/reference/ee460806(v=azure.100)) :

  1. L’élément Verrouillé permet de détecter si une opération de mutation peut être appelée sur un déploiement donné.
  2. L’élément RollbackAllowed vous permet de détecter lorsque l’opération de [restauration de mise à jour ou de mise à niveau](/previous-versions/azure/reference/hh403977(v=azure.100)) peut être appelée sur un déploiement donné.

  Pour effectuer une restauration, il est inutile de vérifier les éléments Verrouillés et RollbackAllowed. Il suffit de vérifier que RollbackAllowed est défini sur true. Ces éléments sont retournés uniquement si ces méthodes sont appelées avec l’en-tête de demande défini sur « x-ms-version : 2011-10-01 » ou une version ultérieure. Pour plus d’informations sur les en-têtes de contrôle de version, consultez [Contrôle de version de gestion de service](/previous-versions/azure/gg592580(v=azure.100)).

Dans certaines situations, la restauration d’une mise à jour ou d’une mise à niveau n’est pas prise en charge, notamment les suivantes :

* Réduction des ressources locale : si la mise à jour augmente les ressources locales d’un rôle, la plateforme Azure n’autorise pas la restauration.
* Limitations de quota : si la mise à jour correspond à une opération de réduction, vous pouvez ne plus avoir suffisamment de capacité de calcul pour effectuer l’opération de restauration. Chaque abonnement Azure a un quota associé qui spécifie le nombre maximal de cœurs qui peuvent être utilisés par les services hébergés appartenant à cet abonnement. Si l’exécution de la restauration d’une mise à jour donnée met votre abonnement au dessus du quota, la restauration ne sera pas activée.
* Condition de concurrence : si la mise à jour initiale est terminée, la restauration est impossible.

Exemple de la situation où le déploiement d’une mise à jour peut être utile si vous utilisez l’opération de [mise à niveau de déploiement](/previous-versions/azure/reference/ee460793(v=azure.100)) en mode manuel pour contrôler la fréquence à laquelle la mise à niveau majeure d’un service hébergé Azure est exécutée.

Lors du déploiement de la mise à niveau, vous appelez [Mettre à niveau un déploiement](/previous-versions/azure/reference/ee460793(v=azure.100)) en mode manuel et commencez à parcourir les domaines de mise à niveau. Si à un moment donné, lorsque vous surveillez la mise à niveau, vous notez que des instances de rôle dans les premiers domaines de mise à niveau ne répondent plus, vous pouvez appeler une opération de [restauration de mise à jour ou de mise à niveau](/previous-versions/azure/reference/hh403977(v=azure.100)) sur le déploiement, ce qui laissera les instances non mises à jour intactes, et restaurera les instances ayant été mises à niveau vers le package et la configuration de service à leur niveau antérieur.

<a name="multiplemutatingoperations"></a>

## <a name="initiating-multiple-mutating-operations-on-an-ongoing-deployment"></a>Lancement de plusieurs opérations de mutation sur un déploiement en cours
Dans certains cas, vous pouvez souhaiter initier plusieurs opérations de mutation simultanées sur un déploiement en cours. Par exemple, vous pouvez effectuer une mise à jour du service et, pendant la propagation de la mise à jour sur votre service, vous pouvez apporter des modifications, par exemple, restaurer la mise à jour, en appliquer une autre, ou encore supprimer le déploiement. Cela peut s’avérer nécessaire si une mise à niveau de service contient un code bogué qui entraîne des échecs répétés d’une instance de rôle mise à niveau. Dans ce cas, le contrôleur de structure Azure ne sera pas en mesure de faire avancer l’application de cette mise à jour, parce qu’un nombre insuffisant d’instances du domaine mis à niveau sont dans un bon état. Cet état est appelé *Déploiement bloqué*. Vous pouvez débloquer le déploiement en restaurant la mise à jour ou en appliquant une nouvelle mise à jour par dessus celle qui a échoué.

Une fois que le contrôleur de structure Azure a reçu la demande initiale de mise à jour ou de mise à niveau, vous pouvez démarrer des opérations de mutation suivantes. Autrement dit, il est inutile d’attendre la fin de l’opération initiale avant de commencer une autre opération de mutation.

Le lancement d’une deuxième opération de mise à jour pendant que la première mise à jour est en cours permet d’effectuer une opération similaire à l’opération de restauration. Si la deuxième mise à jour est en mode automatique, le premier domaine de mise à niveau sera immédiatement, mis à niveau, ce qui pourra éventuellement provoquer la mise hors ligne de plusieurs domaines de mise à niveau hors ligne au même moment dans le temps.

Les opérations de mutation sont les suivantes : [Modification de la configuration du déploiement](/previous-versions/azure/reference/ee460809(v=azure.100)), [Mise à niveau du déploiement](/previous-versions/azure/reference/ee460793(v=azure.100)), [Mise à jour de l’état du déploiement](/previous-versions/azure/reference/ee460808(v=azure.100)), [Suppression du déploiement](/previous-versions/azure/reference/ee460815(v=azure.100)) et [Restauration de mise à jour ou de mise à niveau](/previous-versions/azure/reference/hh403977(v=azure.100)).

Deux opérations, [Obtention du déploiement](/previous-versions/azure/reference/ee460804(v=azure.100)) et [Obtention des propriétés de service cloud](/previous-versions/azure/reference/ee460806(v=azure.100)), retournent l’indicateur Verrouillé qui peut être examiné pour déterminer si une opération de mutation peut être appelée sur un déploiement donné.

Pour appeler la version de ces méthodes qui renvoie un indicateur Verrouillé, vous devez définir un en-tête de requête « x-ms-version: 2011-10-01 » ou ultérieure. Pour plus d’informations sur les en-têtes de contrôle de version, consultez [Contrôle de version de gestion de service](/previous-versions/azure/gg592580(v=azure.100)).

<a name="distributiondfroles"></a>

## <a name="distribution-of-roles-across-upgrade-domains"></a>Distribution des rôles entre domaines de mise à niveau
Azure répartit les instances d’un rôle de manière égale sur un certain nombre de domaines de mise à niveau, qui peuvent être configurés dans le cadre du fichier de définition de service (.csdef). Le nombre maximal de domaines de mise à niveau est de 20, et le nombre par défaut est 5. Pour plus d’informations sur la façon de modifier le fichier de définition de service, consultez [Schéma de définition du service Azure (fichier .csdef)](cloud-services-model-and-package.md#csdef).

Par exemple, si votre rôle comporte dix instances, par défaut, chaque domaine de mise à niveau contient deux instances. Si votre rôle comporte 14 instances, alors quatre des domaines de mise à niveau contiennent trois instances et un cinquième domaine en contient deux.

Les domaines de mise à niveau sont identifiés avec un index de base zéro : l’ID du premier domaine de mise à niveau est égal à 0, et le deuxième domaine de mise à niveau possède un ID de 1 et ainsi de suite.

Le diagramme suivant montre comment un service contenant deux rôles qui sont distribués lorsque le service définit deux domaines de mise à niveau. Le service exécute huit instances de rôle web et neuf instances de rôle de travail.

![Distribution des domaines de mise à niveau](media/cloud-services-update-azure-service/IC345533.png "Distribution des domaines de mise à niveau")

> [!NOTE]
> Notez qu’Azure contrôle la façon dont les instances sont affectées entre d’un domaine de mise à niveau à l’autre. Il est impossible de spécifier quelles sont les instances affectées, et à quel domaine.
>
>

## <a name="next-steps"></a>Étapes suivantes
[Gestion des services cloud](cloud-services-how-to-manage-portal.md)  
[Surveillance des services cloud](cloud-services-how-to-monitor.md)  
[Configuration des services cloud](cloud-services-how-to-configure-portal.md)