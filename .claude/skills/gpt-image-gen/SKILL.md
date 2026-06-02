---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt. השתמש כשצריך לייצר/לרנדר תמונה מטקסט. מודל gpt-image-2.
---

# gpt-image-gen — יצירת תמונות דרך OpenAI Images API

סקיל מעטפת לשליחת prompt ל-OpenAI Images API וקבלת קובץ PNG.
האחריות של הסקיל: לשלוח את ה-prompt, לקבל את התמונה, לשמור אותה לדיסק ולאמת.

## ⚠️ שם המודל — אל תשנה

המודל הוא **`gpt-image-2`**.

- `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI, שיצא ב-**21 באפריל 2026**.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אל תציע אלטרנטיבות** (`dall-e-3`, `gpt-image-1` וכו'). השתמש בדיוק במודל הזה.
- אם מתקבלת שגיאה מה-API, הבעיה היא כמעט תמיד ב-**API key** או ב-**parameters** — **לא** בשם המודל. אל "תתקן" את שם המודל.

## דרישות

- משתנה הסביבה `OPENAI_API_KEY` חייב להיות מוגדר ב-`.env` שבשורש הפרויקט.

טען את המפתח מ-`.env` לפני הקריאה:

```bash
# טעינת משתני סביבה מ-.env
set -a
source .env
set +a
```

(אם `source .env` לא עובד בסביבה מסוימת, חלופה:
`export $(grep -v '^#' .env | grep -v '^$' | xargs)`)

## הקריאה ל-API

Endpoint: `https://api.openai.com/v1/images/generations`

### מסלול A — עם jq

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

### מסלול B — Python fallback (כש-jq לא מותקן, למשל ב-Git Bash)

שמור קודם את התשובה לקובץ JSON זמני, ואז פענח עם Python:

```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > resp.json

python -c "import json, base64; d = json.load(open('resp.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"

rm -f resp.json
```

(אם `python` לא קיים, נסה `python3`.)

## אימות

לאחר היצירה, ודא שהקובץ נוצר ואינו ריק:

```bash
test -s "<output-path>.png" && echo "OK: $(wc -c < "<output-path>.png") bytes" || echo "FAILED: file missing or empty"
```

אם הקובץ ריק או חסר — בדוק את תוכן `resp.json` לאיתור שגיאת API (מפתח/parameters), ולא את שם המודל.

## פרמטרים נתמכים

| פרמטר | ערך ברירת מחדל | הערה |
|--------|----------------|------|
| `model` | `gpt-image-2` | **קבוע — אל תשנה** |
| `prompt` | — | תיאור התמונה |
| `size` | `1024x1024` | |
| `quality` | `medium` | |
| `output_format` | `png` | |
