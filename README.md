# actions

Reusable composite GitHub Actions for mobile app CI/CD.

## ios

Signs, archives, and optionally uploads an iOS app to TestFlight via Fastlane.
Supports two code-signing strategies.

## Signing modes

| Mode | How it works | When to use |
|---|---|---|
| `match` | Fetches encrypted cert + profile from a shared git repo, decrypts with `match_password` | Multiple repos share one signing identity |
| `secrets` | Decodes a raw `.p12` cert + `.mobileprovision` from GitHub secrets, imports into a temporary keychain | Single repo, or you want to avoid a signing repo dependency |

## Usage

### Match mode

```yaml
- name: Build & upload iOS to TestFlight
  uses: mieweb/actions/ios@v1
  with:
    signing_mode: match
    app_identifier: org.mieweb.os.dev
    apple_team_id: ${{ secrets.APPLE_TEAM_ID }}
    match_git_url: https://github.com/mieweb/mobile-signing
    match_git_basic_authorization: ${{ secrets.MATCH_GIT_BASIC_AUTHORIZATION }}
    match_password: ${{ secrets.MATCH_PASSWORD }}
    apple_api_key_id: ${{ secrets.APPLE_API_KEY_ID }}
    apple_api_issuer_id: ${{ secrets.APPLE_API_ISSUER_ID }}
    apple_api_key_p8_base64: ${{ secrets.APPLE_API_KEY_P8_BASE64 }}
    workspace_path: ${{ steps.xcode.outputs.workspace }}
    xcode_scheme: ${{ steps.xcode.outputs.scheme }}
    profile_name: "match AppStore org.mieweb.os.dev"
```

### Secrets mode

```yaml
- name: Build & upload iOS to TestFlight
  uses: mieweb/actions/ios@v1
  with:
    signing_mode: secrets
    app_identifier: org.mieweb.opensource
    apple_team_id: ${{ secrets.APPLE_TEAM_ID }}
    ios_cert_p12_base64: ${{ secrets.IOS_DIST_CERT_P12_BASE64 }}
    ios_cert_password: ${{ secrets.IOS_DIST_CERT_PASSWORD }}
    ios_prov_profile_base64: ${{ secrets.IOS_PROVISIONING_PROFILE_BASE64 }}
    apple_api_key_id: ${{ secrets.APPLE_API_KEY_ID }}
    apple_api_issuer_id: ${{ secrets.APPLE_API_ISSUER_ID }}
    apple_api_key_p8_base64: ${{ secrets.APPLE_API_KEY_P8_BASE64 }}
    workspace_path: ${{ steps.xcode.outputs.workspace }}
    xcode_scheme: ${{ steps.xcode.outputs.scheme }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `signing_mode` | **yes** | — | `match` or `secrets` |
| `app_identifier` | **yes** | — | Bundle ID (e.g. `org.mieweb.os.dev`) |
| `apple_team_id` | **yes** | — | Apple Developer Team ID |
| `match_git_url` | if match | — | HTTPS URL of the match signing repo |
| `match_git_basic_authorization` | if match | — | Base64 `user:token` for the signing repo |
| `match_password` | if match | — | Encryption passphrase for match |
| `match_type` | | `appstore` | Match profile type |
| `match_readonly` | | `true` | Never create/renew certs in CI |
| `match_storage_mode` | | `git` | Match storage backend |
| `ios_cert_p12_base64` | if secrets | — | Base64 distribution cert (.p12) |
| `ios_cert_password` | if secrets | — | Password for the .p12 |
| `ios_prov_profile_base64` | if secrets | — | Base64 provisioning profile |
| `apple_api_key_id` | **yes** | — | App Store Connect API Key ID |
| `apple_api_issuer_id` | **yes** | — | App Store Connect Issuer ID |
| `apple_api_key_p8_base64` | **yes** | — | Base64 API key (.p8) |
| `lane` | | `ios_build_upload` | Fastlane lane to execute |
| `workspace_path` | **yes** | — | Path to `.xcworkspace` |
| `xcode_scheme` | **yes** | — | Xcode scheme name |
| `build_configuration` | | `Release` | Xcode build configuration |
| `export_method` | | `app-store` | Xcode export method |
| `profile_name` | if match | — | Profile name (e.g. `match AppStore org.example.app`) |
| `upload_to_testflight` | | `true` | Upload IPA to TestFlight |
| `ruby_version` | | `3.4` | Ruby version for Fastlane |
| `setup_ruby` | | `true` | Install Ruby + gems (false if caller handles it) |

## Org-level secrets (recommended)

Set shared signing secrets at the **GitHub org level** so every repo inherits
them automatically. The action doesn't care whether a secret comes from org or
repo scope — it just receives the value via `with:`.

Repos can override with repo-level secrets when they need a different cert or
profile (e.g. a different bundle ID).

## Important notes

- **`match_readonly` should always be `true` in CI.** Only set to `false` for
  one-time local seeding of the match signing repo.
- **Xcode must be selected before calling this action.** The action does not
  install or select Xcode — the caller workflow should pin the Xcode version
  with `sudo xcode-select -s /Applications/Xcode_X.Y.app/Contents/Developer`.
- **The `.xcworkspace` must already exist.** Build your native project (e.g.
  `meteor build`, `pod install`) before invoking this action.
