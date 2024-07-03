# Design Patterns 2

## Bridge

Angenommen, wir wollen das Maze aus dem vorherigen Kapitel nun sowohl mit Java Swing als auch für Android implementieren. Hierzu muss die Funktionalität der `draw` Methode der `MapSite`s jeweils für Android und Swing implementiert werden, und das für jede Art von Maze-Komponente. Dies kann zu einer Grossen Anzahl an Unterklassen führen. Eine Alternative bietet das Bridge-Pattern: Die Maze-Komponenten interagieren mit einem abstrakten Implementor-Interface, das den Client von der konkreten Umsetzung (Android vs Swing) abstrahiert. 


```Java
interface MapRenderer {
    void updateGUI(String element);
}
```

Von diesem Implementor-Interface gibt es nun verschiedene konkrete Umsetzungen.


```Java
class PrintRenderer implements MapRenderer {
    public void updateGUI(String element) {
        System.out.println(element);
    }
}
```


```Java
class SwingRenderer implements MapRenderer {
    public void updateGUI(String element) {
        // Code to handle Java Swing output
        System.out.println("Java Swing: "+element);
    }
}
```


```Java
class AndroidRenderer implements MapRenderer {
    public void updateGUI(String element) {
        // Code to handle Android output
        System.out.println("Android: "+element);
    }
}
```

Statt für jede Art von Output eine eigene Unterklasse von `MapSite` zu definieren, bekommt die `MapSite` eine Referenz auf ein Implementor-Objekt, und kann damit unabhängig von der Implementierung der GUI umgesetzt werden.


```Java
public abstract class MapSite {
    protected MapRenderer impl;
    
    public MapSite(MapRenderer renderer) {
        this.impl = renderer;
    }

    public MapSite() {
        this(new PrintRenderer());
    }

    public abstract void draw();
}
```

Die konkreten Komponenten bedienen nun das `impl` Objekt.


```Java
class Door extends MapSite {

    public Door() {
        super();
    }

    public Door(MapRenderer renderer) {
        super(renderer);
    }

    @Override
    public void draw() {
        impl.updateGUI("Door");
    }
}
```


```Java
class Wall extends MapSite {
    
    public Wall() {
        super();
    }

    public Wall(MapRenderer renderer) {
        super(renderer);
    }
    
    @Override
    public void draw() {
        impl.updateGUI("Wall");
    }
}
```


```Java
public abstract class Composite extends MapSite {
    protected List<MapSite> elements = new ArrayList<>();

        
    protected Composite() {
        super();
    }

    protected Composite(MapRenderer renderer) {
        super(renderer);
    }
    
    
    public void add(MapSite x) {
        elements.add(x);
    }
    
    public List<MapSite> getChildren() {
        return elements;
    }
  
    @Override
    public void draw() {
        elements.forEach(e -> e.draw());
    }
}
```


```Java
class Room extends Composite {
    
    public Room() {
        super();
    }

    public Room(MapRenderer renderer) {
        super(renderer);
    }
    
    @Override
    public void draw() {
        impl.updateGUI("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
}
```


```Java
class Maze extends Composite {
    
    public Maze() {
        super();
    }

    public Maze(MapRenderer renderer) {
        super(renderer);
    }
    
    @Override
    public void draw() {
        impl.updateGUI("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
}
```

Nachdem wir `MapSite` so implementiert haben, dass per Default ein `PrintRenderer` verwendet wird, können wir wie gehabt unser Maze erzeugen.


```Java
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


Wir können auch einfach den Renderer wechseln.


```Java
SwingRenderer renderer = new SwingRenderer();

Maze maze = new Maze(renderer);
Room room1 = new Room(renderer);
Room room2 = new Room(renderer);
room1.add(new Wall(renderer));
room1.add(new Wall(renderer));
room1.add(new Wall(renderer));
room1.add(new Wall(renderer));
room2.add(new Wall(renderer));
room2.add(new Wall(renderer));
room2.add(new Wall(renderer));
room2.add(new Wall(renderer));
room1.add(new Door(renderer));
room2.add(new Door(renderer));

maze.add(room1);
maze.add(room2);

