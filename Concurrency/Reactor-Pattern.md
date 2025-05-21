# Design Patterns: Concurrency - Reactor

## Reactor Pattern - Simple Explanation

**SITUATION: YOU'RE HANDLING MULTIPLE I/O EVENTS**  
You have a server handling network connections. 

Initially, you have: 

#### Java
```java
while (true) {
    Socket socket = serverSocket.accept();
    handleConnection(socket);
}
``` 
All good.

**THEN THE REQUIREMENTS GROW**  
Now you need:  
- Multiple connections
- Different event types
- Non-blocking I/O
- Resource efficiency

You're stuck with: 
```java
// Thread per connection - doesn't scale!
while (true) {
    Socket socket = serverSocket.accept();
    new Thread(() -> handleConnection(socket)).start();
}
``` 
Thread explosion!

**WHAT'S THE PROBLEM?**  
- One thread per connection doesn't scale
- Blocking I/O wastes resources
- Hard to manage multiple event types
- Complex resource management

**HOW REACTOR SAVES YOU**  
Use a single thread to handle multiple events.

You say: "Tell me when something happens, I'll handle it." 

And the Reactor:  
- Multiplexes service requests
- Dispatches to handlers
- Non-blocking I/O
- Efficient resource usage

Now you just: 
```java
Reactor reactor = new Reactor();
reactor.register(socket, READ, new AcceptHandler());
reactor.run();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
class Reactor {
    private final Selector selector;
    private final Map<SelectableChannel, EventHandler> handlers;
    
    public Reactor() throws IOException {
        this.selector = Selector.open();
        this.handlers = new ConcurrentHashMap<>();
    }
    
    public void register(SelectableChannel channel, 
                        int ops, 
                        EventHandler handler) throws IOException {
        channel.configureBlocking(false);
        channel.register(selector, ops);
        handlers.put(channel, handler);
    }
    
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                selector.select();
                Set<SelectionKey> selected = selector.selectedKeys();
                for (SelectionKey key : selected) {
                    dispatch(key);
                }
                selected.clear();
            }
        } catch (IOException e) {
            // Handle error
        }
    }
    
    private void dispatch(SelectionKey key) throws IOException {
        if (key.isAcceptable()) {
            handlers.get(key.channel()).handleAccept();
        } else if (key.isReadable()) {
            handlers.get(key.channel()).handleRead();
        } else if (key.isWritable()) {
            handlers.get(key.channel()).handleWrite();
        }
    }
}

interface EventHandler {
    void handleAccept() throws IOException;
    void handleRead() throws IOException;
    void handleWrite() throws IOException;
}

class ServerHandler implements EventHandler {
    private final ServerSocketChannel serverChannel;
    private final Reactor reactor;
    
    public ServerHandler(Reactor reactor, int port) throws IOException {
        this.reactor = reactor;
        this.serverChannel = ServerSocketChannel.open();
        serverChannel.socket().bind(new InetSocketAddress(port));
        reactor.register(serverChannel, SelectionKey.OP_ACCEPT, this);
    }
    
    @Override
    public void handleAccept() throws IOException {
        SocketChannel clientChannel = serverChannel.accept();
        if (clientChannel != null) {
            reactor.register(clientChannel, 
                           SelectionKey.OP_READ,
                           new ClientHandler(reactor, clientChannel));
        }
    }
}
```

#### TypeScript  
```typescript  
interface EventEmitter {
    on(event: string, handler: Function): void;
    emit(event: string, ...args: any[]): void;
}

class Reactor {
    private handlers: Map<string, Set<Function>>;
    
    constructor() {
        this.handlers = new Map();
    }
    
    register(event: string, handler: Function): void {
        if (!this.handlers.has(event)) {
            this.handlers.set(event, new Set());
        }
        this.handlers.get(event)!.add(handler);
    }
    
    dispatch(event: string, ...args: any[]): void {
        const handlers = this.handlers.get(event);
        if (handlers) {
            handlers.forEach(handler => handler(...args));
        }
    }
}

class Server {
    private reactor: Reactor;
    
    constructor() {
        this.reactor = new Reactor();
        this.setupHandlers();
    }
    
    private setupHandlers(): void {
        this.reactor.register('connection', this.handleConnection);
        this.reactor.register('data', this.handleData);
        this.reactor.register('close', this.handleClose);
    }
    
    private handleConnection = (socket: any): void => {
        console.log('New connection');
    }
    
    private handleData = (data: any): void => {
        console.log('Received data:', data);
    }
    
    private handleClose = (): void => {
        console.log('Connection closed');
    }
}
```

