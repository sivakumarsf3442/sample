# Custom Row Support - Implementation Tasks

## 1. Core Infrastructure Setup

- [ ] 1.1 Create `src/schedule/custom-rows/` directory structure with subdirectories: renderer, data-binding, templates, positioning, styling
- [ ] 1.2 Create `src/schedule/custom-rows/index.ts` to export public API
- [ ] 1.3 Define CustomRowSettings interface in `src/schedule/custom-rows/models/custom-row-model.ts` with properties: id, type, template, templateId, target, position, viewTypes, condition, cssClass, height, handlers
- [ ] 1.4 Define CustomRowContext interface in `src/schedule/custom-rows/models/custom-row-model.ts` with properties: events, resource, startTime, endTime
- [ ] 1.5 Create CSS constants in `src/schedule/base/css-constant.ts` for custom row classes: e-custom-row, e-custom-row-content, e-custom-row-error
- [ ] 1.6 Add custom row CSS base styles to `src/schedule/css/schedule-custom-rows.scss` with default padding, margin, borders

## 2. Custom Row Rendering Engine

- [ ] 2.1 Create `src/schedule/custom-rows/render/custom-row-renderer.ts` with CustomRowRenderer class
- [ ] 2.2 Implement `render(config: CustomRowSettings, context: CustomRowContext): HTMLElement` method to create custom row DOM
- [ ] 2.3 Implement template execution logic in `executeTemplate(template, context)` with try-catch error handling
- [ ] 2.4 Implement template selector support in `resolveTemplate(template, templateId)` for ID-based template retrieval
- [ ] 2.5 Implement `insertCustomRow(element: HTMLElement, target: Element, position: string)` to insert row into DOM
- [ ] 2.6 Implement conditional rendering check in `shouldRender(condition, context)` for condition evaluation
- [ ] 2.7 Implement CSS class application in `applyCssClasses(element, cssClass)` with validation
- [ ] 2.8 Implement error placeholder rendering in `renderErrorPlaceholder(error)` for template failures
- [ ] 2.9 Implement custom row element cleanup in `removeElement(element)` with proper resource cleanup

## 3. Virtual Scrolling & DOM Recycling

- [ ] 3.1 Create `src/schedule/custom-rows/virtualization/custom-row-pool.ts` for DOM element pooling
- [ ] 3.2 Implement element recycling logic in `getElement()` and `releaseElement(element)` methods
- [ ] 3.3 Implement element mapping `Map<eventId, customRowElement>` to track custom row elements
- [ ] 3.4 Create `src/schedule/custom-rows/virtualization/virtual-renderer.ts` for lazy rendering
- [ ] 3.5 Implement viewport-based visibility detection in `updateVisibleRows(viewportStart, viewportEnd)` 
- [ ] 3.6 Integrate virtual scrolling with Day/Week view renderers to recycle custom rows on scroll
- [ ] 3.7 Test DOM element count remains constant during scroll with 500+ events

## 4. Custom Row Data Binding

- [ ] 4.1 Create `src/schedule/custom-rows/data-binding/custom-row-data-binder.ts` with CustomRowDataBinder class
- [ ] 4.2 Implement `bindToSchedule(schedule)` to subscribe to Schedule's actionComplete event
- [ ] 4.3 Implement `onDataChange(args)` to detect event data changes and trigger row updates
- [ ] 4.4 Implement `getFilteredEvents(dateTime, resource)` to provide view-specific filtered event data
- [ ] 4.5 Implement `createContext(events, resource, startTime, endTime)` to build CustomRowContext objects
- [ ] 4.6 Implement recurrence expansion logic to provide each occurrence separately in events array
- [ ] 4.7 Implement timezone conversion in context times using Schedule's timezone settings
- [ ] 4.8 Implement diff detection in `hasChanges(oldEvents, newEvents)` to skip unnecessary re-renders
- [ ] 4.9 Test data binding with timezone changes and recurrence patterns

## 5. Custom Row API Implementation

- [ ] 5.1 Add `customRows: CustomRowSettings[]` property to ScheduleModel in `src/schedule/schedule-model.ts` with @Property decorator
- [ ] 5.2 Implement `addCustomRow(config: CustomRowSettings): void` method in Schedule class
- [ ] 5.3 Implement `removeCustomRow(id: string): void` method with DOM cleanup
- [ ] 5.4 Implement `updateCustomRow(id: string, config: Partial<CustomRowSettings>): void` method with partial update support
- [ ] 5.5 Implement `getCustomRowElement(id: string): HTMLElement | null` method
- [ ] 5.6 Implement `refreshCustomRow(id: string): void` method for manual re-render
- [ ] 5.7 Add input validation for CustomRowSettings in `validateCustomRowConfig()` method
- [ ] 5.8 Create internal `Map<id, CustomRowSettings>` for efficient custom row lookup
- [ ] 5.9 Implement auto-ID generation using UUID if id not provided