maze.draw();
```

    Java Swing: Maze consisting of: 
    Java Swing: Room consisting of:
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Door
    Java Swing: Room consisting of:
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Wall
    Java Swing: Door


Natürlich könnte man die Konfiguration des Renderers über die Creational Patterns noch schöner integrieren.

## Facade

Wir haben im obigen Beispiel keine tatsächliche Implementierung von `SwingRenderer` angegeben, und das mit gutem Grund: Um eine Swing-GUI zu verwenden muss eine Vielzahl an Klassen konfiguriert und verwendet werden. Das Facade-Entwurfsmuster versteckt diese Komplexität hinter einer Fassade, die ein einfacheres, abstrakteres Interface zur Verfügrung stellt.


```Java
class SwingFacade {
    public void drawWindow() {
        // ...
        // Actual Swing code
        System.out.println("...simulating Swing code...");
    }
    
    public void renderText(String text) {
        // ...
        // Actual Swing code
        System.out.println("...simulating Swing code...");
    }
    
    public void renderButtons() {
        // ...
        // Actual Swing code
        System.out.println("...simulating Swing code...");
    }
}
```


```Java
class SwingRenderer implements MapRenderer {
    private SwingFacade facade = new SwingFacade();
    
    public void updateGUI(String element) {
        facade.drawWindow();
        facade.renderText(element);
        facade.renderButtons();
    }
}
```


```Java
Door doorUsingFacade = new Door(new SwingRenderer());
```


```Java
doorUsingFacade.draw()
```

    ...simulating Swing code...
    ...simulating Swing code...
    ...simulating Swing code...


## Adapter

Angenommen es gibt bereits eine bestehende Komponente `WebRenderer`, die zwar genau die Funktionalität bietet, die wir in unserem Beispiel von Unterklassen des `MapRenderer` Interfaces erwarten, aber die Klasse `WebRenderer` implementiert dieses Interface nicht und hat anders benannte Methoden. Beispielsweise könnte diese Klasse aus einer verwendeten Bibliothek stammen.


```Java
class WebRenderer {
    void renderStuffOnTheWeb(String element) {
        System.out.println("Web rendering of " + element);
    }
}
```

Das Adapter-Entwurfsmuster erlaubt uns, so eine Komponente in unsere Interfaces einzupassen.


```Java
class WebRendererAdapter implements MapRenderer {
    private WebRenderer adaptee = new WebRenderer();

    @Override
    public void updateGUI(String element) {
        adaptee.renderStuffOnTheWeb(element);
    }
}
```


```Java
Door door = new Door(new WebRendererAdapter());
```


```Java
door.draw();
```

    Web rendering of Door


## Proxy

Ein Proxy ist ein Wrapper-Objekt das vom Client wie das tatsächliche Objekt benutzt wird, und dabei den Zugriff auf das tatsächliche Objekt steuert. Als Beispiel verwenden wir einen Proxy um Lazy Initialisation durchzuführen.


```Java
class WebRendererProxy implements MapRenderer {
    private MapRenderer realSubject = null;
    
    private void setupCostlyInitialisation() {
        if (realSubject == null) {
            System.out.println("Performing costly initialisation");
            realSubject = new WebRendererAdapter();
        }
    }
   
    public void updateGUI(String element) {
        setupCostlyInitialisation();
        realSubject.updateGUI(element);
    }
}
```

Der Proxy ist eine für den Client transparente Schicht zur tatsächlichen Implementierung; wir verwenden diese Schicht für Lazy Initialisation, andere Anwendungsbeispiele sind Remote Proxies, Logging Proxies, oder Zugriffskontrolle.


```Java
Door door = new Door(new WebRendererProxy());
```


```Java
door.draw()
```

    Performing costly initialisation
    Web rendering of Door


## Visitor

Angenommen wir wollen eine komplexe Objektstruktur um Funktionalität erweitern, ohne dabei die neue Funktionalität direkt in den Klassen der Objektstruktur zu implementieren. Beispielsweise könnten wir ein Maze als XML Code ausgeben lassen. Ein einfacher Ansatz wäre, die Funktionalität in eine eigene Klasse zu packen.


```Java
class XMLConverter {
    void visitDoor(Door door) {
        // print out xml for door
        System.out.println("Door");
    }
    
    void visitWall(Wall wall) {
        // print out xml for wall
        System.out.println("Wall");
    }

    void visitRoom(Room room) {
        // print out xml for room
        System.out.println("Room");

    }

