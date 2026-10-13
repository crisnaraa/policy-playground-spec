# Policy Playground — Component Methods

---

## Shell Application (`shell-app`)
| Method | Signature | Description |
|---|---|---|
| `bootstrap()` | `bootstrap(): void` | Initializes the shell, loads Cognito tokens, registers MFE routes |
| `loadMFE(name, config)` | `loadMFE(name: string, config: MFEConfig): Promise<void>` | Dynamically loads a micro-frontend SPA module |
| `getAuthContext()` | `getAuthContext(): AuthContext` | Returns current user identity and role from Cognito |
| `navigateTo(route)` | `navigateTo(route: string): void` | Routes user to a specific MFE feature |

---

## Policy Explorer MFE (`policy-explorer-mfe`)
| Method | Signature | Description |
|---|---|---|
| `listPolicies(filters)` | `listPolicies(filters: PolicyFilter): Promise<PolicyVersion[]>` | Fetches paginated list of available policy versions |
| `getPolicyVersion(id, version)` | `getPolicyVersion(id: string, version: string): Promise<PolicyDetail>` | Returns full policy metadata and script content |
| `comparePolicyVersions(id, v1, v2)` | `comparePolicyVersions(id: string, v1: string, v2: string): Promise<PolicyDiff>` | Returns diff between two policy versions |
| `submitSimulation(policyId, params)` | `submitSimulation(policyId: string, params: SimParams): Promise<JobRef>` | Submits a policy simulation job |

---

## Simulation Studio MFE (`simulation-studio-mfe`)
| Method | Signature | Description |
|---|---|---|
| `createSession(config)` | `createSession(config: SessionConfig): Promise<Session>` | Creates a new simulation session |
| `pollJobStatus(jobId)` | `pollJobStatus(jobId: string): Promise<JobStatus>` | Polls the job status endpoint every N seconds |
| `cancelJob(jobId)` | `cancelJob(jobId: string): Promise<void>` | Sends cancellation request for a running job |
| `getSessionSummary(sessionId)` | `getSessionSummary(sessionId: string): Promise<SessionSummary>` | Returns summary of the current session configuration |

---

## Results Dashboard MFE (`results-dashboard-mfe`)
| Method | Signature | Description |
|---|---|---|
| `loadResults(jobId)` | `loadResults(jobId: string): Promise<ResultSet>` | Loads simulation findings from the Results Service |
| `compareResults(jobIds)` | `compareResults(jobIds: string[]): Promise<ComparisonReport>` | Compares findings across multiple simulations |
| `generateShareLink(sessionId)` | `generateShareLink(sessionId: string): Promise<string>` | Generates a read-only shareable URL for the session |
| `exportReport(jobId, format)` | `exportReport(jobId: string, format: "csv"\|"pdf"): Promise<Blob>` | Exports the simulation report in the requested format |

---

## Admin Panel MFE (`admin-panel-mfe`)
| Method | Signature | Description |
|---|---|---|
| `listUsers()` | `listUsers(): Promise<UserProfile[]>` | Lists all users from Cognito user pool |
| `updateUserRole(userId, role)` | `updateUserRole(userId: string, role: UserRole): Promise<void>` | Updates a user's role/group in Cognito |
| `getSandboxStatus()` | `getSandboxStatus(): Promise<SandboxHealth>` | Returns health and last refresh timestamp of the sandbox DB |
| `viewAuditLog(filters)` | `viewAuditLog(filters: AuditFilter): Promise<AuditEntry[]>` | Queries audit entries from the S3 data lake via Athena |

---

## Auth Service (`auth-service`)
| Method | Signature | Description |
|---|---|---|
| `federateLogin(samlAssertion)` | `federateLogin(samlAssertion: string): Promise<TokenSet>` | Processes SAML assertion from Azure AD and issues Cognito tokens |
| `refreshToken(refreshToken)` | `refreshToken(refreshToken: string): Promise<TokenSet>` | Issues a new access token from a valid refresh token |
| `validateToken(token)` | `validateToken(token: string): Promise<Claims>` | Validates JWT and extracts user claims |
| `getUserRoles(userId)` | `getUserRoles(userId: string): Promise<string[]>` | Returns the list of roles assigned to a user from Cognito groups |

