# best-served-local (aka Yet Another Google Font Downloader written in bash)

## Rationale

Inspired by
[google-font-download](https://github.com/neverpanic/google-font-download.git)
(also in bash 4) and
[google-webfonts-helper](https://google-webfonts-helper.herokuapp.com/fonts)
(node-js) I developed this script to make it easy to automate (via
cron and such) the downloading of remote web fonts and the creation of
high quality
[@font-face css3 rules](https://www.w3.org/TR/css-fonts-3/#font-face-rule)<sup><a
href="#bulletproof">1</a></sup> for serving from my own webserver.

Apart from [bash](https://www.gnu.org/software/bash/) version 4, this
script only depends on [curl](https://curl.haxx.se/).  (It's not
tested on OSX yet).


## Simple usage example

The command:
```bash
./best-served-local "Open Sans:400;bolditalic;light" "Robot:light,black"
```

Will display the following `@font-face` css at-rules and download each
font file to a temporary directory.:

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

## Getting and running the script

The script can be cloned or forked from its
[github repository](https://github.com/ronalde/best-served-local),
[downloaded](http://lacocina.nl/best-served-local), or started straight
from the web (although some would advise against that):

```bash
bash <(wget -q -O - "http://lacocina.nl/best-served-local") "Roboto:100,900"
```

Or, when you prefer `curl`:
```bash
bash <(curl -sL "http://lacocina.nl/best-served-local") -f all Slabo\ 27px
```

To display all command line arguments, run it with `--help` (or `-h`)  argument:
```bash
best-served-local [-o|--outputfile PATH] [-d|--fontdirectory PATH] \
      [-i|--incss-fontpath PATH] \
      [-s|--subsets SUBSETSPEC] [-f|--formats FORMATSPEC] \
      [-n|--skip-downloads] [-x|--skip-local] \
      [-h||--help] \
      FONTSPEC
```


## Features

* It uses the version of the font in the local file name and css `url()` references
* It never overwrites existing css or font files (unless the `--overwrite-...` arguments are used)
* When an existing css file is present, the script will print the result to `stdout`
* It stores the downloaded web fonts in a temporary directory, which path is displayed when the script finishes (unless the `--fontdirectory` argument together with a writable path is specified)
* Warnings and messages are redirected to `stderr` so it should be save to redirect the output using a pipe or redirection, eg. `./best-served-local ... > myfile.css`.


## Fully automated usage example

Run the script on your web server, together with valid values for
`--incss-fontpath`, `--outputfile`, `--fontdirectory` and the
`--overwrite-fonts` and `--overwrite-cssfile` arguments to get a
fully automated powertool:

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

## FONTSPEC

Should be space separated list of family names, with
optional suffix consisting of `:` with a comma separated list of
font weight/style values. 

For example to use *"Open Sans"* in the regular font weight and style use:

```bash
./best-served-local "Open Sans"
```
	  
To get the italic variant next to the regular one (with the same
weight), specify both, eg.

```bash
./best-served-local "Open Sans:regular:italic"
```

Multiple FONTSPECs should be space seperated (and surrounden with
quotes or space-escaped):

```bash
./best-served-local \
  "Open Sans:regular:italic" \
  "Web Font A:extrabold,superlight" \
  "Web Font B:100,200""
```


## FORMATSPECS

`FORMATSPECS` define the font formats for use in the CSS and
downloading. It should be specified as one of the presets, or a
comma-seperated list of specific formats.

The script recognize the following presets, taken from
https://css-tricks.com/snippets/css/using-font-face/:
* **superprogressive**: `woff2`
* **practical**:        *superprogressive* + `woff` (= default)
* **slightlydeeper**:   *practical* + `ttf`
* **all**:              all of the above + `eot`, `otf` and `svg`


For example, to use the (default) *practical* set consisting of the
`woff` and `woff2` formats for the "Open Sans" font (in regular
variant), use:
```bash
./best-served-local "Open Sans" 
```

To get full support (and heavy per page downloads), use:
```bash
./best-served-local -f all "Open Sans" 
```

You want something funky? Just tell the script to do so:
```bash
./best-served-local -f eot,svg "Open Sans" 
```

----

<a name="bulletproof">1</a>: This script uses the 
    [Bulletproof @font-face](http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)
    by Paul Irish.