    void visitMaze(Maze maze) {
        // print out xml for maze
        System.out.println("Maze");
    }  
}
```

Das Problem dabei ist, wie wir entscheiden welche der Funktionen nun aufgerufen werden soll. Wenn wir in irgendeiner Form über unsere Objektstruktur iterieren, müssen wir dabei bei jedem Objekt aufgrund der Klasse entscheiden welche Funktion die passende ist.


```Java
XMLConverter converter = new XMLConverter();

for (MapSite mapSite : Arrays.asList(maze, door)) {
    if (mapSite instanceof Door) {
        converter.visitDoor((Door) mapSite);
    } else if (mapSite instanceof Wall) {
        converter.visitWall((Wall) mapSite);
    } else if (mapSite instanceof Room) {
        converter.visitRoom((Room) mapSite);
    } else if (mapSite instanceof Maze) {
        converter.visitMaze((Maze) mapSite);
    } 
}
```

    Maze
    Door


Das ist umständlich, und man fragt sich vielleicht warum man das nicht einfach mit dynamischem Typing löst:


```Java
class XMLConverter {
    void visit(Door door) {
        // print out xml for door
        System.out.println("Door");
    }
    
    void visit(Wall wall) {
        // print out xml for wall
        System.out.println("Wall");

    }

    void visit(Room room) {
        // print out xml for room
        System.out.println("Room");

    }

    void visit(Maze maze) {
        // print out xml for maze
        System.out.println("Maze");
    }  

    void visit(MapSite mapsite) {
        // fallback, we don't actually want this to be called
        System.out.println("MapSite");
    }  

}
```


```Java
new XMLConverter().visit(maze)
```

    Maze


...scheint ja zu funktionieren. Allerdings funktioniert es nicht mehr, wenn der Code die Abstraktion benutzt, so wie es wohl der Fall wäre wenn man über mehrere Elemente iteriert.


```Java
XMLConverter converter = new XMLConverter();

for (MapSite mapSite : Arrays.asList(maze, door)) {
    converter.visit(mapSite);
}
```

    MapSite
    MapSite


Im Visitor-Pattern wird dieses Problem mit Hilfe von _Double Dispatch_ gelöst. Leider habe ich noch keine gute Möglichkeit gefunden, die Visitor-Klassen/Interfaces in geeigneter Reihenfolge im Jupyter Notebook, sondern muss einen Umweg in den Definition gehen. Diese doppelten Definitionen wären normalerweise nicht notwendig, sondern sind nur ein Problem das hier im Notebook besteht!


```Java
public interface Visitor {
    // Jupyter-Hack: Wird weiter unten vervollständigt
}
```

Wir müssen unsere Klassenhierarchie um ein Interface erweitern, sodass alle Objekte eine `accept` Methode erhalten, die das Double Dispatch durchführen kann.


```Java
public interface Visitable {
    void accept(Visitor v);
}
```

Eine `MapSite` soll `Visitable` sein, d.h. sie muss die Methode `accept` implementieren.


```Java
public abstract class MapSite implements Visitable {
    protected MapRenderer impl;
    
    public MapSite(MapRenderer renderer) {
        this.impl = renderer;
    }

    public MapSite() {
        this(new PrintRenderer());
    }

    public abstract void draw();
    
    public abstract void accept(Visitor v);
}
```

Nachdem sich das Interface verändert hat, müssen alle Klassen der Objektstruktur so erweitert werden, dass sie auch die neue `accept` Methode implementieren.

An dieser Stelle nun der Jupyter-Hack: Die `accept` Methode bleibt vorerst noch leer, normalerweise würde hier gleich der Code folgen...


```Java
class Door extends MapSite {

    @Override
    public void draw() {
        impl.updateGUI("Door");
    }
    
    @Override
    public void accept(Visitor v) {
        // Jupyter-Hack: Wird weiter unten vervollständigt
    }
}
```


```Java
class Wall extends MapSite {
    @Override
    public void draw() {
        impl.updateGUI("Wall");
    }
    
    @Override
    public void accept(Visitor v) {
        // Jupyter-Hack: Wird weiter unten vervollständigt
    }
}
```


```Java
public abstract class Composite extends MapSite {
    protected List<MapSite> elements = new ArrayList<>();

    public void add(MapSite x) {
        elements.add(x);
    }

