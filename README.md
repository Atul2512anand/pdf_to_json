Below is an optimized GitHub‑ready README tailored for running the converter in Google Colab and for the NCERT→JSON NLP workflow , including clear setup, usage, schema, and troubleshooting guidance for reliable downloads in Colab environments. It follows common README best practices so visitors quickly understand what the project does, how to run it, and why it exists for educational PDF datasets and web/NLP pipelines.​

PDF to JSON Converter (Colab)
A browser‑based workflow that converts one or more PDFs into clean per‑page JSON files, packaged as a single ZIP for download, designed for preparing NCERT documents for NLP and web development tasks at iGyan AI. The notebook uses Colab’s upload/download helpers and PyMuPDF text extraction to create a consistent schema that downstream systems can parse, chunk, and index reliably.​

Features
Multiple file upload directly in the notebook with zero local setup, suitable for non‑engineers and batch processing in a shared Colab runtime.​

Per‑page text extraction via PyMuPDF with an optional OCR fallback for scanned PDFs to ensure coverage across NCERT materials and other educational documents.​

One JSON file per input PDF with a consistent schema, bundled into a ZIP for a one‑click browser download from Colab.​

Why we use this
NCERT PDFs need conversion into structured JSON to train and evaluate NLP components for search, Q&A, chunking, embeddings, and web content delivery in a reproducible pipeline.​

A standardized JSON schema minimizes custom parsing across heterogeneous layouts, speeding dataset preparation and improving quality for educational content ingestion at scale.​

JSON schema
Each output JSON contains file_name, page_count, and pages, where pages is an array of objects with page_number and text suitable for rendering, chunking, and indexing in web apps and NLP stacks.​

An example produced by this notebook for an NCERT‑style PDF confirms sequential page numbering and complete page text aligned with this schema, demonstrating readiness for ingestion and training.​

json
{
  "file_name": "string",
  "page_count": 0,
  "pages": [
    { "page_number": 1, "text": "..." }
  ]
}
Quick start (Google Colab)
Open a new Colab notebook and run the following cells in order: install dependencies, define converters, upload and convert, then download the ZIP artifact to your machine for use in web or NLP pipelines.​

Install dependencies:

python
!pip -q install pymupdf pillow pytesseract
!apt-get -y install tesseract-ocr  # needed only if OCR fallback is enabled
Define converters:

python
import io, json, zipfile, pathlib
import fitz  # PyMuPDF
from google.colab import files

OCR_ENABLED = True  # set False to disable OCR fallback

if OCR_ENABLED:
    from PIL import Image
    import pytesseract

def page_text_with_optional_ocr(page):
    txt = page.get_text("text") or ""
    if txt.strip():
        return txt.strip()
    if not OCR_ENABLED:
        return txt.strip()
    pix = page.get_pixmap(dpi=200)
    if pix.alpha:
        pix = fitz.Pixmap(fitz.csRGB, pix)
    img = Image.frombytes("RGB", [pix.width, pix.height], pix.samples)
    return (pytesseract.image_to_string(img) or "").strip()

def pdf_to_json_bytes(filename: str, file_bytes: bytes) -> bytes:
    doc = fitz.open(stream=file_bytes, filetype="pdf")
    pages = []
    for i, page in enumerate(doc, start=1):
        text = page_text_with_optional_ocr(page)
        pages.append({"page_number": i, "text": text})
    data = {"file_name": filename, "page_count": len(doc), "pages": pages}
    return json.dumps(data, ensure_ascii=False, indent=2).encode("utf-8")
PyMuPDF’s page.get_text provides readable per‑page text for most digital PDFs, while the OCR branch handles scanned pages that lack embedded text in many educational sources.​

Upload, convert, and ZIP:

python
uploaded = files.upload()  # choose multiple PDFs in the browser

zip_path = "converted_jsons.zip"
with zipfile.ZipFile(zip_path, mode="w", compression=zipfile.ZIP_DEFLATED) as zf:
    for name, content in uploaded.items():
        stem = pathlib.Path(name).stem
        try:
            json_bytes = pdf_to_json_bytes(name, content)
            zf.writestr(f"{stem}.json", json_bytes)
        except Exception as e:
            zf.writestr(f"{stem}__ERROR.txt", str(e))

print("Wrote:", zip_path)
Creating the ZIP on disk ensures a reliable artifact for the next step in Colab’s environment, especially for larger batches of PDFs.​

Download the ZIP:

python
from google.colab import files
files.download("converted_jsons.zip")
Trigger the download in its own short cell to avoid timing issues where long conversions and downloads combined in one cell can suppress the prompt in some sessions.​

NCERT and layout notes
If reading order appears jumbled in highly formatted chapters, switch to block or word extraction and sort by coordinates as described in PyMuPDF text‑extraction guidance to improve structure for downstream NLP.​

Keep OCR enabled for scanned PDFs, understanding it increases runtime but turns otherwise unsearchable pages into text suitable for training and evaluation datasets.​

Troubleshooting
If the browser download prompt does not appear, confirm the ZIP exists and size looks reasonable, then retry the short download cell or use the Files sidebar to right‑click and download from /content.​

For large archives or flaky prompts, copy the ZIP to your Drive from the notebook and fetch it from Drive later, which is often most reliable for big outputs and longer sessions.​



Acknowledgments
Google Colab for a low‑friction, browser‑based runtime that simplifies batch conversions and artifact downloads for non‑specialists and engineers alike.​

PyMuPDF for fast and flexible text extraction modes that support both simple plain‑text needs and structured extraction for complex page layouts over time.​

