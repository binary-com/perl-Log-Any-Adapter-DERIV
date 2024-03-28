# NAME

Log::Any::Adapter::DERIV - standardised logging to STDERR and JSON file

[![Test status](https://circleci.com/gh/binary-com/perl-Log-Any-Adapter-DERIV.svg?style=shield&circle-token=bed2af8f8e388746eafbbf905cf6990f84dbd69e)](https://app.circleci.com/pipelines/github/binary-com/perl-Log-Any-Adapter-DERIV)

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
their own version or make feature/bug fix suggestions if they seem generally useful:

[https://github.com/binary-com/perl-Log-Any-Adapter-DERIV](https://github.com/binary-com/perl-Log-Any-Adapter-DERIV)

## PARAMETERS

- json\_log\_file

    Specify a file name to which you want the json formatted logs printed into.
    If not given, then it prints the logs to STDERR.

- STDERR

    If it is true, then print logs to STDERR

    If the value is json or text, then print logs with that format

    If the value is just a true value other than \`json\` or \`text\`,
    then if it is running in a container, then it prints the logs in \`json\` format.
    Else if STDERR is a tty, then it prints \`colored text\` format.
    Else it prints non-color text format.

If no parameters provided, then default \`stderr => 1\`;

# METHODS

## apply\_filehandle\_utf8

Applies UTF-8 to filehandle if it is not utf-flavoured already

    $object->apply_filehandle_utf8($fh);

- `$fh` file handle

## format\_line

Formatting the log entry with timestamp, from which the message populated,
severity and message.

If color/colour param passed it adds appropriate color code for timestamp,
log level, from which this log message populated and actual message.
For non-color mode, it just returns the formatted message.

    $object->format_line($data, {color => $color});

- `$data` hashref - The data with stack info like package method from
which the message populated, timestamp, severity and message
- `$opts` hashref - the options color

Returns only formatted string if non-color mode. Otherwise returns formatted
string with embedded ANSI color code using [Term::ANSIColor](https://metacpan.org/pod/Term%3A%3AANSIColor)

## log\_entry

Add format and add color code using `format_line` and writes the log entry

    $object->log_entry($data);

- \*`$data` hashref - The log data

## \_process\_data

Process the data before printing out. Reduce the continues [Future](https://metacpan.org/pod/Future) stack
messages and filter the messages based on log level.

    $object->_process_data($data);

- `$data` hashref - The log data.

Returns a hashref - the processed data

## \_filter\_stack

Filter the stack message based on log level.

    $object->_filter_stack($data);

- `$data` hashref - Log stack data

Returns hashref - the filtered data

## \_collapse\_future\_stack

Go through the caller stack and if continuous [Future](https://metacpan.org/pod/Future) messages then keep
only one at the first.

    $object->_collapse_future_stack($data);

- `$data` hashref - Log stack data

Returns a hashref - the reduced log data

## \_fh\_is\_tty

Check the filehandle opened to tty

- `$fh` file handle

Returns boolean

## \_in\_container

Returns true if we think we are currently running in a container.

At the moment this only looks for a `.dockerenv` file in the root directory;
future versions may expand this to provide a more accurate check covering
other container systems such as \`runc\`.

Returns boolean

## \_linux\_flock\_data

Based on the type of lock requested, it packs into linux binary flock structure
and return the string of that structure.

Linux struct flock: "s s l l i"
	short l\_type short - Possible values: F\_RDLCK(0) - read lock, F\_WRLCK(1) - write lock, F\_UNLCK(2) - unlock
	short l\_whence - starting offset
	off\_t l\_start - relative offset
	off\_t l\_len - number of consecutive bytes to lock
	pid\_t l\_pid - process ID

- `$type` integer lock type - F\_WRLCK or F\_UNLCK

Returns a string of the linux flock structure

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

Return the current log level name.

## \_process\_context

add context key value pair into data object

## set\_context

Set the log context hash

## clear\_context

undef the log context hash

## mask\_sensitive

Mask sensitive data in the message and logs error in case of failure

- `$message` string - The message to be masked

Returns string - The masked message

# AUTHOR

Deriv Group Services Ltd. `DERIV@cpan.org`

# LICENSE

Copyright Deriv Group Services Ltd 2020-2021. Licensed under the same terms as Perl itself.
