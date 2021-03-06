---
title: Vues partielles
author: ardalis
description: Utilisation de vues partielles dans ASP.NET Core MVC
manager: wpickett
ms.author: riande
ms.date: 03/14/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/views/partial
ms.openlocfilehash: 169948e5d7dc8068463ed61114666148b785b217
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/30/2018
---
# <a name="partial-views"></a>Vues partielles

Par [Steve Smith](https://ardalis.com/), [Maher JENDOUBI](https://twitter.com/maherjend) et [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core MVC prend en charge les vues partielles. Celles-ci sont utiles quand vous avez des parties réutilisables de pages web que vous souhaitez partager entre différentes vues.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/partial/sample) ([procédure de téléchargement](xref:tutorials/index#how-to-download-a-sample))

## <a name="what-are-partial-views"></a>Que sont les vues partielles ?

Une vue partielle est une vue affichée dans une autre vue. La sortie HTML générée par l’exécution de la vue partielle est affichée dans la vue appelante (ou parente). Tout comme les vues, les vues partielles utilisent l’extension de fichier *.cshtml*.

## <a name="when-should-i-use-partial-views"></a>Quand dois-je utiliser des vues partielles ?

Les vues partielles sont un moyen efficace de découper des vues volumineuses en composants plus petits. Elles permettent de réduire la duplication du contenu d’une vue et de réutiliser les éléments des vues. Les éléments de disposition communs doivent être spécifiés dans [_Layout.cshtml](layout.md). Le contenu réutilisable non lié à une disposition peut être encapsulé dans des vues partielles.

Si vous avez une page complexe composée de plusieurs parties logiques, il peut s’avérer utile d’utiliser chaque partie en tant que vue partielle. Chaque partie de la page peut être vue séparément du reste de la page. La vue de la page elle-même devient beaucoup plus simple, car elle contient uniquement la structure globale de la page et les appels pour afficher les vues partielles.

Conseil : Suivez le [principe DRY (Ne vous répétez pas)](http://deviq.com/don-t-repeat-yourself/) dans vos vues.

## <a name="declaring-partial-views"></a>Déclaration de vues partielles

Vous créez les vues partielles comme les autres vues : vous créez un fichier *.cshtml* dans le dossier *Vues*. Il n’existe aucune différence sémantique entre une vue partielle et une vue classique, elles sont simplement affichées différemment. Une vue peut être retournée directement depuis le `ViewResult` d’un contrôleur. Cette même vue peut être utilisée comme vue partielle. La principale différence entre le rendu d’une vue et celui d’une vue partielle est que les vues partielles n’exécutent pas *_ViewStart.cshtml* (contrairement aux vues). Pour plus d’informations sur *_ViewStart.cshtml*, consultez [Disposition](layout.md).

## <a name="referencing-a-partial-view"></a>Référencement d’une vue partielle

À partir d’une page de vue, vous pouvez afficher une vue partielle de plusieurs manières. Le plus simple est d’utiliser `Html.Partial`, qui retourne `IHtmlString` et peut être référencé en faisant précéder l’appel de `@` :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Home/About.cshtml?range=9)]

La méthode `PartialAsync` est disponible pour les vues partielles contenant du code asynchrone (bien qu’il soit déconseillé d’utiliser du code dans les vues) :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Home/About.cshtml?range=8)]

Vous pouvez afficher une vue partielle avec `RenderPartial`. Cette méthode ne retourne aucun résultat. Elle effectue un streaming de la sortie affichée directement vers la réponse. Comme elle ne retourne aucun résultat, elle doit être appelée dans un bloc de code Razor (vous pouvez également appeler `RenderPartialAsync` si nécessaire) :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Home/About.cshtml?range=10-12)]

Dans la mesure où elle effectue directement un streaming du résultat, `RenderPartial` et `RenderPartialAsync` peuvent être plus efficaces dans certains scénarios. Toutefois, dans la plupart des cas, il est recommandé d’utiliser `Partial` et `PartialAsync`.

