---
inclusion: manual
---

# Refactoring Legacy Angular to Modern Patterns

## Purpose

Systematically modernize legacy Angular code (pre-v17) to Angular 19-20 patterns.

## When to Use This Guide

- Converting NgModules to standalone
- Updating constructor DI to inject()
- Replacing *ngIf/*ngFor with @if/@for
- Converting BehaviorSubject to signals
- Modernizing existing codebases

## Before You Start

**IMPORTANT:** Call Angular MCP tools for migration guidance:
- Check for official migration schematics
- Get Angular team's recommended approach
- Verify compatibility with your Angular version

## Refactoring Priority

1. **Standalone Components** (removes NgModules)
2. **inject() Function** (cleaner DI)
3. **Modern Control Flow** (@if, @for, @switch)
4. **Signals** (replaces BehaviorSubject for state)
5. **OnPush** (optional performance boost)

## Refactoring Steps

### 1. NgModule → Standalone

**Before:**
```typescript
@NgModule({
  declarations: [MyComponent],
  imports: [CommonModule, FormsModule],
  exports: [MyComponent]
})
export class MyModule {}
```

**After:**
```typescript
@Component({
  selector: 'app-my',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `...`
})
export class MyComponent {}
```

**Bootstrap change:**
```typescript
// OLD
platformBrowserDynamic().bootstrapModule(AppModule);

// NEW
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes), provideHttpClient()]
});
```

### 2. Constructor DI → inject()

**Before:**
```typescript
constructor(
  private http: HttpClient,
  private router: Router
) {}
```

**After:**
```typescript
private http = inject(HttpClient);
private router = inject(Router);
```

**Benefits:** Cleaner, works outside constructor, better tree-shaking

### 3. Control Flow Syntax

***ngIf → @if:**
```html
<!-- Before -->
<div *ngIf="user; else loading">{{ user.name }}</div>
<ng-template #loading>Loading...</ng-template>

<!-- After -->
@if (user) {
  <div>{{ user.name }}</div>
} @else {
  <div>Loading...</div>
}
```

***ngFor → @for:**
```html
<!-- Before -->
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>

<!-- After -->
@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
}
```

***ngSwitch → @switch:**
```html
<!-- Before -->
<div [ngSwitch]="status">
  <p *ngSwitchCase="'loading'">Loading</p>
  <p *ngSwitchDefault>Unknown</p>
</div>

<!-- After -->
@switch (status) {
  @case ('loading') { <p>Loading</p> }
  @default { <p>Unknown</p> }
}
```

### 4. BehaviorSubject → Signal

**Before:**
```typescript
private countSubject = new BehaviorSubject(0);
count$ = this.countSubject.asObservable();

increment() {
  this.countSubject.next(this.countSubject.value + 1);
}

// Template: {{ count$ | async }}
```

**After:**
```typescript
count = signal(0);

increment() {
  this.count.update(c => c + 1);
}

// Template: {{ count() }}
```

**Computed values:**
```typescript
// Before: doubled$ = this.count$.pipe(map(c => c * 2));
// After:
doubled = computed(() => this.count() * 2);
```

### 5. Optional: Add OnPush

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  // Signals work great with OnPush
})
```

### 6. Optional: Type Forms

```typescript
interface UserForm {
  name: string;
  email: string;
}

form = this.fb.group<UserForm>({
  name: this.fb.control('', { nonNullable: true }),
  email: this.fb.control('', { nonNullable: true })
});
```

### 7. Optional: Functional Guards

```typescript
// Before: class-based guard
// After:
export const authGuard = () => inject(AuthService).isAuthenticated();
```

---

## Migration Strategy

**Gradual approach (recommended):**

1. Start with leaf components (no children)
2. Move up the component tree
3. Migrate services alongside components
4. Update routing last
5. Remove modules after all components are standalone

**Per-component checklist:**
- [ ] Convert to standalone
- [ ] Replace constructor DI with inject()
- [ ] Update template (@if, @for)
- [ ] Convert BehaviorSubject to signals (if applicable)
- [ ] Update tests

---

## Common Pitfalls

**Forgetting CommonModule:**
```typescript
@Component({
  standalone: true,
  imports: [CommonModule], // Required for pipes, directives
})
```

**Mutating signals:**
```typescript
// Wrong: this.items().push(item);
// Right:
this.items.update(items => [...items, item]);
```

**inject() outside injection context:**
```typescript
// Wrong: setTimeout(() => inject(Service), 1000);
// Right: private service = inject(Service);
```

---

## Testing Updates

```typescript
// Before
TestBed.configureTestingModule({
  declarations: [MyComponent],
  imports: [CommonModule]
});

// After
TestBed.configureTestingModule({
  imports: [MyComponent] // Standalone component
});
```

---

## Resources

- [Angular Migration Guide](https://angular.dev/reference/migrations)
- [Standalone Guide](https://angular.dev/guide/components/importing)
- [Signals Docs](https://angular.dev/guide/signals)
