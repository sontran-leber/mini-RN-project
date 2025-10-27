# mini React Native project

## Introduce

Mini React Native project initialized by Expo


## Run project

1.Install NPM packages

```bash
npm install
# or
yarn
```

2.Run locally

```bash
npm run start
# or
yarn start
```
# Logging Policy Documentation

## Overview

This React Native application implements a comprehensive logging and error handling strategy that focuses on user experience, network resilience, and debugging capabilities. The logging policy is designed to provide meaningful feedback to users while maintaining minimal console output for production environments.

## Logging Architecture

### 1. Error Handling Hierarchy

The application uses a multi-layered error handling approach:

#### **Global Error Boundary**

- **Location**: `App.tsx` (lines 42-43)
- **Implementation**: React Error Boundary wrapping the entire navigation stack
- **Fallback Component**: `ErrorFallback.tsx`
- **Purpose**: Catches unhandled JavaScript errors and prevents app crashes
- **Logging**: Currently disabled (commented out `logError` function)

```typescript
// Currently disabled logging in ErrorBoundary
// export const logError = (error: Error, info: ErrorInfo) => {
//   console.log("Error:", error?.message);
//   console.log("Component Stack:", info?.componentStack);
// };
```

#### **API Error Interceptor**

- **Location**: `api/axiosconfig.ts` (lines 9-17)
- **Implementation**: Axios response interceptor
- **Purpose**: Normalizes all API errors into a consistent format
- **Logging**: **ACTIVE** - Logs normalized errors for debugging

```typescript
axiosInstance.interceptors.response.use(
  response => response,
  error => {
    const formatted = formatAxiosError(error);
    const normalizedError = Object.assign(new Error(formatted.message), formatted);
    console.log("This is the normalized error processed by interceptor: ", normalizedError);
    return Promise.reject(normalizedError);
  },
);
```

#### **Component-Level Error Handling**

- **Components**: `ErrorView.tsx`, `ErrorFallback.tsx`
- **Purpose**: User-facing error displays with retry mechanisms
- **Logging**: No console logging, focuses on user experience

### 2. Network Status Logging

#### **Network Context**

- **Location**: `context/NetworkContext.tsx`
- **Purpose**: Monitors network connectivity status
- **Logging**: **NONE** - Pure state management
- **User Feedback**: Visual indicators via `NetworkBanner.tsx`

#### **Query Client Configuration**

- **Location**: `config/queryclient.ts`
- **Purpose**: Configures React Query retry and caching behavior
- **Logging**: **NONE** - Configuration only
- **Network Integration**: Integrates with `@react-native-community/netinfo`

### 3. User Feedback Systems

#### **Toast Notifications**

- **Component**: `FormSubmittedToast.tsx`
- **Purpose**: Success feedback for form submissions
- **Logging**: **NONE** - Visual feedback only

#### **Error Modals**

- **Component**: `SubmissionNotification.tsx`
- **Purpose**: Error feedback with retry/cancel options
- **Logging**: **NONE** - User interaction focused

#### **Network Banners**

- **Component**: `NetworkBanner.tsx`
- **Purpose**: Real-time network status indicators
- **Logging**: **NONE** - Visual status only

### 4. Data Persistence Logging

#### **Cache Restoration**

- **Location**: `App.tsx` (lines 26-37)
- **Purpose**: Logs successful cache restoration from AsyncStorage
- **Logging**: **ACTIVE** - Development/debugging information

```typescript
restorePromise.then(() => {
  console.log("Cache restored from AsyncStorage");
});
```

#### **Queue Management**

- **Location**: `api/queue.ts`
- **Purpose**: Manages offline form submissions
- **Logging**: **NONE** - Silent operation

## Current Logging Patterns

### Active Console Logging

1. **API Error Interceptor** (`api/axiosconfig.ts:14`)
   - Logs all normalized API errors
   - Format: `"This is the normalized error processed by interceptor: " + normalizedError`
   - Purpose: Debugging API failures

2. **Cache Restoration** (`App.tsx:34`)
   - Logs successful AsyncStorage cache restoration
   - Format: `"Cache restored from AsyncStorage"`
   - Purpose: Confirming cache persistence

### Commented/Disabled Logging

