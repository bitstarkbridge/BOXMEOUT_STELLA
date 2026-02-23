# PR: Implement Transaction Retry with Exponential Backoff

## Description
This PR implements a robust transaction monitoring and retry mechanism for all blockchain-related services in the BoxMeOut Stella backend.

### Key Changes
- **BaseBlockchainService**: A new abstract base class that centralizes RPC configuration and transaction monitoring logic.
- **Exponential Backoff**: `waitForTransaction` now supports exponential backoff (1s, 2s, 4s, 8s) for `NOT_FOUND` transaction statuses.
- **Network Error Retries**: Implemented a separate retry counter (max 3) for network/RPC connection errors.
- **Dead Letter Queue (DLQ)**: Permanently failed transactions are now logged to a new `BlockchainDeadLetterQueue` database table for administrative review.
- **Service Refactoring**: `AmmService`, `FactoryService`, `OracleService`, `MarketBlockchainService`, and `TreasuryService` have been refactored to extend the base service and utilize the improved retry logic.

## Technical Details
- Added `BlockchainDeadLetterQueue` model to Prisma schema.
- Used `UPSERT` for DLQ entries to avoid duplicates.
- Centralized `waitForTransaction` logic results in cleaner, more maintainable service code.
- Added alerts (STDOUT/Logging) for critical failures.

## Verification
### Unit Tests
- `backend/tests/services/blockchain/base.test.ts`: Verified backoff timing, network retry limits, and DLQ recording.

### Integration Tests
- Verified `AmmService` (Trading API) functionality.
- Verified `OracleService` (Consensus/Attestation) functionality.
- Both test suites pass after refactoring.

## Screenshots/Recordings
- Unit test output:
```
 ✓ tests/services/blockchain/base.test.ts (4)
   ✓ BaseBlockchainService (4)
     ✓ should succeed if transaction is successful on first poll
     ✓ should retry with backoff if transaction is NOT_FOUND
     ✓ should retry network errors and eventually fail to DLQ
     ✓ should stop and fail immediately to DLQ if transaction status is FAILED
```
