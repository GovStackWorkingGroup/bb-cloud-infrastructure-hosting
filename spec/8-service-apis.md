---
description: >-
  This section provides a reference for APIs that should be implemented by this
  Building Block.
---

# 8 Service APIs

The same comment as in section 7 applies. We refrain from defining Meta-APIs here, but may agree to do so at a later stage. The implementations should use standard APIs that are well supported by IaC tooling.

## 8.1. API for Provisioning Virtual Machines <a href="#id-81-api-for-provisioning-virtual-machines" id="id-81-api-for-provisioning-virtual-machines"></a>

Interface for creating, managing, and deallocating virtual machines on the cloud infrastructure.

The API is a REST interface; a resource gets created by and authorized `POST` request to the relevant endpoint (as listed in the catalogue) including the JSON data structure describing the resource properties. The methods `GET` `DELETE`, `PUT`, `PATCH` are supported to list or retrieve details, to destroy a resource and to apply changes to an existing ressource. The normal http response codes indicate whether the request was successful. The authentication happens via a header with the authorization token or via a client cert.

Note that resource creation can be a long-running process. If the request was legal, a success code (200) is returned and a message with the resource UUID, but polling for the status of the resource would be needed to ensure it’s ready before relying on it’s readiness. Details are implementation dependent, a defensive automation apporach would always check on the state of a resource.

The interface is an imperative one – the API call requests the platform to do X. The platform attempts to do so, and if there is a failure, it will be reported. The resource creation may succeed and then later enter into a failure state. This won’t be automatically fixed either but can be seen from the state.

Orchestration services with a declarative interface that describe the wanted target state that the orchestrator then tries to create are possible.

## 8.2. API for Managing Storage Volumes <a href="#id-82-api-for-managing-storage-volumes" id="id-82-api-for-managing-storage-volumes"></a>

Endpoints for creating, attaching, resizing, and deleting storage volumes used by virtual machines and containers.

The same considerations apply as in 8.1.

## 8.3. API for Network Configuration <a href="#id-83-api-for-network-configuration" id="id-83-api-for-network-configuration"></a>

Services for setting up, modifying, and tearing down network configurations, including virtual networks, subnets, and security groups.

The same considerations as in 8.1 apply.

## 8.4. API for Container Orchestration <a href="#id-84-api-for-container-orchestration" id="id-84-api-for-container-orchestration"></a>

Interfaces for deploying, scaling, and managing containerized applications, supporting operations like starting, stopping, and monitoring containers.

The REST API submits hierarchical data structures (typically as YAML) that describe a wanted state, like “there should be two replicas of this pod in the cluster”. (A pod is a set of containers that belong together and are always scheduled together.) The reconciliation loop of the container orchestration software is supposed to ensure that those two replicas get started. If a node dies, it will soon take care of starting another replica to ensure we (almost) always have the desired state.

The data structures are defined by the implementation and allows for being extended via schema for custom resources.

## 8.5. API for IAM Management <a href="#id-85-api-for-iam-management" id="id-85-api-for-iam-management"></a>

Endpoints for managing user identities, roles, permissions, and authentication policies.

Both the virtualization layer and the container orchestration layer allow for defining users, groups, and roles locally. For allowing federated infrastructure, it is desired, however, to use federated identities and configure the users in the IAM component or the customer’s own Identity Provider.

