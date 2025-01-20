---
title: "Vocabulary for Expressing Content Preferences for AI Training"
abbrev: "AIPREF Vocab"
category: info

docname: draft-vaughan-aipref-vocab-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - AI Preferences
 - Opt-Out
 - Vocabulary
venue:
  group: WG
  type: Working Group
  mail: ai-control@ietf.org
  arch: https://datatracker.ietf.org/wg/aipref/about/
  github: thunderpoot/draft-vaughan-aipref-vocab
  latest: https://example.com/LATEST

author:
 -
    fullname: Thom Vaughan
    organization: Common Crawl Foundation
    email: thom@commoncrawl.org

normative:

informative:

#* [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html): Data elements and interchange formats – Representation of dates and times
#* [ISO 3166](https://www.iso.org/iso-3166-country-codes.html): Codes for the representation of names of countries and their subdivisions
#* [draft-illyes-rep-purpose](https://datatracker.ietf.org/doc/draft-illyes-rep-purpose/): Robots Exclusion Protocol User Agent Purpose Extension
#* [TDMRep](https://www.w3.org/community/reports/tdmrep/CG-FINAL-tdmrep-20240510/): Text and Data Mining Reservation Protocol


--- abstract

This document proposes a vocabulary for expressing content preferences for rightsholders who wish to manage the use of their content in AI training. This vocabulary allows publishers to express preferences through metadata or content-delivery protocols. The vocabulary can be applied at different levels of granularity and incorporates preferences for permissions, usage scope, and data retention, providing a foundation for interoperability across various Internet protocols.


--- middle

# Introduction

As AI models become more reliant on large-scale data (driven by scaling laws that link model performance to dataset size), content publishers seek ways to control how their content is used in training these models. This draft provides a vocabulary that enables publishers to signal preferences for AI training concerning their content.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Scope

The AI-PREF vocabulary is limited to expressing content preferences for AI training and does not include enforcement mechanisms or client authentication. Default opt-in or opt-out statuses are beyond the scope of this proposal, as it focuses solely on establishing a standard for signalling explicit preferences. In cases where no preferences are signalled, the decision on whether this constitutes an opt-in or opt-out should be determined at the policy level downstream.

It is important to note that preference signals are advisory.


# Vocabulary Elements / Preference Signals

## Permission

Basic indicators of whether content can be used for AI training.

* **allow\_training**: Boolean
* **restricted\_training:** public, non-commercial, internal, licensed

## Purpose

Defines acceptable uses in training.

* **purpose**: String:
  * **generation**: Creating models that are capable of generating content
  * **embedding**: Converting content to vector representations
    * **classification**: Categorising or labelling content
    * **summary**: Creating condensed versions of content
    * **paraphrase**: Creating derivative versions of content
    * **quotation**: Repetition of a passage or fragment of original content
    * **translation**: Converting content between languages

## Temporal Restrictions

Specifies the date range for training use.

* **effective\_date**: `ISO 8601` Date string
* **expiration\_date**: `ISO 8601` Date string

## Content-Specific Granularity

Defines the scope of applicability. Refers to the level at which preferences apply within the content.

* **scope**: global, content-specific, conditional

## Content Type

Specifies content types the preference applies to.

* **mime\_type**: text, image, video, audio, application, {{!RFC2046}}.

## Derivative Content

Allows or restricts derivatives like summaries.

* **allow\_derivatives**: Boolean
* **derivative\_type**: summary, paraphrase, translation

## Data Retention

Defines content retention period post-training.

* **retention\_period**: `ISO 8601` Duration string (e.g., P3Y6M4DT12H30M5S)

## Preference Persistence

Indicates if preferences should persist in derived datasets, or be optional. A derived dataset is the result of processing, transforming, or extracting information from the original source, such as aggregated statistics and summaries, or subsets of data.

* **metadata\_persistence**: Boolean

## Precedence

Conflicts should be resolved by assigning precedence values (e.g., `high`, `medium`, `low`) to rules, with a defined hierarchy that allows content producers to override publishers, domain operators, and others as necessary.

* **precedence:** Sets priority when preferences conflict with other layered preferences.

## Geographic Restrictions

Specifies regions where preferences apply, `ISO 3166-1`.

