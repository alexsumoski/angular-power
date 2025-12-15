---
name: "angular"
displayName: "Angular"
description: "Build and refactor Angular 19-20 apps using modern patterns - signals, inject(), standalone components, and modern control flow"
keywords: ["angular", "ng", "angular-cli", "signals", "standalone", "component", "directive", "pipe", "service", "rxjs", "reactive-forms", "zoneless", "onpush"]
author: "Alex Sumoski (@alexsumoski)"
---

# Angular Power

## Overview

Build and refactor Angular modern angular applications using modern patterns guided by the official Angular CLI MCP server. This power helps you adopt signals, inject(), standalone components, and modern control flow (@if, @for, @switch) while staying aligned with Angular's official recommendations.

**Mission:** Help you build and refactor Angular apps using modern patterns, guided by Angular MCP tools and official documentation.

**Key Capabilities:**
- Query official Angular documentation via MCP server
- Generate modern Angular code (signals, inject(), standalone)
- Refactor legacy patterns to Angular 19-20 standards
- Get architecture guidance aligned with Angular's recommendations

## Version Compatibility

**CRITICAL:** This power is optimized for Angular 18-20. Always check the project's Angular version first.

### Check Angular Version

Look for `@angular/core` version in `package.json`:

```json
{
  "dependencies": {
    "@angular/core": "^19.0.0"  // Check this version
  }
}
```

### Version-Specific Guidance

