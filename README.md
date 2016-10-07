# best-served-local

> aka "Yet Another Google Web Font Downloader written in bash"

## Rationale

With the advent of using web fonts on your website --which is a good
idea-- you may (involuntary) submit some of the online behaviour of
*your visitors* (called [behavioural targeting]) to Google without their
knowledge or consent. This of course is a very bad idea. This script
assist you in preserving the good while preventing the bad by
downloading Google web fonts to your own webserver and create the
accompying CSS code to serve that yourself, instead of letting your
users downloading them from Google which each visit to your pages.

The script was inspired by [google-font-download] by Clemens Lang and
the functionality of [google-webfonts-helper] (client side browser app
in javascript). The latter uses javascript while I needed bash (v4),
while mr. Lang's script missed some functionality I would like. This
script adds [automation] and creates high quality
[@font-face css3 rules] using the [bulletproof @font-face method] by
Paul Irish and the format presets by Chris Coyier from
css-tricks.com.

Apart from [bash] version 4, this script only depends on either [curl] 
or [wget]. The script hasn't been tested on OSX and Cygwin yet.


## Simple usage example

Each of the following commands will lead to the same result:

* just specifying the font-family and optionally the font-weight(s):
```bash
./best-served-local "Open Sans" "Roboto:bold,thin" > /tmp/fonts.css
```
* or, use the html `link` element from your current templates and pages:
```bash
bash best-served-local -o /tmp/fonts.css \
  "<link href='https://fonts.googleapis.com/css?family=Open+Sans|Roboto:700,100' rel='stylesheet' type='text/css'>"
```
* or, use a css `@import` rule:
```bash
bash <(wget -q -O- "https://lacocina.nl/best-served-local") \
   --outputfile /tmp/fonts.css \
  "@import url(https://fonts.googleapis.com/css?family=Open+Sans|Roboto:700,100);"
```

All three commands will create the file `/tmp/fonts.css` with the
following `@font-face` css at-rules:

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

... and in a (new) temporary directory, the following files are
downloaded, ready to be served to your visitors by your webserver:

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
## running it straight from the web
bash <(wget -q -O - "https://lacocina.nl/best-served-local") "Roboto:100,900"
```

Or, when you prefer `curl`:
```bash
bash <(curl -sL "https://lacocina.nl/best-served-local") -f all Slabo\ 27px
```

Or, download first, run next:
```bash
wget "https://lacocina.nl/best-served-local"
bash best-served-local "Roboto:100,900"
```

Or, download to path, make executable, then run:
```bash
wget -O /usr/local/bin/best-served-local "https://lacocina.nl/best-served-local"
chmod +x /usr/local/bin/best-served-local
best-served-local -o /tmp/fonts.css "Roboto:100,900" "Slabo 27px"
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
--overwrite \
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
  weight/style values, or a full html `link` element or css `@import`
  rule, like the ones produced by https://www.google.com/fonts.
  
  For example, to use *"Open Sans"* in the regular font weight and
  style use:
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

  Or use an html `link` element:
```bash
./best-served-local "<link href='https://fonts.googleapis.com/css?family=Open+Sans:400,400italic' rel='stylesheet' type='text/css'>"
```

  Or use a css `@import` rule:
```bash
./best-served-local "@import url(https://fonts.googleapis.com/css?family=Open+Sans:400,400italic);"
```


### Optional arguments

**`-o PATH`** or **`--outputfile PATH`**
: The `PATH` of the file to save the generated css in. `PATH` can be
  relative to the working directory (eg. `fonts.css` or
  `../static/css/`) or absolute
  (eg. `/srv/www/example.org/static/css/fonts.css`). When not
  specified, the script prints the resulting css to `stdout`.

**`-d PATH`** or **`--fontdirectory PATH`**
: The `PATH` of the directory to save the downloaded web fonts
  in. `PATH` can be relative to the working directory
  (eg. `../static/fonts`) or absolute
  (eg. `/srv/www/example.org/static/css/fonts`). When not specified,
  the script will save the fonts in a temporary directory, which it
  will display when the script has finished.