## 6. Event Handlers & Interactivity

- [ ] 6.1 Create `src/schedule/custom-rows/events/event-handler-manager.ts` for handler management
- [ ] 6.2 Implement click event handler registration in `registerClickHandler(element, handler)`
- [ ] 6.3 Implement hover event handler registration in `registerHoverHandler(element, handler)`
- [ ] 6.4 Implement expand/collapse handler registration in `registerExpandHandler(element, handler)`
- [ ] 6.5 Create event context object containing: eventData, customRowElement, target
- [ ] 6.6 Implement error handling for handler execution with try-catch
- [ ] 6.7 Implement handler cleanup in `removeHandlers(element)` when rows are destroyed
- [ ] 6.8 Add custom row event firing: customRowRender, customRowClick, customRowHover

## 7. View Integration (Day/Week/Month/Timeline/Agenda/Year)

- [ ] 7.1 Create custom row initialization call in Schedule's `initialize()` method
- [ ] 7.2 Integrate custom row rendering into `src/schedule/renderer/day.ts` Day view renderer
- [ ] 7.3 Integrate custom row rendering into `src/schedule/renderer/week.ts` Week view renderer
- [ ] 7.4 Integrate custom row rendering into `src/schedule/renderer/month.ts` Month view renderer
- [ ] 7.5 Integrate custom row rendering into Timeline view renderer
- [ ] 7.6 Integrate custom row rendering into Agenda view renderer
- [ ] 7.7 Integrate custom row rendering into Year view renderer
- [ ] 7.8 Implement view-specific custom row filtering based on viewTypes configuration
- [ ] 7.9 Implement custom row cleanup when switching views to prevent DOM leaks
- [ ] 7.10 Test custom rows render correctly in all 6 view types

## 8. Resource Grouping Integration

- [ ] 8.1 Implement resource context provision in data binding when `target: 'resource'`
- [ ] 8.2 Implement horizontal resource grouping custom row rendering
- [ ] 8.3 Implement vertical resource grouping custom row rendering
- [ ] 8.4 Filter custom row events to match selected resource
- [ ] 8.5 Test custom rows in resource Day view with horizontal grouping
- [ ] 8.6 Test custom rows in resource Week view with vertical grouping

## 9. Drag-Drop & Interaction Compatibility

- [ ] 9.1 Ensure custom row DOM doesn't interfere with event drag targets in `src/schedule/actions/drag.ts`
- [ ] 9.2 Test dragging events with custom rows visible (rows should hide during drag)
- [ ] 9.3 Test custom row positioning after drop (rows should reappear with updated data)
- [ ] 9.4 Test custom row click handlers don't interfere with event selection

## 10. Styling & Theme Integration

- [ ] 10.1 Create `src/schedule/css/schedule-custom-rows.scss` with base custom row styles
- [ ] 10.2 Add custom row style rules to each theme SCSS file: material.scss, bootstrap.scss, bootstrap4.scss, bootstrap5.scss, fabric.scss, highcontrast.scss
- [ ] 10.3 Test custom row styling with light theme (material)
- [ ] 10.4 Test custom row styling with dark theme (material-dark)
- [ ] 10.5 Verify CSS class conflicts don't occur with schedule base classes

## 11. TypeScript Type Definitions

- [ ] 11.1 Create `src/schedule/custom-rows/models/interface.ts` with all custom row interfaces
- [ ] 11.2 Export CustomRowSettings interface for public API in `src/schedule/base/interface.ts`
- [ ] 11.3 Export CustomRowContext interface for public API
- [ ] 11.4 Create type-safe event handler function signatures
- [ ] 11.5 Update `src/index.ts` to export custom row types
- [ ] 11.6 Verify TypeScript strict mode compilation with no `any` types

## 12. Unit Tests - Core Rendering

- [ ] 12.1 Create `spec/schedule/custom-rows/render/custom-row-renderer.spec.ts` test file
- [ ] 12.2 Test custom row DOM creation with template function
- [ ] 12.3 Test custom row DOM creation with template selector
- [ ] 12.4 Test template error handling and error placeholder rendering
- [ ] 12.5 Test CSS class application with single and multiple classes
- [ ] 12.6 Test row height configuration (fixed and auto)
- [ ] 12.7 Test conditional rendering (show/hide based on condition function)
- [ ] 12.8 Test multiple custom rows at same location don't conflict
- [ ] 12.9 Achieve 100% code coverage for renderer module

