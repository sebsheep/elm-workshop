---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
permalink: /demineur/
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

*Ceci est la deuxième partie de l'atelier "Développez une application web qui n'explose pas".
[Retrouvez l'introduction ici]({{site.baseurl}}/).*

Nous allons construire un clone du "démineur". Si vous ne
connaissez pas, vous pouvez
[essayer ici](http://demineur.hugames.fr/#level-3){:target="_blank"}
 [ou là](http://demineur.org/){:target="_blank"}.

Lorsque vous aurez fini l'atelier, n'hésitez pas à
[partager votre travail](https://annuel.framapad.org/p/atelier-elm-demineur-examples-sebbes)


# Mise en place


Commençons par un objectif simple: afficher une seule case de
démineur, selon qu'elle est "révélée" (quand l'utilisateur clique
dessus) ou pas, et si elle contient une bombe ou pas.

Notre modèle contient donc uniquement une cellule avec deux attributs
booléens:
* `revealed`
* `isMine`

Partons du [squelette d'application suivant](https://ellie-app.com/5CZrhZ37M4va1){:target="_blank"}


*Rappel*: pour accéder aux attributs imbriqués, la syntaxe est identique
à celle de JS. Par exemple pour accéder à l'attribut `isMine` de la cellule
contenue dans le modèle, on écrit: `model.cell.isMine`.

> **\>\>\> À vous de jouer !**
>
> 1. En utilisant un `if/then/else`,
>    modifiez la fonction `view` pour afficher une bombe si la cellule
>    est une bombe et la chaîne "0" sinon (on ignore pour l'instant
>    l'attribut `revealed`). On pourra utiliser l'emoji
>    suivant : 💣 (copiez-collez le dans votre code, c'est du texte !
>    Remarque: dans un éditeur "hors-ligne" il peut ne pas s'afficher
>    si la police ne supporte pas les emojis; il ne devrait pas y avoir
>    de soucis dans le navigateur 😊).
> 2. Testez votre code, en remplaçant le `isMine = False` par
>    `isMine = True` dans le
>    `init` puis en recompilant.
> 3. Ajoutez la gestion de `revealed`: s'il est à faux, on n'affiche
>    rien dans la case, sinon, on fait comme en 1.
>
>    Rappel astucieux: dans Ellie, en haut à droite de l'éditeur
>    se trouve un bouton pour formatter votre code automatiquement !
> 4. Testez votre code avec différentes combinaisons de `isMines`
>    et `revealed` en recompilant à chaque fois.
> 5. Extrayez le code que vous venez d'écrire dans une fonction
>    `viewCell : Cell -> Html Msg` et utilisez cette fonction dans
>    `view : Model -> Html Msg`.


# Plusieurs cellules!

On modifie notre modèle, au lieu d'avoir juste UNE cellule, on a une
liste de cellules (notez au passage que `cell` devient `cells`) :
```elm
type alias Model =
    { cells : List Cell }
```

Quelques choses à savoir sur les listes:
1. Liste vide : on peut construire la liste vide avec `[]`.
2. Tous les éléments d'une liste on le même type. Par exemple
   `[1.1, 5.5, 42.5]` a pour type `List Float` et `["Hello", "world"]`
   a pour type `List String`; la liste `[5, "hello"]` n'est pas valide.
3. En Elm, il n'y a pas de `for` ou de `while`. On a mieux : le `map`!
   Cette fonction permet d'application une transformation à tous les
   éléments de la liste. Par exemple:
   ```elm
   double x = 2 * x

   foo = List.map double [1, 2, 3]
   ```
   `foo` vaut alors `[2, 4, 6]` ; essayez dans le REPL !

   Si on veut afficher une liste "foo, bar, baz" en HTML, on peut
   utiliser le code suivant (on rappelle que `text : String -> Html Msg`
   transforme une string "brut" en Html):
   ```elm
   ul []
    [ li [] [text "foo"]
    , li [] [text "bar"]
    , li [] [text "baz"]
    ]
   ```
   On peut alors utiliser la fonction `List.map` pour simplifier le code:
   ```elm
   displayItem : String -> Html Msg
   displayItem itemDescription =
     li [] [text itemDescription]

   ul [] (List.map displayItem ["foo", "bar", "baz"]) -- revient au même qu'avant!
   ```
   On peut encapsuler cela dans une fonction, pour avoir un code très lisible et réutilisable après:
   ```elm
   viewListOfWords : List String -> Html Msg
   viewListOfWords items =
       ul [] (List.map displayItem items)

   -- appels de la fonction :
   viewListOfWords ["Sébastien", "Jean-Baptiste", "Tariq"]
   viewListOfWords ["Bananes", "Abricots", "Pommes", "Pastèques"]

   ```

> **\>\>\> À vous de jouer !**
>
> 1. Changez le modèle comme indiqué ci-dessus.
> 2. Laissez vous guider par le compilateur pour corriger votre code !
>    * pour l'instant codez "en dur" 3 cellules dans le `init`,
>    * dans la fonction `view : Model -> Html Msg`, utilisez `List.map`
>    pour inclure toutes les celulles dans un `div`.
> 3. Une fois que le code compile et que vos 3 cellules s'affichent,
>    creez 100 cellules identiques dans le modèle initial ; la fonction
>    [`List.repeat` (lien cliquable!)](https://package.elm-lang.org/packages/elm/core/latest/List#repeat){:target="_blank"}
>    peut être utile!
> 4. Ajoutez les attributs `style "display" "grid"` et
>    `style "grid-template-columns" "repeat(10, 50px)"` au `div` contenant
>    toutes les cellules pour les afficher en grille.


# Donner des identifiants!

Pour gérer les clicks sur les boutons, nous aurons besoin
que chaque cellule ait un identifiant. Le type `Cell` devient alors:

```elm
type alias Cell =
    { id: Int, isMine : Bool, revealed : Bool }
```

Dans les sections à venir, il n'y aura pas de rendu "graphique" visible,
le compilateur sera notre outil de test principal ! Un petit de patience,
à la fin de la partie "Update !", vous aurez la satisfaction de voir
qu'à partir du moment où le code compile, il fonctionne comme on l'attend.

> **\>\>\> À vous de jouer !**
>
> 1. Changer le type `Cell`, et laissez vous guider par le compilateur pour
>    corriger le code. Pour l'instant mettez 1 comme `id` à toutes les
>    cellules.

**Fonctions anonymes**

Rappelez-vous que grâce à `List.map` on peut appliquer une transformation à une liste. L'exemple que j'avais donné était:
```elm
double x = 2 * x

foo = List.map double [1, 2, 3]
```
Cependant, il est fréquent que la transformation (`double` dans notre cas)
soit très courte et qu'on ne veuille pas lui donner de nom. On peut alors
utiliser une *fonction anonyme* avec la syntaxe
`\<argument> -> <resultat>`; notre exemple devient alors:
```elm
foo = List.map (\x -> 2 * x) [1, 2, 3]
```

> **Remarque :** en javascript, on peut déclarer une fonction anonyme
> de deux façons:
> ```javascript
> function(x) {
>    return 2 * x;
> }
> ```
> ou (ES6):
> ```javascript
> x => 2 * x
> ```

> **\>\>\> À vous de jouer !**
>
> {:start="2"}
> 2. Dans le REPL, en utilisant `List.map` avec une fonction anonyme,
>    transformez la liste `[1, 2, 3]` en `[{id = 1}, {id = 2}, {id = 3}]`.
> 3. Toujours dans le REPL, en utilisant la fonction
>    [`List.range`](https://package.elm-lang.org/packages/elm/core/latest/List#range){:target="_blank"}
>    générez la liste `[{id = 1}, {id = 2}, ..., {id = 100}]`.
> 4. Revenez à l'application et modifiez le `init` pour que chacune des
>    100 cellules ait un `id` différent.

# Générer des messages !

Dans cette partie, on commence à gèrer le "clic gauche" permettant de révéler une case.

Pour cela, on va modifier notre type `Msg` pour indiquer qu'on peut
engendrer le message "La case d'id X doit être révélée". C'est donc un message qui prend un paramètre entier. On l'indique de cette façon:

```elm
type Msg
    = Reveal Int
```
On pourra alors constuire les messages `Reveal 1` pour révéler la cellule
d'identifiant 1, `Reveal 32` pour celle d'identifiant 32...


> **\>\>\> À vous de jouer !**
>
> 1. En haut du fichier, ajoutez l'import: `import Html.Events exposing (onClick)`.
> 2. Modifiez le type `Msg`.
> 3. Modifiez la fonction `viewCell` pour qu'un message adapté soit généré
>    à chaque click sur une cellule.
> 4. En compilant avec l'option `--debug`, un débogueur à "voyage dans
>    le temps est incorporé en bas à droite de la page. Sous "Ellie",
>    il faut aller chercher en haut à droite l'onglet "DEBUG".
>
>    Cliquez sur votre grille sur différentes cellules et vérifiez dans
>    le debogueur que les messages ont bien été générés.


# Update !

Dans cette partie, on intercepte les messages et on modifie la grille
en conséquence.

> **\>\>\> À vous de jouer !**
>
> 1. Écrivez une fonction `revealIfId : Int -> Cell -> Cell` prenant
>    en paramètre un identifiant `id` et une cellule.
>    Si l'identifiant de la cellule n'est pas `id`,
>    la fonction renvoie la cellule sans la modifier.
>    Sinon, la fonction renvoie la cellule avec le champ `revealed`
>    à `True`. Vérifiez que le code compile!

Rappelez-vous que pour l'application "Compteur", nous avions écrit le
code suivant pour réagir aux différents messages :

```elm
case msg of
    Increment ->
        ...

    Decrement ->
        ...
```

Ici, il y a un seul message possible et celui-ci a un argument. Nous
pouvons alors effectuer le filtrage par motif ("pattern matching" en
anglais) suivant:
```elm
case msg of
    Reveal 1 ->
        <reveal cell of id 1>

    Reveal 2 ->
        <reveal cell of id 2>

    ...

    Reveal 100 ->
        <reveal cell of id 100>
```

Il serait bien trop long d'écrire cela de cette manière. Nous pouvons
*capturer* l'identifiant en lui donnant un nom:
```elm
case msg of
    Reveal id ->
        <reveal cell of id ... "id">
```

> {:start="2"}
> 2. Modifiez la fonction update pour intercepter les messages de la
>    forme `Reveal id`. Ne cherchez pas à modifier le modèle, faites juste
>    en sorte d'avoir un code qui compile.
> 3. Modifier le modèle.
>
     **Indication:** on pourra utiliser `List.map`
>    sur `model.cells`, avec une fonction anonyme faisant appel à
>    `revealIfId` (ne cherchez pas à être "efficace" ;) ).
> 4. TADIN ! Cliquez sur votre grille, vous devez la "révéler" au fur et à
>    mesure (bon pour l'instant, ce n'est pas très intéressant,
>    il n'y a soit que des bombes, soit aucune bombe!).

Dans l'étape 3. ci-dessus, on peut en fait se passer de la fonction anonyme
grâce à:

**Application partielle**

En Elm, on peut appliquer *partiellement* les fonctions. Par exemple (essayez dans le REPL!) :
```elm
add a b = a + b

addFive = add 5
```
Observez alors le type de `addFive : number -> number`. C'est une
fonction qui attend encore 1 argument (`add` attend 2 arguments, `addFive`
fourni le premier par défaut!).

On peut ensuite manipuler `addFive` comme n'importe quelle autre fonction à
1 argument numérique. Par exemple `addFive 3` donne `8`. "Moui bon, et alors" me direz vous... eh bien on peut faire:
```elm
List.map addFive [1, 2, 3] -- résultat: [6, 7, 8]
```
Mais d'après la définition de `addFive`, cela revient exactement à:
```elm
List.map (add 5) [1, 2, 3] -- résultat: [6, 7, 8]
```
Plus besoin de définir une fonction auxiliaire!


> **\>\>\> À vous de jouer !**
>
> {:start="5"}
> 5. Reprenez les exemples précédents dans le REPL.
> 6. Reprenez le code issue de l'étape 3. et ré-écrire la fonction
>    d'update sans utiliser de fonction anonyme.


# Placement de bombes

Dans cette partie, on place les bombes de façon aléatoire sur la grille.

> **\>\>\> À vous de jouer !**
>
> 1. Voici la signature de la fonction `List.member : a -> List a -> Bool`.
>    Tentez de comprendre ce qu'elle calcule (combien d'arguments ? de
>    quels types les arguments ? quelle est la valeur de retour ?) et
>    vérifiez le dans le REPL.
> 2. Écrire une fonction `buildGrid : List Int -> List Cell` qui prend
>    en argument la liste des identifiants de cellules qui doivent être des
>    mines. Elle renvoie une liste de 100 cellules d'identifiant de 1
>    à 100 (le code ne devrait pas être très différent de celui du `init`
>    actuel).
>
>    **Indication:** utiliser la fonction `List.member`.
> 3. Réécrire le `init` en appelant `buildGrid [2, 3, 25, 35]` et testez
>    que vos bombes s'affichent au bon endroit.


Pour générer de l'aléatoire, on a d'abord besoin d'installer un package.
Si vous compilez "à la main", vous devez faire un :
```elm
elm install elm/random
```
Sous Ellie, dans la goutière de gauche, il y a un petit icône en deuxième
position vous permettant d'installer un package directement ; il faut chercher `random` et sélectionner le premier module.


## Effectuer des "effets de bord"

Jusqu'ici, on a uniquement programmé des fonctions dite *pures* dans le
sens où si on a appelle plusieurs fois de suite une fonction avec les mêmes
arguments, on aura toujours le même résultat. Et en Elm, les fonctions ne
peuvent agir QUE de cette manière là.

> Ce n'est pas le cas dans des langages impératifs comme JS, Java ou C.
> Par exemple, prenons le code suivant:
> ```javascript
> var lang = "fr";
>
> function greetings(name) {
>     switch(lang) {
>         case "fr":
>             return "Bonjour " + name;
>         case "en":
>             return "Hello " + name;
>         default:
>             // default to Elvish (why not?)
>             return "Suilad " + name;
>     }
> }
> greetings("Sébastien");
> lang = "en";
> greetings("Sébastien");
> ```
> Le premier appel à `greetings("Sébastien")` renvoie "Bonjour Sébastien"
> alors que le second renvoie "Hello Sébastien"... La valeur renvoyée
> dépend d'une configuration globale qui peut changer au cours de
> l'exécution. En Elm ce n'est pas possible!

Et générer un nombre aléatoire ne peut pas être une action "pure": on
veut qu'à chaque appel, le résultat soit différent; cela demande de
conserver quelque part un état mémorisant le dernier nombre généré.

En Elm, c'est le *runtime* qui effectue les effets de bords. Le principe
est le suivant:
* notre fonction `update` demande au runtime d'effectuer une *commande*
  (dans notre cas "*génère moi une donnée aléatoire*").
* le runtime effectue ses calculs "impurs" pour effectuer la
  commande.
* une fois la commande effectuée, le runtime appelle de nouveau la
  fonction `update` avec un message de la forme `CommandPerformed result`.

Le schéma vu précédemment devient alors:

![The Elm Architecture]({{site.baseurl}}/img/tea-cmd.png)

> **\>\>\> À vous de jouer !**
>
> 1. Comme vous l'avez vu, on a légèrement changé le modèle, on doit donc
>    l'indiquer au runtime. Pour cela, modifiez
>    votre `main` par :
>    ```elm
>    main =
>        Browser.element
>           { view = view
>           , update = update
>           , init = \() -> init
>           , subscriptions = always Sub.none
>           }
>   ```
> 2. La compilation nous indique que nos `init` et `update` ne sont pas
>    corrects. Dans votre fonction `update`, transformez :
>    ```elm
>    Reveal id ->
>        <nouveau model>
>    ```
>    en :
>    ```elm
>    Reveal id ->
>        (<nouveau model>, Cmd.none)
>    ```
>    Cela indique au runtime qu'il n'y a aucune commande à effectuer
>    lorsqu'on
>    révèle une case.
>
>    Réalisez une transformation similaire sur le `init`.
> 3. Rajoutez un message en transformant le type `Msg` en:
>    ```elm
>    type Msg
>        = Reveal Int
>        | Mine Int
>    ```
>    Ce nouveau message sera produit par le runtime lorsqu'il aura généré
>    un nombre aléatoire. Ce nombre sera l'identifiant de la cellule
>    minée (pour l'instant, on simplifie avec une seule cellule minée).
>
>    Mettez à jour la fonction `update` pour reconstruire entièrement la
>    grille à la reception de ce message (vous pouvez utiliser
>    `buildGrid`!).
> 4. Rajoutez l'import suivant en haut du fichier `import Random`.
>    Dans le `init`, remplacez le `Cmd.none` par:
>    ```elm
>    Random.generate Mine (Random.int 1 100)
>    ```
>    Cela indique au runtime de générer un nombre entier x entre 1 et 100,
>    puis de produire le message `Mine x`.
>
>    Testez plusieurs fois en mettant `revealed` à `True`, la bombe
>    devrait se trouver à des endroits différents à chaque fois!
> 5. Dans le type `Msg`, transformez `Mine Int` en `Mines (List Int)`.
>
>    Modifiez le `update` en conséquence.
>
>    Grâce à
>    [la documentation](https://package.elm-lang.org/packages/elm/random/latest/Random){:target="_blank"},
>    cherchez comment dire au runtime "*génère 20 entiers
>    entre 1 et 100*" et modifiez le `init` en conséquecne.
>
>    *Remarque: de cette façon, nous n'aurons pas forcément 20 mines car
>    on peut avoir plusieurs fois le même identifiant (voir la partie
>    [Finitions](#finitions) pour une idée d'algorithme.*
>


# Compter les mines !

On cherche ici à compter les mines autour de chaque case.

> **\>\>\> À vous de jouer !**
>
> 1. Écrire une fonction `neighbors : Int -> List Int` prenant en argument
>    l'identifiant d'une cellule et renvoyant la liste des
>    identifiants des "voisins" de cette cellules.
> 2. Écrire une fonction `countMinesAround : Int -> List Cell -> Int`
>    prenant en argument l'identifiant d'une cellule cible,
>    la liste de toutes
>    les cellules et renvoie le nombre de mines autour de la cible.
>
>    Les fonctions
>    [`List.length`](https://package.elm-lang.org/packages/elm/core/latest/List#length)
>    et [`List.filter`](https://package.elm-lang.org/packages/elm/core/latest/List#filter) ainsi que l'utilisation d'une fonction anonyme
>    peuvent être utile.
> 3. Modifier la fonction `viewCell` pour afficher le nombre de mines
>    au voisinage lorsqu'elle est révélée (et que ce n'est pas une mine!).
>
>    Vous devrez pour cela rajouter comme argument la liste de
>    toutes les mines. Réfléchissez à l'ordre des arguments pour
>    pouvoir utiliser une *application pratielle* de cette fonction
>    dans la fonction `view`.
> 4. Révélez toutes les cases (dans `buildGrid`... ou en cliquant sur
>    toutes les cases, c'est vous qui voyez!) et vérifiez que le
>    comptage est correct.


# Ajouter les dapreaux !

On gère ici l'ajout des drapeaux (grâce au "clic droit"). On utilise
pour cela l'événement JS `contextmenu` ; cet évènement n'est pas
supporté "out of the box" en Elm:
1. Installer le package `elm/json`
2. Rajoutez l'import `import Json.Decode`
3. Ajoutez cette fonction :
   ```elm
   onRightClick : msg -> Attribute msg
   onRightClick msg =
       Html.Events.preventDefaultOn "contextmenu" (Decode.succeed ( msg, True ))
   ```
4. Vous pouvez maintenant capturer les "clicks droits" exactement de la
   même façon que vous capturez les "clicks gauche" en utilisant
   `onRightClick myMsg` au lieu de `onClick myMsg`.

> **\>\>\> À vous de jouer !**
>
> 1. Rajoutez un attribut `withFlag : Bool` au type `Cell`.
> 2. En suivant les mêmes étapes que pour la gestion du "click gauche",
>    faites en sorte que lorsque l'utisateur effectue un "click droit"
>    sur une case non révélée, un drapeau s'affiche (on pourra utiliser
>    l'emoji :🚩 ).
>
>    *Remarque:* il faut changer le code à beaucoup d'endroits par rapport
>    aux autres points de cet atelier. Encore une fois, laissez vous
>    guider par le compilateur!


Félicitation! Vous avez un démineur quasi-complet! En revanche, notre
étape introduit la possiblité "d'états impossibles" dans notre modèle.
En effet, que faire avec une cellule qui est "révélée avec un drapeau"
(c'est à dire les attributs `revealed` et `withFlag` sont tous deux à
`True`)?

Même si dans notre code actuel cette situation ne peut pas se
produire, il faudra qu'on soit bien attentifs à ne pas introduire cet
état lorsqu'on modifiera notre code plus tard. Et comme nous somme humains,
il arrivera un moment où nous introdurons cet état vide de sens.

Pour éviter cela, il faut repenser la structure de données : notre cellule
est dans trois états possibles : masquée, révélée ou avec drapeau.
Créons donc un type reflétant cet état et reconstruisons le type
`Cell` :

```elm
type CellStatus
    = Revealed
    | WithFlag
    | Masked


type alias Cell =
    { id : Int, isMine : Bool, cellStatus : CellStatus }

```

> **\>\>\> À vous de jouer !**
>
> {:start="3"}
> 3. Remplacez le type `Cell` par celui donné ci-dessus. Faites en sorte
>    que votre code compile, tout devrait alors fonctionner !
>
>    *Remarque :* comme pour les messages, on peut filtrer par motif
>    sur les valeurs de type `CellStatus` :
>    ```elm
>    case cell.cellStatus of
>        Revealed ->
>            ...
>        WithFlag ->
>            ...
>        Masked ->
>            ...
>    ```


# Finitions

N'oubliez pas de [partager votre travail](https://annuel.framapad.org/p/atelier-elm-demineur-examples-sebbes)!

Pour avoir un jeu pleinement fonctionnel :
1. Si le joueur révèle une mine, affichez un message de défaite,
   empêchez le de continuer à jouer et révélez toutes les mines.
2. Affichez le nombre de drapeaux / nombre de mines.
3. Ajoutez un bouton pour recommencer le jeu.
3. Permettre au joueur de retirer un drapeau s'il effectue un "clic droit"
   sur un drapeau déjà placé.
4. Si un utilisateur clique sur une case n'ayant aucun voisin miné,
   révéler toute la zone sans mines (il faudra programmer une fonction
   récursive ;) ).
5. Faites en sorte d'avoir exactement 20 mines. Pour cela, créez
   une liste constituée de 20 éléments `True`, puis 80 éléments
   `False`; mélanger cette liste grâce à [au module `random-extra`](https://package.elm-lang.org/packages/elm-community/random-extra/latest/Random-List#shuffle)
   (qu'il faudra installer). Puis, au lieu de `List.range`, utiliser
   [`List.indexedMap`](https://package.elm-lang.org/packages/elm/core/latest/List#indexedMap){:target="_blank"}.
4. Ajoutez un compteur de temps. Nous allons capturer chaque
   refraîchissement de la page (c'est à dire à chaque "frame")
   afin de faire "avancer" un compteur de temps.

   Pour cela, ajouter un champ
   `elapsedTime: Float` dans le type `Model`, une variante de
   message `NewFrame Float` puis définissez:
   ```elm
   subscriptions : Model -> Sub Msg
   subscriptions model =
       Browser.Events.onAnimationFrameDelta NewFrame
   ```
   Modifiez ensuite le `main` en remplaçant
   `subscriptions = always Sub.none` par `subscriptions = subscriptions`.

   Cela a pour effet de générer un nouveau message `NewFrame deltaT` à
   chaque frame, le `deltaT` étant égal au temps écoulé depuis la frame
   précédente (exprimé en milisecondes). Vous pouvez alors intercepter
   ce message dans la fonction `update` pour incrémenter le champ
   `elapsedTime`. À vous de jouer ensuite pour afficher le temps
   "seconde par seconde".

# Aller plus loin

## Le guide officiel

C'est ici : [http://guide.elm-lang.org/](http://guide.elm-lang.org/).

Il est synthétique et reprend en profondeur les points explicités dans
cet atelier. Il est ponctué de petits exercices en fin de chaque section
pour s'exercer.

## Demander de l'aide !

Deux grandes plateformes :
* [le slack Elm](https://elmlang.herokuapp.com/) très adapté pour de
  courtes questions, ou un échange avec des programmeurs Elm expérimentés.
  Rejoignez nous sur le channel #france, posez une question sur #beginners
  et publiez vos exploits Elmiens sur #news-and-links!
* [le Discourse](https://discourse.elm-lang.org/) pour des questions plus
  poussées.


## Se retrouver

En france, il y a plusieurs "Meetup" Elm :
* à [Paris](https://www.meetup.com/fr-FR/Meetup-Elm-Paris/),
* à [Toulouse](https://www.meetup.com/fr-FR/Elm-Toulouse/),
* et à [Lyon](https://www.meetup.com/fr-FR/Elm-Lyon-Meetup/).

Paris héberge le plus gros rassemblement mondial autour de Elm :
la [conférence Elm Europe](https://2019.elmeurope.org/).

Envie d'organiser un événement autour de Elm dans votre
ville/entreprise/école d'ingé ? Venez en discuter sur Slack sur
le channel #france!
