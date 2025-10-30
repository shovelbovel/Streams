# Support de Cours : Les Streams en Java
## 4ème année IIR - EMSI
### Ingénierie Informatique et Réseaux

---

## Table des matières
1. Introduction aux Streams
2. Architecture et concepts fondamentaux
3. Création des Streams
4. Opérations intermédiaires
5. Opérations terminales
6. Types de Streams spécialisés
7. Parallélisme et performance
8. Patterns et bonnes pratiques
9. Cas d'utilisation avancés
10. Exercices pratiques

---

## 1. Introduction aux Streams

### 1.1 Qu'est-ce qu'un Stream ?

Un Stream en Java (introduit dans Java 8) est une séquence d'éléments supportant des opérations séquentielles et parallèles pour le traitement de données. Il ne s'agit **pas** d'une structure de données, mais plutôt d'une abstraction permettant de manipuler des collections de manière déclarative et fonctionnelle.

### 1.2 Différence entre Stream et Collection

| Aspect | Collection | Stream |
|--------|-----------|--------|
| Stockage | Stocke les éléments | Ne stocke pas les éléments |
| Modification | Peut être modifiée | Immuable |
| Itération | Externe (boucles) | Interne (automatique) |
| Consommation | Réutilisable | Consommable une seule fois |
| Évaluation | Immédiate | Lazy (paresseuse) |

### 1.3 Avantages des Streams

- **Code plus lisible et concis** : Style déclaratif plutôt qu'impératif
- **Optimisation automatique** : Le compilateur peut optimiser les opérations
- **Parallélisme simplifié** : Traitement parallèle avec `.parallelStream()`
- **Composition d'opérations** : Chaînage fluide des transformations
- **Réduction de la verbosité** : Moins de code boilerplate

```java
// Approche traditionnelle
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.startsWith("A")) {
        result.add(s.toUpperCase());
    }
}

// Approche avec Stream
List<String> result = list.stream()
    .filter(s -> s.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 2. Architecture et concepts fondamentaux

### 2.1 Pipeline de traitement

Un Stream fonctionne selon le modèle du pipeline :

```
Source → Opérations intermédiaires → Opération terminale
```

**Exemple :**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int sum = numbers.stream()              // Source
    .filter(n -> n % 2 == 0)           // Opération intermédiaire
    .map(n -> n * n)                    // Opération intermédiaire
    .reduce(0, Integer::sum);           // Opération terminale

System.out.println("Somme des carrés pairs : " + sum); // 220
```

### 2.2 Évaluation paresseuse (Lazy Evaluation)

Les opérations intermédiaires ne sont **pas exécutées** tant qu'une opération terminale n'est pas appelée.

```java
Stream<String> stream = list.stream()
    .filter(s -> {
        System.out.println("Filtrage : " + s);
        return s.length() > 3;
    })
    .map(s -> {
        System.out.println("Mapping : " + s);
        return s.toUpperCase();
    });

// Rien ne s'affiche ici - aucune opération n'a été exécutée

List<String> result = stream.collect(Collectors.toList());
// Maintenant les opérations sont exécutées
```

### 2.3 Hiérarchie des interfaces Stream

```
BaseStream<T, S>
    ├── Stream<T>           (objets)
    ├── IntStream           (primitifs int)
    ├── LongStream          (primitifs long)
    └── DoubleStream        (primitifs double)
```

---

## 3. Création des Streams

### 3.1 À partir de Collections

```java
List<String> list = Arrays.asList("Java", "Python", "C++");
Stream<String> stream1 = list.stream();
Stream<String> stream2 = list.parallelStream();
```

### 3.2 À partir de tableaux

```java
String[] array = {"A", "B", "C"};
Stream<String> stream = Arrays.stream(array);

// Pour les primitifs
int[] numbers = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(numbers);
```

### 3.3 Avec Stream.of()

```java
Stream<String> stream = Stream.of("A", "B", "C");
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
Stream<String> empty = Stream.empty();
```

### 3.4 Streams infinis

