%%
Copyright (c) 2025 American Republic Wiki
%%
<%*
const generated_note_dataview = 
"\n" + 
"```dataviewjs\n" +
"const slugify = str =>\n" +
"  str.toLowerCase()\n" +
"     .replace(/[\\s_\\/]+/g, \"-\")\n" +        // Replace space, underscore, and slash with dash\n" +
"     .replace(/[,:()\\[\\]]/g, \"\")\n" +       // Remove punctuation\n" +
"     .replace(/[^a-z0-9\\-]/g, \"\")\n" +       // Remove non-alphanumerics except dash\n" +
"     .replace(/-+/g, \"-\")\n" +                // Collapse multiple dashes\n" +
"     .replace(/^-|-$/g, \"\");\n\n" +
"const currentPath = dv.current().file.path;\n" +
"const currentSlug = slugify(currentPath.slice(0,-3));\n\n" +
"const results = [];\n\n" +
"for (let page of dv.pages(\"#note/definition\")) {\n" +
"  const outlinks = page.file.outlinks ?? [];\n" +
"  if (!outlinks.some(link => link.path === currentPath)) continue;\n\n" +
"  const content = await dv.io.load(page.file.path);\n" +
"  const lines = content.split(\"\\n\");\n\n" +
"  const blockRegex = /^\\s*>?\\s*\\^note-definition-[\\w-]+/;\n\n" +
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

let hasNote = false;
let isPicking = true;
let isDir = false;

const dicts = ["Dictionary", "Public Personal Notes/Dictionary"];

function stripPrefix(path) {
	if (path.endsWith('.md')) {
		path = path.slice(0,-3)
	}

  for (let dir of dicts) {
    if (path.startsWith(dir)) {
      return path.slice(dir.length + 1); // Strip the prefix
    }
  }
  return path; // No match
}

const allNotes = app.vault.getMarkdownFiles().filter(
	file => dicts.some(dir => file.path.startsWith(dir))
);
const allDirs = app.vault.getAllFolders().filter(
	file => dicts.some(dir => file.path.startsWith(dir))
);
const definitionOptions = [
	"New", 
	...allNotes.map(note => note.path),
	...allDirs.map(dir => dir.path)
];

let definitionName = await tp.system.suggester(
	stripPrefix, 
	definitionOptions, 
	false, 
	"Select an existing definition or choose 'New':"
);

let fileOrDir = app.vault.getAbstractFileByPath(definitionName);
let baseDir = "";

if (fileOrDir instanceof tp.obsidian.TFolder) {
  console.log("✅ It's a folder: " + fileOrDir.path);
  baseDir = fileOrDir.path;
  hasNote = false;
  isDir = true;
} else if (fileOrDir instanceof tp.obsidian.TFile) {
  console.log("📄 It's a file");
  hasNote = true;
} else {
  console.log("❌ Doesn't exist");
  baseDir = "Dictionary"
  hasNote = false;
}

if (!hasNote) {
	definitionName = await tp.system.prompt(
		"Enter the definition file name or sub/path/file name: ",
	);
}

if (definitionName.endsWith('.md')) {
	definitionName = definitionName.slice(0, -3);
}

// Handle nested path
if (definitionName.includes("/")) {
	const parts = definitionName.split("/");
	const fileName = parts.pop(); // Get the actual note name
	const subDir = parts.join("/");

	// Append subdir to the base folder
	if (baseDir == "") {
		baseDir = subDir;
	} else {
		baseDir = `${baseDir}/${subDir}`;
	}
	definitionName = fileName;
}

const definitionContent = await tp.system.prompt(
	"Enter the definition text:",
	multiline=true
);

const path = `${baseDir}/${definitionName}.md`;
const obsidianLink = `[[${baseDir}/${definitionName}|${definitionName}]]`
const basePathExists = app.vault.getAbstractFileByPath(baseDir);

if (!basePathExists) {
console.log("✅ creating folder");
	folder = await app.vault.createFolder(baseDir);
	console.log("✅ Created folder:", folder.path);
}

const exists = app.vault.getAbstractFileByPath(path);

if (!exists) {
	file = await app.vault.create(path, generated_note_dataview);
	console.log("✅ Created:", file.path);
} else {
	console.log("⚠️ Skipped, already exists:", path);
	new Notice(`⚠️ File already exists: ${path}`)
}

const slugify = str =>
  str.toLowerCase()
     .replace(/[\s_\/]+/g, "-")           // Replace spaces, underscores, and slashes with dash
     .replace(/[,:()\[\]]/g, "")          // Remove punctuation
     .replace(/[^a-z0-9\-]/g, "")         // Remove non-alphanumeric except dash
     .replace(/-+/g, "-")                 // Collapse multiple dashes
     .replace(/^-|-$/g, "");              // Trim leading/trailing dashes


const slug = slugify(`${baseDir}/${definitionName}`);
const timestamp = moment().format("YYYYMMDDHHmmss");
const blockId = `note-definition-${slug}-${timestamp}`;

// Construct the callout with the definition name and content
-%>
>[!note-definition] <% obsidianLink %> #note/definition
> <% definitionContent %> <% tp.file.cursor() %>
> ^<% blockId %>
