# Design Patterns (Teil 1)

Die folgenden Patterns stammen alle aus dem Buch "Entwurfsmuster. Elemente wiederverwendbarer objektorientierter Software ist ein 1994 von Erich Gamma, Richard Helm, Ralph Johnson und John Vlissides". Erklärungen und Beispiele finden sich auch hier: [https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns).

## Observer Pattern

Die `Subject` Klasse beschreibt jene Objekte, die beobachtet werden sollen. Ein `Subject` muss wissen, von wem es beobachtet wird; dies wird in einer Datenstruktur (z.B. Liste) gespeichert.


```Java
interface Subject {
  void attach(Observer o);
  void detach(Observer o);
  void notifyObservers();
  
  String getName();

}
```

Die `Subject`s müssen über die Beobachter nichts wissen, ausser wie man sie von einer Zustandsänderung informiert. Dies wird durch das `Observer`-Interface beschrieben.


```Java
interface Observer {
    void update(Subject s);
}
```

Dieses Interface kann nun von konkreten Beobachtern implementiert werden, die dann mit der Information dass sich das `Subject` geändert hat beliebig umgehen können. Als Beispiel wollen wir einfach nur ausgeben, dass sich der Zustand verändert hat.


```Java
class ConcreteObserver implements Observer {
    @Override
    public void update(Subject s) {
        System.out.println("I have just been notified by " + s.getName());
    }
}
```

Die Implementierung eines `Subject`s fügt Zustand und Verhalten hinzu; die Verwaltung der Observer ist bereits in unserer abstrakten Oberklasse implementiert. Unser Beispiel wird einfach nur den Namen als Zustand speichern. Wenn der Zustand sich ändert, werden die `Observer` informiert mit Hilfe der `notifyObservers` Methode.


```Java
class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();

    private String name;

    public ConcreteSubject(String name) {
        this.name = name;
    }

    @Override
    public String getName() {
        return name;
    }

    public void rename(String newName) {
        name = newName;
        notifyObservers();
    }
    
    @Override
    public void attach(Observer o) {
        observers.add(o);
    }

    @Override
    public void detach(Observer o) {
        observers.remove(o);
    }

    @Override
    public void notifyObservers() {
        for(Observer o : observers) {
            o.update(this);
        }
    }
}
```


```Java
ConcreteSubject subjectA = new ConcreteSubject("A");
ConcreteSubject subjectB = new ConcreteSubject("B");
ConcreteObserver observer = new ConcreteObserver();
```

Observer müssen sich explizit bei Subjects anmelden.


```Java
subjectA.attach(observer);
subjectB.attach(observer);
```

Jedes Mal wenn wir ein `ConcreteSubject` umbenennen, wird unser Observer informiert.


```Java
subjectA.rename("C");
```

    I have just been notified by C



```Java
subjectB.rename("D");
```

    I have just been notified by D


