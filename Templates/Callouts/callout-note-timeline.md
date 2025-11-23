<%*
const definitionTemplateDir = "Templates/Callout Templates/Note Timeline Template";
const definitionTemplateFile = "/Note Timeline Template.md";
const subDefinitionTemplateDir = "/_Timelines/";
const subDefinitionTemplateFile = "/_Timelines/timeline-template.md";

function stripPrefix(path) {
	if (path.endsWith('.md')) {
		path = path.slice(0,-3)
	}
  return path; // No match
}

function get_path(path) { return app.vault.getAbstractFileByPath(path); }
function is_valid(file) { return file.name != "Timelines"; }

const allDirs = app.vault.getAllFolders().filter(file => is_valid(file));

let definitionName = await tp.system.suggester(
	stripPrefix, 
	[
		"New", 
		// Have a version with "/" so the user can
		// indicate they would to put something "new" inside other
		// definitions.
		...allDirs.flatMap(dir => [dir.path, dir.path + "/"])
	], 
	false, 
	"Select an existing definition or choose 'New':"
);

if (definitionName == "New") {
	definitionName = await tp.system.prompt(
		"Enter the definition file name file name: ",
	);
} else if (definitionName.endsWith("/")) {
	definitionName += await tp.system.prompt(
		"Enter the definition file name in " + definitionName + ": ",
	);
}

let definitionPath = get_path(definitionName);
let new_definition = "";

if (definitionPath === null) {
	const msg = "❌ Doesn't exist: " + definitionName + " creating!";
	console.log(msg);
	new Notice(msg);
	let result = await app.vault.copy(
		get_path(definitionTemplateDir),
		definitionName
	);
	definitionPath = get_path(definitionName);
	await app.vault.rename(
		await get_path(definitionPath.path + definitionTemplateFile),
		definitionPath.path + "/" + definitionPath.name + '.md'
	);
	new_definition = definitionPath.path 
		+ subDefinitionTemplateDir 
		+ definitionPath.name 
		+ moment().format("-YYYY-MM-DD-HH-mm-ss") 
		+ '.md';
	await app.vault.rename(
		await get_path(definitionPath.path + subDefinitionTemplateFile),
		new_definition
	);
} else {
    quotes_path_string = definitionPath.path + subDefinitionTemplateDir.slice(0,-1);
	quotes_path = get_path(quotes_path_string);
	console.log("quotes path: ", quotes_path);
	console.log("quotes_path_string: ", quotes_path_string);
	if (quotes_path === null) {
		console.log("Creating: ", quotes_path_string);
		await app.vault.createFolder(quotes_path_string);
	}
	new_definition = definitionPath.path 
		+ subDefinitionTemplateDir 
		+ definitionPath.name 
		+ moment().format("-YYYY-MM-DD-HH-mm-ss") 
		+ '.md';
	await app.vault.copy(
		get_path(definitionTemplateDir + subDefinitionTemplateFile),
		new_definition
	);
	const msg = "✅ Created: " + new_definition;
	console.log(msg);
	new Notice(msg);
}

const parentLink = get_path(
	definitionPath.path + "/" + definitionPath.name + '.md'
)
await app.fileManager.processFrontMatter(
	get_path(new_definition), 
(frontmatter) => {
  console.log("setting link");
  frontmatter["links"] = [];
  frontmatter["links"].push('[[' + parentLink.path + '|' + definitionPath.name + ']]');
});
const leaf = app.workspace.openLinkText(new_definition,"",true);
-%>
![[<% new_definition %>|<%definitionPath.name%>]]
