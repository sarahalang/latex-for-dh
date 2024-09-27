# Final Documentation: Generating LaTeX on the fly in Graz GAMS Repository
Overview
The Graz GAMS repository focuses on XML data and XML standards archiving, aligning with the single-source principle. 
This principle dictates that only the XML data is archived, while all other outputs, such as print formats, are generated on the fly. 
Part of this process involved the creation of PDFs based on LaTeX through a multi-step process.

This document serves as a final record of the method that was used to generate LaTeX from XML data in GAMS, followed by its on-the-fly compilation into PDFs. 
As this process is no longer supported, the aim is to document the insights, issues, and implementation details for future reference in case the functionality is revived or adapted.




Process Outline
Archiving XML Data:
The first step is to archive the XML data. This XML data is essential for the transformation process and remains the core of the repository, as it adheres to the single-source principle.

XSLT Transformation to LaTeX:
The archived XML data undergoes an XSLT transformation to generate LaTeX code. 
This transformation is key to ensuring that a structured, printable output (PDF) can be created from the XML data. 
However, this introduces several challenges, especially in terms of debugging, due to potential issues in the generated LaTeX code.

Compilation to PDF Using Rubber:
Once the LaTeX code is generated, it is compiled on the fly into a PDF using a containerized system, leveraging Python Rubber. 
This method encountered several issues, which are discussed below.

Key Challenges
Debugging LaTeX Code
In the initial implementation, there was no method to inspect the LaTeX code prior to compilation, which complicated the debugging process. 
Issues in the LaTeX code could originate from the XSLT transformation process itself, such as missing or improperly nested curly braces. 
These errors are hard to detect without reviewing the generated LaTeX code.

To address this, Johannes Stiegle, the lead developer of GAMS, introduced the getLaTeX method. 
This method allowed for the inspection of LaTeX code prior to compilation, improving the ability to debug the output. 
This method proved crucial during the development and maintenance of this feature.

Compilation Problems
The containerized environment running Python Rubber for LaTeX compilation faced several issues:

Multiple Compilations:
LaTeX files often require multiple compilation runs to correctly render references, citations, and bibliographies. 
However, despite Rubber’s attempt to compile files multiple times, certain processes like BibTeX did not work as expected.

The team attempted to resolve this by creating an XSLT stylesheet that passed the BibTeX file into the LaTeX file using the filecontents environment. 
However, the BibTeX component still did not compile properly, indicating that the LaTeX process was likely not being run enough times.

Container and Disk Space Issues:
The container responsible for generating and compiling LaTeX grew in size during the process, consuming excessive resources. 
In particular, auxiliary files (with .aux extensions) were retained between runs, causing LaTeX to compile with outdated data. 
This required frequent manual intervention and debugging at the server level, which proved cumbersome and was not accessible to regular contributors.

Docker Compatibility:
As GAMS transitioned to a Dockerized system, mirroring the previous setup became problematic. 
The container bloated significantly during the compilation process, which was unsustainable, especially given the limited resources available for long-term use.

Decline in Usage and Final Decision
Due to the complexity of the setup, its resource demands, and the difficulty of debugging, the dynamic creation and compilation of LaTeX from XML became less viable. 
Moreover, it was noted that in most cases, users did not require dynamic compilation of files; they often needed the same output repeatedly. 
As a result, the team concluded that it was more resource-efficient to manually compile and archive the final PDFs alongside the XML data.

As this process became less utilized, the decision was made to no longer support it actively within GAMS.

Lessons Learned
Inspectability: Providing methods to inspect generated LaTeX before compilation is critical for effective debugging.
Multiple Compilation Requirements: For LaTeX processes requiring multiple runs (e.g., BibTeX), the compilation process needs to be explicitly configured to handle these, 
as standard tools like Python Rubber may not always suffice.
Container Resource Management: In a dynamic compilation environment, ensuring the container does not bloat due to retained auxiliary files is vital for maintaining performance.
Manual Over Dynamic Compilation: In many cases, especially for XML-based projects, manually compiling and archiving outputs can be a more sustainable approach than continuous dynamic generation, 
particularly for infrequently updated data.
Conclusion
The LaTeX generation and compilation system used in GAMS provided valuable insights into the challenges of dynamic print format generation from XML data. 
However, due to the resource intensity and limited use, the decision was made to phase out this functionality. 
The documentation herein provides an overview of the implementation, key challenges, and the reasoning behind the discontinuation of this service. 
Future users looking to reimplement or adapt this method should be aware of the potential pitfalls and consider alternative approaches, especially for large-scale or resource-limited projects.


## Testing Checklist

The following checklist can be used to test the LaTeX functionality in the GAMS repository:

