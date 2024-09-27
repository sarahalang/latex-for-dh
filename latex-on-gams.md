# Final Documentation: Generating LaTeX on the fly in the Graz GAMS Repository

The [Graz GAMS repository](gams.uni-graz.at) focuses on XML data archiving and provides a number of methods for generating different outputs from this XML data on the fly, following the single-source principle. (This principle dictates that only the XML data is archived, while all other outputs, such as print formats, are generated automatically from this data.)
Part of this process involved the creation of PDFs based on LaTeX through a multi-step process.

This document serves as a final record of the method used to generate LaTeX from XML data in GAMS, creating LaTeX code from the XML data on the fly using XSLT followed by compilation into PDFs. 
As this process is no longer supported, the aim is to document the insights and issues here for future reference in case the functionality is revived or adapted.

---

I will provide examples on how this can be done and tutorials for minimal documents in this repository (["LaTeX for Digital Humanities"](https://github.com/sarahalang/latex-for-dh/)). 

---

## How the process used to work
1. **XML data is stored in GAMS:** This XML data is at the core of the repository which implements the single-source principle.
2. **XSLT Transformation generates LaTeX code:** The archived XML data undergoes an XSLT transformation to generate LaTeX code in the `getLaTeXPDF()` method. This is available only for XML files that are archived in the repository, not for XML files located in content data streams. The output of this method used to be accessible via a `getLaTeX()` method (`sdef:TEI/getLaTeX`) which at the time of writing is already not available anymore. However, being able to view the auto-generated LaTeX code is essential for debugging and being able to tell if the compilation errors result from the code itself or the multi-step compilation routine on the server. As of 2024, this method is no longer accessible in the current system (dockerized version of GAMS3).
3. **PDF is compiled from the code using Python's Rubber** ([`latex-rubber`](https://pypi.org/project/latex-rubber/)): Once the LaTeX code is generated, it is compiled into a PDF using a containerized system using Python Rubber. This method encountered several issues, which are discussed below.

---

## Key Challenges
### Debugging LaTeX Code
In the initial implementation, there was no method to inspect the LaTeX code prior to compilation, which complicated the debugging process. 
Errors aren't only introduced in the compilation process, especially as this LaTeX code is being generated through an XSLT transformation, which could have already introduced errors as writing LaTeX code through XSLT transformations is a little tricky. 
Issues in the LaTeX code from the XSLT transformation often originate in the discrepancies of XML and LaTeX, such as white-space management and nesting, resulting in typical errors such as missing or improperly nested curly braces. These errors are hard to detect without reviewing the generated LaTeX code.

To address this, [Johannes Stigler](https://orcid.org/0000-0003-0803-1496), the lead developer of GAMS at the time, introduced the `getLaTeX()` method. 
This method allowed for the inspection of LaTeX code sent into the automated compilation process, making it easier to debug the output. 

### Difficulty with some LaTeX features during compilation 
The containerized environment running Python Rubber also faced several issues:

LaTeX files often require multiple compilation runs to correctly render elements such as references, citations, and bibliographies. 
However, despite the fact that Rubber should actually compile multiple times, we never got certain processes like BibTeX to work as expected (at least not without investing a lot of resources which we deemed wasn't worth it at the time).

Furthermore, the GAMS TEI-to-LaTeX transformation produces one output that can be piped into the `getLaTeXPDF()` method (which only accepts one file). 
However, LaTeX bibliographies often come in extra `.bib` files. 
We attempted to resolve this by creating the XSLT stylesheet so that it placed the contents of the BibTeX file (`.bib`) at the start of the LaTeX file (`.tex`) using the `filecontents` environment. 
However, the BibTeX component still did not compile properly, indicating that the LaTeX process was likely not being run enough times.

### Container and Disk Space Issues:
The container responsible for generating and compiling LaTeX grew in size during the process, consuming excessive resources. 
Additionally, auxiliary files (`.aux`) were retained between runs, causing LaTeX to compile with outdated data. Technically, this container should have been recreated from scratch each time. But in practice at some point, it became apparent that old auxiliary files were still present. 
This required frequent manual intervention and debugging at the server level which isn't ideal if it were supposed to be used by lots of projects.

---

## Lessons Learned
All in all, this process had many issues such as the complexity of the setup, its resource demands, and the difficulty of debugging, the dynamic creation and compilation of LaTeX from XML. 
Moreover, it was noted that in most cases, users did not actually require dynamic compilation of files; they always needed the same output, rendering dynamic compilation unnecessary. 
As a result, the team concluded that it was more resource-efficient to manually compile locally and archive the final PDFs alongside the XML data.

Experiences in the GRaF project (https://gams.uni-graz.at/graf) where excessive use of XML-to-LaTeX XSLT transformations 
was made to generate different versions of printouts for students and teachers (to be used in teaching contexts) show that not only is it resource-intensive 
and relatively cumbersome to generate complicated LaTeX outputs from XML files on the side of those writing the stylesheets, it is also complicated and 
resource-intensive on the side of the GAMS developers and sysadmins. 
Thus, ultimately, the decision to only compile PDF versions based on XML-to-LaTeX transformations locally and archive a static PDF on GAMS is well justified. 


### Experiences from the GRaF Project
In the GRaF project ([GAMS GRaF Project](https://gams.uni-graz.at/graf)), extensive use of XML-to-LaTeX XSLT transformations was employed to generate printouts for both students and teachers in teaching contexts. However, these experiences revealed that generating complex LaTeX outputs from XML files is resource-intensive and cumbersome for both those developing the XSLT stylesheets and the GAMS developers and sysadmins tasked with maintaining the system.

In the GRaF project, some LaTeX functionalities developed specifically for use in GAMS and at the DDH (such as LaTeX for print versions of critical apparatus using the `reledmac` package) were eventually removed from the XSLT stylesheets. This decision was made after numerous efforts to debug the system revealed that GAMS would not compile the LaTeX code properly, making it unviable to continue investing time and energy in these features. 
While transforming TEI to LaTeX could be particularly useful for generating complex print outputs, such as critical editions, these use cases aren't actually used in practice at DDH/Zim, rendering these features pointless and potential sources for lots of work and errors.

Another issue encountered during the GRaF project was that the LaTeX methods were only available for archived XML files anyway, 
not  those provided as content datastreams - so we could only generate LaTeX from the texts in the repository, 
not all accompanying materials (such as project descriptions). For these, the method outlined here (simply compiling the LaTeX locally and archiving the resulting static PDF) was already used. 

### Testing Checklist

The following checklist can be used as an inspiration for testing the LaTeX functionality in the GAMS repository:

- **Test GAMS content model:** Ensure the content model is working as expected for generating LaTeX output.
- **Test TEI to LaTeX (simple and GRaF pipelines):** Validate the transformation of TEI data to LaTeX using both simple (i.e. default template) and pipelines from the [GRaF project](https://gams.uni-graz.at/graf).
- **Example of very simple LaTeX:** Confirm that basic LaTeX examples compile successfully.
- **LaTeX with filecontents bib:** Test the integration of BibTeX through the `filecontents` environment in LaTeX.
- **LaTeX with images:** Verify that LaTeX documents containing images are processed and rendered correctly.
- **LaTeX containing TikZ:** Ensure that LaTeX documents using TikZ (for creating vector graphics) compile as intended.
- **LaTeX containing Ancient Greek:** Confirm that special characters, particularly Ancient Greek, are handled properly during the LaTeX generation and compilation process. This needs special packages and examples can be found in [GRaF](https://gams.uni-graz.at/graf).
- **Is the cache cleared properly after each run or are `.aux` files left lying around?** Even though all `.aux` files should be cleaned out, this was not always the case in the past and resulted in errors that were hard to trace down. 
- **Does `getLaTeX()` work?:** Test the `getLaTeX()` method to ensure that LaTeX code can be inspected before compilation.
- **Does `getLaTeXPDF()` work?:** Verify that the `getLaTeXPDF()` method successfully generates PDFs from LaTeX code.

