# Design Patterns: Architectural - MVVM (Model-View-ViewModel)

## MVVM Pattern - Simple Explanation

**SITUATION: YOU'RE BUILDING A MODERN UI**  
You have a UI that needs to stay in sync with data. 

Initially, you have: 

#### TypeScript
```typescript
class UserScreen {
    private user: User;
    
    updateName(name: string) {
        this.user.name = name;
        this.updateUI();
    }
}
``` 
All good.

**THEN THE REQUIREMENTS EVOLVE**  
Now you need:  
- Two-way data binding
- Data transformation for display
- Input validation
- State management
- Testable UI logic

You're stuck with: 
```typescript
class UserScreen {
    private user: User;
    private formattedName: string;
    private isValid: boolean;
    
    updateName(name: string) {
        this.isValid = this.validate(name);
        if (this.isValid) {
            this.user.name = name;
            this.formattedName = this.formatName(name);
            this.updateUI();
            this.saveUser();
        }
    }
}
``` 
UI logic everywhere!

**WHAT'S THE PROBLEM?**  
- View knows too much about data logic
- Hard to test UI behavior
- No separation of display logic
- Complex state management
- Difficult two-way binding

**HOW MVVM SAVES YOU**  
Introduce a ViewModel that handles UI logic and state.

You say: "ViewModel, manage the view's data and behavior." 

And MVVM:  
- Separates UI logic from views
- Enables data binding
- Makes UI testable
- Manages view state

Now you just: 
```typescript
// View binds to ViewModel properties
<input [(ngModel)]="viewModel.name">

// ViewModel handles logic
class UserViewModel {
    @Observable name: string;
    
    updateName(value: string) {
        this.name = this.formatName(value);
    }
}
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### TypeScript (Angular-style)  
```typescript  
// Model
interface User {
    id: number;
    name: string;
    email: string;
}

// ViewModel
class UserViewModel {
    private _user: User;
    private _nameSubject = new BehaviorSubject<string>('');
    private _errorSubject = new BehaviorSubject<string>('');
    
    name$ = this._nameSubject.asObservable();
    error$ = this._errorSubject.asObservable();
    
    constructor(private userService: UserService) {}
    
    updateName(name: string) {
        if (this.validateName(name)) {
            this._nameSubject.next(name);
            this._errorSubject.next('');
            this.saveUser();
        } else {
            this._errorSubject.next('Invalid name');
        }
    }
    
    private validateName(name: string): boolean {
        return name && name.length >= 2;
    }
    
    private async saveUser() {
        const updated = await this.userService.updateUser({
            ...this._user,
            name: this._nameSubject.value
        });
        this._user = updated;
    }
}

// View (Component)
@Component({
    template: `
        <div>
            <input [ngModel]="name$ | async"
                   (ngModelChange)="updateName($event)">
            <div class="error">{{ error$ | async }}</div>
        </div>
    `
})
class UserComponent {
    constructor(public viewModel: UserViewModel) {}
    
    updateName(name: string) {
        this.viewModel.updateName(name);
    }
}
```

#### C# (WPF-style)  
```csharp  
// Model
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// ViewModel
public class UserViewModel : INotifyPropertyChanged
{
    private User _user;
    private string _name;
    private string _error;
    
    public event PropertyChangedEventHandler PropertyChanged;
    
    public string Name
    {
        get => _name;
        set
        {
            _name = value;
            OnPropertyChanged(nameof(Name));
            ValidateName();
        }
    }
    
    public string Error
    {
        get => _error;
        set
        {
            _error = value;
            OnPropertyChanged(nameof(Error));
        }
    }
    
    private void ValidateName()
    {
        if (string.IsNullOrEmpty(Name) || Name.Length < 2)
        {
            Error = "Invalid name";
            return;
        }
        
        Error = string.Empty;
        SaveUser();
    }
    
    private async Task SaveUser()
    {
        _user.Name = Name;
        await _userService.UpdateUserAsync(_user);
    }
    
    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, 
            new PropertyChangedEventArgs(propertyName));
    }
}

// View (XAML)
<StackPanel>
    <TextBox Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}" />
    <TextBlock Text="{Binding Error}" 
              Visibility="{Binding Error, Converter={StaticVisibilityConverter}}" />
</StackPanel>
```

#### Python (PyQt-style)  
```python  
from PyQt5.QtCore import QObject, pyqtSignal, pyqtProperty

# Model
class User:
    def __init__(self):
        self.id: int = 0
        self.name: str = ""
        self.email: str = ""

# ViewModel
class UserViewModel(QObject):
    nameChanged = pyqtSignal(str)
    errorChanged = pyqtSignal(str)
    
    def __init__(self):
        super().__init__()
        self._user = User()
        self._name = ""
        self._error = ""
    
    @pyqtProperty(str, notify=nameChanged)
    def name(self) -> str:
        return self._name
    
    @name.setter
    def name(self, value: str) -> None:
        if self._name != value:
            self._name = value
            self.nameChanged.emit(value)
            self._validate_name()
    
    @pyqtProperty(str, notify=errorChanged)
    def error(self) -> str:
        return self._error
    
    @error.setter
    def error(self, value: str) -> None:
        if self._error != value:
            self._error = value
            self.errorChanged.emit(value)
    
    def _validate_name(self) -> None:
        if not self.name or len(self.name) < 2:
            self.error = "Invalid name"
            return
        
        self.error = ""
        self._save_user()
    
    def _save_user(self) -> None:
        self._user.name = self.name
        # Save user async

# View (QML)
"""
Item {
    TextField {
        id: nameInput
        text: viewModel.name
        onTextChanged: viewModel.name = text
    }
    
    Text {
        text: viewModel.error
        visible: viewModel.error !== ""
        color: "red"
    }
}
"""
```

#### Swift (iOS-style)  
```swift  
// Model
struct User {
    var id: Int
    var name: String
    var email: String
}

// ViewModel
class UserViewModel {
    @Published private(set) var name: String = ""
    @Published private(set) var error: String = ""
    
    private var user: User
    private let userService: UserService
    
    func updateName(_ newName: String) {
        name = newName
        validateName()
    }
    
    private func validateName() {
        if name.isEmpty || name.count < 2 {
            error = "Invalid name"
            return
        }
        
        error = ""
        saveUser()
    }
    
    private func saveUser() {
        user.name = name
        userService.updateUser(user) { result in
            switch result {
            case .success(let updated):
                self.user = updated
            case .failure(let error):
                self.error = error.localizedDescription
            }
        }
    }
}

// View
struct UserView: View {
    @StateObject private var viewModel: UserViewModel
    
    var body: some View {
        VStack {
            TextField("Name", text: Binding(
                get: { viewModel.name },
                set: { viewModel.updateName($0) }
            ))
            
            if !viewModel.error.isEmpty {
                Text(viewModel.error)
                    .foregroundColor(.red)
            }
        }
    }
}
```

**Key Differences:**  
- TypeScript/Angular uses Observables for reactivity
- C#/WPF uses INotifyPropertyChanged
- Python/PyQt uses signals and properties
- Swift uses Combine framework with @Published

**WHEN SHOULD YOU USE IT?**  
- When building data-driven UIs
- When you need two-way data binding
- When UI logic needs testing
- When using reactive programming

**WHERE YOU'VE ALREADY SEEN IT**  
- Modern web frameworks (Angular, Vue)
- Desktop apps (WPF, UWP)
- Mobile apps (SwiftUI, Jetpack Compose)
- Cross-platform UI frameworks
