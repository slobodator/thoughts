# Yoda style

Well, actually is a minor thing and a matter of taste, but I’d like to put some words on it.

I seem to be a quite old dog and remember dinosaurs, black-white TVsets, `80386` CPU with `1Mb` of RAM, MS-DOS `6.22`
and other funny things.

That times the `C` code was much closer to the computer rather than programmers. So it made sense to write `if (1 == x)`
because `if (x == 1)` and `if (x = 1)` were both valid statements from the compiler perspective but probably the last
one shouldn’t be invoked always (but it was).

But there is 202x at the calendar, we use Java and IDEA with various analysers.

So, to respect colleagues not the compiler and write it straightforward without Yoda style, I’d suggest.

Don’t you mind to write `if (obj != null)` instead of `if (null != obj)`, please?

Also, there is no need to use `equals()` and reverse order for the enums. I’d prefer `if (var == ENUM_CONST)` instead
of `ENUM_CONST.equals(var)`.

SonarQube still suggests to reverse `String` comparison in order to prevent the `NPE` i.e. `SOME_STR.equals(var)`.

Nevertheless, I’d suggest to write it from left to right `var.equals(SOME_VAR)` and not to swallow `null` but manage it
beforehand **explicitly** if it is necessary. Or, at least we have `Objects.equal(var, SOME_STR)`.
