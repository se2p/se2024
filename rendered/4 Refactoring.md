# Code Smells und Refactoring

Das Thema dieser Vorlesung ist das Refactoring, also das Verbessern der Code Qualität ohne dabei die Funktionalität des Codes zu verändern. 

Wie in der Vorlesung besprochen ist Refactoring generell dann empfehlenswert, wenn:
1. Code nicht den Stilvorgaben entspricht (siehe z.B. [CheckStyle](https://checkstyle.sourceforge.io/))
2. Code zu komplex ist (Lines of Code, zyklomatische Komplexität, Halstead Metriken)
3. Es Code Smells im Code gibt.

Ein Grossteil der Vorlesung beschäftigte sich damit, wichtige Code Smells und entsprechende Refactorings einzuführen. Ich werde diese nicht alle nochmal in diesem Jupyter Notebook replizieren, sondern verweise an die folgenden beiden Sammlungen an Smells, Refactorings, und Beispielen:
- https://refactoring.guru/refactoring
- https://refactoring.com/catalog/

# Refactoring Beispiel

Im Folgenden nun die einzelnen Schritte des in der Vorlesung durchexerzierten Beispiels zum Refactoring; viele der einzelnen Schritte sind hierbei automatisierbar indem man die entsprechenden Refactorings in IntelliJ oder Eclipse anwendet.

Die Zwischenschritte sind auch als Commits in folgendem GitHub Repository verfügbar: [https://github.com/se2p/refactoring-example](https://github.com/se2p/refactoring-example) ("refactoring" Branch ).

Das Beispielszenario ist ein Videoverleih-Verwaltungssystem, das aus drei zentralen Klassen besteht. Die Klasse `Movie` unterscheidet drei verschiedene Arten von Filmen (Children's Movie, Regular Movie, New Release); unterschiedliche Arten von Filmen führen zu unterschiedlichen Kosten.


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private int priceCode;

    public Movie(String title, int priceCode) {
        this.title = title;
        this.priceCode = priceCode;
    }

    public int getPriceCode() {
        return priceCode;
    }

    public void setPriceCode(int arg) {
        priceCode = arg;
    }

    public String getTitle() {
        return title;
    }
}
```

Wichtig ist beim Refactoring, dass man bei jedem Schritt überprüfen kann, dass die Funktionalität nicht unabsichtlich verändert wurde. Dies erreicht man mit Hilfe von Tests. Hier deshalb eine JUnit Test Suite für die `Movie` Klasse.

Um JUnit im Jupyter Notebook zu verwenden, bedarf es wieder ein paar Extra-Schritte (die ausserhalb von Jupyter Notebooks nicht notwendig wären).


```Java
%mavenRepo oss-sonatype-snapshots https://oss.sonatype.org/content/repositories/snapshots/
%maven junit:junit:4.13.2
```


```Java
import org.junit.Test;
```


```Java
import static org.junit.Assert.*;
```

Die Tests für `Movie` überdecken alles an implementierter Funktionalität.


```Java
public class MovieTest {
    @Test
    public void test() {
        int priceCode = Movie.REGULAR;
        Movie movie = new Movie("Jaws", priceCode);
        assertEquals(priceCode, movie.getPriceCode());
    }

    @Test
    public void testNewRelease() {
        int priceCode = Movie.NEW_RELEASE;
        Movie movie = new Movie("Spiderman 27", priceCode);
        assertEquals(priceCode, movie.getPriceCode());
    }

    @Test
    public void testChildrensMovie() {
        int priceCode = Movie.CHILDREN;
        Movie movie = new Movie("Teletubbies", priceCode);
        assertEquals(priceCode, movie.getPriceCode());
    }

    @Test
    public void testTitle() {
        String title = "Die Hard";
        int priceCode = Movie.REGULAR;
        Movie movie = new Movie(title, priceCode);
        assertEquals(title, movie.getTitle());
    }

    @Test
    public void testSetPriceCode() {
        Movie movie = new Movie("Die Hard", Movie.REGULAR);
        movie.setPriceCode(Movie.NEW_RELEASE);
        assertEquals(Movie.NEW_RELEASE, movie.getPriceCode());
    }
}

```


```Java
import org.junit.runner.JUnitCore;
JUnitCore junit = new JUnitCore();
junit.addListener(new  org.junit.internal.TextListener(System.out));
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,01
    
    OK (5 tests)
    





    org.junit.runner.Result@d229d40



Initial ist der Code also auch richtig.

Ein `Rental` Objekt soll eine konkrete Verleihung eines Filmes beschreiben, und sich deshalb Film und Dauer merken.


```Java
public class Rental {
    private Movie movie;
    private int daysRented;

    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }

    public int getDaysRented() {
        return daysRented;
    }

    public Movie getMovie() {
        return movie;
    }

}
```

Auch zu dieser Klasse definieren wir Tests, um beim Refactoring sicher zu sein.


```Java
public class RentalTest {

    @Test
    public void testRentalDays() {
        int priceCode = Movie.REGULAR;
        Movie movie = new Movie("Jaws", priceCode);

        Rental rental = new Rental(movie, 2);
        assertEquals(2, rental.getDaysRented());
    }

    @Test
    public void testRentalMovie() {
        int priceCode = Movie.REGULAR;
        Movie movie = new Movie("Jaws", priceCode);

        Rental rental = new Rental(movie, 2);
        assertEquals(movie, rental.getMovie());
    }
}
```


```Java
junit.run(RentalTest.class);
```

    ..
    Time: 0,002
    
    OK (2 tests)
    





    org.junit.runner.Result@5f4c5f9f



Zuletzt noch die Klasse `Customer`; zu einem Kunden wird der Name gespeichert, als auch eine Liste an `Rentals`. Aus diesen lässt sich dann ein textueller Auszug (`statement`) erstellen, wahlweise auch in HTML Format.


```Java
import java.util.ArrayList;
import java.util.List;

public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = 0;
            
            // determine amounts for each line
            switch (each.getMovie().getPriceCode()) {
                case Movie.REGULAR:
                    thisAmount += 2;
                    if (each.getDaysRented() > 2)
                        thisAmount += (each.getDaysRented() - 2) * 1.5;
                    break;
                case Movie.NEW_RELEASE:
                    thisAmount += each.getDaysRented() * 3;
                    break;
                case Movie.CHILDREN:
                    thisAmount += 1.5;
                    if(each.getDaysRented() > 3)
                        thisAmount += (each.getDaysRented() - 3) * 1.5;
                    break;
            }

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = 0;
            
            // determine amounts for each line
            switch (each.getMovie().getPriceCode()) {
                case Movie.REGULAR:
                    thisAmount += 2;
                    if (each.getDaysRented() > 2)
                        thisAmount += (each.getDaysRented() - 2) * 1.5;
                    break;
                case Movie.NEW_RELEASE:
                    thisAmount += each.getDaysRented() * 3;
                    break;
                case Movie.CHILDREN:
                    thisAmount += 1.5;
                    if(each.getDaysRented() > 3)
                        thisAmount += (each.getDaysRented() - 3) * 1.5;
                    break;
            }

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }
}
```

Diese Klasse ist deutlich komplexer als die `Movie` und `Rental` Klassen, und benötigt mehr Tests.


```Java
public class CustomerTest {

    @Test
    public void testCustomerName() {
        Customer customer = new Customer("John Doe");
        assertEquals("John Doe", customer.getName());
    }

    @Test
    public void testCustomerStatementNoRentals() {
        Customer customer = new Customer("John Doe");

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue(statement.contains("Amount owed is 0.0"));
        assertTrue(statement.contains("You earned 0 frequent renter points"));
    }

    @Test
    public void testCustomerHTMLStatementNoRentals() {
        Customer customer = new Customer("John Doe");

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue(statement.contains("Amount owed is 0.0"));
        assertTrue(statement.contains("You earned 0 frequent renter points"));
    }

    @Test
    public void testCustomerStatementOneRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Jaws", Movie.REGULAR);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 2.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 2.0"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatementOneRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Jaws", Movie.REGULAR);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 2.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 2.0"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }


    @Test
    public void testCustomerStatementOneLongRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Jaws", Movie.REGULAR);
        Rental rental = new Rental(movie, 5);
        customer.addRental(rental);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 6.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 6.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatementOneLongRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Jaws", Movie.REGULAR);
        Rental rental = new Rental(movie, 5);
        customer.addRental(rental);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 6.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 6.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }


    @Test
    public void testCustomerStatementChildrenRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Kasperl", Movie.CHILDREN);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 1.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 1.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatementChildrenRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Kasperl", Movie.CHILDREN);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 1.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 1.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }


    @Test
    public void testCustomerStatementChildrenLongRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Kasperl", Movie.CHILDREN);
        Rental rental = new Rental(movie, 5);
        customer.addRental(rental);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 4.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 4.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatementChildrenLongRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Kasperl", Movie.CHILDREN);
        Rental rental = new Rental(movie, 5);
        customer.addRental(rental);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 4.5\", but got: \"" + statement+"\"", statement.contains("Amount owed is 4.5"));
        assertTrue("Expected: \"You earned 1 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 1 frequent renter points"));
    }

    @Test
    public void testCustomerStatementNewReleaseRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Matrix 7", Movie.NEW_RELEASE);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 6.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 6.0"));
        assertTrue("Expected: \"You earned 2 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 2 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatemenNewReleaseRental() {
        Customer customer = new Customer("John Doe");
        Movie movie = new Movie("Matrix 7", Movie.NEW_RELEASE);
        Rental rental = new Rental(movie, 2);
        customer.addRental(rental);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 6.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 6.0"));
        assertTrue("Expected: \"You earned 2 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 2 frequent renter points"));
    }

    @Test
    public void testCustomerStatementTwoRentals() {
        Customer customer = new Customer("John Doe");
        Movie movie1 = new Movie("Jaws", Movie.REGULAR);
        Rental rental1 = new Rental(movie1, 2);
        customer.addRental(rental1);

        Movie movie2 = new Movie("Matrix", Movie.REGULAR);
        Rental rental2 = new Rental(movie2, 1);
        customer.addRental(rental2);

        String statement = customer.statement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 4.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 4.0"));
        assertTrue("Expected: \"You earned 2 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 2 frequent renter points"));
    }

    @Test
    public void testCustomerHtmlStatementTwoRentals() {
        Customer customer = new Customer("John Doe");
        Movie movie1 = new Movie("Jaws", Movie.REGULAR);
        Rental rental1 = new Rental(movie1, 2);
        customer.addRental(rental1);

        Movie movie2 = new Movie("Matrix", Movie.REGULAR);
        Rental rental2 = new Rental(movie2, 1);
        customer.addRental(rental2);

        String statement = customer.htmlStatement();
        assertTrue(statement.contains("Rental Record for John Doe"));
        assertTrue("Expected: \"Amount owed is 4.0\", but got: \"" + statement+"\"", statement.contains("Amount owed is 4.0"));
        assertTrue("Expected: \"You earned 2 frequent renter points\", but got: \"" + statement+"\"", statement.contains("You earned 2 frequent renter points"));
    }
}
```


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,017
    
    OK (15 tests)
    





    org.junit.runner.Result@d567dfe



## Schritt 1: Extract method

Es ist auffällig, dass die `Customer` Klasse deutlich größer ist als die anderen beiden Klassen. Ein Grund dafür liegt im duplizierten Code in den Methoden `statement` und `htmlStatement`. Beispielsweise wird der identische Code verwendet um `thisAmount` pro Film zu erhöhen (= "duplicated code" smell). Hier bietet sich deshalb ein "Extract method" Refactoring an. Hier das Resultat, bei dem eine neue Methode `getAmountFor` eingefügt wurde, in die der duplizierte Code der `statement` und `htmlStatement` Methoden verschoben wurde, und dann durch einen Aufruf dieser neuen Methode ersetzt wurde. Dieser Schritt geht in einer modernen IDE automatisch.


```Java
import java.util.ArrayList;
import java.util.List;

public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = getAmountFor(each);

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = getAmountFor(each);

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }

    private double getAmountFor(Rental each) {
        double thisAmount = 0;
        // determine amounts for each line

        switch (each.getMovie().getPriceCode()) {
            case Movie.REGULAR:
                thisAmount += 2;
                if (each.getDaysRented() > 2)
                    thisAmount += (each.getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                thisAmount += each.getDaysRented() * 3;
                break;
            case Movie.CHILDREN:
                thisAmount += 1.5;
                if(each.getDaysRented() > 3)
                    thisAmount += (each.getDaysRented() - 3) * 1.5;
                break;
        }
        return thisAmount;
    }

    
}
```

Nach jedem Schritt müssen die Tests ausgeführt werden um sicherzustellen, dass alles ok ist.


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,009
    
    OK (15 tests)
    





    org.junit.runner.Result@6d99ceef



## Schritt 2: Rename variable

In der neuen Methode `getAmountFor` hat der Parameter `each` noch den Namen aus dem Kontext, aus dem wir den Code extrahiert haben. Allerdings ist dieser Begriff nur innerhalb der Iteration sinnvoll, als Methodenparameter scheint er wenig hilfreich. Wir können das mit einem "Rename variable" Refactoring korrigieren. Auch `thisAmount` ist wenig intuitiv.


```Java
import java.util.ArrayList;
import java.util.List;

public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = getAmountFor(each);

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = getAmountFor(each);

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }

    private double getAmountFor(Rental aRental) {
        double result = 0;
        // determine amounts for each line

        switch (aRental.getMovie().getPriceCode()) {
            case Movie.REGULAR:
                result += 2;
                if (aRental.getDaysRented() > 2)
                    result += (aRental.getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                result += aRental.getDaysRented() * 3;
                break;
            case Movie.CHILDREN:
                result += 1.5;
                if(aRental.getDaysRented() > 3)
                    result += (aRental.getDaysRented() - 3) * 1.5;
                break;
        }
        return result;
    }

    
}
```


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,012
    
    OK (15 tests)
    





    org.junit.runner.Result@5d6a1a1f



## Schritt 3: Move method

Wenn man die Methode `getAmountFor` genauer betrachtet, so fällt auf, dass fast alles an Berechnungen auf das `aRental` Objekt der Klasse `Rental` verweist. Es scheint, als läge ein Fall des Feature Envy Code Smells vor, und diese Funktionalität sollte eigentlich eher in der `Rental` Klasse liegen. Dies erreichen wir mit einem "Move method" Refactoring.


```Java
public class Rental {
    private Movie movie;
    private int daysRented;

    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }

    public int getDaysRented() {
        return daysRented;
    }

    public Movie getMovie() {
        return movie;
    }

    double getCharge() {
        double result = 0;
        // determine amounts for each line

        switch (getMovie().getPriceCode()) {
            case Movie.REGULAR:
                result += 2;
                if (getDaysRented() > 2)
                    result += (getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                result += getDaysRented() * 3;
                break;
            case Movie.CHILDREN:
                result += 1.5;
                if(getDaysRented() > 3)
                    result += (getDaysRented() - 3) * 1.5;
                break;
        }
        return result;
    }
}
```

Im Zuge der Verschiebung haben wir die Methode auch gleich umbenannt auf `getCharge`, was im Kontext der Klasse `Rental` besser passt.

In der `Customer` Klasse muss der Aufruf nun an das `each` Objekt delegiert werden.


```Java
public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints++;

            // add bonus for a two day new release rental
            if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) && each.getDaysRented() > 1)
                frequentRenterPoints++;

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }
}
```

Nachdem wir `Customer` und `Rental` verändert haben, führen wir beide Test Suites aus.


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,012
    
    OK (15 tests)
    





    org.junit.runner.Result@404d62fd




```Java
junit.run(RentalTest.class);
```

    ..
    Time: 0,001
    
    OK (2 tests)
    





    org.junit.runner.Result@35ca3d77



## Schritt 4: Extract & move method

Obwohl wir bereits Duplicated Code aus `statement` und `htmlStatement` entfernt haben, gibt es noch mehr Duplicated Code; beispielsweise die Berechnung der Frequent Renter Points. Wir können diese Funktionalität analog in eine eigene Methode extrahieren.


```Java
public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints += getFrequentRenterPoints(each);

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints += getFrequentRenterPoints(each);

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }
    
    int getFrequentRenterPoints(Rental aRental) {
        if ((aRental.getMovie().getPriceCode() == Movie.NEW_RELEASE) && aRental.getDaysRented() > 1) {
            return 2;
        } else {
            return 1;
        }
    }
}
```


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,008
    
    OK (15 tests)
    





    org.junit.runner.Result@349377d0



Ebenso analog zum vorherigen Schritt haben wir nun einen Feature Envy Smell, und schieben die Methode `getFrequentRenterPoints` in die `Rental` Klasse.


```Java
public class Rental {
    private Movie movie;
    private int daysRented;

    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }

    public int getDaysRented() {
        return daysRented;
    }

    public Movie getMovie() {
        return movie;
    }

    double getCharge() {
        double result = 0;
        // determine amounts for each line

        switch (getMovie().getPriceCode()) {
            case Movie.REGULAR:
                result += 2;
                if (getDaysRented() > 2)
                    result += (getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                result += getDaysRented() * 3;
                break;
            case Movie.CHILDREN:
                result += 1.5;
                if(getDaysRented() > 3)
                    result += (getDaysRented() - 3) * 1.5;
                break;
        }
        return result;
    }

    int getFrequentRenterPoints() {
        if ((getMovie().getPriceCode() == Movie.NEW_RELEASE) && getDaysRented() > 1) {
            return 2;
        } else {
            return 1;
        }
    }
}
```


```Java
public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {

            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints += each.getFrequentRenterPoints();

            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = 0;
        int frequentRenterPoints = 0;

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            double thisAmount = each.getCharge();

            // add frequent renter points
            frequentRenterPoints += each.getFrequentRenterPoints();

            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmount) + "</li>\n";
            totalAmount += thisAmount;
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }
}
```


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,01
    
    OK (15 tests)
    





    org.junit.runner.Result@3edfd02e




```Java
junit.run(RentalTest.class);
```

    ..
    Time: 0,001
    
    OK (2 tests)
    





    org.junit.runner.Result@73f84dae



## Schritt 5: Extract method

Auch nach 2 Extract Method Refactorings enthalten die beiden Schleifen in `statement` und `htmlStatement` Duplicated Code: Die Berechnung der Frequent Renter Points und die Berechnung der Kosten ist in beiden Fällen dupliziert, die Unterscheidung der beiden Methoden liegt rein im Aufbau des Report-Textes. Wir können diese beiden Berechnungen also extrahieren.


```Java
public class Customer {
    private String name;
    private List<Rental> rentals = new ArrayList<>();

    public Customer(String name) {
        this.name = name;
    }

    public void addRental(Rental arg) {
        rentals.add(arg);
    }

    public String getName() {
        return name;
    }

    public String statement() {
        // creates listing (text output) with all available information about given customer
        double totalAmount = getTotalCharge();
        int frequentRenterPoints = getTotalFrequentRenterPoints();

        String result = "Rental Record for " + getName() + "\n";
        for(Rental each : rentals) {
            // show figures for this rental
            result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
        }

        // add footer lines
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points";
        return result;
    }

    public String htmlStatement() {
        // creates html output with all available information about given customer
        double totalAmount = getTotalCharge();
        int frequentRenterPoints = getTotalFrequentRenterPoints();

        String result = "<h1>Rental Record for " + getName() + "</h1><ul>\n";

        for(Rental each : rentals) {
            // show figures for this rental
            result += "<li>\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "</li>\n";
        }

        // add footer lines
        result += "<ul>\n<p>Amount owed is " + String.valueOf(totalAmount) + "</p>\n";
        result += "<p>You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points</p>";
        return result;
    }

    public double getTotalCharge() {
        return rentals.stream().mapToDouble(Rental::getCharge).sum();
    }

    public int getTotalFrequentRenterPoints() {
        return rentals.stream().mapToInt(Rental::getFrequentRenterPoints).sum();
    }
}
```


```Java
junit.run(CustomerTest.class);
```

    ...............
    Time: 0,014
    
    OK (15 tests)
    





    org.junit.runner.Result@40220e7c



Man könnte meinen, dass wir die `statement` und `htmlStatement` nun ineffizienter gemacht haben, da die Rentals nun jeweils in 3 separaten Schleifen durchlaufen werden. Hierbei ist jedoch die Lesbarkeit des Codes auf jeden Fall wichtiger als die Effizienz des Codes, und es ist nicht zu erwarten, dass die Rental-Liste so lange werden kann, dass man einen Unterschied überhaupt bemerken kann.

## Schritt 6: Move method

Wir haben `getCharge` von `Customer` zu `Rental` geschoben, und damit die Zugriffe auf fremde Attribute stark reduziert. Jedoch merken wir nun, dass wir weiterhin Feature Envy haben: Wir können den Code noch weiter verbessern, indem wir Move Method noch einmal anwenden, und die Methode in die Klasse `Movie` verschieben.


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private int priceCode;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        this.priceCode = priceCode;
    }

    public int getPriceCode() {
        return priceCode;
    }

    public void setPriceCode(int arg) {
        priceCode = arg;
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        double result = 0;
        // determine amounts for each line

        switch (getPriceCode()) {
            case REGULAR:
                result += 2;
                if (daysRented > 2)
                    result += (daysRented - 2) * 1.5;
                break;
            case NEW_RELEASE:
                result += daysRented * 3;
                break;
            case CHILDREN:
                result += 1.5;
                if(daysRented > 3)
                    result += (daysRented - 3) * 1.5;
                break;
        }
        return result;
    }
}
```


```Java
public class Rental {
    private Movie movie;
    private int daysRented;

    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }

    public int getDaysRented() {
        return daysRented;
    }

    public Movie getMovie() {
        return movie;
    }

    double getCharge() {
        return movie.getCharge(daysRented);
    }

    int getFrequentRenterPoints() {
        if ((getMovie().getPriceCode() == Movie.NEW_RELEASE) && getDaysRented() > 1) {
            return 2;
        } else {
            return 1;
        }
    }
}
```


```Java
junit.run(MovieTest.class);
junit.run(RentalTest.class);
```

    .....
    Time: 0,002
    
    OK (5 tests)
    
    ..
    Time: 0,001
    
    OK (2 tests)
    





    org.junit.runner.Result@39cdb964



## Schritt 7: Move method

Das gleiche Problem besteht auch für die `getFrequentRenterPoints` Methode, welche wir daher auch verschieben.


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private int priceCode;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        this.priceCode = priceCode;
    }

    public int getPriceCode() {
        return priceCode;
    }

    public void setPriceCode(int arg) {
        priceCode = arg;
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        double result = 0;
        // determine amounts for each line

        switch (getPriceCode()) {
            case REGULAR:
                result += 2;
                if (daysRented > 2)
                    result += (daysRented - 2) * 1.5;
                break;
            case NEW_RELEASE:
                result += daysRented * 3;
                break;
            case CHILDREN:
                result += 1.5;
                if(daysRented > 3)
                    result += (daysRented - 3) * 1.5;
                break;
        }
        return result;
    }

    int getFrequentRenterPoints(int daysRented) {
        if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1) {
            return 2;
        } else {
            return 1;
        }
    }
}
```


