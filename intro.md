---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
permalink: /
---
<style>
    body {
    counter-reset: chapter; /* create a chapter counter scope */
}
h1:before {
    content: counter(chapter) ". ";
    counter-increment: chapter; /* add 1 to chapter */
}
</style>

*Ceci est la première partie de l'atelier "Développez une application web qui n'explose pas".
[Voir la suite ici]({{site.baseurl}}/demineur).*
# Installation

## Dans le cloud
Pour cet atelier, vous pouvez tout faire dans le navigateur sans
installer quoi que ce soit sur votre machine. Vous avez:
* un éditeur ici : [https://ellie-app.com](https://ellie-app.com){:target="_blank"}

  **Astuces :**
  - `Ctrl + Shift + Enter` pour compiler
  - la petite icône en haut à droite de l'éditeur de code pour formater
    automatiquement le code.
* un REPL ici : [http://elmrepl.cuberoot.in/](http://elmrepl.cuberoot.in/){:target="_blank"}


## En local
Si vous souhaitez suivre cet atelier sur votre machine, voici la procédure.


Pré-requis: `npm` . Pour cela, installer
[NodeJS](https://nodejs.org/en/){:target="_blank"}.
Puis :
```
npm install --global elm elm-format
```

> Si vous rencontrez des soucis de droits d'installation sous linux, essayez
> [cette solution](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally){:target="_blank"}
> puis relancez la commande précédente.

Il faudra également trouver plugin pour votre éditeur de texte pour
avoir le plus de confort possible.
L'un des plus aboutis à l'écriture de cet atelier (mai 2019) est
celui pour IntelliJ.

# Quelques éléments de syntaxe

En Elm, on passe les argument aux fonctions en les séparant par des espaces.
Par exemple, pour arrondir un floattant, on utilise la fonction `round`:
```elm
three = round 3.2
```

Pour *définir* une fonction, on écrit le nom de la fonction, suivi du nom
de ses arguments séparés par des espaces. Par exemple la fonction suivante
ajoute ses deux arguments:
```elm
add a b = a + b
```


> **\>\>\> À vous de jouer !**
>
> Dans le REPL ([http://elmrepl.cuberoot.in/](http://elmrepl.cuberoot.in/){:target="_blank"}
> ou `elm repl` en console), entrez successivement (en séparant les
> lignes par un appui sur "Entrée") puis observez le résultat:
> ```elm
> 1 + 41
> round 3.2
> add a b = a + b
> add 3 5
> ```

Vous remarquez que lors de la définition de `add`, la console nous répond:
```
> add a b = a + b
<function> : number -> number -> number
```
La chaîne `number -> number -> number` est appelé la **signature** de la fonction
`add`.
Elle indique qu'elle prendre deux nombres en paramètres et que son résultat
est également un nombre.

> **\>\>\> À vous de jouer !**
>
> Essayez de deviner la signature de `add3 a b c = a + b + c`... Et vérifiez-le
> dans le REPL!


Comme on vient de le voir, Elm peut "calculer" tout seul le type d'une fonction (on dit *inférer* le type);
cependant, dans un soucis de documentation du code, dans un vrai fichier source
on rajoute le typage juste avant la fonction de cette manière:
```elm
add : number -> number -> number
add a b = a + b
```

En Elm, la structure `if condition then valueA else valueB` est une valeur
(c'est l'équivalent de l'opérateur ternaire `condition?valueA:valueB` de JS). Choses à savoir :
* la clause `else` est obligatoire,
* `valueA` et `valueB` doivent, avoir le même type `T`.
* `if condition then valueA else valueB` est une valeur de type `T`.

Par exemple, on peut déterminer s'il est avant ou après midi:
```elm
displayHour hour = if hour < 12 then "morning!" else "afternoon!"
```

> **\>\>\> À vous de jouer !**
>
> 1. Déterminez le type de `displayHour` et vérifiez votre réponse dans le
>    REPL.
> 2. Modifier la fonction pour qu'elle renvoie "night!" lorsque `hour` est
>    supérieur à 19 (et continue à afficher `morning!` ou `afternoon!` de
>    façon adéquate).


# Une application simple: le compteur


Dans cette partie, nous mettons en place une application implémentant un simple
compteur.


## Version en ligne

Ouvrez juste l'adresse : [https://ellie-app.com](https://ellie-app.com){:target="_blank"} !
Testez le code (rappel: `Ctrl + Maj + Entrée` pour compiler) puis rendez-vous directement à  [The Elm Architecture](#the-elm-architecture).


## Version "local"

> **\>\>\> À vous de jouer !**
>
> 1. Créez un dossier `elm-workshop`.
> 2. Placez vous dans ce dossier et faites un `elm init`.
> 3. Copiez-collez le code ci-dessous dans le fichier `src/Main.elm` (le dossier
>    `src` est déjà créé normalement).
> 4. Lancez `elm reactor` en étant dans le dossier `elm-workshop`.
> 5. Ouvez un navigateur à l'adresse [http://localhost:8000](http://localhost:8000)
>    et testez l'application.

```elm
module Main exposing (main)

import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)


type alias Model =
    { count : Int }


initialModel : Model
initialModel =
    { count = 0 }


type Msg
    = Increment
    | Decrement


update : Msg -> Model -> Model
update msg model =
    case msg of
        Increment ->
            { model | count = model.count + 1 }

        Decrement ->
            { model | count = model.count - 1 }


view : Model -> Html Msg
view model =
    div []
        [ button [ onClick Increment ] [ text "+1" ]
        , div [] [ text (String.fromInt model.count) ]
        , button [ onClick Decrement ] [ text "-1" ]
        ]


main : Program () Model Msg
main =
    Browser.sandbox
        { init = initialModel
        , view = view
        , update = update
        }
```


## The Elm Architecture
Un peu de théorie avant de poursuivre.

Elm incorpore une architecture "Model - View - Update", qui peut être résumée
par le schéma suivant:

![The Elm Architecture]({{site.baseurl}}/img/tea.png)

On définit un *modèle*, qui est transformé en une *vue* affichée par le "runtime"
de Elm. Lorsque l'utilisateur effectue une action, le "runtime" va envoyer
un *message* à la fonction *update*; cette dernière va construire un nouveau
modèle, ce qui va relancer le cycle.


Nous avons juste à spécifier : le modèle, la fonction de vue, la fonction
d'update et les messages. Le runtime de Elm se charge de faire le lien entre
tous ses éléments!

## Défi
Votre mission, même si vous ne l'acceptez pas, est de modifier l'application
"compteur" afin de rajouter un bouton "Remise à zéro" au compteur.

Vous pouvez essayer de comprendre seul comment fonctionne le code ou alors
consulter la partie suivante qui détaille le fonctionnement du programme.

*Si vous avez relevé le défi, vous pouvez
[voir la suite de l'atelier]({{site.baseurl}}/demineur) !*

## Fonctionnement du compteur, ligne à ligne!

```elm
module Main exposing (main)
```
On définit que le fichier courant est un module (et on expose une fonction).

```elm
import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)
```
On importe différents modules. Par défaut, si on écrit juste `import Html`,
on a besoin d'écrire `Html.div` pour utiliser la fonction `div`.

Ici, on importe directement les fonctions `button`, `div`, ... On pourra ainsi
appeler directement ces fonctions sans les préfixer par le nom du module.

```elm
type alias Model =
    { count : Int }
```
On définit notre modèle. On dit qu'il s'agit d'un enregistrement (*record* en
anglais) qui contient un seul champ `count` de type entier.

> **Note aux javascripteurs**
>
> Un enregistrement est en quelque sorte l'équivalent d'un objet en Javascript.
> La différence fondamentale est qu'un enregistrement ne contient QUE des
> données, pas de "méthodes" ou autre concepts similaires.

```elm
initialModel : Model
initialModel =
    { count = 0 }
```
On construit le modèle initial en donnant comme valeur "0" au champ `count`.
> Comme vu dans la section précédente, on indique le type de la valeur
> `initialModel` sur la première ligne ; ce n'est pas obligatoire mais améliore
> la lisibilité du programme et peut aider le compilateur à afficher de meilleurs
> messages d'erreur.

```elm
type Msg
    = Increment
    | Decrement
```
On indique qu'il y a deux messages possibles: on augmente ou on diminue le
compteur. `Increment` et `Decrement` sont maintenant deux valeurs de type `Msg`.


```elm
update : Msg -> Model -> Model
update msg model =
    case msg of
        Increment ->
            { model | count = model.count + 1 }

        Decrement ->
            { model | count = model.count - 1 }
```
Comme l'indique sa signature, cette fonction prend un message et un modèle
en argument et renvoie un nouveau modèle.
> La syntaxe `{ record | field = foo }` construit un nouvel enregistrement
> identique à `record`, sauf pour le champ `field` qui a été affecté à `foo`.
>
> On peut changer plusieurs champs d'un coup en les séparant d'une virgule:
> ```elm
>  { record | field1 = foo, field2 = foo }
> ```


```elm
view : Model -> Html Msg
view model =
    div []
        [ button [ onClick Increment ] [ text "+1" ]
        , div [] [ text (String.fromInt model.count) ]
        , button [ onClick Decrement ] [ text "-1" ]
        ]

```
La fonction de vue. Comme son type l'indique, elle transforme notre modèle en
bon vieux HTML.

`div` est une fonction Elm qui sera convertie en la balise
HTML `<div>`. La première liste d'arguments indique en quelque sorte les
attributs sur cette balise (on pourrait par exemple spécifier une classe CSS,
un `id`, ...). La seconde est la liste des enfants.

Voici la signature de la fonction `text : String -> Html msg`. En effet pour
construire du HTML, Elm ne peut pas directement prendre du texte, il est obligé
de l'envelopper dans du HTML. C'est un peu l'équivalent de
`document.createTextNode` en Javascript.

> En gros, le HTML généré sera (si le modèle est `{count = 0 }`) :
> ```html
> <div>
>   <button onclick="_=> sendMsg(Increment)">+1</button>
>   <div>0</div>
>   <button onclick="_=> sendMsg(Decrement)">-1</button>
> </div>
> ```

```elm
main : Program () Model Msg
main =
    Browser.sandbox
        { init = initialModel
        , view = view
        , update = update
        }
```
Ce dernier bout de code donne au runtime toutes les informations nécessaires.
Ainsi dès qu'un utilisateur va cliquer sur le boutton "+1", un message sera
généré et envoyé à la fonction `update`.


*La première partie de l'atelier est terminée.
[Voir la suite ici]({{site.baseurl}}/demineur).*