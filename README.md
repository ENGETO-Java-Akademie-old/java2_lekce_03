<p align="center">
  <img src="https://engeto.cz/wp-content/uploads/2019/01/engeto-square.png" width="200" height="200">
</p>

# Java 2 - 3. lekce


# Java 2 - Lekce 03

V třetí lekci se zaměříme na novinku přidanou v Javě 8 - stream API v kolekcích a
porovnáme si je s “klasickým” přístupem pomocí cyklu. Abychom je mohli efektivně
používat, tak si představíme koncepty jako lambda, method reference a functional
interface, které Java má na podporu funkcionálního programová

## Základ Funkcionálního programování

Při vzniku jazyka Java nastal velký posun hlavního proudu programování k Objektovému paradigmatu, ale v poslední době vzrostla popularita Funkcionálního paradigmatu. Ve světe JVM se to nejdříve úkazalo na populárních alternativách jako Scala (https://www.scala-lang.org/) nebo Closure (https://clojure.org/), ale později se určité principy dostaly i do Javy 8 a dále.

### Pure function
Čistá funkce musí splňovat dvě základní podmínky:

- Stejný vstup pokaždé vrátí stejný výsledek
- Žádné vedlejší efekty (změna stavu aplikace nebo změna argumentů)
```
y = f(x) * f(x);
```
```
Double impureFunction(Double value){
  return Math.random() * value;
}
Double pureFunc(Double value){
  return value * value;
}
```

Na první pohled to vypadá jednoduše, proč nepsat všechno jako čisté funkce. Bohužel mnoho problémů je komplexnějších a např. volání API nebo komunikace s DB porušuje principy čisté funkce. Řešením je často navrhnout program, tak aby rozumně kombinoval oba přístupy.

### Referential tranparency

Kvůli principu čisté funkce, je možné zapamatovat si jednou zavolaný výsledek a nahradit volání funkce rovnou výsledkem.

### Immutability

Změna datových struktur je náchylná na porušení principu čisté funkce, proto je dobré změnit princip práce s datovými strukury. Jednou vytvořená datová struktura již nejde dál modifikovat. Pokud je potřeba strukturu upravit, tak je možné vytvořit novou strukturu a místo původní struktury použít novou.

Podobný princip je už v Javě použit při práci se Stringy. Jednou vytvořený String již nejde změnit, jenom nahradit novou hodnotou.

###  Rekurze (Recursion)

Princip volání stejné funkce uvnitř těla funkce. Docílíme tím v podobného efektu jako cyklu.

```
public class RecursionSum {
  public static void main(String[] args) {
    int result = sum(10);
    System.out.println(result);
  }
  public static int sum(int k) {
    if (k > 0) {
      return k + sum(k - 1);
    } else {
      return 0;
    }
  }
}
```

### First-class a higher-order

Možnost použít funkci podobně jako proměnou ve volání metody. V Javě je tento konstrukt přidám až později a jenom částečně. Místo toho je možné použít anonymní vnitřní třídy.

- Lambda výrazy nepracují idealně s výjímkami
- Proměnné, ke kterým je přistupováno z vnitřní anonymní třídy musí být final nebo , efektivně final.

### Functional composition
Kombinací volání čistých funkcí je možné docílit řetězení funkcí. Tímto princip je možné složitější funkci poskládat z několika menších funkcí.
```
h(x) = g(f(x))
```
```
Predicate<String> startsWithA = (text) -> text.startsWith("A");
Predicate<String> endsWithX   = (text) -> text.endsWith("x");
Predicate<String> startsWithAAndEndsWithX =
        (text) -> startsWithA.test(text) && endsWithX.test(text);
String  input  = "A hardworking person must relax";
boolean result = startsWithAAndEndsWithX.test(input);
System.out.println(result);
```

```
// Lambda Expression
Predicate<String> lambda = (input) -> input != null;
// Anonymous class
Predicate<String> anonymous = new Predicate<String>() {
    @Override
    public boolean test(String t) {
        return t != null;
    }
};
```


### Lambda výrazy v Javě

Lambda výrazy je možné zapsat několika způsoby podle komplexity

```
parameter -> expression
```
```
(parameter1, parameter2) -> expression
```
```
(parameter1, parameter2) -> { code block }
```

```
import java.util.ArrayList;
class Main {
  public static void main(String[] args) {
    ArrayList<Integer> numbers = new ArrayList<Integer>();
    numbers.add(5);
    numbers.add(9);
    numbers.add(8);
    numbers.add(1);
    numbers.forEach( (n) -> { System.out.println(n); } );
  }
}
```
```
> javac -classpath .:/run_dir/junit-4.12.jar:target/dependency/* -d . Main.java
> java -classpath .:/run_dir/junit-4.12.jar:target/dependency/* Main
5
9
8
1
```

### Predicate
```
Predicate<Integer> lessThan = i -> (i < 20);  

System.out.println(lessThan.test(10));  
```

### Function

```
@FunctionalInterface
interface Square 
{ 
    int calculate(int x); 
} 
  
class Test 
{ 
    public static void main(String args[]) 
    { 
        int a = 5; 
  
        // lambda vyraz
        Square s = (int x) -> x * x; 
  
        int ans = s.calculate(a); 
        System.out.println(ans); 
    } 
} 
```

## Stream API

Od verze Javy 8, je možné používat několik konstruktů funkcionálního programování. Zvlástě výhodné je to při použítí s kolekcemi, kdy místo cyklů a iterátorů je možné řetězit akce za sebou a jednoduše oddělit různé prvky logiky.

### Úvod

Jedním z nejzajímavějších novinek v Jave 8 bylo přidaní java.util.stream. Co je Stream a k čemu slouží?

Stream je API pro využití přístupu funkcionálního programování na toku objektů. Operace se Streamy můžeme rozdělit do 3 kategorií

Vytvoření streamu - např. z kolekce nebo pole
Dílčí operace - vrací další Stream<T>
Konečná operace - vrací objekt jednoznačného typu
Stream je možné použít jen jednou. V případě, že se ho pokusíme použít znovu, tak nastane nehlídaná vyjímka IllegalStateException. Z toho důvodu je možné vždy použít jenom jednu konečnou operaci.

Volání dilčích operací je lazy, což znamená, že se zavolají jenom v případe, že je to potřeba pro výpočet výsledku. Např, pokud nás zajíma jenom první objekt, který projde přes filtr, tak už není potřeba zpracovávat zbytek.

Stream se zpracováva v pořadí, ve kterém se definují. Při návrhu je důležité zamyslet se, které operace redukují prvky ve streamu a případně je přeorganizovat.

```
long size = list.stream().map(element -> {
    expensiveOperation();
}).skip(2).count();
```
```
long size = list.stream().skip(2).map(element -> {
    expensiveOperation();
}).count();
```

### Vytvoření
#### Z pole

```
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Pomocnou metodou Stream.of(...)
```

```
Stream<String> stream = Stream.of("a", "b", "c");
```

#### Z kolekce

```
List<String> list = new ArrayList<>();
list.add("a");
Stream<String> stream = list.stream();
```

### Cykly vs Stream

### Parallel Stream

Před nějakou dobou se výkon jedno jádrových procesorů dostal ke svému limitu a od té doby se počet jader v procesorech začal postupně zvětšovat. Při vývoji aplikací to má bohužel ten efekt, že pro plné využití jejich výkonu, je nutné algoritmus navrhnout tak, aby šel paralelizovat.

Jeden z nejednodušších způsobů, jak toho dosáhnout je napsat operace nad kolekcemi, tak aby se vzájemně neovlivňovali. K tomuto slouží `parallelStream()` 
