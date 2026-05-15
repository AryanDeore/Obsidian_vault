https://medium.com/data-science-at-microsoft/redacting-sensitive-text-from-dicom-medical-images-in-python-

```
Built a PHI redaction pipeline for GDPR compliance (CRAFT → Tesseract → spaCy NER) achieving 100% redaction on 94% of DICOM images, with confidence-based logging and iterative NER retraining.
```

## Goal:
Partner hospitals send DICOM chest CT scans that arrive pre-tagged with clinical metadata (age, sex, COVID status, demographics). Before these images can enter the data lake, patient-identifying information must be stripped for GDPR/HIPAA compliance. That's what this pipeline does.

## Architecture:
1. DICOM Header Scrubbing (rule-based, no ML):
   PHI DICOM metadata is stripped using pydicom.
2. **Text Localization**: Burned-In Text Detection — CRAFT
   Finds _where_ text exists in the image and outputs bounding box coordinates.
   Why CRAFT over Tesseract: Tesseract's detector was designed for scanned documents (high contrast, clean backgrounds), while CT scans have faint gray text on complex anatomical backgrounds.
3. **Text Recognition** — Tesseract (OCR)
   You crop each bounding box CRAFT found and feed it to Tesseract, which reads the pixels and returns a string: "John Smith," "MRN: 12345678," "120 kV," etc.
4. **PHI Classification** — Fuzzy Matching + spaCy NER
	1. **Primary: Fuzzy string matching** — compare each OCR'd string against the known PHI values from the DICOM header (patient name, MRN, DOB) using thefuzz.
	2. **Fallback: spaCy NER** — only invoked when fuzzy matching fails (e.g., OCR misread "John Smith" as "Jchn Smlth" badly enough that fuzzy matching drops below threshold, or the burned-in text contains PHI not in the header like a referring physician name)
5. **Pixel-Level Redaction**:
   
   