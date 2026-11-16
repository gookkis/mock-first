---
description: Create the app's own upload key (or reuse the existing one), wire it into the release build through local.properties, and prove the release bundle is signed with it
allowed-tools: Read, Edit, Write, Glob, Grep, Bash(date*), Bash(cat*), Bash(ls*), Bash(mkdir*), Bash(grep*), Bash(git check-ignore*), Bash(*keytool*), Bash(openssl*), Bash(sh ./gradlew*), Bash(bash gradlew*), Bash(./gradlew*), Bash(find*)
---

# Signing — one upload key per app, wired and verified

Play App Signing holds the app signing key; what you keep is the **upload key**. This
command makes one per app in the publisher's key folder, wires it into the build without
committing a secret, and checks the bundle actually carries it. A bundle signed with the
debug key, or with another app's key, is rejected at upload — this is the cheapest place to
catch it.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path or module name. |
| `--verify` | Only Step 5: build and check the fingerprint. Changes nothing. |
| `--alias <name>` | Alias for a new key. Default `upload`. |

## Absolute rules

- **Never re-key an app that is already on Play.** If the Console already has an upload key
  for this package (the app is live, or a key is recorded for it), reuse that key and alias
  as they are. A new upload key for a live app needs a key-reset request the owner files.
- **Secrets never enter git.** Passwords and the keystore path go in the root
  `local.properties`, which `git check-ignore local.properties` must confirm. Never in
  `gradle.properties`, a committed file, a commit message, or the report.
- **Never print a password.** Report the alias, the file path and the SHA-256 fingerprint.
- Run `date +%Y-%m-%d` for the date.

## Step 0 — Publisher settings

Read `~/.claude/mock-first/publisher.yml` if it exists (shared by every play-store and
android-factory command; schema in `/android-factory:build-app`). Used here: `keys_dir`,
`keys_readme`, `dname`, `key_validity_days` (default 10000). Without the file, ask once for
the key folder and the distinguished name, and offer to write the file.

## Step 1 — Tools

`keytool` is often not on PATH on a machine that only has Android Studio. Look in order:
`$JAVA_HOME/bin/keytool`, `~/android-studio/jbr/bin/keytool` (Linux),
`/Applications/Android Studio.app/Contents/jbr/Contents/Home/bin/keytool` (macOS),
`%ProgramFiles%\Android\Android Studio\jbr\bin\keytool.exe` (Windows). Use the full path from
here on, and export `JAVA_HOME` to the same JBR for every Gradle call — an unset `JAVA_HOME`
fails the build, and a "verification" that then reads a stale bundle passes on the wrong file.

## Step 2 — Find an existing key

1. `local.properties`: `RELEASE_STORE_FILE` set and the file exists → reuse; go to Step 4.
2. `keys_dir`: a folder for this app (`<slug> key/`, or a README row with this package) →
   reuse its keystore, alias and credentials file; go to Step 4.
3. Otherwise create one (Step 3).

## Step 3 — Create the key

Folder `<keys_dir>/<slug> key/`, files:

```
<slug>-upload.jks                 PKCS12, RSA 2048, alias from --alias
<slug>-upload_certificate.pem     exported certificate (for the Console / key-reset)
<slug>-upload.credentials.txt     store + key password, alias, fingerprint, date
```

1. Generate one strong password (`openssl rand -base64 24`), write the credentials file
   **first**, then run
   `keytool -genkeypair -storetype PKCS12 -keystore <jks> -alias <alias> -keyalg RSA
   -keysize 2048 -validity <days> -dname "<dname>" -storepass <pw> -keypass <pw>`.
2. **Check the `.jks` exists and is not empty.** A failed keytool call still leaves the
   credentials file and an empty `.pem` behind; if so, rerun with the same password.
3. `keytool -exportcert -rfc -keystore <jks> -alias <alias> -file <pem>`.
4. Read the SHA-256: `keytool -list -v -keystore <jks> -alias <alias>`.
5. Add a row to `keys_readme` in its own column style (app, package, status, file/alias,
   SHA-256, date). Remind the owner that the folder needs a backup off this machine — losing
   the upload key means a reset request and days of waiting.

## Step 4 — Wire it into the build

1. Root `local.properties` (append, in the file's style, one comment line per app):
   ```properties
   # <App> upload key, <date>
   RELEASE_STORE_FILE=/absolute/path/<slug>-upload.jks
   RELEASE_STORE_PASSWORD=…
   RELEASE_KEY_ALIAS=upload
   RELEASE_KEY_PASSWORD=…
   ```
2. The module's `build.gradle(.kts)` must read them with a property-first fallback and
   apply the config only when the file is set, so a machine without the key still builds:
   ```kotlin
   fun secret(k: String) = (project.findProperty(k) as String?) ?: localProperties.getProperty(k)
   signingConfigs {
       create("release") {
           secret("RELEASE_STORE_FILE")?.let { path ->
               storeFile = file(path)
               storePassword = secret("RELEASE_STORE_PASSWORD")
               keyAlias = secret("RELEASE_KEY_ALIAS")
               keyPassword = secret("RELEASE_KEY_PASSWORD")
           }
       }
   }
   buildTypes { release { if (secret("RELEASE_STORE_FILE") != null) signingConfig = signingConfigs.getByName("release") } }
   ```
   In an android-factory project the convention plugin owns this; change it there, not per app.

## Step 5 — Verify

1. `JAVA_HOME=<jbr> ./gradlew :<app>:bundleRelease` (`sh ./gradlew` when the wrapper has no
   executable bit).
2. `keytool -printcert -jarfile <module>/build/outputs/bundle/release/*.aab` → the SHA-256
   must equal the key's. `CN=Android Debug` or another app's fingerprint is a stop.
3. Print the bundle size. **Over 10 MB** means the browser upload in `/play-store:release`
   cannot carry it; say so now.

## Step 6 — Report

Alias, keystore path, SHA-256, README row added (yes/no), `local.properties` keys written,
bundle path + size + fingerprint match. Next: `/play-store:admob <app>` (or
`/play-store:release <app>` when ads are already wired).
