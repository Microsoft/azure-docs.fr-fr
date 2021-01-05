---
title: Modèles d’intégration de contrat intelligent - Azure Blockchain Workbench
description: Vue d’ensemble des modèles d’intégration de contrat intelligent dans Azure Blockchain Workbench Preview.
ms.date: 11/20/2019
ms.topic: conceptual
ms.reviewer: mmercuri
ms.openlocfilehash: dae63e16356e825d3be31380df1648749e59d8bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015500"
---
# <a name="smart-contract-integration-patterns"></a>Modèles d’intégration de contrat intelligent

Les contrats intelligents représentent généralement un flux de travail métier qui doit être intégré à des systèmes et appareils externes.

Les exigences de ces flux de travail incluent la nécessité d’initier des transactions sur un registre distribué qui contient des données provenant d’un système, service ou appareil externe. Ils requièrent également des systèmes externes qui réagissent à des événements provenant de contrats intelligents sur un registre distribué.

L’intégration de messagerie et d’API REST envoie des transactions à partir de systèmes externes à des contrats intelligents inclus dans une application Azure Blockchain Workbench. Elle envoie également des notifications d’événements aux systèmes externes en fonction des changements qui se produisent dans une application.

Pour les scénarios d’intégration de données, Azure Blockchain Workbench inclut un ensemble de vues de base de données permettant de fusionner une combinaison de données transactionnelles provenant de blockchain et de métadonnées relatives aux applications et aux contrats intelligents.

Dans certains scénarios, tels que ceux liés à la chaîne logistique ou aux médias, l’intégration de documents peut également être nécessaire. Bien qu’Azure Blockchain Workbench ne permette pas d’appeler une API pour gérer les documents directement, des documents peuvent être intégrés dans une application Blockchain. Cette section inclut également ce modèle.

Cette section inclut les modèles identifiés pour l’implémentation de chacun de ces types d’intégration dans vos solutions de bout en bout.

## <a name="rest-api-based-integration"></a>Intégration basée sur l’API REST

Les fonctionnalités de l’application web générée par Azure Blockchain Workbench sont exposées via l’API REST. Les fonctionnalités incluent le téléchargement, la configuration et l’administration Azure Blockchain Workbench des applications, l’envoi de transactions à un registre distribué et l’interrogation des métadonnées de l’application et des données du registre.

L’API REST est principalement utilisée pour les clients interactifs tels que les applications web, mobiles et bot.

Cette section présente les modèles axés sur les API REST qui envoient des transactions à un registre distribué, et les modèles qui interrogent des données sur les transactions dans la base de données *off chain* d’Azure Blockchain Workbench.

### <a name="sending-transactions-to-a-distributed-ledger-from-an-external-system"></a>Envoi de transactions à un registre distribué à partir d’un système externe

L’API REST Azure Blockchain Workbench envoie des requêtes authentifiées afin d’exécuter des transactions sur un registre distribué.

![Envoi de transactions à un registre distribué](./media/integration-patterns/send-transactions-ledger.png)

L’exécution des transactions se produit suivant le processus décrit plus haut, où :

-   L’application externe s’authentifie auprès d’Azure Active Directory configuré dans le cadre du déploiement d’Azure Blockchain Workbench.
-   Les utilisateurs autorisés reçoivent un jeton du porteur qui peut être envoyé avec des requêtes à l’API.
-   Les applications externes appellent l’API REST à l’aide du jeton du porteur.
-   L’API REST prépare la requête sous forme de message et l’envoie au Service Bus. Elle est alors récupérée, signée et envoyée au registre distribué approprié.
-   L’API REST formule une requête vers la base de données SQL Azure Blockchain Workbench afin d’enregistrer la requête et de déterminer l’état d’approvisionnement actuel.
-   La base de données SQL renvoie l’état d’approvisionnement et l’appel d’API renvoie l’ID de l’application externe qui l’a appelée.

### <a name="querying-blockchain-workbench-metadata-and-distributed-ledger-transactions"></a>Interrogation des métadonnées Blockchain Workbench et transactions de registre distribué

L’API REST Azure Blockchain Workbench envoie des requêtes authentifiées afin de rechercher des informations liées à l’exécution de contrat intelligent sur un registre distribué.

![Interrogation des métadonnées](./media/integration-patterns/querying-metadata.png)

