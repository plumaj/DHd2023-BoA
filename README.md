# DHd2023-BoA
This repository contains the set of scripts that will take TEI encoded XML files and process them into a PDF. Furthermore, it contains the files used to prepare the Book of Abstracts of the DHd 2020 conference. Please be aware that the texts are under the standard copyright of the authors, if not stated explicitly otherwise.

The following steps explain how to generate the Book of Abstracts. They are based on [Nina Seemanns](https://github.com/NinaSeemann/DHd2020-BoA) repository, which is based on [Karin Dalziels introduction](https://github.com/karindalziel/TEI-to-PDF), but with some modifications to reflect the changes in the code over the years.

#### Code was tested on a Server running Ubuntu 22.04.2 LTS with 6 cores and 4GB of memory.

Step 0: Check out or download files from github
===============================================

You'll get a folder with the name "DHd2020-BoA". All other
instructions refer to this folder as the root folder.

Step 1: Set up your environment
=======================

1a: Download Saxon
-------------------------

Download Saxon HE from
[The Saxon Sourceforge page](http://saxon.sourceforge.net/) and place
it in the "lib" folder or change the code below to match your download. Code has been tested with Saxon 11.4 HE. 

1b: Download and configure the FO processor
----------------------------------------

Download the FOP Binary files from
[The Apache FOP Page](http://xmlgraphics.apache.org/fop/) and place it
inside the "lib" folder or change the commands below to match your download. Scripts have been tested with FOP 2.8.

#### Change the conf file

The conf file can be found here:  /lib/fop/conf/fop.xconf

You can use &lt;auto-detect/&gt; to register all fonts on your system. If
some fonts are not in the standard directory, you can tell FOP where
to look for them. Place the following inside the &lt;renderer
mime="application/pdf">/&lt;fonts> tags:

	<directory recursive="true">/Non-Standard_Path/Fonts</directory>
	<auto-detect/>

For further information on font embedding, visit the
[ Apache FOP Font page](https://xmlgraphics.apache.org/fop/2.0/fonts.html).

#### Hyphenation with offo

For hyphenation, the [offo hyphenation binaries](http://offo.sourceforge.net/index.html) are needed. 
For FOP 1.1 and higher, there are already compiled pattern files which can be downloaded. Simply follow the link on the homepage.
The fop-hyph.jar has to be copied into FOP's lib folder (./lib/fop-2.8/lib)

#### Increase memory

If you are creating a large book, you may need to increase the memory
FOP uses to process the book. This is done in the config.sh in the
config folder. Currently, java_exec_args is set to 'd64 -Xmx3000m'
which was sufficient to generate this years BoA (283 pages). 

1c: File structure
--------------------

There have been some changes over the years. The final file structure should look like this:

* config (folder)
  * config.sh
* input (folder)
  * images (folder)
  * xml (folder)
* lib (folder)
  * fop (folder)
  * saxon (folder)
  * tei2pdf (folder)
	* empty.xml
	* TEIcorpus_producer.xsl
	* xsl-fo-producer.xsl
* output (folder)
* README.md
* run.sh

1d: Prepare your Files
--------------------------

#### TEI

TEI filenames are not used for categorization, the code uses the category in the header file to place the content in the correct category.
Therefore, all TEI files must be P5 and categorized into 'Workshop', 'Panel',
'Vortrag', 'Doctoral Consortium' or 'Posterpräsentation' so the book
can generate the proper headings. The categorization is already present when
files are downloaded from the ConfTool. For example code see [Karin Dalziels README](https://github.com/karindalziel/TEI-to-PDF).

* xml:id : Make sure the xml:id matches the document name and is
unique! Otherwise, the code will crash when a person is listed as author in
multiple papers, e.g. ab-002.xml must have an xml:id of ab-002:

		<TEI xml:id="ab-002" xmlns="http://www.tei-c.org/ns/1.0">

* graphics : There are two important things to check. (i) Make sure
that every &lt;graphic&gt; (i.e. image) is included in a
&lt;figure&gt;. Otherwise, those images will not be displayed in the final
pdf. Missing figure does not issue a warning! (ii) Make sure, that only **one** 
&lt;graphic&gt; is included in a figure. Otherwise, the second image
will not be displayed. Again, this does not issue a warning. 

#### Images

Change all file paths to local, e.g. "path/to/image/image001.jpg" should be simply "image001.jpg"

1e: Set your configuration options
-----------------------------------------

In the lib/tei2pdf/xsl-fo-producer.xsl file are several options for
setting up how you would like your book to look.
These options haven't changed over time, so see [Karin Dalziels explanation](https://github.com/karindalziel/TEI-to-PDF).

Step 2: Run Files
============

Command line instructions are below. 

1: Place all files and images in input/xml and input/images, respectively

2: Open a terminal and change directory (cd command) to the root folder. 

3: Create the Book_Corpus.xml corpus file with the TEIcorpus_producer.xsl XSL with this command:

	 java -jar lib/SaxonHE9-8-0-7J/saxon9he.jar lib/tei2pdf/empty.xml > output/Book_Corpus.xml

This uses the saxon engine to transform all the XML in the final_xml folder into a TEI Corpus file called "Book_Corpus.xml"

4: Create the .fo file:

	java -jar lib/SaxonHE11-4J/saxon11he.jar output/Book_Corpus.xml lib/tei2pdf/xsl-fo-producer.xsl > output/pdf.fo

This creates a file called pdf.fo. Any errors (missing images, etc) will be exported to the screen. You may get a notice about missing fonts - this means either your font name is incorrect or you have not set FOP to use your system fonts in the conf file (detailed above).

5: Create the .pdf file:

	lib/fop-2.8/fop -d64 -Xmx3000m -c lib/fop-2.8/conf/fop.xconf output/pdf.fo output/pdf.pdf

This will create the final PDF. 

If you get an out of memory error, see section on configuring FOP
above. You will likely get a bunch of font errors, but these may or may not matter. Check your final file to make sure all characters display correctly. 

#### Script / Config file
There are two convenient scripts that help running files. 
* run.sh : Type ./run.sh in the root folder and everything is run at
once. If this fails, running each step individually will give better
error reporting. This file gets the information about the setup from the
config file. 
* config/config.sh : This file contains all information about the
  setup environment, e.g. Saxon-jar, FOP base directory, pdf output directory, ... If your
  environment setup differs from the way described above, you can
  adapt the necessary information here. 
