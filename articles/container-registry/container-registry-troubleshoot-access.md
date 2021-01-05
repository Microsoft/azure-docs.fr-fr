---
title: Résoudre des problèmes de réseau avec un registre
description: Symptômes, causes et résolution de problèmes courants lors de l’accès à un registre de conteneurs Azure dans un réseau virtuel ou derrière un pare-feu
ms.topic: article
ms.date: 10/01/2020
ms.openlocfilehash: 95b32b839d1b3b804a2035b797e1146a09d5236a
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351791"
---
# <a name="troubleshoot-network-issues-with-registry"></a>Résoudre des problèmes de réseau avec un registre

Cet article vous aide à résoudre les problèmes que vous pouvez rencontrer lors de l’accès à un registre de conteneurs Azure dans un réseau virtuel ou derrière un pare-feu. 

## <a name="symptoms"></a>Symptômes

Peuvent inclure un ou plusieurs des symptômes suivants :

* Impossibilité d’envoyer ou d’extraire des images et affichage de l’erreur `dial tcp: lookup myregistry.azurecr.io`.
* Impossibilité d’envoyer ou d’extraire des images et affichage de l’erreur Azure CLI `Could not connect to the registry login server`.
* Impossibilité d’extraire des images du registre vers Azure Container Service ou un autre service Azure.
* Impossibilité d’accéder au registre derrière un proxy HTTPS et réception de l’erreur `Error response from daemon: login attempt failed with status: 403 Forbidden`.
* Impossibilité de configurer les paramètres de réseau virtuel et réception de l’erreur `Failed to save firewall and virtual network settings for container registry`
* Impossibilité de consulter ou d’afficher des paramètres du registre dans le portail Azure ou de gérer le registre à l’aide d’Azure CLI.
* Impossibilité d’ajouter ou de modifier des paramètres de réseau virtuel ou des règles d’accès public.
* La solution ACR Tasks ne peut pas envoyer ou extraire des images.
* Azure Security Center ne peut pas analyser les images figurant dans le registre, ou les résultats d’analyse n’apparaissent pas dans Azure Security Center.

## <a name="causes"></a>Causes

