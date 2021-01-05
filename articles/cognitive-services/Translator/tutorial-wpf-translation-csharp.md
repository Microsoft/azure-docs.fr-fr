---
title: 'Tutoriel : Créer une application de traduction avec WPF, C# - Translator'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous créez une application WPF qui traduit le texte, détecte la langue et corrige l’orthographe avec une seule clé d’abonnement.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 05/26/2020
ms.author: swmachan
ms.custom: devx-track-csharp
ms.openlocfilehash: d239b89aaf0bc140916d38583f4263f7bf660f1a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023614"
---
# <a name="tutorial-create-a-translation-app-with-wpf"></a>Tutoriel : Créer une application de traduction avec WPF

Dans ce tutoriel, vous créez une application [Windows Presentation Foundation (WPF)](/visualstudio/designers/getting-started-with-wpf?view=vs-2019) qui utilise les API Cognitive Services pour la traduction de texte, la détection de langue et la correction orthographique avec une seule clé d’abonnement. Plus précisément, votre application appelle Translator et [Vérification orthographique Bing](https://azure.microsoft.com/services/cognitive-services/spell-check/).

Qu’est-ce que WPF ? Il s’agit d’un framework d’interface utilisateur qui crée des applications de bureau clientes. La plateforme de développement WPF prend en charge un large éventail de fonctionnalités de développement d’application, notamment : modèle d’application, ressources, contrôles, graphismes, disposition, liaison de données, documents et sécurité. C’est un sous-ensemble du .NET Framework, par conséquent, si vous avez déjà créé des applications avec le .NET Framework à l’aide d’ASP.NET ou de Windows Forms, l’expérience de programmation doit être familière. WPF utilise le langage XAML (Extensible Application Markup Language) dans son modèle déclaratif pour la programmation d’application, que nous allons aborder dans les sections suivantes.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un projet WPF dans Visual Studio
> * Ajouter des assemblys et des packages NuGet à votre projet
> * Créer l’interface utilisateur de votre application avec XAML
> * Utiliser Translator pour obtenir les langues, traduire du texte et détecter la langue source
> * Utiliser l’API Vérification orthographique Bing pour valider votre entrée et améliorer l’exactitude de la traduction
> * Exécuter votre application WPF

### <a name="cognitive-services-used-in-this-tutorial"></a>Services Cognitives Services utilisés dans ce tutoriel

Cette liste inclut les services Cognitive Services utilisés dans ce tutoriel. Suivez le lien pour accéder à la référence d’API de chaque fonctionnalité.

| Service | Fonctionnalité | Description |
|---------|---------|-------------|
| Convertisseur | [Obtenir les langues](./reference/v3-0-languages.md) | Récupérez la liste complète des langues prises en charge pour la traduction de texte. |
| Convertisseur | [Translate](./reference/v3-0-translate.md) | Traduisez du texte dans plus de 70 langues. |
| Convertisseur | [Detect](./reference/v3-0-detect.md) | Détectez la langue du texte d’entrée. Inclut le score de confiance pour la détection. |
| Vérification orthographique Bing | [Vérification orthographique](/rest/api/cognitiveservices/bing-spell-check-api-v7-reference) | Corrigez les fautes d’orthographe pour améliorer l’exactitude de la traduction. |

## <a name="prerequisites"></a>Prérequis

Avant de continuer, vous avez besoin des éléments suivants :

* Un abonnement Azure Cognitive Services. [Obtenez une clé Cognitive Services](../cognitive-services-apis-create-account.md#create-a-new-azure-cognitive-services-resource).
* Une machine Windows
* [Visual Studio 2019](https://www.visualstudio.com/downloads/) - Community ou Enterprise

> [!NOTE]
> Nous vous recommandons de créer l’abonnement dans la région USA Ouest pour ce tutoriel. Sinon, vous devez changer les points de terminaison et les régions dans le code au cours de cet exercice.  

## <a name="create-a-wpf-app-in-visual-studio"></a>Créer une application WPF dans Visual Studio

La première chose à faire est de configurer notre projet dans Visual Studio.

1. Ouvrez Visual Studio. Sélectionnez **Créer un projet**.
1. Dans **Créer un projet**, recherchez et sélectionnez **Application WPF (.NET Framework)** . Vous pouvez sélectionner C# dans **Langage** pour affiner les options.
1. Sélectionnez **Suivant**, puis nommez votre projet `MSTranslatorDemo`.
1. Définissez la version du framework à **.NET Framework 4.7.2** ou version ultérieure, puis cliquez sur **Créer**.
   ![Entrez le nom et la version du framework dans Visual Studio](media/name-wpf-project-visual-studio.png)

Votre projet a été créé. Notez que deux onglets sont ouverts : `MainWindow.xaml` et `MainWindow.xaml.cs`. Tout au long de ce tutoriel, nous allons ajouter du code à ces deux fichiers. Nous allons modifier `MainWindow.xaml` pour l’interface utilisateur de l’application. Nous allons modifier `MainWindow.xaml.cs` pour les appels Translator et Vérification orthographique Bing.
   ![Examiner votre environnement](media/blank-wpf-project.png)

Dans la section suivante, nous ajoutons des assemblys et un package NuGet à notre projet pour avoir des fonctionnalités supplémentaires, comme l’analyse JSON.

## <a name="add-references-and-nuget-packages-to-your-project"></a>Ajouter des références et des packages NuGet à votre projet

Notre projet nécessite un certain nombre d’assemblys .NET Framework et NewtonSoft.Json, que nous installons à l’aide du Gestionnaire de package NuGet.

### <a name="add-net-framework-assemblies"></a>Ajouter des assemblys .NET Framework

Nous ajoutons des assemblys à notre projet pour sérialiser et désérialiser des objets, et pour gérer les demandes et les réponses HTTP.

1. Recherchez votre projet dans l’Explorateur de solutions de Visual Studio. Cliquez avec le bouton droit sur votre projet, puis sélectionnez **Ajouter > Référence**, qui ouvre le **Gestionnaire de références**.
1. L’onglet des **assemblys** liste tous les assemblys .NET Framework que vous pouvez référencer. Utilisez la barre de recherche dans le coin supérieur droit pour rechercher des références.
   ![Ajouter des références d’assembly](media/add-assemblies-2019.png)
1. Sélectionnez les références suivantes pour votre projet :
   * [System.Runtime.Serialization](/dotnet/api/system.runtime.serialization)
   * [System.Web](/dotnet/api/system.web)
   * System.Web.Extensions
   * [System.Windows](/dotnet/api/system.windows)
1. Une fois que vous avez ajouté ces références à votre projet, vous pouvez cliquer sur **OK** pour fermer le **Gestionnaire de références**.

> [!NOTE]
> Pour en savoir plus sur les références d’assembly, consultez [Guide pratique pour ajouter ou supprimer une référence à l’aide du Gestionnaire de références](/visualstudio/ide/how-to-add-or-remove-references-by-using-the-reference-manager?view=vs-2019).

### <a name="install-newtonsoftjson"></a>Installer Newtonsoft.Json

Notre application utilise NewtonSoft.Json pour désérialiser des objets JSON. Suivez ces instructions pour installer le package.

1. Recherchez votre projet dans l’Explorateur de solutions de Visual Studio et cliquez dessus avec le bouton droit. Sélectionnez **Gérer les packages NuGet**.
1. Recherchez et sélectionnez l’onglet **Parcourir**.
1. Entrez [NewtonSoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) dans la barre de recherche.

    ![Rechercher et installer Newtonsoft.Json](media/nuget-package-manager.png)

1. Sélectionnez le package et cliquez sur **Installer**.
1. Une fois l’installation effectuée, fermez l’onglet.

## <a name="create-a-wpf-form-using-xaml"></a>Créer un formulaire WPF à l’aide de XAML

Pour utiliser votre application, vous avez besoin d’une interface utilisateur. À l’aide de XAML, nous créons un formulaire qui permet aux utilisateurs de sélectionner les langues d’entrée et de traduction, d’entrer le texte à traduire et d’afficher la sortie de la traduction.

Voyons ce que nous créons.

![Interface utilisateur XAML WPF](media/translator-text-csharp-xaml.png)

L’interface utilisateur comprend ces composants :

| Nom | Type | Description |
|------|------|-------------|
| `FromLanguageComboBox` | Liste déroulante | Affiche la liste des langues prises en charge par Microsoft Translator pour la traduction de texte. L’utilisateur sélectionne la langue de son texte d’origine. |
| `ToLanguageComboBox` | Liste déroulante | Affiche la même liste de langues que `FromComboBox`, mais permet de sélectionner la langue cible du texte à traduire. |
| `TextToTranslate` | TextBox | Permet à l’utilisateur d’entrer du texte à traduire. |
| `TranslateButton` | Bouton | Utilisez ce bouton pour traduire du texte. |
| `TranslatedTextLabel` | Étiquette | Affiche la traduction. |
| `DetectedLanguageLabel` | Étiquette | Affiche la langue détectée du texte à traduire (`TextToTranslate`). |

> [!NOTE]
> Nous créons ce formulaire à l’aide de code source XAML, toutefois, vous pouvez créer le formulaire avec l’éditeur de Visual Studio.

Nous ajoutons le code à notre projet.

1. Dans Visual Studio, sélectionnez l’onglet `MainWindow.xaml`.
1. Copiez ce code dans votre projet, puis sélectionnez **Fichier > Enregistrer MainWindow.xaml** pour enregistrer vos modifications.
   ```xaml
   <Window x:Class="MSTranslatorDemo.MainWindow"
           xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
           xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
           xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
           xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
           xmlns:local="clr-namespace:MSTranslatorDemo"
           mc:Ignorable="d"
           Title="Microsoft Translator" Height="400" Width="700" BorderThickness="0">
       <Grid>
           <Label x:Name="label" Content="Microsoft Translator" HorizontalAlignment="Left" Margin="39,6,0,0" VerticalAlignment="Top" Height="49" FontSize="26.667"/>
           <TextBox x:Name="TextToTranslate" HorizontalAlignment="Left" Height="23" Margin="42,160,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="600" FontSize="14" TabIndex="3"/>
           <Label x:Name="EnterTextLabel" Content="Text to translate:" HorizontalAlignment="Left" Margin="40,129,0,0" VerticalAlignment="Top" FontSize="14"/>
           <Label x:Name="toLabel" Content="Translate to:" HorizontalAlignment="Left" Margin="304,58,0,0" VerticalAlignment="Top" FontSize="14"/>

           <Button x:Name="TranslateButton" Content="Translate" HorizontalAlignment="Left" Margin="39,206,0,0" VerticalAlignment="Top" Width="114" Height="31" Click="TranslateButton_Click" FontSize="14" TabIndex="4" IsDefault="True"/>
           <ComboBox x:Name="ToLanguageComboBox"
                   HorizontalAlignment="Left"
                   Margin="306,88,0,0"
                   VerticalAlignment="Top"
                   Width="175" FontSize="14" TabIndex="2">

           </ComboBox>
           <Label x:Name="fromLabel" Content="Translate from:" HorizontalAlignment="Left" Margin="40,58,0,0" VerticalAlignment="Top" FontSize="14"/>
           <ComboBox x:Name="FromLanguageComboBox"
               HorizontalAlignment="Left"
               Margin="42,88,0,0"
               VerticalAlignment="Top"
               Width="175" FontSize="14" TabIndex="1"/>
           <Label x:Name="TranslatedTextLabel" Content="Translation is displayed here." HorizontalAlignment="Left" Margin="39,255,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="85" BorderThickness="0"/>
           <Label x:Name="DetectedLanguageLabel" Content="Autodetected language is displayed here." HorizontalAlignment="Left" Margin="39,288,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="84" BorderThickness="0"/>
       </Grid>
   </Window>
   ```
Vous devez maintenant voir un aperçu de l’interface utilisateur de l’application dans Visual Studio. Le résultat doit être semblable à l’image ci-dessus.

C’est tout, votre formulaire est prêt. Nous écrivons maintenant du code pour utiliser la traduction de texte et la vérification orthographique Bing.

> [!NOTE]
> Vous pouvez modifier ce formulaire ou créer le vôtre.

## <a name="create-your-app"></a>Créer votre application

`MainWindow.xaml.cs` contient le code qui contrôle notre application. Dans les sections suivantes, nous ajoutons du code pour remplir nos menus déroulants et appeler un certain nombre d’API exposées par Translator et Vérification orthographique Bing.

* Au démarrage du programme et quand `MainWindow` est instancié, la méthode `Languages` de Translator est appelée pour récupérer et remplir nos listes déroulantes de sélection de langues. Cette tâche est effectuée une seule fois au début de chaque session.
* Quand vous cliquez sur le bouton **Traduire**, la sélection de langue et le texte de l’utilisateur sont récupérés, la vérification orthographique est effectuée sur l’entrée, et la langue de traduction et celle détectée sont indiquées à l’utilisateur.
  * La méthode `Translate` de Translator est appelée pour traduire le texte à partir de `TextToTranslate`. Cet appel inclut également les langues `to` et `from` sélectionnées à l’aide des menus déroulants.
  * La méthode `Detect` de Translator est appelée pour déterminer la langue du texte de `TextToTranslate`.
  * La Vérification orthographique Bing est utilisée pour valider `TextToTranslate` et corriger les fautes d’orthographe.

Tout notre projet est encapsulé dans la classe `MainWindow : Window`. Nous commençons par ajouter du code pour définir votre clé d’abonnement, déclarer les points de terminaison pour Translator et Vérification orthographique Bing, et initialiser l’application.

1. Dans Visual Studio, sélectionnez l’onglet `MainWindow.xaml.cs`.
1. Remplacez les instructions `using` préremplies par ce qui suit.  
   ```csharp
   using System;
   using System.Windows;
   using System.Net;
   using System.Net.Http;
   using System.IO;
   using System.Collections.Generic;
   using System.Linq;
   using System.Text;
   using Newtonsoft.Json;
   ```
1. Recherchez la classe `MainWindow : Window` et remplacez-la par ce code :
   ```csharp
   {
       // This sample uses the Cognitive Services subscription key for all services. To learn more about
       // authentication options, see: https://docs.microsoft.com/azure/cognitive-services/authentication.
       const string COGNITIVE_SERVICES_KEY = "YOUR_COG_SERVICES_KEY";
       // Endpoints for Translator and Bing Spell Check
       public static readonly string TEXT_TRANSLATION_API_ENDPOINT = "https://api.cognitive.microsofttranslator.com/{0}?api-version=3.0";
       const string BING_SPELL_CHECK_API_ENDPOINT = "https://westus.api.cognitive.microsoft.com/bing/v7.0/spellcheck/";
       // An array of language codes
       private string[] languageCodes;

       // Dictionary to map language codes from friendly name (sorted case-insensitively on language name)
       private SortedDictionary<string, string> languageCodesAndTitles =
           new SortedDictionary<string, string>(Comparer<string>.Create((a, b) => string.Compare(a, b, true)));

       // Global exception handler to display error message and exit
       private static void HandleExceptions(object sender, UnhandledExceptionEventArgs args)
       {
           Exception e = (Exception)args.ExceptionObject;
           MessageBox.Show("Caught " + e.Message, "Error", MessageBoxButton.OK, MessageBoxImage.Error);
           System.Windows.Application.Current.Shutdown();
       }
       // MainWindow constructor
       public MainWindow()
       {
           // Display a message if unexpected error is encountered
           AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(HandleExceptions);

           if (COGNITIVE_SERVICES_KEY.Length != 32)
           {
               MessageBox.Show("One or more invalid API subscription keys.\n\n" +
                   "Put your keys in the *_API_SUBSCRIPTION_KEY variables in MainWindow.xaml.cs.",
                   "Invalid Subscription Key(s)", MessageBoxButton.OK, MessageBoxImage.Error);
               System.Windows.Application.Current.Shutdown();
           }
           else
           {
               // Start GUI
               InitializeComponent();
               // Get languages for drop-downs
               GetLanguagesForTranslate();
               // Populate drop-downs with values from GetLanguagesForTranslate
               PopulateLanguageMenus();
           }
       }
   // NOTE:
   // In the following sections, we'll add code below this.
   }
   ```
1. Ajoutez votre clé d’abonnement Cognitive Services et enregistrez.

Dans ce bloc de code, nous avons déclaré deux variables membres qui contiennent des informations sur les langues disponibles pour la traduction :

| Variable | Type | Description |
|----------|------|-------------|
|`languageCodes` | Tableau de chaînes |Met en cache les codes de langue. Le service Translator utilise des codes courts, tel que `en` pour l’anglais, pour identifier les langues. |
|`languageCodesAndTitles` | Dictionnaire trié | Mappe les noms « conviviaux » dans l’interface utilisateur aux codes courts utilisés dans l’API. Reste trié par ordre alphabétique sans respecter la casse. |

Ensuite, dans le constructeur `MainWindow`, nous avons ajouté la gestion des erreurs avec `HandleExceptions`. Cela garantit qu’une alerte est lancée si une exception n’est pas gérée. Ensuite, une vérification est exécutée pour confirmer que la clé d’abonnement fournie a 32 caractères. Une erreur est levée si la clé a plus ou moins de 32 caractères.

Si nous avons des clés qui ont la bonne longueur, l’appel `InitializeComponent()` propage l’interface utilisateur en recherchant, chargeant et instanciant la description XAML de la fenêtre principale de l’application.

Enfin, nous avons ajouté du code pour appeler des méthodes afin de récupérer les langues de traduction et remplir les menus déroulants de l’interface utilisateur de notre application. Ne vous inquiétez pas, nous allons étudier le code-behind de ces appels bientôt.

## <a name="get-supported-languages"></a>Obtenir les langues prises en charge

Translator prend actuellement en charge plus de 70 langues. Comme de nouvelles langues seront prises en charge au fil du temps, nous vous recommandons d’appeler la ressource Languages exposée par Translator au lieu de coder en dur la liste des langues dans votre application.

Dans cette section, nous envoyons une demande `GET` à la ressource Languages, en spécifiant que nous voulons la liste des langues disponibles pour la traduction.

> [!NOTE]
> La ressource Languages vous permet de filtrer la prise en charge linguistique avec les paramètres de requête suivants : transliteration, dictionary et translation. Pour plus d’informations, consultez les [informations de référence sur l’API](./reference/v3-0-languages.md).

Avant d’aller plus loin, examinons l’exemple de sortie d’un appel à la ressource Languages :

```json
{
  "translation": {
    "af": {
      "name": "Afrikaans",
      "nativeName": "Afrikaans",
      "dir": "ltr"
    },
    "ar": {
      "name": "Arabic",
      "nativeName": "العربية",
      "dir": "rtl"
    }
    // Additional languages are provided in the full JSON output.
}
```

Dans cette sortie, nous pouvons extraire le code de langue et le `name` d’une langue spécifique. Notre application utilise NewtonSoft.Json pour désérialiser l’objet JSON ([`JsonConvert.DeserializeObject`](https://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_JsonConvert_DeserializeObject__1.htm)).

Reprenons là où nous en étions dans la dernière section et ajoutons à notre application une méthode pour obtenir les langues prises en charge.

1. Dans Visual Studio, ouvrez l’onglet `MainWindow.xaml.cs`.
2. Ajoutez ce code à votre projet :
   ```csharp
   // ***** GET TRANSLATABLE LANGUAGE CODES
   private void GetLanguagesForTranslate()
   {
       // Send request to get supported language codes
       string uri = String.Format(TEXT_TRANSLATION_API_ENDPOINT, "languages") + "&scope=translation";
       WebRequest WebRequest = WebRequest.Create(uri);
       WebRequest.Headers.Add("Accept-Language", "en");
       WebResponse response = null;
       // Read and parse the JSON response
       response = WebRequest.GetResponse();
       using (var reader = new StreamReader(response.GetResponseStream(), UnicodeEncoding.UTF8))
       {
           var result = JsonConvert.DeserializeObject<Dictionary<string, Dictionary<string, Dictionary<string, string>>>>(reader.ReadToEnd());
           var languages = result["translation"];

           languageCodes = languages.Keys.ToArray();
           foreach (var kv in languages)
           {
               languageCodesAndTitles.Add(kv.Value["name"], kv.Key);
           }
       }
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

La méthode `GetLanguagesForTranslate()` crée une demande HTTP GET et utilise le paramètre de chaîne de requête `scope=translation` pour limiter l’étendue de la demande aux langues prises en charge pour la traduction. L’en-tête `Accept-Language` avec la valeur `en` est ajouté pour renvoyer les langues prises en charge en anglais.

La réponse JSON est analysée et convertie en dictionnaire. Ensuite, nous ajoutons les codes de langue à la variable membre `languageCodes`. Nous exécutons ensuite une boucle sur les paires clé/valeur contenant les codes de langue et les noms de langue conviviaux et nous les ajoutons à la variable membre `languageCodesAndTitles`. Les menus déroulants dans le formulaire affichent les noms conviviaux, mais nous avons besoin des codes pour demander la traduction.

## <a name="populate-language-drop-down-menus"></a>Remplir les menus déroulants de langues

L’interface utilisateur est définie en XAML, par conséquent, vous n’avez pas grand-chose à faire pour la configurer en dehors de l’appel à `InitializeComponent()`. La seule chose à faire est d’ajouter les noms de langue conviviaux aux menus déroulants **Langue source** et **Langue cible**. La méthode `PopulateLanguageMenus()` ajoute les noms.

1. Dans Visual Studio, ouvrez l’onglet `MainWindow.xaml.cs`.
2. Ajoutez ce code à votre projet sous la méthode `GetLanguagesForTranslate()` :
   ```csharp
   private void PopulateLanguageMenus()
   {
       // Add option to automatically detect the source language
       FromLanguageComboBox.Items.Add("Detect");

       int count = languageCodesAndTitles.Count;
       foreach (string menuItem in languageCodesAndTitles.Keys)
       {
           FromLanguageComboBox.Items.Add(menuItem);
           ToLanguageComboBox.Items.Add(menuItem);
       }

       // Set default languages
       FromLanguageComboBox.SelectedItem = "Detect";
       ToLanguageComboBox.SelectedItem = "English";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Cette méthode effectue une itération sur le dictionnaire `languageCodesAndTitles` et ajoute chaque clé aux deux menus. Une fois les menus remplis, les langues source et cible par défaut sont définies sur **Detect** et **English** respectivement.

> [!TIP]
> Si nous ne définissons pas de sélection par défaut pour nos menus, l’utilisateur peut cliquer sur **Traduire** sans choisir de langue source ou cible. Les valeurs par défaut évitent d’avoir à résoudre ce problème.

Maintenant que `MainWindow` a été initialisé et que l’interface utilisateur est créée, vous n’avez plus qu’à cliquer sur le bouton **Traduire** pour exécuter le code.

## <a name="detect-language-of-source-text"></a>Détecter la langue du texte source

Nous créons maintenant la méthode de détection de la langue du texte source (texte entré dans notre zone de texte) à l’aide de Translator. La valeur retournée par cette demande est utilisée par la suite dans notre demande de traduction.

1. Dans Visual Studio, ouvrez l’onglet `MainWindow.xaml.cs`.
2. Ajoutez ce code à votre projet sous la méthode `PopulateLanguageMenus()` :
   ```csharp
   // ***** DETECT LANGUAGE OF TEXT TO BE TRANSLATED
   private string DetectLanguage(string text)
   {
       string detectUri = string.Format(TEXT_TRANSLATION_API_ENDPOINT ,"detect");

       // Create request to Detect languages with Translator
       HttpWebRequest detectLanguageWebRequest = (HttpWebRequest)WebRequest.Create(detectUri);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
       detectLanguageWebRequest.ContentType = "application/json; charset=utf-8";
       detectLanguageWebRequest.Method = "POST";

       // Send request
       var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
       string jsonText = serializer.Serialize(text);

       string body = "[{ \"Text\": " + jsonText + " }]";
       byte[] data = Encoding.UTF8.GetBytes(body);

       detectLanguageWebRequest.ContentLength = data.Length;

       using (var requestStream = detectLanguageWebRequest.GetRequestStream())
           requestStream.Write(data, 0, data.Length);

       HttpWebResponse response = (HttpWebResponse)detectLanguageWebRequest.GetResponse();

       // Read and parse JSON response
       var responseStream = response.GetResponseStream();
       var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
       dynamic jsonResponse = serializer.DeserializeObject(jsonString);

       // Fish out the detected language code
       var languageInfo = jsonResponse[0];
       if (languageInfo["score"] > (decimal)0.5)
       {
           DetectedLanguageLabel.Content = languageInfo["language"];
           return languageInfo["language"];
       }
       else
           return "Unable to confidently detect input language.";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Cette méthode envoie une demande HTTP `POST` à la ressource Detect. Elle accepte un seul argument, `text`, qui est transmis dans le corps de la demande. Par la suite, quand nous créons notre demande de traduction, le texte entré dans notre interface utilisateur est passé à cette méthode pour la détection de la langue.

Par ailleurs, cette méthode évalue le score de confiance de la réponse. Si le score est supérieur à `0.5`, la langue détectée s’affiche dans l’interface utilisateur.

## <a name="spell-check-the-source-text"></a>Vérifier l’orthographe du texte source

Nous créons maintenant une méthode de vérification orthographique de notre texte source à l’aide de l’API Vérification orthographique Bing. Cela garantit l’obtention des traductions exactes par Translator. Toutes les corrections de texte source sont transmises dans notre demande de traduction quand l’utilisateur clique sur le bouton **Traduire**.

1. Dans Visual Studio, ouvrez l’onglet `MainWindow.xaml.cs`.
2. Ajoutez ce code à votre projet sous la méthode `DetectLanguage()` :

```csharp
// ***** CORRECT SPELLING OF TEXT TO BE TRANSLATED
private string CorrectSpelling(string text)
{
    string uri = BING_SPELL_CHECK_API_ENDPOINT + "?mode=spell&mkt=en-US";

    // Create a request to Bing Spell Check API
    HttpWebRequest spellCheckWebRequest = (HttpWebRequest)WebRequest.Create(uri);
    spellCheckWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
    spellCheckWebRequest.Method = "POST";
    spellCheckWebRequest.ContentType = "application/x-www-form-urlencoded"; // doesn't work without this

    // Create and send the request
    string body = "text=" + System.Web.HttpUtility.UrlEncode(text);
    byte[] data = Encoding.UTF8.GetBytes(body);
    spellCheckWebRequest.ContentLength = data.Length;
    using (var requestStream = spellCheckWebRequest.GetRequestStream())
        requestStream.Write(data, 0, data.Length);
    HttpWebResponse response = (HttpWebResponse)spellCheckWebRequest.GetResponse();

    // Read and parse the JSON response; get spelling corrections
    var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
    var responseStream = response.GetResponseStream();
    var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
    dynamic jsonResponse = serializer.DeserializeObject(jsonString);
    var flaggedTokens = jsonResponse["flaggedTokens"];

    // Construct sorted dictionary of corrections in reverse order (right to left)
    // This ensures that changes don't impact later indexes
    var corrections = new SortedDictionary<int, string[]>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
    for (int i = 0; i < flaggedTokens.Length; i++)
    {
        var correction = flaggedTokens[i];
        var suggestion = correction["suggestions"][0];  // Consider only first suggestion
        if (suggestion["score"] > (decimal)0.7)         // Take it only if highly confident
            corrections[(int)correction["offset"]] = new string[]   // dict key   = offset
                { correction["token"], suggestion["suggestion"] };  // dict value = {error, correction}
    }

    // Apply spelling corrections, in order, from right to left
    foreach (int i in corrections.Keys)
    {
        var oldtext = corrections[i][0];
        var newtext = corrections[i][1];

        // Apply capitalization from original text to correction - all caps or initial caps
        if (text.Substring(i, oldtext.Length).All(char.IsUpper)) newtext = newtext.ToUpper();
        else if (char.IsUpper(text[i])) newtext = newtext[0].ToString().ToUpper() + newtext.Substring(1);

        text = text.Substring(0, i) + newtext + text.Substring(i + oldtext.Length);
    }
    return text;
}
// NOTE:
// In the following sections, we'll add code below this.
```

## <a name="translate-text-on-click"></a>Traduire le texte en cliquant sur le bouton

La dernière chose que nous devons faire est créer une méthode qui est appelée quand l’utilisateur clique sur le bouton **Traduire** dans l’interface utilisateur.

1. Dans Visual Studio, ouvrez l’onglet `MainWindow.xaml.cs`.
1. Ajoutez ce code à votre projet sous la méthode `CorrectSpelling()` et enregistrez :  
   ```csharp
   // ***** PERFORM TRANSLATION ON BUTTON CLICK
   private async void TranslateButton_Click(object sender, EventArgs e)
   {
       string textToTranslate = TextToTranslate.Text.Trim();

       string fromLanguage = FromLanguageComboBox.SelectedValue.ToString();
       string fromLanguageCode;

       // auto-detect source language if requested
       if (fromLanguage == "Detect")
       {
           fromLanguageCode = DetectLanguage(textToTranslate);
           if (!languageCodes.Contains(fromLanguageCode))
           {
               MessageBox.Show("The source language could not be detected automatically " +
                   "or is not supported for translation.", "Language detection failed",
                   MessageBoxButton.OK, MessageBoxImage.Error);
               return;
           }
       }
       else
           fromLanguageCode = languageCodesAndTitles[fromLanguage];

       string toLanguageCode = languageCodesAndTitles[ToLanguageComboBox.SelectedValue.ToString()];

       // spell-check the source text if the source language is English
       if (fromLanguageCode == "en")
       {
           if (textToTranslate.StartsWith("-"))    // don't spell check in this case
               textToTranslate = textToTranslate.Substring(1);
           else
           {
               textToTranslate = CorrectSpelling(textToTranslate);
               TextToTranslate.Text = textToTranslate;     // put corrected text into input field
           }
       }
       // handle null operations: no text or same source/target languages
       if (textToTranslate == "" || fromLanguageCode == toLanguageCode)
       {
           TranslatedTextLabel.Content = textToTranslate;
           return;
       }

       // send HTTP request to perform the translation
       string endpoint = string.Format(TEXT_TRANSLATION_API_ENDPOINT, "translate");
       string uri = string.Format(endpoint + "&from={0}&to={1}", fromLanguageCode, toLanguageCode);

       System.Object[] body = new System.Object[] { new { Text = textToTranslate } };
       var requestBody = JsonConvert.SerializeObject(body);

       using (var client = new HttpClient())
       using (var request = new HttpRequestMessage())
       {
           request.Method = HttpMethod.Post;
           request.RequestUri = new Uri(uri);
           request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
           request.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
           request.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
           request.Headers.Add("X-ClientTraceId", Guid.NewGuid().ToString());

           var response = await client.SendAsync(request);
           var responseBody = await response.Content.ReadAsStringAsync();

           var result = JsonConvert.DeserializeObject<List<Dictionary<string, List<Dictionary<string, string>>>>>(responseBody);
           var translation = result[0]["translations"][0]["text"];

           // Update the translation field
           TranslatedTextLabel.Content = translation;
       }
   }
   ```

La première étape est d’obtenir les langues « from » et « to », et le texte que l’utilisateur a entré dans notre formulaire. Si la langue source est définie sur **Détecter**, nous appelons `DetectLanguage()` pour déterminer la langue du texte source. Le texte peut être dans une langue non prise en charge par Translator. Dans ce cas, un message informe l’utilisateur et aucune traduction n’est retournée.

Si la langue source est l’anglais (qu’elle soit spécifiée ou détectée), nous vérifions l’orthographe du texte avec `CorrectSpelling()` et appliquons les corrections. Le texte corrigé est rajouté dans la zone de texte pour que l’utilisateur voit qu’une correction a été effectuée.

Le code pour traduire le texte doit vous être familier : générer l’URI, créer une demande, l’envoyer et analyser la réponse. Le tableau JSON peut contenir plusieurs objets pour la traduction, toutefois, notre application en a besoin d’un seul.

Quand une demande a réussi, `TranslatedTextLabel.Content` est remplacé par le texte de `translation`, qui met à jour l’interface utilisateur pour afficher le texte traduit.

## <a name="run-your-wpf-app"></a>Exécuter votre application WPF

Voilà, vous avez une application de traduction opérationnelle créée à l’aide de WPF. Pour exécuter votre application, cliquez sur le bouton **Démarrer** dans Visual Studio.

## <a name="source-code"></a>Code source

Le code source de ce projet est disponible sur GitHub.

* [Explorer le code source](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-C-Sharp-Tutorial)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Informations de référence Microsoft Translator](./reference/v3-0-reference.md)