### GCP IAM Admin API Conformance Testing Strategy

Specific challenges encountered with the **GCP IAM Admin Java Client** that necessitated a different testing approach compared to the standard REST-based conformance tests.

#### 1. Client Transport Limitations (gRPC vs. REST)
Unlike other GCP client libraries we use, the `google-iam-admin` library does not natively expose an HTTP/JSON (REST) transport option in its `StubSettings`. It forces the use of **gRPC**, which requires HTTP/2. This prevented us from using our standard HTTP-based WireMock recording setup.

#### 2. WireMock gRPC Extension Limitations
While we utilized the `wiremock-grpc-extension` to handle the HTTP/2 connection and protocol framing, it currently lacks full parity with the standard WireMock HTTP recorder:
* **No Auto-Recording:** The extension cannot transparently proxy binary gRPC streams and serialize the Protobuf responses into readable JSON files automatically.
* **Strict Matching:** The extension requires precise endpoint matching (e.g., strict leading slashes), which caused `UNIMPLEMENTED` errors when using standard auto-loaded mappings.

#### 3. Implementation of Custom Record/Replay Pattern
To bridge this gap, we implemented a manual Record/Replay workflow:

* **Record Mode:**
    * Bypassed WireMock to connect directly to GCP.
    * Intercepted the response objects.
    * Manually serialized the Protobuf messages to JSON using `JsonFormat` and saved them to a dedicated `recorded_responses` directory (avoiding conflicts with WireMock's auto-scanner).
* **Replay Mode:**
    * Configured the client with **SSL/ALPN** to negotiate HTTP/2 with WireMock.
    * Programmatically loaded the JSON files, parsed them back into Protobuf objects, and registered them as stubs using the gRPC Extension DSL.