Weitere Beispiele: [https://refactoring.guru/design-patterns/observer](https://refactoring.guru/design-patterns/observer)

## Composite

Ein Composite beschreibt eine Hierarchie von Objekten, bei der wir alle Objekte, egal ob sie eigene Teile (Leafs) sind oder nur aus anderen bestehen (Composite) gleich verwenden können. Alle Klassen implementieren ein generelles Interface mit den Operationen, die auf den Bestandteilen des Composites ausführbar sein sollen:


```Java
interface Component {
    void doTheOperation();
}
```

Ein Leaf implementiert nur die Funktionalität des `Component` Interfaces, aber hat keine eigenen Kinder.


```Java
class Leaf implements Component {
    private String name;

    public Leaf(String name) {
        this.name = name;
    }

    @Override
    public void doTheOperation() {
        System.out.println("Operation invoked on leaf "+name);
    }
}
```

Ein Composite verwaltet eine Sammlung von Kindern, und das Ausführen der Operation besteht daraus, die Operation auf allen Kindern durchzuführen.


```Java
class Composite implements Component {
    private List<Component> components = new ArrayList<>();

    public void add(Component c) {
        components.add(c);
    }

    @Override
    public void doTheOperation() {
        System.out.println("Invoked on composite");
        for(Component c : components) {
            c.doTheOperation();
        }
    }
}
```

Als Beispiel erstellen wir ein Composite (`comp1`), das aus zwei weiteren Composites besteht (`comp2`, `comp3`), die jeweils aus diversen Leaves bestehen.


```Java
Leaf leaf1 = new Leaf("A");
Leaf leaf2 = new Leaf("B");
Leaf leaf3 = new Leaf("C");
Leaf leaf4 = new Leaf("D");
Leaf leaf5 = new Leaf("E");
```


```Java
Composite comp1 = new Composite();
Composite comp2 = new Composite();
Composite comp3 = new Composite();
```


```Java
comp1.add(comp2);
comp1.add(comp3);
```


```Java
comp2.add(leaf1);
comp2.add(leaf2);
```


```Java
comp3.add(leaf3);
comp3.add(leaf4);
comp3.add(leaf5);
```

Wir können unsere Funktionalität, dargestellt durch `doTheOperation` auf allen Bestandteilen ausführen, egal ob sie Composites oder Leaves sind.


```Java
leaf1.doTheOperation()
```

    Operation invoked on leaf A


Nachdem `comp1` das äusserste Element unserer Hierarchie ist, wird die Operation auf allen Bestandteilen ausgeführt, wenn wir sie auf `comp1` ausführen.


```Java
comp1.doTheOperation();
```

    Invoked on composite
    Invoked on composite
    Operation invoked on leaf A
    Operation invoked on leaf B
    Invoked on composite
    Operation invoked on leaf C
    Operation invoked on leaf D
    Operation invoked on leaf E


## Composite Beispiel: Maze

Basierend auf dem Beispiel auf den Vorlesungs-Slides entwerfen wir ein Composite, das aus den Bestandteilen eines Mazes besteht.


```Java
public interface MapSite {
    void draw();
}
```


```Java
class Door implements MapSite {
    @Override
    public void draw() {
        System.out.println("Door");
    }
}
```


```Java
class Wall implements MapSite {
    @Override
    public void draw() {
        System.out.println("Wall");
    }
}
```


```Java
abstract class Composite implements MapSite {
    protected List<MapSite> elements = new ArrayList<>();

    public void add(MapSite x) {
        elements.add(x);
    }

    public void remove(MapSite x) {
        elements.remove(x);
    }

    public MapSite getChild(int x) {
        return elements.get(x);
    }

    public int size() {
        return elements.size();
    }

    @Override
    public void draw() {
        elements.forEach(e -> e.draw());
    }
}
```


```Java
class Room extends Composite {
    @Override
    public void draw() {
        System.out.println("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
}
```


```Java
class Maze extends Composite {
    @Override
    public void draw() {
        System.out.println("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
}
```

Um ein `Maze` zu erzeugen, müssen wir alle Komponenten einzeln erzeugen und zusammenbauen (das ist umständlich, und deshalb schauen wir uns dann auch gleich Erzeugungsmuster an).


```Java
public void demo() {
    Maze maze = new Maze();
    Room room1 = new Room();
    Room room2 = new Room();
    room1.add(new Wall());
    room1.add(new Wall());
    room1.add(new Wall());
    room1.add(new Wall());
    room2.add(new Wall());
    room2.add(new Wall());
    room2.add(new Wall());
    room2.add(new Wall());
    room1.add(new Door());
    room2.add(new Door());

    maze.add(room1);
    maze.add(room2);

    maze.draw();
}
```

Der Output ist natürlich nicht wirklich ein graphisches Labyrinth, aber wir können die hierarchische Struktur erkennen.


```Java
demo()
```

    Maze consisting of: 
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door


## Decorator

Wenn wir neue Funktionalität auf mehrere Elemente unserer Klassenhierarchie anwenden wollen, kann es passieren dass die Hierarchie stark wachsen muss; statt eine neue Unterklasse zu erzeugen erlaubt es uns das Decorator Pattern auch, einfach eine generelle Klasse für die neue Funktionalität zu definieren, und damit beliebige Elemente der Hierarchie zu "dekorieren".

Gegeben sei zunächst das Interface der Art von Komponenten die wir dekorieren wollen.


```Java
interface Component {
    public void doIt();
}
```

Konkrete Komponenten implementieren dieses Interface.


```Java
class A implements Component {
    public void doIt() {
        System.out.println("Doing A");
    }
}
```

Ein Decorator implementiert das gleiche Interface, und enthält eine Referenz auf das dekorierte Objekt. Abgesehen von der neuen Funktionalität (`X`) wird einfach an das dekorierte Objekt weitergeleitet.


```Java
class AWithX implements Component {
    private Component anA;

    public AWithX(Component anA) {
        this.anA = anA;
    }

    public void doIt() {
        System.out.println("Doing X");
        anA.doIt();
    }
}
```

Unterschiedliche Funktionalität (`Y`) kann einfach in unterschiedlichen Decorators implementiert werden.


```Java
class AWithY implements Component  {
    private Component anA;

    public AWithY(Component anA) {
        this.anA = anA;
    }

    @Override
    public void doIt() {
        System.out.println("Doing Y");
        anA.doIt();
    }
}
```

Hier zunächst die originale Komponente.


```Java
Component a = new A()
```

...die wir nun dekorieren können:


```Java
Component ax = new AWithX(a)
```


```Java
Component ay = new AWithY(a)
```

Je nachdem, ob wir die Komponente oder einen Decorator aufrufen, erhalten wir nur die Originalfunktionalität, oder die dekorierte Funktionalität.


```Java
a.doIt()
```

    Doing A



```Java
ax.doIt()
```

    Doing X
    Doing A



```Java
ay.doIt()
```

    Doing Y
    Doing A


Nun wenden wir dieses Muster noch auf unser Maze-Beispiel an. Ein `Decorator` ist eine `MapSite`.


```Java
abstract class Decorator implements MapSite {

    protected MapSite wrappedElement;

    public Decorator(MapSite site) {
        this.wrappedElement = site;
    }
}
```

Ein konkreter Decorator muss nun die abstrakte Methode `draw` implementieren, und dabei die Weiterleitung an das Originalobjekt übernehmen.


```Java
class MagicMapSite extends Decorator {
    public MagicMapSite(MapSite site) {
        super(site);
    }

    @Override
    public void draw() {
        System.out.println("****************");
        System.out.print("Magic: ");
        wrappedElement.draw();
        System.out.println("****************");
    }
}
```

Die Anwendung ist wieder gleich wie beim vorherigen Beispiel.


```Java
Door door = new Door()
```


```Java
MagicMapSite magicDoor = new MagicMapSite(door)
```


```Java
door.draw()
```

    Door



```Java
magicDoor.draw()
```

    ****************
    Magic: Door
    ****************


## Factory Method

Wir betrachten nun Erzeugsungsmuster, um die Erzeugung unseres Labyrinths zu vereinfachen. Eine Factory Methode ist eine einfache Art, die Entscheidung welches konkrete Objekt erzeugt wird konfigurierbar zu machen. Angenommen wir haben eine Funktionalität `doOperation` in der ein Integer-Wert erzeugt werden soll. Statt diesen Wert direkt in der Methode zu erzeugen, können wir die Erzeugung in eine abstrakte Methode auslagern. Damit können wir für unterschiedliche Szenarien unterschiedliche Varianten (=Unterklassen) implementieren diesen Wert zu erzeugen.


```Java
abstract class Creator {
    abstract protected int factoryMethod();

    public void doOperation() {
        int x = factoryMethod();
        System.out.println("The value is: "+x);
    }
}
```


```Java
class ConcreteCreatorA extends Creator {
    @Override
    protected int factoryMethod() {
        return 0;
    }
}
```


```Java
class ConcreteCreatorB extends Creator {
    @Override
    protected int factoryMethod() {
        return 100;
    }
}
```

Die beiden Beispiel-Erzeuger implementieren die identische Funktionalität, und unterscheiden sich nur in der Factory.


```Java
ConcreteCreatorA ca = new ConcreteCreatorA();
ConcreteCreatorB cb = new ConcreteCreatorB();
```


```Java
ca.doOperation();
```

    The value is: 0



```Java
cb.doOperation();
```

    The value is: 100


Nun übertragen wir das auf unser Maze-Beispiel: Angenommen, wir wollen das gleiche Maze mit unterschiedlichen Türen erzeugen können; wir lagern dazu die Erzeugung einer konkreten Door in eine Factory Methode aus.


```Java
MapSite createDoor() {
    return new Door();
}
```

Unsere Methode zur Erzeugung des Labyrinths muss dann noch statt des Konstruktors diese Factory-Methode verwenden.


```Java
public void demo() {
    Maze maze = new Maze();
    Room room1 = new Room();
    Room room2 = new Room();
    room1.add(new Wall());
    room1.add(new Wall());
    room1.add(new Wall());
    room1.add(new Wall());
    
    room2.add(new Wall());
    room2.add(new Wall());
    room2.add(new Wall());
    room2.add(new Wall());
    
    room1.add(createDoor());
    room2.add(createDoor());

    maze.add(room1);
    maze.add(room2);

    maze.draw();
}
```

Zunächst die Variante mit der normalen Door.


```Java
demo()
```

    Maze consisting of: 
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door


Nun wechseln wir die Door aus gegen eine dekorierte, "magische" Tür.


```Java
MapSite createDoor() {
    return new MagicMapSite(new Door());
}
```

Ohne Änderung des Codes zur Erzeugung des Labyrinths enthält das Resultat nun "magische Türen".


```Java
demo()
```

    Maze consisting of: 
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    ****************
    Magic: Door
    ****************
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    ****************
    Magic: Door
    ****************


## Abstract Factory

Wenn wir nun von der Erzeugung der Door zu allen Elementen unserer Hierarchie verallgemeinern, erhalten wir eine Abstract Factory. Gegeben eine Element-Hierarchie bestehend aus 2 Arten von Komponenten, von denen es jeweils 2 konkrete Ausprägungen gibt.


```Java
interface AbstractProductA {
    String getValue();
}
```


```Java
interface AbstractProductB {
    String getValue();
}
```


```Java
class ConcreteProductA1 implements AbstractProductA {
    @Override
    public String getValue() {
        return "Product A 1";
    }
}
```


```Java
class ConcreteProductA2 implements AbstractProductA {
    @Override
    public String getValue() {
        return "Product A 2";
    }
}
```


```Java
class ConcreteProductB1 implements AbstractProductB {
    @Override
    public String getValue() {
        return "Product B 1";
    }
}
```


```Java
class ConcreteProductB2 implements AbstractProductB {
    @Override
    public String getValue() {
        return "Product B 2";
    }
}
```

Eine Abstract Factory muss für jede Art von Komponente eine eigene Factory-Methode definieren.


```Java
interface AbstractFactory {
    AbstractProductA createProductA();
    AbstractProductB createProductB();
}
```

Nun können unterschiedliche konkrete Implementierungen der Abstract Factory unterschiedliche konkrete Varianten der Komponenten erzeugen.


```Java
class ConcreteFactory1 implements AbstractFactory {
    @Override
    public AbstractProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ConcreteProductB1();
    }
}
```


```Java
class ConcreteFactory2 implements AbstractFactory {
    @Override
    public AbstractProductA createProductA() {
        return new ConcreteProductA2();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ConcreteProductB2();
    }
}
```

Gegeben ein Kontext in dem die Factory verwendet wird.


```Java
public void printProducts(AbstractFactory factory) {
    AbstractProductA a = factory.createProductA();
    AbstractProductB b = factory.createProductB();
    System.out.println("A: "+a.getValue()+", B: "+b.getValue());
}
```

Je nach Factory ist die Ausgabe unterschiedlich.


```Java
ConcreteFactory1 factory1 = new ConcreteFactory1();
ConcreteFactory2 factory2 = new ConcreteFactory2();
```


```Java
printProducts(factory1);
```

    A: Product A 1, B: Product B 1



```Java
printProducts(factory2);
```

    A: Product A 2, B: Product B 2


Übertragen auf unser Maze-Beispiel benötigen wir beispielsweise Methoden für Räume, Türen, Wände.


```Java
interface MazeFactory {
  Room createRoom();
  MapSite createDoor();
  MapSite createWall();
}
```

Aus Anwendersicht werden nun die Aufrufe der Konstruktoren bzw. Factory-Methoden zu Aufrufen einer übergebenen Factory geändert.


```Java
public void demo(MazeFactory factory) {
    Maze maze = new Maze();
    Room room1 = factory.createRoom();
    Room room2 = factory.createRoom();
    room1.add(factory.createWall());
    room1.add(factory.createWall());
    room1.add(factory.createWall());
    room1.add(factory.createWall());
    
    room2.add(factory.createWall());
    room2.add(factory.createWall());
    room2.add(factory.createWall());
    room2.add(factory.createWall());
    
    room1.add(factory.createDoor());
    room2.add(factory.createDoor());

    maze.add(room1);
    maze.add(room2);

    maze.draw();
}
```

Hier eine Factory, die alle Komponenten in der Standardversion erzeugt.


```Java
class DefaultFactory implements MazeFactory {
    public Room createRoom() {
      return new Room();
    }
    
    public MapSite createDoor() {
      return new Door();
    }
    
    public MapSite createWall() {
      return new Wall();
    }
}
```


```Java
demo(new DefaultFactory())
```

    Maze consisting of: 
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door


Und hier eine Version, die dekorierte Varianten erzeugt.


```Java
class MagicFactory implements MazeFactory {
    public Room createRoom() {
      return new Room();
    }
    
    public MapSite createDoor() {
      return new MagicMapSite(new Door());
    }
    
    public MapSite createWall() {
      return new MagicMapSite(new Wall());
    }
}
```


```Java
demo(new MagicFactory())
```

    Maze consisting of: 
    Room consisting of:
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Door
    ****************
    Room consisting of:
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Wall
    ****************
    ****************
    Magic: Door
    ****************


## Builder

Auch wenn uns der Factory-Ansatz erlaubt die Arten der Komponenten auszutauschen, fehlt uns noch die Flexibilität die Struktur des Labyrinths zu verändern. Hierbei hilft das Builder-Pattern. Zunächst ein einfaches Beispiel, das die Konstruktion einer Liste an Integers vereinfachen soll.


```Java
interface Builder {
    Builder addNumber(int x);
    List<Integer> getProduct();
}
```

Ein konkreter Builder besteht aus Methoden um das Konstrukt zu erweitern (`addNumber`), und einer Methode um das finale Produkt abzufragen (`getProduct`).


```Java
class ConcreteBuilder implements Builder {
    private List<Integer> theList = new ArrayList<>();

    @Override
    public Builder addNumber(int x) {
        theList.add(x);
        return this;
    }

    @Override
    public List<Integer> getProduct() {
        return theList;
    }
}
```

Dadurch, dass die Methoden des `Builder`s den `Builder` selbst zurückgeben, kann die Erzeugung in Ketten von Methodenaufrufen erfolgen.


```Java
ConcreteBuilder builder = new ConcreteBuilder();
```


```Java
builder.addNumber(5).addNumber(10).addNumber(100);
```




    REPL.$JShell$96$ConcreteBuilder@7cf0c5ef




```Java
 System.out.println(builder.getProduct());
```

    [5, 10, 100]



```Java
ConcreteBuilder builder = new ConcreteBuilder();
```


```Java
builder.addNumber(10).addNumber(20).addNumber(30);
```




    REPL.$JShell$96$ConcreteBuilder@47f07156




```Java
 System.out.println(builder.getProduct());
```

    [10, 20, 30]


Wir wenden das Pattern nun wieder auf unser Maze-Beispiel an.


```Java
interface Builder {
    Builder withWall();
    Builder withDoor();
    Builder buildRoom();
    
    Maze getMaze();
}
```


```Java
class MazeBuilder implements Builder {
    private Maze maze = new Maze();
    private Room currentRoom = null;
    
    public Maze getMaze() {
        return maze;
    }
    
    public Builder buildRoom() {
        currentRoom = new Room();
        maze.add(currentRoom);        
        return this;
    }
    
    public Builder withDoor() {
        currentRoom.add(new Door());
        return this;
    }

    public Builder withWall() {
        currentRoom.add(new Wall());
        return this;
    }    
}
```

Die Erzeugung kann nun als Verkettung von Builder-Aufrufen geschehen. Dies gibt uns prinzipiell auch die Flexibilität, die Konstruktur zur Laufzeit erst zu entscheiden.


```Java
public void demo(Builder builder) {
    builder.buildRoom().withDoor().withWall().withWall().withWall().withWall();
    builder.buildRoom().withDoor().withWall().withWall().withWall().withWall();
    Maze maze = builder.getMaze();
    maze.draw();
}
```


```Java
Builder builder = new MazeBuilder();
```


```Java
demo(builder)
```

    Maze consisting of: 
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall


## Prototype

Ein alternativer Ansatz zur Erzeugung komplexer Objektstrukturen liegt darin, ein bestehendes komplexes Objekt zu kopieren, und die Kopie dann anzupassen. Im Prototype-Pattern wird von einem Prototyp-Objekt also eine tiefe Kopie erzeugt. Wir erweitern dazu das `MapSite` Interface um eine `clone` Methode.


```Java
interface MapSite<T extends MapSite> {
    void draw();
    T clone();
}
```

Für Leaf-Objekte in unserer Hierarchie wird diese Methode einfach so implementiert, dass wir ein neues (und damit identisches) Objekt erzeugen.


```Java
public class TheDoor implements MapSite<TheDoor> {
    @Override
    public void draw() {
        System.out.println("Door");
    }
    
    @Override
    public TheDoor clone() {
        return new TheDoor();        
    }
}
```


```Java
public class TheWall implements MapSite<TheWall> {
    @Override
    public void draw() {
        System.out.println("Wall");
    }
    
    public TheWall clone() {
        return new TheWall();
    }    
}
```

Schwieriger ist der Fall eines Composites: Hier müssen Kopien aller Kinder-Objekte erzeugt werden. Wir definieren uns dazu eine neue abstrakte Klasse `MazeComposite`, die zur bestehenden Funktionalität des bisherigen Composites noch die `clone` Methode verlangt. (Wir müssen die neuen `Composite` und `Maze` Klassen leider umbenennen damit das Jupyter Notebook damit glücklich wird).


```Java
public abstract class MazeComposite<T extends MazeComposite> implements MapSite<T> {
    protected List<MapSite> elements = new ArrayList<>();

    public void add(MapSite x) {
        elements.add(x);
    }

    public void remove(MapSite x) {
        elements.remove(x);
    }

    public MapSite getChild(int x) {
        return elements.get(x);
    }

    public int size() {
        return elements.size();
    }

    @Override
    public void draw() {
        elements.forEach(e -> e.draw());
    }
    
    @Override
    public abstract T clone();
}
```

Ein `Maze` ist ein Composite; das Klonen besteht einfach daraus, eine neue Liste zu erzeugen, und darin Kopien hinzuzufügen.


```Java
public class TheMaze extends MazeComposite<TheMaze> {
    @Override
    public void draw() {
        System.out.println("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public TheMaze clone() {
        TheMaze copy = new TheMaze();
        for (MapSite element : elements) {
            copy.add(element.clone());
        }
        return copy;
    }
}
```

Auch ein Raum ist ein Composite und muss die neue abstrakte Klasse erweitern (und wir müssen die Klasse daher anders benennen, damit Jupyter glücklich ist).


```Java
public class TheRoom extends MazeComposite<TheRoom> {
    @Override
    public void draw() {
        System.out.println("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
    @Override
    public TheRoom clone() {
        TheRoom copy = new TheRoom();
        for (MapSite element : elements) {
            copy.add(element.clone());
        }
        return copy;
    }    
}
```

Nachdem wir die Klassen umbenannt haben muss sich auch das Builder-Interface anpassen.


```Java
interface Builder {
    Builder withWall();
    Builder withDoor();
    Builder buildRoom();
    
    TheMaze getMaze();
}
```

...wie auch die Implementierung.


```Java
class MazeBuilder implements Builder {
    private TheMaze maze = new TheMaze();
    private TheRoom currentRoom = null;
    
    public TheMaze getMaze() {
        return maze;
    }
    
    public Builder buildRoom() {
        currentRoom = new TheRoom();
        maze.add(currentRoom);        
        return this;
    }
    
    public Builder withDoor() {
        currentRoom.add(new TheDoor());
        return this;
    }

    public Builder withWall() {
        currentRoom.add(new TheWall());
        return this;
    }    
}
```


```Java
MazeBuilder builder = new MazeBuilder();
builder.buildRoom().withDoor().withWall().withWall().withWall().withWall();
builder.buildRoom().withDoor().withWall().withWall().withWall().withWall();
```




    REPL.$JShell$102B$MazeBuilder@17e49a3d



`thisMaze` sei nun das Prototyp-Objekt.


```Java
TheMaze thisMaze = builder.getMaze()
```

`otherMaze` ist der Klon.


```Java
TheMaze otherMaze = builder.getMaze().clone()
```

Nun ist es einfach, dem Klon einen weiteren Raum hinzuzufügen.


```Java
otherMaze.add(new TheRoom())
```


```Java
thisMaze.draw()
```

    Maze consisting of: 
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall



```Java
otherMaze.draw()
```

    Maze consisting of: 
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall
    Room consisting of:
    Door
    Wall
    Wall
    Wall
    Wall
    Room consisting of:


## Singleton

Angenommen wir wollen die Verteilung von magischen Items im Labyrinth zentral kontrollieren. Es darf also aus Konsistenzgründen nur eine `MazeFactory` geben. Wie kann man sicherstellen, dass die Klasse nur einmal instanziiert wird? Die Antwort ist das Singleton-Muster.


```Java
public class TheMazeFactory {
    private TheMazeFactory() {}
    
    private static TheMazeFactory instance = null;
  
    public static TheMazeFactory getInstance() {
        if (instance == null) {
            instance = new TheMazeFactory();
        }
        return instance;
    }

  
    public TheRoom createRoom() {
        return new TheRoom();
    }
    
    public MapSite createDoor() {
        return new TheDoor();
    }
    
    public MapSite createWall() {
        return new TheWall();
    }
}
```


```Java
TheMazeFactory factory = TheMazeFactory.getInstance();
TheMaze newMaze = new TheMaze();
TheRoom roomA = factory.createRoom();
TheRoom roomB = factory.createRoom();
roomA.add(factory.createWall());
roomA.add(factory.createWall());
roomA.add(factory.createWall());
roomA.add(factory.createWall());
    
roomB.add(factory.createWall());
roomB.add(factory.createWall());
roomB.add(factory.createWall());
roomB.add(factory.createWall());
    
roomA.add(factory.createDoor());
roomB.add(factory.createDoor());

newMaze.add(roomA);
newMaze.add(roomB);
```

Wir können die Variante unserer `demo` Funktion anpassen, der wir bisher eine `MazeFactory` übergeben haben. Da es global nur mehr eine einzige Factory gibt, benötigen wir diesen Parameter nicht mehr.


```Java
newMaze.draw();
```

    Maze consisting of: 
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door
    Room consisting of:
    Wall
    Wall
    Wall
    Wall
    Door

