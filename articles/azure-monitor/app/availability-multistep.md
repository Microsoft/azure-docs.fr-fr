---
title: Surveiller à l’aide de tests web à plusieurs étapes – Azure Application Insights
description: Configuration des tests web multiétape pour surveiller vos applications web avec Azure Application Insights
ms.topic: conceptual
ms.date: 05/26/2020
ms.openlocfilehash: 2df8b7450423c901665090608da83f68b43b30e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87024771"
---
# <a name="multi-step-web-tests"></a>Tests web à plusieurs étapes

Vous pouvez surveiller une séquence enregistrée d’URL et d’interactions avec un site web via des tests web à plusieurs étapes. Cet article vous guidera tout au long du processus de création d’un test web à plusieurs étapes avec Visual Studio Enterprise.

> [!NOTE]
> Les tests web à plusieurs étapes dépendent des fichiers de test web Visual Studio. Il a été [annoncé](https://devblogs.microsoft.com/devops/cloud-based-load-testing-service-eol/) que Visual Studio 2019 est la dernière version avec la fonctionnalité de test web. Il est important de comprendre que même si aucune nouvelle fonctionnalité ne sera ajoutée, les fonctionnalités de test web dans Visual Studio 2019 sont toujours prises en charge et continueront d’être prises en charge pendant le cycle de vie du support du produit. L’équipe de produit Azure Monitor a répondu aux questions concernant l’avenir des tests de disponibilité à plusieurs étapes [ici](https://github.com/MicrosoftDocs/azure-docs/issues/26050#issuecomment-468814101).  
> </br>
> Les tests web comportant plusieurs étapes **ne sont pas pris en charge** dans le cloud [Azure Government](../../azure-government/index.yml).


## <a name="pre-requisites"></a>Conditions préalables

* Visual Studio 2017 Enterprise ou version supérieure.
* Outils de test de charge et de performances web Visual Studio.

Pour localiser les outils requis. Lancez **Visual Studio Installer** > **Composants individuels** > **Débogage et tests** > **Outils de test des performances web et de test de charge**.

![Capture d’écran de l’interface utilisateur Visual Studio Installer avec Composants individuels sélectionné avec une case à cocher en regard de l’élément pour Outils de test des performances web et de test de charge](./media/availability-multistep/web-performance-load-testing.png)

> [!NOTE]
> Les tests web à plusieurs étapes impliquent des coûts supplémentaires. Pour en savoir plus, consultez le [guide de tarification officiel](https://azure.microsoft.com/pricing/details/application-insights/).

## <a name="record-a-multi-step-web-test"></a>Enregistrer un test web à plusieurs étapes 

> [!WARNING]
> Nous ne recommandons plus l’utilisation de l’enregistreur multiétape. L’enregistreur a été développé pour les pages HTML statiques avec des interactions de base et ne fournit pas d’expérience fonctionnelle pour les pages web modernes.

Pour obtenir de l’aide sur la création de tests web Visual Studio, voir la [documentation officielle de Visual Studio 2019](/visualstudio/test/how-to-create-a-web-service-test?view=vs-2019).

## <a name="upload-the-web-test"></a>Charger le test web

1. Dans le portail Application Insights, sur le volet Disponibilité, sélectionnez **Créer un test** > **Type de test** > **Test web multiétape**.

2. Définissez les emplacements de test, la fréquence et les paramètres d’alerte.

### <a name="frequency--location"></a>Fréquence et emplacement

|Paramètre| Explication
|----|----|----|
|**Fréquence de test**| définit la fréquence selon laquelle le test est exécuté à partir de chaque emplacement de test. Avec, par défaut, une fréquence de cinq minutes et cinq emplacements de test, votre site sera testé en moyenne une fois par minute.|
|**Emplacements du test**| Ce sont les lieux d’où nos serveurs envoient des requêtes web à votre URL. **Nous recommandons d’utiliser au moins cinq emplacements de test**, afin de pouvoir faire la distinction entre les problèmes propres à votre site web et les problèmes de réseau. Vous pouvez sélectionner jusqu’à 16 emplacements.

### <a name="success-criteria"></a>Critères de réussite

|Paramètre| Explication
|----|----|----|
| **Délai d’expiration du test** |diminuez cette valeur pour être averti des réponses lentes. Le test est compté comme une erreur si des réponses de votre site n’ont pas été reçues pendant cette période. Si vous avez sélectionné **Analyser les demandes dépendantes**, l’ensemble des images, fichiers de style, scripts et autres ressources dépendantes ont dû être reçus pendant cette période.|
| **Réponse HTTP** | le code d’état retourné est comptabilisé comme un succès. 200 est le code qui indique qu’une page web normale a été retournée.|
| **Correspondance du contenu** | Chaîne telle que « Bienvenue ! » Nous vérifions qu’une correspondance exacte respectant la casse est présente dans chaque réponse. Il doit s'agir d'une chaîne standard sans caractère générique. N'oubliez pas que si votre contenu change, vous devrez peut-être l'actualiser. **La correspondance de contenu est prise en charge uniquement pour les caractères anglais** |

### <a name="alerts"></a>Alertes

|Paramètre| Explication
|----|----|----|
|**Quasi-temps réel (préversion)** | Nous vous conseillons d’utiliser les alertes en quasi-temps réel. La configuration de ce type d’alerte s’effectue après avoir créé votre test de disponibilité.  |
|**Classique** | Nous déconseillons d’utiliser les alertes classiques pour les nouveaux tests de disponibilité.|
|**Seuil d’emplacement de l’alerte**|nous recommandons un minimum de 3 à 5 emplacements. La relation optimale entre le seuil d’emplacement de l’alerte et le nombre d’emplacements de test est **seuil d’emplacement de l’alerte** = **nombre d’emplacements de test - 2, avec un minimum de cinq emplacements de test.**|

## <a name="configuration"></a>Configuration

### <a name="plugging-time-and-random-numbers-into-your-test"></a>Ajout de plug-ins de temps et de nombres aléatoires à votre test

Supposons que vous testiez un outil qui obtient des données temporelles, telles que des actions à partir d’un flux externe. Lorsque vous enregistrez votre test web, vous devez utiliser des heures spécifiques, mais vous les définissez en tant que paramètres de test, à savoir StartTime et EndTime.

![Capture d’écran de ma superbe application de stock](./media/availability-multistep/app-insights-72webtest-parameters.png)

Lorsque vous exécutez le test, vous souhaitez que le paramètre EndTime corresponde systématiquement à l’heure actuelle et le paramètre StartTime à l’heure d’il y a 15 minutes.

Le plug-in Date Time du test web offre un moyen de traitement des heures paramétrables.

1. Ajoutez un plug-in de test web pour chaque valeur de paramètre variable souhaitée. Dans la barre d’outils de test web, sélectionnez **Ajouter un plug-in de test web**.
    
    ![Ajouter un plug-in de test web](./media/availability-multistep/app-insights-72webtest-plugin-name.png)
    
    Dans cet exemple, nous allons utiliser deux instances du plug-in Date Time. Une instance est pour « il y a 15 minutes » et l’autre pour « maintenant ».

2. Ouvrez les propriétés de chaque plug-in. Donnez-lui un nom et configurez-le de manière à utiliser l’heure actuelle. Pour l'un d'eux, définissez Ajouter des minutes = -15.

    ![Paramètres de contexte](./media/availability-multistep/app-insights-72webtest-plugin-parameters.png)

3. Dans les paramètres de test web, utilisez {{nom du plug-in}} pour référencer un nom de plug-in.

    ![StartTime](./media/availability-multistep/app-insights-72webtest-plugins.png)

Maintenant, téléchargez votre test sur le portail. Il va utiliser les valeurs dynamiques à chaque exécution du test.

### <a name="dealing-with-sign-in"></a>Gestion de la connexion

Si vos utilisateurs se connectent à votre application, vous disposez de différentes options pour simuler la connexion et tester les pages suivant la connexion. L’approche que vous utilisez dépend du type de sécurité fourni par l’application.

Dans tous les cas, vous devez créer un compte dans votre application uniquement à des fins de test. Si possible, limitez les autorisations de ce compte de test afin que les tests web n’affectent aucunement les utilisateurs réels.

**Nom d’utilisateur et mot de passe simples** Enregistrez un test web comme à l’accoutumée. Supprimez d’abord les cookies

**Authentification SAML**

|Nom de la propriété| Description|
|----|-----|
| URI d’audience | URI d’audience pour le jeton SAML.  Il s’agit de l’URI du service Access Control Service (ACS), laquelle inclut l’espace de noms et le nom d’hôte ACS. |
| Mot de passe du certificat | Mot de passe du certificat client qui accorde l’accès à la clé privée incorporée. |
| Certificat client  | Valeur du certificat client avec la clé privée au format d’encodage Base64. |
| Identificateur de nom | Identificateur de nom du jeton. |
| Pas après | Intervalle de temps pendant lequel le jeton sera valide.  La valeur par défaut est 5 minutes. |
| Pas avant | Intervalle de temps pendant lequel un jeton créé dans le passé sera valide (pour traiter les décalages de temps).  La valeur par défaut est 5 minutes (négative). |
| Nom du paramètre de contexte cible | Paramètre de contexte qui recevra l’assertion générée. |


**Clé secrète client** Si votre application présente un mode de connexion impliquant une clé secrète client, utilisez ce mode. Azure Active Directory (AAD) est un exemple de service fournissant une connexion avec clé secrète client. Dans AAD, la clé secrète client est la clé d’application.

Voici un exemple de test web d’une application web Azure à l’aide d’une clé d’application :

![Capture d’écran de l’exemple](./media/availability-multistep/client-secret.png)

Récupérez le jeton d’ADD à l’aide de la clé secrète client (clé d’application).
Extrayez le jeton porteur de la réponse.
Appelez l’API à l’aide du jeton porteur de l’en-tête d’autorisation.
Assurez-vous que le test web est un client réel, c’est-à-dire, qu’il possède sa propre application dans AAD, puis utilisez son ID client et sa clé d’application. Votre service soumis à un test possède également sa propre application dans AAD : l’URI ID d’application se retrouve dans le champ dédié aux ressources du test web.

### <a name="open-authentication"></a>Authentification ouverte
Comme exemple d’authentification ouverte, citons la connexion avec votre compte Microsoft ou Google. De nombreuses applications utilisant OAuth fournissent l’alternative de la clé secrète client ; commencez donc par rechercher cet élément.

Si votre test doit se connecter à l’aide d’OAuth, l’approche générale est la suivante :

Utilisez un outil tel que Fiddler pour examiner le trafic entre votre navigateur web, le site d’authentification et votre application.
Effectuez deux connexions ou plus à l’aide d’ordinateurs ou de navigateurs différents, ou à des intervalles longs (pour que les jetons arrivent à expiration).
En comparant les différentes sessions, identifiez le jeton retransmis à partir du site d’authentification, qui est ensuite transmis à votre serveur d’application après la connexion.
Enregistrez un test web à l’aide de Visual Studio.
Paramétrez les jetons, en définissant le paramètre lorsque le jeton est retourné par l’authentificateur et en l’utilisant dans la requête soumise sur le site. (Visual Studio tente de paramétrer le test, mais ne paramètre pas correctement les jetons.)

## <a name="troubleshooting"></a>Dépannage

Consultez l’[article dédié au dépannage](troubleshoot-availability.md).

## <a name="next-steps"></a>Étapes suivantes

* [Availability alerts](availability-alerts.md) (Alertes de disponibilité)
* [Surveiller la disponibilité et de la réactivité d’un site web](monitor-web-app-availability.md)
