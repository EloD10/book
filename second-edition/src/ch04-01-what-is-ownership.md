## What Is Ownership?

Une fonctionalité centrale de Rust est l'*ownership*. Bien que le concept est simple à expliqué, il a une profonde implications au reste de langage.

Tous les programmes doivent savoir gérer la mémoire pendant qu'il tourne. Plusieurs langages ont des *garbage collector* qui regardent constamment s'il n'y a pas de mémoire allouées non-utlisées pendant que le programme tourne; dans d'autres langages, les programmeurs doit explicitement allouer et libérer la mémoire. Rust utilise une troisième approche : la mémoire est gérée par le système de l'*ownership* avec des règles données vérifier pendant la compilation. Il n'y a donc pas de coût de *runtime*.

Parce que l'*ownership* est un nouveau concept pour beaucoup de programmeurs, il faut un certain temps pour l'apprivoiser. La bonnee nouvelles est que plus vous développez avec Rust, plus vous allez être capable de développez naturellement un code qui est sûre et efficace. Keep at it !

Quand vous comprendrez la principe de l'*ownership*, vous aurez de solide fondations pour comprendre les fonctionnalités qui rendent Rust unique. Dans ce chapitre, vous apprendrai l'*ownership* en expliquant sur plusieurs exemples qui 
se base sur une structure de données familiéres : les *strings*.   

<!-- PROD: START BOX -->

> ### The Stack and the Heap
>
> Dans de nombreuses langues de programmation, nous ne devons pas penser à la pile et au tas très souvent. Mais dans un langage de programmation système comme Rust, qu'il s'agisse d'un qu’il s’agisse d’un valeur de la pile (stack en anglais) ou du tas (heap en anglais) a plus d'effet sur la façon dont le langage se comporte et c’est pourquoi nous devons prendre certaines décisions. Nous décrirons des parties de les propriétés de la pile et le tas plus tard dans ce chapitre, pour le moment, voici une brève explication pour vous préparer. La pile et le tas sont deux parties de la mémoire qui est disponible pour votre code à utiliser au moment de l'exécution, mais ils sont structurés de différentes façons. Les valeurs de la pile se stockent dans l'ordre où il les reçoit et supprime les valeurs dans l'ordre inverse. Ceci est appelé * last in, first out *. Pensez à une pile de tuiles: quand vous ajoutez plus de tuiles, vous les placez au dessus de la pile, et quand vous avez besoin d'une tuile, vous en prenez une au dessus. Ajouter ou enlever des plaques du milieu ou à partir du bas ne fonctionnerait pas aussi bien! L'ajout de données s'appelle *pushing onto the stack*, > Et supprimer les données s’appelle *popping off the stack*. 

> La pile (stack) est rapide en raison de la façon dont elle accède aux données: elle ne doit jamais chercher un endroit où obtenir ou mettre de nouvelles données qu’elle les place toujours vers le haut. Une autre propriété qui rend la pile rapide est que toutes les données sur la pile doivent prendre une taille connue et fixe. 

> Pour les données avec une taille inconnue pour nous au moment de la compilation ou d'une taille qui pourrait varier, nous pouvons stocker nos données sur le tas (heap) à la place. A la différence de la pile, le tas est moins organisé : lorsque nous mettons des données sur le tas, nous demandons une certaine quantité d'espace. Le système d'exploitation trouve un endroit vide quelque part dans le tas qui est assez grand, le marque comme étant dans utilisable, et nous retourne un pointeur vers cet emplacement. Ce processus s'appelle *allouer sur le tas*, et parfois nous abrégeons la phrase comme par « Allocation ». Le fait de mettre des valeurs sur la pile n'est pas considéré comme alloué. Étant donné que le pointeur est d’une taille connue et fixe, nous pouvons mémoriser le pointeur sur la pile, mais lorsque nous voulons les données réelles, nous devons suivre le pointeur.

> Pensez à être assis dans un restaurant. Lorsque vous entrez, vous indiquez le nombre de personnes dans votre groupe et le personnel trouve une table vide qui s'adapte à tous puis vous conduit là-bas. Si quelqu'un dans votre groupe arrive en retard, le personnel peut dire où vous avez été assis pour vous trouver. 

