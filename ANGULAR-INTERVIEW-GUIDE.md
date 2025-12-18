<p align="center">
  <img src="https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white" alt="Angular"/>
</p>

<h1 align="center">🅰️ Angular Interview Guide</h1>

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

---

## 📚 Section Overview

| # | Topic | Difficulty |
|---|-------|------------|
| 1 | [Angular Fundamentals](#1-angular-fundamentals) | ⭐⭐ |
| 2 | [Components](#2-components) | ⭐⭐ |
| 3 | [Directives](#3-directives) | ⭐⭐ |
| 4 | [Pipes](#4-pipes) | ⭐⭐ |
| 5 | [Services & DI](#5-services--dependency-injection) | ⭐⭐⭐ |
| 6 | [Routing](#6-routing) | ⭐⭐⭐ |
| 7 | [Forms](#7-forms) | ⭐⭐⭐ |
| 8 | [HTTP & API](#8-http--api-communication) | ⭐⭐⭐ |
| 9 | [RxJS & Observables](#9-rxjs--observables) | ⭐⭐⭐⭐ |
| 10 | [State Management](#10-state-management) | ⭐⭐⭐⭐ |
| 11 | [Performance](#11-performance-optimization) | ⭐⭐⭐ |
| 12 | [Testing](#12-angular-testing) | ⭐⭐⭐ |
| 13 | [Signals](#13-signals-angular-16) | ⭐⭐⭐ |

---

## 1. Angular Fundamentals

### 🔷 1.1 Angular Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     ANGULAR ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                         MODULES                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │                     COMPONENTS                          │  │  │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │  │  │
│  │  │  │  Template   │  │    Class    │  │   Styles    │      │  │  │
│  │  │  │   (HTML)    │◄─│    (TS)     │─►│   (CSS)     │      │  │  │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘      │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  │                              │                                │  │
│  │                              ▼                                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐    │  │
│  │  │  Services   │  │  Directives │  │       Pipes         │    │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 🔷 1.2 Data Binding

```typescript
@Component({
  selector: 'app-user',
  template: `
    <!-- 1. Interpolation (Component → View) -->
    <h1>{{ title }}</h1>
    <p>{{ user.name | uppercase }}</p>
    
    <!-- 2. Property Binding (Component → View) -->
    <img [src]="imageUrl">
    <button [disabled]="isDisabled">Click</button>
    
    <!-- 3. Event Binding (View → Component) -->
    <button (click)="onClick()">Click</button>
    <input (keyup.enter)="onEnter($event)">
    
    <!-- 4. Two-way Binding (Component ↔ View) -->
    <input [(ngModel)]="username">
  `
})
export class UserComponent {
  title = 'User Profile';
  user = { name: 'John' };
  imageUrl = 'avatar.png';
  isDisabled = false;
  username = '';
  
  onClick() { }
  onEnter(event: any) { }
}
```

### 🔷 1.3 Lifecycle Hooks

```typescript
@Component({...})
export class MyComponent implements OnInit, OnChanges, OnDestroy {
  
  @Input() data: string;
  
  constructor() {
    // 1. First - before any lifecycle hook
  }
  
  ngOnChanges(changes: SimpleChanges) {
    // 2. When @Input properties change
  }
  
  ngOnInit() {
    // 3. After first ngOnChanges, once
    // Good for: API calls, initialization
  }
  
  ngDoCheck() {
    // 4. Every change detection run
  }
  
  ngAfterContentInit() {
    // 5. After content (ng-content) projected
  }
  
  ngAfterContentChecked() {
    // 6. After projected content checked
  }
  
  ngAfterViewInit() {
    // 7. After view and child views initialized
  }
  
  ngAfterViewChecked() {
    // 8. After view and child views checked
  }
  
  ngOnDestroy() {
    // 9. Before component destroyed
    // Good for: Cleanup, unsubscribe
  }
}
```

---

## 2. Components

### 🔷 2.1 Component Communication

```typescript
// Parent Component
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      [message]="parentMessage"
      (notify)="onNotify($event)">
    </app-child>
  `
})
export class ParentComponent {
  parentMessage = 'Hello from parent';
  
  onNotify(event: string) {
    console.log('Child says:', event);
  }
}

// Child Component
@Component({
  selector: 'app-child',
  template: `
    <p>{{ message }}</p>
    <button (click)="sendNotification()">Notify Parent</button>
  `
})
export class ChildComponent {
  @Input() message: string;
  @Output() notify = new EventEmitter<string>();
  
  sendNotification() {
    this.notify.emit('Hello from child!');
  }
}
```

### 🔷 2.2 ViewChild & ContentChild

```typescript
@Component({
  selector: 'app-parent',
  template: `
    <input #inputRef>
    <app-child></app-child>
    
    <app-container>
      <p #projectedContent>Projected content</p>
    </app-container>
  `
})
export class ParentComponent implements AfterViewInit {
  @ViewChild('inputRef') inputElement: ElementRef;
  @ViewChild(ChildComponent) childComponent: ChildComponent;
  
  ngAfterViewInit() {
    this.inputElement.nativeElement.focus();
    this.childComponent.someMethod();
  }
}

@Component({
  selector: 'app-container',
  template: `<ng-content></ng-content>`
})
export class ContainerComponent implements AfterContentInit {
  @ContentChild('projectedContent') projectedElement: ElementRef;
  
  ngAfterContentInit() {
    console.log(this.projectedElement.nativeElement.textContent);
  }
}
```

### 🔷 2.3 Change Detection

```typescript
@Component({
  selector: 'app-user',
  template: `<p>{{ user.name }}</p>`,
  changeDetection: ChangeDetectionStrategy.OnPush  // Only check on @Input changes
})
export class UserComponent {
  @Input() user: User;  // Must pass NEW object reference to trigger change
}

// Parent must create new object reference
updateUser() {
  this.user = { ...this.user, name: 'New Name' };  // ✅ Creates new reference
  // this.user.name = 'New Name';  // ❌ Won't trigger OnPush change detection
}
```

---

## 3. Directives

### 🔷 3.1 Built-in Directives

```html
<!-- Structural Directives -->
<div *ngIf="isVisible">Visible content</div>
<div *ngIf="user; else noUser">{{ user.name }}</div>
<ng-template #noUser>No user found</ng-template>

<ul>
  <li *ngFor="let item of items; let i = index; trackBy: trackById">
    {{ i }}: {{ item.name }}
  </li>
</ul>

<div [ngSwitch]="status">
  <p *ngSwitchCase="'active'">Active</p>
  <p *ngSwitchCase="'inactive'">Inactive</p>
  <p *ngSwitchDefault>Unknown</p>
</div>

<!-- Attribute Directives -->
<div [ngClass]="{'active': isActive, 'disabled': isDisabled}">Content</div>
<div [ngStyle]="{'color': textColor, 'font-size': fontSize + 'px'}">Styled</div>
```

### 🔷 3.2 Custom Directive

```typescript
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input('appHighlight') highlightColor = 'yellow';
  
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.highlightColor);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
  }
  
  private highlight(color: string | null) {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}

// Usage
<p [appHighlight]="'lightblue'">Hover over me!</p>
```

---

## 4. Pipes

### 🔷 4.1 Built-in Pipes

```html
<!-- Date Pipe -->
<p>{{ today | date:'fullDate' }}</p>
<p>{{ today | date:'dd/MM/yyyy' }}</p>

<!-- Currency Pipe -->
<p>{{ price | currency:'USD':'symbol':'1.2-2' }}</p>

<!-- Async Pipe (auto-subscribes and unsubscribes) -->
<p>{{ data$ | async }}</p>
<div *ngIf="user$ | async as user">{{ user.name }}</div>

<!-- Other Pipes -->
<p>{{ text | uppercase }}</p>
<p>{{ text | lowercase }}</p>
<p>{{ text | titlecase }}</p>
<p>{{ object | json }}</p>
<p>{{ items | slice:0:5 }}</p>
<p>{{ number | percent }}</p>
```

### 🔷 4.2 Custom Pipe

```typescript
// Pure Pipe (default) - only called when input reference changes
@Pipe({
  name: 'filter',
  pure: true
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchText: string, field: string): any[] {
    if (!items || !searchText) return items;
    
    return items.filter(item => 
      item[field].toLowerCase().includes(searchText.toLowerCase())
    );
  }
}

// Usage
<div *ngFor="let user of users | filter:searchText:'name'">
  {{ user.name }}
</div>

// Impure Pipe - called on every change detection (use with caution!)
@Pipe({
  name: 'impureFilter',
  pure: false
})
export class ImpureFilterPipe { }
```

---

## 5. Services & Dependency Injection

### 🔷 5.1 Creating Services

```typescript
@Injectable({
  providedIn: 'root'  // Singleton, available everywhere
})
export class UserService {
  private apiUrl = '/api/users';
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
}
```

### 🔷 5.2 Injection Scopes

| Scope | How | When |
|-------|-----|------|
| **Root** | `providedIn: 'root'` | Singleton for entire app |
| **Module** | `providers` in NgModule | Singleton per module |
| **Component** | `providers` in Component | New instance per component |

```typescript
// Module-level
@NgModule({
  providers: [DataService]  // Singleton for this module
})
export class FeatureModule {}

// Component-level
@Component({
  providers: [DataService]  // New instance for each component
})
export class MyComponent {}
```

---

## 6. Routing

### 🔷 6.1 Route Configuration

```typescript
const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  
  // Lazy Loading
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule)
  },
  
  // Child Routes
  {
    path: 'products',
    component: ProductsComponent,
    children: [
      { path: '', component: ProductListComponent },
      { path: ':id', component: ProductDetailComponent }
    ]
  },
  
  // Route Guards
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard],
    canDeactivate: [UnsavedChangesGuard]
  },
  
  // Wildcard (404)
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### 🔷 6.2 Route Guards

```typescript
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean | Observable<boolean> {
    
    if (this.authService.isLoggedIn()) {
      return true;
    }
    
    this.router.navigate(['/login'], { 
      queryParams: { returnUrl: state.url } 
    });
    return false;
  }
}

// Angular 14+ functional guard
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  return authService.isLoggedIn() 
    ? true 
    : router.createUrlTree(['/login']);
};
```

### 🔷 6.3 Reading Route Parameters

```typescript
@Component({...})
export class UserDetailComponent implements OnInit {
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    // Snapshot (one-time read)
    const id = this.route.snapshot.paramMap.get('id');
    const query = this.route.snapshot.queryParamMap.get('search');
    
    // Observable (for same-component navigation)
    this.route.paramMap.subscribe(params => {
      const id = params.get('id');
    });
    
    // Route data
    const data = this.route.snapshot.data['pageTitle'];
  }
}
```

---

## 7. Forms

### 🔷 7.1 Template-Driven Forms

```typescript
@Component({
  template: `
    <form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
      <input 
        name="name" 
        [(ngModel)]="user.name" 
        required 
        minlength="3"
        #name="ngModel">
      
      <div *ngIf="name.invalid && name.touched">
        <span *ngIf="name.errors?.['required']">Name is required</span>
        <span *ngIf="name.errors?.['minlength']">Min 3 characters</span>
      </div>
      
      <input 
        name="email" 
        [(ngModel)]="user.email" 
        required 
        email>
      
      <button [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  user = { name: '', email: '' };
  
  onSubmit(form: NgForm) {
    if (form.valid) {
      console.log('Form data:', this.user);
    }
  }
}
```

### 🔷 7.2 Reactive Forms

```typescript
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name">
      <div *ngIf="userForm.get('name')?.invalid && userForm.get('name')?.touched">
        <span *ngIf="userForm.get('name')?.errors?.['required']">Required</span>
      </div>
      
      <input formControlName="email">
      
      <!-- Nested FormGroup -->
      <div formGroupName="address">
        <input formControlName="street">
        <input formControlName="city">
      </div>
      
      <!-- FormArray -->
      <div formArrayName="phones">
        <div *ngFor="let phone of phones.controls; let i = index">
          <input [formControlName]="i">
          <button (click)="removePhone(i)">Remove</button>
        </div>
        <button (click)="addPhone()">Add Phone</button>
      </div>
      
      <button [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent implements OnInit {
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      address: this.fb.group({
        street: [''],
        city: ['']
      }),
      phones: this.fb.array([])
    });
  }
  
  get phones() {
    return this.userForm.get('phones') as FormArray;
  }
  
  addPhone() {
    this.phones.push(this.fb.control(''));
  }
  
  removePhone(index: number) {
    this.phones.removeAt(index);
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

---

## 8. HTTP & API Communication

### 🔷 8.1 HttpClient

```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private baseUrl = '/api';
  
  constructor(private http: HttpClient) {}
  
  // GET
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.baseUrl}/users`);
  }
  
  // GET with params
  searchUsers(query: string): Observable<User[]> {
    const params = new HttpParams().set('q', query);
    return this.http.get<User[]>(`${this.baseUrl}/users`, { params });
  }
  
  // POST
  createUser(user: User): Observable<User> {
    return this.http.post<User>(`${this.baseUrl}/users`, user);
  }
  
  // PUT
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.baseUrl}/users/${id}`, user);
  }
  
  // DELETE
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/users/${id}`);
  }
}
```

### 🔷 8.2 HTTP Interceptors

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }
    
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          this.authService.logout();
        }
        return throwError(() => error);
      })
    );
  }
}

// Register in module
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true }
  ]
})
export class AppModule {}
```

