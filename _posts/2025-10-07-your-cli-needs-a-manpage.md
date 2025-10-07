---
layout: post
title: Your CLI tool needs a man page
author: cristian.lupascu
excerpt: >
  TODO
tags:
  - Software Development
  - Developer Experience
  - Documentation
---

You just bought your new power tool and are ready to use it. You open the box and notice that the user manual is missing,
and instead there's a pamphlet with only the minimum instructions on how to turn it on, and a link to the user manual, 
which **will not be available in 10 years when the company decides to drop support**. 

Sounds familiar? That's most CLI tools out there: The repository has a README file that explains how to install the software, its functionalities in brief and how to contribute.
If you are lucky, there's the copy-pasted raw output of `--help`. 
In bigger projects, there's links to an _actual_ user manual, hosted separately with hopes that will still be up 10 years from now.

> But everyone has internet these days, why should I have to write an offline manual?

Who, like me, works while travelling on Finnish trains knows well that internet connection is spotty at best, non-existent most of the time. 
Without internet, it wouldn't be possible to consult the README or manual website, resulting in either me not working, or blindly trying the tool until I get angry and give up.

Besides, it helps with software preservation: If you have the manual of the last-released version of a software, you can still keep using it, even if the website and repository have been taken down.

## What is a man page?

Manual pages have existed since the 1970s, to document the UNIX system, the C programming language and other internal UNIX tools. Linux and MacOS still use `man` pages.

For example, the beginning of the `man` page for `man` looks like this on MacOS:

```groff
$ man man
MAN(1)                                                                      General Commands Manual                                                                      MAN(1)

NAME
     man, apropos, whatis – display online manual documentation pages

SYNOPSIS
     man [-adho] [-t | -w] [-M manpath] [-P pager] [-S mansect] [-m arch[:machine]] [-p [eprtv]] [mansect] page ...

     man -f [-d] [-M manpath] [-P pager] [-S mansect] keyword ...
     whatis [-d] [-s mansect] keyword ...

     man -k [-d] [-M manpath] [-P pager] [-S mansect] keyword ...
     apropos [-d] [-s mansect] keyword ...

DESCRIPTION
     The man utility finds and displays online manual documentation pages.  If mansect is provided, man restricts the search to the specific section of the manual.

<...snip...>
```

### Why write a man page

There's several reason why writing a `man` page instead of a markdown README is better:

1. `man` ships with virtually every out-of-the-box release of Linux and MacOS;
2. It has been around for more than 40 years, making it rock-solid in terms of maturity and support;
3. It doesn't require any extra tools or dependencies, _it just works_;
4. All pages are formatted the same way, easing the mental load and making search and navigation predictable;
5. Manual pages are accessible offline at any time, removing the dependency on third-parties for documentation;
6. README files or documentation websites are not provided with the software unless you build it yourself.

## How to write a man page

I made a simple CLI tool called [TODO]() that extracts information from PEM or DER-encoded X.509 certificates either from `stdin` or from a file and outputs the result as a table.

### Identify the information the user must know

The first step to write an effective manual page is to know what the user **must** know about `TODO`.

For a user to correctly use `TODO`, it needs to know

1. What `TODO` does, explained concisely but clearly;
2. How to correctly call `TODO`;
3. What options and ENV variables `TODO` takes, why and what each option and ENV variable does;
4. Examples;
5. Exit codes;
6. What files it operates on (disks, mount points, configuration files, etc...);
7. References to other `man` pages, such as libraries, applications it depends on etc...;
8. License and Copyright.

Additionally, the user would benefit from knowing

1. Known bugs, limitations and questionable behaviour of the current version;
2. Caveats or pitfalls users might fall into;
3. How and to whom to report bugs or make inquiries.

### Find and write the required information

The second step is to analyse your code and answer to the aforementioned points.

I like to write a plain text draft where I first dump all the information,
and then progressively refine the contents until I have found all the answers.

While refining the draft, try to make it look like a `man` page already,
so that you get the idea of how the end result will look like.

> PROTIP: Try to create a good project plan at the beginning, so that this step becomes much easier.

For `TODO`, the draft document could look like this:

```txt
TODO reads openssl X.509 certificates in PEM and DER formats and outputs 
the information requested via the given options. TODO is called like

TODO [OPTIONS] --pem INFO,INFO,...
TODO [OPTIONS] --der INFO,INFO,...

OPTIONS and INFO
    -f, --file:         The certificate file
    -v, --verbose:      Enable verbose output
    -d, --debug:        Enable debug logs. It will pollute the terminal,
                        use only if something's wrong and want to find out why.
                        It is recommended to redirect stderr to a file.
    --der:              Indicate the input as DER-formatted
    --pem:              Indicate the input as PEM-formatted
    
    issuer:             The issuer of the input certificate
    created:            The creation date
    expiration:         The expiration date
    subject:            The subject name formatted as (TODO: add standard)
    key_usage:          The key usages, comma separated
    ext_key_usage:      The extended key usages, comma separated
    dns:                The DNS names, comma separated
    ip:                 The IP addresses, comma separated

EXIT CODES
    0   if success
    -1  if no INFO supplied
    1   if file doesn't exist
    2   if file is not of the specified format
    3   if the input is not a X.509 certificate

EXAMPLES
    Read a PEM file and get the subject, key usage and dns information:

    $ TODO --debug --file cert.pem --pem subject,key_usage,dns 2>debug.log

    Subject     CN=a5585b43a22249e58
    Key Usage   Digital Signature, Key Encipherment
    DNS         localhost, ::1

BUGS or LIMITATIONS
    If there's more than one PEM block, 
    only the first one is read and the rest of the file is ignored.

FILES
    None - can be omitted from the man page

ENV variables
    None - can be omitted from the man page

SEE MORE
    openssl(1)
    openssl-x509(1)
    cat(1)
```

### Write the man page

At this point you can start writing the actual `man` page, following the conventions found at `man-pages(7)` on Linux, consultable also online at [man-pages(7)](https://man7.org/linux/man-pages/man7/man-pages.7.html).

The end result would look like [TODO.1]().

### Ship it!

Manual pages live in `/usr/local/share/man/manX` and `/usr/share/man/manX` directories, where `X` stands for the section number. All you need is to copy the manual page file in one of the two directories (preferably `/usr/local/share/man/man1`). I like to use `make` for this:

```Makefile
make:
    cargo build --release

install:
    cp TODO /usr/bin/TODO
    cp TODO.1 /usr/local/share/man/man1
```

After building and running, you can access our manual page!

```
$ man TODO.1

...
```