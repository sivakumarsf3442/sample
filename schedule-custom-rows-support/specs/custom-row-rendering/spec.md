# Custom Row Rendering Specification

## ADDED Requirements

### Requirement: Custom row DOM creation and insertion
The system SHALL create HTML elements for custom rows with proper DOM structure and insert them at specified positions within the scheduler view hierarchy.

#### Scenario: Create custom row with template content
- **WHEN** a custom row is configured with type 'summary' and a template function
- **THEN** the system SHALL create a div with class 'e-custom-row' containing the template-rendered content
- **AND** the system SHALL insert this element after the corresponding event's DOM node

#### Scenario: Insert custom row before event
- **WHEN** a custom row configuration specifies position 'before' and target 'event'
- **THEN** the custom row SHALL be inserted before the event element in the DOM
- **AND** the custom row SHALL be positioned correctly visually within the scheduler layout

#### Scenario: Handle custom row with missing template
- **WHEN** a custom row configuration lacks both template and templateId properties
- **THEN** the system SHALL throw a validation error immediately
- **AND** the error message SHALL indicate "Custom row must have a valid template or templateId"

### Requirement: Template rendering with context
The system SHALL render custom row templates with access to relevant event and schedule context data.

#### Scenario: Pass event data to template function
- **WHEN** a custom row template function is executed
- **THEN** the function SHALL receive an object containing: `{ events: EventModel[], resource?: ResourceModel, startTime: Date, endTime: Date }`
- **AND** the template function SHALL be able to access all event properties (subject, startTime, endTime, etc.)

#### Scenario: Return HTML string from template
- **WHEN** a template function returns an HTML string
- **THEN** the custom row SHALL render the HTML exactly as provided
- **AND** the rendered HTML SHALL be sanitized to prevent XSS attacks

#### Scenario: Return DOM element from template
- **WHEN** a template function returns an HTMLElement object
- **THEN** the custom row SHALL insert this element directly into the DOM
- **AND** existing event handlers on the element SHALL be preserved

#### Scenario: Catch template rendering errors
- **WHEN** a template function throws an exception
- **THEN** the system SHALL catch the error and log it to the console
- **AND** the custom row SHALL display an error placeholder: `<div class="e-custom-row-error">Template rendering failed</div>`
- **AND** the scheduler SHALL continue functioning normally

### Requirement: Template selector support
The system SHALL support retrieving templates from the DOM by element ID.

#### Scenario: Get template by ID selector
- **WHEN** a custom row configuration specifies `templateId: '#my-template'`
- **AND** an element with id 'my-template' exists in the DOM
- **THEN** the system SHALL retrieve the element's innerHTML as the template
- **AND** the custom row SHALL render using this template content

#### Scenario: Handle missing template element
- **WHEN** a custom row configuration specifies `templateId: '#nonexistent'`
- **THEN** the system SHALL throw a validation error
- **AND** the error message SHALL indicate "Template element with ID '#nonexistent' not found"

### Requirement: Custom row CSS class application
The system SHALL apply CSS classes to custom row elements for styling and theming.

#### Scenario: Apply single CSS class
- **WHEN** a custom row configuration specifies `cssClass: 'my-custom-row'`
- **THEN** the rendered custom row element SHALL have class 'my-custom-row' applied
- **AND** the element SHALL retain the base class 'e-custom-row'

#### Scenario: Apply multiple CSS classes
- **WHEN** a custom row configuration specifies `cssClass: 'my-custom-row theme-dark'`
- **THEN** the rendered custom row element SHALL have all classes: 'e-custom-row my-custom-row theme-dark'

#### Scenario: Validate CSS class names
- **WHEN** a custom row configuration specifies invalid CSS class characters like `cssClass: 'invalid@class'`
- **THEN** the system SHALL log a warning to the console
- **AND** the custom row SHALL still render using provided content

### Requirement: Custom row height configuration
The system SHALL support configurable heights for custom rows.

#### Scenario: Set fixed row height
- **WHEN** a custom row configuration specifies `height: 40`
- **THEN** the custom row element SHALL have inline style `height: 40px`
- **AND** content exceeding this height SHALL be hidden or scrollable based on CSS overflow property

#### Scenario: Set auto height
- **WHEN** a custom row configuration specifies `height: 'auto'`
- **THEN** the custom row element SHALL size to fit its content naturally
- **AND** the row height SHALL adjust if content changes

