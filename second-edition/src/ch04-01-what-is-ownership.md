## Qu'est ce que l'Ownership ?

Une fonctionalitée centrale de Rust est l'*ownership*. Bien que le concept est simple à expliquer, il a une profonde implication au reste de langage.

Tous les programmes doivent savoir gérer la mémoire pendant qu'il tourne. Plusieurs langages ont des *garbage collector* qui regardent constamment s'il n'y a pas de mémoires allouées non-utlisées pendant que le programme tourne; dans d'autres langages, les programmeurs doivent explicitement allouer et libérer la mémoire. Rust utilise une troisième approche : la mémoire est gérée par le système de l'*ownership* avec des règles données vérifier pendant la compilation. Il n'y a donc pas de coût de *runtime*.

Parce que l'*ownership* est un nouveau concept pour beaucoup de programmeurs, il faut un certain temps pour l'apprivoiser. La bonnee nouvelles est que plus vous développez avec Rust, plus vous allez être capable de développer naturellement un code qui est sûre et efficace. Keep at it !

Quand vous comprendrez le principe de l'*ownership*, vous aurez de solide fondations pour comprendre les fonctionnalités qui rendent Rust unique. Dans ce chapitre, vous apprendrai l'*ownership* en expliquant sur plusieurs exemples qui 
se base sur une structure de données familiéres : les *strings*.   

<!-- PROD: START BOX -->

> ### La Pile et le Tas
>
> Dans de nombreuses langues de programmation, nous ne devons pas penser à la pile et au tas très souvent. Mais dans un langage de programmation système comme Rust, qu'il s'agisse d'un qu’il s’agisse d’une valeur de la pile (stack en anglais) ou du tas (heap en anglais) a plus d'effet sur la façon dont le langage se comporte et c’est pourquoi nous devons prendre certaines décisions. Nous décrirons des parties de les propriétés de la pile et le tas plus tard dans ce chapitre, pour le moment, voici une brève explication pour vous préparer. La pile et le tas sont deux parties de la mémoire qui est disponible pour votre code à utiliser au moment de l'exécution, mais ils sont structurés de différentes façons. Les valeurs de la pile se stockent dans l'ordre où il les reçoit et supprime les valeurs dans l'ordre inverse. Ceci est appelé * last in, first out *. Pensez à une pile de tuiles: quand vous ajoutez plus de tuiles, vous les placez au dessus de la pile, et quand vous avez besoin d'une tuile, vous en prenez une au-dessus. Ajouter ou enlever des plaques du milieu ou à partir du bas ne fonctionnerait pas aussi bien! L'ajout de données s'appelle *pushing onto the stack*, > Et supprimer les données s’appelle *popping off the stack*. 

> La pile (stack) est rapide en raison de la façon dont elle accède aux données: elle ne doit jamais chercher un endroit où obtenir ou mettre de nouvelles données qu’elle les place toujours vers le haut. Une autre propriété qui rend la pile rapide est que toutes les données sur la pile doivent prendre une taille connue et fixe. 

> Pour les données avec une taille inconnue pour nous au moment de la compilation ou d'une taille qui pourrait varier, nous pouvons stocker nos données sur le tas (heap) à la place. À la différence de la pile, le tas est moins organisé : lorsque nous mettons des données sur le tas, nous demandons une certaine quantité d'espace. Le système d'exploitation trouve un endroit vide quelque part dans le tas qui est assez grand, le marque comme étant dans utilisable, et nous retourne un pointeur vers cet emplacement. Ce processus s'appelle *allouer sur le tas*, et parfois nous abrégeons la phrase comme par « Allocation ». Le fait de mettre des valeurs sur la pile n'est pas considéré comme alloué. Étant donné que le pointeur est d’une taille connue et fixe, nous pouvons mémoriser le pointeur sur la pile, mais lorsque nous voulons les données réelles, nous devons suivre le pointeur.

> Pensez à être assis dans un restaurant. Lorsque vous entrez, vous indiquez le nombre de personnes dans votre groupe et le personnel trouve une table vide qui s'adapte à tous puis vous conduit là-bas. Si quelqu'un dans votre groupe arrive en retard, le personnel peut dire où vous avez été assis pour vous trouver. 

