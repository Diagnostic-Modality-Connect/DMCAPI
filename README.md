
Diagnostic Modality Connect API (DMCAPI)
========================================

The Diagnostic Modality Connect API (DMCAPI) is a focused subset of the Diagnostic Modality Integrator (DMI) project, designed to be directly consumed by healthcare providers and practices. This API provides endpoints and data structures to:

 
 *   Interrogate diagnostic services
 *   Create and manage diagnostic orders
 *   Retrieve diagnostic results
 *   Manage event subscriptions related to diagnostics

 View the documentation at https://diagnostic-modality-connect.github.io/DMCAPI/

FHIR R4 Compatibility
--------------------

**The DMCAPI is now aligned with HL7 FHIR R4 standards** for veterinary diagnostics, providing:

*   **FHIR-compliant resource structures** - Patient, ServiceRequest, DiagnosticReport, Observation, Practitioner, and RelatedPerson
*   **Veterinary extensions** - Support for species, breed, and gender status using standard FHIR patient-animal extension
*   **Backward compatibility** - Existing implementations continue to work while enabling FHIR adoption
*   **Minimal field requirements** - Focused on essential veterinary diagnostic workflow
*   **Standard terminologies** - Support for LOINC, SNOMED CT, UCUM, and NCBI Taxonomy codes

📄 **[FHIR Compatibility Report](FHIR_COMPATIBILITY_REPORT.md)** - Comprehensive evaluation and recommendations  
📘 **[FHIR Mapping Guide](FHIR_MAPPING_GUIDE.md)** - Practical examples and field mappings

Purpose
-------

The DMCAPI exists to streamline the integration of diagnostic modalities within and between healthcare IT systems. By offering a simplified interface that includes only the essential endpoints, the DMCAPI ensures that developers can quickly and efficiently incorporate diagnostic functionalities into their applications without the overhead of unrelated features.
 

Target Audience
---------------

This API is intended for:
 
 *   Veterinary healthcare providers
 *   Veterinary medical practices
 *   Veterinary PIMS (Practice Information Management Systems) developers
 *   Healthcare IT developers integrating veterinary diagnostics
 

Why Use DMCAPI?
---------------
 
*   **FHIR Compliance**: Aligned with HL7 FHIR R4 standards for maximum interoperability
*   **Veterinary-Optimized**: Purpose-built for veterinary diagnostic workflows with appropriate extensions
*   **Compatibility**: Highly compatible with the DMI, ensuring seamless integration with existing systems
*   **Simplicity**: Focused endpoints make it easier to implement and maintain and encourage adoption by healthcare providers and software developers
*   **Efficiency**: Directly addresses the needs of healthcare providers by including only the most relevant functionalities
*   **Open Source**: Benefit from an open-source project with community support and contributions
*   **Standardization**: Adopting a standard API reduces the need to write custom connections between different vendors, promoting consistency and interoperability across systems
*   **Permissive Licensing**: Both the DMI and this subset of DMI are permissively licensed, making them easy to adopt and integrate into practice software and diagnostic modalities