For the federation, OpenID Connect should be supported; the protocol and data structures are described by the [OpenID Connect specs](https://openid.net/wg/connect/specifications/) from the OpenID Foundation’s relevant working group.

## 8.6. API for Monitoring and Logging <a href="#id-86-api-for-monitoring-and-logging" id="id-86-api-for-monitoring-and-logging"></a>

Services for collecting, querying, and analyzing logs and monitoring data from cloud resources and applications.

Implementation specific.

## 8.7. API for Status Page <a href="#id-87-api-for-status-page" id="id-87-api-for-status-page"></a>

Interface for retrieving the current status and health metrics of various cloud services and components, enabling real-time monitoring and transparency.

Infrastructure providers should publish the status of their services to the users, allowing them to determine whether service limitations are happening in the provider’s realm. Providers use an API to publish their status that is then displayed on a web page accessible to least all platform users.

An example API specification would look like this:

```
openapi: "3.0.2"
info:
  title: Status Page API
  version: "1.1.2"
tags:
  - name: phase
    description: Handles incident phases.
  - name: impact
    description: Handles impacts.
  - name: component
    description: Handles components.
  - name: incident
    description: Handles incidents.

servers:
  - url: https://status-api.DOMAIN
    description: DOMAIN public default server.
  - url: http://localhost:3000
    description: Local development server.

components:
  schemas:
    # common definitions
    Id:
      description: Identification for objects. UUID preferred.
      type: string
      format: uuid

    IdField:
      description: Id field for responses.
      type: object
      properties:
        id:
          $ref: "#/components/schemas/Id"
      required:
        - id

    Incremental:
      description: |
        Positive and incrementing number for ordering and identfication of e.g. sub resources.
      type: integer
      minimum: 0

    Generation:
      description: Incremental as generation field for responses.
      type: object
      properties:
        generation:
          $ref: "#/components/schemas/Incremental"
      required:
        - generation

    Order:
      description: Incremental as order field for responses.
      type: object
      properties:
        order:
          $ref: "#/components/schemas/Incremental"
      required:
        - order

    IncrementalList:
      description: List of Incrementals for referencing subresources.
      type: array
      items:
        $ref: "#/components/schemas/Incremental"
      readOnly: true

    DisplayName:
      description: Short and describing name.
      type: string

    Description:
      description: A longer text with detailed information.
      type: string

    Date:
      description: Point in time. Nullable for open end timeframes.
      type: string
      format: date-time
      nullable: true

    SeverityValue:
      description: |
        The severity of an impact affecting a component. Different impact types
        might have different severities impacting different components.
      type: integer
      minimum: 0
      maximum: 100

    # API objects
    ImpactType:
      type: object
      properties:
        displayName:
          $ref: "#/components/schemas/DisplayName"
        description:
          $ref: "#/components/schemas/Description"

    Impact:
      description: An impact connects a component and an incident with an impact type.
      type: object
      properties:
        type:
          $ref: "#/components/schemas/Id"
        reference:
          $ref: "#/components/schemas/Id"
        severity:
          $ref: "#/components/schemas/SeverityValue"

    Severity:
      description: A severity has an identifying name and a numeric value.
      type: object
      properties:
        displayName:
          $ref: "#/components/schemas/DisplayName"
        value:
          $ref: "#/components/schemas/SeverityValue"

    ImpactComponentList:
      description: |
        A list of impacts for an incident.
        Impacts reference components.
      type: array
      items:
        $ref: "#/components/schemas/Impact"

    ImpactIncidentList:
      description: |
        A list of impacts for a component.
        Impacts reference incidents.
      type: array
      items:
        $ref: "#/components/schemas/Impact"
      readOnly: true

    Phase:
      description: |
        A single phase is just its name.
        It can be referenced by its generation and order.
        See: #/components/schemas/PhaseReference
      type: string

    PhaseReference:
      description: To reference a phase, its generation and order is needed.
      type: object
      allOf:
        - $ref: "#/components/schemas/Order"
        - $ref: "#/components/schemas/Generation"

    PhaseList:
      description: Phase resources are always handled as a list.
      type: object
      properties:
        phases:
          type: array
          items:
            $ref: "#/components/schemas/Phase"
      required:
        - phases

    IncidentUpdate:
      description: |
        An update is a sub resource to an incident.
        It's identified by the incident ID and its own order.
        Updates happen in a given order.
      type: object
      properties:
        displayName:
          $ref: "#/components/schemas/DisplayName"
        description:
          $ref: "#/components/schemas/Description"
        createdAt:
          $ref: "#/components/schemas/Date"
      required:
        - order

    Labels:
      description: Labels are free text key value pairs for components.
      type: object
      additionalProperties:
        type: string

    Component:
      type: object
      properties:
        displayName:
          $ref: "#/components/schemas/DisplayName"
        labels:
          $ref: "#/components/schemas/Labels"
        activelyAffectedBy:
          $ref: "#/components/schemas/ImpactIncidentList"

    Incident:
      type: object
      properties:
        displayName:
          $ref: "#/components/schemas/DisplayName"
        description:
          $ref: "#/components/schemas/Description"
        updates:
          $ref: "#/components/schemas/IncrementalList"
        affects:
          $ref: "#/components/schemas/ImpactComponentList"
        beganAt:
          $ref: "#/components/schemas/Date"
        endedAt:
          $ref: "#/components/schemas/Date"
        phase:
          $ref: "#/components/schemas/PhaseReference"
      required:
        - id

    # response data
    PhaseListResponseData:
      type: object
      allOf:
        - $ref: "#/components/schemas/Generation"
        - $ref: "#/components/schemas/PhaseList"

    ImpactTypeResponseData:
      type: object
      allOf:
        - $ref: "#/components/schemas/IdField"
        - $ref: "#/components/schemas/ImpactType"

    ComponentResponseData:
      type: object
      allOf:
        - $ref: "#/components/schemas/IdField"
        - $ref: "#/components/schemas/Component"

    IncidentResponseData:
      type: object
      allOf:
        - $ref: "#/components/schemas/IdField"
        - $ref: "#/components/schemas/Incident"

    IncidentUpdateResponseData:
      type: object
      allOf:
        - $ref: "#/components/schemas/Order"
        - $ref: "#/components/schemas/IncidentUpdate"

  # parameter
  parameters:
    GenerationQueryParameter:
      description: Optional generation in query. E.g. ?generation=7
      name: generation
      in: query
      schema:
        $ref: "#/components/schemas/Incremental"

    ComponentIdPathParameter:
      description: Component ID is required in path.
      name: componentId
      in: path
      required: true
      schema:
        $ref: "#/components/schemas/Id"

    IncidentIdPathParameter:
      description: Incident ID is required in path.
      name: incidentId
      in: path
      required: true
      schema:
        $ref: "#/components/schemas/Id"

    IncidentUpdateOrderPathParameter:
      description: Incident update order (inremental) is required in path.
      name: updateOrder
      in: path
      required: true
      schema:
        $ref: "#/components/schemas/Incremental"

    ImpactTypeIdPathParameter:
      description: Impact type ID is required in path.
      name: impactTypeId
      in: path
      required: true
      schema:
        $ref: "#/components/schemas/Id"

    SeverityNamePathParameter:
      description: Severity name is required in path.
      name: severityName
      in: path
      required: true
      schema:
        $ref: "#/components/schemas/DisplayName"

  # requests
  requestBodies:
    PhaseListRequest:
      description: Send a new list of phases.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/PhaseList"

    ComponentRequest:
      description: Send a component.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Component"

    IncidentRequest:
      description: Send an incident.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Incident"

    IncidentUpdateRequest:
      description: Send an incident update.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/IncidentUpdate"

    ImpactTypeRequest:
      description: Send an impact type.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ImpactType"

    SeverityRequest:
      description: Send an impact type.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Severity"

  # responses
  responses:
    IdResponse:
      description: A request returns an ID.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/IdField"

    GenerationResponse:
      description: A request returns a generation.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Generation"

    OrderResponse:
      description: A request returns an order.
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Order"

    PhaseListResponse:
      description: A request returns a phase list.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/PhaseListResponseData"
            required:
              - data

    ImpactTypeResponse:
      description: A request returns an impact type.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/ImpactTypeResponseData"
            required:
              - data

    ImpactTypeListResponse:
      description: A request returns a list of impact types.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: "#/components/schemas/ImpactTypeResponseData"
            required:
              - data

    SeverityResponse:
      description: A request returns a severities.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/Severity"
            required:
              - data

    SeverityListResponse:
      description: A request returns a list of severities.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: "#/components/schemas/Severity"
            required:
              - data

    ComponentResponse:
      description: A request returns a component.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/ComponentResponseData"
            required:
              - data

    ComponentListResponse:
      description: A request returns a list of components.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: "#/components/schemas/ComponentResponseData"
            required:
              - data

    IncidentResponse:
      description: A request returns an incident.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/IncidentResponseData"
            required:
              - data

    IncidentListResponse:
      description: A request returns a list of incidents.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: "#/components/schemas/IncidentResponseData"
            required:
              - data

    IncidentUpdateResponse:
      description: A request returns an incident update.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                $ref: "#/components/schemas/IncidentUpdateResponseData"
            required:
              - data

    IncidentUpdateListResponse:
      description: A request returns a list of incident updates.
      content:
        application/json:
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: "#/components/schemas/IncidentUpdateResponseData"
            required:
              - data

paths:
  /phases:
    get:
      summary: Get the current generation list of phases.
      operationId: getPhaseList
      tags:
        - phase
      parameters:
        - $ref: "#/components/parameters/GenerationQueryParameter"
      responses:
        "200":
          $ref: "#/components/responses/PhaseListResponse"

    post:
      summary: Create a new generation of the phase list.
      operationId: createPhaseList
      tags:
        - phase
      requestBody:
        $ref: "#/components/requestBodies/PhaseListRequest"
      responses:
        "201":
          $ref: "#/components/responses/GenerationResponse"

  /impacttypes:
    get:
      summary: Get a list of impact types.
      operationId: getImpactTypes
      tags:
        - impact
      responses:
        "200":
          $ref: "#/components/responses/ImpactTypeListResponse"

    post:
      summary: Create a new impact type.
      operationId: createImpactType
      tags:
        - impact
      requestBody:
        $ref: "#/components/requestBodies/ImpactTypeRequest"
      responses:
        "201":
          $ref: "#/components/responses/IdResponse"

  /impacttypes/{impactTypeId}:
    get:
      summary: Get a specific impact type by id.
      operationId: getImpactType
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/ImpactTypeIdPathParameter"
      responses:
        "200":
          $ref: "#/components/responses/ImpactTypeResponse"

    patch:
      summary: Update a specific impact type.
      operationId: updateImpactType
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/ImpactTypeIdPathParameter"
      requestBody:
        $ref: "#/components/requestBodies/ImpactTypeRequest"
      responses:
        "204":
          description: Successful. No Content

    delete:
      summary: Delete an impact type.
      operationId: deleteImpactType
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/ImpactTypeIdPathParameter"
      responses:
        "204":
          description: Successful. No Content

  /severities:
    get:
      summary: Get the current list of severieties.
      operationId: getSeverities
      tags:
        - impact
      responses:
        "200":
          $ref: "#/components/responses/SeverityListResponse"

    post:
      summary: Create a new severity.
      operationId: createSeverity
      tags:
        - impact
      requestBody:
        $ref: "#/components/requestBodies/SeverityRequest"
      responses:
        "204":
          description: Successful. No Content

  /severities/{severityName}:
    get:
      summary: Get a specific severity by its name.
      operationId: getSeverity
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/SeverityNamePathParameter"
      responses:
        "200":
          $ref: "#/components/responses/SeverityResponse"

    patch:
      summary: Update a specific severity.
      operationId: updateSeverity
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/SeverityNamePathParameter"
      requestBody:
        $ref: "#/components/requestBodies/SeverityRequest"
      responses:
        "204":
          description: Successful. No Content

    delete:
      summary: Delete a specific severity.
      operationId: deleteSeverity
      tags:
        - impact
      parameters:
        - $ref: "#/components/parameters/SeverityNamePathParameter"
      responses:
        "204":
          description: Successful. No Content

  /components:
    get:
      summary: Get a list of components.
      operationId: getComponents
      tags:
        - component
      responses:
        "200":
          $ref: "#/components/responses/ComponentListResponse"

    post:
      summary: Create a new component.
      operationId: createComponent
      tags:
        - component
      requestBody:
        $ref: "#/components/requestBodies/ComponentRequest"
      responses:
        "201":
          $ref: "#/components/responses/IdResponse"

  /components/{componentId}:
    get:
      summary: Get a specific component by id.
      operationId: getComponent
      tags:
        - component
      parameters:
        - $ref: "#/components/parameters/ComponentIdPathParameter"
      responses:
        "200":
          $ref: "#/components/responses/ComponentResponse"

    patch:
      summary: Update a component.
      operationId: updateComponent
      tags:
        - component
      parameters:
        - $ref: "#/components/parameters/ComponentIdPathParameter"
      requestBody:
        $ref: "#/components/requestBodies/ComponentRequest"
      responses:
        "204":
          description: Successful. No Content

    delete:
      summary: Delete a component.
      operationId: deleteComponent
      tags:
        - component
      parameters:
        - $ref: "#/components/parameters/ComponentIdPathParameter"
      responses:
        "204":
          description: Successful. No Content

  /incidents:
    get:
      summary: Get a list of incidents between two points in time.
      operationId: getIncidents
      tags:
        - incident
      parameters:
        - in: query
          name: start
          schema:
            type: string
            format: date-time
            default: "2022-01-01T10:10:10.010Z"
          required: true
          description: Start of time frame to query for (RFC3339).
        - in: query
          name: end
          schema:
            type: string
            format: date-time
            default: "2024-01-01T10:10:10.010Z"
          required: true
          description: End of time frame to query for (RFC3339).
      responses:
        "200":
          $ref: "#/components/responses/IncidentListResponse"

    post:
      summary: Create a new incident.
      operationId: createIncident
      tags:
        - incident
      requestBody:
        $ref: "#/components/requestBodies/IncidentRequest"
      responses:
        "201":
          $ref: "#/components/responses/IdResponse"

  /incidents/{incidentId}:
    get:
      summary: Get a specific incident by id.
      operationId: getIncident
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
      responses:
        "200":
          $ref: "#/components/responses/IncidentResponse"

    patch:
      summary: Update an incident.
      operationId: updateIncident
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
      requestBody:
        $ref: "#/components/requestBodies/IncidentRequest"
      responses:
        "204":
          description: Successful. No Content

    delete:
      summary: Delete an incident.
      operationId: deleteIncident
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
      responses:
        "204":
          description: Successful. No Content

  /incidents/{incidentId}/updates:
    get:
      summary: Get a list of updates from a specific incident.
      operationId: getIncidentUpdates
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
      responses:
        "200":
          $ref: "#/components/responses/IncidentUpdateListResponse"

    post:
      summary: Create a new update to a specific incident.
      operationId: createIncidentUpdate
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
      requestBody:
        $ref: "#/components/requestBodies/IncidentUpdateRequest"
      responses:
        "201":
          $ref: "#/components/responses/OrderResponse"

  /incidents/{incidentId}/updates/{updateOrder}:
    get:
      summary: Get a specific update from a specific incident.
      operationId: getIncidentUpdate
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
        - $ref: "#/components/parameters/IncidentUpdateOrderPathParameter"
      responses:
        "200":
          $ref: "#/components/responses/IncidentUpdateResponse"

    patch:
      summary: Update a specific update from a specific incident.
      operationId: updateIncidentUpdate
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
        - $ref: "#/components/parameters/IncidentUpdateOrderPathParameter"
      requestBody:
        $ref: "#/components/requestBodies/IncidentUpdateRequest"
      responses:
        "204":
          description: Successful. No Content

    delete:
      summary: Delete a specific update from a specific incident
      operationId: deleteIncidentUpdate
      tags:
        - incident
      parameters:
        - $ref: "#/components/parameters/IncidentIdPathParameter"
        - $ref: "#/components/parameters/IncidentUpdateOrderPathParameter"
      responses:
        "204":
          description: Successful. No Content
```