> L'accès aux données dans le tas est plus lent que l'accès aux données sur la pile, car nous devons suivre un pointeur pour y arriver. Les processeurs contemporains sont plus rapides s'ils se focalisent sur de la mémoire rapprochée. En continuant l'analogie, considérez un serveur à un restaurant recevant des commandes de plusieurs tables. Il est plus efficace d'obtenir tous les commandes d’une table avant de passer à la table suivante. Prendre une commande de table A, puis une commande de table B, puis une autre de A à nouveau, puis encore une de B. Ce serait encore une fois un processus beaucoup plus lent. De la même façon, un processeur peut faire un meilleur travail si cela fonctionne sur des données proches des autres données (comme c'est le cas sur la pile) plutôt que plus espacés entre eux (comme cela peut être sur le tas). L’allocation d’une grande zone mémoire sur le tas peut-être long.

> Lorsque notre code appelle une fonction, les valeurs passées à la fonction (comprenant potentiellement les pointeurs des données vers le tas) et les variables locales de la fonction peuvent être stockées dans la pile. 

> Lorsque la fonction est terminée, ces valeurs sont apparues hors de la pile. Suivre les parties du code qui utilisent les données sur le tas, en minimisant la quantité de données en double sur le tas et le nettoyage des données inutilisées sur le tas de sorte que nous ne manquons pas d'espace sont tous des problèmes auxquels se heurte l’ownership. Une fois que vous comprendrez l’ownership, vous n'aurez pas besoin de réfléchir à la pile et à la très souvent, mais savoir ce qu’est la gestion des données des tas et pourquoi l’ownership existe peut aider à expliquer pourquoi il fonctionne comme il le fait. 
>
<!-- PROD: END BOX -->

### Les règles de l'Ownership

Tout d'abord, regardons les règles de l’ownership. Gardez ces règles à l'esprit lorsque nous verrons des exemples plus tard :

> 1. Chaque valeur dans Rust a une variable qui est appelée son *propriétaire* 
> 2. Il ne peut y avoir qu'un propriétaire à la fois. 
> 3. Lorsque le propriétaire sort de la portée, la valeur sera abandonnée. 

### Le Scope des Variables

Nous avons déjà parcouru un exemple d'un programme Rust dans le chapitre 2. Maintenant que nous sommes passé une syntaxe de base, nous n’inclurons pas tous les `fn main() {` code dans les exemples, donc si vous suivez, vous Il faudra mettre manuellement les exemples suivants dans une fonction `main`. Par conséquent, nos exemples seront un peu plus concis, nous permettant de nous concentrer sur les détails plutôt que sur le code global. 

Comme premier exemple de l’ownership, nous examinerons le *scope* (ou la portée) de certaines variables. Le scope est une zone d'un programme où un élément est valide. Disons que nous avons une variable qui ressemble à ceci: 

```rust
let s = "hello";
```

La variable `s` réfère à une chaîne littérale, où la valeur de la chaîne est codée en dur dans le texte de notre programme. La variable est valable du point où elle est déclarée jusqu'à la fin de la portée actuelle. La liste 4-1 a des commentaires indiquant jusqu’où la variable `s` est valide: 

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

<span class="caption">Listing 4-1: A variable and the scope in which it is
valid</span>

En d'autres termes, il existe deux points importants :

1. Lorsque `s` entre * dans le scope*, il est valide.
1. Il est valide jusqu'à ce qu'il *sorte du scope*.

À ce stade, la relation entre le scope et quand les variables sont valides est similaire à d'autres langages de programmation. Maintenant, nous allons rajouter une couche par-dessus en s’intéressant au type `String`. 

### Le type `String`

Pour illustrer les règles de l’ownership, nous avons besoin d'un type de données plus complexe que celui que nous avons abordé dans le chapitre 3. Tous les types de données que nous avons précédemment examinés sont stockés dans la pile et sont extraient de la pile lorsque leur scope est terminé, mais nous voulons regarder les données qui sont stockées dans le tas et comprendre comment Rust sait quand nettoyer ces données. 