L’interrogation se produit suivant le processus décrit plus haut, où :

1. L’application externe s’authentifie auprès d’Azure Active Directory configuré dans le cadre du déploiement d’Azure Blockchain Workbench.
2. Les utilisateurs autorisés reçoivent un jeton du porteur qui peut être envoyé avec des requêtes à l’API.
3. Les applications externes appellent l’API REST à l’aide du jeton du porteur.
4. L’API REST interroge les données pour la requête dans la base de données SQL et les retourne au client.

## <a name="messaging-integration"></a>Intégration d’une messagerie

L’intégration d’une messagerie facilite l’interaction avec les systèmes, services et appareils pour lesquels une connexion interactive n’est pas possible ou n’est pas souhaitable. L’intégration d’une messagerie vise deux types de messages : les messages demandant que des transactions soient exécutées sur un registre distribué, et les événements exposés par ce registre lors de l’exécution de transactions.

L’intégration d’une messagerie vise l’exécution et la surveillance de transactions liées à la création d’un utilisateur, la création d’un contrat et l’exécution de transactions sur des contrats, et est principalement utilisée par des systèmes principaux *décentralisés*.

Cette section présente les modèles axés sur les API basées sur des messages qui envoient des transactions à un registre distribué, et les modèles qui représentent des messages d’événement exposés par le registre distribué sous-jacent.

### <a name="one-way-event-delivery-from-a-smart-contract-to-an-event-consumer"></a>Remise d’événement unidirectionnelle d’un contrat intelligent à un consommateur d’événements 

Dans ce scénario, un événement se produit dans un contrat intelligent, par exemple, un changement d’état ou l’exécution d’un type spécifique de transaction. Cet événement est diffusé via un Event Grid aux consommateurs en aval, et ces consommateurs effectuent alors les actions appropriées.

Un exemple de ce scénario est que, lorsqu’une transaction se produit, un consommateur est averti et peut effectuer une action comme l’enregistrement des informations dans une base de données SQL ou le Common Data Service. Ce scénario correspond au modèle suivi par Workbench pour remplir la base de données SQL *off chain*.

Un autre exemple est celui d’un contrat intelligent passant à un état particulier, par exemple, lorsqu’un contrat devient *OutOfCompliance*. Lorsque ce changement d’état se produit, une alerte peut être déclenchée et envoyée au téléphone mobile d’un administrateur.

![Remise d’événement unidirectionnelle](./media/integration-patterns/one-way-event-delivery.png)

Ce scénario se déroule suivant le processus décrit plus haut, où :

-   Le contrat intelligent passe à un nouvel état et envoie un événement au registre.
-   Le registre reçoit et remet l’événement à Azure Blockchain Workbench.
-   Azure Blockchain Workbench est abonné aux événements du registre et reçoit l’événement.
-   Azure Blockchain Workbench publie l’événement aux abonnés sur l’Event Grid.
-   Les systèmes externes sont abonnés à l’Event Grid, exploitent le message et effectuent les actions appropriées.

## <a name="one-way-event-delivery-of-a-message-from-an-external-system-to-a-smart-contract"></a>Remise d’événement unidirectionnelle d’un message d’un système externe à un contrat intelligent

Il existe également un scénario dans le sens opposé. Dans ce cas, un événement est généré par un capteur ou un système externe et les données de cet événement doivent être envoyées à un contrat intelligent.

Un exemple courant réside dans la remise de données de marchés financiers, par exemple, les prix de marchandises, d’actions ou obligations, à un contrat intelligent.

### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>Remise directe d’un Azure Blockchain Workbench dans le format attendu

Certaines applications sont conçues pour s’intégrer à Azure Blockchain Workbench et génèrent et envoient directement des messages dans les formats attendus.

![Remise directe](./media/integration-patterns/direct-delivery.png)

Cette remise se déroule suivant le processus décrit plus haut, où :

-   Un événement se produit dans un système externe qui entraîne la création d’un message destiné à Azure Blockchain Workbench.
-   Le système externe inclut du code écrit pour créer ce message dans un format connu, et l’envoie directement au Service Bus.
-   Azure Blockchain Workbench est abonné aux événements du Service Bus et récupère le message.
-   Azure Blockchain Workbench appelle le registre, en envoyant des données du système externe à un contrat spécifique.
-   À la réception du message, le contrat passe à un nouvel état.

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>Remise d’un message dans un format inconnu à Azure Blockchain Workbench

