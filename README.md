# UnitTools

[![build](https://github.com/Perry3Dnl/UnitTools/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Perry3Dnl/UnitTools/actions/workflows/ci.yml)
[![benchmarks](https://github.com/Perry3Dnl/UnitTools/actions/workflows/benchmarks.yml/badge.svg?branch=main)](https://github.com/Perry3Dnl/UnitTools/actions/workflows/benchmarks.yml)
[![routed double](docs/benchmarks/badges/routed-double.svg)](docs/benchmarks/BASELINE.md)
[![named decimal](docs/benchmarks/badges/named-decimal.svg)](docs/benchmarks/BASELINE.md)
[![span parse](docs/benchmarks/badges/span-parse.svg)](docs/benchmarks/BASELINE.md)
[![span format](docs/benchmarks/badges/span-format.svg)](docs/benchmarks/BASELINE.md)
[![exact decimal](docs/benchmarks/badges/exact-decimal.svg)](docs/benchmarks/BASELINE.md)

UnitTools is a high-performance, dependency-free .NET unit conversion engine built for both simple application code and demanding engineering/scientific workloads. The current core catalog contains **150 conversion families**, **1,858 registered units**, and **9,395 accepted unit names, symbols, aliases, and notation variants**. Those units provide **39,458 directed non-identity unit-to-unit conversion pairs** inside their compatible families, while the core assembly exposes **1,102 public types** and **14,494 public methods/overloads** across direct converters, numeric extensions, typed quantities, parsing/formatting, exact conversion, dynamic detection, composite expressions, custom units, metadata, and arbitrary-scale numbers.

UnitTools does not force one programming model. Use a direct `double`/`float`/`decimal` conversion when that is all you need, fluent numeric extensions when you want compact code, `UnitValue<TUnit>` when the unit should travel with the value, or strongly typed quantities when compile-time dimensional safety matters. Exact decimal/rational routes, allocation-aware span APIs, runtime unit detection, localization, NativeAOT validation, and BenchmarkDotNet regression gates are all part of the same standalone core design.

The current synchronized package version is **`1.0.1`** across `UnitTools`, `UnitTools.SystemTextJson`, and `UnitTools.Analyzers`. The core package targets **.NET Standard 2.0 and .NET 8**. Optional `UnitTools.SystemTextJson` and `UnitTools.Analyzers` companion packages target **.NET Standard 2.0**. The release candidate is validated by **5,218 tests**, public-API compatibility fingerprints, clean NuGet consumers on **netstandard2.0 / net8.0 / net10.0**, NativeAOT smoke applications, and BenchmarkDotNet regression gates. Version `1.0.1` is prepared for release; publication is verified separately on NuGet.org.

> **Scale metrics are generated from the compiled catalog/API, not estimated.** A conversion pair is one directed `from → to` route within the same family and excludes identity conversions such as `Meter → Meter`. Reproduce the current numbers with `dotnet run --project tools/UnitTools.ApiCompat/UnitTools.ApiCompat.csproj -c Release -- --stats`.

## Why UnitTools

UnitTools is designed for developers who want to choose the right safety level for each part of an application instead of being forced into one quantity model. The dependency-free core supports ordinary numeric conversion, fluent numeric extensions, value-plus-unit wrappers, strongly typed quantities, runtime/dynamic conversion, exact decimal/rational paths, composite expressions, custom units, metadata, and arbitrary-scale numbers.

### Choose the API style that fits

```csharp
using UnitTools;
using UnitTools.Length;
using UnitTools.Quantities;

// 1. Straightforward numeric conversion.
double metres = UnitToolsLength.Convert(
    4d,
    UnitToolsLengthUnit.Kilometer,
    UnitToolsLengthUnit.Meter);

// 2. Fluent numeric conversion. The developer controls the current unit.
double metresAgain = 4d.Convert(
    UnitToolsLengthUnit.Kilometer,
    UnitToolsLengthUnit.Meter);

// 3. Value + unit when the unit should travel with the value.
double metresFromValue = new UnitValue<UnitToolsLengthUnit>(
    4d,
    UnitToolsLengthUnit.Kilometer)
    .To(UnitToolsLengthUnit.Meter)
    .Value;

// 4. Strongly typed quantity when compile-time dimensional safety matters.
Length<decimal> distance = Length<decimal>.FromKilometres(4m);
decimal typedMetres = distance.Metres;
```

For runtime text, exact decimal workflows, or composite units:

```csharp
double parsed = UnitToolsDynamic.Convert("36 km/h", "m/s");

decimal exact = UnitToolsExact.Convert(
    1m,
    UnitToolsLengthUnit.Inch,
    UnitToolsLengthUnit.Foot);

double composite = UnitToolsComposite.Convert(
    1d,
    "N*m",
    "J");
```

The core package has **no third-party runtime package dependencies**. Install only the optional JSON or analyzer companion when that integration is useful.


## Performance baseline

UnitTools has a dedicated BenchmarkDotNet suite covering direct/routed conversion, exact decimal arithmetic, named and universal quantities, span parsing/formatting, catalog lookup, dynamic conversion, and composite expressions.

Recent GitHub-hosted runs keep routed `double` conversion in the low-single-digit nanosecond range with **0 B/op**, named `decimal` conversion in the tens-of-nanoseconds range with **0 B/op**, and span parsing/formatting around the ~100 ns range with **0 B/op**. Exact decimal conversion is also allocation-free on the benchmarked hot path. Cached composite parsing/conversion run in the tens-of-nanoseconds range with only **24 B/op** and **48 B/op** respectively. Hosted-runner timings vary between runs, so `docs/benchmarks/latest.json`, the generated badges, and the full baseline are the source of truth rather than hard-coded latency claims.

See [the full benchmark baseline](docs/benchmarks/BASELINE.md) for all measurements and runner details. The badge SVGs are generated from the latest successful `main` benchmark run and committed by the benchmark workflow; hosted-runner timings vary, so they are trend indicators rather than absolute latency guarantees.

Locally, BenchmarkDotNet filters can be used to focus a run, for example:

```sh
dotnet run --project benchmarks/UnitTools.Benchmarks/UnitTools.Benchmarks.csproj -c Release -- --filter "*ConversionBenchmarks*"
dotnet run --project benchmarks/UnitTools.Benchmarks/UnitTools.Benchmarks.csproj -c Release -- --anyCategories Span
```

## Build, test and create a package

Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0), then run these commands from the repository root:

```sh
dotnet restore UnitTools.sln
dotnet build UnitTools.sln --configuration Release --no-restore
dotnet test tests/UnitTools.Tests/UnitTools.Tests.csproj --configuration Release --no-build
dotnet pack src/UnitTools/UnitTools.csproj --configuration Release --no-build --output artifacts/packages
dotnet pack src/UnitTools.SystemTextJson/UnitTools.SystemTextJson.csproj --configuration Release --no-build --output artifacts/packages
dotnet pack src/UnitTools.Analyzers/UnitTools.Analyzers.csproj --configuration Release --no-build --output artifacts/packages
python scripts/verify_package.py artifacts/packages
```

The `.nupkg` and `.snupkg` are written to `artifacts/packages`. The SDK is a build requirement; consumers need a runtime compatible with .NET Standard 2.0.

Install the candidate into an existing application using the absolute path to that directory:

```sh
dotnet add package UnitTools --version 1.0.0 --source /absolute/path/to/UnitTools/artifacts/packages
```

GitHub Actions also builds the solution, runs the NUnit suite, verifies the checked-in public API fingerprint baseline, creates all three packages, and installs the produced packages into clean **netstandard2.0**, **net8.0**, and **net10.0** consumers. Successful runs expose a `nuget-packages` artifact. CI also publishes and executes NativeAOT smoke consumers for the core APIs and the System.Text.Json companion; AOT builds treat trimming/dynamic-code warnings as errors. Parser abuse/fuzz tests and explicit expression-complexity limits guard malformed and adversarial input.

## Use the existing API

```csharp
using UnitTools.Length;
using UnitTools.Length.Metric;

// Static router: source unit -> family base unit -> target unit.
double millimeters = UnitToolsLength.Convert(
    12.5,
    UnitToolsLengthUnit.Centimeter,
    UnitToolsLengthUnit.Millimeter);

// Numeric extension: the same converter and arithmetic.
double centimeters = 12.5;
double alsoMillimeters = centimeters.Convert(
    UnitToolsLengthUnit.Centimeter,
    UnitToolsLengthUnit.Millimeter);

// Direct method for a known conversion.
double meters = UnitToolsMetricLength.CentimeterToMeter(12.5);
```

The root namespace is now `UnitTools`. Converter class names, existing enum values, conversion factors, overloads, and the base-unit conversion flow are preserved from version 1.1.3. Higher-level APIs are additive: strings can be parsed and formatted through the shared catalog, and `UnitValue<TUnit>` can retain a unit when that is useful without replacing the numeric API.


UnitTools deliberately supports multiple safety levels. Strongly typed quantities are available when compile-time dimensional safety matters, while direct methods, numeric/family converters, dynamic conversion, and low-level APIs remain supported for developers who intentionally need them. The optional `UnitTools.Analyzers` package adds IntelliSense guidance for selected escape-hatch patterns without making those APIs unavailable.

Culture-aware numeric parsing/formatting is supported through `IFormatProvider`. Culture-specific unit display names and parsing aliases keep symbols and enum names invariant. Dutch localization is currently **complete** for Angle, Area, Energy, Force, Frequency, Power, Pressure, Speed, Temperature, and Time; Length, Mass, and Volume remain explicitly **partial** until every catalog entry has been reviewed. CI verifies generated localization data and completeness metadata. See [Culture and localization](docs/Localization.md), [Localization coverage](docs/LocalizationCoverage.md), and [Analyzers](docs/Analyzers.md).

```csharp
double miles = UnitToolsParser.Convert("1.8 km", UnitToolsLengthUnit.Mile);

var distance = new UnitValue<UnitToolsLengthUnit>(1.8, UnitToolsLengthUnit.Kilometer);
string text = distance.ToString(); // "1.8 km"
```

`TemperatureDelta` is a separate family from absolute `Temperature`, so temperature differences use scale factors without absolute-temperature offsets.

Runtime family detection is also available when the caller does not know the family in advance:

```csharp
UnitToolsDetectionResult detected = UnitToolsDetector.Detect("2 light-year");

// Or convert directly when the detected family is unambiguous.
double meters = UnitToolsDynamic.Convert("2 light-year", "m");
```

Global detection returns explicit candidates for ambiguous symbols instead of guessing. Family and unit descriptors also expose physical dimensions, including a separate information dimension for digital storage/data rate. Dimensions can be multiplied, divided, and raised to integer powers for future dimensional-analysis APIs. Fuel economy is modeled with unit-specific reciprocal dimensions because `km/L` and `L/100 km` are physically reciprocal representations.

Aliases are definition-driven. Curated vocabulary lives in `definitions/unit-aliases.json`; `scripts/generate_aliases.py` combines those overrides with safe generated human forms for all 150 families and 1,858 unit entries, then emits checked-in static C#. CI verifies the generated files are current, so consumers do not need Python and the runtime remains dependency-free.

Composite unit expressions are supported through the dimensional metadata:

```csharp
double metersPerSecond = UnitToolsComposite.Convert(
    36,
    "km/h",
    "m/s"); // 10

double joules = UnitToolsComposite.Convert(
    1,
    "N*m",
    "J"); // 1

UnitToolsUnitExpression expression =
    UnitToolsExpressionParser.Parse("kg*m/s^2");
```

The expression parser supports multiplication, division, parentheses, integer powers, superscript `²`/`³`, and middle-dot notation. Ambiguous symbols remain explicit when parsed alone. When a complete conversion provides enough dimensional context, ambiguity can be resolved safely; for example `h` can mean hour or Planck constant, but `km/h -> m/s` has only one dimensionally compatible interpretation.

### Custom units

Runtime custom units can be registered without modifying the built-in enums or generated catalogue:

```csharp
UnitToolsCustomUnit perryLength =
    UnitToolsCustomUnits.RegisterLinear(
        UnitToolsFamily.Length,
        "PerryLength",
        "perryLength",
        "Meter",
        1.7018);

double meters = UnitToolsDynamic.Convert(
    "2 perryLength",
    "Meter");
```

Affine units are also supported when a scale and offset are required:

```csharp
UnitToolsCustomUnit customTemperature =
    UnitToolsCustomUnits.RegisterAffine(
        UnitToolsFamily.Temperature,
        "CustomTemperature",
        "°X",
        "Custom temperature",
        "Celsius",
        2.0,
        10.0);
```

Custom aliases cannot shadow built-in aliases in the same family. Registered units participate in family-specific dynamic conversion and global detection. Linear custom units can also participate in composite expressions; affine custom units are rejected as multiplicative expression factors because offsets are not representable by a pure scale expression.

### Arbitrary-scale number conversion

UnitTools now also includes the former Ginor number-conversion engine under `UnitTools.Numbers`. It handles exact sparse base-10 values whose exponents can be far beyond conventional CLR numeric ranges:

```csharp
using UnitTools.Numbers;

ArbitraryScaleNumber huge =
    ArbitraryScaleNumber.Parse("1e100000000000000000000");

Console.WriteLine(huge.ToScientific());
// 1e100000000000000000000
```

Named-scale conversion uses the same straightforward conversion style:

```csharp
ArbitraryScaleNumber billions =
    NumberScaleConverter.Convert(
        "500 million",
        "billion");

Console.WriteLine(billions.ToScientific());
// 5e-1

ArbitraryScaleNumber micros =
    NumberScaleConverter.Convert(
        "1 milli",
        "micro");

Console.WriteLine(micros.ToScientific());
// 1e3
```

The scale catalogue is symmetric around `10^0`. In addition to conventional large names, it includes SI prefixes from `deci` through `quecto` for small values and `deca` through `quetta` for large values.

The integrated generated-name grammar retains Ginor's deterministic naming model and now works in both exponent directions. `ginozillion` is the large generated scale at layer 2, magnitude 4 and coefficient 1, corresponding to `10^10000`. Its tiny counterpart uses the same reversible address with the tiny direction and deterministically becomes `ginuzillion`, corresponding to `10^-10000`:

```csharp
ArbitraryScaleNumber ginozillion =
    ArbitraryScaleNumber.Parse("1 ginozillion");

ArbitraryScaleNumber ginuzillion =
    ArbitraryScaleNumber.Parse("1 ginuzillion");

GeneratedNumberScalePosition large =
    GeneratedNumberNames.Parse("ginozillion");

GeneratedNumberScalePosition tiny =
    GeneratedNumberNames.Parse("ginuzillion");

// large.Direction == GeneratedNumberScaleDirection.Large
// tiny.Direction  == GeneratedNumberScaleDirection.Tiny
```

Generated names are reversible addresses:

```csharp
var position = new GeneratedNumberScalePosition(
    layer: 3,
    magnitude: 27,
    coefficient: 7);

string name = GeneratedNumberNames.GetName(position);
GeneratedNumberScalePosition restored =
    GeneratedNumberNames.Parse(name);
```

The folded Ginor naming model is preserved as well:

```csharp
var address = new GeneratedNumberNameAddress(
    new GeneratedNumberScalePosition(2, 4, 1),
    foldDepth: 1);

string foldedName = GeneratedNumberNames.GetName(address);
GeneratedNumberNameAddress restoredAddress =
    GeneratedNumberNames.ParseAddress(foldedName);

string symbolic = restoredAddress.ToSymbolicExpression();
// 10^(10^(10^4))
```

The number subsystem also exposes `ArbitraryScaleMath`, exact sparse addition/subtraction/multiplication, non-negative integer powers, checked conversion to `double`, `decimal`, and `BigInteger`, and the registered conventional scale catalogue from one through googol plus generated scales.

This functionality is part of the dependency-free core package; existing unit-conversion APIs remain unchanged.

### System.Text.Json serialization

Serialization support is kept out of the dependency-free core package. Install the companion package `UnitTools.SystemTextJson` and register its converter factory:

```csharp
using System.Text.Json;
using UnitTools.SystemTextJson;

var options = new JsonSerializerOptions()
    .AddUnitTools();

var distance = new UnitValue<UnitToolsLengthUnit>(
    1.8,
    UnitToolsLengthUnit.Kilometer);

string json = JsonSerializer.Serialize(distance, options);
// {"value":1.8,"unit":"Kilometer"}

UnitValue<UnitToolsLengthUnit> restored =
    JsonSerializer.Deserialize<UnitValue<UnitToolsLengthUnit>>(
        json,
        options);
```

Deserialization accepts the catalogue's aliases as unit values, while serialization writes the stable enum name. The companion package depends on `System.Text.Json`; the `UnitTools` core package remains free of third-party runtime dependencies.


### Exact decimal and strongly typed quantities

Decimal extension conversions now stay on a decimal-native path. Linear conversion factors are composed as reduced rational numbers before they are applied, so conversions such as inch-to-foot and extreme SI-prefix ratios do not need a `decimal -> double -> decimal` round trip or an underflow-prone intermediate base value.

```csharp
using UnitTools;
using UnitTools.Length;

decimal feet = 1m.Convert(
    UnitToolsLengthUnit.Inch,
    UnitToolsLengthUnit.Foot); // rational factor is exactly 1/12 before decimal rounding

decimal tinyRatio = UnitToolsExact.Convert(
    1m,
    UnitToolsLengthUnit.Quectometer,
    UnitToolsLengthUnit.Rontometer); // 0.001m
```

On .NET 8 or later, `UnitQuantity<T, TUnit>` provides a strongly typed wrapper for every catalog family with uniform physical dimensions. The unit enum is part of the CLR type, so unrelated families cannot be mixed by addition, subtraction, comparison, or assignment.

```csharp
using UnitTools.Quantities;
using UnitTools.Torque;

var torque = UnitQuantity.From(
    12.5m,
    UnitToolsTorqueUnit.NewtonMeter);

var doubled = torque + torque;
decimal value = doubled.As(UnitToolsTorqueUnit.NewtonMeter);
```

The named quantity layer now covers the complete 150-family catalog on .NET 8+. Every uniform family has a first-class named generic type with a property and `From<Unit>` factory for every registered unit, enum-based `From/In/Convert`, generic arithmetic, comparisons, span parsing, and span formatting. The original named types such as `Length<T>`, `Time<T>`, `Pressure<T>`, and `Voltage<T>` retain their existing friendly names while also exposing the rest of their catalog units. Fuel economy remains intentionally represented by the two reciprocal-safe types `FuelEfficiency<T>` and `FuelConsumption<T>`.

```csharp
using UnitTools.Quantities;

Stress<decimal> stress =
    YoungModulus<decimal>.FromGigapascal(200m) *
    Strain<decimal>.FromMicrostrain(500m);

decimal megapascals = stress.Megapascal; // 100m

Impedance<decimal> impedance = Impedance<decimal>.Parse(
    "50 Ω",
    CultureInfo.InvariantCulture);

Voltage<decimal> voltage =
    impedance * ElectricCurrent<decimal>.FromAmperes(2m);
```

The established named quantities such as `Length<T>`, `Time<T>`, `Speed<T>`, `Force<T>`, and `Energy<T>` continue to provide derived arithmetic such as length/time -> speed and force*length -> energy. The additional engineering types retain the reviewed relations including stress*area -> force, modulus*strain -> stress, spring-constant*length -> force, heat-flux-density*area -> power, and impedance*current -> voltage. Catalog-wide generation is guarded in CI so adding or changing a legacy unit cannot silently leave the named typed surface behind.

Integral extensions continue to return `double` to preserve fractional results. Decimal extensions use the shared exact/rational conversion engine. The legacy `PixelPerUnityUnit` entry remains a numeric compatibility alias with its original one-unit-equals-one-metre convention.


## Short numbers: conventional, gaming, and Ginor

UnitTools offers independent numeric notation APIs in `UnitTools.Numbers`.
All parsed values use the same exact `ArbitraryScaleNumber` representation.

```csharp
using UnitTools.Numbers;

var conventional = CompactNumberNotation.Parse("2.5B");  // 2.5e9
var gaming = GamingNumberNotation.Parse("1.25aa");         // 1.25e114
var ginor = ArbitraryScaleNumber.Parse("1 ginozillion"); // 1e10000

string display = CompactNumberNotation.Format(conventional); // 2.5B
string gameText = GamingNumberNotation.Format(gaming);       // 1.25aa
string ginorText = ginor.ToScientific();                     // 1e10000
```

Conventional suffixes are case-sensitive: `K` (10^3), `M` (10^6), `B` (10^9),
`T` (10^12), `Qa`, `Qi`, `Sx`, `Sp`, `Oc`, `No`, `Dc` (10^33). In particular,
lowercase `m` is **not** treated as million; SI uses it for milli.
Above `Dc`, conventional formatting falls back to scientific notation.

Gaming notation is an **explicit optional convention**: `a` = 10^36,
`b` = 10^39, ... `z` = 10^111, `aa` = 10^114,
`ab` = 10^117, and so on with bijective base-26 lettering.
Gaming suffixes must be lowercase. Some games start their letter sequence
at another magnitude; this API deliberately fixes the mapping to make
saved values and round-trips unambiguous. Neither short notation alters
the independent, reversible Ginor-generated naming grammar.

Ginor also provides its own **reversible short addresses**:

```csharp
var position = GeneratedNumberNames.Parse("ginozillion");
string readable = GinorNotation.ToShortName(position); // "G2:4:1"
string compact = GinorNotation.ToCompactName(position); // canonical base-62

var restored = GinorNotation.ParseShortName(readable);
var address = GinorNotation.ParseCompactName(compact);
string original = GeneratedNumberNames.GetName(restored); // "ginozillion"
```

Readable Ginor coordinates use `G{layer}:{magnitude}:{coefficient}`
or `G-{layer}:{magnitude}:{coefficient}` for tiny scales. Folded
addresses append `:F{foldDepth}`. The smaller `G~`-prefixed
representation encodes the same coordinates using canonical base-62.
Use `ParseShortAddress` or `ParseCompactName` to preserve folding.
These are **name addresses**, not guarantees that all higher-layer or
folded values can be materialized as numbers.

## Release readiness

The preview release pipeline is prepared but intentionally does **not** publish automatically. `.github/workflows/publish-nuget.yml` is manual-only, requires execution from `main`, reruns generated-source checks, public API compatibility, tests, analyzer validation, package/consumer verification, and both NativeAOT smoke applications, then uses **NuGet Trusted Publishing through GitHub OIDC** for the publish step.

Before the first nuget.org publication, the remaining account/repository setup is external to the codebase: confirm ownership/availability of the three package IDs, configure the NuGet Trusted Publishing policy for this repository/workflow, and create the protected GitHub `nuget-release` environment. See [Packaging and release](docs/Developer/Packaging.md).

## License

UnitTools and `UnitTools.SystemTextJson` are **source-available** under the **Apache License 2.0 subject to the Commons Clause License Condition v1.0**.

You may use UnitTools free of charge in personal, open, internal, and commercial software. You may distribute and sell products that include or depend on UnitTools. The Commons Clause restriction means you may not sell UnitTools itself, or a product or service whose value derives entirely or substantially from UnitTools, without a separate license from the licensor.

This licensing model is intentionally **not OSI Open Source** because the Commons Clause adds a commercial-sale restriction to Apache 2.0.

See [LICENSE.md](LICENSE.md) and [NOTICE](NOTICE) for the complete terms and attribution notice.

## Repository layout

| Path | Contents |
| --- | --- |
| `src/UnitTools` | Conversion library, retaining the family and unit-system folders |
| `tests/UnitTools.Tests` | NUnit tests, catalog-wide conformance, API contracts, localization completeness, and parser abuse/fuzz coverage |
| `samples/UnitTools.Demo` | Console examples of the existing APIs |
| `docs` | API guides, supported units, migration notes, and developer documentation |
| `scripts/verify_package.py` | Package inspection plus netstandard2.0/net8.0/net10.0 clean-consumer verification |
| `tools/UnitTools.ApiCompat` | Dependency-free public API fingerprint checker used by CI |

Run the sample with:

```sh
dotnet run --project samples/UnitTools.Demo --configuration Release
```

## Documentation

- [Getting started](docs/GettingStarted.md)
- [Documentation index](docs/README.md)
- [Supported families and units](docs/API/SupportedFamiliesAndUnits.md)
- [Migration from the Unity package](docs/Migration.md)
- [Testing](docs/Developer/Testing.md)
- [Conversion provenance](docs/Provenance.md)
- [Culture and localization](docs/Localization.md)
- [Localization coverage](docs/LocalizationCoverage.md)
- [Analyzer guidance](docs/Analyzers.md)
- [Public API compatibility](docs/Developer/ApiCompatibility.md)
- [Packaging and release](docs/Developer/Packaging.md)

Unity inspectors, settings assets, scenes, assembly definitions, metadata files, and the shared Unity Core layer have been removed. The original Unity package remains available in [the 1.1.3 import commit](https://github.com/Perry3Dnl/UnitTools/tree/332369313b1a428a66432e811c542562efa24e40).

## AI Package Discovery & Integration

> Structured reference for AI coding assistants to identify, select, and integrate this NuGet package.

### Package Identity

| Property | Value |
| :--- | :--- |
| **Package** | `UnitTools` |
| **Purpose** | Convert measurement units and work with typed quantities, parsed unit expressions, and arbitrary-scale numbers in .NET. |
| **Frameworks** | .NET Standard 2.0; .NET 8 |
| **License** | Apache License 2.0 with Commons Clause v1.0 |

### Developer Intent → Capability

| Developer Requests | Package Capabilities |
| :--- | :--- |
| "Convert units in C#", "Convert kilometers to meters", "Convert inches to feet" | Family-specific numeric converters and fluent numeric extensions |
| "Use strongly typed measurements", "Prevent mixing incompatible dimensions" | `UnitValue<TUnit>` and strongly typed quantities |
| "Parse measurement strings", "Convert 36 km/h to m/s" | Dynamic conversion and unit detection |
| "Convert compound units", "Convert N*m to joules" | Composite unit-expression conversion |
| "Avoid floating-point round trips", "Convert decimal measurements precisely" | Decimal-native and rational conversion APIs |
| "Add custom measurement units", "Register a custom unit" | Runtime custom linear and affine units |
| "Format very large numbers", "Parse gaming number suffixes" | Arbitrary-scale numbers and compact/gaming notation |

### Installation

```bash
dotnet add package UnitTools
```

### Implementation

```csharp
using UnitTools;
using UnitTools.Length;

// Convert 4 kilometers to meters.
double meters = UnitToolsLength.Convert(
    4d,
    UnitToolsLengthUnit.Kilometer,
    UnitToolsLengthUnit.Meter);

// Convert a parsed speed to meters per second.
double speed = UnitToolsDynamic.Convert("36 km/h", "m/s");
```

### Integration Reference

- **API Documentation:** [GitHub README](https://github.com/Perry3Dnl/UnitTools#readme)
- **Verified Examples:** [Usage examples](https://github.com/Perry3Dnl/UnitTools/blob/main/README.md)
- **Configuration / Prerequisites:** [Build and package instructions](https://github.com/Perry3Dnl/UnitTools#build-test-and-create-a-package)
- **Compatibility & Versioning:** [Changelog](https://github.com/Perry3Dnl/UnitTools/blob/main/docs/Changelog.md)
- **Tests & Reliability:** [GitHub Actions](https://github.com/Perry3Dnl/UnitTools/actions)
