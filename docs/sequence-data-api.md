---
title: Data API
layout: doc
---

# SequenceLogicData API

Public surface of `SequenceLogicData.dll` — the file format I/O library for
AB1, SRD, DAN, and SCF sequencing files.

---

## Core I/O interfaces

```csharp
// Read any supported file → SequenceResult
public interface INucleotideReader
{
    SequenceResult Read(string fileName);
}

// Write a SequenceResult back to disk (async)
public interface INucleotideWriterAsync
{
    Task WriteAsync(SequenceResult nucsPeaks, string fileName);
}
```

Both `AB1ReadWriter` and `DanReaderWriter` implement both interfaces. `SrdReader` implements only `INucleotideReader`.

---

## File format implementations

| Class | Formats | Interfaces |
|---|---|---|
| `AB1ReadWriter` | `.ab1`, `.fsa` | `INucleotideReader`, `INucleotideWriterAsync` |
| `DanReaderWriter` | `.dan` (DAN2–DAN6) | `INucleotideReader`, `INucleotideWriterAsync` |
| `SrdReader` | `.srd` (XML) | `INucleotideReader` |
| `ScfReader` | `.scf` (Staden SCF v3) | `INucleotideReader` |
| `ScfWriter` | `.scf` (Staden SCF v3) | `INucleotideWriterAsync` |

All are concrete classes with a public parameterless constructor, usable as generic type arguments (`where T : INucleotideReader, new()`).

#### SCF-specific behaviours

- **Delta encoding** — SCF v3 uses Staden 2-pass "differences of differences" encoding. `ScfReader` applies two forward accumulation passes on read; `ScfWriter` applies two backward difference passes on write.
- **Amplitude normalization** — after applying any `SCAL`/`OFFS` comments, if the overall signal maximum is below 10 000 (weak-signal runs), all four channels are scaled linearly so the maximum reaches 30 000. This matches the amplitude range AB1 files naturally carry and that SangerBasecaller's peak-detection threshold (1000 on the post-filter signal) requires.
- **Comments** — a subset of Staden comments is parsed (`NAME`, `MACH`, `MODL`, `LANE`, `STEP`, `STRT`, `DCST`, `STOP`, `SCAL`, `OFFS`). Additional instrument-specific keys from third-party files (e.g. ABI 3730xl) are intentionally ignored.

---

## ISequenceResult — narrow read-only view

Used by Storage via `ISequenceResult values = reader.Read(filePath)`.

```csharp
public interface ISequenceResult
{
    // Peaks
    IReadOnlyList<SequencePeak>? Peaks { get; }   // null when file has no analysis
    double[]? PeaksValuesA { get; }               // Gaussian envelope, ACGT order
    double[]? PeaksValuesC { get; }
    double[]? PeaksValuesG { get; }
    double[]? PeaksValuesT { get; }
    double    TimeShiftMs  { get; }               // envelope time offset

    // Raw signal (null when file contains no raw data)
    RawData? RawDataValues { get; }

    // Metadata
    string?          ProjectName          { get; }
    Sample           SampleInfo           { get; }
    Instrument       InstrumentInfo       { get; }
    Tube             TubeInfo             { get; }
    Capillary        CapillaryInfo        { get; }
    ExperimentParams ExperimentParameters { get; }
}
```

---

## SequenceResult — full mutable aggregate

Returned from `Read`, passed to `WriteAsync`. Implements `ISequenceResult`.

```csharp
public class SequenceResult : ISequenceResult
{
    // Sequence data
    public PeaksCollection          Peaks;
    public Dictionary<int,Nucleotide> Nucleotids; // position → base, keyed from 0

    // Raw fluorescence
    public RawData? RawDataValues { get; set; }
    public RawData? IntermediateData { get; set; }

    // Status
    public bool IsAnalyzed { get; }         // true when envelopes are present
    public string? CurrentFormat { get; set; } // "srd" / "dan" / "ab1" / "fsa"
    public SequenceFileFormat Format { get; }  // enum equivalent of CurrentFormat

    // Metadata
    public string?    ProjectName         { get; set; }
    public string?    User                { get; set; }
    public Sample           SampleInfo           { get; set; }
    public Instrument       InstrumentInfo       { get; set; }
    public Tube             TubeInfo             { get; set; }
    public Capillary        CapillaryInfo        { get; set; }
    public ExperimentParams ExperimentParameters { get; set; }
    public Polymer          PolymerInfo          { get; set; }
    public DyeSet           Dyes                 { get; set; }

    // Analysis parameters
    public Matrix         CorrectionMatrix        { get; set; } // 4×4, ACGT order

    // Constructors
    public SequenceResult(PeaksCollection peaks, Dictionary<int,Nucleotide> nucleotids, RawData? rawData);
    public SequenceResult(PeaksCollection peaks, Dictionary<int,Nucleotide> nucleotids);

    // Write basecaller output into this container
    public void SetAnalysisResults(
        List<SequencePeak>        peaks,
        Dictionary<int,Nucleotide> nucleotids,
        double[] envelopesA, double[] envelopesC,
        double[] envelopesG, double[] envelopesT,
        double   timeShiftMs);
}
```

