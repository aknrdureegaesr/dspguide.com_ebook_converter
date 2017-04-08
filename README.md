# &ldquo;The Scientist and Engineer's Guide to Digital Signal Processing&rdquo; turned into an EPUB ebook

The book &ldquo;The Scientist and Engineer's Guide to Digital Signal
Processing&rdquo; by Steven W. Smith, Ph.D.  was recommended to me. I'm
in the process of reading it and enjoy doing so.  I recommend it
myself.

Unfortunately, that book is not open-source.  But it is available via
the website [http://www.dspguide.com/](http://www.dspguide.com/) and a
license is kindly provided that allows, free of charge:

> You may download any or all of the book in electronic form, for your
> personal use.

In my case, my preferred "personal use" is reading on my e-book reader.

The accompanying `Rakefile` contains the program that I used to
download the book in electronic form and convert it into the
ebook-reader EPUB format.

## What does my programm do?

It 

* downloads the book in electronic form from the website, including any
  graphics that are referenced (and exist on that website, see below),

* removes stuff of no value for my ebook-reader intention, such as
  navigation, Javascript and CSS-styles,

* scales all graphics (which are a bit small, see below) via CSS, and

* as its final result, creates an EPUB file `dspguide.epub`.

## Intended uses of this repository

Among other things, you can use this repository to

* do the same as I did towards your own personal use of the book,

* look into the `Rakefile` for fun, learning, or to use it a basis for
  similar endeavours of your own.

## Installation

To run this program, you first need to complete three installation
tasks.

* You need a working installation of a reasonably recent
  [Ruby](https://www.ruby-lang.org/en/).

* Into that installation, you need to integrate the [Nokogiri
  gem](http://www.nokogiri.org/tutorials/installing_nokogiri.html).

* You also need a reasonably recent version of
  [Calibre](http://calibre-ebook.com/).

For what are those needed?

* The program `rake`, that comes with the Ruby installation, and Ruby
  itself are used to download things and for overall flow control.

* Nokogiri is used to manipulate the HTML that goes into the ebook
  conversion process, and also to determine which graphics are
  included.

* Of Calibre, what is actually used is the command-line program
  `ebook-convert`.

This was developed under Linux.  I see no reason why this shouldn't
work on Mac or even Windows.

I could easily come up with a `Dockerfile`, if there is demand for it.
If you had a `Dockerfile`, you'd only need to install the one software
[Docker](https://www.docker.com/), as opposed to the three things you
need to install presently.

## How do you run this?

After you have cloned this repository and your installation is
complete, open a terminal, navigate to the directory that contains the
clone of this repository, more specifically, the `Rakefile`, and type

    rake

Assuming you have internet connectivity, this should

* run a considerable amount of time (some minutes),

* entertain you with some messages, e.g., about the 34 chapters being
  downloaded, and about (a few) images referenced from the HTML that
  do not exist on the remote server,

* when completed, leave a lot of files in the directory, among them
  the file `dspguide.epub`.

## Details you may or may not care about

### Versions used by me

I myself have been using Debian GNU Linux (Jessie) to develop this,
ruby 2.3.3 with Nokogiri 1.7.0.1 (both installed via
[rvm](https://rvm.io/)) and Calibre 2.75.1 (obtained from
[Jessie-backports](https://backports.debian.org/Instructions/)).

### The images

The book's text links to many images.  Formulas and graphs are
typically such images.  On my ebook-reader, the writings in those
images tend to be displayed with small letters on the border of
readability.  In the conversion process, I introduce specific styling
to enlarge those images to 95% of the screen width, which improved
matters somewhat.  Some images are best viewed with the reader
oriented horizontally, others vertically.

A few images referenced by the text do not exist on the remote server.
This is a bug of the remote web site.  A warning is issued for each
such image that's found missing during download.  Those missing image
references are removed before the EPUB is generated.  (I checked that my
browser has the same problem when directly viewing the web site.)

### Individual steps

You can do things step-wise.  To learn what steps are available, see
the output of `rake -T` .

### Download the PDFs that make up the book

You can also use the `Rakefile` to download all chapter PDFs.  For
this, you need neither Nokogiri nor Calibre installed, just Ruby is
enough.  (For PDF download without Nokogiri installed, simply remove
the line `require 'nokogiri'` near the top of the `Rakefile`.)  Run

    rake grab_all_pdfs

### Using a cache (doesn't help much)

I've been running this software behind a
[squid](http://www.squid-cache.org/) cache, but
[www.dspguide.com](http://www.dspguide.com/) isn't cache-friendly.
While the graphics are cach-able, all HTML files are downloaded all
over again on each run of the program.

### This is work in progress

I whipped this together in a single evening and stopped coding when I
considered the result good enough for my reading pleasure.  The time
that went into coding is comparable to the time that went into
authoring this README.  Further improvements (to both) are possible.

One obvious improvement would be to separate the HTML download and the
HTML process, into separate steps.  With that, the processing could be
changed and re-run without the (present) need to repeat the download.

My `Rakefile` is open source under the Apache license.  See the top of
the `Rakefile` for details.  It works for me, but, as is customary in
open source, I do not guarantee anything.
