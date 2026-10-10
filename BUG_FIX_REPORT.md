# Bug Fix Report

## Bugs Identified + Fixes Applied

### Bug 1) Node — Category filter inverted
**What the bug was**: Selecting a category from the dropdown (e.g. “Science”) showed every book except the chosen category.

**How I fixed it**: Replaced the inverted condition `category != ?` with `category = ?`.

- File: [bookRepository.js](file:///c:/Users/91892/OneDrive/Desktop/technocart/CodeTroubleshoot/nodejs/src/repositories/bookRepository.js#L10-L13)

```js
// Bug
conditions.push("category != ?");

// Fix
conditions.push("category = ?");
```

---

### Bug 2) Node — Stock decrease was increasing the quantity
**What the bug was**: Clicking the `-` button (or sending any negative adjustment like `-1` / `-2`) made the quantity go up instead of down.

**How I fixed it**: Removed the broken conditional branch that flipped negatives with `Math.abs`. Now it uses plain `quantity + delta`, which correctly handles both positive and negative adjustments.

- File: [stockService.js](file:///c:/Users/91892/OneDrive/Desktop/technocart/CodeTroubleshoot/nodejs/src/services/stockService.js#L14-L21)

```js
// Bug
let next = book.quantity + delta;
if (delta < 0) {
  next = book.quantity + Math.abs(delta);
}

// Fix
const next = book.quantity + delta;
```

---

### Bug 3) Node — Edit Book always failed with “Published year is out of range”
**What the bug was**: Saving any edit on the Edit Book dialog returned `Published year is out of range`, even when the year was valid. The actual problem was a swapped argument order in the update call (price was being validated as year).

**How I fixed it**: Corrected the argument order in `updateBook(...)` so `published_year` and `price` are passed in the same order the service expects.

- File: [bookController.js](file:///c:/Users/91892/OneDrive/Desktop/technocart/CodeTroubleshoot/nodejs/src/controllers/bookController.js#L47-L59)

```js
// Bug (price/published_year swapped)
bookService.updateBook(id, title, author, isbn, category,
  body.price, body.published_year, body.quantity
);

// Fix
bookService.updateBook(id, title, author, isbn, category,
  body.published_year, body.price, body.quantity
);
```

---

### Bug 4) Node — Dashboard `available_copies` ignored books with quantity exactly 1
**What the bug was**: `/api/dashboard/stats` under-counted `available_copies`. Any book with `quantity = 1` contributed `0` to the total because the SQL only counted rows with `quantity > 1`.

**How I fixed it**: Changed the threshold from `quantity > 1` to `quantity > 0` so every positive stock count is included in available copies.

- File: [statsRepository.js](file:///c:/Users/91892/OneDrive/Desktop/technocart/CodeTroubleshoot/nodejs/src/repositories/statsRepository.js#L5-L9)

```sql
-- Bug
SUM(CASE WHEN quantity > 1 THEN quantity ELSE 0 END) AS available_copies

-- Fix
SUM(CASE WHEN quantity > 0 THEN quantity ELSE 0 END) AS available_copies
```

---

### Bug 5) Python — Downloaded OCR result text file was always empty
**What the bug was**: After a successful OCR process, hitting `/api/results/{id}/download` returned a 0-byte `.txt` file. The recognized text existed and was returned correctly in the API response — only the download came back blank.

**How I fixed it**: `read_result()` was opening the saved file in `"w+"` mode, which truncates the file immediately on open. I replaced it with `path.read_text()` and added an existence guard that raises `FileNotFoundError` when the file is missing.

- File: [files.py](file:///c:/Users/91892/OneDrive/Desktop/technocart/CodeTroubleshoot/python/app/utils/files.py#L29-L35)

```py
# Bug
with path.open("w+", encoding="utf-8") as handle:
    return handle.read()

# Fix
if not path.is_file():
    raise FileNotFoundError(result_id)
return path.read_text(encoding="utf-8")
```

---

## Files Modified

1. `nodejs/src/repositories/bookRepository.js`
2. `nodejs/src/services/stockService.js`
3. `nodejs/src/controllers/bookController.js`
4. `nodejs/src/repositories/statsRepository.js`
5. `python/app/utils/files.py`
