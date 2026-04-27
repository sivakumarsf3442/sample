# Custom Row API Specification

## ADDED Requirements

### Requirement: Schedule.customRows property
The Schedule component SHALL expose a customRows property to store custom row configurations.

#### Scenario: Access custom rows property
- **WHEN** a developer accesses the `schedule.customRows` property
- **THEN** it SHALL return an array of CustomRowSettings objects
- **AND** the property SHALL be of type `CustomRowSettings[]`

#### Scenario: Set custom rows during initialization
- **WHEN** creating a Schedule instance with `customRows` property in config
- **THEN** the custom rows SHALL be applied during Schedule initialization
- **AND** they SHALL render when the view renders

### Requirement: addCustomRow method
The Schedule component SHALL provide an addCustomRow method to register custom rows at runtime.

#### Scenario: Add single custom row
- **WHEN** calling `schedule.addCustomRow(config)` with a valid CustomRowSettings object
- **THEN** the custom row configuration SHALL be added to the schedule
- **AND** the view SHALL re-render to display the new custom row
- **AND** the method SHALL complete without errors

#### Scenario: Add multiple custom rows
- **WHEN** calling `schedule.addCustomRow()` multiple times with different configurations
- **THEN** each custom row SHALL be added independently
- **AND** all custom rows SHALL render in the specified order
- **AND** no previous custom rows SHALL be removed

#### Scenario: Validate custom row configuration
- **WHEN** calling `schedule.addCustomRow()` with missing required fields (e.g., no template)
- **THEN** the system SHALL throw a validation error immediately
- **AND** the custom row SHALL NOT be added to the schedule
- **AND** no view re-render SHALL occur

### Requirement: removeCustomRow method
The Schedule component SHALL provide a removeCustomRow method to remove custom rows.

#### Scenario: Remove existing custom row by ID
- **WHEN** calling `schedule.removeCustomRow('summary-row-1')` where this custom row exists
- **THEN** the custom row configuration SHALL be removed from the schedule
- **AND** the custom row DOM elements SHALL be removed from the view
- **AND** the view SHALL update to reflect the removal

#### Scenario: Remove nonexistent custom row
- **WHEN** calling `schedule.removeCustomRow('nonexistent-id')`
- **THEN** the system SHALL log a warning to the console
- **AND** no error SHALL be thrown
- **AND** the schedule SHALL continue functioning normally

### Requirement: updateCustomRow method
The Schedule component SHALL provide an updateCustomRow method to modify custom row configurations.

#### Scenario: Update custom row template
- **WHEN** calling `schedule.updateCustomRow('row-id', { template: newTemplateFunc })`
- **THEN** the custom row template SHALL be updated to use the new function
- **AND** the custom row SHALL re-render with new content
- **AND** other properties of the custom row SHALL remain unchanged

#### Scenario: Update partial configuration
- **WHEN** calling `schedule.updateCustomRow('row-id', { height: 50 })`
- **THEN** only the specified properties SHALL be updated
- **AND** unspecified properties SHALL retain their previous values
- **AND** the view SHALL re-render to apply changes

#### Scenario: Update nonexistent custom row
- **WHEN** calling `schedule.updateCustomRow('nonexistent-id', config)`
- **THEN** the system SHALL log a warning to the console
- **AND** no row SHALL be created
- **AND** the schedule SHALL continue functioning

### Requirement: getCustomRowElement method
The Schedule component SHALL provide a getCustomRowElement method to retrieve rendered DOM elements.

#### Scenario: Get rendered custom row element
- **WHEN** calling `schedule.getCustomRowElement('row-id')` for an existing, rendered custom row
- **THEN** the method SHALL return the HTMLElement of the custom row
- **AND** the returned element SHALL have class 'e-custom-row'
- **AND** developers SHALL be able to manipulate or inspect the element

#### Scenario: Get element for non-rendered row
- **WHEN** calling `schedule.getCustomRowElement('row-id')` for a custom row not currently rendered
- **THEN** the method SHALL return `null`
- **AND** no error SHALL be thrown

#### Scenario: Get element for nonexistent row
- **WHEN** calling `schedule.getCustomRowElement('nonexistent-id')`
- **THEN** the method SHALL return `null`

### Requirement: refreshCustomRow method
The Schedule component SHALL provide a refreshCustomRow method to force re-rendering.

#### Scenario: Force re-render single row
- **WHEN** calling `schedule.refreshCustomRow('row-id')`
- **THEN** the custom row template function SHALL be re-executed
- **AND** the custom row DOM element SHALL be updated with fresh content
- **AND** the render time SHALL not exceed 100ms

