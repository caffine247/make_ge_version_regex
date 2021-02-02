# `make_ge_version_regexp.sh`

A fork of [William Smith's `Match Version Number or Higher.bash`](https://gist.github.com/talkingmoose/2cf20236e665fcd7ec41311d50c89c0e).

I'm hoping these changes get incorporated into William's script so that I don't
have to maintain this fork.

Feedback is welcome!

## Improvements

- Optimized, Jamf-unsafe pattern generator with 30-55% reduction in character length.
- Simplified, Jamf-safe pattern generator with 14-25% reduction in character length.
- Unit tests
- CLI options
- No spaces in the script name.

*Improvements compared to May 24, 2020 version of William's script.*

## Usage

```
USAGE:
        ./make_ge_version_regex.sh [OPTIONS] VERSION

OPTIONS:
        -h: show help
        -n: not using jamf
        -s: silent mode
```

Example usage:

```
$ ./make_ge_version_regex.sh -sn 1.0.0

Regex for "1.0.0" or higher (51 characters):

^(\d{2}|[2-9]|1\.(\d{2}|[1-9]|0\.(\d{2}|[1-9]|0)))

```

## Unit Tests

Requires [`bash_unit`](https://github.com/pgrange/bash_unit).

```
bash_unit tests.sh
```

## Explanation of Regular Expression Changes to Safe Generator

Take a simple pattern generated with [the original script](https://github.com/moorereason/make_ge_version_regex/blob/fc6a574e35caeb6fd3f97581a48d5832080a1f3d/make_ge_version_regex.sh):

```
Regex for "7" or higher (25 characters):

^(\d{2,}.*|[8-9].*|7.*)$
```

Compare that pattern to what's generated by the safe generator from script in
this repository:

```
Regex for "7" or higher (16 characters):

^(\d{2}|[89]|7)
```

Let's look at each change individually...

#### Reducing `\d{2,}` to `\d{2}`

At this stage of the logic, we know we have a single digit, `7`.  Since we're
looking for any version greater than or equal to 7, we know that any number with
two or more digits will be greater than the string `7`; however, we don't care
that there are three or more digits.  We know that just two digits meets the
requirement, so we remove the `,` from the quantifier to save one character in
the pattern length for each numbered segment in the version string.

#### Removing `.*` and `$`

The `.*` is superfluous for our needs.  By the time we get to the `.*` token
sequence in the pattern, we already know that we've matched the important bits
or not.  For each member of the pattern grouping, we can save two characters in
the pattern length by removing this token sequence.

Additionally, removing the `.*` token sequence necessitates removing the
trailing `$` token which saves one character per pattern.

#### Reducing `[8-9]` to `[89]`

Since `[8-9]` and `[89]` are equivalent, we can save a single character for each
`7` in the version string.

#### Comparing a Complex Version String

Given `7.4.59931.0110`, compare the before and after to see these changes play
out in the generated patterns:

```
^(\d{2,}.*|[8-9].*|7\.\d{2,}.*|7\.[5-9].*|7\.4\.\d{6,}.*|7\.4\.[6-9]\d{4,}.*|7\.4\.599[4-9]\d{1,}.*|7\.4\.5993[2-9].*|7\.4\.59931\.\d{5,}.*|7\.4\.59931\.[1-9]\d{3,}.*|7\.4\.59931\.0[2-9]\d{2,}.*|7\.4\.59931\.01[2-9]\d{1,}.*|7\.4\.59931\.011[1-9].*|7\.4\.59931\.0110.*)$
^(\d{2}|[89]|7\.\d{2}|7\.[5-9]|7\.4\.\d{6}|7\.4\.[6-9]\d{4}|7\.4\.599[4-9]\d{1}|7\.4\.5993[2-9]|7\.4\.59931\.\d{5}|7\.4\.59931\.[1-9]\d{3}|7\.4\.59931\.0[2-9]\d{2}|7\.4\.59931\.01[2-9]\d{1}|7\.4\.59931\.011[1-9]|7\.4\.59931\.0110)
```

## Regular Expression Changes to the Optimized Generator

The optimized generator has all of the features of the safe generator plus
nested groupings at each version string segments to avoid repetition of previous
segments.  The downside of this approach is that it can't be split up for jamf
if the pattern exceeds 255 characters.

Jamf-Safe Pattern:
```
Regex for "1.2.3" or higher (65 characters):

^(\d{2}|[2-9]|1\.\d{2}|1\.[3-9]|1\.2\.\d{2}|1\.2\.[4-9]|1\.2\.3)
```

Optimized Pattern:
```
Regex for "1.2.3" or higher (51 characters):

^(\d{2}|[2-9]|1\.(\d{2}|[3-9]|2\.(\d{2}|[4-9]|3)))
                 ^               ^
```

The key is the nested groups (marked by the carets).  As you can see, each
version string segment is turned into a nested group to keep from having to
repeat the previous segments.

Special thanks to **Mike Dowler** for
[a comment in a Jamf Nation thread](https://www.jamf.com/jamf-nation/feature-requests/10085/allow-regex-longer-than-255-characters-in-smart-group-criteria#responseChild30028)
for this idea.  I figured there was a way to do this but hadn't taken the time
to think through it.

#### Comparing a Complex Version String (Again)

Given `7.4.59931.0110`, compare the safe and optimized patterns:

```
^(\d{2}|[89]|7\.\d{2}|7\.[5-9]|7\.4\.\d{6}|7\.4\.[6-9]\d{4}|7\.4\.599[4-9]\d{1}|7\.4\.5993[2-9]|7\.4\.59931\.\d{5}|7\.4\.59931\.[1-9]\d{3}|7\.4\.59931\.0[2-9]\d{2}|7\.4\.59931\.01[2-9]\d{1}|7\.4\.59931\.011[1-9]|7\.4\.59931\.0110)
^(\d{2}|[89]|7\.(\d{2}|[5-9]|4\.(\d{6}|[6-9]\d{4}|599[4-9]\d{1}|5993[2-9]|59931\.(\d{5}|[1-9]\d{3}|0[2-9]\d{2}|01[2-9]\d{1}|011[1-9]|0110))))
```

## License

[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/)
