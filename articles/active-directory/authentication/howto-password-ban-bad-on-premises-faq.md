---
title: FAQ sur la protection par mot de passe Azure AD en local
description: Consulter les questions fréquentes relatives à la protection par mot de passe Azure AD dans un environnement Active Directory Domain Services local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d5517afe7407da7428d4a83f3d2de67836280c7
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741896"
---
# <a name="azure-ad-password-protection-on-premises-frequently-asked-questions"></a>Questions fréquentes (FAQ) sur la protection par mot de passe Azure AD en local

Cette section fournit des réponses à de nombreuses questions fréquemment posées sur la protection des mots de passe Azure AD.

## <a name="general-questions"></a>Questions générales

**Q : Quels conseils donner aux utilisateurs sur la façon de sélectionner un mot de passe sécurisé ?**

Les conseils de Microsoft sur ce sujet se trouvent sur le lien suivant :

[Conseils pour le choix du mot de passe Microsoft](https://www.microsoft.com/research/publication/password-guidance)

**Q : La protection par mot de passe Azure AD en local est-elle prise en charge sur les clouds non publics ?**

La protection par mot de passe Azure AD en local est prise en charge sur le cloud public et le cloud Arlington. Aucune date n’a été annoncée concernant la disponibilité sur d’autres clouds.

Le portail Azure AD autorise la modification de la configuration spécifique en local « Protection par mot de passe pour Windows Server Active Directory », même dans les clouds non pris en charge. Ces modifications sont rendues persistantes, sans quoi elles ne prennent pas effet. L’inscription d’agents proxys ou de forêts en local n’est pas prise en charge dans les clouds non pris en charge, ce qui entraîne l’échec de toute tentative d’inscription.

**Q : Comment puis-je appliquer des avantages de la protection par mot de passe Azure AD à un sous-ensemble de mes utilisateurs en local ?**

Non pris en charge. Une fois déployée et activée, la protection par mot de passe Azure AD ne fait pas de distinction : tous les utilisateurs reçoivent les mêmes avantages en matière de protection.

**Q : Quelle est la différence entre une modification de mot de passe et une définition (ou réinitialisation) de mot de passe ?**

Une modification de mot de passe survient lorsqu’un utilisateur choisit un nouveau mot de passe après avoir prouvé qu’il a connaissance de l’ancien mot de passe. Par exemple, un changement de mot de passe se produit quand un utilisateur se connecte à Windows et est invité à choisir un nouveau mot de passe.

Une définition de mot de passe (parfois appelée une réinitialisation de mot de passe) survient lorsqu’un administrateur remplace le mot de passe d’un compte par un nouveau mot de passe, par exemple à l’aide de l’outil de gestion Utilisateurs et ordinateurs Active Directory. Cette opération requiert un niveau élevé de privilèges (généralement, administrateur de domaine), et la personne qui effectue l’opération n’a généralement pas connaissance de l’ancien mot de passe. Les scénarios de support technique opèrent souvent des définitions de mot de passe, par exemple, pour aider un utilisateur qui a oublié son mot de passe. Vous rencontrez également des situations de définition de mot de passe lorsqu’un compte d’utilisateur est créé pour la première fois avec un mot de passe.

Le comportement de la stratégie de validation de mot de passe est toujours identique, qu’il s’agisse d’une modification de mot de passe ou d’une définition de mot de passe. Le service d’agent du contrôleur de domaine pour la protection par mot de passe Azure AD consigne différents événements afin de vous informer en cas de modification ou de définition de mot de passe.  Voir [Supervision et journalisation dans la protection par mot de passe Azure AD](./howto-password-ban-bad-on-premises-monitor.md).

**Q : La protection par mots de passe Azure AD valide-t-elle les mots de passe existants après installation ?**

Non. La protection par mot de passe Azure AD peut uniquement appliquer une stratégie de mot de passe à des mots de passe en texte clair pendant une opération de définition ou de modification de mot de passe. Une fois qu’Active Directory a accepté un mot de passe, seuls des hachages spécifiques du protocole d’authentification de ce mot de passe sont conservés. Le mot de passe en texte clair n’étant jamais conservé, la protection par mot de passe Azure AD ne peut pas valider des mots de passe existants.

Après le déploiement initial de la protection par mot de passe Azure AD, l’ensemble des utilisateurs et des comptes commencent à utiliser un mot de passe validé par la protection par mot de passe Azure AD, car leurs mots de passe existants expirent normalement au fil du temps. Si nécessaire, vous pouvez accélérer ce processus en raison d’une expiration manuelle ponctuelle des mots de passe de comptes d’utilisateur.

Les comptes configurés avec l’option « Le mot de passe n’expire jamais » ne sont jamais obligés de modifier leur mot de passe, sauf en cas d’expiration manuelle.

**Q : Pourquoi des événements de mot de passe rejetés sont-ils consignés en double lorsque je tente de définir un mot de passe faible à l’aide du composant logiciel enfichable Utilisateurs et ordinateurs Active Directory ?**

Le composant logiciel enfichable Utilisateurs et ordinateurs Active Directory essaie d’abord de définir le nouveau mot de passe à l’aide du protocole Kerberos. En cas d’échec, le composant logiciel enfichable fera une seconde tentative de définition du mot de passe à l’aide d’un protocole hérité (SAM RPC). Les protocoles spécifiques utilisés ne sont pas importants. Si le nouveau mot de passe est considéré comme faible par la Protection de mots de passe Azure AD, le comportement de ce logiciel enfichable engendre la journalisation de deux ensembles d’événements de rejet de réinitialisation de mot de passe.

**Q : Pourquoi les événements de validation de mot de passe Protection par mot de passe Azure AD sont-ils journalisés avec un nom d’utilisateur vide ?**

Active Directory prend en charge la possibilité de tester un mot de passe pour voir s’il satisfait aux exigences actuelles du domaine en matière de complexité des mots de passe, par exemple à l’aide de l’API [NetValidatePasswordPolicy](/windows/win32/api/lmaccess/nf-lmaccess-netvalidatepasswordpolicy). Lorsqu’un mot de passe est validé de cette manière, les tests comprennent également une validation par des produits basés sur des DLL de filtre de mots de passe, tels que la Protection par mot de passe Azure AD, mais les noms d’utilisateur passés à une DLL de filtre de mot de passe donnée seront vides. Dans ce scénario, la Protection par mot de passe Azure AD validera toujours le mot de passe à l’aide de la stratégie de mot de passe actuellement appliquée et émettra un message de journal des événements pour capturer le résultat, mais le message du journal des événements contiendra des champs de nom d’utilisateur vides.

**Q : L’installation de la protection par mot de passe Azure AD simultanément avec d’autres produits basés sur le filtrage par mot de passe est-elle prise en charge ?**

Oui. La prise en charge de plusieurs DLL de filtrage de mots de passe enregistrés est une fonctionnalité Windows de base et n’est pas spécifique à la protection par mot de passe Azure AD. Toutes les DLL de filtrage de mots de passe enregistrés doivent valider un mot de passe avant de l’accepter.

**Q : Comment puis-je déployer et configurer la protection par mot de passe Azure AD dans mon environnement Active Directory sans utiliser Azure ?**

Non pris en charge. La protection par mot de passe Azure AD est une fonctionnalité Azure qui accepte d’être étendue à un environnement Active Directory en local.

**Q : Comment puis-je modifier le contenu de la stratégie au niveau d’Active Directory ?**

Non pris en charge. La stratégie ne peut être administrée que par le biais du portail Azure AD. Consultez également la question précédente.

**Q : Pourquoi la technologie DFSR est-elle nécessaire pour la réplication sysvol ?**

La technologie FRS (prédécesseur de la technologie DFSR) présente de nombreux problèmes connus et n’est pas du tout prise en charge par les versions plus récentes de Windows Server Active Directory. Aucun essai de la protection par mot de passe Azure AD ne sera effectué sur les domaines configurés en FRS.

Pour plus d’informations, consultez les articles suivants :

[Argumentaire en faveur de la migration de la réplication sysvol vers DFSR](/archive/blogs/askds/the-case-for-migrating-sysvol-to-dfsr)

[The End is Nigh for FRS](https://blogs.technet.microsoft.com/filecab/2014/06/25/the-end-is-nigh-for-frs)

Si votre domaine n’utilise pas encore DFSR, vous DEVEZ le faire migrer pour l’utiliser avant d’installer la protection par mot de passe Azure Active Directory. Pour plus d’informations, consultez le lien suivant :

[Guide de migration de la réplication SYSVOL : Réplication FRS vers DFS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd640019(v=ws.10))

> [!WARNING]
> Le logiciel de l’agent DC de protection par mot de passe Azure AD s’installe actuellement sur des contrôleurs de domaine dans des domaines qui utilisent encore FRS pour la réplication sysvol, mais ce logiciel ne fonctionne pas correctement dans cet environnement. Des effets secondaires supplémentaires peuvent entraîner l’échec de la réplication des fichiers individuels, et le succès apparent des procédures de restauration sysvol qui ne parviennent pas à répliquer tous les fichiers en mode silencieux. Vous devez faire migrer votre domaine pour utiliser DFSR dès que possible, à la fois pour les avantages inhérents à DFSR et pour débloquer le déploiement de la protection par mot de passe Azure AD. Les versions ultérieures du logiciel seront automatiquement désactivées lors de l’exécution dans un domaine qui utilise encore FRS.

**Q : De quelle quantité d’espace disque la fonctionnalité a besoin sur le partage sysvol du domaine ?**

L’utilisation d’un espace précis varie en fonction de facteurs, par exemple la surcharge de chiffrement, le nombre et la longueur des jetons interdits dans la liste globale interdite de Microsoft et dans la liste personnalisée par locataire. Le contenu de ces listes est susceptible d’évoluer à l’avenir. Ainsi, la prévision pour cette fonctionnalité d’au moins cinq (5) mégaoctets d’espace sur le partage sysvol du domaine est une appréciation raisonnable.

**Q : Pourquoi un redémarrage est-il nécessaire pour installer ou mettre à niveau le logiciel de l’agent contrôleur de domaine ?**

Cette exigence est due à un comportement de base de Windows.

**Q : Existe-t-il un moyen de configurer un agent contrôleur de domaine pour utiliser un serveur proxy spécifique ?**

Non. Le serveur proxy étant sans état, n’importe quel serveur proxy en particulier est utilisé.

**Q : Est-il possible de déployer le service proxy de la protection par mot de passe Azure AD, côte à côte avec d’autres services, comme Azure AD Connect ?**

Oui. Azure AD Connect et le service proxy de la protection par mot de passe Azure AD n’entrent jamais en conflit directement.

Malheureusement, une incompatibilité a été détectée entre la version du service du programme de mise à jour de l’agent Microsoft Azure AD Connect installée par le logiciel Azure AD Password Protection Proxy et la version du service installée par le logiciel [Proxy d’application Azure Active Directory](../manage-apps/application-proxy.md). Cette incompatibilité peut empêcher le service de mise à jour de l’agent de contacter Azure pour effectuer les mises à jour logicielles. Il est déconseillé d’installer d’installer Azure AD Password Protection Proxy et Proxy d'application Azure Active Directory sur le même ordinateur.

**Q : Dans quel ordre faut-il installer et inscrire les agents de contrôleur de domaine et les proxys ?**

L’ordre n’a aucune importance pour l’installation de l’agent de proxy, l’installation de l’agent du contrôleur de domaine, l’inscription de la forêt et l’inscription du proxy.

**Q : Dois-je me préoccuper des performances obtenues sur mes contrôleurs de domaine avant de déployer cette fonctionnalité ?**

Le service d’agent contrôleur de domaine de la protection par mot de passe Azure AD ne devrait pas affecter considérablement les performances du contrôleur de domaine dans un déploiement Active Directory sain existant.

Pour la plupart des déploiements Active Directory, les opérations de modification de mot de passe représentent une petite proportion de la charge de travail globale sur un contrôleur de domaine donné. Imaginez, par exemple, un domaine Active Directory avec 10 000 comptes d’utilisateur et une stratégie MaxPasswordAge définie sur 30 jours. En moyenne, ce domaine verra 10 000/30 = ~ 333 opérations de modification de mot de passe par jour, ce qui représente un nombre insignifiant d’opérations, même pour un seul contrôleur de domaine. Envisagez le pire scénario d’utilisation possible : supposez que ces quelque 333 modifications de mot de passe sur un seul contrôleur de domaine soient effectuées dans une même heure. Ce scénario peut notamment se produire lorsque beaucoup d’employés viennent tous travailler un lundi matin. Même dans ce cas, il est toujours question de ~333/60 minutes, soit six modifications de mot de passe par minute, ce qui encore une fois n’est pas une charge importante.

Par contre, si vos contrôleurs de domaine actuels s’exécutent déjà à des niveaux de performances limitées (par exemple, en limite maximale par rapport au processeur, à l’espace disque, aux e/s de disque, etc.), il est conseillé d’ajouter des contrôleurs de domaine supplémentaires, ou d’augmenter l’espace disque disponible avant le déploiement de cette fonctionnalité. Consultez également la question se rapportant à l’utilisation de l’espace disque sysvol, posée plus haut.

**Q : Je veux tester la protection par mot de passe Azure AD sur quelques contrôleurs de domaine seulement dans mon domaine. Est-il possible d’imposer des modifications de mot de passe utilisateur pour utiliser ces contrôleurs de domaine en particulier ?**

Non. Le système d’exploitation client Windows contrôle quel contrôleur de domaine est utilisé lorsqu’un utilisateur change son mot de passe. Le contrôleur de domaine est sélectionné en fonction de facteurs, tels que des attributions de site et de sous-réseau Active Directory, une configuration réseau spécifique à l’environnement, etc. La protection par mot de passe AD Azure ne contrôle pas ces facteurs, elle ne peut pas influer sur la sélection du contrôleur de domaine pour changer un mot de passe utilisateur.

Un moyen d’atteindre en partie cet objectif consisterait à déployer la protection par mot de passe Azure AD sur tous les contrôleurs de domaine dans un site Active Directory donné. Cette approche fournira une couverture raisonnable aux clients Windows qui sont attribués à ce site, et donc également aux utilisateurs qui se connectent à ces clients et changent leur mot de passe.

**Q : Si j’installe le service d’agent contrôleur de domaine de la protection par mot de passe Azure AD sur le contrôleur de domaine principal seulement, est-ce que tous les autres contrôleurs de domaine dans le domaine sont aussi protégés ?**

Non. Quand un mot de passe utilisateur est changé sur un contrôleur de domaine donné qui n’est pas contrôleur de domaine principal, le mot de passe en texte clair n’est jamais envoyé au contrôleur de domaine principal (il s’agit d’une idée fausse répandue). Dès lors qu’un nouveau mot de passe est accepté sur un contrôleur de domaine donné, ce contrôleur de domaine utilise le mot de passe pour créer les différents hachages spécifiques au protocole d’authentification de ce mot de passe, puis conserve ces hachages dans le répertoire. Le mot de passe en texte clair n’est pas gardé. Les hachages mis à jour sont ensuite répliqués sur le contrôleur de domaine principal. Les mots de passe utilisateur peuvent, dans certains cas, être changés directement sur le contrôleur de domaine principal, encore une fois en fonction de divers facteurs, tels que la topologie du réseau et la conception du site Active Directory. (Consultez la question précédente.)

En résumé, le déploiement du service d’agent contrôleur de domaine de la protection par mot de passe Azure AD sur le contrôleur de domaine principal est indispensable pour obtenir 100 % de la couverture de sécurité sur le domaine pour la fonctionnalité. Le seul déploiement de la fonctionnalité sur le contrôleur de domaine principal ne fournit pas les avantages de la sécurité basée sur la protection par mot de passe Azure AD aux autres contrôleurs de domaine dans le domaine.

**Q : Pourquoi le verrouillage intelligent personnalisé ne fonctionne-t-il pas même après l’installation des agents dans mon environnement Active Directory local ?**

Le verrouillage intelligent personnalisé est pris en charge uniquement dans Azure AD. Les modifications apportées aux paramètres de verrouillage intelligent personnalisé dans le portail Azure n’ont aucune incidence sur l’environnement Active Directory local, même avec les agents installés.

**Q : Existe-il un pack d’administration System Center Operations Manager disponible pour la protection par mot de passe Azure AD ?**

Non.

**Q : Pourquoi Azure AD refuse-t-il toujours les mots de passe faibles, même si j’ai configuré la stratégie en mode Audit ?**

Le mode Audit est uniquement pris en charge dans l’environnement Active Directory local. Azure AD est implicitement toujours en mode « appliquer » lorsqu’il évalue les mots de passe.

**Q : Mes utilisateurs voient le message d’erreur Windows classique lorsqu’un mot de passe est rejeté par la protection par mot de passe Azure AD. Est-il possible de personnaliser ce message d’erreur afin que les utilisateurs sachent ce qui s’est réellement produit ?**

Non. Le message d’erreur, affiché par les utilisateurs lorsqu’un mot de passe est rejeté par un contrôleur de domaine, est contrôlé par la machine cliente, et non par le contrôleur de domaine. Ce comportement se manifeste si un mot de passe est rejeté par les stratégies de mot de passe Active Directory par défaut, ou par une solution basée sur un filtre de mot de passe, comme la protection par mot de passe Azure AD.

## <a name="additional-content"></a>Contenu supplémentaire

Les liens suivants ne font pas partie de la documentation de base se rapportant à la protection par mot de passe Azure AD, mais peuvent être une source utile d’informations supplémentaires sur cette fonctionnalité.

[Protection par mot de passe Azure AD désormais disponible au grand public !](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-Password-Protection-is-now-generally-available/ba-p/377487)

[Guide de protection contre le hameçonnage par e-mail – Partie 15 : Implémenter le service de la protection par mot de passe de Microsoft Azure AD (en local également !)](http://kmartins.com/2018/10/14/email-phishing-protection-guide-part-15-implement-the-microsoft-azure-ad-password-protection-service-for-on-premises-too/)

[La protection par mot de passe Azure AD et le verrouillage intelligent sont désormais disponibles en préversion publique !](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-Password-Protection-and-Smart-Lockout-are-now-in-Public/ba-p/245423#M529)

## <a name="microsoft-premierunified-support-training-available"></a>Support de formation Premier\Unifié Microsoft disponible

Si la protection par mot de passe Azure AD et son déploiement dans votre environnement vous intéressent, vous pouvez profiter d’un service proactif de Microsoft, disponible pour les clients qui ont un contrat de support Premier ou Unifié. Le service s’appelle Azure Active Directory : Protection par mot de passe. Pour plus d’informations, contactez votre responsable technique de compte.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez une question sur la protection par mot de passe Azure AD en local qui serait restée sans réponse ici, adressez-nous votre question en soumettant votre commentaire ci-dessous. Merci !

[Déployer la protection par mot de passe d’Azure AD](howto-password-ban-bad-on-premises-deploy.md)