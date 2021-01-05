---
title: Guide de déploiement d’un accès hybride sécurisé à Azure AD avec F5 | Microsoft Docs
description: Tutoriel pour déployer une machine virtuelle F5 BIG-IP Virtual Edition (VE) dans Azure IaaS pour un accès hybride sécurisé
services: active-directory
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 10/12/2020
ms.author: gasinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6c03009b08dcf33bf4b84bc91232af96e7ba2c71
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095183"
---
# <a name="tutorial-to-deploy-f5-big-ip-virtual-edition-vm-in-azure-iaas-for-secure-hybrid-access"></a>Tutoriel pour déployer une machine virtuelle F5 BIG-IP Virtual Edition dans Azure IaaS pour un accès hybride sécurisé

Ce tutoriel vous guide tout au long du processus de déploiement de BIG-IP Virtual Edition (VE) dans Azure IaaS. À la fin de ce tutoriel, vous disposerez des éléments suivants :

- Une machine virtuelle BIG-IP entièrement préparée pour la modélisation d’une preuve de concept d’accès hybride sécurisé (SHA)

- Une instance intermédiaire permettant de tester les nouveaux correctifs et les mises à jour du système BIG-IP

## <a name="prerequisites"></a>Prérequis

Aucune expérience ni connaissance préalable de F5 BIG-IP n’est requise. Toutefois, nous vous recommandons de vous familiariser avec la [terminologie de F5 BIG-IP](https://www.f5.com/services/resources/glossary). Le déploiement d’un système BIG-IP dans Azure pour un accès hybride sécurisé requiert :

- Un abonnement Azure payant ou un [abonnement d’essai](https://azure.microsoft.com/free/) de 12 mois.

- L’une des références SKU de licence F5 BIG-IP suivantes

  - F5 BIG-IP® Best bundle

  - Licence autonome F5 BIG-IP Access Policy Manager™ (APM)

  - Licence de composant additionnel F5 BIG-IP Access Policy Manager™ (APM) sur une instance existante de BIG-IP F5 BIG-IP® Local Traffic Manager™ (LTM)
  
  - [Licence d’essai](https://www.f5.com/trial/big-ip-trial.php) de 90 jours des fonctionnalités complètes BIG-IP.

- Un certificat SAN (autre nom de sujet) ou générique pour publier des applications web via SSL (Secure Socket Layer). [Let’s Encrypt](https://letsencrypt.org/) offre un certificat gratuit de 90 jours à des fins de test.

- Un certificat SSL pour la sécurisation de l’interface de gestion BIG-IP. Un certificat servant à publier des applications web peut être utilisé si son sujet correspond au nom de domaine complet (FQDN) du système BIG-IP. Par exemple, un certificat générique défini avec un sujet *.contoso.com convient pour `https://big-ip-vm.contoso.com:8443`

Les configurations du système de base et du déploiement de machine virtuelle prennent environ 30 minutes. À ce stade, votre plateforme BIG-IP est prête pour l’implémentation des différents scénarios SHA listés [ici](f5-aad-integration.md).

Pour tester ces scénarios, ce tutoriel suppose que le système BIG-IP sera déployé dans un groupe de ressources Azure contenant un environnement Active Directory (AD). Cet environnement doit se composer d’un contrôleur de domaine (DC) et de machines virtuelles d’hôte web (IIS). Le fait de disposer de ces serveurs dans d’autres emplacements sur la machine virtuelle BIG-IP ne pose pas de problème, à condition que le système BIG-IP ait en vue chacun des rôles nécessaires à la prise en charge d’un scénario donné. Les scénarios où la machine virtuelle BIG-IP est connectée à un autre environnement via une connexion VPN sont également pris en charge.

Si vous ne disposez pas des éléments mentionnés ici pour les tests, vous pouvez déployer un environnement de domaine Active Directory entier dans Azure, à l’aide de ce [script](https://github.com/Rainier-MSFT/Cloud_Identity_Lab). Une collection d’exemples d’applications de test peut également être déployée programmatiquement sur un hôte web IIS à l’aide de cette [automatisation par script](https://github.com/jeevanbisht/DemoSuite).

>[!NOTE]
>Le [portail Azure](https://portal.azure.com/#home) évolue en permanence, de sorte que certaines étapes de ce tutoriel peuvent différer de la disposition réelle actuelle dans le portail Azure.

## <a name="azure-deployment"></a>Déploiement Azure

BIG-IP peut être déployé dans différentes topologies. Ce guide se concentre sur un déploiement d’une seule interface réseau (NIC). Toutefois, si votre déploiement BIG-IP requiert plusieurs interfaces réseau pour bénéficier d’une haute disponibilité, de la séparation des réseaux ou d’un débit supérieur à 1 Go, envisagez d’utiliser des [modèles Azure Resource Manager (ARM)](https://clouddocs.f5.com/cloud/public/v1/azure/Azure_multiNIC.html) précompilés de F5.

Effectuez les tâches suivantes pour déployer BIG-IP VE à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps).

1. Connectez-vous au [portail Azure](https://portal.azure.com/#home) avec un compte disposant des autorisations requises pour créer des machines virtuelles. Par exemple, un compte contributeur.

2. Dans la zone de recherche du ruban supérieur, tapez **marketplace** et appuyez sur **Entrée**.

3. Tapez **F5** dans le filtre de la place de marché et appuyez sur **Entrée**.

4. Sélectionnez **+ Ajouter** dans le ruban supérieur et tapez **F5** dans le filtre de la place de marché, puis appuyez sur **Entrée**.

5. Sélectionnez **F5 BIG-IP Virtual Edition (BYOL)**  > **Sélectionner un abonnement logiciel** > **F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)** .

6. Sélectionnez **Create** (Créer).

![Image montrant les étapes permettant de sélectionner un abonnement logiciel](./media/f5ve-deployment-plan/software-plan.png)

7. Parcourez le menu **De base** et utilisez les paramètres suivants :

 |  Détails du projet     |  Value     |
 |:-------|:--------|
 |Abonnement|Abonnement cible pour le déploiement de la machine virtuelle BIG-IP|
 |Resource group | Groupe de ressources Azure existant dans lequel la machine virtuelle BIG-IP sera déployée, ou créez-en un. Doit être le même groupe de ressources que celui de vos machines virtuelles DC et IIS|
 | **Détails de l’instance**|  |
 |Nom de la machine virtuelle| Exemple de machine virtuelle BIG-IP |
 |Région | Cible géographique Azure pour la machine virtuelle BIG-IP |
 |Options de disponibilité| Activez uniquement si vous utilisez la machine virtuelle en production|
 |Image| F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)|
 |Instance Azure Spot| Non, mais n’hésitez pas à l’activer le cas échéant |
 |Taille| La spécification minimale doit être de 2 processeurs virtuels et de 8 Go de mémoire|
 |**Compte administrateur**|  |
 |Type d'authentification|Sélectionnez provisoirement un mot de passe. Vous pourrez opter ultérieurement pour une paire de clés |
 |Nom d’utilisateur|Identité qui sera créée en tant que compte local BIG-IP pour accéder à ses interfaces de gestion. Le nom d’utilisateur respecte la casse.|
 |Mot de passe|Sécurisez l’accès administrateur par un mot de passe fort|
 |**Règles des ports d’entrée**|  |
 |Aucun port d’entrée public|Aucun|

8. Sélectionnez **Suivant : Disques** en laissant toutes les valeurs par défaut, puis sélectionnez **Suivant : Mise en réseau**.

9. Dans le menu **Mise en réseau**, renseignez ces paramètres.

 |interface réseau|      Valeur |
 |:--------------|:----------------|
 |Réseau virtuel|Même réseau virtuel Azure utilisé par votre contrôleur de domaine et vos machines virtuelles IIS, ou créez-en un|
 |Subnet| Même sous-réseau interne Azure que votre contrôleur de domaine et vos machines virtuelles IIS, ou créez-en un|
 |Adresse IP publique |  Aucun|
 |Groupe de sécurité réseau de la carte réseau| Sélectionnez Aucun si le sous-réseau Azure que vous avez sélectionné à l’étape précédente est déjà associé à un groupe de sécurité réseau (NSG). Sinon, sélectionnez De base|
 |Mise en réseau accélérée| Désactivé |
 |**Équilibrage de charge**|     |
 |Équilibrer la charge des machines virtuelles| Non|

10. Sélectionnez **Suivant : Gestion** et renseignez les paramètres suivants.

 |Surveillance|    Value |
 |:---------|:-----|
 |Surveillance détaillée| Désactivé|
 |Diagnostics de démarrage|Activez avec un compte de stockage personnalisé. Permet une connexion à l’interface SSH (Secure Shell) BIG-IP via l’option Console série dans le portail Azure. Sélectionnez un compte de stockage Azure disponible quelconque|
 |**Identité**|  |
 |Identité gérée attribuée par le système|Désactivé|
 |Azure Active Directory|BIG-IP ne prend pas en charge cette option pour le moment|
 |**Arrêt automatique**|    |
 |Activer l’arrêt automatique| Si vous effectuez un test, envisagez de configurer la machine virtuelle BIG-IP-pour qu’elle soit arrêtée quotidiennement|

11. Sélectionnez **Suivant : Avancé** en laissant toutes les valeurs par défaut, puis sélectionnez **Suivant : Étiquettes**.

12. Sélectionnez **Suivant : Vérifier + créer** pour passer en revue vos configurations de machine virtuelle BIG-IP, avant de sélectionner **Créer** pour lancer le déploiement.

13. Le temps requis pour déployer complètement une machine virtuelle BIG-IP est généralement de 5 minutes. Lorsque vous avez terminé, ne sélectionnez pas **Accéder à la ressource**, mais développez plutôt le menu de gauche du portail Azure et sélectionnez **Groupes de ressources** pour accéder à votre nouvelle machine virtuelle BIG-IP. En cas d’échec de la création de la machine virtuelle, sélectionnez **Précédent** et **Suivant**.

## <a name="network-configuration"></a>Configuration réseau

Lors du premier démarrage de la machine virtuelle BIG-IP, sa carte réseau est provisionnée avec une adresse IP privée **principale** émise par le service DHCP (Dynamic Host Configuration Protocol) du sous-réseau Azure auquel elle est connectée. Cette adresse IP sera utilisée par le système d’exploitation TMOS du système BIG-IP pour communiquer avec :

- Communication avec d’autres hôtes et services

- Accès sortant au réseau Internet public

- Accès entrant aux interfaces de gestion SSH et de configuration web du système BIG-IP

L’exposition de l’une de ces interfaces de gestion à Internet augmente la surface d’attaque du système BIG-IP, ce qui explique pourquoi l’adresse IP principale du système BIG-IP n’a pas été provisionnée avec une adresse IP publique pendant le déploiement. Au lieu de cela, une adresse IP interne secondaire et une adresse IP publique associée seront provisionnées pour les services de publication.
Ce mappage 1 à 1 entre une adresse IP privée et une adresse IP publique de machine virtuelle permet au trafic externe d’atteindre une machine virtuelle. Toutefois, une règle de groupe de sécurité réseau Azure est également requise pour autoriser le trafic, à peu près de la même façon qu’un pare-feu.

Le diagramme montre un déploiement d’une seule carte réseau d’un système BIG-IP VE dans Azure, configuré avec une adresse IP principale pour la gestion et les opérations générales, et les adresses IP d’un serveur virtuel distinct pour les services de publication.
Dans cette organisation, une règle de groupe de sécurité réseau autorise le routage du trafic distant destiné à `intranet.contoso.com` vers l’adresse IP publique pour le service publié, avant d’être transféré vers le serveur virtuel BIG-IP.

![Image montrant le déploiement à une seule carte réseau](./media/f5ve-deployment-plan/single-nic-deployment.png) Par défaut, les adresses IP privée et publique émises pour les machines virtuelles Azure sont toujours dynamiques et changeront probablement à chaque redémarrage d’une machine virtuelle. Évitez les problèmes de connectivité imprévus en rendant statique l’adresse IP de gestion du système BIG-IP et procédez de la même façon avec les adresses IP secondaires utilisées pour les services de publication.

1. Dans le menu de votre machine virtuelle BIG-IP, accédez à **Settings** (Paramètres) > **Networking** (Mise en réseau).

2. Dans la vue de mise en réseau, sélectionnez le lien à droite du champ **Network Interface** (Interface réseau).

![Image montrant la configuration réseau](./media/f5ve-deployment-plan/network-config.png)

>[!NOTE]
>Les noms de machine virtuelle sont générés de manière aléatoire lors du déploiement.

3. Dans le volet gauche, sélectionnez **IP configurations** (Configurations IP), puis sélectionnez la ligne **ipconfig1**.

4. Définissez l’option **IP Assignment** (Affectation IP) sur statique et, si nécessaire, modifiez l’adresse IP principale de la machine virtuelle BIG-IP. Ensuite, sélectionnez **Save** (Enregistrer) et fermez le menu ipconfig1.

>[!NOTE]
>Vous utiliserez cette adresse IP principale pour la connexion et la gestion de la machine virtuelle BIG-IP.

5. Sélectionnez **+ Add** (+ Ajouter) dans le ruban supérieur et fournissez un nom pour une adresse IP privée secondaire, par exemple, ipconfig2.

6. Définissez l’option **Allocation** des paramètres d’adresse IP privée sur **Static** (Statique). Le fait de fournir l’adresse IP consécutive supérieure ou inférieure suivante permet de maintenir l’ordre des éléments.

7. Définissez l’adresse IP publique sur **Associate** (Associer) et sélectionnez **Create** (Créer).

8. Donnez un nom à la nouvelle adresse IP publique, par exemple BIG-IP-VM_ipconfig2_Public.

9. Si vous y êtes invité, définissez la référence **SKU** sur Standard.

10. Si vous y êtes invité, définissez **Tier** (Niveau) sur **Global**.

11. Définissez l’option **Assignment** (Affectation) sur **Static** (Statique), puis sélectionnez **OK** deux fois.

Votre machine virtuelle BIG-IP est maintenant prête à être configurée avec :

- **Adresse IP privée principale** : dédiée à la gestion de la machine virtuelle BIG-IP via son utilitaire de configuration web et SSH. Elle sera utilisée par le système BIG-IP comme son adresse **Self-IP** pour se connecter aux services back-end publiés. En outre, elle connecte à des services externes tels que NTP, AD et LDAP.

- **Adresse IP privée secondaire** : utilisée lors de la création d’un serveur virtuel BIG-IP APM pour être à l’écoute d’une demande entrante adressée à un ou plusieurs services publiés.

- **Adresse IP publique** : associée à l’adresse IP privée secondaire, elle permet au trafic client provenant du réseau Internet public d’atteindre le serveur virtuel BIG-IP pour le ou les services publiés.

L’exemple illustre la relation 1 à 1 entre les adresses IP publique et privée de la machine virtuelle. Une carte réseau de machine virtuelle Azure ne peut avoir qu’une seule adresse IP principale, et toutes les adresses IP supplémentaires créées sont considérées secondaires.

>[!NOTE]
>Vous aurez besoin des mappages d’adresses IP secondaires pour la publication des services BIG-IP.

![Image montrant toutes les adresses IP secondaires](./media/f5ve-deployment-plan/secondary-ips.png)

Pour implémenter SHA à l’aide de la **configuration guidée d’accès** BIG-IP, répétez les étapes 5-11 pour créer une paire d’adresses IP privée et publique supplémentaire pour chaque service supplémentaire que vous envisagez de publier via BIG-IP APM. La même approche peut également être utilisée si vous publiez des services en utilisant la **configuration avancée** de BIG-IP. Toutefois, dans ce scénario, vous avez la possibilité d’éviter les frais généraux liés aux adresses IP publiques en utilisant une configuration [SNI (Server Name Indicator)](https://support.f5.com/csp/#/article/K13452). Dans cette configuration, un serveur virtuel BIG-IP accepte tout le trafic client qu’il reçoit et le route jusqu’à sa destination.

## <a name="dns-configuration"></a>Configuration DNS

Il est essentiel que DNS soit configuré pour que les clients résolvent vos services SHA publiés sur la ou les adresses IP publiques de votre machine virtuelle BIG-IP.

Les étapes suivantes supposent que la zone DNS du domaine public utilisé pour vos services SHA est gérée dans Azure. Toutefois, les mêmes principes DNS de création d’enregistrement de localisateur s’appliquent toujours, quel que soit l’endroit où votre zone DNS est gérée.

1. S’il n’est pas déjà ouvert, développez le menu de gauche du portail et accédez à votre machine virtuelle BIG-IP via l’option **Groupes de ressources**.

2. Dans le menu de votre machine virtuelle BIG-IP, accédez à **Settings** (Paramètres) > **Networking** (Mise en réseau).

3. Dans la vue Mise en réseau de la machine virtuelle BIG-IP, sélectionnez la première adresse IP secondaire dans la liste déroulante des configurations IP et sélectionnez le lien correspondant à son **adresse IP publique de carte réseau**.

![Capture d’écran montrant l’adresse IP publique de carte réseau](./media/f5ve-deployment-plan/networking.png)

4. Sous la section **Settings** (Paramètres) du volet gauche, sélectionnez **Configuration** pour afficher le menu des propriétés DNS et d’adresse IP publique pour l’adresse IP secondaire sélectionnée.

5. Sélectionnez et **créez** un enregistrement d’alias, puis sélectionnez votre **zone DNS** dans la liste déroulante. Si une zone DNS n’existe pas encore, elle peut être gérée en dehors d’Azure ou vous pouvez en créer une pour le suffixe de domaine que vous auriez vérifié dans Azure AD.

6. Utilisez les informations suivantes pour créer le premier enregistrement d’alias DNS :

 | Champ | Valeur |
 |:-------|:-----------|
 |Abonnement| Même abonnement que celui de la machine virtuelle BIG-IP|
 |Zone DNS| Zone DNS faisant autorité pour le suffixe de domaine vérifié que vos sites web publiés utiliseront, par exemple, www.contoso.com |
 |Nom | Le nom d’hôte que vous spécifiez est résolu en adresse IP publique associée à l’adresse IP secondaire sélectionnée. Veillez à définir les mappages DNS à IP corrects. Consultez la dernière image dans la section de configuration de mise en réseau, par exemple, intranet.contoso.com > 13.77.148.215|
 | TTL | 1 |
 |Unités de durée de vie | Heures |

7. Sélectionnez **Create** (Créer) pour qu’Azure enregistre ces paramètres dans le système DNS public.

8. Laissez le paramètre **DNS name label (optional)** (Étiquette de nom DNS (facultatif)) et sélectionnez **Save** (Enregistrer) avant de fermer le menu Public IP (Adresse IP publique).

9. Répétez les étapes 1 à 6 pour créer des enregistrements DNS supplémentaires pour chaque service que vous envisagez de publier à l’aide de la configuration guidée BIG-IP.

  Avec les enregistrements DNS en place, vous pouvez utiliser l’un des outils en ligne tels que [DNS Checker](https://dnschecker.org/) pour vérifier qu’un enregistrement créé a été propagé avec succès sur tous les serveurs DNS publics globaux.

  Si vous gérez votre espace de noms de domaine DNS à l’aide d’un fournisseur externe comme [GoDaddy](https://www.godaddy.com/), vous devez créer des enregistrements à l’aide de son propre dispositif de gestion DNS.

>[!NOTE]
>Vous pouvez également utiliser le fichier des hôtes locaux d’un PC si vous testez et échangez fréquemment les enregistrements DNS. Le fichier localhosts sur un PC Windows est accessible en appuyant sur Win + R sur le clavier et en soumettant le mot **drivers** dans la zone Ouvrir. N’oubliez pas qu’un enregistrement localhost fournira uniquement la résolution DNS pour le PC local, et non pour d’autres clients.

## <a name="client-traffic"></a>Trafic client

Par défaut, les réseaux virtuels Azure et les sous-réseaux associés sont des réseaux privés qui ne peuvent pas recevoir le trafic Internet. La carte réseau de votre machine virtuelle BIG-IP doit être attachée au groupe de sécurité réseau que vous avez spécifié lors du déploiement. Pour que le trafic web externe atteigne la machine virtuelle BIG-IP, vous devez définir une règle de groupe de sécurité réseau entrante pour autoriser la traversée des ports 443 (HTTPS) et 80 (HTTP) à partir du réseau Internet public.

1. Dans le menu **Overview** (Présentation) principal de la machine virtuelle BIG-IP, sélectionnez **Networking** (Mise en réseau).

2. Sélectionnez **Add inbound rule** (Ajouter une règle de trafic entrant) pour entrer les propriétés de règle de groupe de sécurité réseau suivantes :

 |     Champ   |   Valeur        |
 |:------------|:------------|
 |Source| Quelconque|
 |Source port ranges| *|
 |Adresses IP de destination|Liste séparée par des virgules de toutes les adresses IP privées secondaires de la machine virtuelle BIG-IP|
 |Ports de destination| 80,443|
 |Protocole| TCP |
 |Action| Autoriser|
 |Priority|Valeur disponible la plus basse entre 100 et 4096|
 |Nom | Nom descriptif, par exemple : `BIG-IP-VM_Web_Services_80_443`|

3. Sélectionnez **Add** (Ajouter) pour valider les modifications, puis fermez le menu **Networking** (Mise en réseau).

Le trafic HTTP et HTTPS provenant d’un emplacement quelconque est désormais autorisé à atteindre toutes vos interfaces secondaires de machine virtuelle BIG-IP. Le fait d’autoriser le port 80 est utile pour permettre à BIG-IP APM de rediriger automatiquement les utilisateurs de HTTP vers HTTPS. Cette règle peut être modifiée pour ajouter ou supprimer des adresses IP de destination, si nécessaire.

## <a name="manage-big-ip"></a>Gérer BIG-IP

Un système BIG-IP est administré via son interface utilisateur de configuration web, accessible à l’aide des différentes méthodes recommandées suivantes :

- À partir d’un ordinateur figurant dans le réseau interne du système BIG-IP

- À partir d’un client VPN connecté au réseau interne de la machine virtuelle BIG-IP

- Publié via un [proxy d’application Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-add-on-premises-application)

Vous devez décider de la méthode la plus appropriée pour pouvoir continuer avec les configurations restantes. Si nécessaire, vous pouvez vous connecter directement à la configuration web à partir d’Internet en configurant l’adresse IP principale du système BIG-IP avec une adresse IP publique. Ajoutez ensuite une règle de groupe de sécurité réseau pour autoriser le trafic 8443 vers cette adresse IP principale. Veillez à limiter la source à votre propre adresse IP approuvée. Sinon, n’importe qui pourra se connecter.

Une fois que vous êtes prêt, confirmez que vous pouvez vous connecter à la configuration web de la machine virtuelle BIG-IP et vous connecter avec les informations d’identification spécifiées lors du déploiement de la machine virtuelle :

- Si vous vous connectez à partir d’une machine virtuelle sur son réseau interne ou via un VPN, connectez-vous directement à l’adresse IP principale et au port de configuration web du système BIG-IP. Par exemple : `https://<BIG-IP-VM_Primary_IP:8443`. Votre navigateur vous indiquera que la connexion n’est pas sécurisée, mais vous pouvez ignorer l’invite jusqu’à ce que le système BIG-IP soit configuré. Si le navigateur insiste pour bloquer l’accès, effacez son cache et réessayez.

- Si vous avez publié la configuration web via le proxy d’application, utilisez l’URL définie pour accéder en externe à la configuration web, sans ajouter le port, par exemple, `https://big-ip-vm.contoso.com`. L’URL interne doit être définie à l’aide du port de configuration web, par exemple, `https://big-ip-vm.contoso.com:8443` 

Un système BIG-IP peut également être géré via son environnement SSH sous-jacent, qui est généralement utilisé pour les tâches de ligne de commande (interface CLI) et l’accès au niveau racine. Plusieurs options existent pour la connexion à l’interface CLI, y compris :

- [Service Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview) : permet des connexions rapides et sécurisées à n’importe quelle machine virtuelle au sein d’un réseau virtuel, depuis n’importe quel emplacement.

- Connexion directe via un client SSH tel que PuTTY via l’approche JIT

- Console série : proposée au bas de la section Support et dépannage du menu de machine virtuelle dans le portail. Elle ne prend pas en charge les transferts de fichiers.

- Comme pour la configuration web, vous pouvez vous connecter directement à l’interface CLI à partir d’Internet en configurant l’adresse IP principale du système BIG-IP avec une adresse IP publique et en ajoutant une règle de groupe de sécurité réseau pour autoriser le trafic SSH. Là encore, veillez à limiter la source à votre propre adresse IP approuvée, si vous utilisez cette méthode.  

## <a name="big-ip-license"></a>Licence BIG-IP

Un système BIG-IP doit être activé et provisionné avec le module APM avant de pouvoir être configuré pour les services de publication et SHA.

1. Reconnectez-vous à la configuration web et, dans la page **General properties** (Propriétés générales), sélectionnez **Activate** (Activer).

2. Dans le champ **Base Registration key** (Clé d’inscription de base), entrez la clé respectant la casse fournie par F5.

3. Laissez le champ **Activation Method** (Méthode d’activation) défini sur **Automatic** (Automatique) et sélectionnez **Next** (Suivant). Le système BIG-IP validera la licence et affichera le contrat de licence utilisateur final (CLUF).

4. Sélectionnez **Accept** (Accepter) et attendez la fin de l’activation, puis sélectionnez **Continue** (Continuer).

5. Reconnectez-vous et, en bas de la page License summary (Récapitulatif des licences), sélectionnez **Next** (Suivant). Le système BIG-IP affiche désormais la liste des modules qui fournissent les différentes fonctionnalités requises pour SHA. Si vous ne voyez pas cela, sous l’onglet principal, accédez à **System** (Système) > **Resource Provisioning** (Provisionnement des ressources).

6. Examinez la colonne de provisionnement pour Access Policy (APM).

![Image montrant le provisionnement d’accès](./media/f5ve-deployment-plan/access-provisioning.png)

7. Sélectionnez **Submit** (Envoyer) et acceptez l’avertissement.

8. Attendez que le système BIG-IP termine l’initialisation des nouvelles fonctionnalités. Plusieurs cycles peuvent être nécessaires avant que l’initialisation soit complète. 

9. Quand vous êtes prêt, sélectionnez **Continue** (Continuer), puis, sous l’onglet **About** (À propos de), sélectionnez **Run the setup utility** (Exécuter l’utilitaire d’installation).

>[!IMPORTANT]
>Les licences F5 doivent obligatoirement être consommées par une seule instance BIG-IP VE à un moment donné. Vous pouvez souhaiter migrer une licence d’une instance vers une autre. Dans ce cas, veillez à [révoquer](https://support.f5.com/csp/article/K41458656) votre licence d’essai sur l’instance active avant de la retirer. Sinon, la licence sera définitivement perdue.

## <a name="provision-big-ip"></a>Provisionner BIG-IP

La sécurisation du trafic de gestion vers et depuis une configuration web BIG-IP est primordiale. Configurez un certificat de gestion d’appareil pour empêcher la compromission du canal de configuration web.

1. Dans la barre de navigation de gauche, accédez à **System** (Système) > **Certificate Management** (Gestion des certificats) > **Traffic Certificate Management** (Gestion des certificats de trafic) > **SSL Certificate List** (Liste des certificats SSL) > **Import** (Importer).

2. Dans la liste déroulante **Import Type** (Type d’importation), sélectionnez **PKCS 12(IIS)** et **Choose File** (Choisir un fichier). Recherchez un certificat web SSL ayant un nom de sujet ou un SAN qui couvre le nom de domaine complet que vous allez attribuer à la machine virtuelle BIG-IP dans les étapes suivantes.

3. Fournissez le mot de passe pour le certificat, puis sélectionnez **Import** (Importer).

4. Dans la barre de navigation de gauche, accédez à **System** (Système) > **Platform** (Plateforme).

5. Configurez la machine virtuelle BIG-IP avec un nom d’hôte complet et le fuseau horaire de son environnement, puis sélectionnez **Update** (Mettre à jour).

![Image montrant les propriétés générales](./media/f5ve-deployment-plan/general-properties.png)

6. Dans la barre de navigation de gauche, accédez à **System** (Système) > **Configuration** > **Device** (Appareil) > **NTP**.

7. Spécifiez une source NTP fiable et sélectionnez **Add** (Ajouter), puis **Update** (Mise à jour). Par exemple : `time.windows.com`

Vous avez maintenant besoin d’un enregistrement DNS pour résoudre le nom de domaine complet du système BIG-IP, spécifié dans les étapes précédentes, avec son adresse IP privée principale. Un enregistrement doit être ajouté au DNS interne de votre environnement ou au fichier localhost d’un PC qui sera utilisé pour se connecter à la configuration web du système BIG-IP. Dans les deux cas, l’avertissement du navigateur ne doit plus s’afficher lorsque vous vous connectez directement à la configuration web. Autrement dit, pas via le proxy d’application ni aucun autre proxy inverse.

## <a name="ssl-profile"></a>Profil SSL

En tant que proxy inverse, un système BIG-IP peut agir comme un service de transfert simple, également appelé proxy transparent, ou comme un proxy complet qui participe activement aux échanges entre les clients et les serveurs.
Un proxy complet crée deux connexions distinctes : une connexion client TCP frontale et une connexion serveur TCP back-end distincte, couplées par un écart conditionnel au milieu. Les clients se connectent au port d’écoute proxy à une extrémité, également appelé **serveur virtuel**, et le proxy établit une connexion distincte et indépendante au serveur back-end. Ceci est bidirectionnel des deux côtés.
Dans ce mode proxy complet, le système F5 BIG-IP est en mesure d’inspecter le trafic. Par conséquent, une interaction avec les demandes et les réponses est également possible. Certaines fonctions, telles que l’équilibrage de charge et l’optimisation des performances web, ainsi que des services de gestion de trafic plus avancés, tels que la sécurité de la couche Application, l’accélération web, le routage des pages et l’accès à distance sécurisé, s’appuient sur cette fonctionnalité.
Lors de la publication de services SSL, le processus de déchiffrement et de chiffrement du trafic entre les clients et les services back-end est traité par les profils SSL BIG-IP.

Deux types de profils existent :

- **SSL client** : La méthode la plus répandue de configuration d’un système BIG-IP pour publier des services internes avec SSL consiste à créer un profil SSL client. Avec un profil SSL client, un système BIG-IP peut déchiffrer les demandes client entrantes avant de les envoyer à un service en aval. Il chiffre ensuite les réponses back-end sortantes avant de les envoyer aux clients.

- **SSL serveur** : Pour les services back-end configurés pour HTTPS, vous pouvez configurer BIG-IP pour utiliser un profil SSL serveur. Avec un profil SSL serveur, le système BIG-IP rechiffre la demande client avant de l’envoyer au service back-end de destination. Lorsque le serveur retourne une réponse chiffrée, le système BIG-IP la déchiffre et la rechiffre avant de l’envoyer au client via le profil SSL client configuré.

Le provisionnement des deux profils SSL client et serveur permet d’avoir le système BIG-IP préconfiguré et prêt pour tous les scénarios SHA.

1. Dans la barre de navigation de gauche, accédez à **System** (Système) > **Certificate Management** (Gestion des certificats) > **Traffic Certificate Management** (Gestion des certificats de trafic) > **SSL Certificate List** (Liste des certificats SSL) > **Import** (Importer).

2. Dans la liste déroulante **Import Type** (Type d’importation), sélectionnez **PKCS 12(IIS)** .

3. Donnez un nom au certificat importé, par exemple, `ContosoWilcardCert`.

4. Sélectionnez **Choose File** (Choisir un fichier) pour accéder au certificat web SSL dont le nom de sujet correspond au suffixe de domaine que vous prévoyez d’utiliser pour les services publiés.

5. Spécifiez le **mot de passe** pour le certificat importé, puis sélectionnez **Import** (Importer).

6. Dans la barre de navigation de gauche, accédez à **Local Traffic** (Trafic local) > **Profiles** (Profils) > **SSL** > **Client**, puis sélectionnez **Create** (Créer).

7. Dans la page **New Client SSL Profile** (Nouveau profil SSL client), fournissez un nom convivial unique pour le nouveau profil SSL client et veillez à ce que le profil parent soit défini sur **clientssl**.

![Image montrant la mise à jour BIG-IP](./media/f5ve-deployment-plan/client-ssl.png)

8. Cochez la case la plus à droite dans les lignes **Certificate Key Chain** (Chaîne de clé du certificat) et sélectionnez **Add** (Ajouter).

9. Dans les trois listes déroulantes, sélectionnez le certificat générique que vous avez importé sans phrase secrète, puis sélectionnez **Add** (Ajouter) > **Finished** (Terminé).

![Image montrant la mise à jour BIG-IP ContosoWildcard](./media/f5ve-deployment-plan/contoso-wildcard.png)

10. Répétez les étapes 6-9 pour créer un **profil de certificat de serveur SSL**. Dans le ruban supérieur, sélectionnez **SSL** > **Server** (Serveur) > **Create** (Créer).

11. Dans la page **New Server SSL Profile** (Nouveau profil SSL serveur), fournissez un nom convivial unique pour le nouveau profil SSL serveur et veillez à ce que le profil parent soit défini sur **serverssl**.

12. Cochez la case la plus à droite dans les lignes Certificate (Certificat) et Key (Clé), sélectionnez votre certificat importé dans la liste déroulante, puis sélectionnez **Finished** (Terminé).

![Image montrant tout le profil serveur de la mise à jour BIG-IP](./media/f5ve-deployment-plan/server-ssl-profile.png)

>[!NOTE]
>Ne vous inquiétez pas si vous ne parvenez pas à fournir un certificat SSL, vous pouvez utiliser les certificats SSL client et serveur BIG-IP autosignés. Vous verrez simplement une erreur de certificat dans le navigateur.

Une dernière étape de la préparation d’un système BIG-IP pour SHA consiste à s’assurer qu’il est en mesure de localiser les ressources qu’il publie, ainsi que le service d’annuaire sur lequel il s’appuie pour l’authentification unique. Un système BIG-IP a deux sources de résolution de noms. Il commence par son fichier local/.../hosts, mais si un enregistrement est introuvable, le système BIG-IP utilise le service DNS avec lequel il a été configuré. La méthode avec le fichier hosts ne s’applique pas aux nœuds APM ni aux pools qui utilisent un nom de domaine complet.

1. Dans la configuration web, accédez à **System** (Système) > **Configuration** > **Device** (Appareil) > **DNS**

2. Dans **DNS Lookup Server List** (Liste de serveurs de recherche DNS), entrez l’adresse IP du serveur DNS de votre environnement.

3. Sélectionnez **Add** (Ajouter) > **Update** (Mise à jour).

Dans le cadre d’une étape distincte et facultative, vous pouvez envisager d’utiliser une [configuration LDAP](https://somoit.net/f5-big-ip/authentication-using-active-directory) pour authentifier les administrateurs système BIG-IP auprès d’Active Directory au lieu de gérer les comptes BIG-IP locaux.

## <a name="update-big-ip"></a>Mettre à jour BIG-IP

Votre machine virtuelle BIG-IP doit être mise à jour pour déverrouiller toutes les nouvelles fonctionnalités décrites dans les [conseils basés sur des scénarios](f5-aad-integration.md). Vous pouvez vérifier la version TMOS du système en plaçant le pointeur de la souris sur le nom d’hôte du système BIG-IP en haut à gauche de la page principale. Il est recommandé d’exécuter la version v15.x ou ultérieure, disponible en tant que [téléchargement F5](https://downloads.f5.com/esd/productlines.jsp). Utilisez ces [conseils](https://support.f5.com/csp/article/K34745165) pour mettre à jour le système d’exploitation (TMOS) principal.

Si la mise à jour du système d’exploitation TMOS principal n’est pas possible, envisagez au moins de mettre à niveau la configuration guidée en suivant les étapes ci-dessous.

1. Dans l’onglet principal de la configuration web BIG-IP, accédez à **Access** (Accéder) > **Guided Configuration** (Configuration guidée).

2. Dans la page **Guided Configuration** (Configuration guidée), sélectionnez **Upgrade Guided Configuration** (Mettre à niveau la configuration guidée).

![Image montrant comment mettre à jour BIG-IP](./media/f5ve-deployment-plan/update-big-ip.png)

3. Dans la boîte de dialogue **Upgrade Guided Configuration** (Mettre à niveau la configuration guidée), sélectionnez **Choose File** (Choisir un fichier) pour charger le pack de cas d’usage téléchargé, et sélectionnez **Upload and Install** (Charger et installer).

4. Une fois la mise à niveau terminée, sélectionnez **Continue** (Continuer).

## <a name="backup-big-ip"></a>Sauvegarder BIG-IP

Le système BIG-IP étant désormais pleinement provisionné, nous vous recommandons d’effectuer une sauvegarde complète de sa configuration :

1. Accédez à **System** (Système) > **Archives** > **Create** (Créer).

2. Fournissez un **nom de fichier** unique et activez **Encryption** (Chiffrement) avec une phrase secrète.

3. Définissez l’option **Private Keys** (Clés privées) sur **Include** (Inclure) pour sauvegarder également les certificats d’appareil et SSL.

4. Sélectionnez **Finished** (Terminé) et attendez la fin du processus.

5. Un état d’opération s’affiche pour indiquer l’état du résultat de la sauvegarde. Sélectionnez **OK**.

6. Enregistrez localement l’archive UCS (User Configuration Set) en choisissant le lien de la sauvegarde, puis sélectionnez **Download** (Télécharger).

En guise d’étape facultative, vous pouvez également effectuer une sauvegarde de l’ensemble du disque système à l’aide d’[instantanés Azure](https://docs.microsoft.com/azure/virtual-machines/windows/snapshot-copy-managed-disk), qui, contrairement à la sauvegarde de la configuration web, fournirait un plan d’urgence pour tester les versions de TMOS ou effectuer une restauration vers un nouveau système.

```PowerShell
# Install modules
Install-module Az
Install-module AzureVMSnapshots

# Authenticate to Azure
Connect-azAccount

# Set subscription by Id
Set-AzContext -SubscriptionId ‘<Azure_Subscription_ID>’

#Create Snapshot
New-AzVmSnapshot -ResourceGroupName '<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>'

#List Snapshots
#Get-AzVmSnapshot -ResourceGroupName ‘<E.g.contoso-RG>'

#Get-AzVmSnapshot -ResourceGroupName ‘<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>' | Restore-AzVmSnapshot -RemoveOriginalDisk 

```

## <a name="restore-big-ip"></a>Restaurer BIG-IP

La restauration d’un système BIG-IP suit une procédure similaire à la sauvegarde et peut également être utilisée pour migrer des configurations entre des machines virtuelles BIG-IP. Les détails sur les chemins de mise à niveau pris en charge doivent être observés avant d’importer une sauvegarde.

1. Accédez à **System** (Système) > **Archives**.

2. Choisissez le lien d’une sauvegarde que vous souhaitez restaurer ou sélectionnez le bouton **Upload** (Charger) pour accéder à une archive UCS précédemment enregistrée qui ne figure pas dans la liste.

3. Spécifiez la phrase secrète pour la sauvegarde et sélectionnez **Restore** (Restaurer).

```PowerShell
# Authenticate to Azure
Connect-azAccount

# Set subscription by Id
Set-AzContext -SubscriptionId ‘<Azure_Subscription_ID>’

#Restore Snapshot
Get-AzVmSnapshot -ResourceGroupName '<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>' | Restore-AzVmSnapshot

```

>[!NOTE]
>Au moment de la rédaction du présent article, l’applet de commande AzVmSnapshot est limitée à la restauration de l’instantané le plus récent, en fonction de la date. Les instantanés sont stockés à la racine du groupe de ressources de la machine virtuelle. Sachez que la restauration d’instantanés entraîne le redémarrage d’une machine virtuelle Azure. Vous devez donc choisir un moment opportun pour cette opération.

## <a name="additional-resources"></a>Ressources supplémentaires

-   [Réinitialiser le mot de passe BIG-IP VE dans Azure](https://clouddocs.f5.com/cloud/public/v1/shared/azure_passwordreset.html)
    -   [Réinitialiser le mot de passe sans utiliser le portail](https://clouddocs.f5.com/cloud/public/v1/shared/azure_passwordreset.html#reset-the-password-without-using-the-portal)

-   [Changer la carte réseau utilisée pour la gestion de BIG-IP VE](https://clouddocs.f5.com/cloud/public/v1/shared/change_mgmt_nic.html)

-   [À propos des routes dans une configuration à une seule carte réseau](https://clouddocs.f5.com/cloud/public/v1/shared/routes.html)

-   [Microsoft Azure : Waagent](https://clouddocs.f5.com/cloud/public/v1/azure/Azure_waagent.html)

## <a name="next-steps"></a>Étapes suivantes

Sélectionnez un [scénario de déploiement](f5-aad-integration.md) et commencez votre implémentation.