```Java
public class Rental {
    private Movie movie;
    private int daysRented;

    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }

    public int getDaysRented() {
        return daysRented;
    }

    public Movie getMovie() {
        return movie;
    }

    double getCharge() {
        return movie.getCharge(daysRented);
    }

    int getFrequentRenterPoints() {
        return movie.getFrequentRenterPoints(daysRented);
    }
}
```


```Java
junit.run(MovieTest.class);
junit.run(RentalTest.class);
```

    .....
    Time: 0,002
    
    OK (5 tests)
    
    ..
    Time: 0,001
    
    OK (2 tests)
    





    org.junit.runner.Result@4c28bb5e



## Schritt 8: Replace Type Code by State Class

Die Unterscheidungen des Verhaltens in der `Movie` Klasse, insbesondere unter Verwendung des Switch-Statements, sind ein Zeichen für eine schlechte Klassenhierarchie. Das Switch-Statement mit einem Type Code (`priceCode`) würde sich gut für ein Replace Type Code with Subclasses Refactoring eignen, allerdings erzeugt dieses potentiell ein Problem: Wenn wir die `Movie` Klasse in Unterklassen (z.B. `RegularMovie`, `NewReleaseMovie`, `ChildrensMovie`) ändern dann (1) muss dazu Client-Code (wo die Movies eingetragen werden) verändert werden, und (2) kann sich der Price Code eines Films im Laufe seiner Lebzeit nicht mehr verändern. Ein New Release soll aber im Endeffekt nur eine Zeit lang ein New Release bleiben. Eine Alternative bietet das Refactoring "Replace Type Code by State Class".

