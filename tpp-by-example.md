
## TPP BY EXAMPLE

The whole [Transformation Priority Premise](http://blog.8thlight.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html) thing has attracted my attention. In a nutshell, Uncle Bob developed a theory where he states that _"As the tests get more specific, the code gets more generic."_, and he called _Transformations_ the set of different changes that are applied to the production code to make it more generic in order to pass a test. More than that, when writing tests you are also applying _Transformations_, and choosing the right transformation for the next test (and later to pass it) leads us to a better overall solution. Going further: Transformations can be ordered by priority. Simpler transformations are of higher priority. Follows the list (from higher to lower priority):

*   ({}–>nil) no code at all -> code that employs nil
*   (nil -> constant)
*   (constant -> constant+) a simple constant to a more complex constant
*   (constant -> scalar) replacing a constant with a variable or an argument
*   (statement -> statements) adding more unconditional statements.
*   (unconditional -> if) splitting the execution path
*   (scalar -> array)
*   (array -> container)
*   (statement -> recursion)
*   (if -> while)
*   (expression -> function) replacing an expression with a function or algorithm
*   (variable -> assignment) replacing the value of a variable.

You must strive to use transformations of higher priority over lower priority ones.

So I decided to try.

### The Problem

Let's tackle a very simple problem:

```
"You are given strings of different lengths. Insert ‘bazinga’ for each continuous set of vowels."
e.g.: "her" -> "hbazingar", "bear" -> "bbazingar"
```



### The Solution

I'll start with the simplest test I can imagine:

```java
class BazingafierTest {

@Test
  public void testBazingafier() throws Exception {
    assertThat(new Bazingafier().bazingafy(""), is(""));
  }

}
```

To pass that test, first I use my IDE to generate some code -and that applies the highest priority transformation: **({} –> nil)**

```java
class Bazingafier {

  public String bazingafy(String word) {
    return null;
  }

}
```
The test isn't passing. By using another transformation, **(nil -> constant)** we can make it pass. We had:

```java
return null;
```

Apply **(nil -> constant)**:

```java
return "";
```


And the tests are passing. Now let's apply a transformation to the test in order to find the next test. In the beginning we had no code so we applied **({} -> nil)** and **(nil -> constant)** to get our first failing test. We can use **(constant -> constant+)** and write a new test. Our previous constant was **""**, now we will make it more complex, for instance: **"a"**.

```java
assertThat(new Bazingafier().bazingafy("a"), is("bazinga"));
```


Observe how this is too much of a big jump. To implement code that passes this test we will need a transformation with very low priority, **(unconditional -> if)**. I could write a different test, such as:

```java
assertThat(new Bazingafier().bazingafy("t"), is("t"));
```


and it would require a higher priority transformation, to make this test pass. Following the priority premise, let's go with that. To make that test pass, I will apply **(constant -> scalar)**.

```java
return word;
```


And all tests should be passing. Next test:

```java
assertThat(new Bazingafier().bazingafy("a"), is("bazinga"));
```


To pass this test I will need the following transformation: **(unconditional -> if)**.

```java
if (word.equals("a")) {
  return "bazinga";
}
return word;
```

That takes care of a single vowel. We will need tests for the others as well.

```java
assertThat(new Bazingafier().bazingafy("e"), is("bazinga"));
assertThat(new Bazingafier().bazingafy("i"), is("bazinga"));
assertThat(new Bazingafier().bazingafy("o"), is("bazinga"));
assertThat(new Bazingafier().bazingafy("u"), is("bazinga"));
```

We can pass this tests by transforming **(statement -> statements)**.

```java
if (word.equals("a")
  || word.equals("e")
  || word.equals("i")
  || word.equals("o")
  || word.equals("u")) {
  return "bazinga";
}
return word;
```

And all tests are passing. That is too big of a condition so I'll refactor and extract a method out of it.

```java
private boolean isVowel(String word) {
  return word.equals("a")
    || word.equals("e")
    || word.equals("i")
    || word.equals("o")
    || word.equals("u");
}

public String bazingafy(String word) {
  if (isVowel(word)) {
    return "bazinga";
  }
  return word;
}
```

I'll refactor the code to have a single return statement. This will make it easier to accommodate further changes.

```java
public String bazingafy(String word) {
  String bazingafiedWord = "";
  if (isVowel(word)) {
    bazingafiedWord = "bazinga";
  } else {
    bazingafiedWord = word;
  }
  return bazingafiedWord;
}
```

And all tests are passing.

Things will get interesting now. We are done with empty strings, single consonants and vowels. From what I can think, the next test that will fail is:

```java
assertThat(new Bazingafier().bazingafy("qa"), is("qbazinga"));
```


Great, test is failing. The more straightforward way I can think of on how to pass this test is by iterating over the characters of the word. That would be a very low priority **(expression -> function)** transformation.

```java
String bazingafiedWord = "";
for (Character letter : word.toCharArray()) {
  if (isVowel(letter.toString())) {
    bazingafiedWord += "bazinga";
  } else {
    bazingafiedWord += letter;
  }
}
return bazingafiedWord;
```

Now for the next test:

```java
assertThat(new Bazingafier().bazingafy("hear"), is("hbazingar"));
```


Now that is an interesting situation. How can we make that pass? I can't see any simple transformation that would make it pass. I could try to keep a reference to the previous letter and then only append **"bazinga"** to **bazingafiedWord** in case the current letter is not a vowel but the previous letter is, but that would break cases where I have a single vowel. Another option would be testing if the current letter is a vowel _and_ the current bazingafied word ends with **"bazinga"** and not append it then. It could work...

But I have this feeling that something went wrong. And that's because I didn't follow the Transformation Priority Premise. Let's go back to when I wrote the first test with more than one character:

```java
assertThat(new Bazingafier().bazingafy("qa"), is("qbazinga"));
```


Before, when faced with this test, I've decided to go with **(expression -> function)**. There I didn't realize that there was a higher priority transformation to be applied: **(statement -> recursion)**

So let's try it. At that moment, the code looked like this:

```java
public String bazingafy(String word) {
String bazingafiedWord = "";
  if (isVowel(word)) {
    bazingafiedWord = "bazinga";
  } else {
    bazingafiedWord = word;
  }
  return bazingafiedWord;
}
```

Applying **(statement -> recursion)**:

```java
public String bazingafy(String word) {
  if (word.isEmpty()) {
    return word;
  }

  String bazingafiedWord = "";
  String firstLetter = String.valueOf(word.charAt(0));

  if (isVowel(firstLetter)) {
    bazingafiedWord = "bazinga";
  } else {
    bazingafiedWord = firstLetter;
  }

  return bazingafiedWord + bazingafy(word.substring(1));
}
```

That passes the test.

The next test:

```java
assertThat(new Bazingafier().bazingafy("hear"), is("hbazingar"));
```


Now looking at my algorithm I can easily find a solution. I can now pass the letter there is being bazingafied into the recursive function (and call it **previousLetter**). Then, if the previous letter was a vowel and the current letter is a vowel I will just skip this letter and keep going with the recursive algorithm. For the first call I'll initialize it with **null**. I have then to add a null check in the **isVowel** method to avoid an exception.

```java
public class Bazingafier {

  public String bazingafy(String word) {
    return bazingafy(null, word);
  }

  private String bazingafy(String previousLetter, String word) {
    if (word.isEmpty()) {
      return word;
    }

    String firstLetter = String.valueOf(word.charAt(0));

    if (isVowel(firstLetter) && isVowel(previousLetter)) {
      return bazingafy(word.substring(1));
    }

    String bazingafiedLetter = "";
    if (isVowel(firstLetter)) {
      bazingafiedLetter = "bazinga";
    } else {
      bazingafiedLetter = firstLetter;
    }

    return bazingafiedLetter + bazingafy(firstLetter, word.substring(1));
  }

  private boolean isVowel(String letter) {
    if (letter == null) return false;
    return letter.equals("a")
           || letter.equals("e")
           || letter.equals("i")
           || letter.equals("o")
           || letter.equals("u");
  }
}
```

That passes all the tests! It could use some refactoring though:

```java
public class Bazingafier {

  public static final String VOWELS = "aeiou";

  public String bazingafy(String word) {
    return bazingafy(null, word);
  }

  private String bazingafy(String previousLetter, String word) {
    String firstLetter = String.valueOf(word.charAt(0));

    if (word.isEmpty()) {
      return word;
    }
    if (areBothVowels(firstLetter, previousLetter)) {
      return bazingafy(word.substring(1)); 
    }
    return bazingafyLetter(firstLetter) + bazingafy(word.substring(1));
  }

  private boolean areBothVowels(String aLetter, String otherLetter) {
    return isVowel(aLetter) && isVowel(anotherLetter);
  }  

  private String bazingafyLetter(String word, String firstLetter) {
    return isVowel(firstLetter) ? "bazinga" : firstLetter
  }

  private boolean isVowel(String letter) {
    return letter != null && VOWELS.contains(letter);
  }
}
```

And I'm done.

### Yes, I know...

..it could also be solved with:

```java
return word.replaceAll("[aeiou]+", "bazinga");
```


But that is not the point. :)

