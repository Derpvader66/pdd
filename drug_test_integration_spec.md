# Business Problems

Users of the Drug Test Integration page experience usability and data access challenges that impede their ability to efficiently search, filter, and interpret drug test records. These challenges include:

- Inability to restrict search results by the user’s authorized county, leading to confusion and potential data exposure.
- Limitations in searching by a known Reference Number without specifying a record type, slowing down data retrieval during urgent workflows.
- Lack of visibility into when test records were received, making it difficult to track timeliness and troubleshoot integration issues.

## Solution Summary
The enhancements address these issues by:

- Restricting the **County dropdown** to reflect only the locations assigned to the user, improving data access control and clarity.
- Introducing a **Reference Number** search type that disables unrelated filters and allows direct retrieval of a record regardless of its type—reducing search friction.
- Displaying the **Received Date** in all relevant search result types to improve traceability, accountability, and troubleshooting efficiency.

---

Feature: County Dropdown Access Control

  Background:
    Given I am logged in
    And I navigate to the Drug Test Integration page

  Scenario Outline: User sees only their assigned counties in the County dropdown
    Given my Personnel page shows current locations: <locations>
    When I expand the County dropdown
    Then the County dropdown options are: <expected options>

    Examples:
      | locations         | expected options   | verification context           |
      | Maricopa          | Maricopa           | Single county assigned         |
      | Maricopa, Pima    | Maricopa, Pima     | Multiple counties assigned     |
      | (none)            | (none)             | No county assigned to user     |

Feature: Reference Number Record Type Functionality

  Background:
    Given I am logged in
    And I navigate to the Drug Test Integration page

  Scenario: Record Type dropdown includes only expected options
    When I expand the Record Type dropdown
    Then the Record Type options include:
      | Reference Number |
    And no unexpected options are present

  Scenario: Selecting Reference Number disables only the expected search fields
    When I select 'Reference Number' from the Record Type dropdown
    Then the following fields are disabled:
      | Timeframe        |
      | Individual       |
      | Provider         |
      | County           |
      | Notification Type |
    And no other search fields are disabled

  Scenario: Searching by valid Reference Number returns the record
    Given I am logged in with county: Maricopa
    And I select 'Reference Number' from the Record Type dropdown
    And I enter a valid reference number associated with Maricopa
    When I click Search
    Then the matching record is returned

  Scenario: Searching by Reference Number for a record outside the user's assigned county returns no results
    Given I am logged in with county: Maricopa
    And I select 'Reference Number' from the Record Type dropdown
    And I enter a valid reference number associated with Pima
    When I click Search
    Then no record is returned

Feature: Received Date Display in Search Results

  Background:
    Given I am logged in
    And I navigate to the Drug Test Integration page

  Scenario Outline: Results show Received Date with proper format and placement
    Given I select record type: <record type>
    And I enter search criteria that returns results
    When I click Search
    Then each result row includes:
      | Column         | Placement         | Format                |
      | Received Date  | After Reference # | MM/DD/YYYY HH:MM AM/PM |
    And the Received Date value matches the expected datetime format

    Examples:
      | record type      |
      | Responses        |
      | Errors           |
      | Unmatched        |
      | Reference Number |