---

## 9. RxJS & Observables

### 🔷 9.1 Observable Basics

```typescript
// Creating Observables
const observable$ = new Observable<number>(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();
});

// From existing data
const fromArray$ = from([1, 2, 3]);
const fromValue$ = of('hello');
const interval$ = interval(1000);  // Emits every second

// Subscribing
const subscription = observable$.subscribe({
  next: value => console.log(value),
  error: err => console.error(err),
  complete: () => console.log('Done')
});

// Unsubscribe
subscription.unsubscribe();
```

### 🔷 9.2 Common Operators

```typescript
// Transformation
this.users$ = this.http.get<User[]>('/api/users').pipe(
  map(users => users.filter(u => u.active)),
  tap(users => console.log('Active users:', users.length))
);

// Flattening Operators
this.userDetails$ = this.route.paramMap.pipe(
  map(params => params.get('id')),
  switchMap(id => this.userService.getUser(id))  // Cancels previous request
);

// Use switchMap for: search, navigation (cancel previous)
// Use mergeMap for: parallel requests (don't cancel)
// Use concatMap for: sequential requests (wait for previous)
// Use exhaustMap for: login button (ignore while in progress)

// Filtering
this.filtered$ = this.items$.pipe(
  filter(item => item.price > 100),
  take(5),                          // Take first 5
  takeUntil(this.destroy$),         // Unsubscribe when destroy$ emits
  debounceTime(300),                // Wait 300ms after last emission
  distinctUntilChanged()            // Only emit if different from previous
);

// Combination
const combined$ = combineLatest([user$, permissions$]).pipe(
  map(([user, permissions]) => ({ user, permissions }))
);

const parallel$ = forkJoin({
  users: this.http.get('/api/users'),
  products: this.http.get('/api/products')
});

// Error Handling
this.data$ = this.http.get('/api/data').pipe(
  retry(3),                         // Retry 3 times
  catchError(err => {
    console.error(err);
    return of([]);                  // Return fallback value
  })
);
```

