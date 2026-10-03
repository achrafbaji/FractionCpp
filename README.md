# FractionCpp — la surcharge de constructeurs en C++

Exemple minimal illustrant **la surcharge de constructeurs**, écrit dans le cadre
d'INF3105 (UQAM, automne 2026).

Une classe peut avoir **plusieurs constructeurs portant le même nom**. Le
compilateur choisit lequel appeler d'après le **nombre** et le **type** des
arguments fournis. C'est ce que montre la classe `Fraction`.

## Les trois constructeurs

```cpp
// fraction.h
class Fraction {
public:
    Fraction();                                  // (1) aucun argument
    Fraction(int numerateur);                    // (2) un argument
    Fraction(int numerateur, int denominateur);  // (3) deux arguments
    ...
private:
    int num;
    int den;
};
```

| Appel | Constructeur choisi | Résultat |
|---|---|---|
| `Fraction a;` | (1) | `0/1` |
| `Fraction b(7);` | (2) | `7/1` |
| `Fraction c(3, 4);` | (3) | `3/4` |
| `Fraction d(1, 0);` | (3) | `1/1` — dénominateur nul corrigé |
| `Fraction e(2, -5);` | (3) | `-2/5` — signe normalisé |

C'est le **même nom** trois fois : ce qui les distingue est leur signature,
c'est-à-dire le nombre et le type des paramètres.

## Ce que chaque constructeur illustre

**(1) Le constructeur sans argument** — définit l'état par défaut de l'objet.
Ici `0/1`, parce qu'une fraction doit toujours avoir un dénominateur valide.

```cpp
Fraction::Fraction()
    : num(0), den(1)
{
}
```

**(2) Un argument** — un entier *est* une fraction de dénominateur 1.

```cpp
Fraction::Fraction(int numerateur)
    : num(numerateur), den(1)
{
}
```

**(3) Deux arguments** — le seul dont le corps `{ }` fait quelque chose :
il **valide**. Un constructeur ne doit jamais laisser naître un objet
incohérent.

```cpp
Fraction::Fraction(int numerateur, int denominateur)
    : num(numerateur), den(denominateur)
{
    if (den == 0) den = 1;                      // pas de division par zero
    if (den < 0) { num = -num; den = -den; }    // le signe reste au numerateur
}
```

## La liste d'initialisation

Le `:` après la signature n'est pas une convention d'écriture : il **construit**
les attributs avec leur valeur, au lieu de les créer vides puis de les affecter.

```cpp
: num(numerateur)      // num NAIT en valant numerateur       -> 1 etape
{ num = numerateur; }  // num nait vide, PUIS on l'affecte    -> 2 etapes
```

Pour deux `int` le résultat est identique, mais la liste devient **obligatoire**
dès qu'un attribut est `const`, est une référence, ou est un objet sans
constructeur par défaut. D'où la règle : tous les attributs passent par la liste.

## Alternative : un seul constructeur à valeurs par défaut

Les trois constructeurs ci-dessus peuvent se réduire à un seul :

```cpp
// dans le .h  (les valeurs par defaut vont ICI, et seulement ici)
Fraction(int numerateur = 0, int denominateur = 1);
```

```cpp
// dans le .cpp  (pas de "= 0" ni "= 1")
Fraction::Fraction(int numerateur, int denominateur)
    : num(numerateur), den(denominateur)
{
    if (den == 0) den = 1;
    if (den < 0) { num = -num; den = -den; }
}
```

Les trois usages continuent de fonctionner. Limite à connaître : **seuls les
derniers paramètres peuvent être omis** — on ne peut pas sauter un argument au
milieu.

## Pièges

- `Fraction a();` **ne crée pas d'objet** : le compilateur y voit la déclaration
  d'une fonction `a` sans paramètre retournant une `Fraction`. On écrit
  `Fraction a;` sans parenthèses.
- Les valeurs par défaut se déclarent **uniquement dans le `.h`**. Les répéter
  dans le `.cpp` est une erreur de compilation.
- Deux constructeurs ne peuvent pas avoir la même signature : `Fraction(int)` et
  `Fraction(int)` sont une redéfinition, pas une surcharge.

## Compilation

```bash
make
./prog
make clean
```

Sans `make` :

```bash
g++ -std=c++11 -Wall main.cpp fraction.cpp -o prog
./prog
```

## Sortie

```
0/1  = 0
7/1  = 7
3/4  = 0.75
1/1  = 1
-2/5  = -0.4
```

## Fichiers

```
fraction.h     declarations : les trois constructeurs et l'interface
fraction.cpp   definitions
main.cpp       un appel par constructeur, pour voir la selection a l'oeuvre
makefile       compilation
```