```java
// Generate - fournit une fonction
Stream<Double> randomNumbers = Stream.generate(Math::random);

// Iterate - séquence basée sur une fonction
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);

// Avec limite
Stream<Integer> first10Even = Stream.iterate(0, n -> n + 2)
    .limit(10);
```

### 3.5 À partir de fichiers

```java
try (Stream<String> lines = Files.lines(Paths.get("data.txt"))) {
    lines.filter(line -> line.contains("Java"))
         .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

### 3.6 Stream Builder

```java
Stream<String> stream = Stream.<String>builder()
    .add("A")
    .add("B")
    .add("C")
    .build();
```

---

## 4. Opérations intermédiaires

Les opérations intermédiaires retournent un nouveau Stream et permettent le chaînage.

### 4.1 filter() - Filtrage

Conserve uniquement les éléments qui satisfont un prédicat.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Résultat : [2, 4, 6, 8, 10]

// Filtres multiples
List<Integer> result = numbers.stream()
    .filter(n -> n > 3)
    .filter(n -> n < 8)
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Résultat : [4, 6]
```

### 4.2 map() - Transformation

Transforme chaque élément selon une fonction.

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Résultat : ["ALICE", "BOB", "CHARLIE"]

List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// Résultat : [5, 3, 7]
```

### 4.3 flatMap() - Aplatissement

Transforme chaque élément en un Stream et aplatit le résultat.

```java
List<List<String>> lists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d"),
    Arrays.asList("e", "f")
);

List<String> flattened = lists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// Résultat : ["a", "b", "c", "d", "e", "f"]

// Exemple avec split
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Streams",
    "Functional Programming"
);

List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// Résultat : ["Hello", "World", "Java", "Streams", "Functional", "Programming"]
```

### 4.4 distinct() - Éléments uniques

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Résultat : [1, 2, 3, 4, 5]
```

### 4.5 sorted() - Tri

```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());
// Résultat : ["Alice", "Bob", "Charlie"]

// Tri avec Comparator
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
// Résultat : ["Bob", "Alice", "Charlie"]

// Tri inversé
List<String> reversed = names.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
```

### 4.6 limit() et skip()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// Résultat : [1, 2, 3, 4, 5]

List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
// Résultat : [6, 7, 8, 9, 10]

// Pagination
List<Integer> page2 = numbers.stream()
    .skip(5)
    .limit(3)
    .collect(Collectors.toList());
// Résultat : [6, 7, 8]
```

### 4.7 peek() - Débogage

Permet d'effectuer une action sur chaque élément sans modifier le Stream.

```java
List<Integer> result = numbers.stream()
    .filter(n -> n > 5)
    .peek(n -> System.out.println("Filtré : " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("Mappé : " + n))
    .collect(Collectors.toList());
```

---

## 5. Opérations terminales

Les opérations terminales déclenchent le traitement et ferment le Stream.

### 5.1 forEach() et forEachOrdered()

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream()
    .forEach(System.out::println);

// Garantit l'ordre même en parallèle
names.parallelStream()
    .forEachOrdered(System.out::println);
```

### 5.2 collect() - Collection des résultats

```java
// Vers List
List<String> list = stream.collect(Collectors.toList());

// Vers Set
Set<String> set = stream.collect(Collectors.toSet());

// Vers Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        String::length
    ));

// Joining
String result = names.stream()
    .collect(Collectors.joining(", "));
// Résultat : "Alice, Bob, Charlie"

// Grouping
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));

// Partitioning
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

### 5.3 reduce() - Réduction

Combine tous les éléments en un seul résultat.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Somme
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// ou
int sum = numbers.stream()
    .reduce(0, Integer::sum);

// Produit
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);

// Maximum
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);
// ou
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// Concaténation
String concatenated = words.stream()
    .reduce("", (a, b) -> a + b);
```

### 5.4 count(), min(), max()

```java
long count = stream.count();

Optional<String> min = stream.min(Comparator.naturalOrder());

Optional<String> max = stream.max(Comparator.comparing(String::length));
```

### 5.5 anyMatch(), allMatch(), noneMatch()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0); // true

boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0); // true

boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0); // true
```

### 5.6 findFirst() et findAny()

```java
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("A"))
    .findFirst();

