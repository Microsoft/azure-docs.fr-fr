---
title: Déployer une passerelle auto-hébergée sur Kubernetes | Microsoft Docs
description: Découvrez comment déployer un composant de passerelle auto-hébergée de la Gestion des API Azure sur Kubernetes.
services: api-management
author: vladvino
manager: gwallace
ms.service: api-management
ms.workload: mobile
ms.topic: article
ms.author: apimpm
ms.date: 04/23/2020
ms.openlocfilehash: 023c2c89b90d6ddc71abc95db325dcdeb7684a2d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89500128"
---
# <a name="deploy-a-self-hosted-gateway-to-kubernetes"></a>Déployer une passerelle auto-hébergée sur Kubernetes

Cet article explique comment déployer la passerelle auto-hébergée de la Gestion des API Azure sur un cluster Kubernetes.

## <a name="prerequisites"></a>Prérequis

- Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
- Créez un cluster Kubernetes.
   > [!TIP]
   > Les [clusters à nœud unique](https://kubernetes.io/docs/setup/#learning-environment) conviennent bien au développement et à l’évaluation. Utilisez des clusters multinœuds [certifiés par Kubernetes](https://kubernetes.io/partners/#conformance) pour des charges de travail de production locales ou dans le cloud.
- [Provisionnez une ressource de passerelle auto-hébergée dans votre instance de la Gestion des API](api-management-howto-provision-self-hosted-gateway.md).

## <a name="deploy-to-kubernetes"></a>Déployer sur Kubernetes

1. Sélectionnez **Gateways** (Passerelles) sous **Deployment and infrastructure** (Déploiement et infrastructure).
2. Sélectionnez la ressource de passerelle auto-hébergée que vous voulez déployer.
3. Sélectionnez **Déploiement**.
4. Un jeton d’accès dans la zone de texte **Token** (Jeton) a été généré automatiquement pour vous, sur la base des valeurs par défaut pour l’expiration (**Expiry**) et la clé secrète (**Secret key**). Si nécessaire, choisissez les valeurs souhaitées dans l’un des contrôles ou dans les deux pour générer un jeton.
5. Sélectionnez l’onglet **Kubernetes** sous **Deployment scripts** (Scripts de déploiement).
6. Sélectionnez le lien du fichier **\<gateway-name\>.yml**, puis téléchargez le fichier YAML.
7. Sélectionnez l’icône **Copier** en bas à droite de la zone de texte **Deploy** (Déployer) pour enregistrer les commandes `kubectl` dans le Presse-papiers.
8. Collez les commandes dans la fenêtre du terminal. La première commande crée un secret Kubernetes contenant le jeton d’accès généré à l’étape 4. La deuxième commande applique le fichier de configuration téléchargé à l’étape 6 au cluster Kubernetes, et s’attend à ce que le fichier figure dans le répertoire actif.
9. Exécutez les commandes pour créer les objets Kubernetes nécessaires dans l’[espace de noms par défaut](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) et démarrez le ou les pods de la passerelle auto-hébergée à partir d’une [image conteneur](https://aka.ms/apim/sputnik/dhub) téléchargée à partir de Microsoft Container Registry.
10. Exécutez la commande suivante pour vérifier si le déploiement a réussi. Notez que le temps nécessaire à la création de tous les objets et à l’initialisation des pods peut être long.
    ```console
    kubectl get deployments
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    <gateway-name>   1/1     1            1           18s
    ```
11. Exécutez la commande suivante pour vérifier si le service a bien été créé. Notez que les adresses IP et les ports de votre service seront différents.
    ```console
    kubectl get services
    NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
    <gateway-name>   LoadBalancer   10.99.236.168   <pending>     80:31620/TCP,443:30456/TCP   9m1s
    ```
12. Revenez au portail Azure, puis sélectionnez **Vue d’ensemble**.
13. Vérifiez qu’**État** affiche une coche verte, suivie d’un nombre de nœuds qui correspond au nombre de réplicas spécifiés dans le fichier YAML. Cet état signifie que les pods de passerelle auto-hébergés déployés communiquent avec le service Gestion des API et présentent une « pulsation » normale.

    ![État de la passerelle](media/how-to-deploy-self-hosted-gateway-kubernetes/status.png)

> [!TIP]
> Exécutez la commande <code>kubectl logs deployment/<gateway-name></code> pour afficher les journaux d’un pod sélectionné de manière aléatoire, s’il en existe plusieurs.
> Exécutez <code>kubectl logs -h</code> pour obtenir un ensemble complet d’options de commande, par exemple, comment afficher les journaux d’un pod ou d’un conteneur spécifique.

## <a name="production-deployment-considerations"></a>Points à prendre en considération pour le déploiement de production

### <a name="access-token"></a>Access token (Jeton d’accès)
Sans un jeton d’accès valide, une passerelle auto-hébergée ne peut pas accéder aux données de configuration ni les télécharger à partir du point de terminaison du service Gestion des API associé. Le jeton d’accès reste valide pendant un maximum de 30 jours. Le jeton doit être regénéré, et le cluster doit être configuré avec un nouveau jeton, manuellement ou via l’automatisation, avant son expiration.

Lorsque vous automatisez l’actualisation des jetons, utilisez cette [opération d’API de gestion](/rest/api/apimanagement/2019-12-01/gateway/generatetoken) pour générer un nouveau jeton. Pour plus d’informations sur la gestion des secrets Kubernetes, consultez le [site Web Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret).

### <a name="namespace"></a>Espace de noms
Les [espaces de noms](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) Kubernetes permettent de diviser un cluster entre plusieurs équipes, projets ou applications. Les espaces de noms fournissent une étendue pour les ressources et les noms. Ils peuvent être associés à un quota de ressources et à des stratégies de contrôle des accès.

Le Portail Azure fournit des commandes pour créer des ressources de passerelle auto-hébergées dans l’espace de noms **par défaut**. Cet espace de noms est automatiquement créé, il existe dans chaque cluster et il ne peut pas être supprimé.
Dans un environnement de production, il est conseillé de [créer et déployer](https://kubernetesbyexample.com/ns/) une passerelle auto-hébergée dans un espace de noms distinct.

### <a name="number-of-replicas"></a>Nombre de réplicas
Le nombre minimal de réplicas adaptés à la production est de deux.

Par défaut, une passerelle auto-hébergée est déployée avec une [stratégie](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) de déploiement de type **RollingUpdate**. Passez en revue les valeurs par défaut. Il est conseillé de définir explicitement les champs [maxUnavailable](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) et [maxSurge](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge), en particulier si vous utilisez un nombre élevé de réplicas.

### <a name="container-resources"></a>Ressources sur les conteneurs
Par défaut, le fichier YAML fourni dans le portail Azure ne spécifie pas les demandes de ressources de conteneur.

Il est impossible de prédire et donc de recommander de manière fiable la quantité de ressources processeur et mémoire pour chaque conteneur, ainsi que le nombre de réplicas nécessaires pour prendre en charge une charge de travail spécifique. Ceci est dû à de nombreux facteurs, comme :

- Le matériel spécifique sur lequel le cluster s’exécute.
- La présence et le type de virtualisation.
- Le nombre et le taux de connexions clientes simultanées.
- Le taux de demandes.
- Le type et le nombre de stratégies configurées.
- La taille de la charge utile, et si les charges utiles sont mises en mémoire tampon ou diffusées en continu.
- La latence du service back-end.

Nous vous recommandons de définir les demandes de ressources sur deux cœurs et 2 Gio comme point de départ. Effectuez un test de charge et augmentez/diminuez la taille des instances en fonction des résultats.

### <a name="container-image-tag"></a>Étiquette de l’image conteneur
Le fichier YAML fourni dans le Portail Azure utilise la balise **la plus récente**. Cette balise référence toujours la version la plus récente de l’image de conteneur de passerelle auto-hébergée.

Il est conseillé d’utiliser une étiquette de version distincte en production pour éviter toute mise à niveau involontaire vers une version plus récente.

Vous pouvez [télécharger la liste complète des balises disponibles](https://mcr.microsoft.com/v2/azure-api-management/gateway/tags/list).

### <a name="dns-policy"></a>Stratégie DNS
La résolution de noms DNS joue un rôle essentiel dans la capacité d’une passerelle auto-hébergée à se connecter aux dépendances dans Azure, et à distribuer des appels d’API aux services back-end.

Le fichier YAML fourni dans le Portail Azure applique la stratégie [ClusterFirst](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy) par défaut. Cette stratégie entraîne le transfert des requêtes de résolution de noms non résolues par le DNS du cluster au serveur DNS en amont hérité du nœud.

Pour en savoir plus sur la résolution de noms dans Kubernetes, consultez le [site Web Kubernetes](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service). Envisagez de personnaliser la [stratégie DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy) ou la [configuration DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-config) en fonction de votre configuration.

### <a name="external-traffic-policy"></a>Stratégie de trafic externe
Le fichier YAML fourni dans le portail Azure définit le champ `externalTrafficPolicy` de l’objet [Service](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#service-v1-core) sur `Local`. Cela préserve l’adresse IP de l’appelant (accessible dans le [contexte de la demande](api-management-policy-expressions.md#ContextVariables)) et désactive l’équilibrage de charge entre les nœuds, éliminant ainsi les tronçons réseau qui en découlent. Sachez que ce paramètre peut entraîner une distribution asymétrique du trafic dans les déploiements avec un nombre inégal de pods de passerelle par nœud.

### <a name="custom-domain-names-and-ssl-certificates"></a>Noms de domaine personnalisés et certificats SSL

Si vous utilisez des noms de domaine personnalisés pour les points de terminaison de gestion des API, en particulier si vous utilisez un nom de domaine personnalisé pour le point de terminaison de gestion, vous devrez peut-être mettre à jour la valeur de `config.service.endpoint` dans le fichier **\<gateway-name\>.yaml** pour remplacer le nom de domaine par défaut par le nom de domaine personnalisé. Assurez-vous que le point de terminaison de gestion est accessible à partir du pod de la passerelle auto-hébergée dans le cluster Kubernetes.

Dans ce scénario, si le certificat SSL utilisé par le point de terminaison de gestion n’est pas signé par un certificat d’autorité de certification connu, vous devez vous assurer que le certificat de l’autorité de certification est approuvé par le pod de la passerelle auto-hébergée.

### <a name="configuration-backup"></a>Sauvegarde de la configuration
Pour en savoir plus sur le comportement de la passerelle auto-hébergée en cas de panne de connectivité temporaire à Azure, consultez [Vue d’ensemble de la passerelle auto-hébergée](self-hosted-gateway-overview.md#connectivity-to-azure).

Configurez un volume de stockage local pour le conteneur de la passerelle auto-hébergée, afin qu’il puisse conserver une copie de sauvegarde de la dernière configuration téléchargée. Si la connectivité est interrompue, le volume de stockage peut utiliser la copie de sauvegarde lors du redémarrage. Le chemin du montage de volume doit être le suivant : <code>/apim/config</code>. Consultez un exemple sur [GitHub](https://github.com/Azure/api-management-self-hosted-gateway/blob/master/examples/self-hosted-gateway-with-configuration-backup.yaml).
Pour en savoir plus sur le stockage dans Kubernetes, consultez le [site Web Kubernetes](https://kubernetes.io/docs/concepts/storage/volumes/).

### <a name="local-logs-and-metrics"></a>Métriques et journaux locaux
La passerelle auto-hébergée envoie des données de télémétrie à [Azure Monitor](api-management-howto-use-azure-monitor.md) et à [Azure Application Insights](api-management-howto-app-insights.md) selon les paramètres de configuration du service Gestion des API associé.
Lorsque la [connectivité à Azure](self-hosted-gateway-overview.md#connectivity-to-azure) est temporairement perdue, le flux des données de télémétrie envoyées vers Azure est interrompu et les données sont perdues pendant toute la durée de la panne.
Il est conseillé de [configurer une supervision locale](how-to-configure-local-metrics-logs.md) pour garantir la capacité à observer le trafic d’API et pour empêcher la perte des données de télémétrie pendant les interruptions de connectivité Azure.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la passerelle auto-hébergée, consultez [Vue d’ensemble de la passerelle auto-hébergée](self-hosted-gateway-overview.md).
