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

## Session 2 — COMPLETE (2026-08-06): CLEAN BUILD, 0 ERRORS

### What made it work
- Installed C++/CLI support component (needed by CUETools.Codecs.TTA only)
- Installed .NET Framework 4.7 Developer Pack from dotnet.microsoft.com
  -> this was the real fix. Dev18 ships no v4.7 targeting pack out of the box.
- `msbuild CUETools.sln /t:Restore` before building (NuGet assets files)
- REVERTED the v4.7.2 retarget (commit 858e7ae). Not needed once the
  Developer Pack was installed, and it broke Freedb <-> CUETools.Processor.

### Key lesson
Most of this solution is SDK-style multi-targeting using <TargetFrameworks>
(net47;net20;netstandard2.0), NOT legacy <TargetFrameworkVersion>. The
retarget only touched 22 legacy stragglers and missed the core projects.
Search for BOTH forms next time.

### Build command (working)
$msbuild = 'F:\VisualStudio\2026\Community\MSBuild\Current\Bin\MSBuild.exe'
& $msbuild .\CUETools.sln /t:Restore /v:minimal
& $msbuild .\CUETools.sln /p:Configuration=Release /p:Platform="Any CPU" /v:minimal

### Binaries produced (F:\cuetools.net\bin\Release\net47\)
- CUETools.Ripper.Console.exe   <- candidate for app automation
- CUERipper.exe, CUERipper.WPF.exe, CUETools.exe
- plugins\CUETools.Ripper.SCSI.dll  <- direct drive access
- CUETools.AccurateRip.dll, CUETools.CTDB.dll, CUETools.CDImage.dll
- plugins\CUETools.Codecs.libFLAC.dll

### Open questions
- .vdproj installer project (CUETools.CTDB.EACPlugin.Installer) never tested.
  Don't care - want binaries, not an MSI.
- PowerShell 5.1 Set-Content strips UTF-8 BOM. Use
  [System.IO.File]::WriteAllText with New-Object System.Text.UTF8Encoding($true)

### Next session
Read CUETools.Ripper.Console\Program.cs argument parsing. Decide:
shell out to the exe, or reference CUETools.Ripper.SCSI.dll directly.

## Session 3 (2026-08-06): Ripper verified working, API understood

### Verified with a real CD
Ripped "John Barry - High Road to China" successfully.
AccurateRip: ok. MusicBrainz: auto-identified. 0 errors, 11x, 5:16.
Drive: N: HL-DT-ST BD-RE BH16NS40, read offset 6.

### What the console ripper does NOT do
- Only 8 switches, all about HOW to read: --paranoid --secure --burst
  --test --quiet --drive --offset --c2mode
- NO switch for output path, format, or per-track splitting
- Writes ONE .wav for the whole disc + a .cue + a .log, to current directory
- Filename hardcoded from MusicBrainz metadata (Program.cs line 198)
- Line 261 has a commented-out FLACWriter. Author had FLAC, disabled it.

### DECISION: Path B - reference the DLLs from my own app
Do NOT modify Program.cs. Use it as reference/example code.
My app references CUETools.Ripper.SCSI.dll etc. directly.

### The core API (Program.cs ~line 260)
audioSource.DetectGaps();
IAudioDest audioDest = new AudioEncoder(settings, destFile);
audioDest.FinalSampleCount = audioSource.Length;
while (audioSource.Read(buff, -1) != 0) {
    arVerify.Write(buff);    // AccurateRip verification
    audioDest.Write(buff);   // encoder
}
audioDest.Close();

KEY INSIGHT: per-track FLAC = swap audioDest when crossing a track
boundary from audioSource.TOC[track]. IAudioDest is an interface, so
WAV and FLAC encoders are interchangeable.

Metadata comes free: meta.artist, meta.album, meta.year,
meta.track[n].name (from MusicBrainz via CTDB).

### Next session
New C# project in VS Code. Reference the built DLLs. Get one disc
ripping to per-track FLAC in a folder I choose.
