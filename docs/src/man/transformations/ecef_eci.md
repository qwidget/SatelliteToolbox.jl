# ECEF and ECI

```@meta
CurrentModule = SatelliteToolbox
DocTestSetup = quote
    using SatelliteToolbox
end
```

This package currently provides the entire IAU-76/FK5 model to transform
reference systems. The following table lists the available coordinate frames and
how they can be referenced in the functions that will be described later on.

| Reference | Type |            Coordinate frame name            |
|-----------|------|---------------------------------------------|
| `ITRF()`  | ECEF | International Terrestrial Reference Frame   |
| `PEF()`   | ECEF | Pseudo-Earth Fixed reference frame          |
| `MOD()`   | ECI  | Mean-Of-Date reference frame                |
| `TOD()`   | ECI  | True-Of-Data reference frame                |
| `GCRF()`  | ECI  | Geocentric Celestial Reference Frame (GCRF) |
| `J2000()` | ECI  | J2000 reference frame                       |
| `TEME()`  | ECI  | True Equator, Mean Equinox reference frame  |

!!! note

    ECEF stands for Earth-Centered, Earth-Fixed whereas ECI stands for
    Earth-Centered Inertial.

## EOP Data

The conversions here sometimes requires additional data related to the Earth
orientation. This information is provided by [IERS](https://www.iers.org)
(International Earth Rotation and Reference Systems Service). The
SatelliteToolbox.jl has the capability to automatically download and parse the
IERS EOP (Earth Orientation Parameters) data.

The function that will automatically download the files, store them in the file
system, and parse the data is:

```julia
function get_iers_eop(data_type::Symbol = :IAU1980; force_download = false)
```

in which:

* `data_type` is a symbol that specify what kind of data is desired (`:IAU1980`
  for IAU1980 data and `:IAU2000A` for IAU2000A data). If omitted, then it
  defaults to `:IAU1980`.
* The files are obtained on a daily-basis by the package RemoteFiles.jl. If the
  user wants to force the download, then the keyword `force_download` should be
  set to `true`.
* This function returns an instance of the structure `EOPData_IAU1980` or
  `EOPData_IAU2000A` depending on the selection of `data_type`. The returned
  value should be passed to the reference frame conversion functions as
  described in the following.

!!! note

    Notice that, although we can fetch IAU2000A data, this IAU2000A theory is
    not implemented yet.

```julia
julia> eop = get_iers_eop();
[ Info: Downloading file 'EOP_IAU1980.TXT' from 'https://datacenter.iers.org/data/latestVersion/223_EOP_C04_14.62-NOW.IAU1980223.txt'.

```

## ECEF to ECEF

One ECEF frame can be converted to another one by the following function:

```julia
function rECEFtoECEF([T,] ECEFo, ECEFf, JD_UTC::Number, eop_data)
```

where it will be computed the rotation from the ECEF reference frame `ECEFo` to
the ECEF reference frame `ECEFf` at the Julian Day [UTC] `JD_UTC`. The rotation
description that will be used is given by `T`, which can be `DCM` or
`Quaternion`. If `T` is omitted, then it defaults to `DCM`. The EOP data
`eop_data` in this case is always necessary. Hence, the user must initialize it
as described in the section [EOP Data](@ref).

```jldoctest
julia> eop_IAU1980 = get_iers_eop(:IAU1980);

julia> rECEFtoECEF(PEF(), ITRF(), DatetoJD(1986,6,19,21,35,0), eop_IAU1980)
3×3 StaticArrays.SArray{Tuple{3,3},Float64,2,9}:
  1.0          0.0         -4.3531e-7
 -6.30011e-13  1.0         -1.44727e-6
  4.3531e-7    1.44727e-6   1.0

julia> rECEFtoECEF(Quaternion, PEF(), ITRF(), DatetoJD(1986,6,19,21,35,0), eop_IAU1980)
Quaternion{Float64}:
  + 0.9999999999997147 - 7.236343481310813e-7.i + 2.1765518308012794e-7.j + 0.0.k
```