Nous utiliserons ici `String` comme exemple et nous nous concentrerons sur les parties de `String` qui concernent l’ownership. Ces aspects s'appliquent également à d'autres types de données complexes fournis par la bibliothèque standard et ceux que vous créez. Nous aborderons `String` plus en profondeur le chapitre 8. 

Nous avons déjà vu les chaînes de littérales, où une valeur de chaîne est codée en dur dans notre programme. Les littérales de chaînes sont pratiques, mais elles ne conviennent pas toujours à toutes les situations dans lesquelles vous souhaitez utiliser le texte. L'une des raisons est qu'ils sont immuables. Une autre est que toutes les valeurs de chaîne peuvent ne pas être connues lorsque nous écrivons notre code : par exemple, que faire si nous voulons prendre une entrée de l'utilisateur et l'enregistrer ? Pour ces situations, Rust a un second type de chaîne, `String`. Ce type est alloué sur le tas et, en tant que tel, est capable de stocker une quantité de texte qui nous est inconnue au moment de la compilation. Vous pouvez créer un `String` à partir d'une littérale de chaîne à l'aide de la fonction `from`, de la manière suivante : 

```rust
let s = String::from("hello");
```

Le double colon (`::`) est un opérateur qui nous permet de nommer cette fonction particulière `from` sous le type de `String` plutôt que d'utiliser une sorte de nom comme `string_from`. Nous étudierons cette syntaxe dans la section « Les méthodes et leur syntaxe » du chapitre 5 et lorsque nous parlerons des namespaces avec les modules du chapitre 7. 

Ce type de chaîne *peut* être mutable : 

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print `hello, world!`
```

Alors, quelle est la différence ici ? Pourquoi `String` peut-il être mutable mais les littérales ne le peuvent-ils pas? La différence est la façon dont ces deux types utilisent la mémoire. 

### La mémoire et l'allocation

Dans le cas d'un littéral de chaîne, nous connaissons le contenu au moment de la compilation, de sorte que le texte est directement codé directement dans l'exécutable final, ce qui rend les littérales de chaînes rapides et efficaces. Mais ces propriétés proviennent uniquement de leur immuabilité. Malheureusement, nous ne pouvons pas mettre un bloc de mémoire dans le binaire pour chaque texte dont la taille est inconnue au moment de la compilation et dont la taille peut changer pendant l'exécution du programme. 

Avec le type `String`, afin de supporter un morceau de texte mutable et utilisable, nous devons allouer une quantité de mémoire dans le tas, inconnue au moment de la compilation, pour contenir le contenu. Cela signifie que : 

1. La mémoire doit être demandée du système d'exploitation au moment de l'exécution. 
2. Nous avons besoin de renvoyer cette mémoire sur le système d'exploitation lorsque nous avons fini avec notre type `String`. 

Cette première partie est faite par nous : lorsque nous appelons `String::from`, son implémentation sert à demander la mémoire dont elle a besoin. Ceci est pratiquement universel dans les langages de programmation. 

Cependant, la deuxième partie est différente. Dans des langues avec un garbage collector (GC), le GC garde une trace et nettoie la mémoire qui n'est plus utilisée, et nous, en tant que programmateur, n'avons pas besoin de penser à cela. Sans GC, c’est à la responsabilité du programmeur de déterminer si la mémoire n'est plus utilisée et d’appeler explicitement le code pour la renvoyer, comme nous l'avons fait pour le demander. Faire cela correctement a toujours été un problème de programmation difficile. Si on oublie, nous perdrons de la mémoire. Si nous le faisons trop tôt, nous aurons une variable invalide. Si nous le faisons deux fois, il y aura un bug. Nous devons coupler une `allocate` avec un `free`. 

Rust prend un chemin différent : la mémoire est automatiquement renvoyée une fois que la variable qui la possède sort du scope. Voici une version de notre exemple du scope de la liste 4-1 à l'aide d'un `String` au lieu d'une chaîne littérale : 

```rust
{
    let s = String::from("hello"); // s is valid from this point forward

    // do stuff with s
}                                  // this scope is now over, and s is no
                                   // longer valid
