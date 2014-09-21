[原文链接:FHIR Connectathon 7](http://wiki.hl7.org/index.php?title=FHIR_Connectathon_7/)       
## Connectathon tracks

This section lists the scenarios that are proposed for this connectathon. More detail will be added when approved.  The scenarios are grouped according to two tracks.  Track 1 is for those new to FHIR and requires minimal preparation in advance of the connectathon (at least for client applications).  Track 2 is for those with some experience with the use of FHIR (or willing to devote up-front time to connectathon preparation) and exercises a more complete set of behavior designed to reflect a full production experience.

###  Track 1 - Patient 

If creating a client, this track should require minimal work in advance of the connectathon, though at least a bit of playing is recommended.  If creating a server, advanced preparation will be required, but this scenario should somewhat limit the effort involved.

Pre-requisites: none

####  1. Register a new patient 

*   Action: (Patient Demographics consumer) creates a new patient and save to Patient Service. The client can assign the Id.

*   Precondition: Patient does not exist in service prior to action

*   Success Criteria: Patient created correctly on server (use browser to inspect Patient)

*   Bonus point: The Patient resource has an extension

&gt;&gt;Note: the requirement for the client to assign the Id has been relaxed. However, if the server assigns the Id, then the client will need to be able to retrieve the Id from the server response or by a patient query.

####  2. Update a patient 

*   Action: (Patient Demographics consumer) updates the patient created in scenario #1 and updates to Patient Service. The patient is retrieved by Id.

*   Precondition: Patient has been created

*   Success Criteria: Patient updated on server (use browser to inspect Patient)

*   Bonus Point #1: Update a patient that has extensions, but leaving the extension untouched.

*   Bonus Point #2: Update a patient that has extensions, and update the extension also.

####  3. Retrieve Patient history 

*   Action: (Patient Demographics consumer) searches the patient Service for the history of a Patient

*   Precondition:  There is a patient that has at least one update

*   Success Criteria: patients history displayed in interface. (use browser query Patient Service)

*   Bonus point: the UI allows the user to display previous versions of the Patient

####  4. Search for a patient on name 

*   Action: (Patient Demographics consumer) searches the patient Service for patients with a given name

*   Precondition: Patients with that name have  been created

*   Success Criteria: patients displayed in interface. (use browser query to confirm)

Some help links:

*   [Java client sample](http://fhirblog.com/2014/07/31/fhir-connectathon-7-for-java-dummies/).

*   [.net client sample](http://fhirblog.com/2014/06/29/c-fhir-client/).

###  Track 2 - Profile 

This track explores interoperability between conformance tooling. There is 3 parts to this track:

*   generating profiles and valuesets

*   getting a server to test conformance

*   implementing a process (server or client) that tests for conformance

The track also explores the use of profile tags.

####  1. Creating Profiles and Value Sets 

This is for any process that produces Profiles and/or value sets

*   Create a profile and/or valuesets

    *   the method of creation is not fixed - it can be by hand, by some conversion process from other formats, or by some tool to profile the content

*   The profile should have these properties:

    *   Profile on Observation

        *   fix observation.name to a limited set of LOINC codes
    *   fix observation.value to a particular type
    *   fix observation.value.units

        *   fix one or more other attributes to max cardinality ..0

        *   fix the value of reliability and status

*   upload (create/update) the created resources on a registry

    *   must get profile/value set references correct on the registry

*   create pass and fail resources, and test them against the profile using the FHIR validation tool (from FHIR DSTU downloads)

For an example of the kind of profile that is intended, see ||link to be provided||

####  2. Getting a server to test conformance 

*   Given a set of observation resources (zip file to be posted here)

*   and a profile (see below)

*   submit the resource for validation using the validation operation on a server

    *   there are no rules about how the submission is prompted

        *   the server [http://fhir.healthintersections.com.au/open](http://fhir.healthintersections.com.au/open) is available, and should be the first system to test on, other servers from #3 below should be tested too

        *   the validation operation requires that the profile be tagged with the full URL of the profile *on the server being tested*

        *   the client must process the return response correctly (pass | fail, not be mislead by hints and warnings

*   the profile master is here (link to be provided). Consult the server administrator for the correct profile tag for the test profile

####  3. Server Validation 

*   host a Profile and value set registry

*   provide an implementation of the validation operation (at the resource level, no need to implement it at the instance level)

*   correctly process profile tags on the validation operation
*   correctly validate the submitted resource against the identified profile
    *   pass|fail outcome must match those produced by the FHIR validation tool (from FHIR DSTU downloads)

###  Track 3 - SMART on FHIR 

This track is the experimental track, focusing on user-facing apps
that launch from an EHR or PHR. SMART on FHIR
uses open standards (FHIR, OAuth2, OpenID Connect) to provide a platform
for health apps that integrate with existing Health IT systems.

*   For an overview, see [http://docs.smartplatforms.org/](http://docs.smartplatforms.org/)

*   Questions? Visit the [SMART on FHIR Google Group](https://groups.google.com/forum/#!forum/smart-on-fhir).

####  1. Build a SMART Server 

Demonstrate a SMART on FHIR server. A successful server will do the following:

1. Support <tt>/Patient</tt> and <tt>/Observation</tt> end points, 
so an app can retrieve demographics and vital signs. 

2. Support the SMART on FHIR launch and authorization,
so an app can request permissions and learn patient context via OAuth2.

**We've created a [quick-start guide for SMART servers](http://docs.smartplatforms.org/tutorials/server-quick-start/)**
with examples of URLs, parameters, LOINC codes, and data payloads you'll need to get started.

*   Other references
<dl>
<dd>[http://fhirblog.com/2014/08/02/smart-on-fhir-part-1/](http://fhirblog.com/2014/08/02/smart-on-fhir-part-1/)
</dd>
<dd>[http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2/](http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2/)
</dd>
</dl>

####  2. Build a SMART App 

Demonstrate a SMART on FHIR client. A successful client will be a Web app
or mobile app that can run against our SMART on FHIR testing server. It should:

*   Talk to SMART's [sandbox server](https://fhir-api.smartplatforms.org) (and any other servers presented at the Connectathon)

*   Authorize to access patient data

*   Retrieve patient data including demographics, vitals, and labs

*   Present some useful view (can be just straight data table) of patient data

**We've created a [quick-start guide for SMART apps](http://docs.smartplatforms.org/tutorials/testing/)**
with code and examples to get you started.

You may also want to explore source code for our
[sample apps on GitHub](https://github.com/smart-on-fhir/apps/tree/gh-pages/static/apps).

You can get started developing from `[http://localhost](http://localhost)`
without registering your app -- but when you want to get your app
running on the Web, just [follow our self-service app registration instructions](http://docs.smartplatforms.org/sandbox/howto/).

###  Track 4 - DICOMweb (Joint with DICOM) 

[DICOMweb](http://dicomweb.hcintegrations.ca/#/home) is the web standard for medical imaging. It is primarily a set of RESTful services, enabling web developers to unlock the power of healthcare images using industry-renowned toolsets.

Although DICOMweb's REST interface and serialization (of DICOM objects) differs from FHIR, FHIR's [ImagingStudy](http://www.hl7.org/implement/standards/fhir/imagingstudy.html) has been developed in cooperation with DICOM to make it easy for DICOM servers to expose their data as FHIR ImagingStudy on the [FHIR REST api](http://www.hl7.org/implement/standards/fhir/http.html), and conversely for natively FHIR servers to expose their ImagingStudy Resources using DICOMweb's REST api.

As both DICOMweb and the FHIR ImagingStudy are recent additions to the standards, the ultimate goal of this track is not just to showcase working implementations but rather to receive feedback on where FHIR and DICOMweb are misaligned based on these implementation efforts.

We propose to focus on [WADO-RS](http://dicomweb.hcintegrations.ca/#/wado) to:

*   For FHIR servers: Expose the FHIR ImagingStudy using DICOMweb's REST interface, converting the ImagingStudy to the xml and/or json [DICOM model](http://dicomweb.hcintegrations.ca/#/xml)

*   For DICOMweb servers: Combine a study with its series and instances as and expose it as a FHIR ImagingStudy resource, using FHIR's REST interface (basically, a [read](http://www.hl7.org/implement/standards/fhir/http.html#read) operation) and FHIR's xml and/or json [serialization model](http://www.hl7.org/implement/standards/fhir/xml.html).