> L'accès aux données dans le tas est plus lent que l'accès aux données sur la pile, car nous devons suivre un pointeur pour y arriver. Les processeurs contemporains sont plus rapides s'ils se focalisent sur de la mémoires rapprochées. En continuant l'analogie, considérez un serveur à un restaurant recevant des commandes de plusieurs tables. Il est plus efficace d'obtenir tous les commandes d’une table avant de passer à la table suivante. Prendre une commande de table A, puis une commande de table B, puis une autre de A à nouveau, puis encore une de B. Ce serait encore une fois un processus beaucoup plus lent. De la même façon, un processeur peut faire un meilleur travail si cela fonctionne sur des données proches des autres données (comme c'est le cas sur la pile) plutôt que plus espacés entre eux (comme cela peut être sur le tas). L’allocation d’une grande zone mémoire sur le tas peut-être long.

> Lorsque notre code appelle une fonction, les valeurs passées à la fonction (comprenant potentiellement les pointeurs des données vers le tas) et les variables locales de la fonction peuvent être stockées dans la pile. 

> Lorsque la fonction est terminée, ces valeurs sont apparues hors de la pile. Suivre les parties du code qui utilisent les données sur le tas, en minimisant la quantité de données en double sur le tas et le nettoyage des données inutilisées sur le tas de sorte que nous ne manquons pas d'espace sont tous des problèmes auxquels se heurte l’ownership. Une fois que vous comprendrez l’ownership, vous n'aurez pas besoin de réfléchir à la pile et à la très souvent, mais savoir ce qu’est la gestion des données des tas et pourquoi l’ownership existe peut aider à expliquer pourquoi il fonctionne comme il le fait. 
>
<!-- PROD: END BOX -->

### Ownership Rules

First, let’s take a look at the ownership rules. Keep these rules in mind as we
work through the examples that illustrate the rules:

> 1. Each value in Rust has a variable that’s called its *owner*.
> 2. There can only be one owner at a time.
> 3. When the owner goes out of scope, the value will be dropped.

### Variable Scope

We’ve walked through an example of a Rust program already in Chapter 2. Now
that we’re past basic syntax, we won’t include all the `fn main() {` code in
examples, so if you’re following along, you’ll have to put the following
examples inside a `main` function manually. As a result, our examples will be a
bit more concise, letting us focus on the actual details rather than
boilerplate code.

As a first example of ownership, we’ll look at the *scope* of some variables. A
scope is the range within a program for which an item is valid. Let’s say we
have a variable that looks like this:

```rust
let s = "hello";
```

The variable `s` refers to a string literal, where the value of the string is
hardcoded into the text of our program. The variable is valid from the point at
which it’s declared until the end of the current *scope*. Listing 4-1 has
comments annotating where the variable `s` is valid:

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

<span class="caption">Listing 4-1: A variable and the scope in which it is
valid</span>

In other words, there are two important points in time here:

1. When `s` comes *into scope*, it is valid.
1. It remains so until it goes *out of scope*.

At this point, the relationship between scopes and when variables are valid is
similar to other programming languages. Now we’ll build on top of this
understanding by introducing the `String` type.

### The `String` Type

To illustrate the rules of ownership, we need a data type that is more complex
than the ones we covered in Chapter 3. All the data types we’ve looked at
previously are stored on the stack and popped off the stack when their scope is
over, but we want to look at data that is stored on the heap and explore how
Rust knows when to clean up that data.

We’ll use `String` as the example here and concentrate on the parts of `String`
that relate to ownership. These aspects also apply to other complex data types
provided by the standard library and that you create. We’ll discuss `String` in
more depth in Chapter 8.

We’ve already seen string literals, where a string value is hardcoded into our
program. String literals are convenient, but they aren’t always suitable for
every situation in which you want to use text. One reason is that they’re
immutable. Another is that not every string value can be known when we write
our code: for example, what if we want to take user input and store it? For
these situations, Rust has a second string type, `String`. This type is
allocated on the heap and as such is able to store an amount of text that is
unknown to us at compile time. You can create a `String` from a string literal
using the `from` function, like so:

```rust
let s = String::from("hello");
```

The double colon (`::`) is an operator that allows us to namespace this
particular `from` function under the `String` type rather than using some sort
of name like `string_from`. We’ll discuss this syntax more in the “Method
Syntax” section of Chapter 5 and when we talk about namespacing with modules in
Chapter 7.

