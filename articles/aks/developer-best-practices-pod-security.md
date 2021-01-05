---
title: Meilleures pratiques du développeur - Sécurité des pods dans Azure Kubernetes Service (AKS)
description: Découvrez les meilleures pratiques du développeur pour apprendre à sécuriser des pods dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 07/28/2020
ms.author: zarhoads
ms.openlocfilehash: 1c7143b6d3479cf3083cfc730301c68dcf4eb705
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900822"
---
# <a name="best-practices-for-pod-security-in-azure-kubernetes-service-aks"></a>Meilleures pratiques pour la sécurité des pods dans Azure Kubernetes Service (AKS)

Lorsque vous développez et exécutez des applications dans Azure Kubernetes Service (ACS), la sécurité de vos pods est primordiale. Vos applications doivent être conçues selon le principe du moindre privilège. La sécurité des données privées est la priorité des clients. Vous ne tenez pas à ce que les informations d’identification telles que les chaînes de connexion aux bases de données, les clés, les secrets ou les certificats, soient exposées au monde extérieur, avec le risque qu’un pirate puisse les exploiter à des fins malveillantes. Ne les ajoutez pas à votre code et évitez de les incorporer dans vos images de conteneur. Vos informations risqueraient d’être exposées et cela limiterait votre capacité à changer ces informations d’identification, puisque les images de conteneur devront être reconstruites.

Cet article décrit les bonnes pratiques de sécurisation des pods dans AKS. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Utiliser le contexte de sécurité des pods pour limiter l’accès aux processus et services ou l’élévation de privilèges
> * S’authentifier avec d’autres ressources Azure à l’aide d’identités de pod managées
> * Demander et récupérer des informations d’identification à partir d’un coffre numérique tel qu’Azure Key Vault

Vous pouvez également consulter les bonnes pratiques pour la [sécurité des clusters][best-practices-cluster-security] et la [gestion des images conteneur][best-practices-container-image-management].

## <a name="secure-pod-access-to-resources"></a>Sécuriser l’accès du pod aux ressources

**Meilleures pratiques** : pour changer d’utilisateur ou de groupe et limiter l’accès aux services et processus de nœud sous-jacents, définissez les paramètres du contexte de sécurité des pods. Affectez le nombre minimal de privilèges requis.

Pour que vos applications s’exécutent correctement, les pods doivent s’exécuter en tant qu’utilisateur ou groupe défini, et non en tant que *racine* . Le `securityContext` pour un pod ou un conteneur vous permet de définir des paramètres tels que *runAsUser* ou *fsGroup* pour assumer les autorisations appropriées. Affectez uniquement les autorisations requises pour l’utilisateur ou le groupe et n’utilisez pas le contexte de sécurité pour assumer des autorisations supplémentaires. Le *runAsUser* , élévation des privilèges, et d’autres paramètres de fonctionnalités Linux sont uniquement disponibles sur les pods et nœuds Linux.

Lorsque vous vous connectez en tant qu’utilisateur non root, les conteneurs ne peuvent pas établir de liaison avec les ports privilégiés inférieurs à 1024. Dans ce scénario, Kubernetes Services peut être utilisé pour masquer le fait qu’une application s’exécute sur un port particulier.

Un contexte de sécurité de pod peut également permettre de définir des fonctionnalités ou autorisations supplémentaires pour accéder à des processus et services. Vous pouvez utiliser les définitions de contexte de sécurité courantes ci-dessous :

* **allowPrivilegeEscalation** définit si le pod peut assumer les privilèges *racine* . Concevez vos applications afin que ce paramètre soit toujours défini sur *false* .
* **Les fonctionnalités Linux** permettent au pod d’accéder aux processus de nœud sous-jacents. Soyez vigilant lorsque vous affectez ces fonctionnalités. Affectez le nombre minimal de privilèges nécessaire. Pour plus d’informations, consultez [Fonctionnalités de Linux][linux-capabilities].
* **SELinux Labels** est un module de sécurité du noyau Linux qui vous permet de définir des stratégies d’accès pour l’accès aux services, aux processus et au système de fichiers. Là encore, affectez le nombre minimal de privilèges nécessaire. Pour plus d’informations, consultez [Options de SELinux dans Kubernetes][selinux-labels]

