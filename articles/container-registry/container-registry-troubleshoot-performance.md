---
title: Résoudre les problèmes de performances de registre
description: Symptômes, causes et résolution des problèmes courants liés aux performances d’un registre
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 15129ebe1da2e52fac106a34863f609c440549ad
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148403"
---
# <a name="troubleshoot-registry-performance"></a>Résoudre les problèmes de performances de registre

Cet article vous aide à résoudre des problèmes que vous pourriez rencontrer en lien avec les performances d’un registre de conteneurs Azure. 

## <a name="symptoms"></a>Symptômes

Peuvent inclure un ou plusieurs des symptômes suivants :

* L’envoi et l’extraction d’images avec l’interface de ligne de commande de Docker prennent plus de temps que prévu.
* Le déploiement d’images sur un service tel qu’Azure Kubernetes Service prend plus de temps que prévu.
* Vous ne parvenez pas à effectuer un grand nombre d’opérations d’envoi ou d’extraction simultanées dans le temps prévu.
* Les opérations d’envoi ou d’extraction dans un registre géorépliqué prennent plus de temps que prévu, ou les opérations d’envoi échouent avec l’erreur `Error writing blob` ou `Error writing manifest`.

## <a name="causes"></a>Causes

* La vitesse de votre connexion réseau peut ralentir les opérations de Registre : [solution](#check-expected-network-speed).
* La compression ou l’extraction de la couche d’image peut être lente sur le client : [solution](#check-client-hardware).  
* Vous atteignez une limite configurée dans le niveau de service ou l’environnement de votre registre : [solution](#review-configured-limits).
* Votre registre géorépliqué a des réplicas dans des régions avoisinantes : [solution](#configure-geo-replicated-registry).
* Vous effectuez une extraction à partir d’un réplica de registre géographiquement distant : [solution](#configure-dns-for-geo-replicated-registry).

Si vous ne résolvez pas votre problème ici, consultez [Résolution de problèmes avancée](#advanced-troubleshooting) et [Étapes suivantes](#next-steps) pour d’autres options.

## <a name="potential-solutions"></a>Solutions possibles

### <a name="check-expected-network-speed"></a>Vérifier la vitesse du réseau attendue

Vérifiez votre vitesse de chargement et de téléchargement Internet, ou servez-vous d’un outil tel qu’AzureSpeed pour tester le [chargement](https://www.azurespeed.com/Azure/Uploadß) et le [téléchargement](https://www.azurespeed.com/Azure/Download) à destination/à partir du service Stockage Blob Azure, qui héberge les couches d’image du registre.

Vérifiez votre taille d’image par rapport à la taille maximale prise en charge, ainsi qu’à la bande passante de chargement et de téléchargement prise en charge pour le niveau de service de votre registre. Si votre registre est au niveau De base ou Standard, envisagez une mise à niveau pour améliorer les performances. 

Pour le déploiement d’images vers d’autres services, vérifiez les régions dans lesquelles se trouvent le registre et la cible. Envisagez de placer le registre et la cible de déploiement dans la même région ou dans des régions proches du réseau afin d’améliorer les performances.

Liens connexes :

* [Niveaux de service Azure Container Registry](container-registry-skus.md)    
* [FAQ sur le registre de conteneurs](container-registry-faq.md)
* [Objectifs de performances et de scalabilité pour le service Stockage Blob Azure](../storage/blobs/scalability-targets.md)

### <a name="check-client-hardware"></a>Vérifier le matériel du client

Le type de disque et le processeur sur le client Docker peuvent affecter la vitesse d’extraction ou de compression des couches d’image sur le client dans le cadre des opérations d’extraction ou d’envoi. Par exemple, l’extraction de la couche sur un lecteur de disque dur prendra plus de temps que sur un disque SSD. Comparez les opérations d’extraction pour des images comparables à partir de votre registre de conteneurs Azure et d’un registre public tel que Docker Hub.

### <a name="review-configured-limits"></a>Examiner les limites configurées

Si vous envoyez et extrayez simultanément de nombreuses images multicouches à destination/à partir de votre registre, examinez les limites de ReadOps et WriteOps prises en charge pour le niveau de service du registre. Si votre registre est au niveau De base ou Standard, envisagez une mise à niveau pour augmenter les limites. Vérifiez également auprès de votre fournisseur de réseau la limitation du réseau qui peut se produire en présence de nombreuses opérations simultanées. 

Examinez la configuration de votre démon Docker pour connaître le nombre maximal de chargements ou téléchargements simultanés pour chaque opération d’envoi ou d’extraction sur le client. Configurez des limites plus élevées si nécessaire.

Chaque couche d’image nécessitant une opération de lecture ou d’écriture de registre distincte, vérifiez le nombre de couches dans vos images. Envisagez des stratégies permettant de réduire le nombre de couches d’image.

Liens connexes :

* [Niveaux de service Azure Container Registry](container-registry-skus.md)
* [dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/)

### <a name="configure-geo-replicated-registry"></a>Configuré un registre géorépliqué

Un client Docker qui transmet une image à un registre géorépliqué peut ne pas envoyer toutes les couches d’image et le manifeste associé à une seule région répliquée. Cette situation peut se produire, car Azure Traffic Manager route les demandes de registre vers le registre répliqué le plus proche du réseau. Si le registre a deux régions de réplication proches, les couches d’image et le manifeste peuvent être distribués aux deux sites, et l’opération Push échoue lors de la validation du manifeste.

Pour optimiser la résolution DNS sur le réplica le plus proche lors de la transmission par push d’images, configurez un registre géorépliqué dans les mêmes régions Azure que la source des opérations Push, ou la région la plus proche si vous travaillez en dehors d’Azure.

Pour résoudre les problèmes des opérations avec un registre géorépliqué, vous pouvez également désactiver temporairement le routage de Traffic Manager vers une ou plusieurs réplications.

Liens connexes :

* [Géoréplication dans Azure Container Registry](container-registry-geo-replication.md)

### <a name="configure-dns-for-geo-replicated-registry"></a>Configurer un DNS pour le registre géorépliqué

Si les opérations d’extraction à partir d’un registre géorépliqué semblent lentes, la configuration du DNS sur le client peut se résoudre en un serveur DNS géographiquement distant. Dans ce cas, Traffic Manager peut acheminer les demandes adressées à un réplica proche du réseau vers le serveur DNS éloigné du client. Exécutez un outil tel que `nslookup` ou `dig` (sur Linux) pour déterminer le réplica vers lequel Traffic Manager achemine les demandes du registre. Par exemple :

```console
nslookup myregistry.azurecr.io
```

Une solution possible consiste à configurer un serveur DNS plus proche.

Liens connexes :

* [Géoréplication dans Azure Container Registry](container-registry-geo-replication.md)
* [Résoudre les problèmes d’opérations Push avec des registres géorépliqués](container-registry-geo-replication.md#troubleshoot-push-operations-with-geo-replicated-registries)
* [Désactiver temporairement le routage vers une réplication](container-registry-geo-replication.md#temporarily-disable-routing-to-replication)
* [FAQ sur Traffic Manager](../traffic-manager/traffic-manager-faqs.md)

### <a name="advanced-troubleshooting"></a>Dépannage avancé

Si vos autorisations d’accès aux ressources du registre le permettent, [vérifiez l’intégrité de l’environnement du registre](container-registry-check-health.md). Si des erreurs sont signalées, consultez les [Informations de référence sur les erreurs](container-registry-health-error-reference.md) pour trouver des solutions potentielles.

Si la [collecte des journaux de ressources](container-registry-diagnostics-audit-logs.md) est activée dans le registre, consultez le journal ContainterRegistryRepositoryEvents. Ce journal stocke des informations sur des opérations telles que des événements d’envoi ou d’extraction. Interrogez le journal sur les [échecs d’opérations au niveau du référentiel](container-registry-diagnostics-audit-logs.md#repository-level-operation-failures). 

Liens connexes :

* [Journaux pour l’évaluation et l’audit de diagnostics](container-registry-diagnostics-audit-logs.md)
* [FAQ sur le registre de conteneurs](container-registry-faq.md)
* [Meilleures pratiques pour Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Étapes suivantes

Si vous ne résolvez pas votre problème ici, consultez les options suivantes.

* Les autres rubriques de dépannage du registre sont les suivantes :
  * [Résoudre les problèmes de connexion au registre](container-registry-troubleshoot-login.md)
  * [Résoudre les problèmes de réseau avec le Registre](container-registry-troubleshoot-access.md)
* Options de [support de la communauté](https://azure.microsoft.com/support/community/)
* [Microsoft Q&A](/answers/products/)
* [Ouvrir un ticket de support](https://azure.microsoft.com/support/create-ticket/)