* **geo\_limitations:** Specifies geographic regions where training permissions apply.

# Implementation Considerations

Implementing the AI-PREF vocabulary effectively can be accomplished using various mechanisms, depending on the needs and existing infrastructure of content publishers. Approaches include, but are not limited to, using HTTP headers, possible extensions to {{!RFC9309}} ({{?PURPOSE=I-D.illyes-rep-purpose}}), and (for example) \<meta\> tags and other embedded data (such as EXIF) for sub-document-level control.

## HTTP Headers

Publishers can use HTTP headers to communicate AI-PREF preferences directly in response to client requests. This approach allows fine-grained control and easy integration into existing server configurations.

**Example header:**


~~~ http-message
AI-PREF: allow_training=true; purpose=generation,classification; retention_period=P3Y6M4DT12H30M5S
~~~

This header specifies that the content can be used for text generation and classification, with a retention period of 3 years, 6 months, 4 days, 12 hours, 30 minutes, and 5 seconds. The syntax and options should be carefully chosen to ensure compatibility with common web servers and clients.

## Robots Exclusion Protocol (REP)

For publishers who already use REP (as defined in [RFC9309](https://datatracker.ietf.org/doc/rfc9309/)), extending REP rules to include AI-PREF preferences could be beneficial.

Example rule:

~~~
User-agent: *
Allow-Training: non-commercial
Purpose: embedding, summarisation
~~~

This REP rule specifies that all user agents are allowed to use the content for non-commercial AI training, limited to embedding and summarisation purposes. Further extensions to REP could specify additional constraints, such as geographic limitations or temporal restrictions.

## \<meta\> Tags for Sub-Document Level Control

To specify AI-PREF preferences at the level of individual HTML documents or specific parts of a document, \<meta\> tags and HTML attributes can be used.

Example \<meta\> tag:


~~~ html
<meta name="AI-PREF" content="allow_training=false; retention_period=0">
~~~


Example HTML attribute:

~~~ html
<div data-aipref="allow_training=false; retention_period=0">
~~~


The methods above specify that AI training is not allowed for the content of this document, with no retention period permitted. \<meta\> tags can be used to provide  specific content preferences for a specific piece of content, and thus provide a flexible way to manage AI training signals at a more granular level.

## “Well-Known” Locations

According to {{!RFC8615}}, “well-known” locations can serve metadata or configuration information that is easily discoverable by automated clients. AI-PREF preferences can be published at a “well-known” URL. There is already the Text and Data Mining Reservation Protocol ([TDMRep](https://www.w3.org/community/reports/tdmrep/CG-FINAL-tdmrep-20240510/)) which has the same or overlapping intent.

Example:

~~~
https://example.com/.well-known/aipref
~~~


At this URL, a JSON or other structured format can specify AI-PREF preferences for the entire domain or specific content types.

Example JSON 1:

~~~ json
{
  "allow_training": false,
  "purpose": ["generation"],
  "retention_period": "0"
}
~~~


Example JSON 2:

~~~ json
{
  "version": "1.0",
  "resources": [
    {
      "path": "/videos/tutorial.mp4",
      "type": "video/mp4",
      "components": [
        {
          "name": "Introduction",
          "time-range": "00:00:00-00:01:00",
          "preferences": {
            "classification": "allowed",
            "embedding": "allowed"
          }
        },
        {
          "name": "Main Content",
          "time-range": "00:01:01-00:05:00",
          "preferences": {
            "generation": "prohibited",
            "summarization": "allowed"
          }
        }
      ]
    }
  ]
}
~~~


This approach simplifies discovery for automated clients and provides a centralised way to communicate content preferences across a domain.

TDMRep Example:

A rightsholder could expose a “well-known” TDMRep file at:


https://example.com/.well-known/tdmrep


Example TDMRep JSON Content:


~~~ json
{
  "version": "1.0",
  "license": "https://example.com/license",
  "contact": {
    "email": "tdm-support@example.com",
    "url": "https://example.com/contact"
  },
  "resources": [
    {
      "path": "/articles/",
      "type": "text/html",
      "restriction": "no-crawling"
    },
    {
      "path": "/api/data/",
      "type": "application/json",
      "restriction": "license-required"
    }
  ]
}
~~~


## Embedded Metadata

Preferences for multimodal data can be embedded directly into file metadata (such as EXIF or XMP) as self-contained control signals.  Compatibility and tamper resistance (e.g. signing) should be considered.

Example EXIF:


~~~
AI-Pref-Allow-Training: false
AI-Pref-Purpose: embedding
AI-Pref-Retention-Period: 0
~~~


Example PDF Metadata Using XMP:


~~~ xml
<x:xmpmeta xmlns:x="adobe:ns:meta/">
   <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
      <rdf:Description rdf:about=""
         xmlns:dc="http://purl.org/dc/elements/1.1/">
         <dc:Rights>Text mining allowed; Data sharing restricted</dc:Rights>
      </rdf:Description>
   </rdf:RDF>
</x:xmpmeta>
~~~


Preferences can be applied at the file level, or even to specific components (e.g., chapters in a PDF or frames in a video).

Example WEBVTT:


~~~
WEBVTT

00:00:00.000 --> 00:01:00.000
Usage Preferences: allow_training=true; purpose=generation,classification

00:01:01.000 --> 00:05:00.000
Usage Preferences: allow_training=false;
~~~


## Content Credentials (ISO 22144\)

  TBD

## ISCC (ISO 24138\)

  TBD

# Example Usage Scenarios

TODO examples


# Security Considerations

This document does not affect the security of the Internet. AI-PREF preferences do not include enforcement mechanisms, which should be addressed by AI model developers. Publishers should be aware that preferences may not prevent unauthorised use and may rely on mutual agreements or legal protections.


# IANA Considerations

This document does not require any immediate IANA actions but may suggest future registry entries for the vocabulary terms to support interoperability.


--- back

# Table of Preference Signals {#t-signals}

This table defines terms and values that specify metadata preferences for the use of content in AI training. Each term includes a description of its purpose and example values:

| Term | Values | Description | Example |
| :---- | :---- | :---- | :---- |
| `allow_training` | Boolean | Basic indicator of whether content can be used for AI training | `allow_training: false` |
| `purpose` | String: `generation, classification, summarisation, embedding`, etc | Defines acceptable applications for training e.g. fine-tuning, classification, summarisation, etc | `purpose: classification, summarisation` |
| `effective_date` | Date string, **ISO 8601** | Start date of when permissions take effect | `effective_date: 2024-10-30T15:52:55.440238` |
| `expiration_date` | Date string, **ISO 8601** | Date after which permissions no longer apply | `expiration_date: 2024-10-30T15:52:55.440238` |
| `scope` | String: `global, content-specific, conditional` | Defines whether the preferences apply universally, to specific content, or under certain conditions | `scope: content-specific` |
| `mime_type` | `text, image, video, audio` | Specifies the type(s) of content the preference applies to | `mime_type: text, image` |
| `allow_derivatives` | Boolean | Indicates whether derivative works (summaries, paraphrasing) are allowed based on content | `allow_derivatives: true` |
| `derivative_type` | String: `summary, paraphrase, translation` | Lists permissible types is `allow_derivatives` is `true` | `derivative_type: summary, paraphrase` |
| `retention_period` | Duration string, **ISO 8601** | Specifies how long content may be retained after use (e.g. after training). | `P3Y6M4DT12H30M5S` representing three years, six months, four days, twelve hours, thirty minutes, and five seconds. |
| `preference_persistence` | Boolean | Whether preferences must persist with derived data, boolean for either `required` or `optional` | `preference_persistence: true` |
| `precedence` | `String:high, medium, low` | Sets priority when preferences conflict with other layered preferences | `precedence: high` |
| `geo_limitations` | Location codes, **ISO 3166** | Specifies geographic regions where training permissions apply | `geo_limitations: EU, US` |



# Acknowledgments
{:numbered="false"}

* Greg Lindahl
* Sebastian Nagel
* Gary Illyes
* Mark Nottingham
* Suresh Krishnan
* Martin Thomson
* Paul Keller
* Leonard Rosenthol
* Special thanks to the program committee and contributing members of the IAB AI-CONTROL Workshop, and aipref Working Group.
