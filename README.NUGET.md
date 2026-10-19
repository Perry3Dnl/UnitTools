# UnitTools

**High-performance .NET unit conversions — straightforward numeric APIs, strongly typed quantities, and precise calculations.**

[GitHub repository](https://github.com/Perry3Dnl/UnitTools) · [Documentation](https://github.com/Perry3Dnl/UnitTools/tree/main/docs) · [Release notes](https://github.com/Perry3Dnl/UnitTools/blob/main/docs/Changelog.md)

UnitTools includes **150 conversion families**, **1,858 registered units**, and **39,458 directed non-identity conversions** within compatible families. Choose direct numeric conversion, fluent extensions, typed quantities, exact decimal/rational conversion, dynamic parsing, or custom unit definitions. The core library has no third-party runtime package dependencies.

## Packages

| NuGet package | Purpose |
| --- | --- |
| `UnitTools` | Full conversion engine and quantity APIs |
| `UnitTools.SystemTextJson` | Optional `System.Text.Json` support for typed unit values |
| `UnitTools.Analyzers` | Optional Roslyn compile-time guidance |

All three packages ship with the **same version**, currently **1.0.2**. Install only the parts you need.

## Installation

```bash
dotnet add package UnitTools
```

For JSON integration or analyzer diagnostics, install the matching companion package in addition to the core package.

## Quick start

```csharp
using UnitTools;
using UnitTools.Length;
using UnitTools.Quantities;

// Numeric conversion: 4 km -> 4000 m.
double meters = UnitToolsLength.Convert(
    4d,
    UnitToolsLengthUnit.Kilometer,
    UnitToolsLengthUnit.Meter);

// Fluent numeric conversion.
double fluentMeters = 4d.Convert(
    UnitToolsLengthUnit.Kilometer,
    UnitToolsLengthUnit.Meter);

// Keep the unit attached to a value.
double wrappedMeters = new UnitValue<UnitToolsLengthUnit>(
    4d, UnitToolsLengthUnit.Kilometer)
    .To(UnitToolsLengthUnit.Meter).Value;

// Compile-time dimensional safety.
Length<decimal> distance = Length<decimal>.FromKilometres(4m);
decimal typedMeters = distance.Metres;
```

### Parsing, exact arithmetic, and composite units

```csharp
double speed = UnitToolsDynamic.Convert("36 km/h", "m/s");
decimal exact = UnitToolsExact.Convert(
    1m, UnitToolsLengthUnit.Inch, UnitToolsLengthUnit.Foot);
double energy = UnitToolsComposite.Convert(1d, "N*m", "J");
```

## Compatibility and design

- Core: .NET Standard 2.0 and .NET 8.
- JSON and analyzer companions: .NET Standard 2.0.
- Standalone core without third-party runtime dependencies.
- Exact decimal/rational paths, span-based parsing/formatting, localization, composite units, and arbitrary-scale number naming.
- Release validation includes tests, API compatibility checks, NativeAOT smoke tests, and clean NuGet consumers.

## Documentation and license

For complete APIs, sample programs, benchmarking, and supported families, visit the [GitHub README](https://github.com/Perry3Dnl/UnitTools#readme).

**License:** Apache License 2.0 with Commons Clause v1.0. Redistribution is subject to the Commons Clause restriction on selling the software itself. Review the [full license](https://github.com/Perry3Dnl/UnitTools/blob/main/LICENSE.md) before redistribution.

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
