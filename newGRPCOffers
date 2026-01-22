# Steps to Add a New gRPC Method to OffersGrpcService

## 1. Update Proto Files

### a. Add new request and response messages
File: `/marketplace/svc-spec-offers/src/main/proto/com/earnin/service/offers/v1/offers_config.proto`

```protobuf
message GetOfferStatsRequest {
  string offer_id = 1;
  optional google.protobuf.Timestamp start_date = 2;
  optional google.protobuf.Timestamp end_date = 3;
}

message GetOfferStatsResponse {
  bool success = 1;
  optional string error_message = 2;

  OfferStats stats = 3;

  message OfferStats {
    string offer_id = 1;
    int32 total_views = 2;
    int32 total_clicks = 3;
    float conversion_rate = 4;

    // Audit
    google.protobuf.Timestamp last_updated_utc = 5;
  }
}
```

### b. Add the new RPC to the service definition
File: `/marketplace/svc-spec-offers/src/main/proto/com/earnin/service/offers/v1/offers.proto`

```protobuf
service OfferService {
  // ... existing RPCs ...
  rpc GetOfferStats(GetOfferStatsRequest) returns (GetOfferStatsResponse) {}
}
```

## 2. Generate Stub Code

Run the appropriate gradle task to generate Kotlin stubs.

## 3. Create a New Request Handler

File: `/marketplace/svc-offers/src/main/kotlin/com/earnin/service/offers/grpc/handlers/GetOfferStatsRequestHandler.kt`

```kotlin
package com.earnin.service.offers.grpc.handlers

import com.earnin.service.offers.v1.GetOfferStatsRequest
import com.earnin.service.offers.v1.GetOfferStatsResponse
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class GetOfferStatsRequestHandler @Inject constructor(
  private val offerStatsRepository: OfferStatsRepository
) {
  suspend fun handle(request: GetOfferStatsRequest): GetOfferStatsResponse {
    // Implement the business logic here
    TODO("Implement GetOfferStats logic")
  }
}
```

## 4. Update OffersGrpcService

File: `/marketplace/svc-offers/src/main/kotlin/com/earnin/service/offers/grpc/OffersGrpcService.kt`

```kotlin
class OffersGrpcService @Inject constructor(
  // ... existing parameters ...
  private val getOfferStatsRequestHandler: GetOfferStatsRequestHandler,
  // ... other parameters ...
) : OfferServiceCoroutineImplBase() {
  // ... existing methods ...

  override suspend fun getOfferStats(request: GetOfferStatsRequest): GetOfferStatsResponse = withSpan("getOfferStats") { span ->
    return@withSpan withErrorHandling(span) {
      getOfferStatsRequestHandler.handle(request)
    }
  }
}
```

## 5. Implement Business Logic in the Handler

Update the `handle` method in `GetOfferStatsRequestHandler.kt`:

```kotlin
suspend fun handle(request: GetOfferStatsRequest): GetOfferStatsResponse {
  try {
    val stats = offerStatsRepository.getStats(request.offerId, request.startDate, request.endDate)
    return GetOfferStatsResponse.newBuilder()
      .setSuccess(true)
      .setStats(stats)
      .build()
  } catch (e: Exception) {
    return GetOfferStatsResponse.newBuilder()
      .setSuccess(false)
      .setErrorMessage("Failed to retrieve offer stats: ${e.message}")
      .build()
  }
}
```

## 6. Update Dependency Injection

Ensure the new handler is properly registered in your DI configuration if needed.

## 7. Add Tests

File: `/marketplace/svc-offers/src/test/kotlin/com/earnin/service/offers/grpc/handlers/GetOfferStatsRequestHandlerTest.kt`

```kotlin
class GetOfferStatsRequestHandlerTest {
  @Test
  fun `getOfferStats returns correct stats for valid request`() {
    // Implement test
  }

  @Test
  fun `getOfferStats returns error for invalid offer id`() {
    // Implement test
  }

  // Add more tests as needed
}
```

Remember to maintain consistency with error handling, logging, and code style as seen in other parts of the codebase.
