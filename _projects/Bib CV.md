---
layout: single
title: "Bib Support for CVs"
excerpt: "Annoyed that all my personal data never changes when personalizing CV/Resume, only the styles."
header:
  teaser: "/images/BIB CV.png"
quality_level: 1
permalink: /bibcv/
---
<!-- OPEN IN OVERLEAF BUTTON -->
[![Open in Overleaf][#overleaf-badge]][#github-repo-zip]
[#overleaf-badge]: https://tinyurl.com/overleaf-badge
[#github-repo-zip]: https://www.overleaf.com/docs?snip_uri=https://www.overleaf.com/docs?snip_uri=https%3A%2F%2Fmdsantia.github.io%2Ffiles%2FBIB_CV.zip
<!-- SOURCE https://gist.github.com/sugatoray/5c9ec0d837bb0cc98bc7d98544a91c6f -->
<!-- <a class="open-overleaf-btn"
   href="https://www.overleaf.com/docs?snip_uri=https%3A%2F%2Fmdsantia.github.io%2Ffiles%2FBIB_CV.zip"
   target="_blank">
   Open in Overleaf
</a> -->
<!-- DOWNLOAD ZIP BUTTON -->
<a class="download-zip-btn"
   href="../files/BIB_CV.zip"
   download>
   Download ZIP
</a>

# Documentation
## Getting Started
Let us familiarize ourselves with the structure of the project. We have four types of files in our project:
1. .bbx
2. .bib
3. .sty
4. .tex

### BBX
The first kind is what defines the structure for the bibtex to output as desired. It may seem overwhelming but there is only two main things to focus on:

#### Declarations
These are the responsible section when adding custom types and fields. 
```LaTex
\DeclareDatamodelFields[type=field,datatype=date]{date,enddate}
\DeclareDatamodelEntrytypes{job,award,talk}
\DeclareDatamodelEntryfields[job]{howpublished,institution,location,position,year,month,endyear,endmonth}
...
\DeclareBibliographyAlias{job}{misc}
```

#### Drivers
This is responsible for the specific format the output is in. Do not be intimidated:
```LaTex
% --- Job entries ---
\DeclareBibliographyDriver{job}{%
  \noindent
  \textbf{\printfield{title}}%
  \hfill\textit{\usebibmacro{printdaterange}}%
  \par
  \printfield{institution}, \printfield{location}%
  \par
  \usebibmacro{cv:printnote}%
  \vspace{0.5em}%
}
```
just use `\noindent` if no indentation, `\printfield{}` to print a value, `\addcomma\addperiod\vspace\hfill` etc. for formatting, other than that is just normal $\TeX{}$.

#### MACROS
Any additional help functions can be built through `macros`. As an example,
```LaTex
% --- INITIALIZATION - Helper macro: print date range ---
\newbibmacro*{printdaterange}{%
  \iffieldundef{year}{}{\printdate}%
  \iffieldundef{endyear}
    {\space--\space\textit{Current}}%
    {\iffieldundef{endmonth}
       {\space--\space\printfield{endyear}}%
    }%
}

...
\usebibmacro{printdaterange} % TO CALL HELPER
```

### BIB
Hopefully, you are familiar with the main structure of a bib file and how to build entries. Nothing changes for this project other than, we have customized certain types, `@job` for example, and added new fields depending on the type, `endyear`. An example,

```bibtex
@job{gta,
  priority     = {01},
  title        = {Graduate Teaching Assistant},
  year         = {2021},
  month        = {08},
  endmonth     = {07},
  endYear      = {2023},
  institution  = {Iowa State University},
  location      = {Ames, IA, USA},
  position     = {Graduate Teaching Assistant},
  note         = {\begin{itemize}
    \item Served as Lead Instructor for a summer section of Calculus II (14 students)\\
    \item Served as a Teaching Assistant for: Calculus I (60 students), Calculus II (90 students), Discrete Mathematics for Business and Social Sciences (138 students), and College Algebra (44 students)
    \end{itemize}
  },
  keywords     = {job}
}
```

Most importantly, I made it so that the `note` field serves as the main _Description Section_ of the `job` for example, and it is written as a list in $\TeX$, but this can be customized within the `.bbx` and `.sty` files.

***ENSURE*** that if there is no `endmonth` or `endyear` define it by the empty braces.

### STY
The `.sty` file is a common type of file to set the preamble commands and constructions of a project. It is was allows in the `.tex` to use the `\usepackage{}` command. Most notably, the basic `cvstyle.sty` is made just to support multiple sorting algorithms so that entries output in a certain order.

The classic `biblatex` sorts:
1. none → citation order (like unsrt).
2. nyt → name, year, title (like plain).
3. ynt → year, name, title.
4. debug → shows internal sort keys for debugging

in addition to the cv specific sorts:
1. recentStart → most recent events listed first
2. chronologicalStart → events listed chronologically
3. currentTop → current ongoing, endyear descending
4. custom → follows `priority` field in `.bib` values

### .TEX
#### Automatic CV
To automatically fill all the entries from all the bibliographies, you can use the `\nocite{*}` command after the `\addbibresource`, otherwise, you would have to olist explicitly all the entries interested in including.
#### Removing Entries
Using the `\addtocategory{hidden}{[key]}` command hides the specific entry from the output.

#### Customizations
You can add other packages styles, section commands, etc. to make your CV stand out, for example 
```LaTex
\resumeSubHeadingListStart
    \resumeSubheading
      {Graduate Mathematics Major Course Work}{August 2021 -- Present}
      {Iowa State University}{Ames, IA}
      \resumeItemListStart
        \resumeItem{Topology}
        \resumeItem{Abstract Algebra I}
      \resumeItemListEnd
  \resumeSubHeadingListEnd
```
from 
```LaTex
%-------------------------
% Resume in Latex
% Author : Jake Gutierrez
% Based off of: https://github.com/sb2nov/resume
% License : MIT
%------------------------
```
***BESIDES THAT*** Go crazy in your `.tex`!