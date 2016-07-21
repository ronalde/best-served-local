# best-served-local (aka Yet Another Google Font Downloader written in bash)

## Rationale

Inspired by
[google-font-download](https://github.com/neverpanic/google-font-download.git)
and
[google-webfonts-helper](https://google-webfonts-helper.herokuapp.com/fonts)
I developed this script to make it easy to automate (via cron and
such) the downloading of remote web fonts and the creation of high
quality css<sup><a href="#bulletproof">1</a></sup> for serving from my
own webserver.


## Simple usage example

The command:
```bash
./best-served-local "Open Sans:400;bolditalic;light" "Some other webfont"
```

will display `@font-face` CSS-at-rules for the "Open Sans" web font
in the variations "*regular*" (`font-weight: 400; font-style:
regular`), "*bold-italic* (`font-weight: 700; font-style: italic`) and
"*light* (`font-weight: 300; font-style: regular`), in the
`latin`-subset, in the file formats of the *practical* superset
(`woff` and `woff2`) as defined by
https://css-tricks.com/snippets/css/using-font-face/. 

Without options, the script will download the font to a temporary directory. 

## Fully automated usage

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
cause the resulting `url()` values in the `srv` attribute to use that
as the path to the font file, eg using `-i ../static/fonts` will
result in the following css:

```css
url('../static/fonts/A_Web_Font_v1_latin.woff2')
```

While omitting that option would result in:
```css
url('A_Web_Font_v1_latin.woff2')
```


## Yet some more extras 

Using the `--skip-local` (or `-x`) argument will make the script skip
the `local` references in the `src` attributes, which comes in handy
if you want maximum control over the appearance of your website.

Using `--skip-downloads` (or `-n`) will not download the font files,
but just output the proper CSS. It will however contact Goole-server
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
* **all**:              all of the above + `otf` and `svg`


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
./best-served-local -f otf,svg "Open Sans" 
```

----

<a name="bulletproof">1</a>: This script uses the 
    [Bulletproof @font-face](http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)
    by Paul Irish.