    public List<MapSite> getChildren() {
        return elements;
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
        impl.updateGUI("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        // Jupyter-Hack: Wird weiter unten vervollständigt
        elements.forEach(x -> x.accept(v));
    }
}
```


```Java
class Maze extends Composite {
    @Override
    public void draw() {
        impl.updateGUI("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        // Jupyter-Hack: Wird weiter unten vervollständigt
        elements.forEach(x -> x.accept(v));
    }
}
```

Nachdem wir das gemacht haben, ist nun auch Jupyter bereit die vollständige Definition zu akzeptieren. Unser Visitor benötigt eine eigene Methode für jedes Element der Objektstruktur.


```Java
public interface Visitor {
    void visitDoor(Door d);
    void visitRoom(Room r);
    void visitMaze(Maze m);
    void visitWall(Wall w);
}
```

Die jeweiligen `accept` Methoden der konkreten Klassen der Objektstruktur rufen dann die entsprechende Methode auf (das ist das Double Dispatch).


```Java
class Door extends MapSite {

    @Override
    public void draw() {
        impl.updateGUI("Door");
    }
    
    @Override
    public void accept(Visitor v) {
        v.visitDoor(this);
    }
}
```


```Java
class Wall extends MapSite {
    @Override
    public void draw() {
        impl.updateGUI("Wall");
    }
    
    @Override
    public void accept(Visitor v) {
       v.visitWall(this);
    }
}
```


```Java
public abstract class Composite extends MapSite {
    protected List<MapSite> elements = new ArrayList<>();

    public void add(MapSite x) {
        elements.add(x);
    }

    public List<MapSite> getChildren() {
        return elements;
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
        impl.updateGUI("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        v.visitRoom(this);
        elements.forEach(x -> x.accept(v));
    }
}
```


```Java
class Maze extends Composite {
    @Override
    public void draw() {
        impl.updateGUI("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        v.visitMaze(this);
        elements.forEach(x -> x.accept(v));
    }
}
```


```Java
public class PrintVisitor implements Visitor {
    @Override
    public void visitDoor(Door d) {
        System.out.println("Here's a door");
    }

    @Override
    public void visitWall(Wall w) {
        System.out.println("Here's a wall");
    }

    @Override
    public void visitRoom(Room r) {
        System.out.println("Here's a room");
    }

    @Override
    public void visitMaze(Maze m) {
        System.out.println("Here's a maze");
    }

}
```


```Java
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
```


```Java
PrintVisitor visitor = new PrintVisitor()
```


```Java
room1.accept(visitor)
```

    Here's a room
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a door



```Java
maze.accept(visitor)
```

    Here's a maze
    Here's a room
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a door
    Here's a room
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a wall
    Here's a door


In dieser Implementierung haben wir die Traversierung der Objektstruktur in der Objektstruktur selbst umgesetzt; d.h. Composite Objekte rufen innerhalb der `accept` Methode die `accept` Methode ihrer Kinder auf. Es ist auch Möglich dies im Visitor selbst umzusetzen. Dies machen wir in folgendem Beispiel.


```Java
public class PrintXMLVisitor implements Visitor {
    @Override
    public void visitDoor(Door d) {
        System.out.println("    <door></door>");
    }

    @Override
    public void visitWall(Wall w) {
        System.out.println("    <wall></wall>");
    }

    @Override
    public void visitRoom(Room r) {
        System.out.println("  <room>");
        for (MapSite mapSite : r.getChildren()) {
            mapSite.accept(this);
        }
        System.out.println("  </room>");
    }

    @Override
    public void visitMaze(Maze m) {
        System.out.println("<maze>");
        for (MapSite mapSite : m.getChildren()) {
            mapSite.accept(this);
        }
        System.out.println("</maze>");
    }

}
```

Nachdem die Traversierung der Kinderelemente nun im Visitor passiert, müssen wir den gleichen Code aus der Objektstruktur selbst entfernen.


```Java
class Room extends Composite {
    @Override
    public void draw() {
        impl.updateGUI("Room consisting of:");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        v.visitRoom(this);
    }
}
```


```Java
class Maze extends Composite {
    @Override
    public void draw() {
        impl.updateGUI("Maze consisting of: ");
        elements.forEach(x -> x.draw());
    }
    
