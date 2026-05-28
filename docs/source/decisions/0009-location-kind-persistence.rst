#####
Location Kind Persistence and Metadata
#####


Summary
=======

This ADR documents the behavior and relationship between general location endpoints and kind-specific endpoints (e.g., Stream, Project), specifically addressing how storing a Location with a set KIND acts as a "marker" and the implications for metadata retention.


Problem Statement
=================

Currently, when a physical location is stored without an assigned kind, it defaults to ``SITE``. If a user explicitly changes the kind to ``PROJECT`` (or another specific kind), it is treated as a "marker" kind. The location endpoint then works off that particular marker kind.

However, this transition can lead to loss of context or missing metadata. For example:
1. A location is stored as a ``SITE``.
2. A stream location is then stored for that ``SITE`` location.
3. If the location is subsequently changed to a ``PROJECT``, it becomes a "marker project" but "forgets" that it was also a stream location.

The location endpoint currently lacks the ability to denote all kinds that a single location might embody, leading to incomplete metadata representation.


Proposed Goals and Behavioral Expectations
=========================================

1. **Location Endpoint as Marker**: The general location endpoint is responsible for establishing the primary identity and "KIND" marker for a location.
2. **Kind-Specific Endpoint Responsibility**: Metadata and database rows specific to a KIND (like stream or project details) are the responsibility of their respective specialized endpoints.
3. **Respect Explicit Kind**: Ensure that if a KIND is explicitly set during storage at the location endpoint, it is preserved and respected as the primary marker.
4. **Handle Marker Kinds**: Recognize and correctly handle "marker" kinds (like ``PROJECT``) without losing underlying location identity (like being a stream location).
5. **Identify Incomplete Metadata**: Provide a mechanism to identify which locations do not have all their expected metadata filled out.
6. **Metadata Density**: It is acceptable for some metadata to be missing or stubbed initially, provided there is a way to denote the state of the location's metadata.


Key Considerations
==================

.. list-table::
   :header-rows: 1
   :widths: 20 25 55

   * - Topic
     - Decision/Observation
     - Justification
   * - Default Kind
     - Defaults to ``SITE`` if unassigned.
     - Maintains backwards compatibility and provides a sane default for physical locations.
   * - Marker Kinds
     - Kinds like ``PROJECT`` act as markers that certain endpoints (like the location endpoint) use for filtering.
     - Allows specialized views of locations based on their primary functional role.
   * - Kind Multiplicity
     - Locations may need to support multiple kinds or at least retain metadata associated with previous kinds.
     - Prevents "identity loss" when a location's primary marker is updated.
   * - View Inclusion
     - Should marker-only rows in tables like ``at_project`` be visible in general views?
     - Open question: Depending on use case, we may or may not want to see these markers in general av_project views.
   * - Stubbing
     - Stubbing missing info is an acceptable short-term solution.
     - Allows the API to remain functional while identifying where more detailed metadata is required.
   * - Endpoint Responsibility
     - Location endpoints establish the KIND as a marker. Kind-specific endpoints (e.g., Stream, Project) handle detailed metadata.
     - Clearly separates general location identity from specialized metadata management.


Example Scenario
~~~~~~~~~~~~~~~~

* **Initial State**: Location ``LOC123`` is created as a ``SITE``.
* **Step 2**: ``LOC123`` is defined as a stream location (associating it with stream-specific metadata).
* **Step 3**: ``LOC123`` is updated to have a ``PROJECT`` kind.
* **Result**: The system now sees ``LOC123`` primarily as a Project marker, but the stream location metadata may be inaccessible or unassociated through standard location queries.


Decision Status
===============

(Status: proposed)


References
==========

Related Types: ``cwms.cda.data.dto.Location``, ``cwms.cda.data.dto.CwmsIdLocationKind``, ``cwms.cda.data.dto.stream.StreamLocation``
Issue/Discussion: [Internal Issue Tracking]
