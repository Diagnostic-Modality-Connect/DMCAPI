# FHIR Mapping Guide
## Diagnostic Modality Connect API (DMCAPI)

**Version:** 1.0  
**FHIR Version:** R4 (4.0.1)  
**Date:** December 1, 2025

---

## Overview

This guide provides practical examples and mappings between DMCAPI resources and FHIR R4 resources. It demonstrates how to use the API in a FHIR-compliant manner while maintaining backward compatibility with existing implementations.

---

## Table of Contents

1. [Patient (FHIR Patient with Veterinary Extensions)](#patient)
2. [Order (FHIR ServiceRequest)](#order-servicerequest)
3. [Report (FHIR DiagnosticReport)](#report-diagnosticreport)
4. [Observation (FHIR Observation)](#observation)
5. [Veterinarian (FHIR Practitioner)](#veterinarian-practitioner)
6. [Client (FHIR RelatedPerson)](#client-relatedperson)
7. [Status Code Mappings](#status-code-mappings)
8. [Common FHIR Patterns](#common-fhir-patterns)

---

## Patient

### FHIR Resource Mapping

**DMCAPI Resource:** `Patient`  
**FHIR Resource:** `Patient` (with patient-animal extension)  
**FHIR Profile:** http://dmcapi.org/fhir/StructureDefinition/veterinary-patient

### Minimal FHIR-Compliant Example

```json
{
  "resourceType": "Patient",
  "id": "74c7cac2-0bd5-4e56-b114-f088a502dc6a",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2024-01-15T10:30:00Z",
    "profile": [
      "http://dmcapi.org/fhir/StructureDefinition/veterinary-patient"
    ]
  },
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/patient-animal",
      "extension": [
        {
          "url": "species",
          "valueCodeableConcept": {
            "coding": [
              {
                "system": "http://hl7.org/fhir/sid/ncbi-taxonomy",
                "code": "9615",
                "display": "Canis lupus familiaris"
              }
            ],
            "text": "Dog"
          }
        },
        {
          "url": "breed",
          "valueCodeableConcept": {
            "coding": [
              {
                "system": "http://dmcapi.org/fhir/CodeSystem/animal-breeds",
                "code": "JACK_RUSSEL_TERRIER",
                "display": "Jack Russell Terrier"
              }
            ]
          }
        },
        {
          "url": "genderStatus",
          "valueCodeableConcept": {
            "coding": [
              {
                "system": "http://hl7.org/fhir/animal-genderstatus",
                "code": "neutered",
                "display": "Neutered"
              }
            ]
          }
        }
      ]
    }
  ],
  "identifier": [
    {
      "system": "http://myvetsystem.com/patient-id",
      "value": "PET-12345"
    }
  ],
  "name": "Laika",
  "birthDate": "2018-05-14"
}
```

### Backward-Compatible Example

For backward compatibility, the API also accepts the simplified format:

```json
{
  "resourceType": "Patient",
  "id": "74c7cac2-0bd5-4e56-b114-f088a502dc6a",
  "name": "Laika",
  "species": "CANIS_LUPUS_FAMILIARIS",
  "breed": "JACK_RUSSEL_TERRIER",
  "sex": "MALE_NEUTERED",
  "birthDate": "2018-05-14",
  "weight": {
    "value": 10.5,
    "unit": "kg",
    "system": "http://unitsofmeasure.org",
    "code": "kg"
  }
}
```

### Field Mappings

| DMCAPI Field | FHIR Field | Notes |
|--------------|------------|-------|
| resourceType | resourceType | Must be "Patient" |
| id | id | UUID identifier |
| name | name[0].text or just name (simplified) | In full FHIR, use HumanName array |
| species | extension[patient-animal].species | Use NCBI Taxonomy codes |
| breed | extension[patient-animal].breed | Custom code system |
| sex | extension[patient-animal].genderStatus | FHIR genderStatus value set |
| birthDate | birthDate | FHIR date format (YYYY-MM-DD) |
| weight | extension or Observation | Weight typically as separate Observation |
| identifier | identifier[] | FHIR Identifier array |

---

## Order (ServiceRequest)

### FHIR Resource Mapping

**DMCAPI Resource:** `Order`  
**FHIR Resource:** `ServiceRequest`  
**FHIR Profile:** http://dmcapi.org/fhir/StructureDefinition/veterinary-diagnostic-order

### Minimal FHIR-Compliant Example

```json
{
  "resourceType": "ServiceRequest",
  "id": "74c7cac2-0bd5-4e56-b114-f088a502dc6a",
  "status": "active",
  "intent": "order",
  "category": [
    {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "108252007",
          "display": "Laboratory procedure"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "58410-2",
        "display": "Complete blood count (hemogram) panel"
      }
    ],
    "text": "Complete Blood Count"
  },
  "subject": {
    "reference": "Patient/74c7cac2-0bd5-4e56-b114-f088a502dc6a",
    "display": "Laika"
  },
  "authoredOn": "2024-01-15T10:00:00Z",
  "requester": {
    "reference": "Practitioner/5cd4983d-be3b-4bfc-a8a2-4371393f26c8",
    "display": "Dr. John Doe, DVM"
  },
  "note": [
    {
      "text": "Patient presented with lethargy and decreased appetite"
    }
  ]
}
```

### Status Code Mapping

| Legacy Status | FHIR Status | Description |
|--------------|-------------|-------------|
| ACCEPTED | draft | Order created, not yet submitted |
| WAITING_FOR_INPUT | draft | Awaiting additional information |
| SUBMITTED | active | Order submitted and active |
| PARTIAL | active | Order active, some results available |
| COMPLETED | completed | All tests completed |
| CANCELLED | cancelled | Order cancelled |
| ERROR | entered-in-error | Order entered in error |

### Field Mappings

| DMCAPI Field | FHIR Field | Notes |
|--------------|------------|-------|
| resourceType | resourceType | Must be "ServiceRequest" |
| id | id | UUID identifier |
| status | status | See status mapping above |
| intent | intent | Required: typically "order" |
| patient | subject | FHIR Reference to Patient |
| veterinarian | requester | FHIR Reference to Practitioner |
| testCodes[] | code.coding[] or orderDetail[] | Array of test codes |
| notes | note[].text | FHIR Annotation array |
| requisitionId | identifier[] | Add to identifier array with appropriate system |
| accessionId | identifier[] | Provider identifier |
| integrationId | extension | Custom extension |
| devices | extension | Custom veterinary extension |

---

## Report (DiagnosticReport)

### FHIR Resource Mapping

**DMCAPI Resource:** `Report`  
**FHIR Resource:** `DiagnosticReport`  
**FHIR Profile:** http://dmcapi.org/fhir/StructureDefinition/veterinary-diagnostic-report

### Minimal FHIR-Compliant Example

```json
{
  "resourceType": "DiagnosticReport",
  "id": "3025460e-7606-4643-a1ff-47c732e8ab20",
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/v2-0074",
          "code": "LAB",
          "display": "Laboratory"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "58410-2",
        "display": "Complete blood count (hemogram) panel"
      }
    ],
    "text": "Hematology Panel"
  },
  "subject": {
    "reference": "Patient/74c7cac2-0bd5-4e56-b114-f088a502dc6a",
    "display": "Laika"
  },
  "basedOn": [
    {
      "reference": "ServiceRequest/74c7cac2-0bd5-4e56-b114-f088a502dc6a"
    }
  ],
  "effectiveDateTime": "2024-01-15T09:00:00Z",
  "issued": "2024-01-15T14:30:00Z",
  "result": [
    {
      "reference": "Observation/obs-hem-001",
      "display": "Hemoglobin"
    },
    {
      "reference": "Observation/obs-hem-002",
      "display": "Hematocrit"
    }
  ],
  "presentedForm": [
    {
      "contentType": "application/pdf",
      "url": "https://example.com/reports/report-123.pdf",
      "title": "Hematology Report - Laika"
    }
  ]
}
```

### Status Code Mapping

| Legacy Status | FHIR Status | Description |
|--------------|-------------|-------------|
| REGISTERED | registered | Report registered, no results yet |
| PARTIAL | partial | Some results available |
| FINAL | final | Report complete and verified |
| CANCELLED | cancelled | Report cancelled |

### Field Mappings

| DMCAPI Field | FHIR Field | Notes |
|--------------|------------|-------|
| resourceType | resourceType | Must be "DiagnosticReport" |
| id | id | UUID identifier |
| status | status | See status mapping |
| code | code | Required: type of report |
| subject | subject | FHIR Reference to Patient |
| orderId | basedOn[].reference | Reference to ServiceRequest |
| testResultsSet[] | result[] | References to Observation resources |
| presentedForm[] | presentedForm[] | FHIR Attachment array |
| createdAt | effectiveDateTime | When specimen collected |
| updatedAt | issued | When report made available |

---

## Observation

### FHIR Resource Mapping

**DMCAPI Resource:** `Observation`  
**FHIR Resource:** `Observation`  
**FHIR Profile:** http://dmcapi.org/fhir/StructureDefinition/veterinary-observation

### Minimal FHIR-Compliant Example

```json
{
  "resourceType": "Observation",
  "id": "obs-hem-001",
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "laboratory",
          "display": "Laboratory"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "718-7",
        "display": "Hemoglobin [Mass/volume] in Blood"
      }
    ],
    "text": "Hemoglobin"
  },
  "subject": {
    "reference": "Patient/74c7cac2-0bd5-4e56-b114-f088a502dc6a"
  },
  "effectiveDateTime": "2024-01-15T09:00:00Z",
  "issued": "2024-01-15T14:30:00Z",
  "valueQuantity": {
    "value": 16.6,
    "unit": "g/dL",
    "system": "http://unitsofmeasure.org",
    "code": "g/dL"
  },
  "interpretation": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/v3-ObservationInterpretation",
          "code": "N",
          "display": "Normal"
        }
      ]
    }
  ],
  "referenceRange": [
    {
      "low": {
        "value": 13.4,
        "unit": "g/dL",
        "system": "http://unitsofmeasure.org",
        "code": "g/dL"
      },
      "high": {
        "value": 20.7,
        "unit": "g/dL",
        "system": "http://unitsofmeasure.org",
        "code": "g/dL"
      },
      "type": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/referencerange-meaning",
            "code": "normal",
            "display": "Normal Range"
          }
        ]
      }
    }
  ]
}
```

### Status Code Mapping

| Legacy Status | FHIR Status | Description |
|--------------|-------------|-------------|
| PENDING | registered | Test registered, not performed |
| DONE | final | Test completed and verified |
| CANCELLED | cancelled | Test cancelled |

### Interpretation Code Mapping

| DMCAPI Code | FHIR Code | Display |
|-------------|-----------|---------|
| N | N | Normal |
| A | A | Abnormal |
| H | H | High |
| HH | HH | Critical high |
| L | L | Low |
| LL | LL | Critical low |

### Field Mappings

| DMCAPI Field | FHIR Field | Notes |
|--------------|------------|-------|
| resourceType | resourceType | Must be "Observation" |
| id | id | UUID identifier |
| status | status | See status mapping |
| code | code | FHIR CodeableConcept with LOINC preferred |
| subject | subject | Reference to Patient (required) |
| valueQuantity | valueQuantity | FHIR Quantity with UCUM units |
| valueString | valueString | For text results |
| interpretation | interpretation[] | FHIR CodeableConcept array |
| referenceRange[] | referenceRange[] | FHIR ReferenceRange array |
| notes | note[].text | FHIR Annotation array |

---

## Veterinarian (Practitioner)

### FHIR Resource Mapping

**DMCAPI Resource:** `Veterinarian`  
**FHIR Resource:** `Practitioner`

### Example

```json
{
  "resourceType": "Practitioner",
  "id": "5cd4983d-be3b-4bfc-a8a2-4371393f26c8",
  "active": true,
  "name": [
    {
      "use": "official",
      "family": "Doe",
      "given": ["John"],
      "prefix": ["Dr."],
      "suffix": ["DVM"],
      "text": "Dr. John Doe, DVM"
    }
  ],
  "telecom": [
    {
      "system": "phone",
      "value": "+1-555-555-5555",
      "use": "work"
    },
    {
      "system": "email",
      "value": "john.doe@vetclinic.com",
      "use": "work"
    }
  ],
  "qualification": [
    {
      "code": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/v2-0360",
            "code": "DVM",
            "display": "Doctor of Veterinary Medicine"
          }
        ],
        "text": "Doctor of Veterinary Medicine"
      },
      "issuer": {
        "display": "State Veterinary Board"
      }
    }
  ]
}
```

### Field Mappings

| DMCAPI Field | FHIR Field | Notes |
|--------------|------------|-------|
| firstName | name[].given[] | Array of given names |
| lastName | name[].family | Family/last name |
| contact.email | telecom[system=email] | Email telecom entry |
| contact.phone | telecom[system=phone] | Phone telecom entry |

---

## Client (RelatedPerson)

### FHIR Resource Mapping

**DMCAPI Resource:** `Client`  
**FHIR Resource:** `RelatedPerson`

### Example

```json
{
  "resourceType": "RelatedPerson",
  "id": "11bd1e6c-2a94-418c-8c2a-3b059f1cb138",
  "active": true,
  "patient": {
    "reference": "Patient/74c7cac2-0bd5-4e56-b114-f088a502dc6a",
    "display": "Laika"
  },
  "relationship": [
    {
      "coding": [
        {
          "system": "http://dmcapi.org/fhir/CodeSystem/veterinary-relationships",
          "code": "OWNER",
          "display": "Pet Owner"
        }
      ],
      "text": "Owner"
    }
  ],
  "name": [
    {
      "use": "official",
      "family": "Smith",
      "given": ["John"],
      "text": "John Smith"
    }
  ],
  "telecom": [
    {
      "system": "phone",
      "value": "+1-555-123-4567",
      "use": "mobile"
    },
    {
      "system": "email",
      "value": "john.smith@email.com"
    }
  ],
  "address": [
    {
      "use": "home",
      "line": ["123 Main Street"],
      "city": "Springfield",
      "postalCode": "12345",
      "country": "US"
    }
  ]
}
```

---

## Status Code Mappings

### Complete Status Code Mapping Table

| Resource | Legacy Status | FHIR Status | Notes |
|----------|--------------|-------------|-------|
| **Patient** | N/A | N/A | No status field |
| **ServiceRequest** | ACCEPTED | draft | Order created |
| | WAITING_FOR_INPUT | draft | Awaiting input |
| | SUBMITTED | active | Order active |
| | PARTIAL | active | Partial results |
| | COMPLETED | completed | All done |
| | CANCELLED | cancelled | Cancelled |
| | ERROR | entered-in-error | Error state |
| **DiagnosticReport** | REGISTERED | registered | Registered |
| | PARTIAL | partial | Partial results |
| | FINAL | final | Final report |
| | CANCELLED | cancelled | Cancelled |
| **Observation** | PENDING | registered | Not performed |
| | DONE | final | Completed |
| | CANCELLED | cancelled | Cancelled |

---

## Common FHIR Patterns

### 1. FHIR Reference Pattern

FHIR uses References to link resources:

```json
{
  "reference": "Patient/74c7cac2-0bd5-4e56-b114-f088a502dc6a",
  "display": "Laika"
}
```

### 2. FHIR CodeableConcept Pattern

For coded values:

```json
{
  "coding": [
    {
      "system": "http://loinc.org",
      "code": "718-7",
      "display": "Hemoglobin [Mass/volume] in Blood"
    }
  ],
  "text": "Hemoglobin"
}
```

### 3. FHIR Quantity Pattern

For measurements with units:

```json
{
  "value": 10.5,
  "unit": "kg",
  "system": "http://unitsofmeasure.org",
  "code": "kg"
}
```

### 4. FHIR Identifier Pattern

For business identifiers:

```json
{
  "system": "http://myvetsystem.com/patient-id",
  "value": "PET-12345"
}
```

### 5. FHIR HumanName Pattern

For person names:

```json
{
  "use": "official",
  "family": "Doe",
  "given": ["John", "Robert"],
  "prefix": ["Dr."],
  "suffix": ["DVM"],
  "text": "Dr. John Robert Doe, DVM"
}
```

---

## Veterinary-Specific Extensions

### Patient Animal Extension

**URL:** `http://hl7.org/fhir/StructureDefinition/patient-animal`

**Sub-extensions:**
- `species` (CodeableConcept) - Animal species
- `breed` (CodeableConcept) - Animal breed
- `genderStatus` (CodeableConcept) - Neutered/spayed status

### Custom DMCAPI Extensions

**Namespace:** `http://dmcapi.org/fhir/StructureDefinition/`

1. **animal-weight** - Current weight of the animal
2. **lab-requisition-info** - Provider-specific requisition parameters
3. **provider-device** - Specific diagnostic device information
4. **integration-id** - DMCAPI integration identifier

---

## Code Systems and Value Sets

### FHIR Standard Code Systems

- **LOINC:** http://loinc.org - Laboratory observation codes
- **SNOMED CT:** http://snomed.info/sct - Clinical terminology
- **UCUM:** http://unitsofmeasure.org - Units of measure
- **NCBI Taxonomy:** http://hl7.org/fhir/sid/ncbi-taxonomy - Species codes

### Custom DMCAPI Code Systems

- **Animal Breeds:** http://dmcapi.org/fhir/CodeSystem/animal-breeds
- **Animal Sex:** http://dmcapi.org/fhir/CodeSystem/animal-sex
- **Veterinary Relationships:** http://dmcapi.org/fhir/CodeSystem/veterinary-relationships

---

## Best Practices

### 1. Always Include resourceType

```json
{
  "resourceType": "Patient",
  ...
}
```

### 2. Use FHIR References for Relationships

Instead of embedding full resources, use references:

```json
{
  "subject": {
    "reference": "Patient/123",
    "display": "Laika"
  }
}
```

### 3. Use Standard Terminologies

Prefer LOINC for lab codes, UCUM for units, SNOMED for clinical concepts.

### 4. Maintain Backward Compatibility

The API supports both FHIR-compliant and simplified formats. When in doubt, include both:

```json
{
  "resourceType": "Patient",
  "name": "Laika",  // Simplified
  "species": "CANIS_LUPUS_FAMILIARIS",  // Simplified
  "extension": [...]  // FHIR extension
}
```

### 5. Validate with FHIR Validators

Use FHIR validation tools to ensure conformance:
- https://validator.fhir.org/
- HAPI FHIR Validator

---

## Migration Guide

### For Existing Implementations

1. **Add resourceType to all requests:** Minimal change, maximum compatibility
2. **Map legacy status codes:** Use mapping tables provided
3. **Update identifiers:** Convert single fields to identifier arrays where appropriate
4. **Add FHIR references:** Gradually introduce Reference pattern for relationships
5. **Implement extensions:** Add FHIR extensions for veterinary-specific data

### Example Migration Path

**Phase 1: Minimal Updates**
```json
{
  "resourceType": "Patient",  // ADD THIS
  "id": "123",
  "name": "Laika",
  "species": "CANIS_LUPUS_FAMILIARIS"
}
```

**Phase 2: Add FHIR Metadata**
```json
{
  "resourceType": "Patient",
  "id": "123",
  "meta": {  // ADD THIS
    "lastUpdated": "2024-01-15T10:00:00Z"
  },
  "name": "Laika",
  "species": "CANIS_LUPUS_FAMILIARIS"
}
```

**Phase 3: Full FHIR Compliance**
```json
{
  "resourceType": "Patient",
  "id": "123",
  "meta": {
    "lastUpdated": "2024-01-15T10:00:00Z",
    "profile": ["http://dmcapi.org/fhir/StructureDefinition/veterinary-patient"]
  },
  "extension": [{  // ADD THIS
    "url": "http://hl7.org/fhir/StructureDefinition/patient-animal",
    "extension": [...]
  }],
  "name": "Laika",
  "species": "CANIS_LUPUS_FAMILIARIS"  // Keep for compatibility
}
```

---

## Additional Resources

- **FHIR R4 Specification:** http://hl7.org/fhir/R4/
- **FHIR Patient Animal Extension:** http://hl7.org/fhir/StructureDefinition/patient-animal
- **Veterinary FHIR Implementation Guide:** http://hl7.org/fhir/us/vetconnect/
- **LOINC Database:** https://loinc.org/
- **UCUM Units:** http://unitsofmeasure.org/
- **FHIR Validator:** https://validator.fhir.org/

---

**Document Version:** 1.0  
**Last Updated:** December 1, 2025  
**Maintained By:** DMCAPI Development Team