### 🔷 9.3 Subjects

```typescript
// Subject - multicast, no initial value
private subject = new Subject<string>();

// BehaviorSubject - has initial/current value
private currentUser = new BehaviorSubject<User | null>(null);
currentUser$ = this.currentUser.asObservable();

setUser(user: User) {
  this.currentUser.next(user);
}

// ReplaySubject - replays last N values to new subscribers
private replaySubject = new ReplaySubject<string>(3);  // Keep last 3

// AsyncSubject - only emits last value when completed
private asyncSubject = new AsyncSubject<string>();
```

### 🔷 9.4 Memory Leak Prevention

```typescript
@Component({...})
export class MyComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // Method 1: takeUntil
    this.userService.getUsers().pipe(
      takeUntil(this.destroy$)
    ).subscribe(users => this.users = users);
    
    // Method 2: Async pipe (auto-unsubscribes)
    // In template: {{ users$ | async }}
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## 10. State Management

### 🔷 10.1 NgRx Store

```typescript
// 1. Define State
interface AppState {
  users: UserState;
}

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

// 2. Create Actions
export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User] Load Users Failure',
  props<{ error: string }>()
);

// 3. Create Reducer
export const userReducer = createReducer(
  initialState,
  on(loadUsers, state => ({ ...state, loading: true })),
  on(loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),
  on(loadUsersFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  }))
);

