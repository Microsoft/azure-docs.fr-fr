---
title: Intégrer Azure Functions à un réseau virtuel Azure
description: Tutoriel pas à pas qui vous montre comment connecter une fonction à un réseau virtuel Azure
ms.topic: article
ms.date: 4/23/2020
ms.openlocfilehash: f50c923104fdfcf26f400f20f0de66a82eb3d245
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87387521"
---
# <a name="tutorial-integrate-functions-with-an-azure-virtual-network"></a>Tutoriel : Intégrer Azure Functions à un réseau virtuel Azure

Ce tutoriel vous montre comment utiliser Azure Functions pour vous connecter aux ressources d’un réseau virtuel Azure. Vous allez créer une fonction ayant accès à internet et à une machine virtuelle qui exécute WordPress dans le réseau virtuel.

> [!div class="checklist"]
> * Créer une application de fonction sur le plan Premium
> * Déployer un site WordPress sur machine virtuelle dans un réseau virtuel
> * Connecter l’application de fonction au réseau virtuel
> * Créer un proxy de fonction pour accéder aux ressources WordPress
> * Demander un fichier WordPress de l’intérieur du réseau virtuel

## <a name="topology"></a>Topologie

Le diagramme suivant représente l’architecture de la solution que vous créez :

 ![Interface utilisateur pour l’intégration du réseau virtuel](./media/functions-create-vnet/topology.png)

Les fonctions qui s’exécutent dans le plan Premium ont les mêmes fonctionnalités d’hébergement que les applications web dans Azure App Service, ce qui inclut la fonctionnalité d’intégration de réseau virtuel. Pour en savoir plus sur l’intégration d’un réseau virtuel, y compris la résolution des problèmes et la configuration avancée, consultez [Intégrer une application à un réseau virtuel Azure](../app-service/web-sites-integrate-with-vnet.md).

## <a name="prerequisites"></a>Prérequis