| Angular Version | Guidance |
|-----------------|----------|
| **18-20** | Use all patterns in this power (signals, standalone, @if/@for, inject()) |
| **17** | Standalone and @if/@for available; signals experimental |
| **14-16** | Standalone available (v14+); use NgModules for older; no @if/@for |
| **<14** | Use NgModules, constructor DI, *ngIf/*ngFor; signals not available |

**If Angular version is outside 18-20 range:**
1. Inform the user about version limitations
2. Adapt recommendations to their Angular version
3. Suggest upgrade path if appropriate
4. Use legacy patterns when modern ones aren't available

## Using the Angular MCP Server

**IMPORTANT:** When the user asks about Angular best practices, migrations, or architecture:

1. **FIRST**: Check the Angular version in `package.json`
2. **SECOND**: Call the Angular MCP server tools (`search_documentation`, `get_best_practices`, etc.)
3. **THEN**: Reconcile the MCP results with the guidelines in this power
4. **IF CONFLICT**: Explain both approaches and favor official docs unless user specifies otherwise

The Angular MCP server provides:
- Official Angular documentation search
- API references for Angular packages
- Best practice recommendations
- Modernization suggestions

**Always defer to MCP tools for authoritative answers.**

## Project Conventions (Customize Me)

> **Maintainers:** Edit this section to match your team's style. Fork this power and adjust these conventions for your organization.

### State Management
- **Default**: Signals for local state + RxJS at edges (HTTP, events)
- **Alternative**: NgRx Signal Store, Akita, NGXS, or other state libraries

### Styling
- **Default**: Framework-agnostic (works with any CSS approach)
- **Alternative**: Angular Material, Tailwind, Bootstrap, custom design system

### Forms
- **Default**: Typed reactive forms for non-trivial forms
- **Alternative**: Template-driven forms for simple cases

### Routing
- **Default**: Standalone component routes with functional guards
- **Alternative**: Module-based routing (legacy apps)

---

## Rule Strength Guide

- **Required** – Core to Angular 19-20; changing may break examples
- **Strongly Recommended** – Safe to change, but power assumes this by default
- **Optional** – Flavor preferences; customize freely

---

## Core Practices (Angular-Idiomatic Defaults)

These follow Angular's official recommendations and modern idioms:

### 1. Signals for Local State (Strongly Recommended)

Use signals for synchronous component state where appropriate:

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubled() }}</p>
      <button (click)="increment()">+</button>
    </div>
  `
})
export class CounterComponent {
  // Signal for mutable state
  count = signal(0);
  
  // Computed signal for derived state
  doubled = computed(() => this.count() * 2);
  
  increment() {
    this.count.update(c => c + 1);
  }
}
```

**When to use signals vs observables:**
- **Signals**: Synchronous state, component-local state, derived values
- **Observables**: Async operations, HTTP requests, complex event streams
- **Both**: Use `toSignal()` to convert observables to signals at component boundaries

### 2. Standalone Components (Required)

Use standalone components for new code (Angular's default since v17):

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-header',
  standalone: true,
  imports: [CommonModule, RouterLink],
  template: `
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </nav>
  `
})
export class HeaderComponent {}
```

### 3. inject() for Dependency Injection (Strongly Recommended)

Use `inject()` function for cleaner dependency injection:

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';

@Component({
  selector: 'app-users',
  standalone: true,
  template: `<div>{{ users() }}</div>`
})
export class UsersComponent {
  private http = inject(HttpClient);
  private router = inject(Router);
  
  users = signal<User[]>([]);
  
  // Constructor injection still works, but inject() is preferred for new code
}
```

### 4. Modern Control Flow (Required)

Use @if, @for, @switch (Angular's built-in control flow since v17):

```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    <!-- Modern @if -->
    @if (users().length > 0) {
      <ul>
        <!-- Modern @for with track -->
        @for (user of users(); track user.id) {
          <li>{{ user.name }}</li>
        } @empty {
          <li>No users found</li>
        }
      </ul>
    } @else {
      <p>Loading...</p>
    }
    
    <!-- Modern @switch -->
    @switch (status()) {
      @case ('loading') { <p>Loading...</p> }
      @case ('error') { <p>Error occurred</p> }
      @case ('success') { <p>Success!</p> }
    }
  `
})
export class UserListComponent {
  users = signal<User[]>([]);
  status = signal<'loading' | 'error' | 'success'>('loading');
}
```

### 5. Typed Reactive Forms (Strongly Recommended)

Use typed forms for type safety:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

interface UserForm {
  name: string;
  email: string;
  age: number;
}

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="name" />
      <input formControlName="email" type="email" />
      <input formControlName="age" type="number" />
      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  private fb = inject(FormBuilder);
  
  // Typed form group
  form = this.fb.group<UserForm>({
    name: this.fb.control('', { nonNullable: true, validators: [Validators.required] }),
    email: this.fb.control('', { nonNullable: true, validators: [Validators.required, Validators.email] }),
    age: this.fb.control(0, { nonNullable: true, validators: [Validators.min(0)] })
  });
  
  onSubmit() {
    if (this.form.valid) {
      const value: UserForm = this.form.getRawValue();
      console.log(value);
    }
  }
}
```

---

## Optional Opinions (Can Be Disabled)

These are opinionated preferences, not Angular requirements. Adjust or ignore based on your team's needs.

### Performance Optimizations (Optional)

**OnPush Change Detection:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  // Signals work great with OnPush
})
```

**Zoneless (Experimental):**
```typescript
// main.ts - Angular 19+ experimental
provideExperimentalZonelessChangeDetection()
```

### Routing Patterns (Optional)

**Functional Guards:**
```typescript
export const authGuard = () => inject(AuthService).isAuthenticated();
```

**Lazy Loading:**
```typescript
{
  path: 'feature',
  loadComponent: () => import('./feature.component').then(m => m.FeatureComponent)
}
```

### Service State Pattern (Optional)

**Expose readonly signals from services:**
```typescript
@Injectable({ providedIn: 'root' })
export class DataService {
  private data = signal<Data[]>([]);
  readonly dataSignal = this.data.asReadonly();
  
  addData(item: Data) {
    this.data.update(items => [...items, item]);
  }
}
```

---

## Quick Reference: Legacy → Modern Patterns

### NgModule → Standalone

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

### Constructor Injection → inject()

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

### *ngIf → @if

**Before:**
```html
<div *ngIf="isVisible">Content</div>
<div *ngIf="user; else loading">{{ user.name }}</div>
<ng-template #loading>Loading...</ng-template>
```

**After:**
```html
@if (isVisible) {
  <div>Content</div>
}

@if (user) {
  <div>{{ user.name }}</div>
} @else {
  <div>Loading...</div>
}
```

### *ngFor → @for

**Before:**
```html
<div *ngFor="let item of items; trackBy: trackById">
  {{ item.name }}
</div>
```

**After:**
```html
@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
}
```

### BehaviorSubject → Signal

**Before:**
```typescript
private countSubject = new BehaviorSubject(0);
count$ = this.countSubject.asObservable();

increment() {
  this.countSubject.next(this.countSubject.value + 1);
}
```

**After:**
```typescript
count = signal(0);

increment() {
  this.count.update(c => c + 1);
}
```

---

## Available Steering Files

- **angular-general.md** - General Angular development helper (auto-loaded)
- **angular-refactor-legacy.md** - Refactoring NgModules, constructor DI, and observables to modern patterns (manual)

## Using This Power

This power activates automatically when you mention Angular keywords. You can:

- **Ask for best practices**: "What's the modern way to handle state in Angular?"
- **Request refactoring**: "Convert this component to standalone with signals"
- **Get architecture guidance**: "How should I structure this feature?"
- **Query official docs**: The MCP server pulls latest Angular documentation

**Remember:** The MCP server provides authoritative answers. This power complements it with practical patterns.

## Resources

- [Angular Documentation](https://angular.dev)
- [Angular CLI MCP Server](https://angular.dev/ai/mcp)
- [Signals Guide](https://angular.dev/guide/signals)
- [Standalone Components](https://angular.dev/guide/components/importing)