Certains systèmes ne peuvent pas être modifiés pour remettre des messages dans les formats standards utilisés par Azure Blockchain Workbench. Dans ces cas, les mécanismes existants et les formats de message de ces systèmes peuvent généralement être utilisés. Plus précisément, les types de messages natifs de ces systèmes peuvent être transformés à l’aide de Logic Apps, d’Azure Functions ou de tout autre code personnalisé pour correspondre à l’un des formats de messagerie standards attendus.

![Format de message inconnu](./media/integration-patterns/unknown-message-format.png)

Cela se produit suivant le processus décrit plus haut, où :

-   Un événement se produit dans un système externe qui entraîne la création d’un message.
-   Une application logique ou un code personnalisé est utilisé pour recevoir ce message et le transformer en un message au format Azure Blockchain Workbench standard.
-   L’application logique envoie le message transformé directement au Service Bus.
-   Azure Blockchain Workbench est abonné aux événements du Service Bus et récupère le message.
-   Azure Blockchain Workbench appelle le registre, en envoyant des données du système externe à une fonction spécifique du contrat.
-   La fonction s’exécute et modifie généralement l’état. Le changement d’état fait avancer le flux de travail métier correspondant dans le contrat intelligent, en activant l’exécution des autres fonctions prévues.

### <a name="transitioning-control-to-an-external-process-and-await-completion"></a>Transition de contrôle vers un processus externe et attente de l’achèvement

Dans certains scénarios, l’exécution d’un contrat intelligent doit être arrêtée et transférée à un processus externe. Ce processus doit alors la terminer, envoyer un message au contrat intelligent, et l’exécution doit se poursuivre dans le contrat intelligent.

#### <a name="transition-to-the-external-process"></a>Transition vers le processus externe

Ce modèle est généralement implémenté selon l’approche suivante :

-   Le contrat intelligent passe à un état spécifique. Dans cet état, aucune ou un nombre limité de fonctions peut être exécuté jusqu’à ce qu’un système externe effectue l’action souhaitée.
-   Le changement d’état est présenté au consommateur en aval sous la forme d’un événement.
-   Le consommateur en aval reçoit l’événement et déclenche l’exécution de code externe.

![Le diagramme montre un changement d’état dans le contrat qui provoque l’accès d’un événement au registre distribué. Blockchain Workbench sélectionne ensuite l’événement et le publie.](./media/integration-patterns/transition-external-process.png)

#### <a name="return-of-control-from-the-smart-contract"></a>Retour de contrôle du contrat intelligent

En fonction des possibilités de personnalisation du système externe, il peut ou non être en mesure de remettre des messages dans l’un des formats standard attendus par Azure Blockchain Workbench. La capacité des systèmes externes à générer l’un de ces messages détermine lequel des deux chemins de retour suivants est utilisé.

##### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>Remise directe d’un Azure Blockchain Workbench dans le format attendu

![Le diagramme montre un message d’API du système externe récupéré par Blockchain Workbench via le Service Bus. Blockchain Workbench envoie ensuite un message en tant que transaction au registre distribué pour le compte de l’agent. Il est transmis au contrat, où elle provoque un changement d’état.](./media/integration-patterns/direct-delivery.png)

Dans ce modèle, la communication avec le contrat et le changement d’état qui s’en suit se déroulent suivant le processus précédent, où :

-   À l’achèvement ou lorsqu’un jalon spécifique de l’exécution du code externe est atteint, un événement est envoyé au Service Bus connecté à Azure Blockchain Workbench.

-   Pour les systèmes qui ne sont pas directement adaptés pour écrire un message conforme aux attentes de l’API, celui-ci est transformé.

-   Le contenu du message est préparé et envoyé à une fonction spécifique du contrat intelligent. Cette remise est effectuée pour le compte de l’utilisateur associé au système externe.

-   La fonction s’exécute et modifie généralement l’état. Le changement d’état fait avancer le flux de travail métier correspondant dans le contrat intelligent, en activant l’exécution des autres fonctions prévues.

### 

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>Remise d’un message dans un format inconnu à Azure Blockchain Workbench

![Format de message inconnu](./media/integration-patterns/unknown-message-format.png)

Dans ce modèle, dans lequel un message au format standard ne peut pas être envoyé directement, la communication avec le contrat et le changement d’état qui s’en suit se déroulent suivant le processus précédent, où :