---

## Domain types

### RawData

Four-channel fluorescence time-series; always in ACGT order.

```csharp
public class RawData
{
    public double[] ValuesA, ValuesC, ValuesG, ValuesT;
    public double   TimeStepSec { get; set; }   // seconds per sample
    public bool     IsEmpty     { get; }

    // Optional EPT channels
    public ChannelData? Voltage             { get; }
    public ChannelData? Temperature         { get; }
    public ChannelData? Current             { get; }
    public ChannelData? DetectorTemperature { get; }

    public RawData(int count);
    public void CreateEpt(int count, double timeStep);
}
```

### SequencePeak

One detected base-call. Per-base weight arrays allow IUPAC ambiguity codes.

```csharp
public class SequencePeak
{
    // Identity
    public int    NucleotidId;    // use NucId constants or NucleotideIdConverter
    public int    NumberInitial;  // original position index
    public int    NumberEdited;   // position after editing

    // Signal geometry
    public float  Amplitude;     // peak height
    public float  SigmaMs;       // Gaussian σ in ms
    public double TimeOutMs;     // peak centre time in ms
    public double TimeOutSec { get; }
    public double SigmaSec  { get; }

    // Quality
    public int    Quality  { get; set; }
    public int    Weight   { get; set; }

    // Flags
    public bool   IsDeleted, IsMatched, NonReal, IsChanged;

    public string Name { get; }   // single-letter IUPAC

    public double YGauss(double xSec);   // Gaussian value at time xSec
    public void   Complementary();       // flip to complement base in-place
}
```

### PeaksCollection

```csharp
public class PeaksCollection
{
    public List<SequencePeak> Peaks;
    public double   TimeShiftMs;
    public double[]? PeaksValuesA, PeaksValuesC, PeaksValuesG, PeaksValuesT;

    public int  PeaksValuesCount { get; }
    public int  Quality          { get; }  // mean Weight across all peaks

    public void Add(SequencePeak thePeak);
}
```

### Nucleotide

```csharp
public class Nucleotide
{
    public int NucleotidId;
    public int NucleotidInitialNumber;
    public int Weight;                  // quality score 0–60
    public int MatchedInitialNumber;    // -1 = not matched

    public const int NotMatched = -1;

    public Nucleotide(int nucId, int weight, int nucInitialNumber);
    public Nucleotide(int nucId, int weight);

    public static bool IsMutation(int nucId);
    public static int  GetCompliment(int nucId);
    public void        Complementary();
}
```

---

## Nucleotide ID constants

```csharp
public static class NucId
{
    public const int Undefined = -2;
    public const int Shift     = -1;
    public const int Empty     =  0;
    public const int A=1, C=2, G=3, N=4, T=5, X=6;
    public const int M=7, R=8, W=9, S=10, Y=11, K=12;
    public const int V=13, H=14, D=15, B=16;
}

public static class NucleotideIdConverter
{
    public static int    ToInt(string s);
    public static int    ToIntWithUndefined(string s);  // returns -2 on unknown
    public static string ToStr(int i);                  // single-letter IUPAC
}
```

---

## Channel ordering

All arrays and matrices inside `SequenceResult` use **ACGT order** (A=0, C=1, G=2, T=3).

| Format | On-disk channel order | Remapped on read? |
|---|---|---|
| `.srd` (XML) | GATC | Yes — `perm = {2, 0, 3, 1}` |
| `.ab1` / `.fsa` (ABIF binary) | GATC | Yes |
| `.scf` (Staden SCF v3) | ACGT | No |
| `.dan` | ACGT (all versions) | No |

---

## File format enums

```csharp
public static class SequenceFileExtension
{
    public const string SRD = "srd", DAN = "dan", AB1 = "ab1",
                        FASTA = "fasta", FAS = "fas", TXT = "txt";
}

public enum SequenceFileFormat { Unknown, SRD, DAN, AB1, FSA, FASTA }
```
