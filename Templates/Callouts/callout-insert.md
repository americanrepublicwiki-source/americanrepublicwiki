<%*
let calloutDir = "Templates/Callouts"; 

const calloutFiles = app.vault.getMarkdownFiles() .filter(file => file.path.startsWith(calloutDir) && file.basename !== "callout-insert");

// Extract the base names without file extension for display
const calloutOptions = calloutFiles.map(f => f.basename);

// Prompt the user to select a callout
const selectedCalloutTitle = await tp.system.suggester(calloutOptions, calloutFiles); 

if (selectedCalloutTitle) { 
   const content = await tp.file.include(selectedCalloutTitle); 
   tR += content.trim(); 
} else { 
    tR += "Error: No callout type selected."; 
}
%>