Wir führen hierzu eine neue Klasse `MovieState` ein, mit den entsprechenden Subklassen. Ein `Movie` kann dann seinen `MovieState` ändern.


```Java
public abstract class MovieState {
    public abstract int getPriceCode();
}

class RegularMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.REGULAR;
    }
}

class NewMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.NEW_RELEASE;
    }
}

class ChildrenMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.CHILDREN;
    }
}
```

In der `Movie` Klasse ersetzen wir das `priceCode` Attribut durch den `movieState`. Damit die API nach aussen unverändert bleibt, übernimmt der Konstruktor weiterhin einen `priceCode`, erzeugt basierend darauf allerdings einen `MovieState`. Wenn sich der `MovieState` ändern soll, wird das von der Methode `setPriceCode` übernommen.


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private MovieState movieState;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        setPriceCode(priceCode);
    }

    public int getPriceCode() {
        return movieState.getPriceCode();
    }

    public void setPriceCode(int priceCode) {
        switch(priceCode) {
            case REGULAR:
                movieState = new RegularMovie();
                break;
            case NEW_RELEASE:
                movieState = new NewMovie();
                break;
            case CHILDREN:
                movieState = new ChildrenMovie();
                break;
            default:
                throw new IllegalArgumentException("Illegal pricecode");
        }
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        double result = 0;
        // determine amounts for each line

        switch (getPriceCode()) {
            case REGULAR:
                result += 2;
                if (daysRented > 2)
                    result += (daysRented - 2) * 1.5;
                break;
            case NEW_RELEASE:
                result += daysRented * 3;
                break;
            case CHILDREN:
                result += 1.5;
                if(daysRented > 3)
                    result += (daysRented - 3) * 1.5;
                break;
        }
        return result;
    }

    int getFrequentRenterPoints(int daysRented) {
        if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1) {
            return 2;
        } else {
            return 1;
        }
    }

}
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,001
    
    OK (5 tests)
    





    org.junit.runner.Result@40c52b69



