Create a Python module named feature_extractor.py for an automotive CCU2 gateway and telematics log diagnosis tool.

Context:
- Logs come from a DLT parser and are already structured
- Each row is a dict with keys:
  line_no, timestamp, ecu, app, context, level, payload

Goal:
Convert raw log rows into structured diagnostic signals.

Implement:

class FeatureExtractor:
    def extract(self, rows: list[dict]) -> dict

Output should be:

{
  "key_events": list[str],
  "error_events": list[str],
  "warning_events": list[str],
  "keywords": list[str],
  "missing_expected_events": list[str],
  "sequence_observations": list[str],
  "candidate_domain": str,
  "event_flags": dict,
  "evidence_refs": list[dict]
}

Domains to detect:
- GATEWAY
- TELEMATICS
- LTE
- SOME/IP
- UDS
- DoIP
- BOOT
- GENERIC

Requirements:

1. Normalize payload safely:
   - handle None
   - strip null characters
   - lowercase for matching
   - preserve original payload

2. Detect keywords from payload/app/context:
   - gateway, forward, routing, dropped, received, send, publish
   - modem, sim, apn, attach, detach, registration, bearer
   - lte, network, signal, backend, data session
   - someip, findservice, offerservice, subscribeeventgroup, subscribeack
   - uds, nrc, diagnostic session, security access
   - doip
   - boot, startup, init, ready

3. Extract key events using deterministic matching:
   - FindService
   - OfferService
   - SubscribeEventgroup
   - SubscribeAck
   - Timeout
   - Registration / Attach
   - Send / Forward / Drop

4. Detect error_events if:
   - level is ERROR/FAIL
   - or payload contains: timeout, failed, error, rejected, unavailable, drop

5. Detect warning_events if:
   - level is WARN
   - or payload contains warning

6. Maintain event_flags like:
   {
     "find_service_seen": bool,
     "offer_service_seen": bool,
     "timeout_seen": bool,
     "lte_registered": bool,
     "data_sent": bool
   }

7. Missing expected events logic:
   - If FindService seen but OfferService not seen → missing OfferService
   - If SubscribeEventgroup seen but SubscribeAck not seen → missing SubscribeAck
   - If data send attempted but no LTE registration → missing LTE readiness

8. Sequence observations:
   - Client request without provider response
   - Data received but not forwarded
   - LTE not ready before send attempt

9. Evidence refs:
   Collect important rows:
   {
     "line_no": int,
     "timestamp": str,
     "payload": str
   }

10. Deduplicate all outputs
11. Keep order stable
12. Use helper methods:
   - _normalize_text
   - _detect_domain
   - _extract_keywords
   - _extract_events
   - _detect_missing_events
   - _build_sequence_observations
   - _collect_evidence

13. No external libraries, pure Python 3.12
14. Add a small __main__ test with sample rows

Return full working file.