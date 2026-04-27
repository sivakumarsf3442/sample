## Context

The EJ2 Schedule component follows a modular architecture where the central `Schedule` class orchestrates views (Day, Week, Month, Timeline), renderers (HeaderRenderer, EventBase), and actions (Drag, Resize, CRUD). Current rendering pipeline: ViewBase renders cells → HeaderRenderer renders headers → EventBase renders appointments.

Custom rows must integrate into this pipeline without disrupting existing event rendering or performance. Key constraints:
- TypeScript/AMD module format (es5 target)
- Must follow EJ2 lifecycle patterns (@Property, @Event decorators)
- Virtual scrolling already used for Day/Week views - custom rows must support virtualization
- Resource grouping already complex (horizontal/vertical) - custom rows must not interfere
- Existing drag-drop, timezone, and export systems must remain functional

Current relevant code patterns:
- `src/schedule/base/schedule.ts`: ~4300 lines - orchestrates all modules
- `src/schedule/renderer/view-base.ts`: Base class for all views (Day, Week, Month, etc.)
- `src/schedule/event-renderer/event-base.ts`: Renders appointment elements
- `src/schedule/actions/crud.ts`: Handles event add/edit/delete
- `src/schedule/actions/data.ts`: Event data transformation and binding

## Goals / Non-Goals

**Goals:**
- Provide a flexible, template-based API for developers to extend scheduler with custom rows
- Support rendering custom rows in all view types (Day, Week, Month, Agenda, Timeline, Year)
- Enable automatic data binding so custom rows update when events change
- Implement positioning controls (above/below events, within resources, view-specific)
- Achieve < 50KB bundle size increase and maintain 60 FPS with large datasets
- Maintain full backward compatibility - feature is opt-in with no breaking changes
- Support styling customization and theme integration (light/dark themes)
- Enable event handlers for custom row interactions (click, hover, expand/collapse)

**Non-Goals:**
- Custom column support or grid-like custom row functionality (GridRowSettings approach)
- Real-time collaboration features for multi-user custom row editing
- Recursive/nested custom rows within custom rows
- Custom row animations beyond show/hide transitions
- Support for custom rows outside of standard view rendering (e.g., header/footer areas)
- Performance profiling and optimization beyond standard virtual scrolling mechanisms

## Decisions

### Decision 1: Architecture - Modular Custom Row System

**Choice**: Create new `src/schedule/custom-rows/` directory with separate modules (render, data-binding, templates, positioning, styling) that register with Schedule via dependency injection.

**Rationale**:
- Keeps custom row logic isolated from 4300-line Schedule class
- Allows feature to be tree-shaken if not used
- Follows existing EJ2 module pattern (actions modules register themselves)
- Clean separation of concerns enables easier testing and maintenance

**Alternatives Considered**:
1. Integrate directly into Schedule class - rejected because it increases core class complexity
2. Implement as mixin on View classes - rejected because each view would duplicate logic
3. Standalone component approach - rejected because custom rows need tight integration with Schedule lifecycle

### Decision 2: Rendering Strategy - Template-Based with Imperative Fallback

**Choice**: Support three template input formats:
- Function: `(eventData: EventModel[]) => string | HTMLElement`
- String selector: `'#custom-row-template'` references DOM element
- HTMLElement: Pass DOM element directly
Wrap rendering in try-catch to handle template errors gracefully.

**Rationale**:
- Function templates provide highest flexibility and match existing Schedule template API
- String selector supports declarative templates (common in enterprise apps)
- Direct element support reduces boilerplate
- Try-catch prevents template errors from breaking scheduler
- Matches patterns in EventBase and other EJ2 components

**Alternatives Considered**:
1. Only function templates - rejected because limits accessibility for non-JS developers
2. Async template rendering - rejected because blocks UI during scroll
3. Virtual DOM approach - rejected because adds dependency and complexity

### Decision 3: Data Binding - Event-Driven Synchronization