## Schritt 9: Move method

Die Methode `getCharge` in der Klasse `Movie` riecht weiterhin nach Feature Envy, ein Zeichen dafür ist auch die Smelly Verwendung des Switch-Statements. Abhilfe schafft ein Move Method Refactoring.


```Java
public abstract class MovieState {
    public abstract int getPriceCode();

    double getCharge(int daysRented) {
        double result = 0;
        // determine amounts for each line

        switch (getPriceCode()) {
            case Movie.REGULAR:
                result += 2;
                if (daysRented > 2)
                    result += (daysRented - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                result += daysRented * 3;
                break;
            case Movie.CHILDREN:
                result += 1.5;
                if(daysRented > 3)
                    result += (daysRented - 3) * 1.5;
                break;
        }
        return result;
    }
}

class RegularMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.REGULAR;
    }
}

class NewMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.NEW_RELEASE;
    }
}

class ChildrenMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.CHILDREN;
    }
}
```


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private MovieState movieState;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        setPriceCode(priceCode);
    }

    public int getPriceCode() {
        return movieState.getPriceCode();
    }

    public void setPriceCode(int priceCode) {
        switch(priceCode) {
            case REGULAR:
                movieState = new RegularMovie();
                break;
            case NEW_RELEASE:
                movieState = new NewMovie();
                break;
            case CHILDREN:
                movieState = new ChildrenMovie();
                break;
            default:
                throw new IllegalArgumentException("Illegal pricecode");
        }
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        return movieState.getCharge(daysRented);
    }

    int getFrequentRenterPoints(int daysRented) {
        if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1) {
            return 2;
        } else {
            return 1;
        }
    }

}
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,002
    
    OK (5 tests)
    





    org.junit.runner.Result@7e3ba217