This kind of string *can* be mutated:

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print `hello, world!`
```

So, what’s the difference here? Why can `String` be mutated but literals
cannot? The difference is how these two types deal with memory.

### Memory and Allocation

In the case of a string literal, we know the contents at compile time so the
text is hardcoded directly into the final executable, making string literals
fast and efficient. But these properties only come from its immutability.
Unfortunately, we can’t put a blob of memory into the binary for each piece of
text whose size is unknown at compile time and whose size might change while
running the program.

With the `String` type, in order to support a mutable, growable piece of text,
we need to allocate an amount of memory on the heap, unknown at compile time,
to hold the contents. This means:

1. The memory must be requested from the operating system at runtime.
2. We need a way of returning this memory to the operating system when we’re
done with our `String`.

That first part is done by us: when we call `String::from`, its implementation
requests the memory it needs. This is pretty much universal in programming
languages.

However, the second part is different. In languages with a *garbage collector
(GC)*, the GC keeps track and cleans up memory that isn’t being used anymore,
and we, as the programmer, don’t need to think about it. Without a GC, it’s the
programmer’s responsibility to identify when memory is no longer being used and
call code to explicitly return it, just as we did to request it. Doing this
correctly has historically been a difficult programming problem. If we forget,
we’ll waste memory. If we do it too early, we’ll have an invalid variable. If
we do it twice, that’s a bug too. We need to pair exactly one `allocate` with
exactly one `free`.

Rust takes a different path: the memory is automatically returned once the
variable that owns it goes out of scope. Here’s a version of our scope example
from Listing 4-1 using a `String` instead of a string literal:

```rust
{
    let s = String::from("hello"); // s is valid from this point forward

    // do stuff with s
}                                  // this scope is now over, and s is no
                                   // longer valid
```

There is a natural point at which we can return the memory our `String` needs
to the operating system: when `s` goes out of scope. When a variable goes out
of scope, Rust calls a special function for us. This function is called `drop`,
and it’s where the author of `String` can put the code to return the memory.
Rust calls `drop` automatically at the closing `}`.

> Note: In C++, this pattern of deallocating resources at the end of an item's
> lifetime is sometimes called *Resource Acquisition Is Initialization (RAII)*.
> The `drop` function in Rust will be familiar to you if you’ve used RAII
> patterns.

This pattern has a profound impact on the way Rust code is written. It may seem
simple right now, but the behavior of code can be unexpected in more
complicated situations when we want to have multiple variables use the data
we’ve allocated on the heap. Let’s explore some of those situations now.

#### Ways Variables and Data Interact: Move

Multiple variables can interact with the same data in different ways in Rust.
Let’s look at an example using an integer in Listing 4-2:

```rust
let x = 5;
let y = x;
```

<span class="caption">Listing 4-2: Assigning the integer value of variable `x`
to `y`</span>

We can probably guess what this is doing based on our experience with other
languages: “Bind the value `5` to `x`; then make a copy of the value in `x` and
bind it to `y`.” We now have two variables, `x` and `y`, and both equal `5`.
This is indeed what is happening because integers are simple values with a
known, fixed size, and these two `5` values are pushed onto the stack.

Now let’s look at the `String` version:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

This looks very similar to the previous code, so we might assume that the way
it works would be the same: that is, the second line would make a copy of the
value in `s1` and bind it to `s2`. But this isn’t quite what happens.

To explain this more thoroughly, let’s look at what `String` looks like under
the covers in Figure 4-3. A `String` is made up of three parts, shown on the
left: a pointer to the memory that holds the contents of the string, a length,
and a capacity. This group of data is stored on the stack. On the right is the
memory on the heap that holds the contents.

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Representation in memory of a `String`
holding the value `"hello"` bound to `s1`</span>

The length is how much memory, in bytes, the contents of the `String` is
currently using. The capacity is the total amount of memory, in bytes, that the
`String` has received from the operating system. The difference between length
and capacity matters, but not in this context, so for now, it’s fine to ignore
the capacity.

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the
pointer, the length, and the capacity that are on the stack. We do not copy the
data on the heap that the pointer refers to. In other words, the data
representation in memory looks like Figure 4-4.

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Representation in memory of the variable `s2`
that has a copy of the pointer, length, and capacity of `s1`</span>

The representation does *not* look like Figure 4-5, which is what memory would
look like if Rust instead copied the heap data as well. If Rust did this, the
operation `s2 = s1` could potentially be very expensive in terms of runtime
performance if the data on the heap was large.

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-5: Another possibility of what `s2 = s1` might
do if Rust copied the heap data as well</span>

Earlier, we said that when a variable goes out of scope, Rust automatically
calls the `drop` function and cleans up the heap memory for that variable. But
Figure 4-4 shows both data pointers pointing to the same location. This is a
problem: when `s2` and `s1` go out of scope, they will both try to free the
same memory. This is known as a *double free* error and is one of the memory
safety bugs we mentioned previously. Freeing memory twice can lead to memory
corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, there’s one more detail to what happens in this
situation in Rust. Instead of trying to copy the allocated memory, Rust
considers `s1` to no longer be valid and therefore, Rust doesn’t need to free
anything when `s1` goes out of scope. Check out what happens when you try to
use `s1` after `s2` is created:

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s1);
```