## 13. Unit Tests - API Methods

- [ ] 13.1 Create `spec/schedule/custom-rows/api.spec.ts` test file
- [ ] 13.2 Test `addCustomRow()` adds row to internal map
- [ ] 13.3 Test `removeCustomRow()` removes row and cleans up DOM
- [ ] 13.4 Test `updateCustomRow()` applies partial updates
- [ ] 13.5 Test `getCustomRowElement()` returns correct element or null
- [ ] 13.6 Test `refreshCustomRow()` re-executes template
- [ ] 13.7 Test ID generation and uniqueness
- [ ] 13.8 Test validation errors for missing required fields
- [ ] 13.9 Achieve 100% code coverage for API module

## 14. Unit Tests - Data Binding

- [ ] 14.1 Create `spec/schedule/custom-rows/data-binding.spec.ts` test file
- [ ] 14.2 Test event data filtering by date
- [ ] 14.3 Test event data filtering by resource
- [ ] 14.4 Test context creation with timezone conversion
- [ ] 14.5 Test recurrence expansion in event data
- [ ] 14.6 Test data change detection and row updates on actionComplete
- [ ] 14.7 Test handling of empty event lists
- [ ] 14.8 Test batch update performance (< 100ms for 50 row updates)
- [ ] 14.9 Achieve 100% code coverage for data-binding module

## 15. Unit Tests - Virtual Scrolling

- [ ] 15.1 Create `spec/schedule/custom-rows/virtualization.spec.ts` test file
- [ ] 15.2 Test element pooling and recycling
- [ ] 15.3 Test element mapping maintains correctness during recycle
- [ ] 15.4 Test DOM element count stays constant during simulated scroll
- [ ] 15.5 Test visible row detection and lazy rendering
- [ ] 15.6 Achieve 100% code coverage for virtualization module

## 16. Integration Tests - View Integration

- [ ] 16.1 Create `spec/schedule/custom-rows/view-integration.spec.ts` test file
- [ ] 16.2 Test custom row rendering in Day view
- [ ] 16.3 Test custom row rendering in Week view
- [ ] 16.4 Test custom row rendering in Month view
- [ ] 16.5 Test custom row rendering in Timeline view
- [ ] 16.6 Test custom row view filtering (viewTypes configuration)
- [ ] 16.7 Test custom row cleanup on view switch
- [ ] 16.8 Test custom rows in all 6 view types render correctly
- [ ] 16.9 Achieve 100% code coverage for view integration

## 17. Integration Tests - Data Scenarios

- [ ] 17.1 Create `spec/schedule/custom-rows/scenarios.spec.ts` test file
- [ ] 17.2 Test custom row updates on event add via UI
- [ ] 17.3 Test custom row updates on event edit via UI
- [ ] 17.4 Test custom row updates on event delete
- [ ] 17.5 Test custom row behavior with recurring events
- [ ] 17.6 Test custom row behavior with timezone changes
- [ ] 17.7 Test custom row behavior with resource grouping
- [ ] 17.8 Test multiple custom row types in same view

## 18. Integration Tests - Drag-Drop

- [ ] 18.1 Create `spec/schedule/custom-rows/drag-drop.spec.ts` test file
- [ ] 18.2 Test event drag-drop works with custom rows present
- [ ] 18.3 Test custom row updates after drag-drop to new time
- [ ] 18.4 Test custom row click doesn't interfere with event click
- [ ] 18.5 Verify drag-drop performance not degraded by custom rows

## 19. Performance Tests

- [ ] 19.1 Create `spec/schedule/custom-rows/performance.spec.ts` test file
- [ ] 19.2 Test initial render time with 100, 500, 1000 events (targets: <500ms, <1000ms, <2000ms)
- [ ] 19.3 Test scroll performance maintains >= 50 FPS with custom rows
- [ ] 19.4 Test memory usage stays < 10MB overhead for 500 events with custom rows
- [ ] 19.5 Test template rendering time < 50ms per row
- [ ] 19.6 Test update performance < 100ms when event changes
- [ ] 19.7 Test virtual scrolling keeps DOM nodes constant during scroll
- [ ] 19.8 Test large dataset (1000+ events) scroll maintains >= 30 FPS

## 20. Accessibility Tests