## Schritt 10: Replace Conditional with Polymorphism

Nachdem das Switch-Statement nun in der `MovieState` Klasse liegt, können wir unser smelly Switch-Statement durch Vererbung ersetzen.

An dieser Stelle macht uns eine zyklische Abhängigkeit leider ein Problem in diesem Jupyter Notebook. Nur für diesen Schritt benenne ich daher die `MovieState`-Klassen in `MovieState2` etc um. Dies ist nicht Teil des Refactorings, sondern nur ein Workaround für das Notebook!


```Java
public abstract class MovieState2 {
    public abstract int getPriceCode();

    public abstract double getCharge(int daysRented);
}

class RegularMovie2 extends MovieState2 {
    @Override
    public int getPriceCode() {
        return Movie.REGULAR; // 0
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 2;
        if(daysRented > 2) {
            result += (daysRented - 2) * 1.5;
        }
        return result;
    }
}

class NewMovie2 extends MovieState2 {
    @Override
    public int getPriceCode() {
        return Movie.NEW_RELEASE; // 1
    }

    @Override
    public double getCharge(int daysRented) {
        return daysRented * 3;
    }
}

class ChildrenMovie2 extends MovieState2 {
    @Override
    public int getPriceCode() {
        return Movie.CHILDREN; // 2
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 1.5;
        if(daysRented > 3) {
            result += (daysRented - 3) * 1.5;
        }
        return result;
    }
}
```


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private MovieState2 movieState;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        setPriceCode(priceCode);
    }

    public int getPriceCode() {
        return movieState.getPriceCode();
    }

    public void setPriceCode(int priceCode) {
        switch(priceCode) {
            case REGULAR:
                movieState = new RegularMovie2();
                break;
            case NEW_RELEASE:
                movieState = new NewMovie2();
                break;
            case CHILDREN:
                movieState = new ChildrenMovie2();
                break;
            default:
                throw new IllegalArgumentException("Illegal pricecode");
        }
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        return movieState.getCharge(daysRented);
    }

    int getFrequentRenterPoints(int daysRented) {
        if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1) {
            return 2;
        } else {
            return 1;
        }
    }

}
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,001
    
    OK (5 tests)
    





    org.junit.runner.Result@6d9cc63a



