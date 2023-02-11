# Pure Java concepts learning notes and takeaway

This is the notes I used to document my Java learning journey for future re-looking up (just like learning other programming, frameworks, BUT just take it seriously for FUN), the content here are takeaway from several articles, books, and resources I've read over the internet NOT from myself, and of course its references are included in the last of each section below.

---

## Content

1. [Prefer _enums_ type rather than _constants integer_](#Prefer-enum-rather-constants-integer)

---

1. [Prefer _enum_ type rather than _constants integer_](#Prefer-enum-rather-constants-integer)

`Enums` should be prefer over `constants` values because they are:

-   _Type safety_: Enums are a type-safe alternative to constant int values, which can lead to subtle bugs if you use the wrong constant value. With enums, you can be sure that you are using the correct value, because you can only use values that are defined in the enum.
-   _Self-documenting_: Enums provide a more self-documenting alternative to constant int values. With enums, you can easily see the set of possible values and what they represent, without having to consult documentation or comments.

-   _Better error messages_: If you make a mistake when using constant int values, you will typically get an error message that includes a number, which can be difficult to understand. With enums, you will get a more meaningful error message that includes the name of the enum value that you used incorrectly.

-   _Extensibility_: If you need to add or remove values from a set of constant int values, you will need to make changes in multiple places in your code. With enums, you can simply add or remove values from the enum, and your code will automatically reflect these changes.

### Code Snippet

```java
class Person {
  static final String MALE = "MALE";
  static final String FEMALE = "FEMALE";
  static final String GAY = "GAY";

  public String getGenderAsMale() {
    return MALE;
  }

  public String getGenderAsFemale() {
    return FEMALE;
  }

  public String getGenderAsGay() {
    return GAY;
  }

  // etc...
}
```

better version with `enum`

```java
Class Person {
  public enum Sex {
    MALE, FEMALE, GAY
  }

  Sex gender;

  public Sex getGender() {
    return gender;
  }

  // etc...
}

```
