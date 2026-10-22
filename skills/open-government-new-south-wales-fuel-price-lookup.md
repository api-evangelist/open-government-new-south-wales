---
name: fuel-price-lookup
description: Authenticate to the NSW Fuel API (FuelCheck) and look up current fuel prices for all stations, a single station, a location or nearby coordinates.
api: openapi/open-government-new-south-wales-fuel-openapi.yml
operations:
- OauthClientCredentialAccesstokenGet
generated: '2026-10-21'
method: generated
---
# Fuel price lookup (NSW Fuel API)

The provider contract sets an operationId only on the token call. The steps below name the other operations by method + path, exactly as they appear in the spec. The overlay in `overlays/` proposes operationIds for them.

1. **Get a token**: `OauthClientCredentialAccesstokenGet` (GET /oauth/client_credential/accesstoken) with `grant_type=client_credentials` and a Basic `Authorization` header built from your API.NSW key and secret. A 401 means bad credentials.
2. **Load reference data**: GET /FuelCheckRefData/v2/fuel/lovs. Send `Authorization: Bearer <token>`, `apikey`, `transactionid`, `requesttimestamp` and `if-modified-since`. Cache the result and re-send `if-modified-since` on later calls.
3. **Fetch prices** using one of these:
   - all stations: GET /FuelPriceCheck/v2/fuel/prices
   - changes only: GET /FuelPriceCheck/v2/fuel/prices/new
   - one station: GET /FuelPriceCheck/v2/fuel/prices/station/{stationCode}
   - by location: POST /FuelPriceCheck/v2/fuel/prices/location (a read-only query with a JSON body)
   - by radius: POST /FuelPriceCheck/v2/fuel/prices/nearby (a read-only query with a JSON body)
4. **Errors**: the spec documents 400, 401 and 500. On a 401, refresh the token and retry once. A 400 means a header or body is missing or malformed. The POST lookups are queries, so retrying them is safe.
