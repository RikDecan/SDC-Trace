# SDC-Trace
Easy and Readable Timeline Analyzer for IEEE 11073 SDC


## Expected Behaviour:

Given a `.pcap` or `.pcapng` capture containing SDC/DPWS traffic, the tool shall:
- Parse network packets offline (no live capture in MVP).
- Identify WS-Discovery traffic (UDP multicast on port 3702).
- Detect and summarize SOAP-over-HTTP interactions.
- Correlate HTTP request/response pairs.
- Extract key SOAP metadata (Action, MessageID, RelatesTo).
- Produce a human-readable chronological timeline.
- Optionally export structured JSON for automation or diffing.

The output should allow an integration engineer to understand:
- When discovery occurred.
- Which endpoints were detected.
- Which services were accessed.
- Whether SOAP calls succeeded or failed.
- The high-level flow of interaction.

The tool is not a Wireshark plugin and does not replace packet inspection. It provides structured narrative insight.

--- 

## Requirements:
### Functional Requirements:
1. Offline Capture Processing
   - Accept .pcap and .pcapng files as input.
   - Support Linux-cooked captures (SLL) and Ethernet.

2.Discovery Detection
   - Detect UDP packets on port 3702.
   - Identify WS-Discovery message types (Probe, Hello, Resolve).
   - Extract:
     - Source IP
     - Destination IP
     - SOAP Action
     - EndpointReference (if present)
3. HTTP/SOAP Parsing
  - Detect HTTP traffic on dynamic or configurable ports.
  - Extract:
    - HTTP method
    - Path
    - Status code
    - Content-Length
    - Content-Type
  - Detect SOAP envelopes.
  - Extract:
    - wsa:Action
    - wsa:MessageID
    - wsa:RelatesTo

4. Correlation
- Match HTTP requests to responses using:
    - `MessageID` ↔ `RelatesTo`
    - Fallback: TCP flow + sequence timing
- Compute approximate request/response latency.

5. Timeline Generation
- Produce chronological event entries such as:
  - Discovery event
  - Metadata retrieval
  - WSDL retrieval
  - MDIB retrieval
  - Subscription start
  - SOAP fault

6. Output Formats
- Human-readable text output (default).
- JSON output (optional flag).

--- 

### Non-Functional Requirements:

1. Deterministic Output
- Same input capture must produce identical output.

2. Performance
- Handle captures up to 100MB without excessive memory growth.
- Process packets in streaming fashion where possible.

3. Extensibility
-Architecture must allow:
  - Additional protocol decoders
  - Deeper XML parsing later
  - Plugin-style event processors

4. Portability
- Must compile and run on:
  - Ubuntu Linux (primary)
  - Windows (secondary goal)

5. No Live Capture in MVP
- Live capture support explicitly deferred to future phase.

---

## Implementation
**Phase 1 – MVP** 

Core Modules:
- `PcapReader`
  - Iterates packets
  - Extracts L3/L4 headers

- `DiscoveryDecoder`
  - Detects UDP 3702
  - Extracts SOAP action
- `HttpDecoder`
  - Detects HTTP requests/responses
  - Extracts basic headers

- `SoapInspector`
  - Lightweight string-based XML scanning
  - Extracts Action, MessageID, RelatesTo

- `Correlator`
  - Matches requests to responses
  - Computes latency

- `TimelineBuilder`
  - Converts events into ordered narrative
  
- `OutputRenderer`
  - Text format
  - JSON format

--- 

**Phase 2 – Enhancements**
- MDIB extraction summary
- Subscription lifecycle detection
- Error categorization (SOAP Fault types)
- Conversation grouping per device
- Capture diff mode

## Acceptance Criteria
The project is considered MVP-complete when:

- Given a known SDC capture:
  - Discovery (Probe/Hello) events are detected.
  - Metadata exchange events are detected.
  - WSDL download events are detected.
  - SOAP 200 and SOAP Fault responses are detected.
  - Request/response pairs are correctly correlated.
- Timeline output is readable and chronologically correct.
- JSON output passes schema validation.
- Unit tests pass.

## Why is this important?
Interoperability debugging in medical systems is time-consuming and error-prone.

Raw packet captures are:
- Verbose
- Low-level
- Difficult to interpret quickly

This tool provides:
- Faster root cause identification
- Clear interaction flow visualization
- Reproducible offline analysis
- Structured data for automated testing

In a medical technology context, improving interoperability debugging directly improves:
- Integration reliability
- Development velocity
- System safety
