# pdf-sign examples

## Basic usage

Sign the last page, signature.png in same folder:
```
/pdf-sign contract.pdf
```

## Explicit signature image

```
/pdf-sign ~/documents/invoice.pdf ~/images/my-sig.png
```

## Expected output

- Input:  `contract.pdf`
- Output: `contract_signed.pdf` (same directory)
- Placement: bottom-right of last page, 35% page width
