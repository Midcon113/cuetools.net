# CUETools fork — session notes

## Purpose
Learning exercise: build CUETools from source. Real goal is to find out whether
CUETools.Ripper.Console can be driven programmatically by the soundtrack app.

## Environment
- Repo: F:\cuetools.net (fork of gchudov/cuetools.net)
- Remotes: origin = Midcon113 fork, upstream = gchudov (fetch only)
- Upstream default branch is `master`, not `main`
- VS Community 2026 (Dev18, 18.8.2) at F:\VisualStudio\2026\Community
- MSBuild: F:\VisualStudio\2026\Community\MSBuild\Current\Bin\MSBuild.exe
- C++ compilers present (VC\Tools\MSVC confirmed)
- VS Community license expires 2026-10-21 unless signed in

## Session 1 — COMPLETE (2026-08-03)
- Forked, cloned, upstream remote added
- 5 submodules initialized: WavPack, WindowsMediaLib, flac, openclnet, taglib-sharp
- MAC_SDK extracted from vendored zip (not a submodule)
- All 5 patches applied, exit code 0 each
- Verified: 4 submodules show ` m`, MAC_SDK files show `??`. Both expected.

## Open questions
1. Is C++/CLI support installed? Check never run:
   vswhere -latest -requires Microsoft.VisualStudio.Component.VC.CLI.Support
   Likely needed for native codec interop.
2. No v4.7 targeting pack in Dev18 (list jumps 4.6.1 -> 4.7.2).
   21 projects target v4.7. Options: retarget to 4.7.2 (preferred),
   install 4.7 Developer Pack, or let VS auto-retarget.
3. Does the Installer Projects extension (.vdproj) exist for Dev18?
   Probably don't care — want binaries, not an MSI.

## Also noted
- Two stale outliers: MusicBrainz.csproj (v1.1), CUETools.CTDB.EACPlugin.csproj (v2.0).
  Leave alone until a build error points there.
- 3 native .vcxproj: CUETools.AVX, CUETools.Codecs.TTA, ttalib-1.1. Unaffected by retarget.
- Never edit anything under ThirdParty\ — separate repos with patches applied.

## Next session
Baseline commit, scripted v4.7 -> v4.7.2 replace across 21 .csproj files,
then first build attempt. Success = categorized error list, not a working build.
