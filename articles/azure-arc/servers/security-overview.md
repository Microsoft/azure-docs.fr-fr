---
title: Présentation de la sécurité
description: Informations relatives à la sécurité des serveurs Azure Arc.
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: be79be3030af76425b54fd683784d0e216ac2cf5
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91329038"
---
# <a name="azure-arc-for-servers-security-overview"></a>Vue d’ensemble de la sécurité Azure Arc pour serveurs

Cet article décrit la configuration et les considérations relatives à la sécurité que vous devez évaluer avant de déployer des serveurs Azure Arc dans votre entreprise.

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

Chaque serveur Azure Arc dispose d’une identité managée, en tant qu’élément d’un groupe de ressources dans un abonnement Azure ; cette identité représente le serveur s’exécutant localement ou sur un autre environnement cloud. L’accès à cette ressource est contrôlé par le [contrôle d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md) standard. À partir de la page [**Contrôle d’accès (IAM)** ](../../role-based-access-control/role-assignments-portal.md#access-control-iam) du portail Azure, vous pouvez vérifier qui a accès à votre serveur Azure Arc.

:::image type="content" source="./media/security-overview/access-control-page.png" alt-text="Contrôle d’accès au serveur Azure Arc" border="false" lightbox="./media/security-overview/access-control-page.png":::

Les utilisateurs et les applications bénéficiant de l’accès à la ressource par le biais du rôle d’administrateur ou de [contributeur](../../role-based-access-control/built-in-roles.md#contributor) peuvent apporter des modifications à la ressource, y compris le déploiement ou la suppression d’[extensions](manage-vm-extensions.md) sur la machine. Les extensions peuvent inclure des scripts arbitraires qui s’exécutent dans un contexte privilégié ; par conséquent, vous devez considérer tout contributeur sur la ressource Azure comme un administrateur indirect du serveur.

Le rôle **Intégration d’Azure Connected Machine** est disponible pour l’intégration à grande échelle, il peut uniquement lire ou créer des serveurs Arc dans Azure. Il ne peut pas servir à supprimer des serveurs déjà inscrits, ou à gérer des extensions. En tant que bonne pratique, nous vous recommandons d’attribuer ce rôle uniquement au principal du service Azure Active Directory (Azure AD) utilisé pour l’intégration des machines à grande échelle.

En tant que membres du rôle **Administrateur des ressources Azure Connected Machine**, les utilisateurs peuvent lire, modifier, réintégrer et supprimer une machine. Ce rôle est conçu pour prendre en charge la gestion des serveurs Arc, mais aucune autre ressource dans le groupe de ressources ou l’abonnement.

## <a name="agent-security-and-permissions"></a>Sécurité et autorisations de l’agent

Pour gérer l’agent Azure Connected Machine (azcmagent) sur Windows, votre compte d’utilisateur doit être membre du groupe Administrateurs local. Sur Linux, vous devez disposer d’autorisations d’accès racine.

L’agent Azure Connected Machine est composé de trois services, qui s’exécutent sur votre machine.

* Le service Hybrid Instance Metadata Service (himds) est responsable de toutes les fonctionnalités principales d’Arc. Entre autres, l’envoi de pulsations à Azure, l’exposition d’un service de métadonnées d’instance locale, pour que d’autres applications découvrent l’ID de ressource Azure de la machine et récupèrent des jetons Azure AD en vue de l’authentification auprès d’autres services Azure. Ce service s’exécute en tant que compte de service virtuel non privilégié sur Windows, et est exécuté avec l’utilisateur **himds** sur Linux.

* Le service de configuration d’invité (GCService) est responsable de l’évaluation d’Azure Policy sur la machine.

* Le service d’extension de configuration d’invité (ExtensionService) est responsable de l’installation, de la mise à jour et de la suppression d’extensions (agents, scripts ou autres logiciels) sur la machine.

Les services de configuration et d’extension d’invités s’exécutent en tant que Système local sur Windows, et en tant que racine sur Linux.

## <a name="using-a-managed-identity-with-arc-enabled-servers"></a>Utilisation d’une identité managée avec les serveurs Arc

Par défaut, l’identité affectée par le système Azure Active Directory qui est utilisée par ARC peut uniquement servir pour mettre à jour l’état du serveur Arc dans Azure. Par exemple, l’état des pulsations *la dernière fois*. Vous pouvez éventuellement affecter des rôles supplémentaires à l’identité si une application sur votre serveur utilise l’identité attribuée par le système pour accéder aux autres services Azure.

Alors que n’importe quelle application s’exécutant sur la machine peut accéder à Hybrid Instance Metadata Service, seules les applications autorisées peuvent demander un jeton Azure AD pour l’identité attribuée par le système. Lors de la première tentative d’accès à l’URI de jeton, le service génère un objet blob de chiffrement, créé de manière aléatoire dans un emplacement sur le système de fichiers, que seuls les appelants approuvés peuvent lire. L’appelant doit ensuite lire le fichier (en prouvant qu’il dispose de l’autorisation appropriée) et faire une nouvelle tentative de demande, avec le contenu du fichier dans l’en-tête d’autorisation, pour parvenir à récupérer un jeton Azure AD.

* Sur Windows, l’appelant doit être membre du groupe local **Administrateurs**, ou du groupe **Applications d’extension de l’agent hybride** pour lire l’objet blob.

* Sur Linux, l’appelant doit être membre du groupe **himds** pour lire l’objet blob.

## <a name="using-disk-encryption"></a>Utilisation du chiffrement de disque

L’agent Azure Connected Machine utilise l’authentification par clé publique pour communiquer avec le service Azure. Après avoir intégré un serveur à Azure Arc, une clé privée est enregistrée sur le disque et utilisée chaque fois que l’agent communique avec Azure. En cas de vol, la clé privée peut être utilisée sur un autre serveur pour communiquer avec le service et faire comme s’il s’agissait du serveur d’origine. Entre autres actions possibles, l’obtention de l’accès à l’identité attribuée par le système et à toutes les ressources auxquelles l’identité a accès. Le fichier de clé privée est protégé afin d’autoriser uniquement au compte **himds** l’accès pour le lire. Pour empêcher les attaques hors connexion, nous vous recommandons vivement d’utiliser le chiffrement de disque complet (par exemple, BitLocker, dm-crypt, etc.) sur le volume du système d’exploitation de votre serveur.

## <a name="next-steps"></a>Étapes suivantes

Avant d'évaluer ou d'activer des serveurs avec Azure Arc sur plusieurs machines hybrides, consultez [Vue d'ensemble d'Azure Connected Machine Agent](agent-overview.md) pour en savoir plus sur les exigences et les détails techniques relatifs à l'agent, ainsi que sur les méthodes de déploiement.