**Choice**: Custom rows receive event data via context object `{ events: EventModel[], resource?: ResourceModel, startTime: Date, endTime: Date }`. Subscribe to Schedule's `actionComplete` event to detect data changes. Re-render affected custom rows when event data mutates.

**Rationale**:
- Existing `actionComplete` event already fires on all CRUD operations
- Event data is already available in event object
- Matches existing data binding patterns in Schedule (via actionComplete)
- Avoids creating new data binding system - reuses what works

**Alternatives Considered**:
1. Two-way binding (like @syncfusion/ej2-inputs) - rejected because overcomplicates Schedule state management
2. Observable/RxJS pattern - rejected because adds dependency and complexity
3. Manual subscription per row - rejected because error-prone and requires developer management

### Decision 4: Position Configuration - Declarative with Validation

**Choice**: `CustomRowSettings` interface includes fields:
- `target: 'event' | 'date' | 'resource' | 'view'` - where to render
- `position: 'before' | 'after'` - relative to target
- `viewTypes?: ScheduleView[]` - if omitted, applies to all views
- `condition?: (eventData: EventModel[]) => boolean` - conditional rendering

DOM structure: Create custom row element after/before event or date cell, within same parent container.

**Rationale**:
- Declarative config is easier to understand and maintain than imperative positioning
- Target + position combination covers 80% of use cases (summary after event, aggregation before resource, etc.)
- View type filtering enables different layouts per view without multiple configurations
- Condition function allows complex rendering logic

**Alternatives Considered**:
1. Pixel-based positioning (like absolute/CSS-based) - rejected because brittle with dynamic heights
2. Parent/sibling selector API - rejected because fragile with view structure changes
3. Hook-based system with custom positioning functions - rejected because adds complexity

### Decision 5: Styling - CSS Class Application + Inline Styles

**Choice**: `CustomRowSettings` includes:
- `cssClass: string` - space-separated class names (applied directly to row element)
- `height?: number | 'auto'` - row height in pixels or auto-fit
- `rowTemplateClass?: string` - class for template container within row

Custom row element structure:
```html
<div class="e-custom-row {cssClass}" style="height: {height}px">
  <div class="e-custom-row-content {rowTemplateClass}">
    <!-- Template content here -->
  </div>
</div>
```

Base styles in `src/schedule/base/css-constant.ts` include `.e-custom-row` and `.e-custom-row-content` with padding/margin defaults.

**Rationale**:
- CSS classes allow theme-aware styling (bootstrap, material, etc. can define .e-custom-row styles)
- Inline styles for row height enable dynamic heights per row
- Separates template styling concerns from layout styling
- Follows existing Schedule CSS architecture (hardcoded class constants)

**Alternatives Considered**:
1. Shadow DOM encapsulation - rejected because increases complexity and breaks existing theme system
2. CSS-in-JS - rejected because Syncfusion uses plain CSS/SCSS
3. Only class-based styling - rejected because height needs to be dynamic per row

### Decision 6: Virtual Scrolling Integration - Recycling Strategy

**Choice**: Extend existing virtual scroll system to track custom row DOM elements. When a row becomes off-screen, mark for recycling. When new row needed, check recycle pool before creating new element. Maintain mapping of `eventId -> customRowElement` for reuse.