## Schritt 11: Move method

Auch die Berechnung der Frequent Renter Points, die aktuell noch in der `Movie` Klasse liegt, ist besser in der `MovieState` Klasse aufgehoben. Wir führen also ein Move Method Refactoring durch.


```Java
public abstract class MovieState {
    public abstract int getPriceCode();

    public abstract double getCharge(int daysRented);

    public int getFrequentRenterPoints(int daysRented) {
        if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1) {
            return 2;
        } else {
            return 1;
        }
    }

}

class RegularMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.REGULAR;
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 2;
        if(daysRented > 2) {
            result += (daysRented - 2) * 1.5;
        }
        return result;
    }
}

class NewMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.NEW_RELEASE;
    }

    @Override
    public double getCharge(int daysRented) {
        return daysRented * 3;
    }
}

class ChildrenMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.CHILDREN;
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 1.5;
        if(daysRented > 3) {
            result += (daysRented - 3) * 1.5;
        }
        return result;
    }
}
```


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private MovieState movieState;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        setPriceCode(priceCode);
    }

    public int getPriceCode() {
        return movieState.getPriceCode();
    }

    public void setPriceCode(int priceCode) {
        switch(priceCode) {
            case REGULAR:
                movieState = new RegularMovie();
                break;
            case NEW_RELEASE:
                movieState = new NewMovie();
                break;
            case CHILDREN:
                movieState = new ChildrenMovie();
                break;
            default:
                throw new IllegalArgumentException("Illegal pricecode");
        }
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        return movieState.getCharge(daysRented);
    }

    int getFrequentRenterPoints(int daysRented) {
        return movieState.getFrequentRenterPoints(daysRented);
    }

}
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,001
    
    OK (5 tests)
    





    org.junit.runner.Result@3bfcd615



