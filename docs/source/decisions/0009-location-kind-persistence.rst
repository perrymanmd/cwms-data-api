#####
Location Kind Persistence and Metadata
#####

Summary
=======

This ADR defines the relationship between the general Location identity and specialized Location Kinds (e.g., Stream, Project, Basin). It establishes the concept of "Marker" kinds versus "Metadata" rows and provides a mapping of Kinds to their respective database tables.

Problem Statement
=================

CWMS locations can embody multiple roles (e.g., a physical site that is both a Stream Gage and a Weather Gage). Currently, the transition between these kinds is not always well-defined, leading to potential data orphans or loss of specialized metadata. We need a clear policy on how the "KIND" column in ``AT_PHYSICAL_LOCATION`` interacts with specialized tables like ``AT_STREAM`` or ``AT_PROJECT``.

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
     -
   * - EMBANKMENT
     - X
     -
     -
     -
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
     -
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
     -
     -
     -
     -
   * - LOCK
     - X
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
     -
     -
     -
   * - STREAM_LOCATION
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
     -
     -
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
     -
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
     -
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
     - X
   * - WEATHER_GAGE
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
     -
     -
     -
   * - ENTITY
     - X
     -
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
     -
     -

**Legend:**
- **X**: Required (The presence of this Kind requires a row in the corresponding table).
- (Blank): Not Allowed or Not Applicable for the primary definition of the Kind.

Terminology
===========

Marker
------
A location is "marked" as a specific Kind in the ``AT_PHYSICAL_LOCATION`` table, but may or may not have the corresponding metadata rows in specialized tables yet. The Kind in ``AT_PHYSICAL_LOCATION`` serves as the primary functional role indicator.

Orphan
------
An "orphan" occurs when a location's Kind is designated (e.g., as a ``PROJECT``), but no corresponding row exists in the kind-specific metadata table (e.g., ``AT_PROJECT``). While functionally a "marker," this state may be considered incomplete for certain API operations.

Behavioral Rules
================

Kind Transitions
----------------
1. **New Rows Required**: Changing a Location's Kind in ``AT_PHYSICAL_LOCATION`` should generally require the creation of a new row in the corresponding ``AT_<?>`` table if it doesn't already exist.
2. **Preservation of Existing Data**: Storing a new Kind marker should not automatically delete existing metadata from other kind-specific tables. A location that was a ``STREAM_GAGE`` and is now marked as a ``PROJECT`` should ideally retain its gage metadata unless explicitly removed.

API Endpoint Expectations
-------------------------
1. **Filtering by Kind**: The general Location endpoint (getAll) should filter based on the Kind marker in ``AT_PHYSICAL_LOCATION``.
2. **Specialized Endpoints**: Kind-specific endpoints (e.g., ``/projects``, ``/streams``) must decide whether to return "marker-only" (orphan) locations.
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

References
==========

Related Types: ``cwms.cda.data.dto.Location``, ``cwms.cda.data.dto.CwmsIdLocationKind``
Database Tables: ``AT_PHYSICAL_LOCATION``, ``AT_STREAM``, ``AT_BASIN``, ``AT_GAGE``, ``AT_ENTITY``, ``AT_PROJECT``, ``AT_EMBANKMENT``, ``AT_OUTLET``, ``AT_TURBINE``, ``AT_LOCK``, ``AT_OVERFLOW``, ``AT_STREAM_LOCATION``, ``AT_STREAM_REACH``, ``AT_PUMP``
