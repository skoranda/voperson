# voPerson v1.1.0

* [Introduction](#introduction)
* [Discussion on Identifiers](#discussion-on-identifiers)
  * [Platform Identifier](#platform-identifier)
  * [General Application Identifier](#general-application-identifier)
  * [Application Specific Identifiers](#application-specific-identifiers)
  * [External Identifiers](#external-identifiers)
     * [eduPerson and SAML Considerations](#eduperson-and-saml-considerations)
  * [Comparison of Identifier Characteristics](#comparison-of-identifier-characteristics)
* [voPerson Attribute Options](#voperson-attribute-options)
  * [`app-*` Attribute Description Option](#app--attribute-description-option)
  * [`internal` Attribute Description Option](#internal-attribute-description-option)
  * [`prior` Attribute Description Option](#prior-attribute-description-option)
  * [`role-*` Attribute Description Option](#role--attribute-description-option)
  * [`scope-*` Attribute Description Option](#scope--attribute-description-option)
  * [`time-#` Attribute Description Option](#time--attribute-description-option)
* [voPerson Object Class and Attributes](#voperson-object-class-and-attributes)
  * [voPerson Object Class Definition](#voperson-object-class-definition)
  * [`voPersonAffiliation` Attribute Definition](#vopersonaffiliation-attribute-definition)
  * [`voPersonApplicationUID` Attribute Definition](#vopersonapplicationuid-attribute-definition)
  * [`voPersonAuthorName` Attribute Definition](#vopersonauthorname-attribute-definition)
  * [`voPersonCertificateDN` Attribute Definition](#vopersoncertificatedn-attribute-definition)
  * [`voPersonCertificateIssuerDN` Attribute Definition](#vopersoncertificateissuerdn-attribute-definition)
  * [`voPersonExternalAffiliation` Attribute Definition](#vopersonexternalaffiliation-attribute-definition)
  * [`voPersonExternalID` Attribute Definition](#vopersonexternalid-attribute-definition)
  * [`voPersonID` Attribute Definition](#vopersonid-attribute-definition)
  * [`voPersonPolicyAgreement` Attribute Definition](#vopersonpolicyagreement-attribute-definition)
  * [`voPersonScopedAffiliation` Attribute Definition](#vopersonscopedaffiliation-attribute-definition)
  * [`voPersonSoRID` Attribute Definition](#vopersonsorid-attribute-definition)
  * [`voPersonStatus` Attribute Definition](#vopersonstatus-attribute-definition)
* [Recommended Attribute Usage](#recommended-attribute-usage)
* [Sample LDIF](#sample-ldif)
* [References](#references)
 
---

# Introduction

*voPerson* is both a set of recommendations and an ldap attribute schema (object class),
intended to provide a common reference point for attribute management within a Virtual
Organization (VO)<sup>1</sup>. The primary goal is to provide a common deployment model to
facilitate application integration within the organization. Although the primary audience
is virtual organizations, this object class may be useful for others as well. 

It is expected that voPerson will be used alongside the *[eduPerson](http://macedir.org/specs/eduperson)*
object class, which in turn is expected to be used alongside *[person](https://tools.ietf.org/html/rfc4519)*,
*[organizationalPerson](https://tools.ietf.org/html/rfc4519)*, and *[inetOrgPerson](https://tools.ietf.org/html/rfc2798)*
object classes. This document includes recommendations for the usage of select attributes
from these object classes. 

voPerson makes use of *attribute options*, a formal part of the LDAP specification ([RFC 4512](https://tools.ietf.org/html/rfc4512)
§2.5) not widely used. Attribute options can provide metadata about values for the attribute.
It may be necessary to configure the directory server to deliver attribute options.

<sup>1</sup>_A VO is an organization that includes members whose identity information is
obtained from multiple sources, at least one of which is external to the organization.
The organization may or may not be a legal entity._

# Discussion on Identifiers

The proper management of identifiers is likely to be one of the most common challenges of
application integration for an organization. Some more background about identifiers is
provided in [eduPerson](http://macedir.org/specs/eduperson) §1.2. The identifier characteristics
defined below follow the definitions introduced in eduPerson.

This section introduces common terminology for various types of identifiers likely to be
of use to the organization. 

## Platform Identifier

The *platform identifier* is a unique, persistent identifier assigned to each participant
within the organization, intended primarily for internal systems use. It may be assigned
randomly or sequentially, but should not be derived from or have an association with other
attributes belonging to the participant. Specifically, it should not be derived from the
participant's name, since a person's name could change over time. The platform identifier
may also be referred to as an *enterprise identifier*.

## General Application Identifier

The *general application identifier* is a human friendly identifier used when the participant
must provide an identifier to an application. This type of identifier is also commonly
referred to as a username, NetID, or uid. It is referred to here as "general" because of
the preference to use this as the default identifier for organizational applications, but
with the recognition that it may not be suitable for every application. Thus the general
application identifier stands in contrast to *application specific identifiers*
(discussed below).

This identifier may be assigned or self-selected, but will typically reflect the participant's
name in some way, often through the use of initials or a combination of initials and full name.
It is important to note, however, that the human friendliness of this identifier also makes it
unsuitable to use as a primary key within an application. This might at first seem
counterintuitive when considered alongside the recommendation that it serves as an application
identifier, however a best practice would be for the application to map the general application
identifier to the platform identifier, and then use the platform identifier as its internal key.

## Application Specific Identifiers

Unfortunately, the general application identifier is typically not sufficient for all use cases.
Common exceptional patterns include

* Legacy applications that already maintain their own identifiers, possibly predating the
  introduction of the general application identifier.
* Proprietary applications that impose constraints on the format of the identifier, such as
  requiring identifiers that look like email addresses or identifiers that are constrained
  to 8 characters.

Application specific identifiers are intended to be used for a single application (or a single
application cluster). The specific format and assignment mechanism of these identifiers will
vary according to the application.

Rather than define an individual attribute for each application specific identifier (resulting
over time in an increasingly difficult to maintain custom object class), voPerson recommends
the use of attribute options to flag the appropriate application for each value of a single attribute.

## External Identifiers

External identifiers are those not assigned or managed by the organization, and can come in
several forms:

* Identifiers used by external authentication services, such as federated or social identity
  providers.

* Identifiers used by data sources that contribute records (such as service eligibility) to
  the organization.

All external identifiers are scoped, either explicitly or implicitly. Explicitly scoped
identifiers include the scoping in the identifier itself (for example, `user@domain.org`),
while implicitly scoped identifiers are effectively scoped by association with the system which asserted them.

External identifiers must typically be tracked for various reasons. For example, external
identifiers used by authentication services must be correlated to the organization's person
record in order to correlate a login event to access to eligible applications and services.
This mapping might be done at a proxy or gateway, or at each application. Data source
identifiers must be tracked so that the appropriate records can be updated on changes to
the source data.

### eduPerson and SAML Considerations

When receiving external identifiers, for example via the SAML protocol, the following order
of preference is recommended:

1. eduPersonOrcid (When the asserting party has properly collected the ORCID ID in accordance
   with ORCID guidelines.)
1. [urn:oasis:names:tc:SAML:attribute:subject-id](https://wiki.oasis-open.org/security/SAMLSubjectIDAttr)
1. eduPersonUniqueId
1. [urn:oasis:names:tc:SAML:attribute:pairwise-id](https://wiki.oasis-open.org/security/SAMLSubjectIDAttr)
1. eduPersonTargetedID or SAML2 Persistent NameID (Only suitable when all of the organization's
   service providers appear as a single entityID to the federation, typically by being placed
   behind a proxy.)
1. eduPersonPrincipalName (In practice, a number of institutions reassign this attribute.)

The received attributes often cannot be stored in the corresponding eduPerson attribute within
the organization's directory, since most of these attributes are defined as single value. It
is fairly typical for organizations to need to store multiple attributes from different
external sources, such as when a participant moves from one institution to another, or if a
participant is multiply affiliated.

## Comparison of Identifier Characteristics

<table>
 <tr>
  <th></th>
  <th>Platform Identifier</th>
  <th>General Application Identifier</th>
  <th>Application Specific Identifier</th>
  <th>External Identifier</th>
 </tr>
 
 <tr>
  <th>Persistence</th>
  <td>Permanent</td>
  <td>Changeable</td>
  <td>Changeable</td>
  <td>Varies</td>
 </tr>
 
 <tr>
  <th>Privacy</th>
  <td>Opaque</td>
  <td>Not necessarily opaque</td>
  <td>Not necessarily opaque</td>
  <td>Varies</td>
 </tr>
 
 <tr>
  <th>Uniqueness</th>
  <td>Unique within the namespace of the platform</td>
  <td>Across all general application identifiers within the platform</td>
  <td>Across each application</td>
  <td>Global, due to scoping</td>
 </tr>
 
 <tr>
  <th>Reassignment</th>
  <td>No</td>
  <td>Discouraged but possible</td>
  <td>Discouraged but possible</td>
  <td>Varies</td>
 </tr>
 
 <tr>
  <th>Human Palatability</th>
  <td>No</td>
  <td>Encouraged</td>
  <td>Possible</td>
  <td>Varies</td>
 </tr>
 
 <tr>
  <th>Source</th>
  <td>Assigned by organization</td>
  <td>Assigned by organization or self selected</td>
  <td>Varies</td>
  <td>External organization</td>
 </tr>
</table>

# voPerson Attribute Options

voPerson makes use of *attribute options*, a formal part of the LDAP specification ([RFC 4512](https://tools.ietf.org/html/rfc4512)
§2.5). Attribute options can provide metadata about values for the attribute. It may be
necessary to configure the directory server to deliver attribute options.

According to [RFC 4520](https://tools.ietf.org/html/rfc4520),

> Options beginning with "x-" are for Private Use and cannot be registered.

Furthermore,

> Options beginning with "e-" are reserved for experiments and will be registered
> on a First Come First Served basis.

The attribute options introduced by voPerson should be considered experimental. However,
following the guidance of [RFC 6648](https://tools.ietf.org/html/rfc6648), voPerson
attribute options use neither prefix.

* At this time, the attribute options defined here are not being formally registered.
  They may be revised according to use in practice.
  
* In the future, if the decision is made to register these options at that time there will
  presumably be a number of deployments using them. Switching from x- or e- prefix notation
  to a non-prefixed notation introduces an unnecessary transitional burden.
  
* Since [RFC 4520](https://tools.ietf.org/html/rfc4520) was first published in 2006, no
  additional attribute description options have been added to the [IANA registry](https://www.iana.org/assignments/ldap-parameters/ldap-parameters.xhtml#ldap-parameters-4).
  The likelihood of conflict in practice is therefore quite low. 

## Important Considerations When Using Attribute Options

Care should be taken when using Attribute Options, as a client unaware of
attribute options may not comprehend the intent, and may behave incorrectly.
For example, if a client that does not parse the `internal` tag may not realize a
value should not be released further, and if the client does not parse the `prior`
tag it may not realize the attribute value is no longer valid.

If downstream clients cannot be tightly managed, it may be desirable to place
appropriate ACLs on Attribute Options (if supported by the LDAP server), or even
to write to two separate LDAP servers (one with Attribute Options and one without).

## `app-*` Attribute Description Option

Used to identify an application specific value for an attribute. `app-` is considered
a prefix, with the remainder of the option consisting of an application specific label.

### Example

```
uid: plee
uid;app-wiki: plee@myvo.org
```

## `internal` Attribute Description Option

The attribute is for internal-to-the-organization use only, and may not be further
released by the client.

### Example

```
mobile;internal: +1 646 555 1212
```

## `prior` Attribute Description Option

Used to identify a previous (deprecated) value for an attribute. That is, any value
labeled prior should not be considered current. The exact semantics will vary
according to the attribute. Typical scenarios include maintaining the prior values
for searching purposes.

### Example

```
voPersonID: V097531
voPersonID;prior: V097522
```

## `role-*` Attribute Description Option

Used to correlate multi-valued attributes. Any value labeled with the same role
label should be considered belonging to the same source. The suffix of the option
(ie: anything after `role-`), has no specific meaning, and should be treated as
opaque except for comparison purposes (caseExactMatch).

### Example

```
ou;role-7975: Department of Parapsychology
ou;role-8801: Department of Astrology
title;role-7975: Precognitive Data Analyst
title;role-8801: Celestial Observer
```

## `scope-*` Attribute Description Option

A scoping label for attributes that are implicitly scoped based on their source.
The suffix of the option (ie: anything after `scope-`) is a label describing the
scope or source of the value. It may have specific meaning in a local context,
but should otherwise be treated as opaque except for comparison purposes
(caseExactMatch).

### Example

```
voPersonSoRID;scope-hrms: E00747400
```

## `time-#` Attribute Description Option

Used to indicate the most recent update time of the attribute. The value is an
integer in Unix (or epoch) time. `time-` may be applied to any attribute in the
voPerson schema, or to any other schema if its use would be non-conflicting with
the definition of that schema.

### Example

```
voPersonPolicyAgreement;time-1516593822: https://myvo.org/policies/acceptable-use
```

# voPerson Object Class and Attributes

Version 1

## voPerson Object Class Definition

```
( 1.3.6.1.4.1.34998.3.3.1
	NAME 'voPerson'
	AUXILIARY
	MAY ( voPersonAffiliation $
            voPersonApplicationUID $
            voPersonAuthorName $
            voPersonCertificateDN $
            voPersonCertificateIssuerDN $
            voPersonExternalAffiliation $
            voPersonExternalID $
            voPersonID $ 
            voPersonPolicyAgreement $ 
            voPersonScopedAffiliation $
            voPersonSoRID $
            voPersonStatus )
)
```

## `voPersonAffiliation` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.10</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.10
        NAME 'voPersonAffiliation'
        DESC 'voPerson Affiliation Within Local Scope'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior user affiliation, no longer current</li>
    <li><code>role-</code>: Denotes a role specific affiliation if present (otherwise overall person affiliation)</li>
    <li><code>scope-<i>sourcelabel</i></code>: Denotes affiliation as asserted by source system</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

An organization-specific affiliation, intended to parallel but expand upon
`eduPersonAffiliation`, allowing for finer grained descriptions of affiliations
(such as graduate student, undergraduate, research, PI, administrator, etc).

### Alternate Approaches

* This attribute could be defined with a controlled vocabulary (similar to
  `eduPersonAffiliation`), however there is not currently community consensus
  as to what those values would be. A constrained attribute can be added later.

### Example

```
voPersonAffiliation: gradstudent
voPersonAffiliation: researcher
```

## `voPersonApplicationUID` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.1</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.1
        NAME 'voPersonApplicationUID'
        DESC 'voPerson Application-Specific User Identifier'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>app-<i>applicationLabel</i></code>: Denotes application identifier is for</li>
    <li><code>prior</code>: Denotes prior user identifier, no longer current</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

An application specific identifier, corresponding roughly with the inetOrgPerson *uid*,
but constrained to one application or one cluster of applications. The use of the `app-`
attribute option is required to denote the target application.

### Alternate Approaches

* A similar approach is possible leveraging the already existing *uid* attribute, but
  requiring the use of the `app-` attribute option. However, creating a new attribute
  clarifies the single application aspect of this identifier.
* A solution is possible without the use of attribute options, using either the existing
  *uid* attribute or the new `voPersonApplicationUID` attribute, by including the target
  application into the attribute value. For example:
  <code>urn:cilogon:appid:<i>applicationlabel</i>:<i>identifier</i></code>. However,
  this imposes significant additional complexity on clients to parse each value looking
  for the one that is appropriate.

### Example

```
voPersonApplicationUID;app-wiki: plee@myvo.org
voPersonApplicationUID;app-wiki;prior: patlee@myvo.org
```

## `voPersonAuthorName` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.2</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.2
        NAME 'voPersonAuthorName'
        DESC 'voPerson Author Name'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior author name, no longer current</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

The formal author name of the subject, suitable for publication purposes.

### Alternate Approaches

* An attribute option (eg: *author*) could be attached to the existing *cn* attribute.
  However, creating a new attribute clarifies the targeted purpose of this attribute.
  
### Example

```
voPersonAuthorName: P Q Lee
voPersonAuthorName;prior: P Q Smith
```

## `voPersonCertificateDN` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.3</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.3
        NAME 'voPersonCertificateDN'
        DESC 'voPerson Certificate Distinguished Name'
        EQUALITY distinguishedNameMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.12' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior Certificate DN, no longer current</li>
    <li><code>scope-</code>: Identifies source to correlate with <code>voPersonCertificateIssuerDN</code></li>
   </ul>
  </td>
 </tr>
</table>

### Definition

The Subject Distinguished Name of an X.509 certificate held by the person.

### Alternate Approaches

* The *userCertificate* attribute from the pkiUser object class ([RFC 4523](https://tools.ietf.org/html/rfc4523))
  requires binary encoding of the complete certificate. 
* A similar approach is possible leveraging the already existing *uid* attribute, but
  requiring the use of the `app-` attribute option, or defining a new *x-509* attribute
  option. However, creating a new attribute allows for the correct equality syntax.
  
### Example

```
voPersonCertificateDN;scope-cert1: CN=Pat Lee A251,O=Example,C=US,DC=cilogon,DC=org
```

## `voPersonCertificateIssuerDN` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.4</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.4
        NAME 'voPersonCertificateIssuerDN'
        DESC 'voPerson Certificate Issuer DN'
        EQUALITY distinguishedNameMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.12' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>scope-</code>: Identifies source to correlate with <code>voPersonCertificateDN</code></li>
   </ul>
  </td>
 </tr>
</table>

### Definition

The Subject Distinguished Name of the X.509 certificate issuer. The use of the `scope`
attribute option is required to denote the correlating certificate.

### Alternate Approaches

* The issuer DN could be encoded in the same attribute value as the certificate DN, but
  then clients would need to perform more sophisticated parsing to extract the values.
  
### Example

```
voPersonCertificateDN;scope-cert1: CN=CILogon Basic CA 1, O=CILogon, C=US, DC=cilogon, DC=org
```

## `voPersonExternalAffiliation` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.11</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.11
        NAME 'voPersonExternalAffiliation'
        DESC 'voPerson Scoped External Affiliation'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior user affiliation, no longer current</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

An organization-specific affiliation as per `voPersonAffiliation`, but
obtained from an external organization, and therefore scoped.

### Alternate Approaches

* This attribute could be consolidated with `voPersonAffiliation`, however it
  is clearer to separate local from external attributes in storage.

### Example

```
voPersonExternalAffiliation: gradstudent@yourvo.org
voPersonExternalAffiliation: protocol-rpg@university.edu
```

## `voPersonExternalID` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.5</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.5
        NAME 'voPersonExternalID'
        DESC 'voPerson External Identifier'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior external identifier, no longer current</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

An identifier for a person, which may be explicitly scoped, typically as issued by an external
authentication service such as a federated or social identity provider. This is
a user-centric identifier, in the context of "bring your own identity". If the
user doesn't know the identifier itself, they at least know the credentials
necessary to generate it.

### Alternate Approaches

* A similar approach is possible leveraging the already existing *uid* attribute,
  but with the use of a new *external* option. However, creating a new attribute
  clarifies the external application aspect of this identifier.
* *eduPersonPrincipalName* and *eduPersonUniqueId* are single valued, and so not suitable.
* *eduPersonTargetedID* is multi-valued, but the targeted semantics are not suitable.

### Example

```
voPersonExternalID: plee@university.edu
voPersonExternalID: 610400998542241058734@google.com
voPersonExternalID: https://university.edu/idp/shibboleth!https://research.org/shibboleth!a6c2c4d4-08b9-4ca7-8ff9-43d83e6e1d35
```

## `voPersonID` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.6</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.6
        NAME 'voPersonID'
        DESC 'voPerson Unique Identifier'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior unique identifier (perhaps deprecated as a
        result of a record merge), no longer current</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

The platform or enterprise identifier.

### Alternate Approaches

* A similar approach is possible leveraging the already existing *uid* attribute, but
  with the use of a new *platform* option. However, creating a new attribute clarifies
  the unique aspect of this identifier.
* *uniqueIdentifier* was defined in [RFC 1274](https://tools.ietf.org/html/rfc1274), but
  its object class *pilotObject* was obsoleted by [RFC 4524](https://tools.ietf.org/html/rfc4524).
* *eduPersonUniqueID* is scoped.
* The description of *employeeNumber* in [RFC 2798](https://tools.ietf.org/html/rfc2798)
  is "numeric", though the syntax does not require it.
  
### Example

```
voPersonID: V097531
```

## `voPersonPolicyAgreement` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.7</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.7
        NAME 'voPersonPolicyAgreement'
        DESC 'voPerson Policy Agreement Indicator'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>time-#</code>: Denotes time the policy was agreed to
   </ul>
  </td>
 </tr>
</table>

### Definition

The URL describing a policy that has been agreed to by the person identified by
this record.

### Alternate Approaches

* A similar approach is possible leveraging the already existing *labeleduri* attribute,
  but even with the use of attribute options it would not be clear that this is a
  document the subject has agreed to, as opposed to being a document created by or
  for the subject.
  
### Example

```
voPersonPolicyAgreement;time-1516593822: https://myvo.org/policies/acceptable-use
```

## `voPersonScopedAffiliation` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.12</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.12
        NAME 'voPersonScopedAffiliation'
        DESC 'voPerson Affiliation With Explicit Local Scope'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>No</td>
 </tr>
</table>

### Definition

Used for transport only, this attribute maps `voPersonAffiliation` when used
across the wire. The resulting value should be stored in `voPersonExternalAffiliation`
by the receiving entity.

### Alternate Approaches

* `voPersonAffiliation` could be used instead, but could cause confusion due to
  representation of scoping.

### Example

Will vary by protocol.

## `voPersonSoRID` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.8</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.8
        NAME 'voPersonSoRID'
        DESC 'voPerson External Identifier'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>prior</code>: Denotes prior external identifier, no longer current</li>
    <li><code>scope-<i>sourcelabel</i></code>: Denotes source system</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

An implicitly scoped identifier for a person, typically as issued by a system of record.
This is a "behind the scenes" or system-centric identifier, and generally cannot be used
for authentication. The user may not even know it exists. 

### Alternate Approaches

* A similar approach is possible leveraging the already existing *uid* attribute, but
  with the use of a new *external* option. However, creating a new attribute clarifies
  the external application aspect of this identifier.

### Example

```
voPersonSoRID;scope-hrms: E00747400
```

## `voPersonStatus` Attribute Definition

<table>
 <tr>
  <th>OID</th>
  <td>1.3.6.1.4.1.34998.3.3.1.9</td>
 </tr>
 
 <tr>
  <th>RFC4512 Definition</th>
  <td>
<pre>( 1.3.6.1.4.1.34998.3.3.1.9
        NAME 'voPersonStatus'
        DESC 'voPerson Status'
        EQUALITY caseIgnoreMatch
        SYNTAX '1.3.6.1.4.1.1466.115.121.1.15' )</pre>
  </td>
 </tr>
 
 <tr>
  <th>Multiple Values?</th>
  <td>Yes, when used with `scope-` attribute option</td>
 </tr>
 
 <tr>
  <th>Attribute Options</th>
  <td>
   <ul>
    <li><code>role-</code>: Denotes a role specific status if present (otherwise overall person status)</li>
    <li><code>scope-<i>sourcelabel</i></code>: Denotes status is as asserted by source system</li>
   </ul>
  </td>
 </tr>
</table>

### Definition

The person's status within the organization. The following standard values are defined,
though the values of this attribute are not constrained to them:

* active
* approved
* confirmed
* declined
* deleted
* denied
* duplicate
* expired
* gracePeriod
* invited
* pending
* pendingApproval
* pendingConfirmation
* suspended

### Example

```
voPersonStatus: active
```

# Recommended Attribute Usage

<table>
 <tr>
  <th>Attribute</th>
  <th>Schema</th>
  <th>Recommended Usage</th>
 </tr>

 <tr>
  <td>DN</td>
  <td></td>
  <td>Construction of the DN should leverage the platform identifier (voPersonID) as the unique component.</td>
 </tr>

 <tr>
  <td>cn</td>
  <td>person</td>
  <td>The fully formatted "official" name, possibly as asserted by the person.</td>
 </tr>

 <tr>
  <td>displayName</td>
  <td>inetOrgPerson</td>
  <td>Self selected/preferred name, fully formatted.</td>
 </tr>

 <tr>
  <td>eduPersonAffiliation</td>
  <td>eduPerson</td>
  <td>Current broad affiliation within the organization.</td>
 </tr>

 <tr>
  <td>eduPersonNickName</td>
  <td>eduPerson</td>
  <td>Self selected/preferred given name.</td>
 </tr>

 <tr>
  <td>eduPersonOrcid</td>
  <td>eduPerson</td>
  <td>ORCID.</td>
 </tr>

 <tr>
  <td>givenName</td>
  <td>inetOrgPerson</td>
  <td>"Official" given name. Strictly speaking this should include middle names, but in practice usually doesn't.</td>
 </tr>

 <tr>
  <td>sn</td>
  <td>person</td>
  <td>"Official" surname.</td>
 </tr>

 <tr>
  <td>uid</td>
  <td>inetOrgPerson</td>
  <td>The general application identifier.</td>
 </tr>

 <tr>
  <td>voPersonAffiliation</td>
  <td>voPerson</td>
  <td>Current fine grained affiliation within the organization.</td>
 </tr>

 <tr>
  <td>voPersonApplicationUID</td>
  <td>voPerson</td>
  <td>Application specific identifiers.</td>
 </tr>

 <tr>
  <td>voPersonAuthorName</td>
  <td>voPerson</td>
  <td>Author (publication) name.</td>
 </tr>

 <tr>
  <td>voPersonCertificateDN</td>
  <td>voPerson</td>
  <td>X.509 certificate subject distinguished name.</td>
 </tr>

 <tr>
  <td>voPersonCertificateIssuerDN</td>
  <td>voPerson</td>
  <td>X.509 certificate issuer subject distinguished name.</td>
 </tr>

 <tr>
  <td>voPersonExternalAffiliation</td>
  <td>voPerson</td>
  <td>Current affiliations obtained from federated or social identity providers.</td>
 </tr>

 <tr>
  <td>voPersonExternalID</td>
  <td>voPerson</td>
  <td>Scoped identifiers obtained from federated or social identity providers.</td>
 </tr>

 <tr>
  <td>voPersonID</td>
  <td>voPerson</td>
  <td>Platform identifier.</td>
 </tr>

 <tr>
  <td>voPersonPolicyAgreement</td>
  <td>voPerson</td>
  <td>Policy agreements.</td>
 </tr>

 <tr>
  <td>voPersonSoRID</td>
  <td>voPerson</td>
  <td>Implicitly scoped identifiers obtained from systems of record.</td>
 </tr>

 <tr>
  <td>voPersonStatus</td>
  <td>voPerson</td>
  <td>Current status within the organization.</td>
 </tr>
</table>

# Sample LDIF

```
dn: voPersonID=V097531, ou=People, dc=myvo, dc=org
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: eduPerson
objectClass: eduMember
objectClass: voPerson
cn: Patricia Q Lee
displayName: Pat Lee
eduPersonAffiliation: staff
eduPersonAffiliation: member
eduPersonNickName: Pat
eduPersonOrcid: http://orcid.org/0000-0002-1825-0097
givenName: Patricia
sn: Lee
sn;prior: Smith
uid: plee
voPersonAffiliation: gradstudent
voPersonAffiliation: researcher
voPersonApplicationUID;app-wiki: plee@myvo.org
voPersonAuthorName: P Q Lee
voPersonCertificateDN;scope-cert1: CN=Pat Lee A251,O=Example,C=US,DC=cilogon,DC=org
voPersonCertificateIssuerDN;scope-cert1: CN=CILogon Basic CA 1, O=CILogon, C=US, DC=cilogon, DC=org
voPersonExternalAffiliation: visitingscholar@yourvo.org
voPersonExternalID: plee@university.edu
voPersonExternalID: 610400998542241058734@google.com
voPersonID: V097531
voPersonID;prior: V097522
voPersonPolicyAgreement;time-1516593822: https://myvo.org/policies/acceptable-use
voPersonSoRID;scope-hrms: E00747400
voPersonStatus: active
```

# References

1. [eduPerson](http://macedir.org/specs/eduperson) eduPerson Object Class Specification
1. [IANA Attribute Description Options Registry](https://www.iana.org/assignments/ldap-parameters/ldap-parameters.xhtml#ldap-parameters-4)
1. [RFC 1274](https://tools.ietf.org/html/rfc1274) The COSINE and Internet X.500 Schema
1. [RFC 2798](https://tools.ietf.org/html/rfc2798) Definition of the inetOrgPerson LDAP Object Class
1. [RFC 4512](https://tools.ietf.org/html/rfc4512) Lightweight Directory Access Protocol (LDAP): Directory Information Models
1. [RFC 4519](https://tools.ietf.org/html/rfc4519) Lightweight Directory Access Protocol (LDAP): Schema for User Applications
1. [RFC 4520](https://tools.ietf.org/html/rfc4520) Internet Assigned Numbers Authority (IANA) Considerations for the Lightweight Directory Access Protocol (LDAP)
1. [RFC 4523](https://tools.ietf.org/html/rfc4523) Lightweight Directory Access Protocol (LDAP) Schema Definitions for X.509 Certificates
1. [RFC 4524](https://tools.ietf.org/html/rfc4524) COSINE LDAP/X.500 Schema
1. [RFC 6648](https://tools.ietf.org/html/rfc6648) Deprecating the "X-" Prefix and Similar Constructs in Application Protocols
1. [SAML V2.0 Subject Identifier Attributes Profile Version 1.0](https://wiki.oasis-open.org/security/SAMLSubjectIDAttr)

# Changelog

voPerson version numbers follows [Semantic Versioning](https://semver.org/).

A patch version change (eg: v1.0.0 to v1.0.1) should not require an update to
the LDAP server, except to correct an error.

A minor version change (eg: v1.0.0 to v1.1.0) may require an update to the LDAP
server, as it must be made aware of the new attributes. However, a minor
version change will not break existing LDAP entries.

A major version change (eg: v1.0.0 to v2.0.0) requires a reconfiguration to the
LDAP server, and may require existing records to be modified.

## [v1.1.0](https://github.com/voperson/voperson/tree/1.1.0)

* Added voPersonAffiliation, voPersonExternalAffiliation, and voPersonScopedAffiliation.
* Added "Important Considerations When Using Attribute Options".
* Errata.

## [v1.0.0](https://github.com/voperson/voperson/tree/1.0.0)

* Initial Release.
