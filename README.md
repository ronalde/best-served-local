# best-served-local

> aka "Yet Another Google Web Font Downloader written in bash"

## Rationale

Inspired by [google-font-download] by Clemens Lang (also written in
bash v4) and [google-webfonts-helper] (client side javascript) I
developed this script to make it easy to [automate](#automated) (via
cron and such) the downloading of remote web fonts and the creation of
high quality [@font-face css3 rules] using the
[bulletproof @font-face method] by Paul Irish for serving from my own
webserver.

Apart from [bash] version 4, this script only depends on [curl]. 
It's not tested on OSX yet.


## Simple usage example

The command:
```bash
./best-served-local "Open Sans" "Roboto:light,black"
```

Will display the following `@font-face` css at-rules:

```css
/* >>> script generated css starts here >>> */
@font-face {
   font-family: 'Open Sans';
   src: 
        local('Open Sans'), local('OpenSans'), 
		url('Open_Sans_v13_latin_400.woff2') format('woff2'),
		url('Open_Sans_v13_latin_400.woff') format('woff');
		font-style:  normal;
		font-weight: 400;
}
@font-face {
 	font-family: 'Roboto';
	src: 
	     local('Roboto Light'), local('Roboto-Light'), 
	     url('Roboto_Light_v15_latin_300.woff2') format('woff2'),
	     url('Roboto_Light_v15_latin_300.woff') format('woff');
	     font-style:  normal;
	     font-weight: 300;
}
@font-face {
	font-family: 'Roboto';
	src: 
	     local('Roboto Black'), local('Roboto-Black'), 
	     url('Roboto_Black_v15_latin_900.woff2') format('woff2'),
	     url('Roboto_Black_v15_latin_900.woff') format('woff');
	font-style:  normal;
	font-weight: 900;
}
/* <<< script generated css ends here <<< */
```

And in the temporary directory, the following files are downloaded,
ready to be passed over to your webserver:

```
/tmp/best-served-local.XXXX
├── Open_Sans_v13_latin_400.woff
├── Open_Sans_v13_latin_400.woff2
├── Roboto_Black_v15_latin_900.woff
├── Roboto_Black_v15_latin_900.woff2
├── Roboto_Light_v15_latin_300.woff
└── Roboto_Light_v15_latin_300.woff2
```


## Getting and running the script

The script can be cloned or forked from its [github repository],
[downloaded], or started straight from the web (although some would
advise against that):

```bash
bash <(wget -q -O - "http://lacocina.nl/best-served-local") "Roboto:100,900"
```

Or, when you prefer `curl`:
```bash
bash <(curl -sL "http://lacocina.nl/best-served-local") -f all Slabo\ 27px
```

To display all [commandline arguments], run it with `--help` (or `-h`)
argument. It will display something like:

```bash
best-served-local [-o|--outputfile PATH] [-d|--fontdirectory PATH] \
      [-i|--incss-fontpath PATH] \
      [-s|--subsets SUBSETSPEC] [-f|--formats FORMATSPEC] \
      [-n|--skip-downloads] [-x|--skip-local] \
	  [--overwrite-cssfile] [--overwrite-fonts] \
      [-h|--help] \
      FONTSPEC
```

The only required argument is [FONTSPEC]; all other arguments are
[optional].


## Features

* It uses the version of the font in the local file name and css
  `url()` references.
* It never overwrites existing css or font files (unless the
  `--overwrite-...` arguments are used);
  * When an existing css file is present, the script will notify the
    user and print the result to `stdout`.
* It stores downloaded web fonts in a temporary directory, which
  path is displayed when the script finishes (unless the
  `--fontdirectory` argument together with a writable path is
  specified).
* Warnings and messages are redirected to `stderr` so it should be
  save to redirect the output using a pipe or redirection,
  eg. `./best-served-local ... > myfile.css`.


## Fully automated usage example

Using (valid and tested) values for the commandline arguments
`--incss-fontpath`, `--outputfile`, `--fontdirectory` and using
`--overwrite-fonts` and `--overwrite-cssfile`, the script can be used
to get a fully automated powertool for your webserver:

```bash

./best-served-local --incss-fontpath /static/fonts \
--outputfile /var/www/example.org/static/css/fontdefs.css \
--fontdirectory /var/www/example.org/static/fonts/
--overwrite-fonts \
--overwrite-cssfile \
--formats superprogressive \
--subsets latin-ext \
"Open Sans:300,400,700" "Roboto:100,100italic,regular,italic,900"

```

## Some extra features

Setting a value for the `--incss-fontpath` (or `-i`) argument will
cause the resulting `url()` values in the `src` attribute to use that
as the path to the font file, eg using `-i ../static/fonts` will
result in the following css:

```css
src: url('../static/fonts/A_Web_Font_v1_latin.woff2')
```

While omitting that option would result in:
```css
src: url('A_Web_Font_v1_latin.woff2')
```

Using the `--skip-local` (or `-x`) argument will make the script skip
the `local` references in the `src` attributes, which comes in handy
if you want maximum control over the appearance of your website.

Using `--skip-downloads` (or `-n`) will not download the font files,
but just output the proper CSS. It will however contact Goggle-server
to verify the proper css attributes and such.

## Commandline arguments reference

### Required argument

FONTSPEC
: A `FONTSPEC` is a space separated list of family names, surrounded
  with quotes or space-escaped, with an optional suffix, consisting of
  `:`, followed by a comma separated list of font weight/style values.

  For example, to use *"Open Sans"* in the regular font weight and style
  use:

```bash
./best-served-local "Open Sans"
```

  To get the italic variant with the same weight, next to the regular
  one, specify both, eg:

```bash
./best-served-local "Open Sans:regular:italic"
## or
./best-served-local "Open Sans:400:italic"
```

  Multiple `FONTSPEC`s can be set as follows:

```bash
./best-served-local "Open Sans:regular:italic" "Web Font A:extrabold,superlight"
```

### Optional arguments

### FORMATSPECS

`FORMATSPECS` define the font file formats to use. It should be
specified as one of the *presets*, or a comma-seperated list of
specific formats.

The script recognize the following presets, taken from
https://css-tricks.com/snippets/css/using-font-face/:

superprogressive
: `woff2`

practical
: *superprogressive* + `woff` (= default)

slightlydeeper
: *practical* + `ttf`

all
: *slightlydeeper* + `eot` + `otf` + `svg`

For example, to use the default *practical* set, consisting of the
`woff` and `woff2` formats, for the "Open Sans" font, use:

```bash
./best-served-local "Open Sans" 
```

To get full browser support (and thus heavy per page downloads), use:
```bash
./best-served-local -f all "Open Sans" 
```

You want something funky? Just tell the script to do so:
```bash
./best-served-local -f eot,svg "Open Sans" 
```

---------------

[google-font-download]: 
  https://github.com/neverpanic/google-font-download.git/ "`google-font-download`"

[google-webfonts-helper]: 
  https://github.com/majodev/google-webfonts-helper/ "`google-webfonts-helper`"

[@font-face css3 rules]: 
  https://www.w3.org/TR/css-fonts-3/#font-face-rule

[bulletproof @font-face method]: 
  http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/ \
  ""bulletproof @font-face method""
  
[bash]: 
  https://www.gnu.org/software/bash/
  
[curl]:
  https://curl.haxx.se/

[github repository]: 
  https://github.com/ronalde/best-served-local

[downloaded]:
  http://lacocina.nl/best-served-local

[FONTSPEC]:
  #fontspec

[commandline arguments]:
  #commandline-arguments-reference

[optional]:
  #optional-commandline-arguments