**`-i PATH`** or **`--incss-fontpath PATH`**
: The `PATH` of the directory to which font files will be referenced
  in the generated css using its `url` value. For example setting
  `PATH` to `../downloadedfonts` will lead to the following css `src:
  url('../downloadedfonts/A_Web_Font_v1_latin.woff2')`.
  
**`-n`** or **`--no-downloads`**
: Setting this argument causes the script to **not download** the web
  font files, which it by default does do.
  > However, this does **not prevent the script to contact google's font
  > servers** for obtaining information on each requested font.

**`-x`** or **`--skip-local`**
: Prevents the inclusion of the `local()` references in the `src`
  attribute. Although the use of such references to fonts installed on
  the browsers' system saves downloading them at page load, they might
  differ from the versions served by your webserver. Skipping such
  references makes sure the browser gets what you want them to get.

**`-f FORMATSPECS`** or **`--formats FORMATSPECS`**
: `FORMATSPECS` define the font file formats to use. It should be
  specified as one of the *presets*, or a comma-separated list of
  specific formats. The script recognize the presets from
  [Using @font-face] by Chris Coyier from css-tricks.com:

  * `superprogressive`: [woff2]
  * `practical`: `superprogressive` + [woff] \(= default)
  * `slightlydeeper`: `practical` + [ttf]
  * `all`: `slightlydeeper` + [eot] + [otf] + [svg]

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
./best-served-local -f "eot,svg" "Open Sans" 
```

**`-s SUBSETSPEC`** or **`--subsets SUBSETSPEC`**
: Comma-separated list of `SUBSETS`. A `SUBSET` is one of the
  following predefined names: `cyrillic`, `cyrillic-ext`, `greek`,
  `greek-ext`, `latin` (default), `latin-ext`, `vietnamese` and `all`.

**`-overwrite-css`**
: When used together with `--outputfile PATH`, using this argument
makes the script overwrite the file specified with `PATH` in case it
exists, which normally does not happen.

**`-overwrite-fonts`**
: When used together with `--fontdirectory PATH`, using this argument
  makes the script overwrite the font files in the directory specified
  with `PATH` in case they exists, which normally does not happen.

**`-overwrite`**
: Sets both `--overwrite-css` and `--overwrite-fonts`.


---------------

[google-font-download]: 
  https://github.com/neverpanic/google-font-download.git/ "`google-font-download`"

[google-webfonts-helper]: 
  https://github.com/majodev/google-webfonts-helper/ "`google-webfonts-helper`"

[@font-face css3 rules]: 
  https://www.w3.org/TR/css-fonts-3/#font-face-rule

[woff2]: 
  https://www.w3.org/TR/WOFF2/

[woff]: 
  https://www.w3.org/TR/WOFF/

[ttf]:
  https://www.microsoft.com/typography/TrueTypeFonts.mspx

[eot]:
  https://en.wikipedia.org/wiki/Embedded_OpenType

[otf]:
  http://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=66391

[svg]:
  https://www.w3.org/TR/SVG/fonts.html

[bulletproof @font-face method]: 
  http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/ 
  "*bulletproof @font-face method*"
  
[bash]: 
  https://www.gnu.org/software/bash/
  
[curl]:
  https://curl.haxx.se/

[wget]:
  https://www.gnu.org/software/wget/
  
[github repository]: 
  https://github.com/ronalde/best-served-local

[downloaded]:
  https://lacocina.nl/best-served-local

[using @font-face]:
  https://css-tricks.com/snippets/css/using-font-face

[automation]: #fully-automated-usage-example

[FONTSPEC]: #required-argument

[commandline arguments]: #commandline-arguments-reference

[optional]: #optional-arguments

[behavioural targeting]:
	http://dare.uva.nl/document/2/154442
