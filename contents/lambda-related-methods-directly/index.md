<!-- .slide: data-state="main" data-background="../../images/master.png" data-background-size="contain" -->
# Lambda<!-- .element: style="text-align: center;font-size: 2em;" -->
###  Related Methods Directly in Lists and Maps

---

### Topics in This Section

- List
  - forEach (applies to all Iterables)
  - removeIf (applies to all Collections)
  - replaceAll
  - sort
- Map
  - forEach
  - computeIfAbsent (and compute, computeIfPresent)
  - merge
  - replaceAll

---

### Overview

- Lists and other collections
  - Have methods that are shortcuts for very similar Stream methods
  - Often modify the existing List, unlike the Stream versions
  - With very large Lists, the new methods might have small performance advantages
vs. the similar Stream methods
- Maps
  - Have methods that significantly extend their functionality vs. Java 7
  - No equivalent Stream methods

---

### Lists: Overview of New Java 8 Methods

- forEach
  - Identical to forEach for Streams, but saves you from calling “stream()” first
- removeIf
  - Like filter with negation of the Predicate, but removeIf modifies the original List
- replaceAll
  - Like map, but replaceAll modifies the original List
  - Also, with replaceAll, the Function must map values to the same type as in List
- sort
  - Takes Comparator just like stream.sorted and Arrays.sort

---

### forEach (Applies to All Iterables)

- Basic syntax

```java
someList.forEach(someConsumer)
employeeList.forEach(System.out::println)
```

- Equivalent Stream code

```java
someList.stream().forEach(someConsumer)
employeeList.stream().forEach(System.out::println)
```

- Advantages
  - Slightly shorter code
  - Same performance
- Disadvantages
  - None

---

### removeIf (Applies to All Collections)

- Basic syntax

```java
someList.removeIf(somePredicate)
stringList.removeIf(s -> s.contains("q"))
```

- Equivalent Stream code

```java
someList = someList.stream().filter(somePredicate.negate()).collect(Collectors.toList())
stringList = stringList.stream().filter(s -> !s.contains("q")).collect(Collectors.toList())
//If you want to be sure the new List is same concrete type as old one, then you should do ... collect(Collectors.toCollection(YourListType::new))
```

- Advantages
  - Shorter code if you want to modify the original List
  - Possible slight performance gain for very large Lists
- Disadvantages
  - Longer code if you want to result to be new List

---

### replaceAll

- Basic syntax

```java
someList.replaceAll(someUnaryOperator)
stringList.replaceAll(String::toUpperCase)
```

- Equivalent Stream code

```java
someList = someList.stream().map(someFunction).collect(Collectors.toList())
stringList = stringList.stream().map(String::toUpperCase).collect(Collectors.toList())
```

- Advantages
  - Shorter code if you want to modify the original List
  - Possible slight performance gain for very large Lists
- Disadvantages
  - Longer code if you want to result to be new List
  - replaceAll must map to same type as entries in original List, whereas map can
produce streams of totally different types

---

### sort

- Idea

```java
someList.sort(someComparator)
employeeList.sort(Comparator.comparing(Employee::getLastName))
```

- Equivalent Stream method

```java
someList = someList.stream().sorted(someComparator).collect(Collectors.toList())
stringList = stringList.stream().sorted(Comparator.comparing(Employee::getLastName)).collect(Collectors.toList())
```

- Advantages
  - Shorter code if you want to modify the original List
  - Large performance gain for very large LinkedLists (not for ArrayLists)
- Disadvantages
  - Longer code if you want to result to be new List

---

### Maps: Overview of New Java 8 Methods

- forEach(function)
  - Similar to forEach for Stream and List, except function takes two arguments: the
key and the value
- replaceAll(function)
  - For each Map entry, passes the key and the value to the function, takes the output,
and replaces the old value with it
  - Similar to replaceAll for List, except function takes two arguments: key and value
- merge(key, initialValue, function)
  - If no value is found for the key, store the initial value
  - Otherwise pass old value and initial value to the function, and overwrite current