---

## Policy Service (`policy-service`)
| Method | Signature | Description |
|---|---|---|
| `syncFromCodeCommit(event)` | `syncFromCodeCommit(event: PipelineEvent): Promise<void>` | Receives CodePipeline trigger and syncs new policy version to DynamoDB |
| `listVersions(policyId)` | `listVersions(policyId: string): Promise<PolicyVersion[]>` | Returns all versions of a given policy |
| `getScript(policyId, version)` | `getScript(policyId: string, version: string): Promise<string>` | Returns the raw SQL/Python script content for a policy version |
| `validatePolicy(script)` | `validatePolicy(script: string): Promise<ValidationResult>` | Runs schema and syntax validation on a policy script |

---

## Simulation Service (`simulation-service`)
| Method | Signature | Description |
|---|---|---|
| `submitJob(request)` | `submitJob(request: JobRequest): Promise<JobRef>` | Validates request and spawns an ECS Task for execution |
| `getJobStatus(jobId)` | `getJobStatus(jobId: string): Promise<JobStatus>` | Returns current status of a simulation job from DynamoDB |
| `listJobsBySession(sessionId)` | `listJobsBySession(sessionId: string): Promise<JobRef[]>` | Returns all jobs belonging to a given session |
| `cancelJob(jobId)` | `cancelJob(jobId: string): Promise<void>` | Stops the ECS Task and marks job as CANCELLED in DynamoDB |
| `onJobComplete(jobId, result)` | `onJobComplete(jobId: string, result: JobResult): Promise<void>` | Callback invoked by Job Executor; updates DynamoDB and triggers audit event |

---

## Job Executor (`job-executor`)
| Method | Signature | Description |
|---|---|---|
| `initialize(jobConfig)` | `initialize(jobConfig: JobConfig): Promise<void>` | Loads policy script and sandbox DB credentials from environment/SSM |
| `executeSQLPolicy(script, params)` | `executeSQLPolicy(script: string, params: ExecParams): Promise<ResultSet>` | Runs SQL script against the sandbox database |
| `executePythonPolicy(script, params)` | `executePythonPolicy(script: string, params: ExecParams): Promise<ResultSet>` | Executes Python policy script in an isolated subprocess |
| `writeResults(jobId, results)` | `writeResults(jobId: string, results: ResultSet): Promise<void>` | Serializes results to Parquet/JSON and writes to S3 |
| `reportCompletion(jobId, status)` | `reportCompletion(jobId: string, status: JobStatus): Promise<void>` | Notifies Simulation Service of job completion/failure |

---

## Results Service (`results-service`)
| Method | Signature | Description |
|---|---|---|
| `getResults(jobId)` | `getResults(jobId: string): Promise<ResultSet>` | Queries Athena for simulation results from S3 |
| `compareResults(jobIds)` | `compareResults(jobIds: string[]): Promise<ComparisonReport>` | Executes Athena query to compare multiple result sets |
| `createShareToken(sessionId, userId)` | `createShareToken(sessionId: string, userId: string): Promise<string>` | Creates a read-only share token in DynamoDB with TTL |
| `validateShareToken(token)` | `validateShareToken(token: string): Promise<SessionRef>` | Validates and resolves a share token to a session |

---

## Audit Service (`audit-service`)
| Method | Signature | Description |
|---|---|---|
| `publishEvent(event)` | `publishEvent(event: AuditEvent): Promise<void>` | Publishes an audit event to the SQS ingestion queue |
| `ingestFromQueue()` | `ingestFromQueue(): Promise<void>` | Reads batched events from SQS and writes to S3 data lake |
| `queryAuditLog(filters)` | `queryAuditLog(filters: AuditFilter): Promise<AuditEntry[]>` | Queries audit entries via Athena over S3 |