    @Override
    public void accept(Visitor v) {
        v.visitMaze(this);
    }
}
```


```Java
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
```


```Java
PrintXMLVisitor visitor = new PrintXMLVisitor()
```


```Java
maze.accept(visitor)
```

    <maze>
      <room>
        <wall></wall>
        <wall></wall>
        <wall></wall>
        <wall></wall>
        <door></door>
      </room>
      <room>
        <wall></wall>
        <wall></wall>
        <wall></wall>
        <wall></wall>
        <door></door>
      </room>
    </maze>


## Command

Das Command-Pattern erlaubt es uns, auszuführende Aktionen als Objekte darzustellen. Diese Objekte lassen sich herumreichen, und es ist möglich erst zu geeigneten Zeiten die Aktionen ausführen.


```Java
interface Command {
    void execute();
}
```

Wir erzeugen ein Dummy-Command für die Maze-Objektstruktur, das einfach nur aus einer Ausgabe besteht.


```Java
public class PrintCommand implements Command {
    private MapSite site;
    
    public PrintCommand(MapSite site) {
        this.site = site;
    }
    
    @Override
    public void execute() {
        System.out.println("Executing command on " + site.getClass().getSimpleName());
    }
}
```


```Java
public class OtherCommand implements Command {
    private MapSite site;
    
    public OtherCommand(MapSite site) {
        this.site = site;
    }

    @Override
    public void execute() {
        // site.otherStuff();
        // do other stuff

    }
}
```

Nun möchten wir ein Command für jedes Objekt innerhalb einer Objektstruktur einsammeln. Dafür eignet das Visitor Pattern.


```Java
public class CommandVisitor implements Visitor {

    private List<Command> commands = new ArrayList<>();

    public List<Command> getCommands() {
        return commands;
    }

    @Override
    public void visitDoor(Door d) {
        commands.add(new PrintCommand(d));
    }

    @Override
    public void visitWall(Wall w) {
        commands.add(new PrintCommand(w));
    }

    @Override
    public void visitRoom(Room r) {
        commands.add(new PrintCommand(r));
        r.getChildren().forEach(m -> m.accept(this));
    }

    @Override
    public void visitMaze(Maze m) {
        commands.add(new PrintCommand(m));
        m.getChildren().forEach(ms -> ms.accept(this));
    }
}
```

Nun können wir unsere Commands erzeugen, und nach belieben Ausführen.


```Java
CommandVisitor visitor = new CommandVisitor();
maze.accept(visitor);
```


```Java
List<Command> commands = visitor.getCommands();
```


```Java
commands.size()
```




    13




```Java
for (Command command : visitor.getCommands()) {
    command.execute();
}
```

    Executing command on Maze
    Executing command on Room
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door
    Executing command on Room
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door


## Strategy

Das Strategy-Muster wird verwendet, wenn zahlreiche zusammenhängende Klassen sich nur im Verhalten unterscheiden, oder verschiedene Varianten eines Algorithmus benötigt werden. Ein Beispiel für unser Maze-Szenario wären unterschiedliche Traversierungsstrategien, z.B. Preorder/Postorder. Das Strategy-Pattern erlaubt es, diese Strategien nach Belieben (und zur Laufzeit) zu wechseln, und auch einfach neue Strategien einzubauen.


```Java
interface TraversalStrategy {
    List<MapSite> traverse(MapSite element);
}
```


```Java
public class PreorderTraversalStrategy implements TraversalStrategy {
    public List<MapSite> traverse(MapSite element) {
        List<MapSite> elements = new ArrayList<MapSite>();
        elements.add(element);
        if (element instanceof Composite) {
            for (MapSite map : ((Composite)element).getChildren()) {
                elements.addAll(traverse(map));
            }
        }
        return elements;
    }
}
```


```Java
public class PostorderTraversalStrategy implements TraversalStrategy {
    public List<MapSite> traverse(MapSite element) {
        List<MapSite> elements = new ArrayList<MapSite>();
        if (element instanceof Composite) {
            for (MapSite map : ((Composite)element).getChildren()) {
                elements.addAll(traverse(map));
            }
        }
        elements.add(element);

        return elements;
    }
}
```


```Java
TraversalStrategy strategy = new PreorderTraversalStrategy();
List<MapSite> preorderElements = strategy.traverse(maze)
```


```Java
for (MapSite element : preorderElements) {
    new PrintCommand(element).execute();
}
```

    Executing command on Maze
    Executing command on Room
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door
    Executing command on Room
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door



```Java
TraversalStrategy strategy = new PostorderTraversalStrategy();
List<MapSite> postorderElements = strategy.traverse(maze);
```


```Java
for (MapSite element : postorderElements) {
    new PrintCommand(element).execute();
}
```

    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door
    Executing command on Room
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Wall
    Executing command on Door
    Executing command on Room
    Executing command on Maze

