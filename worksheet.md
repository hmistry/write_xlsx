---
layout: default
title: Worksheet Method
---

### WORKSHEET METHODS
A new worksheet is created by calling the `add_worksheet()` method from a workbook object:

    worksheet1 = workbook.add_worksheet
    worksheet2 = workbook.add_worksheet

The following methods are available through a new worksheet:

* [write](#write)
* [write_number](#write_number)
* [write_string](#write_string)
* [write_rich_string](#write_rich_string)
* [keep_leading_zeros](#keep_leading_zeros)
* [write_blank](#write_blank)
* [write_row](#write_row)
* [write_col](#write_col)
* [write_date_time](#write_date_time)
* [write_url](#write_url)
* [write_url_range](#write_url_range)
* [write_formula](#write_formula)
* [write_comment](#write_comment)
* [show_comments](#show_comments)
* [set_comments_author](#set_comments_author)
* [insert_image](#insert_image)
* [insert_chart](#insert_chart)
* [insert_shape](#insert_shape)
* [insert_button](#insert_button)
* [data_validation](#data_validation)
* [conditional_formatting](#conditional_formatting)
* [add_sparkline](#add_sparkline)
* [add_table](#add_table)
* [name](#name)
* [activate](#activate)
* [select](#select)
* [hide](#hide)
* [set_first_sheet](#set_first_sheet)
* [protect](#protect)
* [set_selection](#set_selection)
* [set_row](#set_row)
* [set_default_row](#set_default_row)
* [set_column](#set_column)
* [outline_settings](#outline_settings)
* [freeze_panes](#freeze_panes)
* [split_panes](#split_panes)
* [merge_range](#merge_range)
* [merge_range_type](#merge_range_type)
* [set_zoom](#set_zoom)
* [right_to_left](#right_to_left)
* [hide_zero](#hide_zero)
* [set_tab_color](#set_tab_color)
* [autofilter](#autofilter)
* [filter_column](#filter_column)
* [filter_column_list](#filter_column_list)

#### <a name="cell-notation" class="anchor" href="#cell-notation"><span class="octicon octicon-link" /></a>Cell notation

WriteXLSX supports two forms of notation to designate the position of cells:
Row-column notation and A1 notation.

Row-column notation uses a zero based index for both row and column
while A1 notation uses the standard Excel alphanumeric sequence of column letter and 1-based row.

For example:

    (0, 0)      # The top left cell in row-column notation.
    ('A1')      # The top left cell in A1 notation.

    (1999, 29)  # Row-column notation.
    ('AD2000')  # The same cell in A1 notation.

Row-column notation is useful if you are referring to cells programmatically:

    (0..9).each do |i|
        worksheet.write( i, 0, 'Hello' )    # Cells A1 to A10
    end

A1 notation is useful for setting up a worksheet manually and for working with formulas:

    worksheet.write('H1', 200)
    worksheet.write('H2', '=H1+1')

In formulas and applicable methods you can also use the A:A column notation:

    worksheet.write('A1', '=SUM(B:B)')


For simplicity, the parameter lists for the worksheet method calls in the following sections
are given in terms of row-column notation.
In all cases it is also possible to use A1 notation.

Note: in Excel it is also possible to use a R1C1 notation.
This is not supported by WriteXLSX.

#### <a name="write" class="anchor" href="#write"><span class="octicon octicon-link" /></a>write(row, column, token, format)

Excel makes a distinction between data types such as strings, numbers, blanks, formulas and hyperlinks.
To simplify the process of writing data the `write()` method acts as a general alias
for several more specific methods:

* [write_string](#write_string)
* [write_number](#write_number)
* [write_blank](#write_blank)
* [write_formula](#write_formula)
* [write_url](#write_url)
* [write_row](#write_row)
* [write_col](#write_col)

The general rule is that if the data looks like a something then a something is written.
Here are some examples in both row-column and A1 notation:

                                                     # Same as:
    worksheet.write(0, 0, 'Hello'                ) # write_string()
    worksheet.write(1, 0, 'One'                  ) # write_string()
    worksheet.write(2, 0,  2                     ) # write_number()
    worksheet.write(3, 0,  3.00001               ) # write_number()
    worksheet.write(4, 0,  ""                    ) # write_blank()
    worksheet.write(5, 0,  ''                    ) # write_blank()
    worksheet.write(6, 0,  undef                 ) # write_blank()
    worksheet.write(7, 0                         ) # write_blank()
    worksheet.write(8, 0,  'http://www.perl.com/') # write_url()
    worksheet.write('A9',  'ftp://ftp.cpan.org/' ) # write_url()
    worksheet.write('A10', 'internal:Sheet1!A1'  ) # write_url()
    worksheet.write('A11', 'external:c:\foo.xlsx') # write_url()
    worksheet.write('A12', '=A3 + 3*A4'          ) # write_formula()
    worksheet.write('A13', '=SIN(PI()/4)'        ) # write_formula()
    worksheet.write('A14', \@array               ) # write_row()
    worksheet.write('A15', [\@array]             ) # write_col()

    # Write an array formula. Not available in Spreadsheet::WriteExcel.
    worksheet.write('A19', '{=SUM(A1:B1*A2:B2)}' ) # write_formula()

The "looks like" rule is defined by regular expressions:

`write_number()` if `token` is a number based on the following regex:
`token =~ /^(\[+-\]?)(?=\d|\.\d)\d\*(\.\d\*)?(\[Ee\](\[+-\]?\d+))?$/`.

`write_blank()` if `token` is nil or a blank string: `nil, "" or ''`.

`write_url()` if `token` is a http, https, ftp or mailto URL based on the following regexes:
`token =~ m|^\[fh\]tt?ps?\://| or token =~ m|^mailto:|`.

`write_url()` if `token` is an internal or external sheet reference based on the following regex:
`token =~ \[^(in|ex)ternal:\]`.

`write_formula()` if the first character of `token` is `"="`.

`write_array_formula()` if the `token` matches `/^{=.\*}$/`.

`write_row()` if `token` is an array.

`write_col()` if `token` is an array of array.

`write_string()` if none of the previous conditions apply.

The `format` parameter is optional.
It should be a valid Format object, see ["CELL FORMATTING"](#cell_formatting.html):

    format = workbook.add_format
    format.set_bold
    format.set_color('red')
    format.set_align('center')

    worksheet.write(4, 0, 'Hello', format )    # Formatted string

The `write()` method will ignore empty strings or `nil` tokens unless a `format` is
also supplied.
As such you needn't worry about special handling for empty or `nil` values in your data.
See also the [`write_blank()`](#write_blank) method.

The write methods return:

    0 for success.
    -1 for insufficient number of arguments.
    -2 for row or column out of bounds.
    -3 for string too long.

#### <a name="write_number" class="anchor" href="#write_number"><span class="octicon octicon-link" /></a>write_number(row, column, number, format = nil)

Write an integer or a float to the cell specified by `row` and `column`:

    worksheet.write_number(0, 0, 123456)
    worksheet.write_number('A2', 2.3451)

See the note about ["Cell notation"](#cell-notation).
The `format` parameter is optional.

In general it is sufficient to use the `write()` method.

Note: some versions of Excel 2007 do not display the calculated values of formulas
written by WriteXLSX. Applying all available Service Packs to Excel should fix this.

#### <a name="write_string" class="anchor" href="#write_string"><span class="octicon octicon-link" /></a>write_string(row, column, string, format = nil)

Write a string to the cell specified by `row` and `column`:

    worksheet.write_string(0, 0, 'Your text here')
    worksheet.write_string('A2', 'or here')

The maximum string size is 32767 characters.
However the maximum string segment that Excel can display in a cell is 1000.
All 32767 characters can be displayed in the formula bar.

The `format` parameter is optional.

The `write()` method will also handle strings in UTF-8 format.
See also the `unicode_\*.rb` programs in the examples directory of the distro.

In general it is sufficient to use the `write()` method.
However, you may sometimes wish to use the `write_string()` method to write data
that looks like a number but that you don't want treated as a number.
For example, zip codes or phone numbers:

    # Write as a plain string
    worksheet.write_string('A1', '01209')

However, if the user edits this string Excel may convert it back to a number.
To get around this you can use the Excel text format @:

    # Format as a string. Doesn't change to a number when edited
    format1 = workbook.add_format(:num_format => '@')
    worksheet.write_string('A2', '01209', format1)

See also the note about ["Cell notation"](#cell-notation).

#### <a name="write_rich_string" class="anchor" href="#write_rich_string"><span class="octicon octicon-link" /></a>write_rich_string(row, column, format, string, ..., cell_format = nil)

The `write_rich_string()` method is used to write strings with multiple formats.
For example to write the string "This is bold and this is italic" you would use the following:

    bold   = workbook.add_format("bold   => 1)
    italic = workbook.add_format("italic => 1)

    worksheet.write_rich_string('A1',
      'This is ', bold, 'bold', ' and this is ', italic, 'italic')

The basic rule is to break the string into fragments and put a `format` object before the fragment
that you want to format. For example:

    # Unformatted string.
      'This is an example string'

    # Break it into fragments.
      'This is an ', 'example', ' string'

    # Add formatting before the fragments you want formatted.
      'This is an ', format, 'example', ' string'

    # In WriteXLSX
    worksheet.write_rich_string('A1',
      'This is an ', format, 'example', ' string')

String fragments that don't have a format are given a default format. So for example when writing
the string "Some bold text" you would use the first example below but it would be equivalent to the second:

    # With default formatting:
    bold    = workbook.add_format(:bold => 1)

    worksheet.write_rich_string('A1',
        'Some ', bold, 'bold', ' text')

    # Or more explicitly:
    bold    = workbook.add_format(:bold => 1)
    default = workbook.add_format

    worksheet.write_rich_string('A1',
      default, 'Some ', bold, 'bold', default, ' text')

As with Excel, only the font properties of the format such as font name, style, size, underline,
color and effects are applied to the string fragments.
Other features such as border, background, text wrap and alignment must be applied to the cell.

The `write_rich_string()` method allows you to do this by using the last argument as a cell format
(if it is a format object). The following example centers a rich string in the cell:

    bold   = workbook.add_format(:bold  => 1)
    center = workbook.add_format(:align => 'center')

    worksheet.write_rich_string('A5',
      'Some ', bold, 'bold text', ' centered', center)

See the `rich_strings.rb` example in the distro for more examples.

    bold   = workbook.add_format(:bold        => 1)
    italic = workbook.add_format(:italic      => 1)
    red    = workbook.add_format(:color       => 'red')
    blue   = workbook.add_format(:color       => 'blue')
    center = workbook.add_format(:align       => 'center')
    super  = workbook.add_format(:font_script => 1)

    # Write some strings with multiple formats.
    worksheet.write_rich_string('A1',
      'This is ', bold, 'bold', ' and this is ', italic, 'italic')

    worksheet.write_rich_string('A3',
      'This is ', red, 'red', ' and this is ', blue, 'blue')

    worksheet.write_rich_string('A5',
      'Some ', bold, 'bold text', ' centered', center)

    worksheet.write_rich_string('A7',
      italic, 'j = k', super, '(n-1)', center)

As with `write_sting()` the maximum string size is 32767 characters.
See also the note about ["Cell notation"](#cell-notation).

#### <a name="write_blank" class="anchor" href="#write_blank"><span class="octicon octicon-link" /></a>write_blank(row, column, format)

Write a blank cell specified by `row` and `column`:

    worksheet.write_blank(0, 0, format)

This method is used to add formatting to a cell which doesn't contain a string or number value.

Excel differentiates between an "Empty" cell and a "Blank" cell.
An "Empty" cell is a cell which doesn't contain data whilst a "Blank" cell is a cell
which doesn't contain data but does contain formatting.
Excel stores "Blank" cells but ignores "Empty" cells.

As such, if you write an empty cell without formatting it is ignored:

    worksheet.write('A1', nil, format)    # write_blank()
    worksheet.write('A2', nil)            # Ignored

This seemingly uninteresting fact means that you can write arrays of data
without special treatment for nil or empty string values.

See the note about ["Cell notation"](#cell-notation).

#### <a name="write_row" class="anchor" href="#write_row"><span class="octicon octicon-link" /></a>write_row(row, column, array, format = nil)

The `write_row()` method can be used to write a 1D or 2D array of data in one go.

This is useful for converting the results of a database query into an Excel worksheet.
The `write()` method is then called for each element of the data. For example:

    array = ['awk', 'gawk', 'mawk']

    worksheet.write_row(0, 0, array)

    # The above example is equivalent to:
    worksheet.write(0, 0, array[0])
    worksheet.write(0, 1, array[1])
    worksheet.write(0, 2, array[2])

As with all of the `write` methods the `format` parameter is optional.
If a format is specified it is applied to all the elements of the data array.

You can write 2D arrays of data in one go. For example:

    eec =  [
                ['maggie', 'milly', 'molly', 'may'  ],
                [13,       14,      15,      16     ],
                ['shell',  'star',  'crab',  'stone']
           ]

    worksheet.write_row('A1', @eec)

Would produce a worksheet as follows:

     -----------------------------------------------------------
    |   |    A    |    B    |    C    |    D    |    E    | ...
     -----------------------------------------------------------
    | 1 | maggie  | 13      | shell   | ...     |  ...    | ...
    | 2 | milly   | 14      | star    | ...     |  ...    | ...
    | 3 | molly   | 15      | crab    | ...     |  ...    | ...
    | 4 | may     | 16      | stone   | ...     |  ...    | ...
    | 5 | ...     | ...     | ...     | ...     |  ...    | ...
    | 6 | ...     | ...     | ...     | ...     |  ...    | ...

To write the data in a row-column order refer to the `write_col()` method below.

Any `nil` values in the data will be ignored unless a `format` is applied to the data,
in which case a formatted blank cell will be written.
In either case the appropriate `row` or `column` value will still be incremented.

The `write_row()` method returns the first error encountered when writing the elements of the data
or zero if no errors were encountered.
See the return values described for the `write()` method above.

See also the `write_arrays.rb` program in the examples directory of the distro.

#### <a name="write_col" class="anchor" href="#write_col"><span class="octicon octicon-link" /></a>write_col(row, column, array, format = nil)

The `write_col()` method can be used to write a 1D or 2D array of data in one go.
This is useful for converting the results of a database query into an Excel worksheet.
The `write()` method is then called for each element of the data. For example:

    array = ['awk', 'gawk', 'mawk']

    worksheet.write_col(0, 0, array)

    # The above example is equivalent to:
    worksheet.write(0, 0, array[0])
    worksheet.write(1, 0, array[1])
    worksheet.write(2, 0, array[2])

As with all of the `write` methods the `format` parameter is optional.
If a `format` is specified it is applied to all the elements of the data array.

This allows you to write 2D arrays of data in one go. For example:

    eec =  [
                ['maggie', 'milly', 'molly', 'may'  ],
                [13,       14,      15,      16     ],
                ['shell',  'star',  'crab',  'stone']
           ]

    worksheet.write_col('A1', eec)

Would produce a worksheet as follows:

     -----------------------------------------------------------
    |   |    A    |    B    |    C    |    D    |    E    | ...
     -----------------------------------------------------------
    | 1 | maggie  | milly   | molly   | may     |  ...    | ...
    | 2 | 13      | 14      | 15      | 16      |  ...    | ...
    | 3 | shell   | star    | crab    | stone   |  ...    | ...
    | 4 | ...     | ...     | ...     | ...     |  ...    | ...
    | 5 | ...     | ...     | ...     | ...     |  ...    | ...
    | 6 | ...     | ...     | ...     | ...     |  ...    | ...

To write the data in a column-row order refer to the `write_row()` method above.

Any nil values in the data will be ignored unless a format is applied to the data,
in which case a formatted blank cell will be written.
In either case the appropriate `row` or `column` value will still be incremented.

The `write_col()` method returns the first error encountered when writing the elements of the data
or zero if no errors were encountered. See the return values described for the `write()` method above.

See also the `write_arrays.rb` program in the examples directory of the distro.

#### <a name="write_date_time" class="anchor" href="#write_date_time"><span class="octicon octicon-link" /></a>write_date_time(row, col, date_string, format)

The `write_date_time()` method can be used to write a date or time to the cell specified by `row` and `column`:

    worksheet.write_date_time('A1', '2004-05-13T23:20', date_format)

The `date_string` should be in the following format:

    yyyy-mm-ddThh:mm:ss.sss

This conforms to an ISO8601 date but it should be noted that the full range of ISO8601 formats are not supported.

The following variations on the `date_string` parameter are permitted:

    yyyy-mm-ddThh:mm:ss.sss         # Standard format
    yyyy-mm-ddT                     # No time
              Thh:mm:ss.sss         # No date
    yyyy-mm-ddThh:mm:ss.sssZ        # Additional Z (but not time zones)
    yyyy-mm-ddThh:mm:ss             # No fractional seconds
    yyyy-mm-ddThh:mm                # No seconds

Note that the `T` is required in all cases.

A date should always have a `format`, otherwise it will appear as a number,
see ["DATES AND TIME IN EXCEL"](dates_and_time.html) and ["CELL FORMATTING"](cell_formatting.html).
Here is a typical example:

    date_format = workbook.add_format(:num_format => 'mm/dd/yy')
    worksheet.write_date_time('A1', '2004-05-13T23:20', date_format)

Valid dates should be in the range 1900-01-01 to 9999-12-31,
for the 1900 epoch and 1904-01-01 to 9999-12-31, for the 1904 epoch.
As with Excel, dates outside these ranges will be written as a string.

See also the `date_time.rb` program in the examples directory of the distro.

#### <a name="write_url" class="anchor" href="#write_url"><span class="octicon octicon-link" /></a>write_url(row, col, url, format = nil, label = nil)

Write a hyperlink to a URL in the cell specified by `row` and `column`.
The hyperlink is comprised of two elements: the visible label and the invisible link.
The visible label is the same as the link unless an alternative label is specified.
The `label` parameter is optional.
The `label` is written using the `write()` method.
Therefore it is possible to write strings, numbers or formulas as labels.

The `format` parameter is also optional, however, without a format the link won't look like a link.

The suggested format is:

    format = workbook.add_format(:color => 'blue', :underline => 1)

Note, this behaviour is different from writeexcel gem which provides a default hyperlink format
if one isn't specified by the user.

There are four web style URI's supported: `http://`, `https://`, `ftp://` and `mailto:`:

    worksheet.write_url(0, 0, 'ftp://www.ruby.org/',       format)
    worksheet.write_url('A3', 'http://www.ruby.com/',      format)
    worksheet.write_url('A4', 'mailto:jmcnamara@cpan.org', format)

You can display an alternative string using the `label` parameter:

    worksheet.write_url(1, 0, 'http://www.ruby.com/', format, 'Ruby )

If you wish to have some other cell data such as a number or a formula
you can overwrite the cell using another call to `write_\*()`:

    worksheet.write_url('A1', 'http://www.ruby.com/')

    # Overwrite the URL string with a formula. The cell is still a link.
    worksheet.write_formula('A1', '=1+1', format)

There are two local URIs supported: `internal:` and `external:`.
These are used for hyperlinks to internal worksheet references or external workbook and worksheet references:

    worksheet.write_url('A6',  'internal:Sheet2!A1',              format)
    worksheet.write_url('A7',  'internal:Sheet2!A1',              format)
    worksheet.write_url('A8',  'internal:Sheet2!A1:B2',           format)
    worksheet.write_url('A9',  %q{internal:'Sales Data'!A1},      format)
    worksheet.write_url('A10', 'external:c:\temp\foo.xlsx',       format)
    worksheet.write_url('A11', 'external:c:\foo.xlsx#Sheet2!A1',  format)
    worksheet.write_url('A12', 'external:..\foo.xlsx',            format)
    worksheet.write_url('A13', 'external:..\foo.xlsx#Sheet2!A1',  format)
    worksheet.write_url('A13', 'external:\\\\NET\share\foo.xlsx', format)

All of the these URI types are recognised by the `write()` method, see above.

Worksheet references are typically of the form `Sheet1!A1`.
You can also refer to a worksheet range using the standard Excel notation: `Sheet1!A1:B2`.

In external links the workbook and worksheet name must be separated by the `#` character: `external:Workbook.xlsx#Sheet1!A1'`.

You can also link to a named range in the target worksheet.
For example say you have a named range called my_name in the workbook c:\temp\foo.xlsx you could link to it as follows:

    worksheet.write_url('A14', 'external:c:\temp\foo.xlsx#my_name')

Excel requires that worksheet names containing spaces or non alphanumeric characters
are single quoted as follows 'Sales Data'!A1.
If you need to do this in a single quoted string then you can either escape the single quotes \\'
or use the quote operator %q{}.

Links to network files are also supported.
MS/Novell Network files normally begin with two back slashes as follows \\\\NETWORK\etc.
In order to generate this in a single or double quoted string
you will have to escape the backslashes, '\\\\\\\\NETWORK\\etc'.

If you are using double quote strings then you should be careful to escape anything
that looks like a metacharacter.

Finally, you can avoid most of these quoting problems by using forward slashes.
These are translated internally to backslashes:

    worksheet.write_url('A14', "external:c:/temp/foo.xlsx")
    worksheet.write_url('A15', 'external://NETWORK/share/foo.xlsx')

Note: WriteXLSX will escape the following characters in URLs as required by Excel:
`\s` `"` `<` `>` `\` `[` `]` `backquote` `^` `{` `}` unless the URL already contains `%xx` style escapes.
In which case it is assumed that the URL was escaped correctly by the user and will by passed directly to Excel.

See also, the note about ["Cell notation"](#cell-notation).

#### <a name="write_formul" class="anchor" href="#write_formul"><span class="octicon octicon-link" /></a>write_formula(row, column, formula, format = nil, value = nil)

Write a formula or function to the cell specified by `row` and `column`:

    worksheet.write_formula(0, 0, '=$B$3 + B4')
    worksheet.write_formula(1, 0, '=SIN(PI()/4)')
    worksheet.write_formula(2, 0, '=SUM(B1:B5)')
    worksheet.write_formula('A4', '=IF(A3>1,"Yes", "No")')
    worksheet.write_formula('A5', '=AVERAGE(1, 2, 3, 4)')
    worksheet.write_formula('A6', '=DATEVALUE("1-Jan-2001")')

Array formulas are also supported:

    worksheet.write_formula('A7', '{=SUM(A1:B1\*A2:B2)}')

See also the `write_array_formula()` method below.

See the note about ["Cell notation"](#cell-notation).
For more information about writing Excel formulas see ["FORMULAS AND FUNCTIONS IN EXCEL"](formulas_and_functions.html)

If required, it is also possible to specify the calculated value of the `formula`.
This is occasionally necessary when working with non-Excel applications that don't calculate the value
of the formula.
The calculated `value` is added at the end of the argument list:

    worksheet.write('A1', '=2+2', format, 4)

However, this probably isn't something that you will ever need to do.
If you do use this feature then do so with care.

#### <a name="write_array_formula" class="anchor" href="#write_array_formula"><span class="octicon octicon-link" /></a>write_array_formula(first_row, first_col, last_row, last_col, formula, format, value)

Write an array formula to a cell range.
In Excel an array formula is a formula that performs a calculation on a set of values.
It can return a single value or a range of values.

An array formula is indicated by a pair of braces around the formula: `{=SUM(A1:B1\*A2:B2)}`.
If the array formula returns a single value then the `first_` and `last_` parameters should be the same:

    worksheet.write_array_formula('A1:A1', '{=SUM(B1:C1\*B2:C2)}')

It this case however it is easier to just use the `write_formula()` or `write()` methods:

    # Same as above but more concise.
    worksheet.write('A1', '{=SUM(B1:C1\*B2:C2)}')
    worksheet.write_formula('A1', '{=SUM(B1:C1\*B2:C2)}')

For array formulas that return a range of values you must specify the range
that the return values will be written to:

    worksheet.write_array_formula('A1:A3',    '{=TREND(C1:C3,B1:B3)}')
    worksheet.write_array_formula(0, 0, 2, 0, '{=TREND(C1:C3,B1:B3)}')

If required, it is also possible to specify the calculated value of the formula.
This is occasionally necessary when working with non-Excel applications that don't calculate the value of the formula.
The calculated `value` is added at the end of the argument list:

    worksheet.write_array_formula('A1:A3', '{=TREND(C1:C3,B1:B3)}', format, 105)

In addition, some early versions of Excel 2007 don't calculate the values of array formulas when they aren't supplied.
Installing the latest Office Service Pack should fix this issue.

See also the `array_formula.rb` program in the examples directory of the distro.

Note: Array formulas are not supported by writeexcel gem.

#### <a name="store_formula" class="anchor" href="#store_formula"><span class="octicon octicon-link" /></a>store_formula(formula)

Deprecated. This is a writeexcel gem method that is no longer required by WriteXLSX. See below.

#### <a name="repeat_formula" class="anchor" href="#repeat_formula"><span class="octicon octicon-link" /></a>repeat_formula(row, col, formula, format)

Deprecated. This is a writeexcel gem method that is no longer required by WriteXLSX.

In writeexcel gem, it was computationally expensive to write formulas since they were parsed
by a recursive descent parser. The `store_formula()` and `repeat_formula()` methods were used
as a way of avoiding the overhead of repeated formulas by reusing a pre-parsed formula.

In WriteXLSX this is no longer necessary since it is just as quick to write a formula
as it is to write a string or a number.

The methods remain for backward compatibility but new WriteXLSX programs shouldn't use them.

#### <a name="write_comment" class="anchor" href="#write_comment"><span class="octicon octicon-link" /></a>write_comment(row, column, string, ... )

The `write_comment()` method is used to add a comment to a cell.
A cell comment is indicated in Excel by a small red triangle in the upper right-hand corner of the cell.
Moving the cursor over the red triangle will reveal the comment.

The following example shows how to add a comment to a cell:

    worksheet.write(        2, 2, 'Hello')
    worksheet.write_comment(2, 2, 'This is a comment.')

As usual you can replace the `row` and `column` parameters with an A1 cell reference.
See the note about ["Cell notation"](#cell-notation).

    worksheet.write(        'C3', 'Hello')
    worksheet.write_comment('C3', 'This is a comment.')

In addition to the basic 3 argument form of `write_comment()` you can pass in several optional
key/value pairs to control the format of the comment. For example:

    worksheet.write_comment('C3', 'Hello', :visible => 1, :author => 'Ruby')

Most of these options are quite specific and in general the default comment behaviour will be all that you need.
However, should you need greater control over the format of the cell comment the following options are available:

    :author
    :visible
    :x_scale
    :width
    :y_scale
    :height
    :color
    :start_cell
    :start_row
    :start_col
    :x_offset
    :y_offset

##### Option: author
This option is used to indicate who is the author of the cell comment.
Excel displays the author of the comment in the status bar at the bottom of the worksheet.
This is usually of interest in corporate environments where several people might review
and provide comments to a workbook.

    worksheet.write_comment('C3', 'Atonement', :author => 'Ian McEwan')

The default author for all cell comments can be set
using the `set_comments_author()` method (see below).

    worksheet.set_comments_author('Perl')

##### Option: visible
This option is used to make a cell comment visible when the worksheet is opened.
The default behaviour in Excel is that comments are initially hidden.
However, it is also possible in Excel to make individual or all comments visible.
In WriteXLSX individual comments can be made visible as follows:

    worksheet.write_comment('C3', 'Hello', :visible => 1)

It is possible to make all comments in a worksheet visible using the
`show_comments()` worksheet method (see below). Alternatively, if all of the cell
comments have been made visible you can hide individual comments:

    worksheet.write_comment('C3', 'Hello', :visible => 0)

##### Option: x_scale
This option is used to set the width of the cell comment box as a factor of
the default width.

    worksheet.write_comment('C3', 'Hello', :x_scale => 2)
    worksheet.write_comment('C4', 'Hello', :x_scale => 4.2)

##### Option: width
This option is used to set the width of the cell comment box explicitly in pixels.

    worksheet.write_comment('C3', 'Hello', :width => 200)

##### Option: y_scale
This option is used to set the height of the cell comment box as a factor
of the default height.

    worksheet.write_comment('C3', 'Hello', :y_scale => 2)
    worksheet.write_comment('C4', 'Hello', :y_scale => 4.2)

##### Option: height
This option is used to set the height of the cell comment box explicitly in pixels.

    worksheet.write_comment('C3', 'Hello', :height => 200)

##### Option: color
This option is used to set the background colour of cell comment box.
You can use one of the named colours recognised by WriteXLSX or a colour index.
See "COLOURS IN EXCEL".

    worksheet.write_comment('C3', 'Hello', :color => 'green')
    worksheet.write_comment('C4', 'Hello', :color => 0x35)      # Orange

##### Option: start_cell
This option is used to set the cell in which the comment will appear.
By default Excel displays comments one cell to the right and one cell above the cell
to which the comment relates. However, you can change this behaviour if you wish.
In the following example the comment which would appear by default in cell D2 is moved to E2.

    worksheet.write_comment('C3', 'Hello', :start_cell => 'E2')

##### Option: start_row
This option is used to set the row in which the comment will appear.
See the `:start_cell` option above. The row is zero indexed.

    worksheet.write_comment('C3', 'Hello', :start_row => 0)

##### Option: start_col
This option is used to set the column in which the comment will appear.
See the `:start_cell` option above. The column is zero indexed.

    worksheet.write_comment('C3', 'Hello', :start_col => 4)

##### Option: x_offset
This option is used to change the x offset, in pixels, of a comment within a cell:

    worksheet.write_comment('C3', $comment, :x_offset => 30)

##### Option: y_offset
This option is used to change the y offset, in pixels, of a comment within a cell:

    worksheet.write_comment('C3', $comment, :x_offset => 30)

You can apply as many of these options as you require.

###### Note about using options that adjust the position of the cell comment such as `:start_cell`, `:start_row`, `:start_col`, `:x_offset` and `:y_offset`:
Excel only displays offset cell comments when they are displayed as "visible".
Excel does not display hidden cells as moved when you mouse over them.

###### Note about row height and comments.
If you specify the height of a row that contains a comment then WriteXLSX will
adjust the height of the comment to maintain the default or user specified
dimensions. However, the height of a row can also be adjusted automatically
by Excel if the text wrap property is set or large fonts are used in the cell.
This means that the height of the row is unknown to the module at run time and
thus the comment box is stretched with the row. Use the `set_row()` method to
specify the row height explicitly and avoid this problem.

#### <a name="show_comments" class="anchor" href="#show_comments"><span class="octicon octicon-link" /></a>show_comments

This method is used to make all cell comments visible when a worksheet is opened.

    worksheet.show_comments

Individual comments can be made visible using the visible parameter of the `write_comment` method (see above):

    worksheet.write_comment('C3', 'Hello', :visible => 1)

If all of the cell comments have been made visible you can hide individual comments as follows:

    worksheet.show_comments
    worksheet.write_comment('C3', 'Hello', :visible => 0)

#### <a name="set_comments_author" class="anchor" href="#set_comments_author"><span class="octicon octicon-link" /></a>set_comments_author

This method is used to set the default author of all cell comments.

    worksheet.set_comments_author('Perl')

Individual comment authors can be set using the author parameter of the `write_comment` method (see above).

The default comment author is an empty string, '', if no author is specified.

#### <a name="insert_image" class="anchor" href="#insert_image"><span class="octicon octicon-link" /></a>insert_image(row, col, filename, x, y, x_scale, y_scale)

Partially supported. Currently only works for 96 dpi images.

This method can be used to insert a image into a worksheet.
The image can be in PNG, JPEG or BMP format.
The `x`, `y`, `x_scale` and `y_scale` parameters are optional.

    worksheet1.insert_image('A1', 'ruby.bmp')
    worksheet2.insert_image('A1', '../images/ruby.bmp')
    worksheet3.insert_image('A1', '.c:\images\ruby.bmp')

The parameters `x` and `y` can be used to specify an offset from the top left
hand corner of the cell specified by `row` and `col`.
The offset values are in pixels.

    worksheet1.insert_image('A1', 'ruby.bmp', 32, 10)

The offsets can be greater than the width or height of the underlying cell.
This can be occasionally useful if you wish to align two or more images relative to the same cell.

The parameters `x_scale` and `y_scale` can be used to scale the inserted image
horizontally and vertically:

    # Scale the inserted image: width x 2.0, height x 0.8
    worksheet.insert_image('A1', 'ruby.bmp', 0, 0, 2, 0.8)

Note: you must call `set_row()` or `set_column()` before `insert_image()`
if you wish to change the default dimensions of any of the rows or columns
that the image occupies. The height of a row can also change if you use a font
that is larger than the default. This in turn will affect the scaling of your image.
To avoid this you should explicitly set the height of the row using `set_row()`
if it contains a font size that will change the row height.

BMP images must be 24 bit, true colour, bitmaps.
In general it is best to avoid BMP images since they aren't compressed.

#### <a name="insert_chart" class="anchor" href="#insert_chart"><span class="octicon octicon-link" /></a>insert_chart(row, col, chart, x, y, x_scale, y_scale)

This method can be used to insert a Chart object into a worksheet.
The Chart must be created by the `add_chart()` Workbook method and it must have
the embedded option set.

    chart = workbook.add_chart(:type => 'line', :embedded => 1)

    # Configure the chart.
    ...

    # Insert the chart into the a worksheet.
    worksheet.insert_chart('E2', chart)

See `add_chart()` for details on how to create the Chart object
and [Chart](chart.html) for details on how to configure it.
See also the chart_\*.rb programs in the examples directory of the distro.

The `x`, `y`, `x_scale` and `y_scale` parameters are optional.

The parameters `x` and `y` can be used to specify an offset from the top left
hand corner of the cell specified by `row` and `col`.
The offset values are in pixels.

    worksheet1.insert_chart('E2', chart, 3, 3)

The parameters `x_scale` and `y_scale` can be used to scale the inserted chart
horizontally and vertically:

    # Scale the width by 120% and the height by 150%
    worksheet.insert_chart('E2', chart, 0, 0, 1.2, 1.5)

#### <a name="insert_shape" class="anchor" href="#insert_shape"><span class="octicon octicon-link" /></a>insert_shape(row, col, shape, x, y, x_scale, y_scale)

This method can be used to insert a Shape object into a worksheet.
The Shape must be created by the `add_shape()` Workbook method.

    shape = workbook.add_shape(:name => 'My Shape', :type => 'plus')

    # Configure the shape.
    shape.set_text('foo')
    ...

    # Insert the shape into the a worksheet.
    worksheet.insert_shape('E2', shape)

See `add_shape()` for details on how to create the Shape object
and [Shape](shape.html) for details on how to configure it.

The `x`, `y`, `x_scale` and `y_scale` parameters are optional.

The parameters `x` and `y` can be used to specify an offset from the top left
hand corner of the cell specified by `row` and `col`.
The offset values are in pixels.

    worksheet1.insert_shape('E2', chart, 3, 3)

The parameters `x_scale` and `y_scale` can be used to scale the inserted shape
horizontally and vertically:

    # Scale the width by 120% and the height by 150%
    worksheet.insert_shape('E2', shape, 0, 0, 1.2, 1.5)

See also the `shape\*.rb` programs in the examples directory of the distro.

#### <a name="insert_button" class="anchor" href="#insert_button"><span class="octicon octicon-link" /></a>insert_button(row, col, properties)

The `insert_button()` method can be used to insert an Excel form button into
a worksheet.

This method is generally only useful when used in conjunction with the
[Workbook#add_vba_project()](workbook.html#add_vba_project) method to tie the
button to a macro from an embedded VBA project:

    workbook  = WriteXLSX.new('file.xlsm')
    ...
    workbook.add_vba_project('./vbaProject.bin')

    worksheet.insert_button('C2', :macro => 'my_macro')

The properties of the button that can be set are:

    :macro
    :caption
    :width
    :height
    :x_scale
    :y_scale
    :x_offset
    :y_offset

##### Option: macro
This option is used to set the macro that the button will invoke when the user
clicks on it. The macro should be included using the
[Workbook#add_vba_project()](workbook.html#add_vba_project) method shown above.

    worksheet.insert_button('C2', :macro => 'my_macro')

The default macro is ButtonX_Click where X is the button number.

##### Option: caption
This option is used to set the caption on the button.
The default is Button X where X is the button number.

    worksheet.insert_button('C2', :macro => 'my_macro', :caption => 'Hello')

##### Option: width
This option is used to set the width of the button in pixels.

    worksheet.insert_button('C2', :macro => 'my_macro', :width => 128)

The default button width is 64 pixels which is the width of a default cell.

##### Option: height
This option is used to set the height of the button in pixels.

    worksheet.insert_button('C2', :macro => 'my_macro', :height => 40)

The default button height is 20 pixels which is the height of a default cell.

##### Option: x_scale
This option is used to set the width of the button as a factor of the default width.

    worksheet.insert_button('C2', :macro => 'my_macro', :x_scale => 2.0)

##### Option: y_scale
This option is used to set the height of the button as a factor of the default height.

    worksheet.insert_button('C2', :macro => 'my_macro', :y_scale => 2.0)

##### Option: x_offset
This option is used to change the x offset, in pixels, of a button within a cell:

    worksheet.insert_button('C2', :macro => 'my_macro', :x_offset => 2)

##### Option: y_offset
This option is used to change the y offset, in pixels, of a comment within a cell.

Note: Button is the only Excel form element that is available in WriteXLSX.
Form elements represent a lot of work to implement and the underlying VML syntax
isn't very much fun.

#### <a name="data_validation" class="anchor" href="#data_validation"><span class="octicon octicon-link" /></a>data_validation()

The `data_validation()` method is used to construct an Excel data validation
or to limit the user input to a dropdown list of values.

    worksheet.data_validation('B3',
        {
            :validate => 'integer',
            :criteria => '>',
            :value    => 100
        })

    worksheet.data_validation('B5:B9',
        {
            :validate => 'list',
            :value    => ['open', 'high', 'close']
        })

This method contains a lot of parameters and is described in detail in
a separate section ["DATA VALIDATION IN EXCEL"](data_validation.html).

See also the `data_validate.rb` program in the examples directory of the distro

#### <a name="conditional_formatting" class="anchor" href="#conditional_formatting"><span class="octicon octicon-link" /></a>conditional_formatting()

The `conditional_formatting()` method is used to add formatting to a cell
or range of cells based on user defined criteria.

    worksheet.conditional_formatting( 'A1:J10',
        {
            :type     => 'cell',
            :criteria => '>=',
            :value    => 50,
            :format   => format1
        }
    )

This method contains a lot of parameters and is described in detail
in a separate section ["CONDITIONAL FORMATTING IN EXCEL"](conditional_formatting.html).

See also the `conditional_format.rb` program in the examples directory of the distro

#### <a name="add_sparkline" class="anchor" href="#add_sparkline"><span class="octicon octicon-link" /></a>add_sparkline()

The `add_sparkline()` worksheet method is used to add sparklines to a cell or a range of cells.

    worksheet.add_sparkline(
        {
            :location => 'F2',
            :range    => 'Sheet1!A2:E2',
            :type     => 'column',
            :style    => 12
        }
    )

This method contains a lot of parameters and is described in detail
in a separate section ["SPARKLINES IN EXCEL"](sparklines.html).

See also `sparklines1.rb` and `sparklines2.rb` example programs
in the examples directory of the distro.

Note: Sparklines are a feature of Excel 2010+ only.
You can write them to an XLSX file that can be read by Excel 2007
but they won't be displayed.

#### <a name="add_table" class="anchor" href="#add_table"><span class="octicon octicon-link" /></a>add_table()

The `add_table()` method is used to group a range of cells into an Excel Table.

    worksheet.add_table('B3:F7', { ... } )

This method contains a lot of parameters and is described in detail
in a separate section ["TABLES IN EXCEL"](tables.html).

See also the `tables.rb` program in the examples directory of the distro

#### <a name="name" class="anchor" href="#name"><span class="octicon octicon-link" /></a>name()

The `name()` method is used to retrieve the name of a worksheet. For example:

    workbook.sheets.each do |sheet|
        print sheet.name
    end

For reasons related to the design of WriteXLSX and to the internals of Excel
there is no `set_name()` method.
The only way to set the worksheet name is via the `add_worksheet()` method.

#### <a name="activate" class="anchor" href="#activate"><span class="octicon octicon-link" /></a>activate()

The `activate()` method is used to specify which worksheet is initially
visible in a multi-sheet workbook:

    worksheet1 = workbook.add_worksheet('To')
    worksheet2 = workbook.add_worksheet('the')
    worksheet3 = workbook.add_worksheet('wind')

    worksheet3.activate

This is similar to the Excel VBA activate method.
More than one worksheet can be selected via the `select()` method, see below,
however only one worksheet can be active.

The default active worksheet is the first worksheet.

#### <a name="select" class="anchor" href="#select"><span class="octicon octicon-link" /></a>select()

The `select()` method is used to indicate that a worksheet is selected
in a multi-sheet workbook:

    worksheet1.activate
    worksheet2.select
    worksheet3.select

A selected worksheet has its tab highlighted.
Selecting worksheets is a way of grouping them together so that, for example,
several worksheets could be printed in one go.
A worksheet that has been activated via the `activate()` method will also appear as selected.

#### <a name="hide" class="anchor" href="#hide"><span class="octicon octicon-link" /></a>hide()

The `hide()` method is used to hide a worksheet:

    worksheet2.hide

You may wish to hide a worksheet in order to avoid confusing a user with
intermediate data or calculations.

A hidden worksheet can not be activated or selected so this method is mutually
exclusive with the `activate()` and `select()` methods.
In addition, since the first worksheet will default to being the active worksheet,
you cannot hide the first worksheet without activating another sheet:

    worksheet2.activate
    worksheet1.hide

#### <a name="set_first_sheet" class="anchor" href="#set_first_sheet"><span class="octicon octicon-link" /></a>set_first_sheet()

The `activate()` method determines which worksheet is initially selected.
However, if there are a large number of worksheets the selected worksheet
may not appear on the screen. To avoid this you can select which is the leftmost
visible worksheet using `set_first_sheet()`:

    20.times { workbook.add_worksheet }

    worksheet21 = workbook.add_worksheet
    worksheet22 = workbook.add_worksheet

    worksheet21.set_first_sheet
    worksheet22.activate

This method is not required very often. The default value is the first worksheet.

#### <a name="protect" class="anchor" href="#protect"><span class="octicon octicon-link" /></a>protect(password, options)

The `protect()` method is used to protect a worksheet from modification:

    worksheet.protect

The `protect()` method also has the effect of enabling a cell's locked and
hidden properties if they have been set.
A locked cell cannot be edited and this property is on by default for all cells.
A hidden cell will display the results of a formula but not the formula itself.

See the `protection.rb` program in the examples directory of the distro for an
illustrative example and the `set_locked` and `set_hidden` format methods
in ["CELL FORMATTING"](cell_formatting.html).

You can optionally add a password to the worksheet protection:

    worksheet.protect('drowssap')

Passing the empty string '' is the same as turning on protection without a password.

Note, the worksheet level password in Excel provides very weak protection.
It does not encrypt your data and is very easy to deactivate.
Full workbook encryption is not supported by WriteXLSX since it requires a completely
different file format and would take several man months to implement.

You can specify which worksheet elements you wish to protect by passing a hash
with any or all of the following keys:

    # Default shown.
    options = {
        :objects               => 0,
        :scenarios             => 0,
        :format_cells          => 0,
        :format_columns        => 0,
        :format_rows           => 0,
        :insert_columns        => 0,
        :insert_rows           => 0,
        :insert_hyperlinks     => 0,
        :delete_columns        => 0,
        :delete_rows           => 0,
        :select_locked_cells   => 1,
        :sort                  => 0,
        :autofilter            => 0,
        :pivot_tables          => 0,
        :select_unlocked_cells => 1
    }

The default boolean values are shown above. Individual elements can be protected as follows:

    worksheet.protect('drowssap', :insert_rows => 1)

#### <a name="set_selection" class="anchor" href="#set_selection"><span class="octicon octicon-link" /></a>set_selection(first_row, first_col, last_row, last_col)

This method can be used to specify which cell or cells are selected in a worksheet.
The most common requirement is to select a single cell, in which case
`last_row` and `last_col` can be omitted. The active cell within a selected
range is determined by the order in which `first` and `last` are specified.
It is also possible to specify a cell or a range using A1 notation.
See the note about ["Cell notation"](#cell-notation).

Examples:

    worksheet1.set_selection(3, 3)          # 1. Cell D4.
    worksheet2.set_selection(3, 3, 6, 6)    # 2. Cells D4 to G7.
    worksheet3.set_selection(6, 6, 3, 3)    # 3. Cells G7 to D4.
    worksheet4.set_selection('D4')          # Same as 1.
    worksheet5.set_selection('D4:G7')       # Same as 2.
    worksheet6.set_selection('G7:D4')       # Same as 3.

The default cell selections is (0, 0), 'A1'.

#### <a name="set_row" class="anchor" href="#set_row"><span class="octicon octicon-link" /></a>set_row(row, height, format, hidden, level, collapsed)

This method can be used to change the default properties of a row.
All parameters apart from `row` are optional.

The most common use for this method is to change the height of a row:

    worksheet.set_row(0, 20)    # Row 1 height set to 20

If you wish to set the format without changing the height you can pass undef as the height parameter:

    worksheet.set_row(0, nil, format)

The format parameter will be applied to any cells in the row
that don't have a format. For example

    worksheet.set_row(0, nil, format1 )     # Set the format for row 1
    worksheet.write('A1', 'Hello')          # Defaults to format1
    worksheet.write('B1', 'Hello', format2) # Keeps format2

If you wish to define a row format in this way you should call the method
before any calls to `write()`. Calling it afterwards will overwrite any
format that was previously specified.

The `hidden` parameter should be set to 1 if you wish to hide a row.
This can be used, for example, to hide intermediary steps in a complicated
calculation:

    worksheet.set_row(0, 20,  format, 1)
    worksheet.set_row(1, nil, nil,    1)

The `level` parameter is used to set the outline level of the row.
Outlines are described in "OUTLINES AND GROUPING IN EXCEL".
Adjacent rows with the same outline level are grouped together into a single outline.

The following example sets an outline level of 1 for rows 1 and 2 (zero-indexed):

    worksheet.set_row(1, nil, nil, 0, 1)
    worksheet.set_row(2, nil, nil, 0, 1)

The `hidden` parameter can also be used to hide collapsed outlined rows when
used in conjunction with the $level parameter.

    worksheet.set_row(1, nil, nil, 1, 1)
    worksheet.set_row(2, nil, nil, 1, 1)

For collapsed outlines you should also indicate which row has the
collapsed + symbol using the optional `collapsed` parameter.

    worksheet.set_row(3, nil, nil, 0, 0, 1)

For a more complete example see the `outline.rb` and `outline_collapsed.rb`
programs in the examples directory of the distro.

Excel allows up to 7 outline levels.
Therefore the $level parameter should be in the range 0 <= $level <= 7.

#### <a name="set_column" class="anchor" href="#set_column"><span class="octicon octicon-link" /></a>set_column(first_col, last_col, width, format, hidden, level, collapsed)

This method can be used to change the default properties of a single column or
a range of columns. All parameters apart from $first_col and $last_col are optional.

If `set_column()` is applied to a single column the value of `first_col` and `last_col`
should be the same. In the case where `last_col` is zero it is set to the same
value as `first_col`.

It is also possible, and generally clearer, to specify a column range using
the form of A1 notation used for columns.
See the note about ["Cell notation"](#cell-notation).

Examples:

    worksheet.set_column(0, 0, 20)    # Column  A   width set to 20
    worksheet.set_column(1, 3, 30)    # Columns B-D width set to 30
    worksheet.set_column('E:E', 20)   # Column  E   width set to 20
    worksheet.set_column('F:H', 30)   # Columns F-H width set to 30

The width corresponds to the column width value that is specified in Excel.
It is approximately equal to the length of a string in the default font of Calibri 11.
Unfortunately, there is no way to specify "AutoFit" for a column in the Excel file format.
This feature is only available at runtime from within Excel.

As usual the format parameter is optional, for additional information,
see ["CELL FORMATTING"](#cell-formatting).
If you wish to set the format without changing the width you can pass nil
as the width parameter:

    worksheet.set_column(0, 0, nil, format)

The `format` parameter will be applied to any cells in the column that don't
have a format. For example

    worksheet.set_column('A:A', nil, format1)    # Set format for col 1
    worksheet.write('A1', 'Hello')               # Defaults to format1
    worksheet.write('A2', 'Hello', format2)      # Keeps format2

If you wish to define a column format in this way you should call the method
before any calls to `write()`. If you call it afterwards it won't have any effect.

A default row format takes precedence over a default column format

    worksheet.set_row(0, nil, format1)           # Set format for row 1
    worksheet.set_column('A:A', nil, format2)    # Set format for col 1
    worksheet.write('A1', 'Hello')               # Defaults to format1
    worksheet.write('A2', 'Hello')               # Defaults to format2

The `hidden` parameter should be set to 1 if you wish to hide a column.
This can be used, for example, to hide intermediary steps in a complicated calculation:

    worksheet.set_column('D:D', 20,  format, 1)
    worksheet.set_column('E:E', nil, nil,    1)

The `level` parameter is used to set the outline level of the column.
Outlines are described in "OUTLINES AND GROUPING IN EXCEL".
Adjacent columns with the same outline level are grouped together into a single outline.

The following example sets an outline level of 1 for columns B to G:

    worksheet.set_column('B:G', nil, nil, 0, 1)

The `hidden` parameter can also be used to hide collapsed outlined columns
when used in conjunction with the `level` parameter.

    worksheet.set_column('B:G', nil, nil, 1, 1)

For collapsed outlines you should also indicate which row has the
collapsed + symbol using the optional `collapsed` parameter.

    worksheet.set_column('H:H', nil, nil, 0, 0, 1)

For a more complete example see the `outline.rb` and `outline_collapsed.rb`
programs in the examples directory of the distro.

Excel allows up to 7 outline levels.
Therefore the $level parameter should be in the range 0 <= $level <= 7.

#### <a name="set_default_row" class="anchor" href="#set_default_row"><span class="octicon octicon-link" /></a>set_default_row(height, hide_unused_rows)

The `set_default_row()` method is used to set the limited number of default
row properties allowed by Excel.
These are the default height and the option to hide unused rows.

    worksheet.set_default_row(24)  # Set the default row height to 24.

The option to hide unused rows is used by Excel as an optimisation so that
the user can hide a large number of rows without generating a very large file
with an entry for each hidden row.

    worksheet.set_default_row(nil, 1)

See the `hide_row_col.rb` example program.

#### <a name="outline_settings" class="anchor" href="#outline_settings"><span class="octicon octicon-link" /></a>outline_settings(visible, symbols_below, symbols_right, auto_style)

The `outline_settings()` method is used to control the appearance of outlines in Excel.
Outlines are described in "OUTLINES AND GROUPING IN EXCEL".

The `visible` parameter is used to control whether or not outlines are visible.
Setting this parameter to 0 will cause all outlines on the worksheet to be hidden.
They can be unhidden in Excel by means of the "Show Outline Symbols" command button.
The default setting is 1 for visible outlines.

    worksheet.outline_settings(0)

The `symbols_below` parameter is used to control whether the row outline symbol
will appear above or below the outline level bar. The default setting is 1 for
symbols to appear below the outline level bar.

The `symbols_right` parameter is used to control whether the column outline
symbol will appear to the left or the right of the outline level bar.
The default setting is 1 for symbols to appear to the right of the outline level bar.

The `auto_style` parameter is used to control whether the automatic outline
generator in Excel uses automatic styles when creating an outline.
This has no effect on a file generated by WriteXLSX but it does have an effect
on how the worksheet behaves after it is created.
The default setting is 0 for "Automatic Styles" to be turned off.

The default settings for all of these parameters correspond to Excel's default parameters.

The worksheet parameters controlled by `outline_settings()` are rarely used.

#### <a name="freeze_panes" class="anchor" href="#freeze_panes"><span class="octicon octicon-link" /></a>freeze_panes(row, col, top_row, left_col)

This method can be used to divide a worksheet into horizontal or vertical
regions known as panes and to also "freeze" these panes so that
the splitter bars are not visible.
This is the same as the Window->Freeze Panes menu command in Excel

The parameters `row` and `col` are used to specify the location of the split.
It should be noted that the split is specified at the top or left of a cell
and that the method uses zero based indexing.
Therefore to freeze the first row of a worksheet it is necessary to specify
the split at row 2 (which is 1 as the zero-based index).
This might lead you to think that you are using a 1 based index
but this is not the case.

You can set one of the `row` and `col` parameters as zero if you do not want
either a vertical or horizontal split.

Examples:

    worksheet.freeze_panes(1, 0)    # Freeze the first row
    worksheet.freeze_panes('A2')    # Same using A1 notation
    worksheet.freeze_panes(0, 1)    # Freeze the first column
    worksheet.freeze_panes('B1')    # Same using A1 notation
    worksheet.freeze_panes(1, 2)    # Freeze first row and first 2 columns
    worksheet.freeze_panes('C2')    # Same using A1 notation

The parameters `top_row` and `left_col` are optional.
They are used to specify the top-most or left-most visible row or column in the
scrolling region of the panes. For example to freeze the first row and to have
the scrolling region begin at row twenty:

    worksheet.freeze_panes(1, 0, 20, 0)

You cannot use A1 notation for the $top_row and `left_col` parameters.

See also the `panes.rb` program in the examples directory of the distribution.

#### <a name="split_panes" class="anchor" href="#split_panes"><span class="octicon octicon-link" /></a>split_panes(y, x, top_row, left_col)

This method can be used to divide a worksheet into horizontal or vertical
regions known as panes. This method is different from the `freeze_panes()` method
in that the splits between the panes will be visible to the user
and each pane will have its own scroll bars.

The parameters `y` and `x` are used to specify the vertical and horizontal
position of the split. The units for `y` and `x` are the same as those used
by Excel to specify row height and column width. However, the vertical and
horizontal units are different from each other. Therefore you must specify
the `y` and `x` parameters in terms of the row heights and column widths that
you have set or the default values which are 15 for a row and 8.43 for a column.

You can set one of the `y` and `x` parameters as zero if you do not want either
a vertical or horizontal split. The parameters `top_row` and `left_col` are
optional. They are used to specify the top-most or left-most visible row or
column in the bottom-right pane.

Example:

    worksheet.split_panes(15, 0,  )    # First row
    worksheet.split_panes(0,  8.43)    # First column
    worksheet.split_panes(15, 8.43)    # First row and column

You cannot use A1 notation with this method.

See also the `freeze_panes()` method and the `panes.rb` program in the examples
directory of the distribution.

#### <a name="merge_range" class="anchor" href="#merge_range"><span class="octicon octicon-link" /></a>merge_range(first_row, first_col, last_row, last_col, token, format)

The `merge_range()` method allows you to merge cells that contain other types
of alignment in addition to the merging:

    format = workbook.add_format(
        :border => 6,
        :valign => 'vcenter',
        :align  => 'center'
    )

    worksheet.merge_range('B3:D4', 'Vertical and horizontal', format)

`merge_range()` writes its $token argument using the worksheet `write()` method.
Therefore it will handle numbers, strings, formulas or urls as required.
If you need to specify the required `write_\*()` method use the
`merge_range_type()` method, see below.

The full possibilities of this method are shown in the `merge3.rb` to
`merge6.rb` programs in the examples directory of the distribution.

#### <a name="merge_range_type" class="anchor" href="#merge_range_type"><span class="octicon octicon-link" /></a>merge_range_type(type, first_row, first_col, last_row, last_col, ... )

The `merge_range()` method, see above, uses `write()` to insert the required
data into to a merged range. However, there may be times where this isn't
what you require so as an alternative the `merge_range_type()` method allows
you to specify the type of data you wish to write. For example:

    worksheet.merge_range_type('number',  'B2:C2', 123,    format1)
    worksheet.merge_range_type('string',  'B4:C4', 'foo',  format2)
    worksheet.merge_range_type('formula', 'B6:C6', '=1+2', format3)

The `type` must be one of the following, which corresponds to a `write_\*()` method:

    'number'
    'string'
    'formula'
    'array_formula'
    'blank'
    'rich_string'
    'date_time'
    'url'

Any arguments after the range should be whatever the appropriate method accepts:

    worksheet.merge_range_type('rich_string', 'B8:C8',
                                  'This is ', bold, 'bold', format4)

Note, you must always pass a format object as an argument, even if it is a default format.

#### <a name="set_zoom" class="anchor" href="#set_zoom"><span class="octicon octicon-link" /></a>set_zoom(scale)

Set the worksheet zoom factor in the range 10 <= $scale <= 400:

    worksheet1.set_zoom(50)
    worksheet2.set_zoom(75)
    worksheet3.set_zoom(300)
    worksheet4.set_zoom(400)

The default zoom factor is 100.
You cannot zoom to "Selection" because it is calculated by Excel at run-time.

Note, `set_zoom()` does not affect the scale of the printed page.
For that you should use `set_print_scale()`.

#### <a name="right_to_left" class="anchor" href="#right_to_left"><span class="octicon octicon-link" /></a>right_to_left()

The `right_to_left()` method is used to change the default direction of the
worksheet from left-to-right, with the A1 cell in the top left, to
right-to-left, with the A1 cell in the top right.

    worksheet.right_to_left

This is useful when creating Arabic, Hebrew or other near or far eastern
worksheets that use right-to-left as the default direction.

#### <a name="hide_zero" class="anchor" href="#hide_zero"><span class="octicon octicon-link" /></a>hide_zero()

The `hide_zero()` method is used to hide any zero values that appear in cells.

    worksheet.hide_zero

In Excel this option is found under Tools->Options->View.

#### <a name="set_tab_color" class="anchor" href="#set_tab_color"><span class="octicon octicon-link" /></a>set_tab_color()

The `set_tab_color()` method is used to change the colour of the worksheet tab.
You can use one of the standard colour names provided by the Format object
or a colour index.
See "COLOURS IN EXCEL" and the `set_custom_color()` method.

    worksheet1.set_tab_color('red')
    worksheet2.set_tab_color(0x0C)

See the `tab_colors.rb` program in the examples directory of the distro.

#### <a name="autofilter" class="anchor" href="#autofilter"><span class="octicon octicon-link" /></a>autofilter(first_row, first_col, last_row, last_col)

This method allows an autofilter to be added to a worksheet.
An autofilter is a way of adding drop down lists to the headers of a 2D range
of worksheet data.
This allows users to filter the data based on simple criteria so that
some data is shown and some is hidden.

To add an autofilter to a worksheet:

    worksheet.autofilter(0, 0, 10, 3)
    worksheet.autofilter('A1:D11')    # Same as above in A1 notation.

Filter conditions can be applied using the `filter_column()` or
`filter_column_list()` method.

See the `autofilter.rb` program in the examples directory of the distro
for a more detailed example.

#### <a name="filter_column" class="anchor" href="#filter_column"><span class="octicon octicon-link" /></a>filter_column(column, expression)

The `filter_column` method can be used to filter columns in a autofilter range
based on simple conditions.

NOTE: It isn't sufficient to just specify the filter condition.
You must also hide any rows that don't match the filter condition.
Rows are hidden using the `set_row()` visible parameter.
WriteXLSX cannot do this automatically since it isn't part of the file format.
See the `autofilter.rb` program in the examples directory of the distro for an example.

The conditions for the filter are specified using simple expressions:

    worksheet.filter_column('A', 'x > 2000')
    worksheet.filter_column('B', 'x > 2000 and x < 5000')

The `column` parameter can either be a zero indexed column number
or a string column name.

The following operators are available:

    Operator        Synonyms
       ==           =   eq  =~
       !=           <>  ne  !=
       >
       <
       >=
       <=

       and          &&
       or           ||

The operator synonyms are just syntactic sugar to make you more comfortable using
the expressions. It is important to remember that the expressions will be
interpreted by Excel and not by ruby.

An expression can comprise a single statement or two statements separated
by the `and` and `or` operators. For example:

    'x <  2000'
    'x >  2000'
    'x == 2000'
    'x >  2000 and x <  5000'
    'x == 2000 or  x == 5000'

Filtering of blank or non-blank data can be achieved by using a value of Blanks
or NonBlanks in the expression:

    'x == Blanks'
    'x == NonBlanks'

Excel also allows some simple string matching operations:

    'x =~ b*'   # begins with b
    'x !~ b*'   # doesn't begin with b
    'x =~ *b'   # ends with b
    'x !~ *b'   # doesn't end with b
    'x =~ *b*'  # contains b
    'x !~ *b*'  # doesn't contains b

You can also use `*` to match any character or number and ? to match any single
character or number. No other regular expression quantifier is supported by
Excel's filters. Excel's regular expression characters can be escaped using ~.

The placeholder variable x in the above examples can be replaced by any simple
string. The actual placeholder name is ignored internally so the following are
all equivalent:

    'x     < 2000'
    'col   < 2000'
    'Price < 2000'

Also, note that a filter condition can only be applied to a column in a range
specified by the `autofilter()` Worksheet method.

See the `autofilter.rb` program in the examples directory of the distro for
a more detailed example.

Note writeexcel gem supports Top 10 style filters.
These aren't currently supported by WriteXLSX but may be added later.

#### <a name="filter_column_list" class="anchor" href="#filter_column_list"><span class="octicon octicon-link" /></a>filter_column_list(column, matches)

Prior to Excel 2007 it was only possible to have either 1 or 2 filter conditions
such as the ones shown above in the filter_column method.

Excel 2007 introduced a new list style filter where it is possible to specify
1 or more 'or' style criteria.
For example if your column contained data for the first six months the initial
data would be displayed as all selected as shown on the left.
Then if you selected 'March', 'April' and 'May' they would be displayed as
shown on the right.

    No criteria selected      Some criteria selected.

    [/] (Select all)          [X] (Select all)
    [/] January               [ ] January
    [/] February              [ ] February
    [/] March                 [/] March
    [/] April                 [/] April
    [/] May                   [/] May
    [/] June                  [ ] June

The `filter_column_list()` method can be used to represent these types of filters:

    worksheet.filter_column_list('A', 'March', 'April', 'May')

The `column` parameter can either be a zero indexed column number or a string column name.

One or more criteria can be selected:

    worksheet.filter_column_list(0, 'March')
    worksheet.filter_column_list(1, 100, 110, 120, 130)

NOTE: It isn't sufficient to just specify the filter condition.
You must also hide any rows that don't match the filter condition.
Rows are hidden using the `set_row()` visible parameter.
WriteXLSX cannot do this automatically since it isn't part of the file format.
See the `autofilter.rb` program in the examples directory of the distro for an example.

#### <a name="convert_date_time" class="anchor" href="#convert_date_time"><span class="octicon octicon-link" /></a>convert_date_time(date_string)

The `convert_date_time()` method is used internally by the `write_date_time()`
method to convert date strings to a number that represents an Excel date and time.

It is exposed as a public method for utility purposes.

The `date_string` format is detailed in the `write_date_time()` method.