L’exemple de manifeste YAML suivant définit les paramètres du contexte de sécurité pour indiquer :

* qu’un pod est exécuté en tant qu’utilisateur portant l’ID *1000* et faisant partie du groupe avec l’ID *2000* ;
* qu’il n’est pas possible d’élever les privilèges pour utiliser `root` ;
* que les fonctionnalités Linux peuvent accéder aux interfaces réseau et à l’horloge (matérielle) en temps réel de l’hôte.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    fsGroup: 2000
  containers:
    - name: security-context-demo
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      securityContext:
        runAsUser: 1000
        allowPrivilegeEscalation: false
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
```

Consultez votre opérateur de cluster pour déterminer les paramètres de contexte de sécurité dont vous avez besoin. Essayez de concevoir vos applications de manière à réduire les autorisations et accès supplémentaires dont le pod a besoin. Il existe d’autres fonctionnalités de sécurité qui permettent de limiter l’accès en utilisant les paramètres AppArmor et seccomp (environnement informatique sécurisé) qui peuvent être implémentés par les opérateurs du cluster. Pour plus d’informations, consultez [Sécuriser l’accès du conteneur aux ressources][apparmor-seccomp].

## <a name="limit-credential-exposure"></a>Limiter l’exposition des informations d’identification

**Meilleures pratiques** -ne définissez pas d’informations d’identification dans le code de votre application. Utilisez des identités managées pour les ressources Azure pour permettre à votre pod de demander l’accès à d’autres ressources. Vous devez également utiliser un coffre-fort numérique, tel qu’Azure Key Vault, pour stocker et récupérer des clés numériques et informations d’identification. Les identités managées sont conçues pour être utilisées avec des images conteneurs et des pods Linux uniquement.

Pour limiter le risque d’exposition d’informations d’identification dans le code de votre application, évitez d’utiliser des informations d’identification fixes ou partagées. Vous ne devez inclure aucune information d’identification ou clé directement dans votre code. Si ces informations d’identification sont exposées, l’application doit être mise à jour et redéployée. Une meilleure approche consiste à attribuer aux pods leur propre identité ainsi qu’un moyen de s’authentifier d’eux-mêmes, ou de récupérer automatiquement les informations d’identification à partir d’un coffre numérique.

### <a name="use-azure-container-compute-upstream-projects"></a>Utiliser des projets Azure Container Compute Upstream

> [!IMPORTANT]
> Les projets open source AKS associés ne sont pas pris en charge par le support technique Azure. Ils sont fournis pour permettre aux utilisateurs une installation automatique dans les clusters, et une collecte de commentaires à partir de notre communauté.

Les [projets open source AKS associés][aks-associated-projects] suivants vous permettent d’authentifier automatiquement les pods, ou de demander des informations d’identification et des clés auprès d’un coffre numérique. Ces projets sont gérés par l’équipe Azure Container Compute Upstream et font partie d’une [plus large liste de projets disponibles](https://github.com/Azure/container-compute-upstream/blob/master/README.md#support).

 * [Identité de pod Azure Active Directory][aad-pod-identity]
 * [Fournisseur Azure Key Vault pour le pilote Secrets Store CSI](https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage)

#### <a name="use-pod-managed-identities"></a>Utiliser des identités de pod managées

Une identité managée pour les ressources Azure permet à un pod de s’authentifier lui-même auprès de services Azure qui le prennent en charge, comme le service Stockage ou SQL. Le pod se voit attribuer une identité Azure qui lui permet de s’authentifier auprès d’Azure Active Directory et de recevoir un jeton numérique. Ce jeton numérique peut être présenté à d’autres services Azure qui vérifient si le pod est autorisé à accéder au service et à effectuer les actions requises. Cette approche signifie par exemple qu’aucun secret n’est nécessaire pour les chaînes de connexion à la base de données. Le worflow simplifié du système d’identités de pod managées est représenté dans le schéma suivant :

:::image type="content" source="media/developer-best-practices-pod-security/basic-pod-identity.svg" alt-text="Workflow simplifié du système d’identités de pod managées dans Azure":::

Avec une identité managée, il n’est pas nécessaire d’inclure des informations d’identification dans le code de votre application pour accéder à un service, tel que Stockage Azure. Puisque chaque pod s’authentifie avec sa propre identité, vous pouvez contrôler et réviser les accès. Si votre application se connecte auprès d’autres services Azure, utilisez des identités managées pour limiter la réutilisation d’informations d’identification et le risque d’exposition.

Pour plus d’informations sur les identités de pod, consultez [Configurer un cluster AKS pour utiliser des identités de pod managées et avec vos applications][aad-pod-identity]

#### <a name="use-azure-key-vault-with-secrets-store-csi-driver"></a>Utiliser Azure Key Vault avec le pilote Secrets Store CSI

L’utilisation du projet d’identités de pod vous permet de vous authentifier auprès des services Azure qui le prennent en charge. Si vos propres services ou applications n’ont pas d’identités managées pour les ressources Azure, vous pouvez toujours utiliser des informations d’identification ou des clés pour vous authentifier. Vous pouvez utiliser un coffre numérique pour stocker le contenu de ces secrets.

Lorsque les applications ont besoin d’informations d’identification, elles communiquent avec le coffre numérique, récupèrent le dernier contenu des secrets, puis se connectent au service nécessaire. Azure Key Vault peut être utilisé comme coffre numérique. Le schéma suivant illustre de façon simplifiée le workflow utilisé pour récupérer des informations d’identification à partir d’Azure Key Vault à l’aide d’identités de pod managées :

:::image type="content" source="media/developer-best-practices-pod-security/basic-key-vault.svg" alt-text="Workflow simplifié du système d’identités de pod managées dans Azure":::

Key Vault vous permet de stocker et faire tourner régulièrement les secrets, tels que les informations d’identification, les clés de compte de stockage ou les certificats. Vous pouvez intégrer Azure Key Vault à un cluster AKS à l’aide du [fournisseur Azure Key Vault pour le pilote Secrets Store CSI](https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage). Le pilote Secrets Store CSI permet au cluster AKS de récupérer en mode natif le contenu des secrets à partir de Key Vault, et fournit celui-ci uniquement au pod demandeur. Collaborez avec votre opérateur de cluster pour déployer le pilote Secrets Store CSI sur les nœuds Worker AKS. Vous pouvez utiliser une identité managée par un pod pour demander l’accès à Key Vault et récupérer le contenu des secrets par le biais du pilote Secrets Store CSI.

Azure Key Vault et le pilote Secrets Store CSI peuvent être utilisés pour les nœuds et les pods Linux qui nécessitent une version de Kubernetes supérieure ou égale à 1.16. Pour les nœuds et les pods Windows, vous avez besoin de Kubernetes version 1.18 ou supérieure.

## <a name="next-steps"></a>Étapes suivantes

Cet article était dédié à la sécurisation de vos pods. Pour implémenter quelques-unes de ces pratiques, consultez les articles suivants :

* [Utiliser des identités managées pour les ressources Azure avec AKS][aad-pod-identity]
* [Intégrer Azure Key Vault à AKS][aks-keyvault-csi-driver]

<!-- EXTERNAL LINKS -->
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity#demo
[aks-keyvault-csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage
[linux-capabilities]: http://man7.org/linux/man-pages/man7/capabilities.7.html
[selinux-labels]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#selinuxoptions-v1-core
[aks-associated-projects]: https://awesomeopensource.com/projects/aks?categoryPage=11

<!-- INTERNAL LINKS -->
[best-practices-cluster-security]: operator-best-practices-cluster-security.md
[best-practices-container-image-management]: operator-best-practices-container-image-management.md
[aks-pod-identities]: operator-best-practices-identity.md#use-pod-identities
[apparmor-seccomp]: operator-best-practices-cluster-security.md#secure-container-access-to-resources