## Schritt 12: Replace Conditional with Polymorphism

Nun können wir den Code weiter vereinfachen, indem wir auch auf `getFrequentRenterPoints` ein Replace Conditional with Polymorphism Refactoring anwenden.


```Java
public abstract class MovieState {
    public abstract int getPriceCode();

    public abstract double getCharge(int daysRented);

    public int getFrequentRenterPoints(int daysRented) {
        return 1;
    }
}

class RegularMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.REGULAR;
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 2;
        if(daysRented > 2) {
            result += (daysRented - 2) * 1.5;
        }
        return result;
    }
}

class NewMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.NEW_RELEASE;
    }

    @Override
    public double getCharge(int daysRented) {
        return daysRented * 3;
    }

    @Override
    public int getFrequentRenterPoints(int daysRented) {
        if (daysRented > 1) {
            return 2;
        } else {
            return 1;
        }

    }
}

class ChildrenMovie extends MovieState {
    @Override
    public int getPriceCode() {
        return Movie.CHILDREN;
    }

    @Override
    public double getCharge(int daysRented) {
        double result = 1.5;
        if(daysRented > 3) {
            result += (daysRented - 3) * 1.5;
        }
        return result;
    }
}
```


```Java
public class Movie {
    public static final int CHILDREN = 2;
    public static final int REGULAR = 0;
    public static final int NEW_RELEASE = 1;
    private String title;
    private MovieState movieState;

    // regular movie or for children or ...
    public Movie(String title, int priceCode) {
        this.title = title;
        setPriceCode(priceCode);
    }

    public int getPriceCode() {
        return movieState.getPriceCode();
    }

    public void setPriceCode(int priceCode) {
        switch(priceCode) {
            case REGULAR:
                movieState = new RegularMovie();
                break;
            case NEW_RELEASE:
                movieState = new NewMovie();
                break;
            case CHILDREN:
                movieState = new ChildrenMovie();
                break;
            default:
                throw new IllegalArgumentException("Illegal pricecode");
        }
    }

    public String getTitle() {
        return title;
    }

    double getCharge(int daysRented) {
        return movieState.getCharge(daysRented);
    }

    int getFrequentRenterPoints(int daysRented) {
        return movieState.getFrequentRenterPoints(daysRented);
    }

}
```


```Java
junit.run(MovieTest.class);
```

    .....
    Time: 0,001
    
    OK (5 tests)
    





    org.junit.runner.Result@5807d1ba



Nun hat der Code insgesamt deutlich weniger Code Smells. Natürlich gäbe es auch noch andere Varianten den gleichen Code zu implementieren, und wir werden uns im Laufe der Vorlesung daher noch genauer mit der Frage nach gutem _Design_ beschäftigen. Zum Abschluss führen wir aber sicherheitshalber nochmal _alle_ Tests aus:


```Java
junit.run(MovieTest.class);
junit.run(RentalTest.class);
junit.run(CustomerTest.class);
```

    .....
    Time: 0,001
    
    OK (5 tests)
    
    ..
    Time: 0,002
    
    OK (2 tests)
    
    ...............
    Time: 0,008
    
    OK (15 tests)
    





    org.junit.runner.Result@6d62973d