```

Il existe un point naturel sur lequel nous pouvons désallouer la mémoire que notre `String` a besoin pour le système d'exploitation : lorsque `s` sort du scope. Lorsqu'une variable sort du scope, Rust appelle une fonction spéciale pour nous. Cette fonction s'appelle `drop`, et c'est là que l'auteur de `String` peut mettre le code pour renvoyer la mémoire. Rust appelle `drop` automatiquement à la fermeture de `}`. 

> Note: In C++, this pattern of deallocating resources at the end of an item's
> lifetime is sometimes called *Resource Acquisition Is Initialization (RAII)*.
> The `drop` function in Rust will be familiar to you if you’ve used RAII
> patterns.

This pattern has a profound impact on the way Rust code is written. It may seem
simple right now, but the behavior of code can be unexpected in more
complicated situations when we want to have multiple variables use the data
we’ve allocated on the heap. Let’s explore some of those situations now.

#### Intéraction entre les Données et les Variables : le Déplacement

Plusieurs variables peuvent interagir avec les mêmes données de différentes façons dans Rust. Regardons un exemple en utilisant un entier de la liste 4-2

```rust
let x = 5;
let y = x;
```

<span class="caption">Liste 4-2: Assigné la valeur de `x`à `y`</span>

Nous pouvons probablement deviner ce que cela fait en fonction de notre expérience avec d'autres langues : "Affecter la valeur `5` à `x` ; Puis faire une copie de la valeur de `x` et la lier à `y`. "Nous avons maintenant deux variables, `x` et `y`, et les deux sont égales à `5` . Et c'est en effet ce qui se passe car les nombres entiers sont des valeurs simples avec une taille connue et fixe, et ces deux valeurs `5` sont stockées dans la pile. 

Voyons maintenant la version avec `String` :

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Cela ressemble beaucoup au code précédent, donc nous pouvons supposer que la façon dont il fonctionnerait serait la même : c'est-à-dire que la deuxième ligne rendrait une copie de la valeur dans `s1` et la lierait à `s2`. Mais ce n'est pas tout à fait ce qui se passe. 

Pour expliquer cela plus en profondeur, regardons ce à quoi ressemble un `String` dans la Figure 4-3. Un `String` est composé de trois parties, illustrées à gauche : un pointeur vers la mémoire hébergeant le contenu de `String`, une longueur et une capacité. Ce groupe de données est stocké sur la pile. À droite se trouve la mémoire sur le tas qui héberge le contenu. 

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Représentation dans la mémoire d'un `String`
hébergeant la valeur `"hello"` rattachée à `s1`</span>

La longueur est la quantité de mémoire, en octets, que le contenu du `String` est en train d’utiliser. La capacité est la quantité totale de mémoire, en octets, que `String` a reçue du système d'exploitation. La différence entre la longueur et la capacité importe, mais pas dans ce contexte, alors, pour l'instant, c’est mieux d'ignorer la capacité. 

Lorsque nous assignons `s1` à `s2`, la chaîne de caractère `String` est copiée, ce qui signifie que nous copions le pointeur, la longueur et la capacité qui sont sur la pile. Nous ne copions pas les données stockées sur le tas à qui le pointeur se réfère. En d'autres termes, la représentation de données en mémoire ressemble à la Figure 4-4. 

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Réprésentation dans la mémoire de la variable `s2`
qui a une copie du pointeur, de la longueur et de la capacité `s1`</span>

La représentation ne ressemble *pas* à la Figure 4-5, à quoi ressemblerait la mémoire si Rust avait plutôt copié les données stockées dans le tas (qui se dit « heap » en anglais, pour rappelle). Si Rust avait fait cela, l'opération `s2 = s1` pourrait être très coûteuse en termes de performances d'exécution si les données sur le tas étaient importantes. 

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-5: Une autre possibilité de ce que `s2 = s1` Rust aurait pu faire si les données du tas</span>

Auparavant, nous avons dit que lorsqu'une variable va en dehors du scope, Rust appelle automatiquement la fonction de `drop` et nettoie la mémoire du tas pour cette variable. Mais la figure 4-4 montre les deux pointeurs de données pointant vers le même emplacement. C'est un problème: lorsque `s2` et `s1` sortent de la portée, ils essayeront tous deux de libérer la même mémoire. Ceci est connu comme une erreur de *double free* et est l'un des bugs de sécurité de mémoire que nous avons mentionnés précédemment. Libérer de la mémoire deux fois peut entraîner une corruption de mémoire, ce qui peut potentiellement entraîner des vulnérabilités de sécurité. 

Pour assurer la sécurité de la mémoire, il y a un autre détail qui se passe dans cette situation dans Rust. Au lieu d'essayer de copier la mémoire allouée, Rust considère que `s1` n'est plus valable et, par conséquent, Rust n'a pas besoin de tout libérer lorsque `s1` sort de la portée. Découvrez ce qui se passe lorsque vous essayez d'utiliser `s1` après la création de `s2` : 

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s1);
```