Optional<String> any = names.parallelStream()
    .filter(n -> n.startsWith("B"))
    .findAny();
```

---

## 6. Types de Streams spécialisés

### 6.1 IntStream, LongStream, DoubleStream

Streams optimisés pour les types primitifs, évitant le boxing/unboxing.

```java
// IntStream
IntStream intStream = IntStream.range(1, 11); // 1 à 10
IntStream intStreamClosed = IntStream.rangeClosed(1, 10); // 1 à 10 inclus

int sum = IntStream.range(1, 101).sum();
double average = IntStream.range(1, 101).average().orElse(0.0);
int max = IntStream.range(1, 101).max().orElse(0);

// Statistiques
IntSummaryStatistics stats = IntStream.range(1, 101).summaryStatistics();
System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

### 6.2 Conversion entre types

```java
// Stream<Integer> vers IntStream
IntStream intStream = Stream.of(1, 2, 3)
    .mapToInt(Integer::intValue);

// IntStream vers Stream<Integer>
Stream<Integer> stream = IntStream.range(1, 10)
    .boxed();

// IntStream vers DoubleStream
DoubleStream doubleStream = IntStream.range(1, 10)
    .asDoubleStream();
```

### 6.3 Exemple pratique avec primitifs

```java
// Calcul de la moyenne des notes supérieures à 10
double moyenne = IntStream.of(8, 12, 15, 9, 18, 11, 14)
    .filter(note -> note >= 10)
    .average()
    .orElse(0.0);

System.out.println("Moyenne : " + moyenne);
```

---

## 7. Parallélisme et performance

### 7.1 Streams parallèles

```java
List<Integer> numbers = IntStream.rangeClosed(1, 1000)
    .boxed()
    .collect(Collectors.toList());

// Stream séquentiel
long start = System.currentTimeMillis();
long sum1 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
long end = System.currentTimeMillis();
System.out.println("Séquentiel : " + (end - start) + "ms");

// Stream parallèle
start = System.currentTimeMillis();
long sum2 = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();
end = System.currentTimeMillis();
System.out.println("Parallèle : " + (end - start) + "ms");
```

### 7.2 Quand utiliser le parallélisme ?

**✅ Utilisez parallelStream() quand :**
- Grand volume de données (> 10 000 éléments généralement)
- Opérations coûteuses en CPU
- Absence de dépendances entre éléments
- Opérations sans effet de bord

**❌ Évitez parallelStream() quand :**
- Petit volume de données
- Opérations rapides
- Dépendances entre éléments
- Opérations avec effet de bord (modifications externes)
- Structure de données non thread-safe

### 7.3 Configuration du pool de threads

```java
// Par défaut, utilise ForkJoinPool.commonPool()
// Nombre de threads = nombre de processeurs disponibles

System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "4");
```

### 7.4 Exemple de performance

```java
public class PerformanceTest {
    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 10_000_000)
            .boxed()
            .collect(Collectors.toList());
        
        // Test séquentiel
        long start = System.nanoTime();
        long sum = numbers.stream()
            .mapToLong(i -> fibonacci(i % 40))
            .sum();
        long duration = System.nanoTime() - start;
        System.out.println("Séquentiel: " + duration / 1_000_000 + "ms");
        
        // Test parallèle
        start = System.nanoTime();
        sum = numbers.parallelStream()
            .mapToLong(i -> fibonacci(i % 40))
            .sum();
        duration = System.nanoTime() - start;
        System.out.println("Parallèle: " + duration / 1_000_000 + "ms");
    }
    
    private static long fibonacci(int n) {
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```

---

## 8. Patterns et bonnes pratiques

### 8.1 Pattern : Traitement de données métier

