# Caracal

[![Build Status](http://img.shields.io/travis/trade-informatics/caracal.svg?style=flat)](https://travis-ci.org/trade-informatics/caracal)
[![Gem Version](http://img.shields.io/gem/v/caracal.svg?style=flat)](https://rubygems.org/gems/caracal)


## Overview

Caracal is a ruby library for dynamically creating professional-quality Microsoft Word documents using an HTML-style syntax.

Caracal is not a magical HTML to Word translator. Instead, it is a markup language for generating Office Open XML (OOXML). Programmers create Word documents by issuing a series of simple commands against a document object. When the document is rendered, Caracal takes care of translating those Ruby commands into the requisite OOXML. At its core, the library is essentially a templating engine for the :docx format.

Or, said differently, if you use [Prawn](https://github.com/prawnpdf/prawn) for PDF generation, you'll probably like Caracal. Only you'll probably like it better. :)


### Why is Caracal Needed?

We created Caracal to satisfy a genuine business requirement.  We were working on a system that produced a periodic PDF report and our clients asked if the report could instead be generated as a Word document, which would allow them to perform edits before passing the report along to their clients.

Now, as you may have noticed, the Ruby community has never exactly been known for its enthusiastic support of Microsoft standards. So it might not surprise you to learn that the existing options on Rubygems for Word document generation were limited.  Those libraries, by and large, fell into a couple of categories:

* **HTML to Word Convertors**  
We understand the motivating idea here (two output streams from one set of instructions), but the reality is the number of possible permutations of nested HTML tags is simply too great for this strategy to ever work for anything other than the simplest kinds of documents. Most of these libraries rely on a number of undocumented assumptions about the structure of your HTML (which undermines the whole value proposition of a convertor) and fail to support basic features of a professional-quality Word document (e.g., images, lists, tables, etc). The remaining libraries simply did not work at all.

* **Weekend Projects**  
We also found a number of inactive projects that appeared to be experiments in the space. Obviously, these libraries were out of the question for a commercial product.

What we wanted was a Prawn-style library for the :docx format. In the absence of an active project organized along those lines, we decided to write one.


### Design

Caracal is designed to separate the process of parsing and collecting rendering instructions from the process of rendering itself.

First, the library consumes all programmer instructions and organizes several collections of data models that capture those instructions. These collections are ordered and nested exactly as the instructions we given. Each model is contains all the data required to render it and is responsible for declaring itself valid or invalid.

***Note**: Some instructions create more than one model. For example, the `img` method both appends both an `ImageModel` to the main contents collection and determines whether or not a new `RelationshipModel` should be added to the relationships collection.*

Only after all the programmer instructions have been parsed does the document attempt to render the collection to XML. Generally speaking renderers simply convert model data to markup that conforms to the OOXML specification.  But, this strategy gives the rendering process a tremendous amount of flexibility in the rare cases where renderers combine data from more than one collection.


### File Structure

You may not know that .docx files are simply a zipped collection of XML documents that follow the OOXML standard. (We didn't, in any event.) This means constructing a .docx file from scratch actually requires the creation of several files.  Caracal abstracts users from this process entirely.

For each Caracal request, the following document structure will be created and zipped into the final output file:

    example.docx
      |- _rels
      	|- .rels
      |- docProps
        |- app.xml
        |- core.xml
      |- word
        |- _rels
          |- document.xml.rels
        |- media
          |- image001.png
          |- image002.png
          ...
        |- document.xml
        |- fontTable.xml
        |- footer.xml
        |- numbering.xml
        |- settings.xml
        |- styles.xml
      |- [Content_Types].xml


### File Descriptions

The following provides a brief description for each component of the final document:

**_rels/.rels**  
Defines an internal identifier and type for global content items. *This file is generated automatically by the library based on other user directives.*

**docProps/app.xml**  
Specifies the name of the application that generated the document. *This file is generated automatically by the library based on other user directives.*

**docProps/core.xml**  
Specifies the title of the document. *This file is generated automatically by the library based on other user directives.*

**word/_rels/document.xml.rels**  
Defines an internal identifier and type with all external content items (images, links, etc). *This file is generated automatically by the library based on other user directives.*

**word/media/**  
A collection of media assets (each of which should have an entry in document.xml.rels).

**word/document.xml**  
The main content file for the document.

**word/fontTable.xml**  
Specifies the fonts used in the document.

**word/footer.xml**  
Defines the formatting of the document footer.

**word/numbering.xml**  
Defines ordered and unordered list styles.

**word/settings.xml**  
Defines global directives for the document (e.g., whether to show background images, tab widths, etc). Also, establishes compatibility with older versions on Word.

**word/styles.xml**  
Defines all paragraph and table styles used through the document.  Caracal adds a default set of styles to match its HTML-like content syntax.  These defaults can be overridden.

**[Content_Types].xml**  
Pairs extensions and XML files with schema content types so Word can parse them correctly. *This file is generated automatically by the library based on other user directives.*


### Units

OpenXML properties are specified in several different units, depending on which attribute is being set.

**Points**  
Most spacing declarations are measured in full points.

**Half Points**  
All font sizes are measure in half points.  A font size of 24 is equivalent to 12pt.

**Eighth Points**  
Borders are measured in 1/8 points.  A border size of 4 is equivalent to 0.5pt.

**Twips**  
A twip is 1/20 of a point.  Word documents are printed at 72dpi.  1in == 72pt == 1440 twips.

**Pixels**  
In Word documents, pixels are equivalent to points.

**EMUs (English Metric Unit)**  
EMUs are a virtual unit designed to facilitate the smooth conversion between inches, milliimeters, and pixels for images and vector graphics.  1in == 914400 EMUs == 72dpi x 100 x 254.

At present, Caracal expects values to be specified in whichever unit the OOXML requires. This is admittedly difficult for new Caracal users. Eventually, we'll probably implement a utility object under the hood to convert user-specified units into the format expected by OOXML. 


### Syntax Flexibility

Generally speaking, Caracal commands will accept instructions via any combination of a parameters hash and/or a block.  For example, all of the folowing commands are equivalent.

```ruby
docx.style 'special', size: 24, bold: true

docx.style 'special', size: 24 do
  bold true
end

docx.style 'special' do
  size 24
  bold true
end
```

Parameter options are always evaluated before block options. This means if the same option is provided in the parameter hash and in the block, the value in the block will overwrite the value from the parameter hash. Tread carefully.


### Validations

All Caracal models perform basic validations on their attributes, but this is, without question, the least sophisticated part of the library at present.

In forthcoming versions of Caracal, we'll be looking to expand the `InvalidModelError` class to provide broader error reporting abilities across the entire library.


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'caracal'
```

Then execute:

```bash
bundle install
```


## Commands

In the following examples, the variable `docx` is assumed to be an instance of Caracal::Document.

```ruby
docx = Caracal::Document.new('example_document.docx')
```

Most code examples show optional values being passed in a block.  As noted above, you may also pass these options as a parameter hash or as a combination of a parameter hash and a block.


### File Name

The final output document's title can be set at initialization or via the `file_name` method.

```ruby
docx = Caracal::Document.new('example_document.docx')

docx.file_name 'different_name.docx'
```

The current document name can be returned by invoking the `name` method:

```ruby
docx.name    # => 'example_document.docx'
```

*The default file name is caracal.docx.*


### Page Size

Page dimensions can be set using the `page_size` method.  The method accepts two parameters for controlling the width and height of the document.

*Page size defaults to United States standard A4, portrait dimensions (8.5in x 11in).*

```ruby
# options via block
docx.page_size do
  width   12240     # sets the page width. units in twips.
  height  15840     # sets the page height. units in twips.
end

# options via hash
docx.page_size width: 12240, height: 15840
```

The `page_size` command will produce the following XML in the `document.xml` file:

```xml
<w:sectPr>
  <w:pgSz w:w="12240" w:h="15840"/>
</w:sectPr>
```

### Page Margins

Page margins can be set using the `page_margins` method.  The method accepts four parameters for controlling the margins of the document.  
*Margins default to 1.0in for all sides.*

```ruby
# options via block
docx.page_margins do
  left    720     # sets the left margin. units in twips.
  right   720     # sets the right margin. units in twips.
  top     1440    # sets the top margin. units in twips.
  bottom  1440    # sets the bottom margin. units in twips.
end

# options via hash
docx.page_margins left: 720, right: 720, top: 1440, bottom: 1440
```

The `page_margins` command above will produce the following XML in the `document.xml` file:

```xml
<w:sectPr>
  <w:pgMar w:left="720" w:right="720" w:top="1440" w:bottom="1440"/>
</w:sectPr>
```

### Page Breaks

Page breaks can be added via the `page` method.  The method accepts no parameters.

```ruby
docx.page     # starts a new page.
```

The `page` command will produce the following XML in the `document.xml` file:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:br w:type="page"/>
  </w:r>
</w:p>
```

### Page Numbers

Page numbers can be added to the footer via the `page_numbers` method.  The method accepts an optional parameter for controlling the alignment of the text.

*Page numbers are turned off by default.*

```ruby
# no options
docx.page_numbers true

# options via block
docx.page_numbers true do
  align :right     # controls text alignment. defaults to :center.
end

# options via hash
docx.page_numbers true, align: :right
```

The default command will produce the following `footer.xml` file contents.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:ftr xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing" xmlns:w10="urn:schemas-microsoft-com:office:word" xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml" xmlns:sl="http://schemas.openxmlformats.org/schemaLibrary/2006/main" xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main" xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture" xmlns:c="http://schemas.openxmlformats.org/drawingml/2006/chart" xmlns:lc="http://schemas.openxmlformats.org/drawingml/2006/lockedCanvas" xmlns:dgm="http://schemas.openxmlformats.org/drawingml/2006/diagram">
  <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
    <w:pPr>
      <w:contextualSpacing w:val="0"/>
      <w:jc w:val="center"/>
    </w:pPr>
    <w:fldSimple w:dirty="0" w:instr="PAGE" w:fldLock="0">
      <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
        <w:rPr/>
      </w:r>
    </w:fldSimple>
    <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
      <w:rPr>
        <w:rtl w:val="0"/>
      </w:rPr>
    </w:r>
  </w:p>
</w:ftr>
```

*It will also automatically add the correct notation to the `w:sectPr` node of the `document.xml` file.*


### Fonts

Fonts are added to the font table file by calling the `font` method and passing the name of the font.  At present, Caracal only supports declaring the primary font name.

```ruby
docx.font name: 'Arial'
docx.font do
  name 'Droid Serif'
end
```

These commands will produce the following `fontTable.xml` file contents:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:fonts xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing" xmlns:w10="urn:schemas-microsoft-com:office:word" xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml" xmlns:sl="http://schemas.openxmlformats.org/schemaLibrary/2006/main" xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main" xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture" xmlns:c="http://schemas.openxmlformats.org/drawingml/2006/chart" xmlns:lc="http://schemas.openxmlformats.org/drawingml/2006/lockedCanvas" xmlns:dgm="http://schemas.openxmlformats.org/drawingml/2006/diagram">
  <w:font w:name="Arial"/>
  <w:font w:name="Droid Serif"/>
</w:fonts>
```

### Styles

Style classes can be added using the `style` method.  The method accepts several optional parameters to control the rendering of text using the style.

```ruby
# options via block
docx.style do
  type      :paragraph      # :paragraph or :table
  id        'Heading1'      # sets the internal identifier for the style.
  name      'heading 1'     # set the friendly name of the style.
  color     '333333'        # sets the text color. values in hex RGB.
  font      'Droid Serif'   # sets the font family.
  size      28              # set the font size. units in half points.
  bold      false           # sets the font weight.
  italic    false           # sets the font style.
  underline false           # sets whether or not to underline the text.
  align     :left           # sets the alignment. accepts :left, :center, :right, and :both.
  top       100             # sets the spacing above the paragraph. units in twips.
  bottom    0               # sets the spacing below the paragraph. units in twips.
  spacing   360             # sets the spacing between lines. units in twips.
end
```

The `style` command above would produce the following XML:
```xml
<w:style w:styleId="Heading1" w:type="paragraph">
  <w:name w:val="heading 1"/>
  <w:basedOn w:val="Normal"/>
  <w:next w:val="Normal"/>
  <w:pPr>
    <w:keepNext w:val="0"/>
    <w:keepLines w:val="0"/>
    <w:widowControl w:val="0"/>
    <w:contextualSpacing w:val="1"/>
  </w:pPr>
  <w:rPr>
    <w:rFonts w:cs="Droid Serif" w:hAnsi="Droid Serif" w:eastAsia="Droid Serif" w:ascii="Droid Serif"/>
    <w:sz w:val="28"/>
  </w:rPr>
</w:style>
```

### Paragraphs

Text can be added using the `p` method.  The `p` either takes a string and a `class` option or a block of `text`-like commands.  

Text within a `p` block can be further defined using the `text` and `link` methods.  The `text` method takes a text string and the optional parameters `style`, `color`, `size`, `bold`, `italic`, and `underline`.  See below for details on the `link` method.

```ruby
docx.p 'some text', style: 'my_style'

docx.p do
  text 'Here is a sentence with a ', style: 'my_style'
  link 'link', 'https://www.example.com'
  text ' to something awesome', color: '555555', size: 16, bold: true, italic: true, underline: true
end
```


A `p` block might yield the following XML:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:contextualSpacing w:val="0"/>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">Here is a sentence with a </w:t>
  </w:r>
  <w:hyperlink r:id="rId6">
    <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
      <w:rPr>
        <w:color w:val="1155cc"/>
        <w:u w:val="single"/>
        <w:rtl w:val="0"/>
      </w:rPr>
      <w:t xml:space="preserve">link</w:t>
    </w:r>
  </w:hyperlink>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:b w:val="1"/>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve"> to something awesome.</w:t>
  </w:r>
</w:p>
```

### Headers

Headers can be added using the `h1`, `h2`, etc. methods.  Text within a header block can be further defined using the `text` method.

*Ultimately, headers are just paragraphs that use header styles.*

```ruby
docx.h3 'Heading 3'
```

The `h3` block above will yield the following XML:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:pStyle w:val="Heading3"/>
    <w:contextualSpacing w:val="0"/>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">Heading 3</w:t>
  </w:r>
</w:p>
```

### Links

Links can be added inside paragraphs by using the `link` method.  The method accepts several optional parameters for controlling the style and behavior of the rule.  

*At present, all links are assumed to be external.*

```ruby
# no options
docx.p do
  link 'Example Text', 'https://wwww.example.com'
end

# options via block
p do
  link 'Example Text', 'https://wwww.example.com' do
    style       'my_style'   # sets the style class. defaults to nil.
    color       '0000ff'     # sets the color of the text. defaults to 1155cc.
    size        24           # sets the font size. units in half-points. defaults to nil.
    bold        false        # sets whether or not the text will be bold. defaults to false.
    italic      false        # sets whether or not the text will be italic. defaults to false.
    underline   true         # sets whether or not the text will be underlined. defaults to true.
  end
end

# options via hash
p do
  link 'Example Text', 'https://wwww.example.com', color: '0000ff', underline: false
end
```

The `link` command with default properties will produce the following XML output:

```xml
<w:hyperlink r:id="rId1">
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:color w:val="1155cc"/>
      <w:u w:val="single"/>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">Example Text</w:t>
  </w:r>
</w:hyperlink>
```

*Caracal will automatically generate the relationship entries required by the OpenXML standard.*

```xml
<Relationship Target="https://www.example.com" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink" TargetMode="External" Id="rId1"/>
```

### Images

Images can be added by using the `img` method.  The method accepts several optional parameters for controlling the style and placement of the asset.

```ruby
# options via block
docx.img image_url('example.png') do
  width   396       # sets the image width. units specified in pixels.
  height  216       # sets the image height. units specified in pixels.
  align   :right    # controls the justification of the image. default is :left.
  top     10        # sets the top margin. units specified in pixels.
  bottom  10        # sets the bottom margin. units specified in pixels.
  left    10        # sets the left margin. units specified in pixels.
  right   10        # sets the right margin. units specified in pixels.
end

# options via hash
docx.img image_url('example.png'), width: 396, height: 216, align: :right
```

The `img` command with default properties will produce the following XML output:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:spacing w:lineRule="auto" w:line="276"/>
    <w:contextualSpacing w:val="0"/>
    <w:jc w:val="right"/>
    <w:rPr/>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:drawing>
      <wp:inline distR="114300" distT="114300" distB="114300" distL="114300">
        <wp:extent cy="2743200" cx="5029200"/>
        <wp:effectExtent t="0" b="0" r="0" l="0"/>
        <wp:docPr id="1" name="image00.png"/>
        <a:graphic>
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
            <pic:pic>
              <pic:nvPicPr>
                <pic:cNvPr id="0" name="image00.png"/>
                <pic:cNvPicPr preferRelativeResize="0"/>
              </pic:nvPicPr>
              <pic:blipFill>
                <a:blip r:embed="rId5"/>
                <a:srcRect t="0" b="0" r="0" l="0"/>
                <a:stretch>
                  <a:fillRect/>
                </a:stretch>
              </pic:blipFill>
              <pic:spPr>
                <a:xfrm>
                  <a:ext cy="2743200" cx="5029200"/>
                </a:xfrm>
                <a:prstGeom prst="rect"/>
                <a:ln/>
              </pic:spPr>
            </pic:pic>
          </a:graphicData>
        </a:graphic>
      </wp:inline>
    </w:drawing>
  </w:r>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
  </w:r>
</w:p>
```

*Caracal will automatically generate the relationship entries required by the OpenXML standard.*

```xml
<Relationship Target="media/image00.png" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Id="rId5"/>
```

### Rules

Horizontal rules can be added using the `hr` method.  The method accepts several optional parameters for controlling the style of the rule.  

```ruby
# no options
docx.hr                 # defaults to a thin, single line.

# options via block
docx.hr do
  color   '333333'   # controls the color of the line. defaults to auto.
  line    :double    # controls the line style (single or double). defaults to single.
  size    8          # controls the thickness of the line. units in 1/8 points. defaults to 4.
  spacing 4          # controls the spacing around the line. units in points. defaults to 1.
end

# options via hash
docx.hr color: '333333', line: :double, size: 8, spacing: 2  
```

The `hr` command with default properties will produce the following XML output:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:pBdr>
      <w:top w:color="auto" w:space="1" w:val="single" w:sz="4"/>
    </w:pBdr>
  </w:pPr>
</w:p>
```

### Ordered Lists

Ordered lists can be added using the `ol` and `li` methods.  The `li` method substantially follows the same rules as the the `p` method; here, simpler examples are demonstrated.

```ruby
docx.ol do
  li 'First item'
  li 'Second item'
end
```

The `ol` and `li` commands with default properties will produce the following XML (assuming the ordered list styles have the abstractNumId=2 in the `numbering.xml` file).

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:numPr>
      <w:ilvl w:val="0"/>
      <w:numId w:val="2"/>
    </w:numPr>
    <w:ind w:left="720" w:hanging="359"/>
    <w:contextualSpacing w:val="1"/>
    <w:rPr>
      <w:u w:val="none"/>
    </w:rPr>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">First item</w:t>
  </w:r>
</w:p>
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:numPr>
      <w:ilvl w:val="0"/>
      <w:numId w:val="2"/>
    </w:numPr>
    <w:ind w:left="720" w:hanging="359"/>
    <w:contextualSpacing w:val="1"/>
    <w:rPr>
      <w:u w:val="none"/>
    </w:rPr>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">Second item</w:t>
  </w:r>
</w:p>
```

### Unordered Lists

Unordered lists can be added using the `ul` and `li` methods.  The `li` method substantially follows the same rules as the the `p` method; here, simpler examples are demonstrated.

```ruby
docx.ul do
  li 'First item'
  li 'Second item'
end
```

The `ul` and `li` commands with default properties will produce the following XML (assuming the ordered list styles have the abstractNumId=1 in the `numbering.xml` file).

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:numPr>
      <w:ilvl w:val="0"/>
      <w:numId w:val="1"/>
    </w:numPr>
    <w:ind w:left="720" w:hanging="359"/>
    <w:contextualSpacing w:val="1"/>
    <w:rPr>
      <w:u w:val="none"/>
    </w:rPr>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">First item</w:t>
  </w:r>
</w:p>
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:numPr>
      <w:ilvl w:val="0"/>
      <w:numId w:val="1"/>
    </w:numPr>
    <w:ind w:left="720" w:hanging="359"/>
    <w:contextualSpacing w:val="1"/>
    <w:rPr>
      <w:u w:val="none"/>
    </w:rPr>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
    <w:t xml:space="preserve">Second item</w:t>
  </w:r>
</w:p>
```

### Tables

Tables can be added using the `table` method.  The method accepts several optional paramters to control the layout and style of the table cells.

```ruby
table data, border: 8 do
  cell_style  rows(0), background_color: '4a86e8', bold: true
end
```

Given the a data structure with two rows and five columns, the `table` method would produce the following XML:

```xml
<w:tbl>
  <w:tblPr>
    <w:tblStyle w:val="KixTable1"/>
    <w:bidiVisual w:val="0"/>
    <w:tblW w:w="10800.0" w:type="dxa"/>
    <w:jc w:val="left"/>
    <w:tblBorders>
      <w:top w:color="000000" w:space="0" w:val="single" w:sz="8"/>
      <w:left w:color="000000" w:space="0" w:val="single" w:sz="8"/>
      <w:bottom w:color="000000" w:space="0" w:val="single" w:sz="8"/>
      <w:right w:color="000000" w:space="0" w:val="single" w:sz="8"/>
      <w:insideH w:color="000000" w:space="0" w:val="single" w:sz="8"/>
      <w:insideV w:color="000000" w:space="0" w:val="single" w:sz="8"/>
    </w:tblBorders>
    <w:tblLayout w:type="fixed"/>
    <w:tblLook w:val="0600"/>
  </w:tblPr>
  <w:tblGrid>
    <w:gridCol w:w="2160"/>
    <w:gridCol w:w="2160"/>
    <w:gridCol w:w="2160"/>
    <w:gridCol w:w="2160"/>
    <w:gridCol w:w="2160"/>
    <w:tblGridChange w:id="0">
      <w:tblGrid>
        <w:gridCol w:w="2160"/>
        <w:gridCol w:w="2160"/>
        <w:gridCol w:w="2160"/>
        <w:gridCol w:w="2160"/>
        <w:gridCol w:w="2160"/>
      </w:tblGrid>
    </w:tblGridChange>
  </w:tblGrid>
  <w:tr>
    <w:tc>
      <w:tcPr>
        <w:shd w:fill="4a86e8"/>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:b w:val="1"/>
            <w:color w:val="ffffff"/>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">Field</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:shd w:fill="4a86e8"/>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:b w:val="1"/>
            <w:color w:val="ffffff"/>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">Response</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:shd w:fill="4a86e8"/>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:b w:val="1"/>
            <w:color w:val="ffffff"/>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">Perf. Quality</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:shd w:fill="4a86e8"/>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:b w:val="1"/>
            <w:color w:val="ffffff"/>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">Data Quality</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:shd w:fill="4a86e8"/>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:b w:val="1"/>
            <w:color w:val="ffffff"/>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">State</w:t>
        </w:r>
      </w:p>
    </w:tc>
  </w:tr>
  <w:tr>
    <w:tc>
      <w:tcPr>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">After-Hours trading</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">Yes</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">B</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">B</w:t>
        </w:r>
      </w:p>
    </w:tc>
    <w:tc>
      <w:tcPr>
        <w:tcMar>
          <w:top w:w="100.0" w:type="dxa"/>
          <w:left w:w="100.0" w:type="dxa"/>
          <w:bottom w:w="100.0" w:type="dxa"/>
          <w:right w:w="100.0" w:type="dxa"/>
        </w:tcMar>
      </w:tcPr>
      <w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
        <w:pPr>
          <w:spacing w:lineRule="auto" w:after="0" w:line="240" w:before="0"/>
          <w:ind w:left="0" w:firstLine="0"/>
          <w:contextualSpacing w:val="0"/>
        </w:pPr>
        <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
          <w:rPr>
            <w:rtl w:val="0"/>
          </w:rPr>
          <w:t xml:space="preserve">published</w:t>
        </w:r>
      </w:p>
    </w:tc>
  </w:tr>
</w:tbl>
```

### Line Breaks

Line breaks can be added via the `br` method.  The method accepts no parameters.

```ruby
docx.br     # adds a blank line using the default paragrpah style.
```

The `br` command will produce the folowing XML:

```xml
<w:p w:rsidP="00000000" w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000" w:rsidRDefault="00000000">
  <w:pPr>
    <w:contextualSpacing w:val="0"/>
  </w:pPr>
  <w:r w:rsidRPr="00000000" w:rsidR="00000000" w:rsidDel="00000000">
    <w:rPr>
      <w:rtl w:val="0"/>
    </w:rPr>
  </w:r>
</w:p>
```


## Template Rendering

Caracal includes [Tilt](https://github.com/rtomayko/tilt) integration to facilitate its inclusion in other frameworks.  Rails integration can be added via the [Caracal-Rails](https://github.com/trade-informatics/caracal-rails) gem.


## Defaults

[Unsure how best to handle this without code exploration. Not a critical element for the first version.]


## Contributing

1. Fork it ( https://github.com/trade-informatics/caracal/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request


## Why is It Called Caracal?

Because my son likes caracals. :)


## Inspiration

A tip of the hat to the wonderful PDF generation library [Prawn](https://github.com/prawnpdf/prawn).


## License

Copyright (c) 2014 Trade Informatics, Inc

[MIT License](https://github.com/trade-informatics/caracal/blob/master/LICENSE.txt)
