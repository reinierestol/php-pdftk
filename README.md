php-pdftk
=========

[![Build Status](https://secure.travis-ci.org/mikehaertl/php-pdftk.png)](http://travis-ci.org/mikehaertl/php-pdftk)
[![Latest Stable Version](https://poser.pugx.org/mikehaertl/php-pdftk/v/stable.svg)](https://packagist.org/packages/mikehaertl/php-pdftk)
[![Total Downloads](https://poser.pugx.org/mikehaertl/php-pdftk/downloads)](https://packagist.org/packages/mikehaertl/php-pdftk)
[![Latest Unstable Version](https://poser.pugx.org/mikehaertl/php-pdftk/v/unstable.svg)](https://packagist.org/packages/mikehaertl/php-pdftk)
[![HHVM Status](http://hhvm.h4cc.de/badge/yiisoft/yii2-dev.png)](http://hhvm.h4cc.de/package/mikehaertl/php-pdftk)
[![License](https://poser.pugx.org/mikehaertl/php-pdftk/license.svg)](https://packagist.org/packages/mikehaertl/php-pdftk)

A PDF conversion and form utility based on pdftk.

**The `pdftk` command must be installed and working on your system.**

> Note: This library is written for pdftk 2.x versions. You should be able to
> use it with pdftk 1.x but not all methods will work there. For details consult
> the man page of pdftk on your system.

## Features

*php-pdftk* brings the full power of `pdftk` to PHP - and more.

 * Fill forms, either from a XFDF/FDF file or from a data array (UTF-8 safe for unflattened forms, requires pdftk 2.x !)
 * Create XFDF or FDF files from PHP arrays (UTF-8 safe!)
 * Create FDF files from filled PDF forms
 * Combine pages from several PDF files into a new PDF file
 * Split a PDF into one file per page
 * Add background or overlay PDFs
 * Read out meta data about PDF and form fields
 * Set passwords and permissions

## Installation

You should use use [composer](https://getcomposer.org/) to install this library.

```
composer require mikehaertl/php-pdftk
```

## Examples

### Operations

Please consult the `pdftk` man page for each operation to find out how each operation works
in detail and which options are available.

> Note: You can always only perform **one** of the following operations on a single PDF instance.
> Below you can find a workaround if you need multiple operations.

For all operations you can either save the PDF locally through `saveAs($name)` or send it to the
browser with `send()`. If you pass a filename to `send($name)` the client browser will open a download
dialogue whereas without a filename it will usually display the PDF inline.

#### Fill Form

Fill a PDF form with data from a PHP array or an XFDF/FDF file.

```php
use mikehaertl\pdftk\Pdf;

// Fill form with data array
$pdf = new Pdf('form.pdf');
$pdf->fillForm(array('name'=>'ÄÜÖ äüö мирано čárka'))
    ->needAppearances()
    ->saveAs('filled.pdf');

// Fill form from FDF
$pdf = new Pdf('form.pdf');
$pdf->fillForm('data.xfdf')
    ->saveAs('filled.pdf');
```

> Note: When filling in UTF-8 data, you should always add the needAppearances() option.
> This will make sure, that the PDF reader takes care of using the right fonts for rendering,
> something that pdftk can't do for you.

#### Create a XFDF/FDF file from a PHP array

> Note: This is a bonus feature that is not available from `pdftk`.

```php
use mikehaertl\pdftk\FdfFile;

$fdf = new XfdfFile(['name'=>'Jürgen мирано']);
$fdf->saveAs('data.xfdf');

$fdf = new FdfFile(['name'=>'Jürgen мирано']);
$fdf->saveAs('data.fdf');
```

#### Cat

Assemble a PDF from pages of one or more PDF files.

```php
use mikehaertl\pdftk\Pdf;

// Extract pages 1-5 and 7,4,9 into a new file
$pdf = new Pdf('my.pdf');
$pdf->cat(1, 5)
    ->cat(array(7, 4, 9))
    ->saveAs('new.pdf');

// Combine pages from several files, demonstrating several ways how to add files
$pdf = new Pdf(array(
    'A' => 'file1.pdf',                 // Reference file as 'A'
    'B' => ['file2.pdf','pass**word'],  // Reference file as 'B'
));
$pdf->addFile('file3.pdf','C','**secret**pw');  // Reference file as 'C'
$pdf->cat(1, 5, 'A')                // pages 1-5 from A
    ->cat(3, null, 'B')             // page 3 from B
    ->cat(7, 'end', 'B', null, 'east') // pages 7-end from B, rotated East
    ->cat('end',3,'A','even')       // even pages 3-end in reverse order from A
    ->cat([2,3,7], 'C')             // pages 2,3 and 7 from C
    ->saveAs('new.pdf');
```

#### Shuffle

Like `cat()` but create "*streams*" and fill the new PDF with one page from each
stream at a time.

```php
use mikehaertl\pdftk\Pdf;

// new.pdf will have pages A1, B3, A2, B4, A3, B5, ...
$pdf = new Pdf(array(
    'A' => 'file1.pdf',     // Reference file as 'A'
    'B' => 'file2.pdf',     // Reference file as 'B'
));
$pdf->shuffle(1, 5, 'A')    // pages 1-5 from A
    ->shuffle(3, 8, 'B')    // pages 3-8 from B
    ->saveAs('new.pdf');
```

#### Burst

Split a PDF file into one file per page.

```php
use mikehaertl\pdftk\Pdf;

$pdf = new Pdf('my.pdf');
$pdf->burst('page_%d.pdf');     // Supply a printf() pattern
```

#### Add background PDF

Add another PDF file as background.

```php
use mikehaertl\pdftk\Pdf;

// Set background from another PDF (first page repeated)
$pdf = new Pdf('my.pdf');
$pdf->background('back.pdf')
    ->saveAs('watermarked.pdf');

// Set background from another PDF (one page each)
$pdf = new Pdf('my.pdf');
$pdf->multiBackground('back_pages.pdf')
    ->saveAs('watermarked.pdf');
```

#### Add overlay PDF

Add another PDF file as overlay.

```php
use mikehaertl\pdftk\Pdf;

// Stamp with another PDF (first page repeated)
$pdf = new Pdf('my.pdf');
$pdf->stamp('overlay.pdf')
    ->saveAs('stamped.pdf');

// Stamp with another PDF (one page each)
$pdf = new Pdf('my.pdf');
$pdf->multiStamp('overlay_pages.pdf')
    ->saveAs('stamped.pdf');
```

#### Generate FDF

Create a FDF file from a given filled PDF form.

```php
use mikehaertl\pdftk\Pdf;

// Create FDF from PDF
$pdf = new Pdf('form.pdf');
$pdf->generateFdfFile('data.fdf');
```

#### Get PDF data

Read out metadata or form field information from a PDF file.

```php
use mikehaertl\pdftk\Pdf;

// Get data
$pdf = new Pdf('my.pdf');
$data = $pdf->getData();

// Get form data fields
$pdf = new Pdf('my.pdf');
$data = $pdf->getDataFields();
```

#### How to perform more than one operation on a PDF

As stated above, you can only perform one of the preceeding operations on a single PDF instance.
If you need more than one operation you can feed one `Pdf` instance into another:

```php
use mikehaertl\pdftk\Pdf;

// Extract pages 1-5 and 7,4,9 into a new file
$pdf = new Pdf('my.pdf');
$pdf->cat(1, 5)
    ->cat(array(7, 4, 9));

// We now use the above PDF as source file for a new PDF
$pdf2 = new Pdf($pdf);
$pdf2->fillForm(array('name'=>'ÄÜÖ äüö мирано čárka'))
    ->needAppearances()
    ->saveAs('filled.pdf');
```

### Options

You can combine the above operations with one or more of the following options.

```php
use mikehaertl\pdftk\Pdf;

$pdf = new Pdf('my.pdf');

$pdf->allow('AllFeatures')      // Change permissions
    ->flatten()                 // Merge form data into document
    ->compress($value)          // Compress/Uncompress
    ->keepId('first')           // Keep first/last Id of combined files
    ->dropXfa()                 // Drop newer XFA form from PDF
    ->dropXmp()                 // Drop newer XMP data from PDF
    ->needAppearances()         // Make clients create appearance for form fields
    ->setPassword($pw)          // Set owner password
    ->setUserPassword($pw)      // Set user password
    ->passwordEncryption(128)   // Set password encryption strength
    ->saveAs('new.pdf');

// Example: Fill PDF form and merge form data into PDF
// Fill form with data array
$pdf = new Pdf('form.pdf');
$pdf->fillForm(array('name'=>'My Name'))
    ->flatten()
    ->saveAs('filled.pdf');

// Example: Remove password from a PDF
$pdf = new Pdf;
$pdf->addPage('my.pdf', null, 'some**password')
    ->saveAs('new.pdf');
```

The class uses [php-shellcommand](https://github.com/mikehaertl/php-shellcommand) to execute
`pdftk`. You can pass `$options` for its `Command` class as second argument to the constructor:

```php
use mikehaertl\pdftk\Pdf;

$pdf = new Pdf('my.pdf', [
    'command' => '/some/other/path/to/pdftk',
    'useExec' => true,
]);
```

## API

Please consult the source files for a full documentation of each method.
