# Chen — Search Log

קובץ זה מתעד כל חיפוש רשת שחן ביצעה. לפני כל חיפוש חדש, חן `Grep`s את הקובץ הזה כדי לבדוק אם כבר חיפשה משהו דומה ב-30 הימים האחרונים. אחרי כל חיפוש מוצלח, חן מוסיפה entry בפורמט שלמטה.

This file tracks every web research action Chen has performed. Before each new search, Chen `Grep`s this file for prior work on the same keywords (30-day dedup window). After each successful search, Chen appends an entry in the format below.

**Format per entry:**

```
## YYYY-MM-DD HH:MM | <topic>
**מילות מפתח:** keyword1, keyword2
**שאילתות שנעשו:** "query 1", "query 2"
**מקורות שנמצאו:**
- [title](URL) - איכות: ⭐⭐⭐⭐ - <note>
- [title](URL) - איכות: ⭐⭐⭐ - <note>
**נבחר:** <chosen source and why>
**קובץ ב-Content:** <filename>.md
---
```

---

<!-- Search entries appended below by Chen. -->
