# NAME

Log::Any::Adapter::DERIV - standardised logging to STDERR and JSON file

# SYNOPSIS

    use Log::Any;

    # print text log to STDERR, json format when inside docker container,
    # colored text format when STDERR is a tty, non-colored text format when
    # STDERR is redirected.
    use Log::Any::Adapter ('DERIV');

    #specify STDERR directly
    use Log::Any::Adapter ('DERIV', stderr => 1)

    #specify STDERR's format
    use Log::Any::Adapter ('DERIV', stderr => 'json')

    #specify the json log name
    use Log::Any::Adapter ('DERIV', json_log_file => '/var/log/program.json.log');

# DESCRIPTION

Applies some opinionated log handling rules for [Log::Any](https://metacpan.org/pod/Log%3A%3AAny).

**This is extremely invasive**. It does the following, affecting global state
in various ways:

- applies UTF-8 encoding to STDERR
- writes to a `.json.log` file.
- overrides the default [Log::Any::Proxy](https://metacpan.org/pod/Log%3A%3AAny%3A%3AProxy) formatter to provide data as JSON
- when stringifying, may replace some problematic objects with simplified versions

An example of the string-replacement approach would be the event loop in asynchronous code:
it's likely to have many components attached to it, and dumping that would effectively end up
dumping the entire tree of useful objects in the process. This is a planned future extension,
not currently implemented.

## Why

This is provided as a CPAN module as an example for dealing with multiple outputs and formatting.
The existing [Log::Any::Adapter](https://metacpan.org/pod/Log%3A%3AAny%3A%3AAdapter) modules tend to cover one thing, and it's
not immediately obvious how to extend formatting, or send data to multiple logging mechanisms at once.

Although the module may not be directly useful, it is hoped that other teams may find
parts of the code useful for their own logging requirements.

There is a public repository on Github, anyone is welcome to fork that and implement
their own version or make feature/bugfix suggestions if they seem generally useful:

[https://github.com/binary-com/perl-Log-Any-Adapter-DERIV](https://github.com/binary-com/perl-Log-Any-Adapter-DERIV)

## PARAMETERS

- json\_log\_file

    Specify a file name that the json format log file will be printed into.
    If not given, then a default file 'program\_name.json.log' will be used.

- STDERR

    If it is true, then print logs to STDERR

    If the value is json or text, then print logs with that format

    If the value is just a true value other than \`json\` or \`text\`,
    then if it is running in a container, then the logs is \`json\` format.
    Else if STDERR is a tty will be \`colored text\` format.
    Else if will be a non-color text format.

If no any parameter, then default \`stderr => 1\`;

## apply\_filehandle\_utf8

- `$fh` file handle

## format\_line

Format the log entry

- `$data` log data -severity, message, stack
- `$opts` options - color

Returns the color details for log

## log\_entry

Writes the log entry

- \*`$data` log data

## \_process\_data

Process the data before printing out.

Takes the following arguments as named parameters:

- `$self`
- `data`

    The log data.

Returns processed data

## \_filter\_stack

In some cases we don't want to print stack info. This function is used to filter out the stack info.

Takes the following arguments as named parameters:

- `$self`
- `data`

    The log data.

Returns processed data

## \_collapse\_future\_stack

The future [FUTURE](https://metacpan.org/pod/FUTURE) frames are too much and too tedious. This method will keep only one
frame if there are many continuously future frames.

Takes the following arguments as named parameters

- `$data` Log stack data

Returns log data

## \_fh\_is\_tty

Check the filehandle opened to tty
Returns boolean

## \_in\_container

Checks in the container
Returns boolean

## \_linux\_flock\_data

- `$type` lock type - F\_WRLCK or F\_UNLCK

Returns a FLOCK structure

## \_flock

call fcntl to lock or unlock a file handle

- `$fh` file handle
- `$type` lock type, either F\_WRLCK or F\_UNLCK

Returns boolean or undef

## \_lock

Lock a file handler with fcntl.

- `$fh` File handle

Returns boolean

## \_unlock

Unlock a file handler locked by fcntl

- `$fh` File handle

Returns boolean

## level

return the current log level name

# AUTHOR

Deriv Group Services Ltd. `DERIV@cpan.org`

# LICENSE

Copyright Deriv Group Services Ltd 2020-2021. Licensed under the same terms as Perl itself.