result with that output
- computeIfAbsent(key, function)
  - If value is found for the key, return it
  - Otherwise pass the key to the function, store the output in Map, and return it

---

### forEach

- forEach(function): idea
  - For each Map entry, passes the key and the value to the function
- Example

```java
map.forEach((key, value) -> System.out.printf("(%s,%s)%n", key, value));

public class MapUtils {
  public static <K,V> void printMapEntries(Map<K,V> map) {
    map.forEach((key, value) -> System.out.printf("(%s,%s)%n", key, value));
  }
}

```


---

### replaceAll

- replaceAll(function): idea
  - For each Map entry, passes the key and the value to the function, then replaces existing value with that output
- Example

```java
shapeAreas.replaceAll((shape, area) -> Math.abs(area));

public class NumberMap {
  private static Map<Integer,String> numberMap = new HashMap<>();
  static {
    numberMap.put(1, "uno");
    numberMap.put(2, "dos");
    numberMap.put(3, "tres");
  }
  public static void main(String[] args) {
    MapUtils.printMapEntries(numberMap);
    numberMap.replaceAll((number, word) -> word.toUpperCase());
    MapUtils.printMapEntries(numberMap);
  }

 /**
  * (1,uno)
  * (2,dos)
  * (3,tres)
  * (1,UNO)
  * (2,DOS)
  * (3,TRES)
  */
}
```

---

### merge(key, initialValue, function)

- Idea
  - Lets you update existing values
    - If no value is found for the key, store the initial value
    - Otherwise pass old value and initial value to the function, and overwrite current result
with that updated result
- Example (creates message or adds it on end of old one)

```java
messages.merge(key, message, (old, initial) -> old + initial);
```

Or, equivalently

```java
messages.merge(key, message, String::concat);
```


---

### merge: Example Usage

- Given an array of ints, produce Map that has counts of how many times each entry
appeared
- For example, if you have a very large array of grades on exams (all from 0-100),
you can use this to sort them in O(N), instead of a comparison-based sort that is
O(N log(N)).


```java
public class CountEntries {
  public static void main(String[] args) {
     int[] nums = { 1, 2, 3, 3, 3, 3, 4, 2, 2, 1 };
     MapUtils.printMapEntries(countEntries(nums));
  }

  public static Map<Integer,Integer> countEntries(int[] nums) {
    Map<Integer,Integer> counts = new HashMap<>();
      for(int num: nums) {
        counts.merge(num, 1, (old, initial) -> old + 1);
      }
    return(counts);
  }
}

/**
 * Result
 * (1,2)
 * (2,3)
 * (3,4)
 * (4,1)
 */
```

---

### computeIfAbsent(key, function)

- Idea
  - Lets you remember previous computations
    - If value is found for the key, return it
    - Otherwise pass the key to the function, store the output in Map, and return it
  - If this technique is applied to entire result of a method call, it is known as
memoization (like memorization without the “r”). Memoization applied to recursive
functions that have overlapping subproblems results in automatic dynamic
programming and can provide huge performance gains.

---

### Example1 : Prime Numbers

- Idea
  - Use my Primes utility class to find primes of a given size. Once you find an n-digit
prime, remember it and return it next time that you are asked for one of that size.
- Original version

```java
public static BigInteger findPrime1(int numDigits) {
  return(Primes.findPrime(numDigits));
}
//Finding large (e.g., 200-digit) primes is expensive, so if you do not need new results each time, this is wasteful.
```

- Memoized version (after making Map<Integer,BigInteger>)

```java
public static BigInteger findPrime(int numDigits) {
  return(primes.computeIfAbsent(numDigits, n -> Primes.findPrime(n)));
}
```

  - First time you ask for prime of size n: calculates it. Second time: returns old result.

---

###### Example

