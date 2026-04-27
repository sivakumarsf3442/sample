# Custom Row Data Binding Specification

## ADDED Requirements

### Requirement: Event data synchronization
The system SHALL automatically update custom rows when event data changes.

#### Scenario: Update custom row on event add
- **WHEN** a new event is added to the Schedule via `addEvent()` or drag-drop
- **THEN** the `actionComplete` event fires
- **AND** custom rows with `target: 'date'` or `target: 'event'` SHALL be re-rendered with updated data
- **AND** the custom row content SHALL reflect the newly added event

#### Scenario: Update custom row on event edit
- **WHEN** an event is modified via event editor or drag-drop
- **THEN** the `actionComplete` event fires
- **AND** custom rows targeting that event's date/resource SHALL be re-rendered
- **AND** the custom row template SHALL receive updated event data

#### Scenario: Update custom row on event delete
- **WHEN** an event is deleted
- **THEN** the `actionComplete` event fires
- **AND** custom rows affected by this deletion SHALL be re-rendered
- **AND** the custom row content SHALL no longer include the deleted event

#### Scenario: Batch event updates
- **WHEN** multiple events are modified in succession (e.g., via API calls)
- **THEN** custom rows SHALL be re-rendered for each update
- **AND** no duplicate renders SHALL occur for the same row

### Requirement: Filtered event data provision
The system SHALL provide filtered event data to custom row templates based on context.

#### Scenario: Provide events for date cell
- **WHEN** a custom row with `target: 'date'` renders for a specific date
- **THEN** the template context SHALL contain only events occurring on that date
- **AND** the events array SHALL be of type EventModel[]
- **AND** all event properties SHALL be accessible (subject, startTime, endTime, etc.)

#### Scenario: Provide events for resource
- **WHEN** a custom row with `target: 'resource'` renders in a resource view
- **THEN** the template context SHALL include `resource: ResourceModel` property
- **AND** the events array SHALL be filtered to events for that resource only

#### Scenario: Provide time range context
- **WHEN** a custom row renders
- **THEN** the template context SHALL include `startTime` and `endTime` properties
- **AND** these SHALL define the time period for the custom row's data
- **AND** developers can use these to implement custom filtering logic

#### Scenario: Handle empty event list
- **WHEN** a custom row is rendered for a date/resource with no events
- **THEN** the template context events array SHALL be empty `[]`
- **AND** the condition function (if present) SHALL receive this empty array
- **AND** developers can render an appropriate empty state

### Requirement: Real-time data update with diff detection
The system SHALL minimize unnecessary re-renders by detecting actual data changes.

#### Scenario: Skip re-render on unchanged data
- **WHEN** `actionComplete` fires but the events affecting this custom row are unchanged
- **THEN** the custom row SHALL NOT be re-rendered
- **AND** no template function calls SHALL occur for unchanged rows

#### Scenario: Re-render on data change
- **WHEN** `actionComplete` fires and event data relevant to the custom row has changed
- **THEN** the custom row SHALL be re-rendered
- **AND** template function SHALL be called with new data

#### Scenario: Handle complex data changes
- **WHEN** event properties change (e.g., subject, startTime, color)
- **THEN** the diff detection SHALL identify relevant changes
- **AND** affected custom rows SHALL be updated

#### Scenario: Performance with many events
- **WHEN** a view contains 500+ events and multiple custom rows
- **THEN** each `actionComplete` event processing SHALL complete within 100ms
- **AND** only affected custom rows SHALL be re-rendered
- **AND** the UI SHALL remain responsive

### Requirement: Recurrence event handling
The system SHALL handle recurrence patterns correctly in custom row data binding.

#### Scenario: Provide recurrence context
- **WHEN** a custom row template receives event data containing recurring events
- **THEN** each visible occurrence SHALL appear in the events array
- **AND** all occurrences SHALL reference the parent recurrence rule
- **AND** developers can access recurrence information via event properties

#### Scenario: Handle recurrence modification
- **WHEN** a single occurrence of a recurring event is modified
- **THEN** only that occurrence's custom row SHALL be updated
- **AND** other occurrences' custom rows SHALL NOT be affected

#### Scenario: Handle series deletion
- **WHEN** an entire recurring event series is deleted
- **THEN** all custom rows for that series SHALL be updated
- **AND** the custom rows SHALL reflect the series deletion

