%%
Copyright (c) 2025 American Republic Wiki
%%
<%*
const quote = await tp.system.prompt("Enter the quote text:");

// Get a list of existing note titles for author and citation suggestions
const allNotes = app.vault.getMarkdownFiles();
const noteTitles = ["Self", "Custom", ...allNotes.map(note => note.basename)];

// Unified function to handle suggester with fallback to raw input
async function getLinkedInput(promptMessage, options = noteTitles) {
  const selectorPromptMessage = promptMessage + " (select 'Custom' for custom input):";
  let selection = await tp.system.suggester(options, options, false, selectorPromptMessage);
  let selected_self = false;
  
  if (selection === "Custom") {
    selection = await tp.system.prompt(promptMessage + " (raw input):");
  } else if (selection === "Self") {
    selected_self = true;
    selection = `[[${tp.file.title}]]`; // Link to current note
  } else if (selection && allNotes.some(note => note.basename === selection)) {
    selection = `[[${selection}]]`; // Link to the note if it matches an existing title
  }
  
  return [selection, selected_self];
}

// Helper function to extract properties (e.g., lit/type) from linked notes
async function getCurrentNoteProperty(property) {
  // Read the frontmatter of the note
  const property_value = await tp.frontmatter[property];
  if (property_value !== undefined) {
    return property_value;
  } else {
    // Property not found in frontmatter
    return [];
  }
}

// Prompt for author
const [author,selected_self] = await getLinkedInput("Enter or select the author's name");
console.log('author: ',author)
console.log('selected_self: ',selected_self)
// Prompt for citation only if author is provided
let citation = "";
if (author) {
  let litType = [];
  
  if (selected_self) {
    litType = await getCurrentNoteProperty("lit/type");
  }
  console.log(litType)
  if (litType.includes("book")) {
    // Prompt for page number
    const pageNumber = await tp.system.prompt("Enter the page number (pg:)");
    citation = `pg: ${pageNumber}`;
  } else if (litType.length > 0) {
    // Extend this section to handle other lit/types if needed
    // For now, proceed with the usual citation prompt
    const citationOptions = ["Custom", ...allNotes.map(note => note.basename)];
    citation = await getLinkedInput("Enter or select the citation or source", citationOptions);
  } 
}

let header = "";
if (author) {
  header = `*${author}*`;
  if (citation) {
    header += `, *${citation}*`;
  }
}

const slugify = str =>
  str.toLowerCase()
     .replace(/[\s]+/g, "-")
     .replace(/[,:()\[\]]/g, "")
     .replace(/[^a-z0-9\-]/g, "")
     .replace(/-+/g, "-")
     .replace(/^-|-$/g, "");

const slug = slugify(author);
const timestamp = moment().format("YYYYMMDDHHmmss");
const blockId = `note-quote-${slug}-${timestamp}`;
-%>
>[!note-quote] <% header %> #note/quote
> <% quote %><% tp.file.cursor() %>
> ^<% blockId %>
