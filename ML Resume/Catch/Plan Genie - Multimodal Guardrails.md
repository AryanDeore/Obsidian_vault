[[Plan Genie - Architecture|Plan Genie]] is a customer facing chatbot to help customer chat about their current plan or compare plans from multiple providers. Customers are allowed to upload a [[SBC - Extraction Pipeline|SBC]] of their current plan or the plan they want to compare. Letting people upload files introduce some challenges:
1. Do we know if the file uploaded is pdf? How do we verify that?
2. Do we know if the uploaded pdf is pure or a [**Polyglot**]{A polyglot file is a single file that is valid and interpretable as two or more different file formats, such as both a PDF and a ZIP archive.} with a zip,JS,PhP,HTML? How do we remove the 2nd format?
3. If it is a valid-single-format pdf, does it have white on white text that could be prompt injection? How do we remove the hidden text but keep the other text?

**Architecture:**
Broken into Header - Content - Reference table - Trailer - EOF

Pdf is a graph of objects (pages, font, images etc). The root of this graph is `/root`
1. **Canonicalization - re-save** a pdf: parser like `qpdf` starts parsing at the root node and follows the entire tree until all the nodes are visited. The nodes that have no links are dropped on resave. So a polyglots second format binary that was just appended after `EOF` is never reached by a parser because nothing points to it. *Resaving the pdf binary drops the non-pdf binary*
   It is also possible to add a hidden layer, zero-point font, white on white font to the pdf. Since these are valid pdf objects and a real nodes in graph, re-saving does not remove them.
![[pdf_structure.png]]

2. **Rasterization: text → Img**: 
   The visible text vectors are converted to image pixels.
   Explanation: Every letter in a font family is not stored as a picture. It is stored as a Mathematical formula of connect the dots for each letter. The font file for "Helvetica Bold" has one of these outlines for every letter, and that outline can be scaled up or down to any size without losing its shape. Pdf’s structure has access to the font library from system and the letters in the body. Since a pdf stores vector definitions of fonts, images, tables, it paints every visual element the same way over to a new canvas.
   *Converting text to images hides the white on white zero-point font attacks.* What this does not hide is an image overlay of legible black-on-white text. The black-on-white font will be converted as an image.
3. **Multimodal/VLM content-safety guardrail:** 
   Use Nvidia Nemo Guardrails: [Input safety with Vision model](https://docs.nvidia.com/nemo/guardrails/get-started/tutorials/multimodal)
   This only accepts images. So we have to rasterize each PDF page to an image first.