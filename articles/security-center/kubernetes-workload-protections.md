---
title: Protections de charge de travail pour vos charges de travail Kubernetes
description: Découvrez comment utiliser l’ensemble de recommandations de sécurité d’Azure Security Center pour la protection des charges de travail Kubernetes
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 09/12/2020
ms.author: memildin
ms.openlocfilehash: 08bcb74fd50be0eeb7a73c0743db2c4f3a57be32
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030847"
---
# <a name="protect-your-kubernetes-workloads"></a>Protéger vos charges de travail Kubernetes

Cette page explique comment utiliser l’ensemble de recommandations de sécurité d’Azure Security Center pour la protection des charges de travail Kubernetes.

Pour en savoir plus sur ces fonctionnalités, consultez [Meilleures pratiques pour la protection de charge de travail à l’aide du contrôle d’admission Kubernetes](container-security.md#workload-protection-best-practices-using-kubernetes-admission-control)

Security Center offre davantage de fonctionnalités de sécurité de conteneur si vous activez Azure Defender. Plus précisément :

- Analysez vos registres de conteneurs pour identifier les vulnérabilités avec [Azure Defender pour les registres de conteneurs](defender-for-container-registries-introduction.md)
- Recevez des alertes de détection des menaces en temps réel pour vos clusters K8s avec [Azure Defender pour Kubernetes](defender-for-kubernetes-introduction.md)

> [!TIP]
> Pour obtenir la liste de *toutes* les recommandations de sécurité qui peuvent s’afficher pour les clusters et nœuds Kubernetes, consultez la [section Conteneur](recommendations-reference.md#recs-containers) de la table de référence de recommandations.



## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|PRÉVERSION<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Prix :|Gratuit|
|Rôles et autorisations obligatoires :|**Propriétaire** ou **Administrateur de la sécurité** pour modifier une attribution<br>**Lecteur** pour afficher les recommandations|
|Clusters pris en charge :|Kubernetes v1.14 (ou version ultérieure) est requis<br>Aucune ressource PodSecurityPolicy (ancien modèle PSP) sur les clusters<br>Les nœuds Windows ne sont pas pris en charge|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||


## <a name="set-up-your-workload-protection"></a>Configurer la protection de votre charge de travail

Azure Security Center comprend un ensemble de recommandations qui sont disponibles lorsque vous installez le **module complémentaire Azure Policy pour Kubernetes**.

### <a name="step-1-deploy-the-add-on"></a>Étape 1 : Déployer le module complémentaire

Pour configurer les recommandations, installez le **module complémentaire Azure Policy pour Kubernetes**. 

- Vous pouvez déployer automatiquement ce module complémentaire comme cela est expliqué dans [Activer le provisionnement automatique des extensions](security-center-enable-data-collection.md#enable-auto-provisioning-of-extensions). Lorsque le provisionnement automatique du module complémentaire est défini sur « activé », l’extension est activée par défaut dans tous les clusters existants et futurs (qui répondent à la configuration requise de l’installation du module complémentaire).

- Pour déployer manuellement le module complémentaire

    1. Dans la page de recommandations, recherchez la recommandation « **Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters** ». 

        :::image type="content" source="./media/defender-for-kubernetes-usage/recommendation-to-install-policy-add-on-for-kubernetes.png" alt-text="Recommandation **Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters**":::

        > [!TIP]
        > La recommandation est incluse dans cinq contrôles de sécurité différents, et celui que vous sélectionnez à l’étape suivante importe peu.

    1. À partir de l’un des contrôles de sécurité, sélectionnez la recommandation pour afficher les ressources sur lesquelles vous pouvez installer le module complémentaire.
    1. Sélectionnez le cluster approprié, puis **Corriger**.

        :::image type="content" source="./media/defender-for-kubernetes-usage/recommendation-to-install-policy-add-on-for-kubernetes-details.png" alt-text="Page de détails de recommandation pour **Le module complémentaire Azure Policy pour Kubernetes doit être installé et activé sur vos clusters**":::

### <a name="step-2-view-and-configure-the-bundle-of-13-recommendations"></a>Étape 2 : Afficher et configurer le bundle de 13 recommandations

1. Environ 30 minutes après l’installation du module complémentaire, Security Center affiche l’état d’intégrité des clusters pour les recommandations suivantes, chacune dans le contrôle de sécurité approprié, comme indiqué ci-dessous :

    > [!TIP]
    > Certaines recommandations ont des paramètres qui doivent être personnalisés via Azure Policy pour les utiliser efficacement. Par exemple, pour tirer parti de la recommandation **Les images conteneur doivent être déployées à partir de registres approuvés uniquement**, vous devez définir vos registres de confiance.
    > 
    > Si vous n’entrez pas les paramètres nécessaires pour les recommandations qui requièrent une configuration, vos charges de travail seront affichées comme défectueuses.

    | Nom de la recommandation                                                         | Contrôle de sécurité                         | Configuration requise |
    |-----------------------------------------------------------------------------|------------------------------------------|------------------------|
    | Les limites de mémoire et du processeur du conteneur doivent être appliquées                          | Protéger les applications contre les attaques DDoS | Non                     |
    | Les conteneurs privilégiés doivent être évités                                     | Gérer l’accès et les autorisations            | Non                     |
    | Le système de fichiers racine immuable (en lecture seule) doit être appliqué pour les conteneurs     | Gérer l’accès et les autorisations            | Non                     |
    | Tout conteneur avec une élévation des privilèges doit être évité                       | Gérer l’accès et les autorisations            | Non                     |
    | L’exécution des conteneurs en tant qu’utilisateur racine doit être évitée                           | Gérer l’accès et les autorisations            | Non                     |
    | Éviter les conteneurs partageant des espaces de noms d’hôte sensibles              | Gérer l’accès et les autorisations            | Non                     |
    | Les fonctionnalités Linux les moins privilégiées doivent être appliquées pour les conteneurs       | Gérer l’accès et les autorisations            | **Oui**                |
    | L’utilisation des montages de volume HostPath de pad doit être limitée à une liste connue    | Gérer l’accès et les autorisations            | **Oui**                |
    | Les conteneurs doivent écouter uniquement sur les ports autorisés                              | Restreindre l’accès réseau non autorisé     | **Oui**                |
    | Les services doivent écouter uniquement sur les ports autorisés                                | Restreindre l’accès réseau non autorisé     | **Oui**                |
    | L’utilisation du réseau hôte et des ports doit être limitée                     | Restreindre l’accès réseau non autorisé     | **Oui**                |
    | Le remplacement ou la désactivation du profil AppArmor des conteneurs doit être limité | Corriger les configurations de sécurité        | **Oui**                |
    | Les images conteneur doivent être déployées à partir de registres approuvés uniquement            | Corriger les vulnérabilités                | **Oui**                |
    |||


1. Pour forcer les recommandations avec des paramètres à être personnalisées, définissez les paramètres :

    1. Dans le menu de Security Center, sélectionnez **Stratégie de sécurité**.
    1. Sélectionnez l’abonnement approprié.
    1. Dans la section **Stratégie par défaut de Security Center**, sélectionnez **Afficher la stratégie actuelle**.
    1. Sélectionnez « ASC Default ».
    1. Ouvrez l’onglet **Paramètres** et modifiez les valeurs selon vos besoins.
    1. Sélectionnez **Vérifier + enregistrer**.
    1. Sélectionnez **Enregistrer**.


1. Pour appliquer les recommandations, 

    1. Ouvrez la page Détails de la recommandation et sélectionnez **Refuser** :

        :::image type="content" source="./media/defender-for-kubernetes-usage/enforce-workload-protection-example.png" alt-text="Option Refuser pour le paramètre Azure Policy":::

        Cette opération ouvre le volet dans lequel vous définissez l’étendue. 

    1. Une fois que vous avez défini l’étendue, sélectionnez **Remplacer par Refuser**.

1. Pour voir quelles recommandations s’appliquent à vos clusters :

    1. Ouvrez la page d'[inventaire des ressources](asset-inventory.md) de Security Center et utilisez le filtre de type de ressource pour **Services Kubernetes**.

    1. Sélectionnez un cluster à examiner et passez en revue les recommandations disponibles pour celui-ci. 

1. Lorsque vous affichez une recommandation de l’ensemble de protection de la charge de travail, vous voyez le nombre de pods affectés (« composants Kubernetes ») avec le cluster. Pour obtenir la liste des pods spécifiques, sélectionnez le cluster, puis **Effectuer une action**.

    :::image type="content" source="./media/defender-for-kubernetes-usage/view-affected-pods-for-recommendation.gif" alt-text="Affichage des pods affectés pour une recommandation K8s"::: 

1. Pour tester l’application, utilisez les deux déploiements Kubernetes ci-dessous :

    - L’une est pour un déploiement sain, conforme au lot de recommandations de protection de la charge de travail.
    - L’autre est pour un déploiement défectueux, non conforme avec *n’importe laquelle* des recommandations.

    Déployez les fichiers d’exemple .yaml tels quels ou utilisez-les comme référence pour corriger votre propre charge de travail (étape VIII)  


## <a name="healthy-deployment-example-yaml-file"></a>Exemple de fichier .yaml de déploiement sain

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-healthy-deployment
  labels:
    app: redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
      annotations:
        apparmor.security.beta.kubernetes.io/pod: runtime/default
        container.apparmor.security.beta.kubernetes.io/redis: runtime/default
    spec:
      containers:
      - name: redis
        image: healthyClusterRegistry.azurecr.io/redis:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 250Mi
        securityContext:
          privileged: false
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000
---
apiVersion: v1
kind: Service
metadata:
  name: redis-healthy-service
spec:
  type: LoadBalancer
  selector:
    app: redis
  ports:
  - port: 80
    targetPort: 80
```

## <a name="unhealthy-deployment-example-yaml-file"></a>Exemple de fichier .yaml de déploiement défectueux

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-unhealthy-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:      
      labels:
        app: nginx
    spec:
      hostNetwork: true
      hostPID: true 
      hostIPC: true
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - containerPort: 9001
          hostPort: 9001
        securityContext:
          privileged: true
          readOnlyRootFilesystem: false
          allowPrivilegeEscalation: true
          runAsUser: 0
          capabilities:
            add:
              - NET_ADMIN
        volumeMounts:
        - mountPath: /test-pd
          name: test-volume
          readOnly: true
      volumes:
      - name: test-volume
        hostPath:
          # directory location on host
          path: /tmp
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-unhealthy-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 6001
    targetPort: 9001
```



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à configurer la protection d’une charge de travail Kubernetes. 

Pour d’autres informations connexes, consultez les pages suivantes : 

- [Recommandations de Security Center pour les conteneurs](recommendations-reference.md#recs-containers)
- [Alertes pour le niveau du cluster AKS](alerts-reference.md#alerts-akscluster)
- [Alertes pour niveau de l’hôte du conteneur](alerts-reference.md#alerts-containerhost)