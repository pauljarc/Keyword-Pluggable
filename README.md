# NAME

Keyword::Pluggable - define new keywords in pure Perl

# SYNOPSIS

    package Some::Module;

    use Keyword::Pluggable;

    sub import {
        # create keyword 'provided', expand it to 'if' at parse time
        Keyword::Pluggable::define
            keyword => 'provided',
            package => scalar(caller),
            code    => 'if',
        ;
    }

    sub unimport {
       # disable keyword again
       Keyword::Pluggable::undefine keyword => 'provided', package => scalar(caller);
    }

    'ok'

# DESCRIPTION

Warning: This module is still new and experimental. The API may change in
future versions. The code may be buggy. Also, this module is a fork from
`Keyword::Simple`, that somehow got stalled. If its author accepts pull
requests, then it will probably be best to use it instead.

This module lets you implement new keywords in pure Perl. To do this, you need
to write a module and call
[`Keyword::Pluggable::define`](#keyword-pluggable-define) in your `import`
method. Any keywords defined this way will be available in the scope
that's currently being compiled. The scope can be lexical, packaged, and global.

## Functions

- `Keyword::Pluggable::define %options`
    - keyword

        The keyword is injected in the scope currently being compiled

    - code (string or coderef)

        For every occurrence of the keyword, your coderef will be called and its result
        will be injected into perl's parse buffer, so perl will continue parsing as if
        its contents had been the real source code in the first place. First paramater
        to the eventual coderef will be all code textref following the keyword to be replaced,
        if examination and change is needed.

    - expression

        String value; if `"statement"`, then the injected code will be parsed as a
        statement.  If `"expression"`, if will be parsed as an expression.  If
        `"dynamic"`, then `code` must be a coderef rather than a string, returning
        a true value to indicate an expression or a false value to indicate a
        statement.  (For backward compatibility, a false value for `expression` is
        treated as `"statement"`, and any unrecognized value is treated as
        `"expression"`.)

    - global

        Boolean flag; if set, then the scope is global, otherwise it is lexical or packaged

    - package

        If set, the scope will be limited to that package, otherwise it will be lexical
- `Keyword::Pluggable::undefine %options`

    Allows options: `keyword`, `global`, `package` (see above).

    Disables the keyword in the given scope. You can call this from your
    `unimport` method to make the `no Foo;` syntax work.

# BUGS AND LIMITATIONS

This module depends on the [pluggable keyword](https://metacpan.org/pod/perlapi.html#PL_keyword_plugin)
API introduced in perl 5.12. `parse_` functions were introduced in 5.14.
Older versions of perl are not supported.

Every new keyword is actually a complete statement or an expression by itself. The parsing magic
only happens afterwards. This means that e.g. the code in the ["SYNOPSIS"](#synopsis)
actually does this:

    provided ($foo > 2) {
          ...
    }

    # expands to

    ; if
    ($foo > 2) {
          ...
    }

The `;` represents a no-op statement, the `if` was injected by the Perl code,
and the rest of the file is unchanged. This also means your it can
only occur at the beginning of a statement, not embedded in an expression.
To be able to do that, use `expression => 1` flag.

Keywords in the replacement part of a `s//.../e` substitution aren't handled
correctly and break parsing.

There are barely any tests.

# AUTHOR

Lukas Mai, `<l.mai at web.de>`

Dmitry Karasik , `<dmitry at karasik.eu.org>`

# COPYRIGHT & LICENSE

Copyright (C) 2012, 2013 Lukas Mai.
Copyright (C) 2018 Dmitry Karasik

This program is free software; you can redistribute it and/or modify it
under the terms of either: the GNU General Public License as published
by the Free Software Foundation; or the Artistic License.

See http://dev.perl.org/licenses/ for more information.