- **Test GAMS content model:** Ensure the content model is working as expected for generating LaTeX output.
- **Test TEI to LaTeX (simple and GRaF pipelines):** Validate the transformation of TEI data to LaTeX using both simple and GRaF pipelines.
- **Example of very simple LaTeX:** Confirm that basic LaTeX examples compile successfully.
- **LaTeX with filecontents bib:** Test the integration of BibTeX through the `filecontents` environment in LaTeX.
- **LaTeX with images:** Verify that LaTeX documents containing images are processed and rendered correctly.
- **LaTeX containing TikZ:** Ensure that LaTeX documents using TikZ (for creating vector graphics) compile as intended.
- **LaTeX containing Ancient Greek:** Confirm that special characters, particularly Ancient Greek, are handled properly during the LaTeX generation and compilation process.
- **Is the cache cleared or are .aux files left lying around?:** Check if auxiliary files (e.g., `.aux`) are cleared after each compilation or if they accumulate unnecessarily.
- **Does `getLaTeX` work?:** Test the `getLaTeX` function to ensure that LaTeX code can be inspected prior to compilation.
- **Does `getLaTeXPDF` work?:** Verify that the `getLaTeXPDF` function successfully generates PDFs from LaTeX code.



## Known Issues and Lessons Learned

The method to get the LaTeX code generated by the XSLT stylesheet used to be called sdef:TEI/getLaTeX but can't even be accessed anymore on the current state of the system (as of 2024). 

All in all, experiences in the GRaF project (https://gams.uni-graz.at/graf) where excessive use of XML-to-LaTeX XSLT transformations 
was made to generate different versions of printouts for students and teachers (to be used in teaching contexts) show that not only is it resource intensive 
and relatively cumbersome to generate complicated LaTeX outputs from XML files on the side of those writing the stylesheets, it is also complicated and 
resource-intensive on the side of the GAMS developers and sysadmins. 
Thus, ultimately, the decision to only compile PDF versions based on XML-to-LaTeX transformations locally and archive a static PDF on GAMS is well justified. 

Due to the problems with compiling LaTeX in GAMS, certain LaTeX functionalities developed for the GRaF project (and to be used at DDH more broadly) 
were taken out of the stylesheet again as it turned out that GAMS will not compile this properly and lots of effort and time had already been invested 
in debugging the functionalities that were actually necessary. Ultimately, developing complicated LaTeX in this context seemed like a waste of time in the end. 
However, TEI-to-LaTeX only becomes really interesting for these complicated cases (such as creating a print version of critical apparatus using the LaTeX reledmac package). 
However, since people weren't really using these in practice anyway, they were not needed. 

Another issue encountered during the GRaF project was that the LaTeX methods were only available for archived XML files anyway, 
not  those provided as content datastreams - so we could only generate LaTeX from the texts in the repository, 
not all accompanying materials (such as project descriptions). For these, the method outlined here (of simply compiling the LaTeX locally and archiving the resulting static PDF) was already used. 

### Deprecated Methods
- The method to retrieve LaTeX code generated by the XSLT stylesheet, `sdef:TEI/getLaTeX`, was once central to the LaTeX generation process.
- However, as of 2024, this method is no longer accessible in the current system.

### Experiences from the GRaF Project
- In the GRaF project ([GAMS GRaF Project](https://gams.uni-graz.at/graf)), extensive use of XML-to-LaTeX XSLT transformations was employed to generate printouts
- for both students and teachers in teaching contexts. However, these experiences revealed that generating complex LaTeX outputs
- from XML files is resource-intensive and cumbersome for both those developing the XSLT stylesheets and the GAMS developers and sysadmins tasked with maintaining the system.
  
- The decision to compile the PDF versions locally and archive them as static PDFs in GAMS is well justified.
- This avoids the resource overhead and the complexity of compiling LaTeX dynamically, especially for more intricate transformations.

### Challenges with LaTeX Functionalities
- In the GRaF project, certain LaTeX functionalities developed specifically for use in GAMS and at the DDH
- (such as LaTeX for print versions of critical apparatus using the `reledmac` package) were eventually removed from the XSLT stylesheets.
- This decision was made after numerous efforts to debug the system revealed that GAMS could not compile the LaTeX code properly, making it unviable to continue developing these features.

- While transforming TEI to LaTeX could be particularly useful for generating complex print outputs, such as critical editions,
- these use cases were not prevalent in practice, leading to a decision to de-prioritise these advanced features.

### Limited Scope of LaTeX Methods
- An additional issue encountered during the GRaF project was that the LaTeX methods only worked with XML files already archived in the repository. For content datastreams or accompanying materials (e.g., project descriptions), the LaTeX generation method could not be applied. In these cases, the recommended approach was to compile LaTeX locally and archive the static PDF files.