```java
public class StudentProcessor {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
            new Student("Alice", 18, Arrays.asList(15.5, 17.0, 16.5)),
            new Student("Bob", 19, Arrays.asList(12.0, 13.5, 11.0)),
            new Student("Charlie", 18, Arrays.asList(18.0, 19.0, 17.5)),
            new Student("David", 20, Arrays.asList(9.0, 10.5, 8.5))
        );
        
        // Étudiants admis (moyenne >= 12) triés par moyenne décroissante
        List<Student> admitted = students.stream()
            .filter(s -> s.getAverage() >= 12.0)
            .sorted(Comparator.comparing(Student::getAverage).reversed())
            .collect(Collectors.toList());
        
        // Moyenne générale de la classe
        double classAverage = students.stream()
            .mapToDouble(Student::getAverage)
            .average()
            .orElse(0.0);
        
        // Groupement par âge
        Map<Integer, List<Student>> byAge = students.stream()
            .collect(Collectors.groupingBy(Student::getAge));
        
        // Statistiques
        DoubleSummaryStatistics stats = students.stream()
            .mapToDouble(Student::getAverage)
            .summaryStatistics();
        
        System.out.println("Moyenne de classe : " + classAverage);
        System.out.println("Meilleure note : " + stats.getMax());
        System.out.println("Étudiants admis : " + admitted.size());
    }
}

class Student {
    private String name;
    private int age;
    private List<Double> grades;
    
    public Student(String name, int age, List<Double> grades) {
        this.name = name;
        this.age = age;
        this.grades = grades;
    }
    
    public double getAverage() {
        return grades.stream()
            .mapToDouble(Double::doubleValue)
            .average()
            .orElse(0.0);
    }
    
    // Getters...
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

### 8.2 Pattern : Validation et traitement d'erreurs

```java
public class DataValidator {
    public static List<String> processEmails(List<String> emails) {
        return emails.stream()
            .filter(email -> email != null && !email.isEmpty())
            .filter(DataValidator::isValidEmail)
            .map(String::toLowerCase)
            .distinct()
            .sorted()
            .collect(Collectors.toList());
    }
    
    private static boolean isValidEmail(String email) {
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }
}
```

### 8.3 Bonnes pratiques

**1. Éviter les effets de bord**
```java
// ❌ Mauvais - modifie une variable externe
List<Integer> results = new ArrayList<>();
stream.forEach(n -> results.add(n * 2)); // Effet de bord

// ✅ Bon - utilise collect
List<Integer> results = stream
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

**2. Utiliser des méthodes de référence**
```java
// ❌ Moins lisible
stream.map(s -> s.toUpperCase())

// ✅ Plus lisible
stream.map(String::toUpperCase)
```

**3. Extraire la logique complexe**
```java
// ❌ Lambda complexe
stream.filter(student -> {
    double avg = student.getGrades().stream()
        .mapToDouble(Double::doubleValue)
        .average()
        .orElse(0.0);
    return avg >= 12.0 && student.getAge() < 25;
})

// ✅ Méthode extraite
stream.filter(this::isEligible)

private boolean isEligible(Student student) {
    return student.getAverage() >= 12.0 && student.getAge() < 25;
}
```

**4. Ne pas réutiliser un Stream**
```java
// ❌ Erreur - Stream déjà consommé
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.collect(Collectors.toList()); // IllegalStateException

// ✅ Créer un nouveau Stream
list.stream().forEach(System.out::println);
list.stream().collect(Collectors.toList());
```

---

## 9. Cas d'utilisation avancés

### 9.1 Traitement de logs

```java
public class LogAnalyzer {
    public static void analyzeLog(String logFile) throws IOException {
        Map<String, Long> errorCounts = Files.lines(Paths.get(logFile))
            .filter(line -> line.contains("ERROR"))
            .map(line -> line.split("\\|")[2]) // Extrait le type d'erreur
            .collect(Collectors.groupingBy(
                error -> error,
                Collectors.counting()
            ));
        
        errorCounts.entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(10)
            .forEach(entry -> 
                System.out.println(entry.getKey() + ": " + entry.getValue())
            );
    }
}
```

### 9.2 Traitement de fichiers CSV

