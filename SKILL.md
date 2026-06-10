---
name: paper-md-zh-translator
description: Translate extractable-text English biology, biomedical, bioinformatics, computational biology, genomics, protein engineering, omics, and life-science paper PDFs into searchable Simplified Chinese Markdown and black-on-white A4 PDF deliverables. Use when the user wants a faster reflowed Chinese paper PDF rather than layout-preserving page-image translation; figures, tables, and display equations are cropped as images, References and back matter stay in English, and a separate structured Chinese summary PDF is produced for research articles or reviews.
---

# Paper Markdown Chinese Translator

## Core Contract

For each extractable-text English paper PDF, create one paper-specific folder and finally deliver exactly three PDFs:

1. The copied original PDF with its filename unchanged.
2. A searchable reflowed Simplified Chinese PDF rendered from Markdown.
3. A searchable Chinese summary PDF rendered from the summary Markdown.

Markdown files, cropped assets, rendered previews, extracted text JSON, layout maps, HTML, and other work files are temporary implementation artifacts. Delete them before the final response, after PDF text extraction checks pass.

This skill is for speed, readability, and searchable text. It does not preserve the original PDF page layout. Do not use whole-page image translation for this skill.

## Output Layout

Create a folder named from the original PDF filename without `.pdf`.

Final folder contents must be:

```text
<pdf-basename>/
  <pdf-basename>.pdf
  <pdf-basename>_zh.pdf
  <pdf-basename>_summary.pdf
```

During work, temporary files may exist:

```text
<pdf-basename>/
  <pdf-basename>.pdf
  <pdf-basename>_zh.md
  <pdf-basename>_zh.pdf
  <pdf-basename>_summary.md
  <pdf-basename>_summary.pdf
  assets/
    figure_001.png
    table_001.png
    equation_001.png
  work/
    extracted_text.json
    layout.json
    figure_table_map.json
    references.txt
    untranslated_back_matter.md
    <pdf-basename>_zh.html
```

These temporary files must be removed by the finalization step.

If a folder already exists, create a sibling with a numeric suffix unless the user explicitly asks to overwrite.

## Stop Conditions

This skill only supports PDFs with extractable text. Run `scripts/extract_pdf_structure.py` first. Stop and tell the user the PDF is not suitable for this fast Markdown workflow if:

- the PDF is scanned/image-only;
- most pages have little or no extracted text;
- text extraction order is visibly broken enough that faithful translation would be unreliable;
- the paper's figures/tables/equations cannot be located well enough without cropping very large page regions.

Do not automatically fall back to the slower layout-preserving image workflow.

## Workflow

1. Copy the original PDF into the paper folder unchanged.
2. Extract and quality-check text:
   ```bash
   python scripts/extract_pdf_structure.py \
     --pdf "<paper-folder>/<basename>.pdf" \
     --out "<paper-folder>/work/extracted_text.json" \
     --references-out "<paper-folder>/work/references.txt" \
     --back-matter-out "<paper-folder>/work/untranslated_back_matter.md"
   ```
3. Read `references/translation-rules.md` before drafting `<basename>_zh.md`.
4. Use `extracted_text.json`, rendered page previews, and visual inspection to identify figures, tables, and display equations. Crop them into `assets/`:
   ```bash
   python scripts/crop_region.py \
     --pdf "<paper-folder>/<basename>.pdf" \
     --page 3 \
     --bbox "72,220,540,610" \
     --out "<paper-folder>/assets/figure_001.png"
   ```
   `--bbox` defaults to PDF points in bottom-left coordinates. Use small padding only. Crops may include the original English caption, but do not crop broad page areas.
5. Translate only the main body into Simplified Chinese. Keep canonical English model, dataset, software, gene/protein, database, metric, DOI, URL, and citation names.
6. Insert cropped figures/tables at the first body location that references them. If no first reference is found, insert near the original section in source order and record the reason in `work/figure_table_map.json`.
7. Write translated captions in Markdown while preserving labels such as `Figure 1` and `Table 1`.
8. Append untranslated English back matter directly after the Chinese body, then append English References/Bibliography.
9. Generate black-on-white HTML for inspection:
   ```bash
   python scripts/markdown_to_black_white_html.py \
     --md "<paper-folder>/<basename>_zh.md" \
     --out "<paper-folder>/work/<basename>_zh.html"
   ```