#### Scenario: Validate height values
- **WHEN** a custom row configuration specifies `height: 15` (below minimum of 20)
- **THEN** the system SHALL log a warning
- **AND** the height SHALL be constrained to minimum 20px

### Requirement: View-specific rendering
The system SHALL render different custom rows for different Schedule view types.

#### Scenario: Render rows only for specified views
- **WHEN** a custom row configuration specifies `viewTypes: ['Day', 'Week']`
- **THEN** the custom row SHALL only render when viewing Day or Week views
- **AND** the custom row SHALL NOT render in Month, Agenda, Timeline, or Year views

#### Scenario: Apply to all views by default
- **WHEN** a custom row configuration omits the `viewTypes` property
- **THEN** the custom row SHALL render in all available view types
- **AND** each view SHALL render the row appropriately for its layout

#### Scenario: Hide custom row on view switch
- **WHEN** the user switches from a view where custom rows should render to a view where they shouldn't
- **THEN** the scheduler SHALL remove (unmount) the custom row DOM elements
- **AND** no memory leaks SHALL occur from orphaned event handlers

### Requirement: Conditional row rendering
The system SHALL support conditional rendering based on data evaluation.

#### Scenario: Show row if condition is true
- **WHEN** a custom row configuration includes `condition: (events) => events.length > 0`
- **AND** the event array contains at least one event
- **THEN** the custom row SHALL be rendered

#### Scenario: Hide row if condition is false
- **WHEN** a custom row configuration includes `condition: (events) => events.length > 0`
- **AND** the event array is empty
- **THEN** the custom row SHALL NOT be rendered
- **AND** no DOM element SHALL be created for this row

#### Scenario: Condition error handling
- **WHEN** a condition function throws an exception
- **THEN** the system SHALL catch the error and log it
- **AND** the custom row SHALL default to visible (render)

### Requirement: Multiple custom rows per location
The system SHALL support rendering multiple custom rows at the same location without conflicts.

#### Scenario: Stack multiple rows
- **WHEN** two custom row configurations target the same event with position 'after'
- **THEN** both custom rows SHALL be rendered
- **AND** they SHALL appear in the order they were added to the configuration
- **AND** both rows SHALL be visible and not overlap

#### Scenario: Manage render order
- **WHEN** custom rows are configured with different positions (one 'before', one 'after')
- **AND** both target the same event
- **THEN** the 'before' row SHALL appear above the event element
- **AND** the 'after' row SHALL appear below the event element
- **AND** the visual hierarchy SHALL be maintained

### Requirement: Custom row element cleanup
The system SHALL properly remove custom row DOM elements when needed.

#### Scenario: Remove row on unmount
- **WHEN** a custom row is removed via `removeCustomRow()` method
- **THEN** all DOM elements for that row SHALL be removed from the document
- **AND** all associated event handlers SHALL be detached
- **AND** no references SHALL remain in memory

#### Scenario: Clear rows on view switch
- **WHEN** the user switches to a view type not in the custom row's `viewTypes` list
- **THEN** the row's DOM element SHALL be removed
- **AND** cleanup code SHALL execute to prevent memory leaks

#### Scenario: Cleanup on Schedule destroy
- **WHEN** the Schedule component is destroyed
- **THEN** all custom row DOM elements SHALL be removed
- **AND** all custom row event listeners SHALL be detached
- **AND** the custom row rendering module SHALL be uninitialized

## MODIFIED Requirements

### Requirement: Event rendering pipeline
The existing event rendering system SHALL be extended to support custom row rendering alongside standard event rendering.

#### Scenario: Render event and custom row together
- **WHEN** a view renders an event with associated custom rows
- **THEN** the event element SHALL be rendered first
- **AND** then custom row elements SHALL be rendered according to their position configuration
- **AND** both elements SHALL be properly positioned in the DOM hierarchy
- **AND** no visual conflicts SHALL occur

#### Scenario: Custom rows don't interfere with event layout
- **WHEN** custom rows are rendered in the same container as events
- **THEN** custom rows SHALL NOT affect the positioning or sizing of standard event elements
- **AND** event drag-drop targets SHALL remain accessible
- **AND** event click handlers SHALL continue to function normally

## REMOVED Requirements

None.
