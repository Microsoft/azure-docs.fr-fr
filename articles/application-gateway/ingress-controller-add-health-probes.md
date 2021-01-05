---
title: Ajouter des sondes d’intégrité à vos pods AKS
description: Cet article fournit des informations sur l’ajout de sondes d’intégrité (préparation et/ou disponibilité) aux pods AKS avec Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 8c8b8b0090877db7abc8fae0e44f928e8b10dcf5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84808005"
---
# <a name="add-health-probes-to-your-service"></a>Ajouter des sondes d’intégrité à votre service
Par défaut, le contrôleur d’entrée approvisionne une sonde HTTP GET pour les pods exposés.
Les propriétés de la sonde peuvent être personnalisées en ajoutant une [sonde de préparation ou d’activité](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) à vos spécifications `deployment`/`pod`.

## <a name="with-readinessprobe-or-livenessprobe"></a>Avec `readinessProbe` ou `livenessProbe`
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aspnetapp
spec:
  replicas: 3
  template:
    metadata:
      labels:
        service: site
    spec:
      containers:
      - name: aspnetapp
        image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 3
          timeoutSeconds: 1
```

Référence de l’API Kubernetes :
* [Sondes de conteneur](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
* [Action HttpGet](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#httpgetaction-v1-core)

> [!NOTE]
> * `readinessProbe` et `livenessProbe` sont pris en charge lors d’une configuration avec `httpGet`.
> * La détection sur un port autre que celui exposé sur le pod n’est pas prise en charge pour le moment.
> * `HttpHeaders`, `InitialDelaySeconds`, et `SuccessThreshold` ne sont pas pris en charge.

##  <a name="without-readinessprobe-or-livenessprobe"></a>Sans `readinessProbe` ou `livenessProbe`
Si les sondes ci-dessus ne sont pas fournies, le contrôleur d’entrée fait une hypothèse que le service est accessible sur `Path` spécifié pour l’annotation `backend-path-prefix` ou `path` spécifié dans la définition `ingress` pour le service.

## <a name="default-values-for-health-probe"></a>Valeurs par défaut pour la sonde d’intégrité
Pour toute propriété qui ne peut pas être déduite par la sonde de disponibilité/d’activité, les valeurs par défaut sont définies.

| Propriété de sonde Application Gateway | Valeur par défaut |
|-|-|
| `Path` | / |
| `Host` | localhost |
| `Protocol` | HTTP |
| `Timeout` | 30 |
| `Interval` | 30 |
| `UnhealthyThreshold` | 3 |