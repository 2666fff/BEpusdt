# Fungame BEPUSDT Patches

This branch carries the Fungame payment fixes on top of the official
`v03413/bepusdt` release.

## Remotes

```text
origin   https://github.com/2666fff/BEpusdt.git
upstream https://github.com/v03413/bepusdt.git
```

`origin` is the Fungame fork. `upstream` is the official project.

## Branch

```text
fungame-tronfix-1.23.6
```

Base release:

```text
v1.23.6 / eeb7ac1
```

## Patches

1. `app/task/tron.go`

   Official `v1.23.6` filters TRON transactions with
   `trans.Result.Result`. Some public FullNode responses report this field as
   false even when the transaction `ret[].contractRet` is `SUCCESS`.

   The Fungame patch adds `transactionSucceeded()` and prefers
   `trans.GetTransaction().GetRet()[].ContractRet == SUCCESS`, falling back to
   the old field only when `ret` is absent.

2. `app/model/registry.go`

   Official `v1.23.6` hard-codes the TRX block-scan minimum amount as
   `0.1 TRX`. Fungame has valid small TRX orders such as `0.03` and `0.08`,
   so this branch lowers the TRX scan minimum to `0.000001 TRX`.

3. `app/version.go`

   The runtime version string is set to
   `v1.23.6-fungame-tronfix-20260602` so production binaries can be identified
   quickly.

## Build

```powershell
cd E:\repo\bepusdt-src
corepack enable
corepack prepare pnpm@10.25.0 --activate
cd web
pnpm install --frozen-lockfile
pnpm run build:prod
cd ..
Copy-Item -Path .\web\dist\* -Destination .\static\secure -Recurse -Force
gofmt -w .\app\task\tron.go .\app\model\registry.go .\app\version.go
go test .\app\model .\app\task
$env:GOOS='linux'
$env:GOARCH='amd64'
$env:CGO_ENABLED='0'
go build -o .\dist\bepusdt-linux-amd64-trx-minfix .\main
```

Do not build without the web assets, otherwise the binary may fail at runtime
because `static/secure/secure.html` is missing.

## Upgrade Workflow

When the official project publishes a new version:

```powershell
cd E:\repo\bepusdt-src
git fetch upstream --tags
git switch -c fungame-tronfix-<official-version> <official-tag>
git cherry-pick 2b2b9c1
```

If the cherry-pick conflicts, inspect these files first:

```text
app/task/tron.go
app/model/registry.go
app/version.go
```

Before shipping, verify:

```powershell
go test .\app\model .\app\task
.\dist\bepusdt-linux-amd64-trx-minfix version
```

Then replace `fungame.ovh/bepusdt/bepusdt`, commit the binary in
`fungame.ovh`, push it, and pull on the server.