// 4. Create Effects
@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersFailure({ error: error.message })))
        )
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}

// 5. Create Selectors
export const selectUserState = createFeatureSelector<UserState>('users');
export const selectAllUsers = createSelector(
  selectUserState,
  state => state.users
);
export const selectLoading = createSelector(
  selectUserState,
  state => state.loading
);

// 6. Use in Component
@Component({
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <ul>
      <li *ngFor="let user of users$ | async">{{ user.name }}</li>
    </ul>
  `
})
export class UserListComponent implements OnInit {
  users$ = this.store.select(selectAllUsers);
  loading$ = this.store.select(selectLoading);
  
  constructor(private store: Store) {}
  
  ngOnInit() {
    this.store.dispatch(loadUsers());
  }
}
```

---

## 11. Performance Optimization

### 🔷 11.1 OnPush Change Detection

```typescript
@Component({
  selector: 'app-user-card',
  template: `<div>{{ user.name }}</div>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user!: User;  // Only re-renders when input reference changes
}
```

### 🔷 11.2 TrackBy Function

```html
<!-- Without trackBy: Entire list re-rendered on any change -->
<li *ngFor="let item of items">{{ item.name }}</li>

<!-- With trackBy: Only changed items re-rendered -->
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
```

```typescript
trackById(index: number, item: Item): number {
  return item.id;
}
```

### 🔷 11.3 Lazy Loading

```typescript
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule)
  }
];
```

### 🔷 11.4 Virtual Scrolling

```html
<cdk-virtual-scroll-viewport itemSize="50" class="viewport">
  <div *cdkVirtualFor="let item of items" class="item">
    {{ item.name }}
  </div>