- [ ] 20.1 Create `spec/schedule/custom-rows/accessibility.spec.ts` test file
- [ ] 20.2 Test custom row elements have keyboard navigation support
- [ ] 20.3 Test custom row elements have proper ARIA labels
- [ ] 20.4 Test custom row focus management (keyboard Tab navigation)
- [ ] 20.5 Test custom row screen reader announcements
- [ ] 20.6 Test custom row expand/collapse keyboard support
- [ ] 20.7 Verify WCAG 2.1 Level AA compliance for custom rows

## 21. E2E Tests

- [ ] 21.1 Create `e2e/tests/custom-rows/custom-rows.e2e.spec.ts` test file
- [ ] 21.2 Test adding custom row via UI and verifying rendering
- [ ] 21.3 Test removing custom row via API and verifying cleanup
- [ ] 21.4 Test custom row updates when events are modified
- [ ] 21.5 Test view switching with custom rows (no flashing/corruption)
- [ ] 21.6 Test drag-drop with custom rows present
- [ ] 21.7 Test custom row click handlers fire correctly
- [ ] 21.8 Test custom rows in all view types via E2E

## 22. Demo & Documentation

- [ ] 22.1 Create `demos/schedule/custom-rows/default.html` demo page with basic custom row
- [ ] 22.2 Create `demos/schedule/custom-rows/summary.html` demo with summary row example
- [ ] 22.3 Create `demos/schedule/custom-rows/detail.html` demo with detail row expansion
- [ ] 22.4 Create `demos/schedule/custom-rows/aggregation.html` demo with resource aggregation
- [ ] 22.5 Add JSDoc comments to all custom row API methods
- [ ] 22.6 Create API documentation section in `README.md` for custom rows
- [ ] 22.7 Create developer guide with code examples for common use cases
- [ ] 22.8 Update CustomRowSupport.md with implementation details

## 23. Export/Import Support

- [ ] 23.1 Update Excel export logic in `src/schedule/actions/export.ts` to include custom rows
- [ ] 23.2 Test custom row content exports correctly to Excel
- [ ] 23.3 Update PDF export to include custom rows if applicable
- [ ] 23.4 Test PDF export with custom rows (if supported)
- [ ] 23.5 Custom rows excluded from ICS export (per spec)

## 24. Framework Wrapper Updates

- [ ] 24.1 Update Angular wrapper at `@syncfusion/ej2-angular-schedule` to expose customRows API
- [ ] 24.2 Update React wrapper at `@syncfusion/ej2-react-schedule` to expose customRows API
- [ ] 24.3 Update Vue wrapper at `@syncfusion/ej2-vue-schedule` to expose customRows API
- [ ] 24.4 Update Blazor wrapper at `src/blazor/schedule.ts` to expose customRows API
- [ ] 24.5 Test custom rows work correctly through each framework wrapper

## 25. Cross-Browser Testing

- [ ] 25.1 Test custom rows in Chrome (latest 2 versions)
- [ ] 25.2 Test custom rows in Firefox (latest 2 versions)
- [ ] 25.3 Test custom rows in Safari (latest version)
- [ ] 25.4 Test custom rows in Edge (latest version)
- [ ] 25.5 Test custom rows on mobile browsers (iOS Safari, Chrome Android)
- [ ] 25.6 Verify responsive behavior on tablet sizes

## 26. Memory Leak Prevention

- [ ] 26.1 Profile memory usage with heap snapshots during custom row lifecycle
- [ ] 26.2 Verify no memory leaks from event handlers
- [ ] 26.3 Verify no memory leaks from template cache
- [ ] 26.4 Verify no orphaned DOM nodes after row removal
- [ ] 26.5 Run memory leak detection tool on 1000-event dataset with custom rows
- [ ] 26.6 Document memory profiling results in `CustomRowSupport.md`

## 27. Code Review & Quality

- [ ] 27.1 Ensure all custom row code follows EJ2 coding standards
- [ ] 27.2 Run ESLint on all new custom row modules
- [ ] 27.3 Check bundle size increase is < 50KB as specified
- [ ] 27.4 Verify backward compatibility (no breaking changes to existing APIs)
- [ ] 27.5 Peer code review by 2+ team members
- [ ] 27.6 Run full test suite and achieve 100% coverage for new code

## 28. Merge & Release Prep

- [ ] 28.1 Merge feature branch to main
- [ ] 28.2 Update CHANGELOG.md with custom row feature details
- [ ] 28.3 Update version number in package.json
- [ ] 28.4 Generate API documentation from JSDoc
- [ ] 28.5 Publish to npm and verify package contents
- [ ] 28.6 Tag release in git
- [ ] 28.7 Create release notes with custom row feature highlights
