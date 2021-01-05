---
title: Durcissement de la sécurité des hôtes de machines virtuelles AKS
description: Découvrir le durcissement de la sécurité du système d’exploitation hôte des machines virtuelles AKS
services: container-service
author: mlearned
ms.topic: article
ms.date: 09/11/2019
ms.author: mlearned
ms.custom: mvc
ms.openlocfilehash: 84b826ce33b5395db5bd38e883b3a0fb3425725b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86244036"
---
# <a name="security-hardening-for-aks-agent-node-host-os"></a>Durcissement de la sécurité du système d’exploitation hôte du nœud de l’agent AKS

Azure Kubernetes Service (AKS) est un service sécurisé conforme aux normes SOC, ISO, PCI DSS et HIPAA. Cet article traite du durcissement de la sécurité appliqué aux hôtes de machines virtuelles AKS. Pour plus d’informations sur la sécurité AKS, consultez [Concepts de sécurité pour les applications et les clusters dans AKS (Azure Kubernetes Service)](./concepts-security.md).

> [!Note]
> Ce document se limite aux agents Linux dans AKS uniquement.

Les clusters AKS sont déployés sur des machines virtuelles hôtes, qui exécutent un système d’exploitation doté d’une sécurité optimisée et utilisée pour les conteneurs en cours d’exécution sur AKS. Ce système d’exploitation hôte est basé sur une image **Ubuntu 16.04.LTS** ayant fait l’objet d’un durcissement de la sécurité et d’optimisations (voir les détails du durcissement de la sécurité).

L’objectif du système d’exploitation hôte avec un durcissement de la sécurité est de réduire la surface d’attaque et d’optimiser le déploiement de conteneurs de manière sécurisée.

> [!Important]
> Le système d’exploitation avec une sécurité durcie n’a PAS été testé selon les critères CIS. Même s’il y a des critères communs avec les bancs d’essai CIS, l’objectif n’est pas d’être conforme à CIS. L’objectif du durcissement du système d’exploitation hôte consiste à converger vers un niveau de sécurité cohérent avec les propres normes de sécurité d’hôte interne de Microsoft.

## <a name="security-hardening-features"></a>Caractéristiques du durcissement de la sécurité

* AKS fournit un système d’exploitation hôte à sécurité optimisée par défaut. Il n’existe aucune option pour sélectionner un autre système d’exploitation.

* Azure applique des correctifs quotidiens (y compris des correctifs de sécurité) aux hôtes de machines virtuelles AKS. Certains de ces correctifs nécessitent un redémarrage et d’autres non. Vous êtes responsable de la planification des redémarrages nécessaires des hôtes de machines virtuelles AKS. Pour obtenir des conseils sur l’automatisation des mises à jour correctives AKS, consultez l’[application de correctifs pour les nœuds AKS](./node-updates-kured.md).

## <a name="what-is-configured"></a>Ce qui est configuré

| CIS  | Description d’audit|
|---|---|
| 1.1.1.1 |Vérifier que le montage des systèmes de fichiers cramfs est désactivé|
| 1.1.1.2 |Vérifier que le montage des systèmes de fichiers freevxfs est désactivé|
| 1.1.1.3 |Vérifier que le montage des systèmes de fichiers jffs2 est désactivé|
| 1.1.1.4 |Vérifier que le montage des systèmes de fichiers HFS est désactivé|
| 1.1.1.5 |Vérifier que le montage des systèmes de fichiers HFS Plus est désactivé|
|1.4.3 |Garantir l’authentification requise pour le mode mono-utilisateur |
|1.7.1.2 |Vérifier que la bannière d’avertissement de connexion locale est correctement configurée |
|1.7.1.3 |Vérifier que la bannière d’avertissement de connexion à distance est correctement configurée |
|1.7.1.5 |Vérifier que des autorisations sont configurées sur /etc/issue |
|1.7.1.6 |Vérifier que des autorisations sont configurées sur /etc/issue.net |
|2.1.5 |Vérifier que --streaming-connection-idle-timeout n’a pas la valeur 0 |
|3.1.2 |Vérifier que l’envoi de redirection de paquets est désactivé |
|3.2.1 |Vérifier que les packages routés sources ne sont pas acceptés |
|3.2.2 |Vérifier que les redirections ICMP ne sont pas acceptées |
|3.2.3 |Vérifier que les redirections ICMP sécurisées ne sont pas acceptées |
|3.2.4 |Vérifier que les paquets suspects sont consignés dans un journal |
|3.3.1 |Vérifier que les annonces de routeur IPv6 ne sont pas acceptées |
|3.5.1 |Vérifier que le protocole DCCP est désactivé |
|3.5.2 |Vérifier que le protocole SCTP est désactivé |
|3.5.3 |Vérifier que RDS est désactivé |
|3.5.4 |Vérifier que TIPC est désactivé |
|4.2.1.2 |Vérifier que la journalisation est configurée |
|5.1.2 |Vérifier que des autorisations sont configurées sur /etc/crontab |
|5.2.4 |Vérifier que le transfert SSH X11 est désactivé |
|5.2.5 |Vérifier que SSH MaxAuthTries est défini sur une valeur inférieure ou égale à 4 |
|5.2.8 |Vérifier que la connexion racine SSH est désactivée |
|5.2.10 |Vérifier que SSH PermitUserEnvironment est désactivé |
|5.2.11 |Vérifier que seuls les algorithmes MAX approuvés sont utilisés |
|5.2.12 |Vérifier que l’intervalle du délai d’inactivité SSH est configuré |
|5.2.13 |Vérifier que SSH LoginGraceTime est défini sur une minute ou moins |
|5.2.15 |Vérifier que la bannière d’avertissement SSH est configurée |
|5.3.1 |Vérifier que les critères de création de mot de passe sont configurés |
|5.4.1.1 |Vérifier que l’expiration du mot de passe est de 90 jours ou moins |
|5.4.1.4 |Vérifier que le verrouillage de mot de passe inactif est de 30 jours ou moins |
|5.4.4 |Vérifier que le masque umask d’utilisateur par défaut est 027 ou plus restrictif |
|5.6 |Vérifier que l’accès à la commande su est restreint|

## <a name="additional-notes"></a>Remarques supplémentaires
 
* Pour réduire encore la surface d’attaque, certains pilotes de module de noyau inutiles ont été désactivés dans le système d’exploitation.

* Le système d’exploitation avec durcissement de la sécurité est conçu et géré spécifiquement pour AKS et n’est PAS pris en charge en dehors de la plateforme AKS.

## <a name="next-steps"></a>Étapes suivantes  

Pour plus d’informations sur la sécurité AKS, consultez les articles suivants : 

[Azure Kubernetes Service (AKS)](./intro-kubernetes.md)

[Concepts de sécurité AKS](./concepts-security.md)

[Bonnes pratiques AKS](./best-practices.md)