**Rationale**:
- Day/Week views already use virtual scrolling - avoid reinventing
- DOM recycling is proven efficient pattern in Schedule (used for events)
- Reduces memory churn and DOM node count
- Minimal performance impact on month view (which doesn't virtualize)

**Alternatives Considered**:
1. No virtual scrolling for custom rows - rejected because causes performance issues with 1000+ events
2. Always create/destroy - rejected because creates 200+ DOM operations per scroll frame
3. Separate virtual scroll system - rejected because duplicates logic and creates maintenance burden

### Decision 7: Event Handlers - Explicit Registration

**Choice**: `CustomRowSettings` includes `handlers?: { click?: Function, hover?: Function, expand?: Function }`. Each handler receives `{ eventData: EventModel[], customRowElement: HTMLElement, target: HTMLElement }`. Handlers must return void (no chaining).

**Rationale**:
- Explicit handlers are discoverable and easier to document
- Event object provides necessary context (data + DOM)
- Void return simplifies handler contract
- Matches existing event handler patterns in Schedule

**Alternatives Considered**:
1. HTML5 event attributes (onclick, onhover) - rejected because promotes inline handlers and harder to manage
2. EventEmitter pattern - rejected because adds complexity
3. Pub/Sub with custom events - rejected because overcomplicates API

### Decision 8: View Integration - Per-View Rendering Calls

**Choice**: Each view renderer (Day, Week, Month, etc.) calls `customRowModule.render(viewContext)` at end of its render pipeline. View context includes view type, date range, events, resources. Custom row module filters applicable rows by viewTypes and target.

**Rationale**:
- Leverages existing view rendering flow - minimal disruption
- Each view controls when custom rows render (after its own render completes)
- Easy to disable custom rows (just skip render call)
- Clear separation: view owns its custom rows

**Alternatives Considered**:
1. Centralized custom row rendering - rejected because doesn't know view-specific context
2. Hook into every event render - rejected because creates N+1 render calls
3. Custom row render before view render - rejected because needs view context (date range, etc.)

### Decision 9: API Methods - Standard CRUD Pattern

**Choice**: Schedule adds methods:
- `addCustomRow(config: CustomRowSettings): void` - Add row(s) to `this.customRows` array, trigger re-render
- `removeCustomRow(id: string): void` - Remove row by ID, cleanup DOM
- `updateCustomRow(id: string, config: Partial<CustomRowSettings>): void` - Update config, re-render
- `getCustomRowElement(id: string): HTMLElement | null` - Return rendered DOM element
- `refreshCustomRow(id: string): void` - Force refresh (re-render template)

Internally track custom rows in `Map<id, CustomRowSettings>` for O(1) lookups.

**Rationale**:
- Standard CRUD API is familiar to developers
- ID-based tracking enables efficient updates without re-rendering all rows
- Methods are simple and avoid method chains
- Matches existing Schedule API patterns

**Alternatives Considered**:
1. Array mutation API (Schedule.customRows.push) - rejected because harder to track changes
2. Builder pattern (fluent API) - rejected because overcomplicates simple operations
3. Binding API (like dataSource) - rejected because custom rows are local config, not data

### Decision 10: Lifecycle Integration - Init, Render, Update, Destroy

**Choice**: Custom row lifecycle hooks:
- **Init**: When Schedule initializes, load default `customRows` from config
- **Render**: When view renders, render applicable custom rows
- **Update**: When event data changes (actionComplete), update affected custom rows
- **Destroy**: When Schedule destroys, cleanup custom row DOM and handlers

Implement via Schedule's existing lifecycle hooks (initialize → render → destroy).

**Rationale**:
- Follows Schedule's component lifecycle pattern
- Hooks into existing event flow (actionComplete)
- Proper cleanup prevents memory leaks
- Clear phase separation enables testing

**Alternatives Considered**:
1. Observer pattern - rejected because adds complexity
2. Manual lifecycle management - rejected because error-prone
3. State machine approach - rejected because overkill for two states (render/destroy)

## Risks / Trade-offs

### Risk 1: Performance Degradation with Large Custom Row Content
**Mitigation**: 
- Document best practices for template performance (avoid heavy computations)
- Implement template rendering timeout (warn if >100ms)
- Provide performance demo with 1000+ events
- Implement lazy rendering for off-screen rows

### Risk 2: Custom Rows Break Existing Drag-Drop
**Mitigation**:
- Test drag-drop with custom rows in all view types
- Ensure custom rows don't interfere with event drag targets
- If conflict detected, hide custom rows during drag operation
- Document interaction behavior

### Risk 3: Styling Conflicts with Theme System
**Mitigation**:
- Use namespaced CSS classes (.e-custom-row-*)
- Test with all bundled themes (material, bootstrap, fabric, etc.)
- Provide fallback styles if custom classes not found
- Document CSS precedence rules

### Risk 4: Memory Leaks from Event Handlers
**Mitigation**:
- Track all registered event handlers in Map
- Cleanup handlers when custom rows are destroyed
- Test memory with profile sampler in unit tests
- Document handler cleanup requirements

### Risk 5: Complexity in Resource Grouped Views
**Mitigation**:
- Carefully test custom rows with resource grouping (all group modes)
- May need separate resource aggregation positioning logic
- Start with simple non-resource views, extend to resources

### Risk 6: Template Error Breaks Scheduler
**Mitigation**:
- Wrap template execution in try-catch
- Display error placeholder instead of breaking render
- Log errors to console for debugging
- Test with malformed templates

### Trade-off 1: No Async Templates
- **Trade**: Async template rendering (Promises/async-await) not supported
- **Reason**: Blocks UI during scroll/render, causes flickering
- **Mitigation**: Document workaround (pre-compute data, use sync template)

### Trade-off 2: Custom Rows Re-render on View Switch
- **Trade**: Custom rows fully re-render when switching between views
- **Reason**: Different view types may have different layouts
- **Mitigation**: Acceptable for most use cases; optimize if performance test fails

### Trade-off 3: No Custom Row Animation
- **Trade**: Custom rows don't animate in/out, appear/disappear instantly
- **Reason**: Adds complexity, minimal UX benefit
- **Mitigation**: CSS animations can be applied via custom classes if needed

## Migration Plan

### Phase 1: Core Implementation (Weeks 1-2)
- Implement custom row rendering engine in `src/schedule/custom-rows/render.ts`
- Create data binding module in `src/schedule/custom-rows/data-binding.ts`
- Add template rendering in `src/schedule/custom-rows/template.ts`
- Update Schedule class to include custom row properties/methods
- Add integration with Day/Week views (high priority views)

### Phase 2: Extended Integration (Weeks 3-4)
- Integrate with Month/Agenda/Year views
- Implement Timeline custom row support
- Add resource grouping integration
- Implement virtual scrolling support
- Add event handlers system

### Phase 3: Testing & Optimization (Weeks 5-6)
- Unit tests for all custom row modules (target 100% coverage)
- Integration tests with all view types
- Performance benchmarks and optimization
- Cross-browser testing
- Memory leak profiling

### Phase 4: Documentation & Framework Wrappers (Weeks 7-8)
- API documentation with JSDoc
- Demo pages for common use cases
- Update Angular/React/Vue wrappers
- Create migration guide for existing users
- Code review and feedback incorporation

### Rollback Strategy
- Feature is opt-in (disabled by default) - no existing code affected
- If critical issues found: remove Schedule.customRows property and render calls
- Fallback: Revert to previous git commit

## Open Questions

1. **Custom Row IDs**: Should IDs be auto-generated (UUID) or developer-provided? Current design: developer-provided with auto-generation fallback.

2. **Recurrence Handling**: For recurring events, should custom row appear once per recurrence or once per series? Current design: once per visible occurrence (developer can filter in template).

3. **Export/Print**: Should custom rows be included in Excel/PDF/ICS exports? Current design: included in Excel/PDF, skipped in ICS (no standard).

4. **RTL Support**: Do custom rows need RTL (right-to-left) language support? Current design: style via CSS, Schedule handles RTL system setup.

5. **Mobile Touch Handlers**: Should custom rows support touch events (swipe, long-press)? Current design: click/hover only, swipe can be added later if needed.

6. **Custom Row Dependencies**: Can custom rows depend on external libraries? Current design: no restrictions - developer's responsibility to handle dependencies.

7. **Performance Threshold**: What's acceptable render time for custom rows (50ms, 100ms, 200ms)? Current design: target 50-100ms, warn if exceeded.

8. **Nested Events**: For events with custom detail rows expanded, should child rows indent? Current design: no nesting - all rows at same level, controlled via template/CSS.
