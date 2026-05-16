# PDF Sign

Overlay a visual signature image onto a PDF file. The signature image is stored as a skill asset alongside this skill file and reused across all future invocations.

## Usage

```
/pdf-sign <path-to-pdf>
/pdf-sign --update-signature
```

- `<path-to-pdf>` — sign this PDF using the stored signature asset.
- `--update-signature` — replace the stored signature asset with a new image (no PDF required).

## Skill asset

The signature image is stored at:
- **Windows:** `%USERPROFILE%\.claude\commands\pdf-sign.signature.png`
- **macOS/Linux:** `~/.claude/commands/pdf-sign.signature.png`

This location is always used — the skill never looks in the PDF directory or the working directory.

## Steps

1. **Determine the skill asset path.**
   - On Windows: `$env:USERPROFILE\.claude\commands\pdf-sign.signature.png`
   - On macOS/Linux: `~/.claude/commands/pdf-sign.signature.png`

2. **Handle `--update-signature` flag.** If `$ARGUMENTS` is `--update-signature`, skip to step 8 (signature registration) instead of signing a PDF.

3. **Parse the PDF path** from `$ARGUMENTS`.

4. **Check for stored signature asset.**
   Check whether the skill asset file exists. If it does not exist, tell the user:
   > "No signature found. Please provide a path to your signature image (PNG with transparent background works best) and I'll store it for future use."
   Then go to step 8 to register it before continuing.

5. **Check Python is available.**
   ```bash
   python --version 2>&1
   ```
   If not found, stop and tell the user:
   > "This skill requires Python. Please install it from https://python.org and ensure it is on your PATH, then try again."

6. **Check dependencies.** Verify `pymupdf` is installed:
   ```bash
   python -c "import fitz" 2>&1
   ```
   If missing, ask: "pymupdf is not installed. Can I install it now with `pip install pymupdf`?"
   Only install if the user confirms. If they decline, stop and tell them to install it manually.

7. **Ask the user for placement details:**
   - Which page? (default: last page)
   - Position: bottom-right (default), bottom-left, bottom-center, or custom x/y coordinates
   - Size: small (25% page width), medium (35%, default), large (50%), or custom width in points

   Then generate and run the following Python script, adapted to the user's choices:

```python
import fitz  # pymupdf
import os

pdf_path = "<resolved-pdf-path>"
sig_path = "<skill-asset-path>"
output_path = pdf_path.replace(".pdf", "_signed.pdf")

page_index = -1        # -1 = last page
position = "bottom-right"
sig_width_fraction = 0.35

doc = fitz.open(pdf_path)
page = doc[page_index]
pw, ph = page.rect.width, page.rect.height

img_doc = fitz.open(sig_path)
w0, h0 = img_doc[0].rect.width, img_doc[0].rect.height
img_doc.close()

sig_w = pw * sig_width_fraction
sig_h = sig_w * (h0 / w0)
margin = 20

if position == "bottom-right":
    x0, y0 = pw - sig_w - margin, ph - sig_h - margin
elif position == "bottom-left":
    x0, y0 = margin, ph - sig_h - margin
elif position == "bottom-center":
    x0, y0 = (pw - sig_w) / 2, ph - sig_h - margin
else:
    x0, y0 = 50, 50

rect = fitz.Rect(x0, y0, x0 + sig_w, y0 + sig_h)
page.insert_image(rect, filename=sig_path)

doc.save(output_path)
doc.close()
print(f"Saved: {output_path}")
```

   Report the output file path when done. If placement looks wrong, offer to re-run with adjusted parameters.

8. **Signature registration / replacement.**
   Ask the user to provide the path to their signature image file. Then copy it to the skill asset path, overwriting any existing file:
   ```bash
   # Windows
   copy "<provided-path>" "$env:USERPROFILE\.claude\commands\pdf-sign.signature.png"
   # macOS/Linux
   cp "<provided-path>" "~/.claude/commands/pdf-sign.signature.png"
   ```
   Confirm to the user: "Signature saved. It will be used for all future `/pdf-sign` calls."
   If this was triggered by a missing signature during a sign request (step 4), continue to step 5 after saving.

## Notes

- Output is always saved as `<original-name>_signed.pdf` alongside the original, never overwriting it.
- PNG with a transparent background produces the cleanest result; JPEG will include a white box.
- For multi-page signing, ask whether to sign all pages or specific ones, then loop over the relevant page indices.