10. Render searchable PDFs. Prefer a reliable local Markdown/HTML-to-PDF route if available. Otherwise use the bundled ReportLab renderer:
    ```bash
    python scripts/render_black_white_markdown_pdf.py \
      --md "<paper-folder>/<basename>_zh.md" \
      --out "<paper-folder>/<basename>_zh.pdf"
    python scripts/render_black_white_markdown_pdf.py \
      --md "<paper-folder>/<basename>_summary.md" \
      --out "<paper-folder>/<basename>_summary.pdf"
    ```
11. Validate working deliverables and PDF text extraction:
    ```bash
    python scripts/validate_outputs.py \
      --folder "<paper-folder>" \
      --basename "<basename>" \
      --stage working
    ```
12. Finalize outputs. This re-extracts text from both generated PDFs, enforces searchable-text thresholds, deletes all non-final files/directories in the paper folder, and leaves only three PDFs:
    ```bash
    python scripts/finalize_outputs.py \
      --folder "<paper-folder>" \
      --basename "<basename>" \
      --confirm-delete-intermediates
    ```
13. Validate final folder state:
    ```bash
    python scripts/validate_outputs.py \
      --folder "<paper-folder>" \
      --basename "<basename>" \
      --stage final
    ```

## Translation Scope

Translate:

- title, abstract, keywords;
- main body sections such as Introduction, Background, Methods, Results, Discussion, Conclusion;
- figure/table captions, preserving `Figure 1` / `Table 1` labels;
- body text that references figures, tables, equations, and citations.

Do not translate:

- References or Bibliography;
- Acknowledgements/Acknowledgments;
- Funding;
- Data availability, Code availability, Materials availability;
- Author contributions;
- Competing interests, Conflict of interest, Ethics declarations;
- Supplementary information;
- text inside figure/table/equation images.

Append untranslated non-body material as English text in the same Markdown/PDF after the Chinese body.

## Figure, Table, and Equation Rules

- Save figures as `assets/figure_001.png`, `assets/figure_002.png`, etc.
- Save tables as `assets/table_001.png`, `assets/table_002.png`, etc.
- Save display equations as `assets/equation_001.png`, `assets/equation_002.png`, etc.
- Do not translate inside cropped images.
- Inline formulas inside a paragraph remain as original symbol text.
- Display equations written on their own line are cropped as images and inserted back into the Markdown.
- Crops may include the original English caption, but Markdown must still contain the translated Chinese caption.

## Style Requirements

Both `<basename>_zh.pdf` and `<basename>_summary.pdf` must be:

- A4;
- single-column;
- white background;
- black text only;
- no colored headings, backgrounds, borders, highlights, or decorative elements;
- simple academic report style;
- searchable and copyable wherever the content is text.

Figure/table/equation images may retain their original colors because they are source paper content. The surrounding document style must remain black on white.

## Summary

Read `references/summary-templates.md` before drafting `<basename>_summary.md`.

Research articles must use exactly these top-level sections:

1. 输入输出
2. 训练 / 验证 / 测试数据集
3. 损失函数
4. 训练目标

Reviews, surveys, and perspectives must use exactly these top-level sections:

1. 研究领域划分
2. 代表性方法对比与演进
3. 数据资源与评价标准
4. 当前痛点与挑战
5. 未来趋势

The summary PDF follows the same black-on-white searchable PDF requirements.

## Final Checks

Before finishing:

- Confirm `validate_outputs.py --stage working` reports `ok: true` before cleanup.
- Confirm `finalize_outputs.py` reports `ok: true` and shows extracted text character counts for `<basename>_zh.pdf` and `<basename>_summary.pdf`.
- Confirm `validate_outputs.py --stage final` reports `ok: true`.
- Confirm the final folder contains only `<basename>.pdf`, `<basename>_zh.pdf`, and `<basename>_summary.pdf`.
- Report only the three final PDF paths. Do not report temporary Markdown, assets, work directories, or deleted intermediate paths.
