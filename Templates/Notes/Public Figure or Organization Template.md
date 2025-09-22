---
tags: 
links:
---
## Overview
John Doe is a renowned technologist and entrepreneur known for his contributions to artificial intelligence.

## Quotes
```dataviewjs
const slugify = str =>
  str.toLowerCase()
     .replace(/[\s]+/g, "-")
     .replace(/[,:()\[\]]/g, "")
     .replace(/[^a-z0-9\-]/g, "")
     .replace(/-+/g, "-")
     .replace(/^-|-$/g, "");

const currentPath = dv.current().file.path;
const currentSlug = slugify(dv.current().file.name);

const results = [];

for (let page of dv.pages("#note/progress-update").sort(p => p.file.name, 'desc')) {
  const outlinks = page.file.outlinks ?? [];
  if (!outlinks.some(link => link.path === currentPath)) continue;

  const content = await dv.io.load(page.file.path);
  const lines = content.split("\n");

  const blockRegex = /^\s*>?\s*\^note-quote-[\w-]+/;

  for (let rawLine of lines) {
    const line = rawLine.trim().replace(/^>\s*/, "");
    if (blockRegex.test(line) && line.includes(currentSlug)) {
      const blockId = line.replace(/^\^/, "").trim();
      results.push({ file: page.file.path, id: blockId });
    }
  }
}

for (let r of results) {
  dv.paragraph(`📄 From: ${dv.fileLink(r.file)}`);
  dv.paragraph(`![[${r.file}#^${r.id}]]`);
}
```

## [Insert] Career 

## Notes

