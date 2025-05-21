# Design Patterns: Structural - Facade

## Facade Pattern - Simple Explanation

**SITUATION: YOU’RE DEALING WITH A COMPLEX SUBSYSTEM**  
You’re building a home theater app with components like `DVDPlayer`, `Projector`, and `Speakers`. 

To watch a movie, you need to: 

#### Java
```java
dvdPlayer.turnOn(); 
projector.setInput("DVD"); 
speakers.setVolume(5); 
dvdPlayer.play();
``` 

All good.

**THEN THE REQUIREMENTS GROW**  
Now you need to add more steps: turn off lights, adjust the screen, etc. 

You’re stuck repeating: 

```java
lights.dim(); 
screen.lower(); 
dvdPlayer.turnOn(); 
projector.setInput("DVD"); 
speakers.setVolume(5); 
dvdPlayer.play();
``` 
Everywhere. 

It’s a lot of steps.

**WHAT’S THE PROBLEM?**  
- The subsystem is complex with many steps to perform a task.  
- Clients need to know how all components work together.  
- Code becomes tightly coupled to the subsystem.  
- It’s repetitive and error-prone.

**HOW FACADE SAVES YOU**  
Instead of managing all components directly, you create a `HomeTheaterFacade`. 

You say: “Hey Facade, just watch a movie for me.” 

And the Facade:  
- Hides the complexity of the subsystem.  
- Provides a simple interface to perform tasks.  
- Coordinates all components for you.  

Now you just: 
```java
HomeTheaterFacade theater = new HomeTheaterFacade(); 
theater.watchMovie();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
class DVDPlayer {  
    public void turnOn() {  
        System.out.println("DVD Player on");  
    }  
    public void play() {  
        System.out.println("Playing DVD");  
    }  
}  
class Projector {  
    public void setInput(String input) {  
        System.out.println("Projector set to " + input);  
    }  
}  
class Speakers {  
    public void setVolume(int level) {  
        System.out.println("Speakers volume set to " + level);  
    }  
}  
class HomeTheaterFacade {  
    private DVDPlayer dvd;  
    private Projector projector;  
    private Speakers speakers;  
    public HomeTheaterFacade() {  
        this.dvd = new DVDPlayer();  
        this.projector = new Projector();  
        this.speakers = new Speakers();  
    }  
    public void watchMovie() {  
        dvd.turnOn();  
        projector.setInput("DVD");  
        speakers.setVolume(5);  
        dvd.play();  
    }  
}  
// Usage  
HomeTheaterFacade theater = new HomeTheaterFacade();  
theater.watchMovie();  
```

#### TypeScript  
```typescript  
class DVDPlayer {  
    turnOn() {  
        console.log("DVD Player on");  
    }  
    play() {  
        console.log("Playing DVD");  
    }  
}  
class Projector {  
    setInput(input: string) {  
        console.log(`Projector set to ${input}`);  
    }  
}  
class Speakers {  
    setVolume(level: number) {  
        console.log(`Speakers volume set to ${level}`);  
    }  
}  
class HomeTheaterFacade {  
    private dvd: DVDPlayer;  
    private projector: Projector;  
    private speakers: Speakers;  
    constructor() {  
        this.dvd = new DVDPlayer();  
        this.projector = new Projector();  
        this.speakers = new Speakers();  
    }  
    watchMovie() {  
        this.dvd.turnOn();  
        this.projector.setInput("DVD");  
        this.speakers.setVolume(5);  
        this.dvd.play();  
    }  
}  
// Usage  
const theater = new HomeTheaterFacade();  
theater.watchMovie();  
```

#### Python  
```python  
class DVDPlayer:  
    def turn_on(self):  
        print("DVD Player on")  
    def play(self):  
        print("Playing DVD")  
class Projector:  
    def set_input(self, input):  
        print(f"Projector set to {input}")  
class Speakers:  
    def set_volume(self, level):  
        print(f"Speakers volume set to {level}")  
class HomeTheaterFacade:  
    def __init__(self):  
        self.dvd = DVDPlayer()  
        self.projector = Projector()  
        self.speakers = Speakers()  
    def watch_movie(self):  
        self.dvd.turn_on()  
        self.projector.set_input("DVD")  
        self.speakers.set_volume(5)  
        self.dvd.play()  
# Usage  
theater = HomeTheaterFacade()  
theater.watch_movie()  
```

#### Go  
```go  
package main  
import "fmt"  
type DVDPlayer struct{}  
func (d *DVDPlayer) TurnOn() {  
    fmt.Println("DVD Player on")  
}  
func (d *DVDPlayer) Play() {  
    fmt.Println("Playing DVD")  
}  
type Projector struct{}  
func (p *Projector) SetInput(input string) {  
    fmt.Printf("Projector set to %s\n", input)  
}  
type Speakers struct{}  
func (s *Speakers) SetVolume(level int) {  
    fmt.Printf("Speakers volume set to %d\n", level)  
}  
type HomeTheaterFacade struct {  
    dvd *DVDPlayer  
    projector *Projector  
    speakers *Speakers  
}  
func (h *HomeTheaterFacade) WatchMovie() {  
    h.dvd.TurnOn()  
    h.projector.SetInput("DVD")  
    h.speakers.SetVolume(5)  
    h.dvd.Play()  
}  
// Usage  
func main() {  
    theater := &HomeTheaterFacade{&DVDPlayer{}, &Projector{}, &Speakers{}}  
    theater.WatchMovie()  
}  
```

**Key Differences:**  
- Java and TypeScript use classes with clear encapsulation.  
- Python leverages its dynamic nature, keeping the facade simple.  
- Go uses structs and methods, focusing on composition.  
- All simplify interaction with complex subsystems.

**WHEN SHOULD YOU USE IT?**  
- When you have a complex subsystem with many interactions.  
- When you want to provide a simple interface to clients.  
- When you need to decouple clients from the subsystem.

**WHERE YOU’VE ALREADY SEEN IT**  
`JDBC` in Java (simplifies database access).  
`SLF4J` logging facade (unifies different logging frameworks).