</cdk-virtual-scroll-viewport>
```

---

## 12. Angular Testing

### 🔷 12.1 Component Testing

```typescript
describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userService: jasmine.SpyObj<UserService>;
  
  beforeEach(async () => {
    const spy = jasmine.createSpyObj('UserService', ['getUsers']);
    
    await TestBed.configureTestingModule({
      declarations: [UserComponent],
      providers: [
        { provide: UserService, useValue: spy }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should display users', () => {
    userService.getUsers.and.returnValue(of([
      { id: 1, name: 'John' }
    ]));
    
    fixture.detectChanges();
    
    const compiled = fixture.nativeElement;
    expect(compiled.querySelector('.user-name').textContent).toContain('John');
  });
});
```

### 🔷 12.2 Service Testing

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();
  });
  
  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    
    service.getUsers().subscribe(users => {
      expect(users.length).toBe(1);
      expect(users[0].name).toBe('John');
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
});
```

---

## 13. Signals (Angular 16+)

### 🔷 13.1 Signal Basics

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ doubleCount() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  // Create a signal
  count = signal(0);
  
  // Computed signal (derived state)
  doubleCount = computed(() => this.count() * 2);
  
  constructor() {
    // Effect runs when signals change
    effect(() => {
      console.log('Count changed to:', this.count());
    });
  }
  
  increment() {
    // Update signal
    this.count.update(value => value + 1);
    // Or set directly
    // this.count.set(this.count() + 1);
  }
}
```

### 🔷 13.2 Signal vs Observable

| Aspect | Signals | Observables |
|--------|---------|-------------|
| **Subscription** | Auto (in template) | Manual |
| **Async support** | Sync only | Sync & Async |
| **Cleanup** | Automatic | Manual (unsubscribe) |
| **Learning curve** | Lower | Higher |
| **Use case** | UI state | Async operations |

---

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