You’ll get an error like this because Rust prevents you from using the
invalidated reference:

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

If you’ve heard the terms “shallow copy” and “deep copy” while working with
other languages, the concept of copying the pointer, length, and capacity
without copying the data probably sounds like a shallow copy. But because Rust
also invalidates the first variable, instead of calling this a shallow copy,
it’s known as a *move*. Here we would read this by saying that `s1` was *moved*
into `s2`. So what actually happens is shown in Figure 4-6.

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-6: Representation in memory after `s1` has been
invalidated</span>

That solves our problem! With only `s2` valid, when it goes out of scope, it
alone will free the memory, and we’re done.

In addition, there’s a design choice that’s implied by this: Rust will never
automatically create “deep” copies of your data. Therefore, any *automatic*
copying can be assumed to be inexpensive in terms of runtime performance.

#### Ways Variables and Data Interact: Clone

If we *do* want to deeply copy the heap data of the `String`, not just the
stack data, we can use a common method called `clone`. We’ll discuss method
syntax in Chapter 5, but because methods are a common feature in many
programming languages, you’ve probably seen them before.

Here’s an example of the `clone` method in action:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

This works just fine and is how you can explicitly produce the behavior shown
in Figure 4-5, where the heap data *does* get copied.

When you see a call to `clone`, you know that some arbitrary code is being
executed and that code may be expensive. It’s a visual indicator that something
different is going on.

#### Stack-Only Data: Copy

There’s another wrinkle we haven’t talked about yet. This code using integers,
part of which was shown earlier in Listing 4-2, works and is valid:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

But this code seems to contradict what we just learned: we don’t have a call to
`clone`, but `x` is still valid and wasn’t moved into `y`.

The reason is that types like integers that have a known size at compile time
are stored entirely on the stack, so copies of the actual values are quick to
make. That means there’s no reason we would want to prevent `x` from being
valid after we create the variable `y`. In other words, there’s no difference
between deep and shallow copying here, so calling `clone` wouldn’t do anything
differently from the usual shallow copying and we can leave it out.

Rust has a special annotation called the `Copy` trait that we can place on
types like integers that are stored on the stack (we’ll talk more about traits
in Chapter 10). If a type has the `Copy` trait, an older variable is still
usable after assignment. Rust won’t let us annotate a type with the `Copy`
trait if the type, or any of its parts, has implemented the `Drop` trait. If
the type needs something special to happen when the value goes out of scope and
we add the `Copy` annotation to that type, we’ll get a compile time error. To
learn about how to add the `Copy` annotation to your type, see Appendix C on
Derivable Traits.

So what types are `Copy`? You can check the documentation for the given type to
be sure, but as a general rule, any group of simple scalar values can be
`Copy`, and nothing that requires allocation or is some form of resource is
`Copy`. Here are some of the types that are `Copy`:

* All the integer types, like `u32`.
* The boolean type, `bool`, with values `true` and `false`.
* All the floating point types, like `f64`.
* Tuples, but only if they contain types that are also `Copy`. `(i32, i32)` is
`Copy`, but `(i32, String)` is not.

### Ownership and Functions

The semantics for passing a value to a function are similar to assigning a
value to a variable. Passing a variable to a function will move or copy, just
like assignment. Listing 4-7 has an example with some annotations showing where
variables go into and out of scope:

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

<span class="caption">Listing 4-7: Functions with ownership and scope
annotated</span>

If we tried to use `s` after the call to `takes_ownership`, Rust would throw a
compile time error. These static checks protect us from mistakes. Try adding
code to `main` that uses `s` and `x` to see where you can use them and where
the ownership rules prevent you from doing so.

### Return Values and Scope

Returning values can also transfer ownership. Here’s an example with similar
annotations to those in Listing 4-7:

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

The ownership of a variable follows the same pattern every time: assigning a
value to another variable moves it. When a variable that includes data on the
heap goes out of scope, the value will be cleaned up by `drop` unless the data
has been moved to be owned by another variable.

Taking ownership and then returning ownership with every function is a bit
tedious. What if we want to let a function use a value but not take ownership?
It’s quite annoying that anything we pass in also needs to be passed back if we
want to use it again, in addition to any data resulting from the body of the
function that we might want to return as well.

It’s possible to return multiple values using a tuple, like this:

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

But this is too much ceremony and a lot of work for a concept that should be
common. Luckily for us, Rust has a feature for this concept, and it’s called
*references*.
