# Design Patterns: Behavioral - State

## State Pattern - Simple Explanation

**SITUATION: YOU’RE MANAGING OBJECT STATES**  
You’re building a media player with states like `Playing`, `Paused`. 

Initially, you use flags: 

#### Java
```java
if (state == "playing") { 
    play(); 
} else { 
    pause(); 
}
``` 

All good.

**THEN THE STATES GROW**  
Now you have:  
- `Stopped`  
- `Buffering`  
- `Ended`  

You’re stuck with: 
```java
if (state == "playing") { 
    /* ... */ 
} else if (state == "paused") { 
    /* ... */ 
}
``` 
Everywhere. It’s a mess.

**WHAT’S THE PROBLEM?**  
- State logic is scattered across the code.  
- Adding a new state means changing all conditionals.  
- It violates the Open/Closed Principle.  
- Bugs creep in as states grow.

**HOW STATE SAVES YOU**  
Instead of conditionals, you create `State` classes like `PlayingState`, `PausedState`. 

You say: “Hey State, handle this action for me.” 

And the State:  
- Encapsulates behavior for each state.  
- Lets the player delegate to the current state.  
- Makes adding new states easy.  

Now you just: 
```java
player.setState(new PlayingState()); 
player.play();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface State {  
    void play(MediaPlayer player);  
    void pause(MediaPlayer player);  
}  
class PlayingState implements State {  
    public void play(MediaPlayer player) {  
        System.out.println("Already playing");  
    }  
    public void pause(MediaPlayer player) {  
        player.setState(new PausedState());  
        System.out.println("Paused");  
    }  
}  
class PausedState implements State {  
    public void play(MediaPlayer player) {  
        player.setState(new PlayingState());  
        System.out.println("Playing");  
    }  
    public void pause(MediaPlayer player) {  
        System.out.println("Already paused");  
    }  
}  
class MediaPlayer {  
    private State state;  
    public MediaPlayer() {  
        this.state = new PausedState();  
    }  
    public void setState(State state) {  
        this.state = state;  
    }  
    public void play() {  
        state.play(this);  
    }  
    public void pause() {  
        state.pause(this);  
    }  
}  
// Usage  
MediaPlayer player = new MediaPlayer();  
player.play();  
```

#### TypeScript  
```typescript  
interface State {  
    play(player: MediaPlayer): void;  
    pause(player: MediaPlayer): void;  
}  
class PlayingState implements State {  
    play(player: MediaPlayer) {  
        console.log("Already playing");  
    }  
    pause(player: MediaPlayer) {  
        player.setState(new PausedState());  
        console.log("Paused");  
    }  
}  
class PausedState implements State {  
    play(player: MediaPlayer) {  
        player.setState(new PlayingState());  
        console.log("Playing");  
    }  
    pause(player: MediaPlayer) {  
        console.log("Already paused");  
    }  
}  
class MediaPlayer {  
    private state: State;  
    constructor() {  
        this.state = new PausedState();  
    }  
    setState(state: State) {  
        this.state = state;  
    }  
    play() {  
        this.state.play(this);  
    }  
    pause() {  
        this.state.pause(this);  
    }  
}  
// Usage  
const player = new MediaPlayer();  
player.play();  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class State(ABC):  
    @abstractmethod  
    def play(self, player):  
        pass  
    @abstractmethod  
    def pause(self, player):  
        pass  
class PlayingState(State):  
    def play(self, player):  
        print("Already playing")  
    def pause(self, player):  
        player.set_state(PausedState())  
        print("Paused")  
class PausedState(State):  
    def play(self, player):  
        player.set_state(PlayingState())  
        print("Playing")  
    def pause(self, player):  
        print("Already paused")  
class MediaPlayer:  
    def __init__(self):  
        self.state = PausedState()  
    def set_state(self, state):  
        self.state = state  
    def play(self):  
        self.state.play(self)  
    def pause(self):  
        self.state.pause(self)  
# Usage  
player = MediaPlayer()  
player.play()  
```

#### Go  
```go  
package main  
import "fmt"  
type State interface {  
    Play(player *MediaPlayer)  
    Pause(player *MediaPlayer)  
}  
type PlayingState struct{}  
func (s *PlayingState) Play(player *MediaPlayer) {  
    fmt.Println("Already playing")  
}  
func (s *PlayingState) Pause(player *MediaPlayer) {  
    player.SetState(&PausedState{})  
    fmt.Println("Paused")  
}  
type PausedState struct{}  
func (s *PausedState) Play(player *MediaPlayer) {  
    player.SetState(&PlayingState{})  
    fmt.Println("Playing")  
}  
func (s *PausedState) Pause(player *MediaPlayer) {  
    fmt.Println("Already paused")  
}  
type MediaPlayer struct {  
    state State  
}  
func (p *MediaPlayer) SetState(state State) {  
    p.state = state  
}  
func (p *MediaPlayer) Play() {  
    p.state.Play(p)  
}  
func (p *MediaPlayer) Pause() {  
    p.state.Pause(p)  
}  
// Usage  
func main() {  
    player := &MediaPlayer{state: &PausedState{}}  
    player.Play()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for state transitions, ensuring type safety.  
- Python uses a dynamic approach with method overriding.  
- Go uses interfaces and composition, avoiding inheritance.  
- All encapsulate state-specific behavior.

**WHEN SHOULD YOU USE IT?**  
- When an object’s behavior changes based on its state.  
- When you have many states with complex transitions.  
- When you want to avoid long conditional chains.

**WHERE YOU’VE ALREADY SEEN IT**  
`TCPConnection` states (e.g., established, closed).  
`Game states` in game development (e.g., menu, playing).