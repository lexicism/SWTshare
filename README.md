I'd like to add a few notes about some points in this thread that I believe there is a bit of confusion on. I have limited time at the moment, but I will try to come back later to revise my post to be a bit more technical.

On October 2, 2023, balakrishnan1978 opened an issue about how Acrobat Read out loud reads Alternate Text that has been assigned to Formula tags in PDFs created with accessibility features using the tagpdf package. They were concerned that the Read out loud feature was explicitly pronouncing the backslashes in LaTeX formulas, which is quite repetitive (something which should be avoided for accessibility).

To address this concern, we need to understand a few things:

1. The Read out loud feature that Adobe includes in Acrobat is **not** a screen reader. It generally shouldn't be the primary focus of accessibility testing, although it can be useful for some people in terms of accessibility and can be helpful in casual, quick testing. However, a PDF that can be correctly read by Acrobat's Read out loud isn't necessarily an accessible document. A PDF that can be correctly read by a screen reader *would* be an accessible document.

    An example of a screen reader would be [NVDA](https://www.nvaccess.org/download/), a program which has a suggested donation but is also available free of charge.

    (Curious about when Acrobat's Read out loud feature would be helpful for accessibility? An example might be if someone had a temporary disability that made looking at screens difficult, meaning they need transitory tools, but they don't need a full-on screen reader. However, as stated before, the threshold of being correctly read by Read out loud is not high enough for true accessibility testing.)

2. Assistive technology (such as a screen reader) pronouncing the LaTeX formula for math in place of the math is not accessible. I'm working on some notes about why this is, but in short the assistive technology needs to pronounce "spoken math" from a spoken math generator such as [MathSpeak](https://www.seewritehear.com/learn/mathspeak-and-mathspeak-rules/) or [Clear Speak](https://onlinelibrary.wiley.com/doi/full/10.1002/ets2.12103) (as examples; there are other spoken math generators).
3. Because PDFs haven't really supported MathML all that much in the past, the transitory solution that has up to now been used was representing equations with images and assigning spoken math for the equation as Alternate Text for the Figure tag of that image. Alternate Text is pronounced "as-is," meaning you can't put code in the Alternate Text field for the screen reader to interpret. If a symbol such as a backslash would be explicitly pronounced as a backslash in the main body of the document, then in the Alternate Text field it will also be pronounced as a backslash by the screen reader. If a symbol such as a dash would be pronounced as a pause in the main body of the document, it will be pronounced as a pause by the screen reader. 

    There are ways to adjust the settings of a screen reader such as NVDA so that it will pronounce all symbols explicitly (for example, a dash would be pronounced as "dash" instead of as a pause; all of that just depends on the preference of the person using the screen reader).

In their opening post, balakrishnan1978 shares the following as their Minimal Working Example:

```
\DocumentMetadata{testphase={phase-III,math},pdfstandard=UA-1,pdfversion=2.0,debug=pdfmanagement,debug={xmp-export=test,uncompress}}
\documentclass{article}

\usepackage{fontspec}
\usepackage{unicode-math}
\setmainfont{STIX Two Text}
\setmathfont{STIX Two Math}
\begin{document}

In order to learn a bilingual mapping that connects two embeddings into one space, the mapping matrix $W_{x}$ makes $Y_{t}=W_{X}X_{s}$, we employ an adversarial training to learn the mapping. In particular, the generator $G$ tries to make a mapping matrix to confuse the discriminator. In order to get orthogonal parameterization, we note that transforming the source word embedding into the target, its transpose should also transform the target to the source. The $G$ tries to learn a mapping $W_{X}$ to maximize a source representation $W_{X}X_{s}$ to mapping into a target embedding $Y_{t}$. The discriminator D is a binary classifier which aims to enhance its ability to distinguish $Y_{t}$ and $W_{X}X_{s}$. To achieve this goal, we maximize the ability of the discriminator and minimize the probability of generator. \vspace*{-3pt}
\begin{equation}
score_{L_{s},L_{t}} = ∑_{i=1}^{max length(L_{s},L_{t})} ± CSCL(x_{s}^{i},y_{t}^{j})
\end{equation}
\end{document}
```

and was concerned that Read out loud was explicitly pronouncing the backslashes for the math.

As of today, the way Read out loud pronounces the first sentence of the document body of their MWE (which contains two examples of inline math) as: "In order to learn a bilingual mapping that connects two embeddings into one space, the mapping matrix **Latex formula starts backslash begin math W underscore X backslash end math Latex formula ends** makes **Latex formula starts backslash begin math Y underscore t equals W underscore X X underscore S backslash end math Latex formula ends**."

 Read out loud pronounces it this way because the Alternate Text fields are set to, for example, things like "LaTeX formula starts \begin {math} W_{x} \end {math} LaTeX formula ends " (this is for $W_{X}$). In order to be sure Read out loud pronounces the LaTeX equation literally, the Alternate Text field would have to be set to "backslash begin left brace math right brace W unerscore left brace x right brace backslash end left brace math right brace" or something similar. 

However, Read out loud should not be pronouncing the LaTeX equation at all; it should be pronouncing spoken math, and spoken math generators take the issue of literal symbol pronunciation into account. If you plug "W_{x}" into the [MathJax Speech Converter Demo](https://mathjax.github.io/MathJax-demos-web/speech-generator/convert-with-speech.html), you can see that the Alternate Text field should be set to "upper W Subscript x" if you use Verbose-setting Mathspeak. (Mathspeak in the verbose setting would probably be the most appropriate engine in the MathJax demo for tagpdf to use, given the package is intended for lab use.)

Finally, I believe the real issue that balakrishnan1978 is concerned about is how math should be spoken in PDFs for accessibility testing, which poses the question "How does NVDA pronounce the math in balakrishnan1978's MWE?" (after compiling the MWE using LuaLaTeX). The answer is that it doesn't—in fact, it doesn't pronounce most of the document at all. The first words NVDA pronounces are "we employ an adversarial training. . . ." This is because the tags are incorrectly nested and Alternate Text is being used where Actual Text should be used.

If Formula tags aren't nested within paragraphs, the document (not just the content within the Formula tag) will not pronounce correctly. Here are the conditions for pronunciation of Alternate Text and Actual Text of a Formula tag:

- Formula tags with **both** Alternate Text and Actual Text: The Alternate Text will suppress the Actual Text and NVDA will pronounce the Alternate Text.
- Formula tags with **only** Alternate Text and **no** Actual Text: NVDA will pronounce nothing.
- Formula tags with **no** Alternate Text and **only** Actual Text: NVDA will pronounce the Actual Text.

This makes inuitive sense if you look at the definitions of Actual Text and Alternate Text and the way they're billed, even though Actual Text is supposed to be used for as short of phrases as possible, but that matter is for another day.

(If you wanted to use a Figure tag with only Alternate Text, you could do that, but NVDA will announce the math as a "Graphic" which is unnecessary and makes the document needlessly repetitive without adding any new information. If you tag text or an image as a Figure tag and assign Alternate Text, NVDA will pronounce it, regardless of whether the Figure tag is or is not nested under a Paragraph tag. NVDA will not pronounce the Alternate Text of a Figure tag if the Figure tag is nested under a Formula tag.)

These issues should probably be kept in mind for anyone wanting to develop the above regex solutions further. For example, let's look at the regex version posted by OP balakrishnan1978 in response to davidcarlisle.

Now, it can't be pronounced by NVDA directly after compilation, because the Actual Text field is empty and the Formula tag for the display equation isn't nested in a Paragraph tag, but we can quickly change those things in Acrobat to see how the document *would* be read if those issues were resolved by the package. (I don't have time write now to draft a TEX file that would implement these solutions directly, but I will try to post updates if I have time). 

I used "placeholder" in the Actual Text fields of the Formula tags.

To test the PDF in NVDA, after making the above edits, open NVDA by pressing Ctrl + Alt + N, then open NVDA settings by pressing NVDA Key + N, then navigate to Preferences > Settings > Speech > Punctuation/symbol level and select your desired setting (start with "some"). Back in the PDF, mouse over an element to hear it read; if you're having problems, try closing it and opening it again to hear to entire document read aloud from start to finish.

The equation needs to be pronounced correctly regardless of the Punctuation/symbol level setting. Most users probably want to listen to the main text of the document with certain punctuation pronounced as a pause (such as parenthesis) and they need to be able to hear intelligible math when they get to it without opening up the NVDA settings and switching "some" to "all."

This table records how NVDA reads the Alt Text for $XYZ_{S}$ ("LaTeX formula starts \begin {math} XYZ_{s} \end {math} LaTeX formula ends") with different Punctuation/symbol levels:

**NVDA Pronunciations of "LaTeX formula starts \begin {math} XYZ_{s} \end {math} LaTeX formula ends"**

| Punctuation/symbol level | Pronunciation |
|---|---|
|None|"La T E X formula starts begin math X Y Z S end math La T E X formula ends"|
|Some|"La T E X formula starts begin math X Y Z S end math La T E X formula ends "|
|Most|"La T E X formula starts backslash begin left brace math right brace X Y Z line left brace S right brace end left brace math right brace La T E X formula ends |
|All|"La T E X formula starts backslash begin left brace math right brace X Y Z line left brace S right brace end left brace math right brace La T E X formula ends "|

When further developing solutions to fix the current Alt Text behavior, it is important to keep in mind that the Pronunciation column needs to be the same for all rows in the above table, something you won't be able to evaluate by listening in Acrobat's Read out loud.

While there is apparently a [development version](https://ctan.math.washington.edu/tex-archive/macros/latex/contrib/tagpdf/tagpdf.pdf#page=5) of NVDA available that can interpret MathML (which would obviate the need for a package to set the spoken math as a static element within a PDF), it isn't available publicly yet, and it might be good to continue the development of setting correct spoken math in the case that it takes a long time to become publicly available, or the initial rollout faces issues.

I used LuaHBTeX, Version 1.18.0 (TeX Live 2024), NVDA 2024.4.2.35031, and Adobe Acrobat Pro Continuous Release Version 2025.001.20435.