1. **Error Boundary Logging** (`components/ErrorFallback.tsx:18-21`)
   - Currently commented out
   - Would log error messages and component stack traces
   - Purpose: Debugging unhandled errors

2. **Query State Logging** (`screens/HomeScreen.tsx:55`)
   - Currently commented out
   - Would log React Query states
   - Purpose: Debugging data fetching

## Error Classification System

### Unified Error Format

- **Location**: `config/errors.ts`
- **Type**: `UnifiedApiError`
- **Purpose**: Standardizes error information across the application

```typescript
export type UnifiedApiError = {
  message: string;
  status?: number;
  isNetworkError?: boolean;
  isTimeOut?: boolean;
  isCanceled?: boolean;
  original?: any;
};
```

### Error Categories

1. **Network Errors**
   - Detected by: `ERR_NETWORK` code or network-related messages
   - User Feedback: Network banner + retry options
   - Logging: Via API interceptor

2. **Timeout Errors**
   - Detected by: `ECONNABORTED`, `ERR_CANCELED`, or timeout messages
   - User Feedback: Timeout-specific error messages
   - Logging: Via API interceptor

3. **Server Errors**
   - Detected by: HTTP status codes (404, 5xx)
   - User Feedback: Status-specific error messages
   - Logging: Via API interceptor

4. **Validation Errors**
   - Detected by: Form validation rules
   - User Feedback: Inline form error messages
   - Logging: **NONE** - Client-side validation

## Offline-First Logging Strategy

### Queue Management

- **Purpose**: Handles form submissions when offline
- **Storage**: AsyncStorage with timestamp
- **Logging**: **NONE** - Silent background operation
- **Recovery**: Automatic retry when network returns

### Cache Strategy

- **Library**: React Query with AsyncStorage persister
- **Duration**: 24 hours (`maxAge: 1000 * 60 * 60 * 24`)
- **Logging**: Cache restoration confirmation only
- **Behavior**: Automatic refetch on network reconnection

## Production Considerations

### Current State

- **Minimal Console Output**: Only 2 active console.log statements
- **User-Focused**: Emphasis on user feedback over console logging
- **Error Recovery**: Comprehensive retry mechanisms

### Recommendations

1. **Enable Error Boundary Logging** (Optional)
   - Uncomment `logError` function in `ErrorFallback.tsx`
   - Add error reporting service integration
   - Consider log levels for production vs development

2. **Add Logging Levels**
   - Implement different log levels (DEBUG, INFO, WARN, ERROR)
   - Use environment variables to control logging verbosity
   - Consider using a logging library like `react-native-logs`

3. **Error Reporting Service**
   - Integrate with services like Sentry, Bugsnag, or Crashlytics
   - Send error logs to external service instead of console
   - Include user context and device information

4. **Performance Monitoring**
   - Add performance logging for API calls
   - Monitor cache hit rates
   - Track user interaction patterns

## Security Considerations

### Data Privacy

- **No Sensitive Data**: Current logging doesn't include user data
- **Error Messages**: Sanitized error messages prevent data leakage
- **Network Status**: Only connectivity status, not network details

### Log Retention

- **Console Logs**: Not persisted (development only)
- **AsyncStorage**: Local device storage only
- **No External Transmission**: Current implementation doesn't send logs externally

## Testing and Debugging

### Test Coverage

- **Location**: `AddingPostForm.test.tsx`
- **Status**: Currently commented out
- **Focus**: Form validation error handling
- **Logging**: No logging in test scenarios

### Debug Tools

- **React Query DevTools**: Available for data fetching debugging
- **Network Inspector**: Built-in React Native debugging
- **Console Logs**: Minimal but targeted for API errors

## Summary

This application implements a **user-centric logging policy** that prioritizes:

1. **User Experience**: Comprehensive error handling with retry mechanisms
2. **Network Resilience**: Offline-first approach with queue management
3. **Minimal Console Output**: Only essential debugging information
4. **Error Recovery**: Multiple fallback strategies for different error types

The logging strategy balances debugging needs with production performance, focusing on user feedback rather than verbose console output. The system is designed to be resilient to network issues while providing clear feedback to users about application state.

## IDE

Tools for development application

| Name               | Website                                  |
| ------------------ | ---------------------------------------- |
| Visual Studio Code | [https://code.visualstudio.com/download] |