1.  À l’achèvement ou lorsqu’un jalon spécifique de l’exécution du code externe est atteint, un événement est envoyé au Service Bus connecté à Azure Blockchain Workbench.
2.  Une application logique ou un code personnalisé est utilisé pour recevoir ce message et le transformer en un message au format Azure Blockchain Workbench standard.
3.  L’application logique envoie le message transformé directement au Service Bus.
4.  Azure Blockchain Workbench est abonné aux événements du Service Bus et récupère le message.
5.  Azure Blockchain Workbench appelle le registre, en envoyant des données du système externe à un contrat spécifique.
6. Le contenu du message est préparé et envoyé à une fonction spécifique du contrat intelligent. Cette remise est effectuée pour le compte de l’utilisateur associé au système externe.
7.  La fonction s’exécute et modifie généralement l’état. Le changement d’état fait avancer le flux de travail métier correspondant dans le contrat intelligent, en activant l’exécution des autres fonctions prévues.

## <a name="iot-integration"></a>Intégration d’IoT

Un scénario d’intégration courant consiste à inclure dans un contrat intelligent des données de télémétrie récupérées de capteurs. En fonction des données fournies par les capteurs, les contrats intelligents peuvent effectuer des actions et modifier l’état du contrat.

Par exemple, si un camion de livraison de médicaments voit sa température atteindre 110 degrés, l’efficacité des médicaments peut être affectée et entraîner un problème de santé publique s’il n’est pas détecté et retiré de la chaîne logistique. Si un conducteur pousse son véhicule à 160 kilomètres/heure, les informations du capteur risquent d’entraîner une résiliation du contrat d’assurance par son assureur. S’il s’agit d’un véhicule de location, les données GPS peuvent indiquer quand le conducteur est sorti du périmètre couvert par son contrat de location et entraîner la facturation de pénalités.

Le défi réside dans le fait que ces capteurs peuvent fournir des données en continu et qu’il n’est pas approprié d’envoyer toutes ces données à un contrat intelligent. Une approche classique consiste à limiter le nombre de messages envoyés au blockchain mais à remettre tous les messages à un magasin secondaire. Par exemple, remettre les messages reçus à intervalle fixe uniquement, une fois par heure par exemple, et lorsqu’une valeur est en dehors de la plage convenue pour un contrat intelligent. La vérification des valeurs en dehors des tolérances garantit la bonne réception et exécution des données pertinentes pour la logique métier des contrats. La vérification de la valeur selon un intervalle garantit le bon fonctionnement du capteur. Toutes les données sont envoyées à un magasin de rapports secondaire pour permettre des rapports, des analyses et un Machine Learning plus approfondis. Par exemple, même si des résultats de capteurs GPS ne soient pas nécessaires toutes les minutes pour un contrat intelligent, ils peuvent fournir des données intéressantes à utiliser dans des rapports ou des itinéraires de mappage.

Sur la plateforme Azure, l’intégration à des appareils se fait généralement avec IoT Hub. IoT Hub fournit le routage des messages en fonction du contenu, et permet l’utilisation du type de fonctionnalité décrit plus haut.

![Messages IoT](./media/integration-patterns/iot.png)

Le processus décrit un modèle :

-   Un appareil communique directement ou via une passerelle locale avec IoT Hub.
-   IoT Hub reçoit les messages et les évalue par rapport aux itinéraires établis qui vérifient le contenu du message, par exemple. *Le capteur signale-t-il une température supérieure à 50 degrés ?*
-   L’IoT Hub envoie des messages qui répondent aux critères à un Service Bus défini pour l’itinéraire.
-   Une application logique ou un autre code écoute le Service Bus défini par IoT Hub pour l’itinéraire.
-   L’application logique ou autre code récupère et transforme le message dans un format connu.
-   Le message transformé, désormais dans un format standard, est envoyé au Service Bus pour Azure Blockchain Workbench.
-   Azure Blockchain Workbench est abonné aux événements du Service Bus et récupère le message.
-   Azure Blockchain Workbench appelle le registre, en envoyant des données du système externe à un contrat spécifique.
-   À la réception du message, le contrat évalue les données et peut changer d’état en fonction du résultat de cette évaluation, par exemple, pour une température élevée, passer à l’état *Out of Compliance*.