* Un pare-feu ou proxy de client empêche l’accès : [solution](#configure-client-firewall-access).
* Les règles d’accès au réseau public dans le registre empêchent l’accès : [solution](#configure-public-access-to-registry).
* La configuration du réseau virtuel empêche l’accès : [solution](#configure-vnet-access).
* Vous tentez d’intégrer Azure Security Center ou certains autres services Azure avec un registre qui a un point de terminaison privé, un point de terminaison de service ou des règles d’accès aux IP publiques : [solution](#configure-service-access).

## <a name="further-diagnosis"></a>Diagnostics plus poussés 

Exécutez la commande [az acr check-health](/cli/azure/acr#az-acr-check-health) pour obtenir des informations supplémentaires sur l’intégrité de l’environnement de registre, et éventuellement l’accès à un registre cible. Par exemple, diagnostiquez certains problèmes de connectivité ou de configuration de réseau. 

Consultez [Vérifier l’intégrité d’un registre de conteneurs Azure](container-registry-check-health.md) pour obtenir des exemples de commandes. Si des erreurs sont signalées, examinez les [Informations de référence sur les erreurs](container-registry-health-error-reference.md) et les sections suivantes pour obtenir les solutions recommandées.

> [!NOTE]
> Certains symptômes de connectivité réseau peuvent également se produire en cas de problème avec l’authentification ou l’autorisation du registre. Consultez [Résoudre les problèmes de connexion au registre](container-registry-troubleshoot-login.md).

## <a name="potential-solutions"></a>Solutions possibles

### <a name="configure-client-firewall-access"></a>Configurer l’accès au pare-feu du client

Pour accéder à un registre à partir d’un serveur proxy ou d’un emplacement situé derrière un pare-feu de client, configurez des règles de pare-feu pour accéder aux points de terminaison REST public et de données du registre. Si des [points de terminaison de données dédiés](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) sont activés, vous avez besoin de règles pour accéder aux éléments suivants :

* Point de terminaison REST : `<registryname>.azurecr.io`
* Point de terminaison de données : `<registry-name>.<region>.data.azurecr.io`

Pour un registre géorépliqué, configurez l’accès au point de terminaison de données pour chaque réplica régional.

Derrière un proxy HTTPS, vérifiez que vos client et démon Docker sont configurés pour le comportement du proxy.

Les journaux de ressources de registre dans la table ContainerRegistryLoginEvents peuvent aider à diagnostiquer une tentative de connexion bloquée.

Liens connexes :

* [Configurer des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu](container-registry-firewall-access-rules.md)
* [Configuration du proxy HTTP/HTTPS](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
* [Géoréplication Azure Container Registry](container-registry-geo-replication.md)
* [Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics](container-registry-diagnostics-audit-logs.md)

### <a name="configure-public-access-to-registry"></a>Configurer un accès public à un registre

Si vous accédez à un registre via Internet, vérifiez que le registre autorise l’accès au réseau public à partir de votre client. Par défaut, un registre de conteneurs Azure permet d’accéder aux points de terminaison de registre publics à partir de tous les réseaux. Un registre peut limiter l’accès à des réseaux sélectionnés ou à des adresses IP sélectionnées. 

Si le registre est configuré pour un réseau virtuel avec un point de terminaison de service, la désactivation de l’accès au réseau public a également pour effet de désactiver l’accès via le point de terminaison de service. Si votre registre est configuré pour un réseau virtuel avec une liaison privée, les règles de réseau IP ne s’appliquent pas aux points de terminaison privés du registre. 

Liens connexes :

* [Configurer des règles de réseau IP public](container-registry-access-selected-networks.md)
* [Connexion privée à un registre de conteneurs Azure à l’aide d’Azure Private Link](container-registry-private-link.md)
* [Restreindre l’accès à un registre de conteneurs à l’aide d’un point de terminaison de service dans un réseau virtuel Azure](container-registry-vnet.md)


### <a name="configure-vnet-access"></a>Configurer l’accès au réseau virtuel

Vérifiez que le réseau virtuel est configuré avec un point de terminaison privé pour une liaison privée ou un point de terminaison de service (préversion). Actuellement, un point de terminaison Azure Bastion n’est pas pris en charge.

Examinez les règles de groupe de sécurité réseau et les balises de service utilisées pour limiter le trafic à partir d’autres ressources du réseau vers le registre. 

Si un point de terminaison de service est configuré pour le registre, vérifiez qu’une règle de réseau est ajoutée au registre qui autorise l’accès à partir de ce sous-réseau. Le point de terminaison de service prend uniquement en charge l’accès à partir des machines virtuelles et de clusters AKS dans le réseau.

Si vous souhaitez restreindre l’accès au registre à l’aide d’un réseau virtuel dans un autre abonnement Azure, veillez à inscrire le fournisseur de ressources `Microsoft.ContainerRegistry` dans cet abonnement. [Inscrivez le fournisseur de ressources](../azure-resource-manager/management/resource-providers-and-types.md) pour Azure Container Registry à l’aide du portail Azure, de l’interface de ligne de commande Azure ou d’autres outils Azure.

Si le Pare-feu Azure ou une solution similaire sont configurés dans le réseau, vérifiez que le trafic sortant à partir d’autres ressources telles qu’un cluster AKS est activé pour atteindre les points de terminaison du registre.

Si un point de terminaison privé est configuré, vérifiez que DNS résout le nom de domaine complet public du registre, par exemple *myregistry.azurecr.io* à l’adresse IP privée du registre. Utilisez un utilitaire réseau comme `dig` ou `nslookup` pour la recherche DNS.

Liens connexes :

* [Connexion privée à un registre de conteneurs Azure à l’aide d’Azure Private Link](container-registry-private-link.md)
* [Restreindre l’accès à un registre de conteneurs à l’aide d’un point de terminaison de service dans un réseau virtuel Azure](container-registry-vnet.md)
* [Règles de réseau sortantes et noms FQDN requis pour les clusters AKS](../aks/limit-egress-traffic.md#required-outbound-network-rules-and-fqdns-for-aks-clusters)
* [Kubernetes: Débogage de résolution DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
* [Balises de service du réseau virtuel](../virtual-network/service-tags-overview.md)

### <a name="configure-service-access"></a>Configurer l’accès au service

Actuellement, Azure Security Center ne peut pas effectuer l’[analyse de vulnérabilité des images](../security-center/defender-for-container-registries-introduction.md?bc=%2fazure%2fcontainer-registry%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fcontainer-registry%2ftoc.json) dans un registre qui restreint l’accès aux points de terminaison privés, aux sous-réseaux sélectionnés ou aux adresses IP. En outre, les ressources des services suivants ne peuvent pas accéder à un registre de conteneurs avec des restrictions réseau :

* Azure DevOps Services 
* Azure Container Instances
* Tâches Azure Container Registry

Si l’accès ou l’intégration de ces services Azure à votre registre de conteneurs est nécessaire, supprimez la restriction réseau. Par exemple, supprimez les points de terminaison privés du registre, ou supprimez ou modifiez les règles d’accès publiques du registre.

Liens connexes :

* [Analyse d’images Azure Container Registry par Security Center](../security-center/defender-for-container-registries-introduction.md)
* Fournir des [commentaires](https://feedback.azure.com/forums/347535-azure-security-center/suggestions/41091577-enable-vulnerability-scanning-for-images-that-are)
* [Configurer des règles de réseau IP public](container-registry-access-selected-networks.md)
* [Connexion privée à un registre de conteneurs Azure à l’aide d’Azure Private Link](container-registry-private-link.md)


## <a name="advanced-troubleshooting"></a>Dépannage avancé

Si la [collecte des journaux de ressources](container-registry-diagnostics-audit-logs.md) est activée dans le registre, consultez le journal ContainterRegistryLoginEvents. Ce journal stocke les événements d’authentification et l’état, y compris l’identité et l’adresse IP entrantes. Interrogez le journal sur les [échecs d’authentification du registre](container-registry-diagnostics-audit-logs.md#registry-authentication-failures). 

Liens connexes :

* [Journaux pour l’évaluation et l’audit de diagnostics](container-registry-diagnostics-audit-logs.md)
* [FAQ sur le registre de conteneurs](container-registry-faq.md)
* [Ligne de base de sécurité Azure pour Azure Container Registry](security-baseline.md)
* [Meilleures pratiques pour Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Étapes suivantes

Si vous ne résolvez pas votre problème ici, consultez les options suivantes.

* Les autres rubriques de dépannage du registre sont les suivantes :
  * [Résoudre les problèmes de connexion au registre](container-registry-troubleshoot-login.md) 
  * [Résoudre les problèmes de performances de registre](container-registry-troubleshoot-performance.md)
* Options de [support de la communauté](https://azure.microsoft.com/support/community/)
* [Microsoft Q&A](/answers/products/)
* [Ouvrir un ticket de support](https://azure.microsoft.com/support/create-ticket/)