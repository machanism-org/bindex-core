<!-- @guidance: 
Create the `Function Tolls` page:
- **Download:**  
   Add a download link for the bindex.jar:  
   [![Download Bindex-Core](https://a.fsdn.com/con/app/sf-download-button)](https://sourceforge.net/projects/machanism/files/machai/bindex/bindex.jar/download)
   You can use it by classpass as a bindex related functinal tools for [MCP Machai Server](https://machai.machanism.org/mcp-machai-server/index.html).
- Analyze classes in the folder: `src/main/java/org/machanism/machai/bindex/ai/tools` and use this information to create the page content but do not mentionad this as a package details.
- If the function tool class is annotated with the `@SupportedFor` annotation, specify this in the description of the function tool methods.
- Write a general description of the each functional tool.
- Describe a feature and input parameters.
- Organize your output so that each act is easy to identify and understand.
- Ensure your descriptions are user-friendly and help the reader quickly determine the function and appropriate use case for each act.
-->

# Function Tools

Bindex-Core provides AI-facing functions for retrieving Bindex metadata, finding relevant libraries, and registering descriptors. It also provides a JSON Schema resource and a prompt template to help agents validate and create Bindex files. These capabilities are useful in AI agents and MCP-compatible integrations that need to discover libraries, inspect metadata, or publish descriptors.

## Download

[![Download Bindex-Core](https://a.fsdn.com/con/app/sf-download-button)](https://sourceforge.net/projects/machanism/files/machai/bindex/bindex.jar/download)

Add `bindex.jar` to the classpath to use these Bindex-related function tools with the [MCP Machai Server](https://machai.machanism.org/mcp-machai-server/index.html).

## Functions

### `get_bindex`

Retrieves a Bindex descriptor for a project or library. Provide a registered Bindex identifier, a direct HTTP(S) URL to a descriptor, or a `file://` path to a local JSON descriptor. Relative local paths are resolved against the project directory. An optional GraphQL-style selection query can reduce the response to requested top-level fields.

**Use it when:** you need to inspect a registered descriptor or load metadata directly from a remote or local file. A selection query is useful when you only need fields such as `name`, `version`, or `classification` and want to keep the response smaller.

**Input parameters:**

| Parameter | Required | Description |
| --- | --- | --- |
| `id` | Yes | A Bindex identifier such as `groupId:artifactId:version`, an HTTP(S) URL to a `bindex.json` file, or a `file://` path. Relative local file paths use the active project directory. |
| `graphql_query` | No | A GraphQL-style selection query, such as `{ name classification { languages } }`, that selects top-level fields from the returned descriptor. |

**Returns:** the complete Bindex descriptor when no query is supplied, or a descriptor containing the requested top-level fields that exist when a query is supplied. Nested selection syntax is accepted, but nested objects are returned whole rather than recursively filtered.

### `pick_libraries`

Recommends libraries based on a natural-language description of project needs or requirements. Results are selected through vector search using the requested relevance threshold and maximum result count.

**Use it when:** you need library or dependency ideas for a feature, technology stack, or implementation goal, ranked by semantic relevance.

**Input parameters:**

| Parameter | Required | Description |
| --- | --- | --- |
| `prompt` | Yes | The project need, desired functionality, technology stack, or feature for which libraries should be recommended. |
| `score` | No | The minimum relevance score; only results at or above this threshold are included. Default: `0.85`. |
| `limits` | No | The maximum number of recommendations to retrieve. Default: `25`. |

**Returns:** a collection of recommended Bindex records. Raise `score` to filter for closer matches, or increase `limits` to request more results.

### `register_bindex`

Reads a Bindex descriptor from a local project file or an HTTP(S) URL, sets the Bindex schema reference, and registers the descriptor. It returns the identifier of the saved record.

**Use it when:** you want to add or update metadata from a `bindex.json` file in the project or from a descriptor hosted remotely. Local files must be within the active project directory.

**Input parameters:**

| Parameter | Required | Description |
| --- | --- | --- |
| `file` | No | A relative or project-contained file path, or an HTTP(S) URL. Defaults to `bindex.json`. |

**Returns:** the identifier assigned to the registered record.

### `register_bindex_json`

Registers a Bindex descriptor supplied directly as a JSON object. The tool sets the Bindex schema reference before saving and returns the saved record identifier.

**Use it when:** an agent or integration already has a descriptor in memory and should register it without first writing a local file or fetching a URL.

**Input parameters:**

| Parameter | Required | Description |
| --- | --- | --- |
| `bindex_json` | Yes | The structured Bindex JSON object to register. |

**Returns:** the identifier assigned to the registered record.

## Supporting Resources and Prompts

### `getBindexSchema` resource

Provides the Bindex v2 JSON Schema as UTF-8 `application/json` content. Use this schema to check a descriptor’s structure and metadata before registering it.

**Input:** `uri` — the resource URI supplied by the tool framework. The resource is exposed at `file:///schema/bindex-schema-v2.json`.

**Returns:** the schema document as a UTF-8 JSON string.

### `generate-bindex` prompt

Loads the Markdown template containing instructions and contextual prompts for generating a Bindex file. Use it as starting context when creating a descriptor that follows the Bindex format.

**Input:** none.

**Returns:** the Markdown generation template as a string.

## Response and Usage Notes

- `get_bindex` reads by repository identifier, HTTP(S) URL, or `file://` path. A supplied selection query filters top-level fields only; any selected nested object is returned whole.
- `pick_libraries` returns recommended Bindex records according to the specified score threshold and result limit.
- Both registration functions set the descriptor’s schema reference before saving and return the saved record identifier.
- File-based registration is restricted to the active project directory; remote descriptor URLs must be accessible through HTTP(S).