#### Python  
```python  
import asyncio
from typing import Dict, Callable, Any

class Reactor:
    def __init__(self):
        self.handlers: Dict[str, Callable] = {}
        self.loop = asyncio.get_event_loop()
    
    def register(self, event: str, handler: Callable):
        self.handlers[event] = handler
    
    async def dispatch(self, event: str, *args, **kwargs):
        if event in self.handlers:
            await self.handlers[event](*args, **kwargs)
    
    def run(self):
        self.loop.run_forever()
    
    def stop(self):
        self.loop.stop()

class Server:
    def __init__(self, host: str, port: int):
        self.reactor = Reactor()
        self.host = host
        self.port = port
    
    async def start(self):
        server = await asyncio.start_server(
            self.handle_connection, self.host, self.port
        )
        async with server:
            await server.serve_forever()
    
    async def handle_connection(self, 
                              reader: asyncio.StreamReader, 
                              writer: asyncio.StreamWriter):
        try:
            while True:
                data = await reader.read(100)
                if not data:
                    break
                await self.reactor.dispatch('data', data)
                writer.write(data)
                await writer.drain()
        finally:
            writer.close()
            await writer.wait_closed()
            await self.reactor.dispatch('close')

# Usage
async def main():
    server = Server('localhost', 8080)
    server.reactor.register('data', lambda data: print(f"Got: {data}"))
    await server.start()

if __name__ == "__main__":
    asyncio.run(main())
```

#### Go  
```go  
package main

import (
    "fmt"
    "net"
)

type EventHandler interface {
    HandleEvent(event Event)
}

type Event struct {
    Type    string
    Data    interface{}
    Channel net.Conn
}

type Reactor struct {
    handlers map[string][]EventHandler
    events   chan Event
}

func NewReactor() *Reactor {
    return &Reactor{
        handlers: make(map[string][]EventHandler),
        events:   make(chan Event),
    }
}

func (r *Reactor) Register(eventType string, handler EventHandler) {
    r.handlers[eventType] = append(r.handlers[eventType], handler)
}

func (r *Reactor) Dispatch(event Event) {
    if handlers, exists := r.handlers[event.Type]; exists {
        for _, handler := range handlers {
            handler.HandleEvent(event)
        }
    }
}

func (r *Reactor) Run() {
    for event := range r.events {
        r.Dispatch(event)
    }
}

type Server struct {
    reactor *Reactor
    listener net.Listener
}

func NewServer(address string) (*Server, error) {
    listener, err := net.Listen("tcp", address)
    if err != nil {
        return nil, err
    }
    
    return &Server{
        reactor:  NewReactor(),
        listener: listener,
    }, nil
}

func (s *Server) Start() {
    go s.reactor.Run()
    
    for {
        conn, err := s.listener.Accept()
        if err != nil {
            fmt.Printf("Error accepting connection: %v\n", err)
            continue
        }
        
        go s.handleConnection(conn)
    }
}

func (s *Server) handleConnection(conn net.Conn) {
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            s.reactor.events <- Event{Type: "close", Channel: conn}
            return
        }
        
        s.reactor.events <- Event{
            Type:    "data",
            Data:    buffer[:n],
            Channel: conn,
        }
    }
}
```

**Key Differences:**  
- Java uses NIO's Selector for event multiplexing
- TypeScript implements event-driven approach
- Python leverages asyncio for async I/O
- Go uses channels and goroutines for concurrency

**WHEN SHOULD YOU USE IT?**  
- When handling multiple I/O connections
- When scalability is important
- When you need non-blocking operations
- When resource efficiency is critical

**WHERE YOU'VE ALREADY SEEN IT**  
- Node.js event loop
- Netty framework
- Redis event loop
- Network servers
