[<img alt="GitHub Workflow" src="https://img.shields.io/github/workflow/status/propensive/iridescence/Build/main?style=for-the-badge" height="24">](https://github.com/propensive/iridescence/actions)
[<img src="https://img.shields.io/badge/gitter-discuss-f00762?style=for-the-badge" height="24">](https://gitter.im/propensive/iridescence)
[<img src="https://img.shields.io/discord/633198088311537684?color=8899f7&label=DISCORD&style=for-the-badge" height="24">](https://discord.gg/CHCPjERybv)
[<img src="https://img.shields.io/matrix/propensive.iridescence:matrix.org?label=MATRIX&color=0dbd8b&style=for-the-badge" height="24">](https://app.element.io/#/room/#propensive.iridescence:matrix.org)
[<img src="https://img.shields.io/twitter/follow/propensive?color=%2300acee&label=TWITTER&style=for-the-badge" height="24">](https://twitter.com/propensive)
[<img src="https://img.shields.io/maven-central/v/com.propensive/iridescence-core_2.12?color=2465cd&style=for-the-badge" height="24">](https://search.maven.org/artifact/com.propensive/iridescence-core_2.12)
[<img src="https://vent.dev/badge/propensive/iridescence" height="24">](https://vent.dev/)

<img src="/doc/images/github.png" valign="middle">

# Iridescence

_Iridescence_ implements several algorithms for working with colors represented in different forms.

## Features

- represents colors using a variety of different color models
- work with colors in RGB, HSV, CMY, CMYK, HSL, L\*a\*b\* and XYZ
- convert between any colors
- utilize color profiles (where necessary)
- provides a standard palette of named colors
- print colors as CSS, Hex or ANSI
- brighten, lighten, darken and blend colors
- calculate perceptual deltas between colors


## Getting Started

_Iridescence_ provides seven different ways of representing colors:
- `Srgb`: [sRGB](https://en.wikipedia.org/wiki/SRGB), 
- `Xyz`: [CIE 1931 XYZ](https://en.wikipedia.org/wiki/CIE_1931_color_space)
- `Cielab`: [L\*a\*b\*](https://en.wikipedia.org/wiki/CIELAB_color_space)
- `Cmy`: [CMY](https://en.wikipedia.org/wiki/CMY_color_model)
- `Cmyk`: [CMKY](https://en.wikipedia.org/wiki/CMYK_color_model)
- `Hsl`: [HSL](https://en.wikipedia.org/wiki/HSL_and_HSV)
- `Hsv`: [HSV](https://en.wikipedia.org/wiki/HSL_and_HSV)

Each color model uses either three or four continuous coordinates, all represented in Iridescence as `Double`s
in the unit interval (0 ≤ *c* ≤ 1), to describe an apparently full spectrum of colors perceived by the human
eye.

Given the complex nature of sight and color, different models make different tradeoffs in their representations
of different colors. While sRGB is the most direct representation of the colored light emitted by a computer
monitor, and indeed the most common representation for computers, CMY and CMYK are more common in printing.

Meanwhile, the HSL and HSV representations representations use the natural qualititative properties of hue,
saturation, lightness and brightness, and the XYZ and L\*a\*b\* color spaces are derived empirically. L\*a\*b\*
attempts to maintain the property that the Euclidean distance between two colors is proportional to the
perceptual difference between those colors, as determined by experimentation.

The particular color model should be chosen according to the requirements of the particular task.

## A Quick Example

```scala
import iridescence.*

given profile = profiles.Ultralume50

val pink: Cielab = colors.Ivory.cielab.mix(colors.DarkMagenta.cielab)
val palePink: Srgb = pink.srgb.hsv.tint(0.5).srgb
println(s"${color.ansiFg24}Hello World!")
```

## Types

Iridescence provides case classes to immutably represent each of the seven color models, above. Colors in one
representation can be directly converted into many of the other representations, and the remaining conversions
can be performed indirectly.

In general, every color representation provides the `Color#srgb` method to convert it to an `Srgb` value.
Conversely, the `Srgb` type provides the methods `cmy`, `cmyk`, `cielab`, `xyz`, `hsv` and `hsl` to convert to
these alternative representations.

While it would be possible to provide an n×n set of methods for converting between any pair of representations,
conversions which rely on an unspecified intermediate representation (for example converting between HSL and
CMYK) are generally _not_ provided unless the intermediate representation is a necessary step in the
calculation. This is to make it clear when conversions are happening.

For example, the methods `Hsl#srgb` and `Srgb#xyz` both exist, but `Hsl#xyz` is not implemented. However,
`Srgb#cielab` _is_ provided, even though the conversion is made via an intermediate XYZ value.

Here are some examples:
```scala
import iridescence.*

val DeepPink: Srgb = Srgb(1, 0.078, 0.576)
val Gold: Hsv = Srgb(1, 0.843, 0).hsv
val Gold2: Cmyk = Gold.srgb.cmyk
```

## Palette

The `colors` object provides a standard palette of about 140 named colors defined in sRGB space.

## Color profiles

Certain color representations rely on additional information that characterizes the conditions under which the
colors are encoded, and this information is necessary for conversions between certain color spaces.

For example, to convert from `Srgb` to `Cielab` requires a profile. Profiles are provided through the `Profile`
type, and several are provided in the `profiles` object. These should be specified, implicitly or explicitly
with each conversion, like so:

```scala
val color = DeepPink.cielab(using profiles.MidMorningDaylight)
```
or,
```scala
given Profile = profiles.CoolFluorescent
val color = LawnGreen.xyz
```

For generality, conversions to `Srgb` _always_ require a profile to be given (even for conversions where it is
not used). This restriction may be lifted later. A good default profile to use is the `Daylight` profile.
```scala
given Profile = profiles.Daylight
```

## Color methods

Additional methods are provided on certain color types for producing new colors from old. In general, these
methods are particular to the color model being used.

For example, the methods `saturate`, `desaturate`, `pure` and `rotate` (for changing the hue) are provided on
`Hsl` and `Hsv` types, while `Hsv` additionally provides `shade`, `tint` and `tone` methods. These latter
methods take `black` and/or `white` parameters to specify the amount of shading, tinting or toning to be
applied.

`Cielab` provides a `delta` method for comparing two colors (returning a `Double` in the unit interval), and the
`mix` method for combining two colors. `Cielab#mix` takes another `Cielab` color as its first parameter, and
a mix ratio (again, in the unit interval) as an optional second parameter. If left unspecified, it defaults to
the midpoint between the two colors.

Use of these methods might typically involve converting a color to the model which defines them, then applying
them as necessary, before converting back. For example,
```scala
colors.IndianRed.hsv.tone(0.2, 0.4).srgb
```

## Serialization

Different formats, languages and protocols will represent colors as strings in a number of different ways.
Iridescence provides serialization methods to the following formats:
- 24-bit ANSI foreground and background escape codes,
- RGB CSS, in the form `rgb(100, 78, 12)`,
- HSL CSS, in the form `hsl(310, 12%, 84%)`,
- 12-bit and 24-bit hexadecimal, e.g. `#afc` or `#ffed00`

These are available on the `Srgb` type, with the exception of `Hsl#css`.

## Limitations

There is no support for transparency.

## Status

Iridescence is classified as __fledgling__. Propensive defines the following five stability levels for open-source projects:

- _embryonic_: for experimental or demonstrative purposes only, without guarantee of longevity
- _fledgling_: of proven utility, seeking contributions, but liable to significant redesigns
- _maturescent_: major design decisions broady settled, seeking probatory adoption and refinement of designs
- _dependable_: production-ready, subject to controlled ongoing maintenance and enhancement; tagged as version `1.0` or later
- _adamantine_: proven, reliable and production-ready, with no further breaking changes ever anticipated

## Availability

Iridescence&rsquo;s source is available on GitHub, and may be built with [Fury](https://github.com/propensive/fury) by
cloning the layer `propensive/iridescence`.
```
fury layer clone -i propensive/iridescence
```
or imported into an existing layer with,
```
fury layer import -i propensive/iridescence
```
A binary is available on Maven Central as `com.propensive:iridescence-core_<scala-version>:0.4.0`. This may be added
to an [sbt](https://www.scala-sbt.org/) build with:
```
libraryDependencies += "com.propensive" %% "iridescence-core" % "0.4.0"
```

## Contributing

Contributors to Iridescence are welcome and encouraged. New contributors may like to look for issues marked
<a href="https://github.com/propensive/iridescence/labels/good%20first%20issue"><img alt="label: good first issue"
src="https://img.shields.io/badge/-good%20first%20issue-67b6d0.svg" valign="middle"></a>.

We suggest that all contributors read the [Contributing Guide](/contributing.md) to make the process of
contributing to Iridescence easier.

Please __do not__ contact project maintainers privately with questions, as other users cannot then benefit from
the answers.

## Author

Iridescence was designed and developed by [Jon Pretty](https://twitter.com/propensive), and commercial support and
training is available from [Propensive O&Uuml;](https://propensive.com/).



## License

Iridescence is copyright &copy; 2021-21 Jon Pretty & Propensive O&Uuml;, and is made available under the
[Apache 2.0 License](/license.md).