## <a name="data-integration"></a>Intégration des données

En plus des API REST et basées sur des messages, Azure Blockchain Workbench permet également d’accéder à une base de données SQL remplie avec des métadonnées d’application et de contrat, ainsi que des données transactionnelles de registres distribués.

![Intégration des données](./media/integration-patterns/data-integration.png)

L’intégration de données est bien connue :

-   Selon son comportement normal, Azure Blockchain Workbench stocke des métadonnées sur des applications, flux de travail, contrats et transactions.
-   Les systèmes ou outils externes présentent une ou plusieurs boîtes de dialogue pour faciliter la collecte d’informations sur la base de données, comme le nom du serveur de base de données, le nom de la base de données, le type d’authentification, les informations d’identification de connexion et les vues de base de données à utiliser.
-   Les requêtes sont écrites sur des vues de base de données afin de faciliter la consommation en aval par les systèmes externes, services, rapports, outils de développement et outils de productivité d’entreprise.

## <a name="storage-integration"></a>Intégration d’un stockage

Plusieurs scénarios peuvent nécessiter l’intégration de fichiers attestables. Pour diverses raisons, il n’est pas approprié de placer des fichiers sur un blockchain. Au lieu de cela, vous pouvez adopter l’approche courante qui consiste à effectuer un hachage cryptographique (par exemple, SHA-256) sur un fichier et à partager ce hachage sur un registre distribué. Une réexécution du hachage à tout moment doit retourner le même résultat. Si le fichier est modifié, même s’il ne s’agit que d’un seul pixel dans une image, le hachage retourne une valeur différente.

![Intégration d’un stockage](./media/integration-patterns/storage-integration.png)

Le modèle peut être implémenté, dans lequel :

-   Un système externe conserve un fichier dans un mécanisme de stockage tel que le Stockage Azure.
-   Un hachage est généré avec le fichier, ou le fichier et les métadonnées associées, comme un identificateur du propriétaire, l’URL où se trouve le fichier, etc.
-   Le hachage et les métadonnées sont envoyés à une fonction d’un contrat intelligent comme *FileAdded*
-   Ultérieurement, le fichier et les métadonnées peuvent être de nouveau hachés et comparés aux valeurs stockées dans le registre.

## <a name="prerequisites-for-implementing-integration-patterns-using-the-rest-and-message-apis"></a>Conditions préalables à l’implémentation de modèles d’intégration à l’aide des API REST et de messages

Les éléments suivants sont nécessaires pour faciliter l’interaction d’un système ou appareil externe avec le contrat intelligent via l’API REST ou de messages :

1. Dans Azure Active Directory pour le consortium, un compte représentant le système ou appareil externe est créé.
2. Des fonctions sont définies dans un ou plusieurs contrats intelligents appropriés pour votre application Azure Blockchain Workbench afin d’accepter les événements de votre système ou appareil externe.
3. Le fichier de configuration de l’application de votre contrat intelligent inclut le rôle qui est affecté au système ou à l’appareil.
4. Le fichier de configuration de l’application de votre contrat intelligent identifie dans quels états cette fonction est appelée par le rôle défini.
5. Le fichier de configuration de l’application et ses contrats intelligents sont chargés dans Azure Blockchain Workbench.

Une fois que l’application est chargée, le compte Azure Active Directory du système externe est affecté au contrat et au rôle associé.

## <a name="testing-external-system-integration-flows-prior-to-writing-integration-code"></a>Test des flux d’intégration de système externe avant d’écrire le code d’intégration 

L’intégration à des systèmes externes est une condition essentielle de nombreux scénarios. Il est préférable de pouvoir valider la conception d’un contrat intelligent avant ou parallèlement au développement du code d’intégration à des systèmes externes.

L’utilisation d’Azure Active Directory (Azure AD) peut optimiser considérablement de la productivité du développeur et le retour sur investissement. Plus précisément, la durée d’intégration de code à un système externe peut être énorme. En utilisant Azure AD et la génération automatique de l’expérience utilisateur par Azure Blockchain Workbench, vous pouvez permettre aux développeurs de se connecter à Blockchain Workbench en tant que système externe et de renseigner les valeurs du système externe par le biais de l’expérience utilisateur. Vous pouvez rapidement développer et valider des idées dans un environnement de preuve de concept avant que le code d’intégration soit écrit pour les systèmes externes.
