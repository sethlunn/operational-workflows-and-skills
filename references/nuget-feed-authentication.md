# NuGet Feed Authentication

Use this reference when a `dotnet` restore, build, or test against the private QuadPay Azure Artifacts
feed fails to authenticate. A feed `401` is a recoverable local credential problem, not a blocker to
report back to the user.

## Symptom

```text
error NU1301: Unable to load the service index for source
  https://pkgs.dev.azure.com/quadpay/_packaging/QuadPay/nuget/v3/index.json.
error NU1301:   Response status code does not indicate success: 401 (Unauthorized).
warning : The plugin credential provider could not acquire credentials.
```

## Cause

The local `az` session still shows a signed-in user, but its multi-factor claim has expired:

```text
AADSTS50078: Presented multi-factor authentication has expired due to policies configured by
your administrator, you must refresh your multi-factor authentication to access
'499b84ac-1321-427f-aa17-267ca6975798'.
```

`499b84ac-1321-427f-aa17-267ca6975798` is the fixed Azure DevOps resource ID.

## Fix

Refresh the MFA claim, then mint a token and hand it to NuGet:

```bash
# 1. Opens an auth page in the browser for the user to complete.
az login --tenant "fefb8bdf-9eeb-4659-96e1-490e8d20b96e" \
         --scope "499b84ac-1321-427f-aa17-267ca6975798/.default"

# 2. Mint the token and restore in a single shell invocation.
export VSS_NUGET_EXTERNAL_FEED_ENDPOINTS="{\"endpointCredentials\":[{\"endpoint\":\"https://pkgs.dev.azure.com/quadpay/_packaging/QuadPay/nuget/v3/index.json\",\"username\":\"seth.lunn@zip.co\",\"password\":\"$(az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798 --query accessToken -o tsv)\"}]}"
dotnet restore <project-or-solution>
```

Rules:

- Run `az login` as a background command and tell the user an auth page is opening, rather than asking
  them to run a command themselves.
- Run these with the sandbox disabled — both steps need network, and `az login` needs keychain access.
- Shell state does not persist between tool calls. Re-export `VSS_NUGET_EXTERNAL_FEED_ENDPOINTS` in
  every subsequent `dotnet` invocation, including `build` and `test`.
- Expand the token inline. Never echo it into output; print only its length when confirming.
- Treat a residual `NU1900` warning about package vulnerability data as benign. It reflects the audit
  source query, not the code under change.

## Dead ends

Do not spend turns re-testing these; all were confirmed to fail:

- Re-running restore with the sandbox disabled — still `401`. The sandbox is not the cause.
- `~/.nuget/NuGet/NuGet.Config` — holds `cloudsmith` credentials only, nothing for `QuadPayArtifacts`.
- The credential provider session cache at
  `~/Library/Application Support/MicrosoftCredentialProvider/SessionTokenCache.dat` — present but empty.
- The Rider keychain entry `RiderNuGetCredentialsRepo::<feed-url>` — read is denied outside Rider.
- `dotnet restore --interactive` device-code flow — works in principle, but Seth considers it a dated
  approach and expects the browser-based `az login` path instead.