```java
public class PrimeMap {
  private static Map<Integer,BigInteger> primes = new HashMap<>();
  public static void main(String[] args) {
    List<Integer> lengths = Arrays.asList(2, 10, 20, 100, 200);
    System.out.println("First pass");
    lengths.forEach(size -> System.out.printf(" %3s-digit prime: %s.%n", size, findPrime(size)));
    System.out.println("Second pass");
    lengths.forEach(size -> System.out.printf(" %3s-digit prime: %s.%n", size, findPrime(size)));
  }
  public static BigInteger findPrime(int numDigits) {
    return(primes.computeIfAbsent(numDigits, n -> Primes.findPrime(n)));
  }
}

/*
 * Result
 * First pass
 * 2-digit prime: 37.
 * 10-digit prime: 9865938581.
 * 20-digit prime: 81266731015996542377.
 * 100-digit prime: 1303427...3109.
 * 200-digit prime: 8579052...2809.
 * Second pass
 * 2-digit prime: 37.
 * 10-digit prime: 9865938581.
 * 20-digit prime: 81266731015996542377.
 * 100-digit prime: 1303427...3109.
 * 200-digit prime: 8579052...2809.
 */

```

---

### Example 2: Fibonacci Numbers

- Idea
  - Make a recursive function to calculate numbers in the sequence 0, 1, 1, 2, 3, 5, 8...
    - Fib(0) = 0
    - Fib(1) = 1
    - Fib(n) = Fib(n-1) + Fib(n-2)
- Problem
  - The straightforward recursive version has overlapping subproblems: when
computing Fib(n-1), it computes Fib(n-2), yet it repeats that computation when
finding Fib(n-2) directly. This results in exponential complexity.
- Solutions
  - In this case, you can build up from the bottom with an iterative version.
  - But, for many other problems, it is hard to find non-recursive solution, and that solution
is far more complex than the iterative one. E.g., recursive-descent parsing, finding
change with fewest coins, longest common substring, many more.
  - Better solution: use recursive version, then memoize it with computeIfAbsent

---

###### Example

```java

// Unmemoized Version
public static int fib1(int n) {
  if (n <= 1) {
    return(n);
  } else {
    return(fib1(n-1) + fib1(n-2));
  }
}

// Memoized Version:
private static Map<Integer,Integer> fibMap = new HashMap<>();
  public static int fib(int num) {
    return
  fibMap.computeIfAbsent(num, n -> {
    if (n <= 1) {
      return(n);
    } else {
      return(fib(n-1) + fib(n-2));
    }
  });
}

// Test Case
public static void profileFib() {
  for(i=0; i<47; i++) {
    Op.timeOp(() -> System.out.printf("fib1(%s)= %s.%n", i, fib1(i)));
    Op.timeOp(() -> System.out.printf("fib(%s) = %s.%n", i, fib(i)));
  }
}

/*
 * Result
 * fib1(0)= 0.
 * Elapsed time: 0.001 seconds.
 * fib(0) = 0.
 * Elapsed time: 0.001 seconds.
 * ...
 * fib1(45)= 1134903170.
 * Elapsed time: 5.293 seconds.
 * fib(45) = 1134903170.
 * Elapsed time: 0.000 seconds.
 * fib1(46)= 1836311903.
 * Elapsed time: 8.572 seconds.
 * fib(46) = 1836311903.
 * Elapsed time: 0.000 seconds.
 */
```

---

### Summary

- Lists
  - **forEach** : Identical to forEach for Streams, but saves you from calling “stream()” first
  - **removeIf** : Like filter with negation of the Predicate, but removeIf modifies the original List
  - **replaceAll** : Like map, but replaceAll modifies the original List
  - **sort** : Takes Comparator just like stream.sorted and Arrays.sort, and modifies original List
- Maps
  - **forEach(function)** and **replaceAll(function)** : Similar to versions for List, except function takes two arguments: the key and the value
  - **merge(key, initialValue, function)** : Lets you update old values
  - **computeIfAbsent(key, function)** : Lets you make memoized functions that remember previous calculations