#### Scenario: Refresh with updated context
- **WHEN** calling `schedule.refreshCustomRow('row-id')` after event data has changed
- **THEN** the template function SHALL receive the updated event data
- **AND** the custom row content SHALL reflect these updates

#### Scenario: Refresh nonexistent row
- **WHEN** calling `schedule.refreshCustomRow('nonexistent-id')`
- **THEN** the system SHALL log a warning
- **AND** no error SHALL be thrown

### Requirement: CustomRowSettings interface
The system SHALL define a TypeScript interface for custom row configuration.

#### Scenario: CustomRowSettings with all properties
- **WHEN** creating a CustomRowSettings object with all optional properties
- **THEN** the object SHALL have properties: id, type, template, templateId, target, position, viewTypes, condition, cssClass, height, handlers
- **AND** TypeScript compilation SHALL succeed without type errors
- **AND** all property types SHALL match their expected definitions

#### Scenario: Minimum valid configuration
- **WHEN** creating a CustomRowSettings object with only required properties (type, target, template)
- **THEN** the object SHALL be valid and complete
- **AND** optional properties SHALL have appropriate default values
- **AND** TypeScript compilation SHALL succeed

#### Scenario: Type checking on configuration
- **WHEN** assigning a value of incorrect type to a CustomRowSettings property
- **THEN** TypeScript compilation SHALL fail
- **AND** IDEs SHALL display type mismatch warnings

### Requirement: Custom row event handlers
The system SHALL support event handler registration on custom rows.

#### Scenario: Register click handler
- **WHEN** a custom row configuration includes `handlers: { click: myHandler }`
- **AND** a user clicks on the custom row element
- **THEN** the click handler function SHALL be invoked
- **AND** the handler SHALL receive event context object: `{ eventData, customRowElement, target }`

#### Scenario: Register hover handler
- **WHEN** a custom row configuration includes `handlers: { hover: myHandler }`
- **AND** user mouse enters/exits the custom row
- **THEN** the hover handler SHALL be invoked
- **AND** the handler SHALL receive the custom row element

#### Scenario: Register expand handler
- **WHEN** a custom row of type 'detail' includes `handlers: { expand: myHandler }`
- **AND** user clicks the expand button on the detail row
- **THEN** the expand handler SHALL be invoked to handle expand/collapse logic

#### Scenario: Handler receives correct context
- **WHEN** a handler function is called
- **THEN** it SHALL receive an object with properties: `{ eventData: EventModel[], customRowElement: HTMLElement, target: HTMLElement }`
- **AND** developers SHALL be able to access event data and DOM elements

#### Scenario: Handler exceptions are caught
- **WHEN** an event handler throws an exception
- **THEN** the exception SHALL be caught and logged
- **AND** the scheduler SHALL continue functioning normally
- **AND** other event handlers SHALL continue to execute

### Requirement: Custom row types enumeration
The system SHALL define available custom row types.

#### Scenario: Summary type
- **WHEN** creating a custom row with `type: 'summary'`
- **THEN** the custom row type SHALL be recognized as 'summary'
- **AND** it SHALL indicate a summary/aggregate row

#### Scenario: Detail type
- **WHEN** creating a custom row with `type: 'detail'`
- **THEN** the custom row type SHALL be recognized as 'detail'
- **AND** it SHALL indicate an expandable detail row

#### Scenario: Aggregation type
- **WHEN** creating a custom row with `type: 'aggregation'`
- **THEN** the custom row type SHALL be recognized as 'aggregation'
- **AND** it SHALL indicate resource/group aggregation row

#### Scenario: Custom type
- **WHEN** creating a custom row with `type: 'custom'`
- **THEN** the custom row type SHALL be recognized as 'custom'
- **AND** developers SHALL have full control over rendering

#### Scenario: Invalid type validation
- **WHEN** attempting to create a custom row with `type: 'invalid'`
- **THEN** the system SHALL throw a validation error
- **AND** the error message SHALL list valid type options

## MODIFIED Requirements

### Requirement: Schedule model extensibility
The existing ScheduleModel interface SHALL be extended to include custom row support.

#### Scenario: ScheduleModel includes customRows property
- **WHEN** examining the ScheduleModel interface definition
- **THEN** it SHALL include `@Property() customRows: CustomRowSettings[]`
- **AND** the property SHALL participate in Schedule's property system (@Property decorator)
- **AND** property change notifications SHALL fire when customRows is modified

## REMOVED Requirements

None.
