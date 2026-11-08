# Code workspace

This folder contains the Piper/Recipes projects.

## Main web repository

Use the deployable overlay here:

```sh
cd ~/code/shopping-recipes-worktrees/shortcut-hints
```

`shopping-recipes-worktrees/shortcut-hints-source` is a full Mealie source checkout used for source work. Normal Piper Recipes deployment uses `shortcut-hints`, where existing-source changes live in `mealie-web/piper.patch` and new files live in `mealie-web/overrides/`.

## Inspect and sync

```sh
git status --short --branch
git fetch origin main
git log --oneline --decorate -5 origin/main
git rev-parse origin/main
git merge origin/main
```

Publish a ready release to main:

```sh
git push origin HEAD:main
```

## Prepare a clean pinned Mealie checkout

Recipes is an overlay on the exact revision in `mealie-web/upstream.json`. Do not apply the patch to a dirty or already-prepared checkout.

```sh
cd ~/code/shopping-recipes-worktrees/shortcut-hints
REV=$(python3 -c 'import json; print(json.load(open("mealie-web/upstream.json"))["revision"])')
rm -rf /private/tmp/mealie-piper-build
git clone https://github.com/mealie-recipes/mealie.git /private/tmp/mealie-piper-build
git -C /private/tmp/mealie-piper-build checkout "$REV"
python3 scripts/prepare-recipes-web.py /private/tmp/mealie-piper-build
```

## Build the frontend

Use Node 24 and pnpm 11.23.0:

```sh
export PATH="$HOME/.nvm/versions/node/v24.22.2/bin:$PATH"
cd /private/tmp/mealie-piper-build/frontend
pnpm install --frozen-lockfile
pnpm exec nuxt prepare
pnpm test:ci
pnpm generate
```

The deployable build is `/private/tmp/mealie-piper-build/frontend/.output/public` after `pnpm generate`.

## Manual production deployment

First set the real VPS host. Do not use the local Mac hostname:

```sh
export VPS_HOST="YOUR_VPS_IP_OR_HOSTNAME"
ssh "deploy@$VPS_HOST" 'hostname && test -d /opt/mealie'
```

Deploy the current commit:

```sh
cd ~/code/shopping-recipes-worktrees/shortcut-hints
export HOST="deploy@$VPS_HOST"
export GITHUB_SHA="$(git rev-parse HEAD)"
bash scripts/deploy-recipes-web.sh /private/tmp/mealie-piper-build/frontend/.output/public
```

The script uploads a versioned release, builds the Docker image on the VPS, switches the active release, restarts only Mealie, and rolls back automatically if its health check fails. It does not reset the database or volumes.

## Verify production

```sh
cd ~/code/shopping-recipes-worktrees/shortcut-hints
python3 scripts/verify-recipes.py \
  --public-url https://recipes.piper.sydney \
  --evidence verification/runs/manual-$(git rev-parse --short HEAD)
```

Then open:

```text
https://recipes.piper.sydney/g/home
```

Verify the signed-in UI at desktop `1440x900`, phone `390x844`, and narrow phone `320x740` when the change is responsive.

## GitHub Actions deployment

```sh
gh workflow enable deploy-recipes-web.yml
git push origin HEAD:main
gh run list --workflow deploy-recipes-web.yml --limit 3
gh run view RUN_ID --verbose
gh run watch RUN_ID --exit-status
```

If Actions fails before starting, inspect the run. Account billing or spending-limit problems must be fixed in GitHub before Actions can deploy; use the manual SSH deploy above instead.

## Rollback

On the VPS:

```sh
ssh "deploy@$VPS_HOST"
cd /opt/mealie
ls -1 mealie-web/releases
ln -sfn /opt/mealie/mealie-web/releases/GOOD_SHA mealie-web/current.next
mv -Tf mealie-web/current.next mealie-web/current
printf 'PIPER_RELEASE=%s\n' GOOD_SHA > mealie-web/current.env
bash compose-mealie.sh up -d --no-deps mealie
```

Only roll back to a release that includes the custom backend required for personal-list privacy.

## Useful checks

```sh
python3 scripts/check-web.py
python3 -m unittest scripts/test_check_recipes_aasa.py scripts/test_generate_recipes_aasa.py
git diff --check
git status --short
```

## Exact commands used for the recent release

```sh
git fetch origin main
git push origin HEAD:main
python3 scripts/check-web.py
python3 -m unittest scripts/test_check_recipes_aasa.py scripts/test_generate_recipes_aasa.py
python3 scripts/prepare-recipes-web.py /private/tmp/piper-clean-verify
gh workflow enable deploy-recipes-web.yml
gh workflow run deploy-recipes-web.yml --ref main
gh run watch RUN_ID --exit-status
python3 scripts/verify-recipes.py --public-url https://recipes.piper.sydney --evidence verification/runs/release-78202f1
```

## iOS Recipes/Piper app

The native app repository is:

```sh
cd ~/code/mealie-pocket
```

The deploy helper is already configured for the Recipes/Piper branding and is the preferred way to install a Debug build on a connected iPhone.

### Check the phone and signing setup

Connect one iPhone by USB, unlock it, trust this Mac, enable Developer Mode, and sign into Xcode with the Apple ID/team. Then run:

```sh
cd ~/code/mealie-pocket
python3 scripts/deploy-iphone.py --dry-run
```

If more than one phone is connected:

```sh
python3 scripts/deploy-iphone.py --dry-run --device PHONE_UDID
```

### Build and install directly on an iPhone

```sh
cd ~/code/mealie-pocket
python3 scripts/deploy-iphone.py
```

The script detects the connected iPhone, builds `MealiePocket` with `xcodebuild`, installs it with `xcrun devicectl`, and launches it. To install without launching:

```sh
python3 scripts/deploy-iphone.py --no-launch
```

To use a specific checkout:

```sh
python3 scripts/deploy-iphone.py --root ~/code/mealie-pocket-worktrees/main-latest
```

To use a paid Apple Developer team and keep widget App Groups enabled:

```sh
python3 scripts/deploy-iphone.py --team YOUR_TEAM_ID --app-groups
```

The default personal-team build disables widget App Group sharing because free provisioning does not support the required entitlement.

### Simulator build and tests

```sh
cd ~/code/mealie-pocket
bash scripts/check-unit-tests.sh
xcodebuild -project MealiePocket.xcodeproj \
  -scheme MealiePocket \
  -destination 'platform=iOS Simulator,name=iPhone 18 Pro' \
  CODE_SIGNING_ALLOWED=NO test
```

Use an installed simulator name if `iPhone 18 Pro` is unavailable:

```sh
xcrun simctl list devices available
```

Open the project in Xcode when you need to change signing, inspect build errors, or run a live preview:

```sh
open ~/code/mealie-pocket/MealiePocket.xcodeproj
```

### Useful iPhone troubleshooting

```sh
xcrun devicectl list devices
xcodebuild -project MealiePocket.xcodeproj -scheme MealiePocket -showBuildSettings
```

Common fixes:

- Unlock the phone and reconnect it by USB.
- Tap **Trust** on the phone when prompted.
- Enable **Developer Mode** under Privacy & Security.
- Open Xcode → Settings → Accounts and sign in.
- Use `--device UDID` if multiple phones are connected.
- Use `--team TEAM_ID` if the default signing team is not yours.
- Run without `--app-groups` for a free personal team.
