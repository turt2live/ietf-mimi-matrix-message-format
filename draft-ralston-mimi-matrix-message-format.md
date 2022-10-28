---
title: "Matrix Message Format"
abbrev: "Matrix Message Format"
category: info

docname: draft-ralston-mimi-matrix-message-format-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - matrix
 - mimi
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "turt2live/ietf-mimi-matrix-message-format"
  latest: "https://turt2live.github.io/ietf-mimi-matrix-message-format/draft-ralston-mimi-matrix-message-format.html"

author:
 -
    fullname: Travis Ralston
    organization: The Matrix.org Foundation C.I.C.
    email: travisr@matrix.org

normative:
  MxEvents:
    target: https://spec.matrix.org/v1.4/client-server-api/#events
    title: "Events | Client-Server API"
    date: 2022
    author:
      - org: The Matrix.org Foundation C.I.C.
  MSC1767:
    target: https://github.com/matrix-org/matrix-spec-proposals/blob/d6046d8402e7a3c7a4fcbc9da16ea9bad5968992/proposals/1767-extensible-events.md
    title: "Extensible event types & fallback in Matrix (v2)"
    date: 2022
    author:
      - name: Matthew Hodgson
        org: The Matrix.org Foundation C.I.C.
      - name: Travis Ralston
        org: The Matrix.org Foundation C.I.C.
  MSC3551:
    target: https://github.com/matrix-org/matrix-spec-proposals/blob/5bf2118e8ac873e7845b1eedde8dd7bc187ed673/proposals/3551-extensible-events-files.md
    title: "Extensible Events - Files"
    date: 2021
    author:
      - name: Travis Ralston
        org: The Matrix.org Foundation C.I.C.
informative:
  MSC3381:
    target: https://github.com/matrix-org/matrix-spec-proposals/blob/95fdc44b904d2b4d2f227db99050e539e43f3509/proposals/3381-polls.md
    title: "Polls (mk II)"
    date: 2022
    author:
      - name: Travis Ralston
        org: The Matrix.org Foundation C.I.C.
  DMLS:
    target: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org
    title: "Decentralised MLS"
    author:
      - name: Hubert Chathi
        org: The Matrix.org Foundation C.I.C.
    date: 2021
    seriesinfo:
      Web: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org
    format:
      ORG: https://gitlab.matrix.org/matrix-org/mls-ts/-/blob/dd57bc25f6145ddedfb6d193f6baebf5133db7ed/decentralised.org

--- abstract

This document specifies a message format using Matrix for messaging interoperability.


--- middle

# Introduction

Interoperable instant messaging requires a common format for all participants to contribute to the conversation or state of the room. Matrix, alongside its existing transport functionality [TODO: Link to draft-ralston-mimi-matrix-transport], has support for arbitrarily extensible payloads of information called "events" to carry information between machines and users, all with encryption if desired.

Communication between machines would largely take the form of key sharing or VoIP signalling (in a future phase of MIMI) while communication between users would be text, images, etc from normal Instant Messaging (IM) semantics.

# Matrix Events

Events {{MxEvents}} are JSON objects which get signed and adapted by the server before being forwarded to other servers/users in the room. These JSON objects have a few key fields:

* `sender`: The user ID (`@alice:example.org`) which sent the event.
* `room_id`: The room ID (`!room:example.org`) for where the event was sent.
* `type`: A string the client can use to determine how to render the event.
* `content`: Type-specific JSON object.
* Other fields (TODO: define these in detail when more relevant to the doc).

Under MSC1767 {{MSC1767}} (a spec change proposal in the existing Matrix open standard ecosystem), callers would be able to combine useful chunks of content (other event types) to indicate how a client should render the event, if it does not already know.

An example of a simple text message would be:

```
{
    "type": "m.message",
    "content": {
        "m.text": "i am a fish",
        "m.html": "i am a <strong>fish</strong>"
    }
}
```

This can be made more complex if the sender chooses to mix in other mimetypes:

```
{
  "type": "m.message",
  "content": {
    "m.message": [
        { "mimetype": "text/html", "body": "i am a <strong>fish</strong>" },
        { "mimetype": "text/plain", "body": "i am a fish" },
        { "mimetype": "application/vnd.exampleorg.message+html", "body": "<content>i am a <strong>fish</strong></content>" }
    ]
  }
}
```

To demonstrate extensibility, a file upload {{MSC3551}} might look like:

```
{
  "type": "m.file",
  "content": {
    "m.text": "Upload: foo.pdf (12 KB)",
    "m.file": {
      "url": "mxc://example.org/abcd1234",
      "name": "foo.pdf",
      "mimetype": "application/pdf",
      "size": 12345
    }
  }
}
```

In this example, clients which do not understand `m.file` but do understand `m.text` (or `m.message`) would show just the plain text instead of a download button or link. The alternative to falling back would be to hide the unrenderable event, causing the conversation history to be interrupted: this has fairly obvious negative consequences on user experience. Instead, by defining a fallback mechanism the user is still able to participate in the conversation, though might need to ask for more information. It is expected that the "base types" (text messages, images, videos, and generic files) would be supported by all clients to ensure there are sufficient building blocks for future extensibility.

A more complete use-case for extensible events is described by "MSC3381: Polls" {{MSC3381}} - clients which do not yet have support for polls can present their users with text fallback for the question and the question asker can manually tally up "improper" responses (if those users simply sent text messages in response to the question). Clients which do support polls would simply show the poll and its question/options for the user to click on - their response would be sent to the room as a (deliberately) unrenderable event for other clients to tally up automatically.

# Encryption

Matrix has specified an encryption algorithm for events called Megolm, however for the purposes of MIMI it would be desirable to adopt MLS {{!I-D.ietf-mls-protocol}} instead. Some bookkeeping changes are required to support MLS in a decentralized environment like Matrix: those are currently defined by {{DMLS}}.

# Security Considerations

TODO Security. Future drafts should consider the encryption aspects in particular.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
