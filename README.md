Shape
=====

Shape is a tool for manipulating data matrices such as csv files.  At present,
no features are implemented.


Usage
-----

    shape [options] [<input_file>...]
    shape [options] <selector> [<input_file>...]
    shape [options] <selector> map <cmd> [<input_file>...]

    OPTIONS:
        -d --delim=<delim>      Specify axis delimiters.  This does not include
                                the primary axis, which is specified separately.
                                [Default: '\s']
        -p --pdelim=<pdelim>    Specify primary axis delimiters.  
                                [Default: '\n']
        -l --odelim=<odelim>    Specify output delimiters.
        -e --empty=<empty>      Specify the string that should be used to
                                represent empty cells.


Motivation
----------

For decades there have existed a plethora of command line tools used to process
streams of text.  The majority of these tools have operated either line by line,
or in some cases on space-separated pieces of text within the line.  I have
recently found it difficult to use command line tools on data files that have
higher degrees of granularity in the data. `shape` is my attempt to create a
bridge between multidimensional data files and command line text processing
tools such as `sort`, `unique`, and `comm`.


Data Matrix
-----------

A Data Matrix is any text file that contains data meant to represent an
N-dimensional matrix.  Individual cells in the matrix are separated by
delimiter characters, which more broadly separate indices along axes.

For example, a csv file is a data matrix where the primary axis is delimited by
a newline character, and the secondary axis is delimited by a comma.  However,
imagine a csv file where each column represents a 3D point by separating X, Y,
and Z values with a semicolon. Now we have a third axis.  This delimiter nesting
can be used to create a data matrix with an arbitrary number of dimensions.

Sometimes the contained data will want to use a delimiter as part of its data.
This can be accomplished in two ways.  The delimiter character can be escaped
by preceding it with an escape character (typically backslash), in order to
indicate that it should not be interpreted as a delimiter.  Alternatively, data
can be quoted by prepending an open quote character and appending a close quote
character.


### Delimiters

Delimiters are used to specify separations along axes in the data. By default,
a newline character delimits the primary axis, and one or more whitespace
characters delimit the secondary axis. Delimiters can be specified with the
`--delim` option.  The argument to this option should be a list of RE2 regular
expressions used to match axis delimiters, separated by the forward slash
character (e.g. `\s+/,/;`). If a forward slash must be used in a delimiter
expression, it can be escaped with the backslash character. 

It is assumed that the primary axis is typically delimited by a newline
character.  Users can specify a different primary axis delimiter with the
`--pdelim` option.

When matching delimiters with regular expressions, they are evaluated in order
from most significant (primary).

When it comes time to output a result, these delimiters must be recreated. To
specify output delimiters, use the `--odelim` option. They can be explicitly set
in a similar syntax to the `--delim` option (e.g. \s/,/;)`. Alternatively, the
user can use `--odelim=first` to tell shape to use the first axis delimiter it
encounters as the delimiter for all fields, or `--odelim=recent` to use the most
recent delimiter it encounters. 'first' and 'recent' are keywords that cannot be
used as output delimiters. If there is an incredible surge in demand for using
these words as delimiters, this additional functionality might be patched.


### Row Sizes

Sometimes data files have rows with mismatched sizes.  The size of an axis is
assumed to be the maximum number of cells found under a row of that axis.  If a
row of that axis has fewer cells than one of its counterparts, the remaining
cells are assumed to be an empty string.  This default empty cell can be
modified with the `--empty` option.