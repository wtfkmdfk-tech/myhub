---
name: thesis-workflow
description: >
  Records known issues, error patterns, and verified solutions for the WIT graduation thesis project
  (Spark + Kafka + PyTorch real-time intrusion detection system, python-docx generation).
  Triggered when the user mentions "build", "docx", "error", "fix", "问题", or encounters build failures.
---

# Thesis Project Workflow — Problems & Solutions

## 1. python-docx Chinese filename

**Problem:** `PermissionError` when saving .docx with Chinese filename directly.

**Solution:** Use ASCII-only filename in `doc.save()`, then rename with `os.rename()`. Or use a short Latin filename consistently (e.g., `thesis.docx`).

**Why:** python-docx uses `ZipFile` internally, which can conflict with Windows codepage handling of Unicode filenames.

---

## 2. Build script must be a .py file

**Problem:** Chinese text with single quotes (e.g., `HotCloud'10`) breaks inline bash heredoc scripts.

**Solution:** Always write the build script as a standalone `.py` file, then run with `python build_thesis.py`. Never use inline `python -c "..."` or heredoc with Chinese content for complex scripts.

---

## 3. GitHub network is unreliable on this machine

**Problem:** `raw.githubusercontent.com` and `github.com` frequently fail with `Recv failure: Connection was reset` or exit code 28 (timeout).

**Solution:** Use `cdn.jsdelivr.net/gh/...` as the primary CDN for downloading from GitHub repos. Fallback pattern: `https://cdn.jsdelivr.net/gh/{owner}/{repo}/{path}`

**Why:** jsDelivr CDN is accessible even when GitHub direct access is blocked or unstable on this network.

---

## 4. Python command is `python` not `python3`

**Problem:** `python3` command not found.

**Solution:** Always use `python` on this Windows system. For checking Python: `python --version`.

---

## 5. GBK encoding in terminal

**Problem:** Printing Chinese text to terminal causes `UnicodeEncodeError` with 'gbk' codec.

**Solution:** At the start of build scripts, add:
```python
import sys
sys.stdout.reconfigure(encoding='utf-8')
```

Or capture output to a file and read with utf-8 encoding.

---

## 6. f-string backslash SyntaxError (Python 3.11)

**Problem:** Python 3.11+ does not allow backslashes inside f-string `{}` expressions.

```python
# This fails:
run = f'\w:fldChar \w:fldCharType="begin"'
```

**Solution:** Extract the regex pattern or XML string to a variable before the f-string:
```python
tag = 'w:fldChar'
run = f'<{tag} ...'
```

---

## 7. Word TOC field not generating

**Problem:** The TOC field in the generated .docx does not update when right-clicking in Word.

**Solution:** Construct the TOC field code with proper XML elements in this exact order:
1. `<w:fldChar w:fldCharType="begin"/>`
2. `<w:instrText xml:space="preserve"> TOC \\o "1-3" \\h \\z \\u \\* MERGEFORMAT </w:instrText>`
3. `<w:fldChar w:fldCharType="separate"/>`
4. Display text (optional, shown if fields are not updated)
5. `<w:fldChar w:fldCharType="end"/>`

Key: add `\\* MERGEFORMAT` switch, use proper escaping of backslashes in `instrText`. Tell user to press Ctrl+A then F9 to update.

---

## 8. Duplicate table captions in docx

**Problem:** Table captions appear twice — once from the bold-line handler (`**表5.1 ...**`) and once from the table parser's auto-detection of preceding lines.

**Solution:** Only ONE mechanism should handle captions. Preferred approach: disable caption look-back in the table parser (`make_table(doc, ..., caption=None)`), and rely entirely on the bold-line handler for all captions. Ensure ALL tables in markdown have explicit `**表X-X ...**` lines before them.

---

## 9. PermissionError when overwriting .docx

**Problem:** `PermissionError: [Errno 13]` — the previous .docx is locked by another process (e.g., Windows Explorer, Word, or a previous session).

**Solution:** Save to a new filename with incremented version number (e.g., `毕业论文v2.docx`, `毕业论文v3.docx`). Do NOT try to `rm` or overwrite the locked file.

---

## 10. Chinese quotation marks in Python strings

**Problem:** Chinese text containing `"` and `"` (full-width quotes) causes `SyntaxError` because Python interprets them as string delimiters.

**Solution:** Use single quotes `'...'` for strings that contain Chinese double quotation marks `"..."`.

---

## 11. Matplotlib requires numpy (not available)

**Problem:** `import matplotlib` fails with `ModuleNotFoundError: No module named 'numpy'`.

**Solution:** Instead of using matplotlib for diagrams, create visual architecture diagrams using python-docx tables with colored cells (`set_cell_shading`). This avoids external dependencies and produces Word-native output.

---

## 12. RGBColor hex string error

**Problem:** `RGBColor(*color)` where `color = "2F5496"` fails because unpacking a string gives individual characters.

**Solution:** Use `RGBColor.from_string(color_hex)` to create RGBColor from a hex string like `"2F5496"`.

---

## 13. Heading formatting — avoid chained calls

**Problem:** `doc.add_heading(level=1).add_run("text").font.size = Pt(14)` creates two overlapping runs.

**Solution:** Use separate variable assignment:
```python
h = doc.add_heading(level=1)
h.alignment = WD_ALIGN_PARAGRAPH.CENTER
r = h.add_run("标题")
r.font.size = Pt(14)
r.font.name = "黑体"
r.bold = True
r._element.rPr.rFonts.set(qn("w:eastAsia"), "黑体")
```

---

## 14. Markdown table parsing — column count mismatch

**Problem:** Tables with merged cells (e.g., `| **合计** | | | **~5900** |`) produce empty cell strings that get filtered out by `if c.strip()`, causing column count mismatch.

**Solution:** In `parse_table_from_md`, use `split('|')` and keep all cells including empty ones:
```python
cells = [c.strip() for c in ln.split('|')][1:-1]  # strip outer empty from leading/trailing |
```
But preserve standard filtering for normal data tables. Handle both cases with `rows_consistent_check`.

---

## 15. Build checklist (always verify after build)

After generating the .docx, always verify:
1. **Total headings** match expected count (H1/H2/H3)
2. **Total tables** match expected count (data tables + diagram)
3. **Italic runs = 0** (no italic formatting)
4. **No duplicate captions** (each bold + 表/图 line appears exactly once)
5. **Chinese abstract** is present
6. **English abstract** is present
7. **Heading sizes**: H1=14pt, H2=12pt, H3=12pt
8. **H1 font**: 黑体; **H2/H3 font**: 宋体

Use `python -c "from docx import Document; doc = Document('file.docx'); ..."` for all checks.