Pour ce tutoriel, il est important de comprendre l’adressage IP et la gestion des sous-réseaux. Vous pouvez commencer par [cet article qui couvre les principes fondamentaux de l’adressage et de la gestion de sous-réseaux](https://support.microsoft.com/help/164015/understanding-tcp-ip-addressing-and-subnetting-basics). De nombreux autres articles et vidéos sont disponibles en ligne.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-a-function-app-in-a-premium-plan"></a>Créer une application de fonction sur le plan Premium

Tout d’abord, vous créez une application de fonction sur le [plan Premium]. Ce plan fournit une mise à l’échelle serverless lors de la prise en charge de l’intégration d’un réseau virtuel.

[!INCLUDE [functions-premium-create](../../includes/functions-premium-create.md)]  

Vous pouvez épingler l’application de fonction au tableau de bord en sélectionnant l’icône d’épingle en haut à droite. L’épinglage permet de revenir plus facilement à cette application de fonction après avoir créé votre machine virtuelle.

## <a name="create-a-vm-inside-a-virtual-network"></a>Créer un réseau virtuel dans un réseau virtuel

Ensuite, créez une machine virtuelle préconfigurée qui exécute WordPress à l’intérieur d’un réseau virtuel ([WordPress LEMP7 Max Performance](https://jetware.io/appliances/jetware/wordpress4_lemp7-170526/profile?us=azure) par Jetware). Une machine virtuelle WordPress est utilisée en raison de son faible coût et de son côté pratique. Ce même scénario fonctionne avec n’importe quelle ressource dans un réseau virtuel, tel que les API REST, les environnements App Service et les autres services Azure. 

1. Dans le portail, choisissez **+ Créer une ressource** dans le volet de navigation de gauche, dans le champ de recherche, tapez `WordPress LEMP7 Max Performance` et appuyez sur Entrée.

1. Choisissez **Wordpress LEMP Max Performance** dans les résultats de recherche. Sélectionnez un abonnement logiciel dans **Wordpress LEMP Max Performance for CentOS** en tant qu’**abonnement logiciel** et sélectionnez **Créer**.

1. Sous l’onglet **De base**, utilisez les paramètres de machine virtuelle de la manière spécifiée dans le tableau sous l’image :

    ![Onglet De base pour la création d’une machine virtuelle](./media/functions-create-vnet/create-vm-1.png)

    | Paramètre      | Valeur suggérée  | Description      |
    | ------------ | ---------------- | ---------------- |
    | **Abonnement** | Votre abonnement | Abonnement sous lequel vos ressources sont créées. | 
    | **[Groupe de ressources](../azure-resource-manager/management/overview.md)**  | myResourceGroup | Choisissez `myResourceGroup` ou le groupe de ressources que vous avez créé avec votre application de fonction. L’utilisation du même groupe de ressources pour l’application de fonction, la machine virtuelle WordPress et le plan d’hébergement facilitera le nettoyage des ressources lorsque vous aurez terminé ce tutoriel. |
    | **Nom de la machine virtuelle** | VNET-Wordpress | Le nom de la machine virtuelle doit être unique dans le groupe de ressources |
    | **[Région](https://azure.microsoft.com/regions/)** | (Europe) Europe Ouest | Choisissez une région proche de chez vous ou près des fonctions qui accèdent à la machine virtuelle. |
    | **Taille** | B1s | Choisissez **Changer la taille**, puis sélectionnez l’image standard B1s, qui comporte 1 processeur virtuel et 1 Go de mémoire. |
    | **Type d’authentification** | Mot de passe | Pour utiliser l’authentification par mot de passe, vous devez également spécifier un **nom d’utilisateur**, un **mot de passe** sécurisé, puis **Confirmer le mot de passe**. Pour ce tutoriel, vous n’aurez pas besoin de vous connecter à la machine virtuelle, sauf si vous devez résoudre des problèmes. |

1. Choisissez l’onglet **Mise en réseau** et sous Configurer les réseaux virtuels, sélectionnez **Créer nouveau**.

1. Dans **Créer un réseau virtuel**, utilisez les paramètres figurant dans le tableau sous l’image :

    ![Onglet Mise en réseau de création d’une machine virtuelle](./media/functions-create-vnet/create-vm-2.png)

    | Paramètre      | Valeur suggérée  | Description      |
    | ------------ | ---------------- | ---------------- |
    | **Nom** | myResourceGroup-vnet | Vous pouvez utiliser le nom par défaut généré pour votre réseau virtuel. |
    | **Plage d’adresses** | 10.10.0.0/16 | Utilisez une plage d’adresses unique pour le réseau virtuel. |
    | **Nom du sous-réseau** | Tutorial-Net | Nom du sous-réseau. |
    | **Plage d’adresses** (sous-réseau) | 10.10.1.0/24   | La taille du sous-réseau définit le nombre d’interfaces qui peuvent être ajoutées au sous-réseau. Ce sous-réseau est utilisé par le site WordPress.  Un sous-réseau `/24` fournit 254 adresses d’hôte. |

1. Sélectionnez **OK** pour créer le réseau virtuel.

1. De retour sous l’onglet **Mise en réseau**, choisissez **Aucune** pour **Adresse IP publique**.

1. Choisissez l’onglet **Gestion**, puis dans **Compte de stockage des diagnostics**, choisissez le compte de stockage que vous avez créé avec votre application de fonction.

1. Sélectionnez **Revoir + créer**. Une fois la validation terminée, sélectionnez **Créer**. Le processus de création d’une machine virtuelle prend quelques minutes. La machine virtuelle créée peut uniquement accéder au réseau virtuel.

1. Une fois la machine virtuelle créée, choisissez **Accéder à la ressource** pour voir la page de votre nouvelle machine virtuelle, puis choisissez **Mise en réseau** sous **Paramètres**.

1. Vérifiez qu’il n’y a aucune **adresse IP publique**. Prenez note de l’**adresse IP privée**, qui vous permet de vous connecter à la machine virtuelle à partir de votre application de fonction.

    ![Paramètres réseau dans la machine virtuelle](./media/functions-create-vnet/vm-networking.png)

Vous disposez maintenant d’un site WordPress déployé entièrement au sein de votre réseau virtuel. Ce site n’est pas accessible par Internet.

## <a name="connect-your-function-app-to-the-virtual-network"></a>Connecter l’application de fonction au réseau virtuel

Avec un site WordPress en cours d’exécution sur une machine virtuelle dans un réseau virtuel, vous pouvez maintenant connecter votre application de fonction à ce réseau virtuel.

1. Dans votre nouvelle application de fonction, sélectionnez **Mise en réseau** dans le menu de gauche.

1. Sous **Intégration de réseau virtuel**, sélectionnez **Cliquez ici pour configurer**.

    :::image type="content" source="./media/functions-create-vnet/networking-0.png" alt-text="Choisir la mise en réseau dans l’application de fonction":::

1. Dans la page **Intégration au réseau virtuel**, sélectionnez **Ajouter un réseau virtuel**.

    :::image type="content" source="./media/functions-create-vnet/networking-2.png" alt-text="Choisir la mise en réseau dans l’application de fonction":::

1. Dans **État de la fonctionnalité réseau**, utilisez les paramètres figurant dans le tableau sous l’image :

    ![Définir le réseau virtuel de l’application de fonction](./media/functions-create-vnet/networking-3.png)

    | Paramètre      | Valeur suggérée  | Description      |
    | ------------ | ---------------- | ---------------- |
    | **Réseau virtuel** | MyResourceGroup-vnet | Ce réseau virtuel est celui que vous avez créé précédemment. |
    | **Sous-réseau** | Créer un sous-réseau | Créez un sous-réseau dans le réseau virtuel que votre application de fonction utilisera. L’intégration au réseau virtuel doit être configurée pour utiliser un sous-réseau vide. Peu importe que vos fonctions utilisent un sous-réseau différent de celui de votre machine virtuelle. Le réseau virtuel achemine automatiquement le trafic entre les deux sous-réseaux. |
    | **Nom du sous-réseau** | Function-Net | Nom du nouveau sous-réseau. |
    | **Bloc d’adresses du réseau virtuel** | 10.10.0.0/16 | Choisissez le même bloc d’adresses utilisé par le site WordPress. Vous devez avoir un seul bloc d’adresses défini. |
    | **Plage d’adresses** | 10.10.2.0/24   | La taille du sous-réseau limite le nombre total d’instances vers lequel votre application de fonction du plan Premium peut effectuer un scale-out. Cet exemple utilise un sous-réseau `/24` avec 254 adresses d’hôte disponibles. Ce sous-réseau est surprovisionné, mais facile à calculer. |

1. Sélectionnez **OK** pour ajouter le sous-réseau. Fermez les pages **Intégration de réseau virtuel** et **État de la fonctionnalité réseau** pour revenir à la page de votre application de fonction.

L’application de fonction peut désormais accéder au réseau virtuel dans lequel le site WordPress s’exécute. Ensuite, vous utilisez [Azure Functions Proxies](functions-proxies.md) pour retourner un fichier à partir du site WordPress.

## <a name="create-a-proxy-to-access-vm-resources"></a>Créer un proxy pour accéder aux ressources de machine virtuelle

Avec l’intégration de réseau virtuel activée, vous pouvez créer un proxy dans votre application de fonction pour transférer les demandes à la machine virtuelle en cours d’exécution dans le réseau virtuel.

1. Dans votre application de fonction, sélectionnez **Proxies** dans le menu de gauche, puis **Ajouter**. Utilisez les paramètres de proxy indiqués dans le tableau sous l’image ci-après :

    :::image type="content" source="./media/functions-create-vnet/create-proxy.png" alt-text="Choisir la mise en réseau dans l’application de fonction":::

    | Paramètre  | Valeur suggérée  | Description      |
    | -------- | ---------------- | ---------------- |
    | **Nom** | Plant | Vous pouvez utiliser n’importe quel nom. Il est utilisé pour identifier le proxy. |
    | **Modèle de routage** | /plant | Route qui mappe à une ressource de machine virtuelle. |
    | **URL du serveur principal** | http://<YOUR_VM_IP>/wp-content/themes/twentyseventeen/assets/images/header.jpg | Remplacez `<YOUR_VM_IP>` par l’adresse IP de votre machine virtuelle WordPress que vous avez créée précédemment. Ce mappage retourne un seul fichier à partir du site. |

1. Sélectionnez **Créer** pour ajouter le proxy à votre application de fonction.

## <a name="try-it-out"></a>Faites un essai

1. Dans votre navigateur, essayez d’accéder à l’URL que vous avez utilisée comme **URL du serveur principal**. Comme prévu, la demande arrive à expiration. Un délai d’expiration est atteint parce que votre site WordPress est connecté uniquement à votre réseau virtuel et pas à Internet.

1. Copiez la valeur **URL du proxy** à partir de votre nouveau proxy et collez-la dans la barre d’adresse de votre navigateur. L’image retournée provient du site WordPress en cours d’exécution au sein de votre réseau virtuel.

    ![Fichier image Plant retourné à partir du site WordPress](./media/functions-create-vnet/plant.png)

Votre application de fonction est connectée à la fois à Internet et à votre réseau virtuel. Le proxy reçoit une demande via Internet, puis agit en tant que proxy HTTP simple pour transférer cette demande vers le réseau virtuel connecté. Le proxy relaie ensuite la réponse pour vous la renvoyer publiquement via Internet.

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, le site WordPress sert d’API qui est appelée à l’aide d’un proxy dans l’application de fonction. Ce scénario constitue un bon tutoriel, car il est facile à configurer et à visualiser. Vous pouvez utiliser n’importe quelle autre API déployée au sein d’un réseau virtuel. Vous pouvez également créer une fonction avec du code qui appelle les API déployées au sein du réseau virtuel. Un scénario plus réaliste est une fonction qui utilise des API clientes de données pour appeler une instance SQL Server déployée dans le réseau virtuel.

Les fonctions exécutées dans un plan Premium partagent la même infrastructure App Service sous-jacente que les applications web dans les plans PremiumV2. Toute la documentation pour les [applications web dans Azure App Service](../app-service/overview.md) s’applique aux fonctions de votre plan Premium.

> [!div class="nextstepaction"]
> [En savoir plus sur les options de mise en réseau dans Functions](./functions-networking-options.md)

[Plan Premium]: functions-scale.md#premium-plan