### Requirement: Timezone-aware data binding
The system SHALL respect timezone settings when providing event data to custom rows.

#### Scenario: Events shown in selected timezone
- **WHEN** a Schedule is configured with a timezone (e.g., 'America/New_York')
- **AND** event times are stored in UTC
- **THEN** custom row templates SHALL receive event times converted to the selected timezone
- **AND** `startTime` and `endTime` in context SHALL be in the selected timezone

#### Scenario: Daylight saving time handling
- **WHEN** events span daylight saving time boundaries
- **THEN** event times in custom row context SHALL be correctly adjusted
- **AND** duration calculations SHALL account for DST changes

### Requirement: Resource data in custom row context
The system SHALL provide resource information when rendering custom rows in resource views.

#### Scenario: Access resource in aggregation row
- **WHEN** a custom row with `target: 'resource'` renders
- **THEN** the template context SHALL include `resource: ResourceModel`
- **AND** developers can access resource properties (name, id, color, etc.)
- **AND** the events array SHALL be pre-filtered to this resource

#### Scenario: Horizontal grouping resource context
- **WHEN** resources are displayed horizontally and custom row renders
- **THEN** the resource context SHALL indicate the specific resource column
- **AND** the data binding SHALL be scoped to that resource

#### Scenario: Vertical grouping resource context
- **WHEN** resources are displayed vertically and custom row renders
- **THEN** the resource context SHALL indicate the specific resource group
- **AND** the data binding SHALL be scoped to that resource

### Requirement: Computed properties and aggregations
The system SHALL support computed properties in custom row data.

#### Scenario: Compute event count
- **WHEN** a template needs to display event count
- **THEN** developers can compute via `context.events.length`
- **AND** this computation SHALL occur before rendering
- **AND** the count SHALL be accurate and reflect filtered events

#### Scenario: Compute total duration
- **WHEN** a template computes total event duration
- **THEN** developers can sum event durations from context.events
- **AND** the computation SHALL respect event times (startTime, endTime)
- **AND** overlapping events SHALL be handled correctly

#### Scenario: Compute occupancy percentage
- **WHEN** a resource aggregation row displays occupancy
- **THEN** developers can compute percentage from available time vs event time
- **AND** the context SHALL provide sufficient data for this calculation

### Requirement: Data binding lifecycle
The system SHALL properly manage data binding through Schedule lifecycle phases.

#### Scenario: Initialize data binding on Schedule create
- **WHEN** Schedule is created with custom rows configured
- **THEN** data binding listeners SHALL be registered
- **AND** custom rows SHALL render with initial data

#### Scenario: Update data binding on view change
- **WHEN** user switches views (e.g., Month to Week)
- **THEN** data binding context SHALL update to reflect new view's date range
- **AND** custom rows SHALL be re-rendered with view-appropriate data

#### Scenario: Cleanup data binding on Schedule destroy
- **WHEN** Schedule is destroyed
- **THEN** all data binding listeners SHALL be unregistered
- **AND** no orphaned event subscriptions SHALL remain
- **AND** memory SHALL be properly released

### Requirement: Two-way binding for detail expansion
The system SHALL support two-way binding for expandable detail rows.

#### Scenario: Sync detail expansion state
- **WHEN** a user expands a detail row
- **THEN** the expand state SHALL be tracked
- **AND** template re-renders SHALL preserve the expansion state
- **AND** other UI interactions SHALL reflect this state

#### Scenario: Update detail content on event change
- **WHEN** an event changes while its detail row is expanded
- **THEN** the detail row content SHALL update
- **AND** the row SHALL remain expanded
- **AND** the new content SHALL be rendered within 100ms

## MODIFIED Requirements

### Requirement: Schedule actionComplete event
The existing actionComplete event system SHALL be extended to support custom row data binding.

#### Scenario: actionComplete fires for all CRUD operations
- **WHEN** actionComplete event fires after any CRUD operation (add/edit/delete)
- **THEN** custom row data binding handlers SHALL be triggered
- **AND** custom rows SHALL be updated if their data context changed
- **AND** the custom row update SHALL complete before actionComplete listener functions execute

## REMOVED Requirements

None.
