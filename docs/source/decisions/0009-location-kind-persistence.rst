#####
Location Kind Persistence and Metadata
#####

Summary
=======

This ADR defines the relationship between the general Location identity and specialized Location Kinds (e.g., Stream, Project, Basin). It establishes the concept of "Marker" kinds versus "Metadata" rows and provides a mapping of Kinds to their respective database tables.

Problem Statement
=================

CWMS locations can embody multiple roles (e.g., a physical site that is both an Embankment and a Stream Location). Currently, changing a location's Kind in ``AT_PHYSICAL_LOCATION`` generally requires a corresponding row in the specialized ``AT_<?>`` table. If the row does not exist, the operation may fail or the Kind may not be properly updated.

The concept of "Marker Kinds"—where a Kind is set in ``AT_PHYSICAL_LOCATION`` as a functional indicator without requiring immediate population of specialized metadata—is not currently supported. This ADR addresses how such a system would work, allowing for more flexible location management and preventing loss of specialized metadata during transitions.

Location Kind and Table Mapping
===============================

The following table defines the required and allowed associations between Location Kinds and database tables.

.. list-table:: Location Kind to Table Mapping
   :header-rows: 1
   :stub-columns: 1

   * - Location Kind
     - AT_PHYSICAL_LOCATION
     - AT_STREAM
     - AT_BASIN
     - AT_GAGE
     - AT_ENTITY
     - AT_PROJECT
     - AT_EMBANKMENT
     - AT_OUTLET
     - AT_TURBINE
     - AT_LOCK
     - AT_OVERFLOW
     - AT_STREAM_LOCATION
     - AT_STREAM_REACH
     - AT_PUMP
   * - SITE
     - X
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - STREAM
     - X
     - X
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - BASIN
     - X
     -
     - X
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - PROJECT
     - X
     -
     -
     - A
     - A
     - X
     -
     -
     -
     -
     - A
     -
     -
     -
   * - EMBANKMENT
     - X
     -
     -
     - A
     -
     -
     - X
     -
     -
     -
     - A
     -
     -
     -
   * - OUTLET
     - X
     -
     -
     -
     -
     -
     -
     - X
     -
     -
     - A
     -
     -
     -
   * - TURBINE
     - X
     -
     -
     -
     -
     -
     -
     -
     - X
     -
     - A
     -
     -
     -
   * - LOCK
     - X
     -
     -
     - A
     - A
     -
     -
     -
     -
     - X
     - A
     -
     -
     - A
   * - STREAM_LOCATION
     - X
     -
     -
     - A
     - A
     -
     -
     -
     -
     -
     -
     - X
     -
     -
   * - GATE
     - X
     -
     -
     -
     -
     -
     -
     - X
     -
     - A
     - A
     -
     -
     -
   * - OVERFLOW
     - X
     -
     -
     -
     -
     -
     -
     - X
     -
     -
     - X
     - A
     -
     -
   * - STREAM_GAGE
     - X
     -
     -
     - X
     -
     -
     -
     -
     -
     -
     -
     - X
     -
     - A
   * - STREAM_REACH
     - X
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     - X
     -
   * - PUMP
     - X
     -
     -
     - A
     - A
     -
     - A
     -
     -
     - A
     -
     - X
     -
     - X
   * - WEATHER_GAGE
     - X
     -
     -
     - X
     - A
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - ENTITY
     - X
     -
     -
     - A
     - X
     -
     -
     -
     -
     -
     -
     -
     -
     -

**Legend:**
- **X**: Required in the current system. Under the proposed Marker system, this indicates the table where metadata *would* reside if the Kind is more than just a marker.
- **A**: Allowed.
- (Blank): Not Allowed.

Terminology
===========

Marker
------
A location is "marked" as a specific Kind in the ``AT_PHYSICAL_LOCATION`` table, but may or may not have the corresponding metadata rows in specialized tables yet. The Kind in ``AT_PHYSICAL_LOCATION`` serves as the primary functional role indicator.

Behavioral Rules
================

Kind Transitions
----------------
1. **Current Behavior (New Rows Required)**: Currently, changing a Location's Kind in ``AT_PHYSICAL_LOCATION`` requires the creation of a new row in the corresponding ``AT_<?>`` table if it doesn't already exist.
2. **Proposed Marker Support**: Under the proposed Marker system, the Kind in ``AT_PHYSICAL_LOCATION`` can be updated independently. If no specialized metadata row exists, the location is considered a "Marker" of that Kind.
3. **Preservation of Existing Data**: Storing a new Kind marker should not automatically delete existing metadata from other kind-specific tables. A location that was a ``STREAM_GAGE`` and is now marked as a ``PROJECT`` should retain its gage metadata unless explicitly removed.

API Endpoint Expectations
-------------------------
1. **Filtering by Kind**: The general Location endpoint (getAll) should filter based on the Kind marker in ``AT_PHYSICAL_LOCATION``.
2. **Specialized Endpoints**: Kind-specific endpoints (e.g., ``/projects``, ``/streams``) must decide whether to return "marker-only" locations.
    - *Proposed*: A "marker-project" should be visible to the project endpoint, but may return null or default values for specialized fields if the ``AT_PROJECT`` row is missing.
3. **Workflow**: Defining a complex Kind (like a Project) involves two steps:
    - Establishing the identity and Marker via the Location endpoint.
    - Populating specialized metadata via the Kind-specific endpoint.

Implementation Strategy
=======================

1. **Database Procedures**: Leverage existing CWMS database procedures for storing locations and kinds, ensuring they handle the cross-table logic correctly.
2. **CDA Endpoints**: Update CDA controllers to respect the marker-based filtering and handle "sparse" metadata gracefully.
3. **Risk Mitigation**: Ensure that existing applications expecting "complete" rows in specialized tables are not broken by the introduction of marker-only entries.

Decision Status
===============

(Status: proposed)

Notes on Current State
======================

The current behavior of the CWMS Data API and the underlying database procedures is that a location's Kind is tightly coupled with its specialized metadata. If a user attempts to change the Kind to ``PROJECT`` via the location endpoint, but no row exists in ``AT_PROJECT``, the system may revert to ``SITE`` or fail to update as expected. This ADR serves as a blueprint for decoupling these concepts to support "Marker Kinds".

References
==========

Related Types: ``cwms.cda.data.dto.Location``, ``cwms.cda.data.dto.CwmsIdLocationKind``
Database Tables: ``AT_PHYSICAL_LOCATION``, ``AT_STREAM``, ``AT_BASIN``, ``AT_GAGE``, ``AT_ENTITY``, ``AT_PROJECT``, ``AT_EMBANKMENT``, ``AT_OUTLET``, ``AT_TURBINE``, ``AT_LOCK``, ``AT_OVERFLOW``, ``AT_STREAM_LOCATION``, ``AT_STREAM_REACH``, ``AT_PUMP``
