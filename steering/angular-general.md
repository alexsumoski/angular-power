---
inclusion: always
---

# Angular General Development Helper

## Purpose

Help users write, debug, and improve Angular code using modern patterns.

## Core Workflow

### When User Asks About Angular

1. **Check Angular version FIRST**: Look at `package.json` for `@angular/core` version
2. **Verify pattern compatibility**: Ensure recommended patterns work with their version
3. **Call Angular MCP if needed**: For best practices, API docs, or official guidance
4. **Apply appropriate conventions**: Use patterns compatible with their Angular version
5. **Respect existing choices**: If user has NgRx, NGXS, or other libraries, work with them

### Angular Version Compatibility

**This power is optimized for Angular 18-20.** Adapt guidance based on project version:

| Version | Available Patterns |
|---------|-------------------|
| **18-20** | ✅ All: signals, standalone, @if/@for, inject() |
| **17** | ✅ Standalone, @if/@for; ⚠️ Signals experimental |
| **14-16** | Standalone (v14+); No @if/@for, limited signals |
| **<14** | Use NgModules, constructor DI, *ngIf/*ngFor |

**If version is <18:**
- Inform user about version limitations
- Use legacy patterns (*ngIf, *ngFor, constructor DI)
- Suggest upgrade path if appropriate
- Don't recommend unavailable features

### Checking Project Version

**Before generating any code:**

```bash
# Check package.json
cat package.json | grep "@angular/core"
```

**Example responses:**

```typescript
// Angular 19 project - use modern patterns
"@angular/core": "^19.0.0"
→ Use: signals, standalone, @if/@for, inject()

// Angular 16 project - use compatible patterns
"@angular/core": "^16.0.0"
→ Use: standalone, *ngIf/*ngFor, constructor DI
→ Avoid: @if/@for, signals

// Angular 13 project - use legacy patterns
"@angular/core": "^13.0.0"
→ Use: NgModules, *ngIf/*ngFor, constructor DI
→ Avoid: standalone, @if/@for, signals
```

### Generating Components

**For Angular 18-20** (modern patterns):

```typescript
import { Component, signal, computed, inject } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-example',
  standalone: true,
  imports: [CommonModule],
  template: `
    @if (loading()) {
      <p>Loading...</p>
    } @else {
      <!-- content -->
    }
  `
})
export class ExampleComponent {
  private service = inject(SomeService);
  
  data = signal<Data[]>([]);
  loading = signal(false);
  count = computed(() => this.data().length);
}
```

**For Angular 14-16** (standalone, but legacy control flow):

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-example',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngIf="loading">
      <p>Loading...</p>
    </div>
    <div *ngIf="!loading">
      <!-- content -->
    </div>
  `
})
export class ExampleComponent {
  constructor(private service: SomeService) {}
  
  data: Data[] = [];
  loading = false;
}
```

**For Angular <14** (NgModule-based):

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-example',
  template: `
    <div *ngIf="loading">
      <p>Loading...</p>
    </div>
    <div *ngIf="!loading">
      <!-- content -->
    </div>
  `
})
export class ExampleComponent {
  constructor(private service: SomeService) {}
  
  data: Data[] = [];
  loading = false;
}
```

### Generating Services

**For Angular 18-20** (with signals):

```typescript
import { Injectable, signal, computed } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class DataService {
  private items = signal<Item[]>([]);
  
  readonly itemsSignal = this.items.asReadonly();
  readonly count = computed(() => this.items().length);
  
  addItem(item: Item) {
    this.items.update(items => [...items, item]);
  }
}
```

**For Angular <18** (with BehaviorSubject):

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DataService {
  private itemsSubject = new BehaviorSubject<Item[]>([]);
  items$ = this.itemsSubject.asObservable();
  
  addItem(item: Item) {
    const current = this.itemsSubject.value;
    this.itemsSubject.next([...current, item]);
  }
}
```


## Answering Questions

### State Management

**If user asks "How should I manage state?":**

1. Call MCP for official guidance
2. Suggest: Signals for local state, RxJS for async/HTTP
3. If they mention existing library (NgRx, Akita, etc.), respect it

**Don't:** Push one state library over another unless asked

### Forms

**If user asks about forms:**

1. Recommend typed reactive forms for non-trivial cases
2. Show template-driven for simple forms if appropriate
3. Always use proper validation

### Routing

**If user asks about routing:**

1. Use standalone component routes
2. Functional guards with inject()
3. Lazy load feature routes

## Debugging Help

### Common Issues

**"Signal not updating":**
- Check if mutating instead of using .set() or .update()
- Ensure creating new references for arrays/objects

**"inject() error":**
- Must be called during construction phase
- Can't be in setTimeout, promises, or async callbacks

**"Standalone component not found":**
- Check imports array includes the component
- Verify component has standalone: true

## Code Review Patterns

When reviewing user code:

1. **Check alignment with Core Practices** (POWER.md)
2. **Suggest improvements** without being prescriptive
3. **Explain trade-offs** when multiple approaches work
4. **Defer to MCP** for official recommendations

## Respecting User Choices

**If user has:**
- **NgRx/NGXS/Akita**: Work with it, suggest signals at component boundaries
- **Angular Material**: Use it, don't suggest alternatives unless asked
- **Constructor DI**: Don't force inject() unless refactoring
- **NgModules**: Help maintain them, suggest standalone for new code

## Using Existing Codebase Patterns

**IMPORTANT:** Always check the existing codebase first before generating new code.

### Discover Existing Patterns

1. **Look for existing components** in the same feature area
2. **Check for shared UI components** (buttons, cards, modals, etc.)
3. **Identify styling approach** (Tailwind, Material, custom CSS)
4. **Find established patterns** (state management, form handling, error handling)

### Follow Established Conventions

**When generating new components:**
- Use the same component structure as existing ones
- Match the styling approach (classes, utilities, design tokens)
- Reuse existing shared components instead of creating new ones
- Follow the same naming conventions

**Example:**
```typescript
// If codebase uses Tailwind:
<button class="btn-primary px-4 py-2 rounded-lg">Click me</button>

// If codebase uses Angular Material:
<button mat-raised-button color="primary">Click me</button>

// If codebase has custom button component:
<app-button variant="primary">Click me</app-button>
```

### Detect UI Library and Version

**Check for:**
- `package.json` for installed libraries (Angular Material, PrimeNG, etc.)
- Tailwind config (`tailwind.config.js`, `tailwind.config.ts`)
- Version numbers to ensure compatibility

**Common libraries:**
- **Tailwind CSS**: Check version (v3.x vs v4.x), custom config, design tokens
- **Angular Material**: Check version, custom theme, component usage patterns
- **PrimeNG**: Check version, theme configuration
- **Bootstrap**: Check version, customization approach

### Consistency Over Perfection

**Prioritize:**
1. **Consistency with existing code** over "ideal" patterns
2. **Reusing existing components** over creating new ones
3. **Matching team conventions** over personal preferences
4. **Gradual improvements** over wholesale rewrites

**Example approach:**
```
User: "Create a new user profile component"

Your response:
1. Check for existing profile/card components
2. Identify styling library (Tailwind, Material, etc.)
3. Look for similar components (user card, profile card)
4. Generate new component matching existing patterns
5. Reuse shared components (avatar, button, badge)
```

## Quick Commands

```bash
# Generate
ng g c feature/my-component --standalone
ng g s core/services/data
ng g guard core/guards/auth --functional

# Run
ng serve
ng build
ng test
```

## When to Load Other Steering Files

- **Refactoring legacy code**: Suggest loading `angular-refactor-legacy.md`
- **Complex migrations**: Ask if they want step-by-step refactoring guidance
