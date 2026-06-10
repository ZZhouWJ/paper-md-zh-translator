# paper-md-zh-translator

`paper-md-zh-translator` is a Codex skill for translating extractable-text English life-science papers into searchable Simplified Chinese PDF deliverables.

它适合处理 biology、biomedical、bioinformatics、computational biology、protein engineering、genomics、omics 等生命科学论文 PDF。该 skill 走快速 Markdown 重排路线：正文翻译为中文，图表和展示公式从原 PDF 小区域裁剪后插入，References 和 back matter 保持英文追加。

## What It Produces

For each input PDF, the skill creates one paper-specific output folder and leaves exactly three final PDFs:

```text
<pdf-basename>/
  <pdf-basename>.pdf
  <pdf-basename>_zh.pdf
  <pdf-basename>_summary.pdf
```

- `<pdf-basename>.pdf`: original PDF copied unchanged.
- `<pdf-basename>_zh.pdf`: searchable, reflowed Simplified Chinese paper PDF.
- `<pdf-basename>_summary.pdf`: searchable Simplified Chinese technical summary PDF.

Temporary Markdown files, cropped assets, extraction JSON, previews, and HTML files are removed after validation.

## Important Limits

This skill is designed for speed and searchable text. It does not preserve the original PDF page layout.

It only supports PDFs with extractable text. It should stop instead of translating when:

- the PDF is scanned or image-only;
- most pages have little or no extractable text;
- text extraction order is badly broken;
- figures, tables, or display equations cannot be cropped without using very large page regions.

The skill does not translate:

- References or Bibliography;
- acknowledgements, funding, data availability, author contributions, competing interests, supplementary information, and similar back matter;
- text inside figure/table/equation images.

## Requirements

The bundled scripts use Python and PDF/image libraries:

```bash
pip install pypdf pillow reportlab
```

For figure/table/equation cropping, install Poppler so `pdftoppm` is available:

```bash
brew install poppler
```

Codex Desktop may already provide these dependencies through its bundled runtime. Other agents or local environments usually need the packages installed manually.

## Installation

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/ZZhouWJ/paper-md-zh-translator.git ~/.codex/skills/paper-md-zh-translator
```

Restart Codex or reload skills if your environment requires it.

## Usage

In Codex, mention the skill and provide an English paper PDF:

```text
使用 paper-md-zh-translator 处理 /path/to/DeepCodon.pdf
```

or:

```text
[$paper-md-zh-translator] Translate this English biology PDF into Chinese deliverables: /path/to/paper.pdf
```

The expected final response should report only the three final PDF paths.

## Summary Format

For research articles, the summary PDF uses exactly these top-level sections:

1. 输入输出
2. 训练 / 验证 / 测试数据集
3. 损失函数
4. 训练目标

For reviews, surveys, and perspectives, it uses:

1. 研究领域划分
2. 代表性方法对比与演进
3. 数据资源与评价标准
4. 当前痛点与挑战
5. 未来趋势

## Repository Layout

```text
paper-md-zh-translator/
  SKILL.md
  agents/
    openai.yaml
  references/
    summary-templates.md
    translation-rules.md
  scripts/
    extract_pdf_structure.py
    crop_region.py
    markdown_to_black_white_html.py
    render_black_white_markdown_pdf.py
    validate_outputs.py
    finalize_outputs.py
```

## License

See `LICENSE`.