```java
public class CSVProcessor {
    public static void processCSV(String filename) throws IOException {
        List<Employee> employees = Files.lines(Paths.get(filename))
            .skip(1) // Skip header
            .map(line -> line.split(","))
            .map(parts -> new Employee(
                parts[0],
                Integer.parseInt(parts[1]),
                Double.parseDouble(parts[2])
            ))
            .collect(Collectors.toList());
        
        // Statistiques salariales par département
        Map<String, DoubleSummaryStatistics> salaryStats = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingDouble(Employee::getSalary)
            ));
        
        salaryStats.forEach((dept, stats) -> {
            System.out.println(dept + ":");
            System.out.println("  Moyenne: " + stats.getAverage());
            System.out.println("  Min: " + stats.getMin());
            System.out.println("  Max: " + stats.getMax());
        });
    }
}

class Employee {
    private String name;
    private String department;
    private double salary;
    
    public Employee(String name, String department, double salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
    
    // Getters...
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
}
```

### 9.3 API REST - Filtrage dynamique

```java
public class ProductService {
    private List<Product> products;
    
    public List<Product> search(ProductFilter filter) {
        Stream<Product> stream = products.stream();
        
        if (filter.getMinPrice() != null) {
            stream = stream.filter(p -> p.getPrice() >= filter.getMinPrice());
        }
        
        if (filter.getMaxPrice() != null) {
            stream = stream.filter(p -> p.getPrice() <= filter.getMaxPrice());
        }
        
        if (filter.getCategory() != null) {
            stream = stream.filter(p -> p.getCategory().equals(filter.getCategory()));
        }
        
        if (filter.getKeyword() != null) {
            stream = stream.filter(p -> 
                p.getName().toLowerCase().contains(filter.getKeyword().toLowerCase())
            );
        }
        
        return stream
            .sorted(getComparator(filter.getSortBy()))
            .skip(filter.getOffset())
            .limit(filter.getLimit())
            .collect(Collectors.toList());
    }
    
    private Comparator<Product> getComparator(String sortBy) {
        switch (sortBy) {
            case "price": return Comparator.comparing(Product::getPrice);
            case "name": return Comparator.comparing(Product::getName);
            default: return Comparator.comparing(Product::getId);
        }
    }
}
```

### 9.4 Agrégation de données

```java
public class SalesAnalyzer {
    public static void analyzeSales(List<Sale> sales) {
        // Ventes totales par produit
        Map<String, Double> totalByProduct = sales.stream()
            .collect(Collectors.groupingBy(
                Sale::getProductName,
                Collectors.summingDouble(Sale::getAmount)
            ));
        
        // Top 5 des produits
        List<Map.Entry<String, Double>> top5 = totalByProduct.entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(5)
            .collect(Collectors.toList());
        
        // Ventes par mois
        Map<String, Double> monthlyRevenue = sales.stream()
            .collect(Collectors.groupingBy(
                sale -> sale.getDate().format(DateTimeFormatter.ofPattern("yyyy-MM")),
                Collectors.summingDouble(Sale::getAmount)
            ));
        
        // Statistiques globales
        DoubleSummaryStatistics stats = sales.stream()
            .mapToDouble(Sale::getAmount)
            .summaryStatistics();
        
        System.out.println("Chiffre d'affaires total : " + stats.getSum());
        System.out.println("Vente moyenne : " + stats.getAverage());
        System.out.println("Vente maximale : " + stats.getMax());
    }
}
```

---

## 10. Exercices pratiques

### Exercice 1 : Gestion d'une bibliothèque

Créez une classe `Library` avec une liste de livres. Implémentez les méthodes suivantes en utilisant les Streams :

```java
class Book {
    private String title;
    private String author;
    private int year;
    private double price;
    private String genre;
    
    // Constructeur et getters...
}

class Library {
    private List<Book> books;
    
    // TODO: Implémenter avec Streams
    public List<Book> findBooksByAuthor(String author) {
        // Retourner tous les livres d'un auteur
    }
    
    public Map<String, List<Book>> groupByGenre() {
        // Grouper les livres par genre
    }
    
    public double getAveragePrice() {
        // Calculer le prix moyen
    }
    
    public List<Book> getTop10MostRecent() {
        // Retourner les 10 livres les plus récents
    }
    
    public Map<String, Long> countBooksByAuthor() {
        // Compter le nombre de livres par auteur
    }
}
```

### Exercice 2 : Analyse de transactions

```java
class Transaction {
    private String id;
    private LocalDate