> [!NOTE]
> Si vos vues doivent exécuter du code, le modèle recommandé consiste à utiliser un [composant de vue](view-components.md) au lieu d’une vue partielle.

### <a name="partial-view-discovery"></a>Découverte des vues partielles

Quand vous référencez une vue partielle, vous pouvez faire référence à son emplacement de plusieurs façons :

```text
// Uses a view in current folder with this name
// If none is found, searches the Shared folder
@Html.Partial("ViewName")

// A view with this name must be in the same folder
@Html.Partial("ViewName.cshtml")

// Locate the view based on the application root
// Paths that start with "/" or "~/" refer to the application root
@Html.Partial("~/Views/Folder/ViewName.cshtml")
@Html.Partial("/Views/Folder/ViewName.cshtml")

// Locate the view using relative paths
@Html.Partial("../Account/LoginPartial.cshtml")
```

Vous pouvez avoir différentes vues partielles portant le même nom dans des dossiers de vues distincts. Quand vous référencez des vues par leur nom (sans extension de fichier), les vues de chaque dossier utilisent la vue partielle située dans le même dossier. Vous pouvez également spécifier une vue partielle par défaut à utiliser, en la plaçant dans le dossier *Partagé*. La vue partielle partagée est utilisée par toutes les vues qui n’ont pas leur propre version de la vue partielle. Une vue partielle par défaut (dans *Partagé*) peut être remplacée par une vue partielle ayant le même nom dans le dossier de la vue parente.

Les vues partielles peuvent être *chaînées*. En d’autres termes, une vue partielle peut appeler une autre vue partielle (tant que vous ne créez pas de boucle). Dans chaque vue ou vue partielle, les chemins relatifs sont toujours relatifs par rapport à cette vue, et non par rapport à la vue racine ou parente.

> [!NOTE]
> Si vous déclarez un `section` [Razor](razor.md) dans une vue partielle, il n’est pas visible pour ses parents. Il se limite à la vue partielle.

## <a name="accessing-data-from-partial-views"></a>Accès aux données à partir de vues partielles

Quand une vue partielle est instanciée, elle obtient une copie du dictionnaire de `ViewData` de la vue parente. Les mises à jour apportées aux données de la vue partielle ne sont pas conservées dans la vue parente. Tout changement apporté à `ViewData` dans une vue partielle est perdu quand la vue partielle est retournée.

Vous pouvez passer une instance de `ViewDataDictionary` à la vue partielle :

```csharp
@Html.Partial("PartialName", customViewData)
   ```

Vous pouvez également passer un modèle dans une vue partielle. Il peut s’agir du modèle de vue de la page, ou d’une partie de celui-ci, ou d’un objet personnalisé. Vous pouvez passer un modèle à `Partial`,`PartialAsync`, `RenderPartial` ou `RenderPartialAsync` :

```csharp
@Html.Partial("PartialName", viewModel)
   ```

Vous pouvez passer une instance de `ViewDataDictionary` et un modèle de vue à une vue partielle :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Articles/Read.cshtml?range=15-16)]

Le code ci-dessous montre la vue *Views/Articles/Read.cshtml*, qui contient deux vues partielles. La seconde vue partielle passe un modèle et `ViewData` à la vue partielle. Vous pouvez passer le nouveau dictionnaire de `ViewData` tout en conservant le `ViewData` existant, si vous utilisez la surcharge de constructeur du `ViewDataDictionary` mis en évidence ci-dessous :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Articles/Read.cshtml)]

*Views/Shared/AuthorPartial* :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Shared/AuthorPartial.cshtml)]

Le *ArticleSection* partiel :

[!code-html[Main](partial/sample/src/PartialViewsSample/Views/Articles/ArticleSection.cshtml)]

Au moment de l’exécution, les vues partielles sont affichées dans la vue parente, elle-même affichée dans le fichier *_Layout.cshtml* partagé

![sortie de vue partielle](partial/_static/output.png)
