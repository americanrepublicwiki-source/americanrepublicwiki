%%
Copyright (c) 2025 American Republic Wiki
%%
```dataviewjs
const here = dv.current().file.folder;                // e.g. "test"
const updatesFolder = `${here}/_Timelines`;     

// Do NOT use a string query; filter by exact folder
let pages = dv.pages()
  .where(p => p.file.path.startsWith(updatesFolder) && p.file.ext == "md");

if (pages.length === 0) {
  dv.paragraph("_No terms found._  (Tip: run **Dataview: Force Refresh** after creating files via Templater.)");
} else {
  // Table with full note embedded
  for (let r of pages) {
	  dv.paragraph(`📄 From: ${dv.fileLink(r.file.path)}`);
	  dv.paragraph(`![[${r.file.path}]]`);
	}
}
```
