<%*
const generated_note_dataview = 
"\n" + 
"```dataviewjs\n" +
"const slugify = str =>\n" +
"  str.toLowerCase()\n" +
"     .replace(/[\\s]+/g, \"-\")\n" +
"     .replace(/[,:()\\[\\]]/g, \"\")\n" +
"     .replace(/[^a-z0-9\\-]/g, \"\")\n" +
"     .replace(/-+/g, \"-\")\n" +
"     .replace(/^-|-$/g, \"\");\n\n" +
"const currentPath = dv.current().file.path;\n" +
"const currentSlug = slugify(dv.current().file.name);\n\n" +
"const results = [];\n\n" +
"for (let page of dv.pages(\"#note/progress-update\")) {\n" +
"  const outlinks = page.file.outlinks ?? [];\n" +
"  if (!outlinks.some(link => link.path === currentPath)) continue;\n\n" +
"  const content = await dv.io.load(page.file.path);\n" +
"  const lines = content.split(\"\\n\");\n\n" +
"  const blockRegex = /^\\s*>?\\s*\\^note-progress-update-[\\w-]+/;\n\n" +
"  for (let rawLine of lines) {\n" +
"    const line = rawLine.trim().replace(/^>\\s*/, \"\");\n" +
"    if (blockRegex.test(line) && line.includes(currentSlug)) {\n" +
"      const blockId = line.replace(/^\\^/, \"\").trim();\n" +
"      results.push({ file: page.file.path, id: blockId });\n" +
"    }\n" +
"  }\n" +
"}\n\n" +
"for (let r of results) {\n" +
"  dv.paragraph(`📄 From: ${dv.fileLink(r.file)}`);\n" +
"  dv.paragraph(`![[${r.file}#^${r.id}]]`);\n" +
"}\n" +
"```";


//const dictionaryDirs = ["Dictionary", "Public Notes/Dictionary"];

// Get all notes from both directories and combine them into one list
const allNotes = app.vault.getMarkdownFiles()
//.filter(file =>
//  dictionaryDirs.some(dir => file.path.startsWith(dir))
//);
const definitionOptions = ["New", ...allNotes.map(note => note.basename)]; // Add "New" at the start

// Prompt user to select an existing definition or create a new one
let taskName = await tp.system.suggester(definitionOptions, definitionOptions, false, "Select an existing definition or choose 'New':");
let hasNote = false;

// If "New" is selected, prompt for the definition name
if (taskName === "New") {
  taskName = await tp.system.prompt("Enter the project or task name:");
} else {
  hasNote = true;
}

// Prompt for the definition content
const progressUpdateContent = await tp.system.prompt("Enter the progress update:");

// Check if the note already exists
if (taskName.startsWith("[[") && taskName.endsWith("]]")) {
  const noteTitle = taskName.slice(2, -2); // Strip [[ ]]
  
  const possiblePaths = app.vault.getMarkdownFiles().map(f => f.path);
  const alreadyExists = possiblePaths.some(p => p === `${noteTitle}.md`);

  if (!alreadyExists) {
    const file = await app.vault.create(`${noteTitle}.md`, generated_note_dataview);
    console.log("✅ Created:", file.path);
  } else {
    console.log("⚠️ Skipped, already exists:", `${noteTitle}.md`);
    new Notice(`⚠️ File already exists: ${noteTitle}.md`);
  }
}

if (hasNote) {
   taskName = "[[" + taskName + "]]"
}

const slugify = str =>
  str.toLowerCase()
     .replace(/[\s]+/g, "-")
     .replace(/[,:()\[\]]/g, "")
     .replace(/[^a-z0-9\-]/g, "")
     .replace(/-+/g, "-")
     .replace(/^-|-$/g, "");

const slug = slugify(taskName);
const timestamp = moment().format("YYYYMMDDHHmmss");
const blockId = `note-progress-update-${slug}-${timestamp}`;

// Construct the callout with the definition name and content
-%>
>[!note-progress-update] <% taskName %> #note/progress-update 
> <% progressUpdateContent %><% tp.file.cursor() %>
> ^<% blockId %>