Vous obtiendrez une erreur comme celle-ci, car Rust vous empêche d'utiliser la référence invalide :

```text
error[E0382]: use of moved value: `s1`
 --> src/main.rs:4:27
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`,
which does not implement the `Copy` trait
```

Si vous avez entendu les termes «copie superficielle» et «copie profonde» en travaillant avec d'autres langages de programmations, le concept de copier le pointeur, la longueur et la capacité sans copier les données ressemble probablement à une copie superficielle. Mais parce que Rust invalide également la première variable, au lieu d'appeler cette copie superficielle, elle est connue comme un mouvement. Ici, nous lisons cela en disant que `s1` été *déplacé* dans `s2`. Donc, voilà ce qui se passe réellement figure à la figure 4-6. 

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-6: Réprésentation dans la mémoire de `s1` après invalidation</span>

Cela résout notre problème ! Avec seulement `s2` valide, quand il sort du scope, il sera le seul à subir une dés-allocation, et nous avons finis. 

De plus, il y a un choix de conception qui est impliqué par ça : Rust ne créera jamais automatiquement de copies « profondes » de vos données. Par conséquent, toute copie *automatique* peut être considérée comme peu coûteuse en termes de performances d'exécution. 

#### Intéraction entre les Données et les Variables: le Clonage

Si nous voulons copier *profondément* les données de `String` stockées dans le tas, pas seulement les données de la pile, nous pouvons utiliser une méthode commune appelée `clone`. Nous allons discuter de la syntaxe de la méthode au chapitre 5, mais parce que les méthodes sont une caractéristique commune dans de nombreuses langues de programmation, vous les avez probablement vus auparavant. 

Voici un exemple de la méthode `clone` en action :

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Cela fonctionne très bien et c'est la façon dont vous pouvez produire explicitement le comportement indiqué dans la Figure 4-5, où les données sur le tas sont copiées.

Lorsque vous voyez un appel de `clone`, vous savez qu'un code arbitraire est en cours d'exécution et que ce code peut être coûteux. C'est un indicateur visuel que quelque chose de différent se passe. 

#### Récupération des Données de la Pile seulement : la Copie

Il y a encore une autre ruse dont nous n'avons pas encore parlé. Ce code utilisant des nombres entiers, dont une partie a été montré plus haut dans la liste 4-2, ce code fonctionne et est valide : 

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

Mais ce code semble contredire ce que nous venons d'apprendre: nous n'avons pas d'appel à `clone`, mais `x` est toujours valide et n'a pas été déplacé dans `y`. 

La raison en est que les types comme les entiers qui ont une taille connue au moment de la compilation sont entièrement stockés sur la pile, donc les copies des valeurs réelles sont rapides à réaliser. Cela signifie qu'il n'y a aucune raison que nous voudrions empêcher que `x` soit valide après la création de la variable `y`. En d'autres termes, il n'y a pas de différence entre les copies profondes et peu profondes ici, alors appeler le `clone` ne fera rien de différent de la copie superficielle habituelle donc nous pouvons le laisser tomber. 

Rust a une annotation spéciale appelée *trait* `Copy` que nous pouvons placer sur des types comme des entiers qui sont stockés sur la pile (nous parlerons plus des traits au chapitre 10). Si un type a le trait `Copy`, une ancienne variable est toujours utilisable après affectation. Rust ne nous permettra pas d'annoter un type avec le trait `Copy` si le type ou l'une de ses parties a implémenté le trait `Drop`. Si ce type a besoin d’une opération particuliére lorsque la valeur sort du scope et que nous ajoutons l'annotation `Copy` à ce type, nous obtiendrons une erreur de compilation. Pour en savoir plus sur l'ajout de l'annotation `Copy` à votre type, voir l'annexe C dans la partie « Derivable Traits ». 

Alors, quels types sont annotés `Copy` ? Vous pouvez vérifier la documentation pour le type donné pour être sûr, mais en règle générale, tout groupe de valeurs scalaires simples peut être annoté `Copy`, et rien qui nécessite une allocation ou une forme de ressource est annotés `Copy`. Voici quelques-uns des types qui sont annotés `Copy` : 

* Tous les types entiers, comme `u32`. 
* Le type booléen, `bool`, avec des valeurs `true` et `false`. 
* Tous les types de virgule flottante, comme `f64`. 
* Les tuples, mais seulement s'ils contiennent des types qui sont également des `Copy`. `(i32, i32)` est `Copy` , mais `(i32, String)` ne l'est pas.

### L'Ownership et les fonctions

La sémantique pour passer une valeur à une fonction est similaire à attribuer une valeur à une variable. Passer une variable à une fonction se déplacera ou se copiera, tout comme une affectation. La liste 4-7 a un exemple avec quelques annotations montrant où les variables entrent et sortent du scope : 

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope.

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here.
    let x = 5;                      // x comes into scope.

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward.

} // Here, x goes out of scope, then s. But since s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope.
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope.
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

<span class="caption">Listing 4-7: Fonctions avec ownership et le scope
annotés</span>

Si nous essayions d'utiliser `s` après l'appel à `takes_ownership`, Rust renverra une erreur de compilation. Ces contrôles statiques nous protègent contre les erreurs. Essayez d'ajouter un code au `main` qui utilise `s` et `x` pour voir où vous pouvez les utiliser et où les règles de l’ownership vous les empêche de le faire. 

### Retourner les valeurs et le scope

Les valeurs de retour peuvent également transférer l’ownership. Voici un exemple avec des annotations similaires à celles de Listing 4-7 : 

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1.

    let s2 = String::from("hello");     // s2 comes into scope.

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3.
} // Here, s3 goes out of scope and is dropped. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it.

    let some_string = String::from("hello"); // some_string comes into scope.

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function.
}

// takes_and_gives_back will take a String and return one.
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope.

    a_string  // a_string is returned and moves out to the calling function.
}
```

L’ownership d'une variable suit le même modèle à chaque fois : l'attribution d'une valeur à une autre variable la déplace. Lorsqu'une variable qui inclut des données sur le tas dépasse la portée, la valeur sera nettoyée par `drop` à moins que les données n'aient été déplacées pour être stockées à une autre variable. 

Prendre possession et ensuite la renvoyer avec chaque fonction est un peu fastidieux. Que faire si nous voulons laisser une fonction utiliser une valeur mais ne pas en prendre possession ? Il est ennuyeux que tout ce que nous passons doit également être renvoyé si nous voulons l'utiliser à nouveau, en plus de toute donnée résultant du corps de la fonction que nous pourrions vouloir retourner. 

Il est possible de renvoyer plusieurs valeurs à l'aide d'un tuple, comme ceci:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String.

    (s, length)
}
```

Mais c'est beaucoup trop de travail pour un concept qui devrait être commun. Heureusement pour nous, Rust a une fonctionnalité pour ce concept qu’on appelle des *références*. 
