# best-served-local

> aka "Yet Another Google Web Font Downloader written in bash"

## Rationale

Inspired by the idea of [google-font-download] by Clemens Lang (also
written in bash v4) and the functionality of [google-webfonts-helper]
(client side browser app in javascript), I developed this script to
make it easy to [automate] (via cron and such) the
downloading of remote web fonts and the creation of high quality
[@font-face css3 rules] using the [bulletproof @font-face method] by
Paul Irish for serving from my own webserver.

Apart from [bash] version 4, this script only depends on [curl]. 

> It's not tested on OSX yet.


## Simple usage example

The commands:
```bash
./best-served-local "Open Sans" "Roboto:bold,thin"
```

and --using the html link element:
```bash
./best-served-local "<link href='https://fonts.googleapis.com/css?family=Open+Sans|Roboto:700,100' rel='stylesheet' type='text/css'>"
```

and --using the @import javascript statement:
```bash
./best-served-local "@import url(https://fonts.googleapis.com/css?family=Open+Sans|Roboto:700,100);"
```

will all display the following `@font-face` css at-rules:

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
	font-family: 'Roboto Bold';
	src: 
	     local('Roboto Bold'), local('Roboto-Bold'), 
	     url('Roboto_Bold_v15_latin_700.woff2') format('woff2'),
	     url('Roboto_Bold_v15_latin_700.woff') format('woff');
	font-style:  normal;
	font-weight: 700;
}
@font-face {
 	font-family: 'Roboto Thin';
	src: 
	     local('Roboto Thin'), local('Roboto-Thin'), 
	     url('Roboto_Thin_v15_latin_100.woff2') format('woff2'),
	     url('Roboto_Thin_v15_latin_100.woff') format('woff');
	     font-style:  normal;
	     font-weight: 100;
}
/* <<< script generated css ends here <<< */
```

And in the temporary directory, the following files are downloaded,
ready to be passed over to your webserver:

```
/tmp/best-served-local.XXXX
├── Open_Sans_v13_latin_400.woff
├── Open_Sans_v13_latin_400.woff2
├── Roboto_Bold_v15_latin_700.woff
├── Roboto_Bold_v15_latin_700.woff2
├── Roboto_Thin_v15_latin_100.woff
└── Roboto_Thin_v15_latin_100.woff2
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

**`FONTSPEC`**
: A `FONTSPEC` is either a space separated list of family names,
  surrounded with quotes or space-escaped, with an optional suffix,
  consisting of `:`, followed by a comma separated list of font
  weight/style values, or a full html link-element, as produced by
  https://www.google.com/fonts.  For example, to use *"Open Sans"* in
  the regular font weight and style use:
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

Or use an html 'link' element:
```bash
./best-served-local "<link href='https://fonts.googleapis.com/css?family=Open+Sans:400,400italic' rel='stylesheet' type='text/css'>"
```

Or use a javascript '@import' statement:
```bash
./best-served-local "@import url(https://fonts.googleapis.com/css?family=Open+Sans:400,400italic);"
```


### Optional arguments

**`-o PATH`** or **`--outputfile PATH`**
: The `PATH` of the file to save the generated css in. `PATH` can be
  relative to the working directory (eg. `fonts.css` or
  `../static/css/`) or absolute
  (eg. `/srv/www/example.org/static/css/fonts.css`).

**`-d PATH`** or **`--fontdirectory PATH`**
: The `PATH` of the directory to save the downloaded web fonts
  in. `PATH` can be relative to the working directory
  (eg. `../static/fonts`) or absolute
  (eg. `/srv/www/example.org/static/css/fonts`).

**`-i PATH`** or **`--incss-fontpath PATH`**
: The `PATH` of the directory to which font files will be referenced
  in the generated css using its `url` value. For example setting
  `PATH` to `../downloadedfonts` will lead to the following css `src:
  url('../downloadedfonts/A_Web_Font_v1_latin.woff2')`.
  
**`-n`** or **`--no-downloads`**
: Setting this argument causes the script to **not download** the web
  font files, which it by default does.

**`-x`** or **`--skip-local`**
: Prevents the inclusion of the 'local()' references in the `src`
  attribute.  The reason some people might want that is to prevent
  erronous versions of locally installed fonts to be used by the
  browser, instead of the ones served by your webserver.

**`-f FORMATSPECS`** or **`--formats FORMATSPECS`**
: `FORMATSPECS` define the font file formats to use. It should be
  specified as one of the *presets*, or a comma-separated list of
  specific formats. The script recognize the following presets, taken
  from https://css-tricks.com/snippets/css/using-font-face/:

`superprogressive`
: `woff2`

`practical`
: `superprogressive` + `woff` (= default)

`slightlydeeper`
: `practical` + `ttf`

`all`
: `slightlydeeper` + `eot` + `otf` + `svg`

For example, to use the default *practical* set, consisting of the
  `woff` and `woff2` formats, for the "Open Sans" font, use:

**`-s SUBSETSPEC`** or **`--subsets SUBSETSPEC`**
: Comma-separated list of `SUBSETS`. A `SUBSET` is one of the
  following predefined names: `cyrillic`, `cyrillic-ext`, `greek`,
  `greek-ext`, `latin` (default), `latin-ext`, `vietnamese` and `all`.

**`-overwrite-cssfile`**
: When used together with `--outputfile PATH`, using this argument
makes the script overwrite the file specified with `PATH` in case it
exists, which normally does not happen.

**`-overwrite-fonts`**
: When used together with `--fontdirectory PATH`, using this argument
  makes the script overwrite the font files in the directory specified
  with `PATH` in case they exists, which normally does not happen.


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
  http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/ 
  "*bulletproof @font-face method*"
  
[bash]: 
  https://www.gnu.org/software/bash/
  
[curl]:
  https://curl.haxx.se/

[github repository]: 
  https://github.com/ronalde/best-served-local

[downloaded]:
  http://lacocina.nl/best-served-local

[automate]:
  #fully-automated-usage-example

[FONTSPEC]:
  #required-argument

[commandline arguments]:
  #commandline-arguments-reference

[optional]:
  #optional-arguments
