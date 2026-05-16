# PDF Sign

Overlay a visual signature image onto a PDF file.

## Usage

```
/pdf-sign <path-to-pdf> [signature-image]
```

- `<path-to-pdf>` — required. Path to the PDF to sign.
- `[signature-image]` — optional. Path to the signature image (PNG with transparent background works best). If omitted, look for `signature.png` in the same folder as the PDF, then in the current directory.

## Steps

1. **Parse arguments** from `$ARGUMENTS`. Extract the PDF path and optional signature image path.

2. **Locate the signature image.** If not provided, check:
   - Same directory as the PDF: `signature.png`
   - Current working directory: `signature.png`
   If still not found, ask the user to provide a path to their signature image.

3. **Check Python is available.**
   ```bash
   python --version 2>&1
   ```
   If the command is not found or returns an error, stop immediately and tell the user:
   > "This skill requires Python. Please install it from https://python.org and make sure it's on your PATH, then try again."
   Do not proceed further.

4. **Check dependencies.** Verify `pymupdf` is installed:
   ```bash
   python -c "import fitz" 2>&1
   ```
   If missing, install it:
   ```bash
   pip install pymupdf
   ```

5. **Ask the user for placement details** (unless they already specified):
   - Which page? (default: last page)
   - Position: bottom-right, bottom-left, bottom-center, or custom x/y coordinates
   - Size: small (25% page width), medium (35%), large (50%), or custom width in points

6. **Generate and run a Python script** to apply the signature overlay. Use this pattern:

```python
import fitz  # pymupdf

pdf_path = "<resolved-pdf-path>"
sig_path = "<resolved-signature-path>"
output_path = pdf_path.replace(".pdf", "_signed.pdf")

# Placement config (fill in from user input)
page_index = -1        # -1 = last page
position = "bottom-right"
sig_width_fraction = 0.35  # fraction of page width

doc = fitz.open(pdf_path)
page = doc[page_index]
pw, ph = page.rect.width, page.rect.height

sig_w = pw * sig_width_fraction
img_rect_tmp = fitz.Rect(0, 0, 1, 1)
sig_img = fitz.open(sig_path)
sig_natural = sig_img[0].rect if sig_img.is_pdf else fitz.Rect(0, 0, *fitz.open(sig_path).get_page_pixmap(0).size) if False else None

# Compute natural aspect ratio from the image
pix = fitz.open(sig_path)
# For image files, open as document to get dimensions
img_doc = fitz.open(sig_path)
w0, h0 = img_doc[0].rect.width, img_doc[0].rect.height
img_doc.close()
sig_h = sig_w * (h0 / w0)

margin = 20  # points from edge

if position == "bottom-right":
    x0, y0 = pw - sig_w - margin, ph - sig_h - margin
elif position == "bottom-left":
    x0, y0 = margin, ph - sig_h - margin
elif position == "bottom-center":
    x0, y0 = (pw - sig_w) / 2, ph - sig_h - margin
else:
    x0, y0 = 50, 50  # custom fallback

rect = fitz.Rect(x0, y0, x0 + sig_w, y0 + sig_h)
page.insert_image(rect, filename=sig_path)

doc.save(output_path)
doc.close()
print(f"Saved: {output_path}")
```

   Adapt the script based on the user's actual choices before running it.

7. **Report the result.** Tell the user the output file path. If the output looks wrong (e.g. the image has wrong aspect ratio or unexpected placement), offer to re-run with adjusted parameters.

## Notes

- Output is always saved as `<original-name>_signed.pdf` alongside the original, never overwriting it.
- PNG with a transparent background produces the cleanest result; JPEG will have a white box.
- For multi-page signing, ask the user whether to sign all pages or just specific ones, then loop over the relevant page indices.
