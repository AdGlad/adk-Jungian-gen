# AI Coding Agent Instructions for adk-Jungian-gen

## Architecture Overview
This is a Google ADK-based book generation system with two main components:
- **Agent Workflow** (`p_book_gen/`): Sequential agents parse user spec JSON, generate outlines, front matter, parallel chapters, merge into Kindle-formatted Markdown, save to GCS.
- **Cloud Run EPUB Builder** (`main.py`): Eventarc-triggered service that converts Markdown to EPUB using pandoc and uploads to GCS.

Data flows: ADK agents → GCS manuscript → job JSON upload → Cloud Run → EPUB upload.

## Key Patterns
- **Agent State Management**: Store canonical data in `ctx.session.state` (e.g., `state["book_title"]`, `state["chapter_1_text"]`) for downstream agents.
- **GCS Integration**: Use `google.cloud.storage.Client()` for uploads/downloads. URIs like `gs://bucket/path`. Environment: `BOOK_GEN_BUCKET`.
- **Job JSON Format**: Must include `manuscript_gs_uri`, `output_epub_gs_uri`, `metadata` (title, author, etc.). Placed under `/jobs/` with `-epub-job.json` suffix.
- **Markdown Formatting**: Use `<mbp:pagebreak />` for Kindle pagebreaks. UK English. H1 for title, H2 for chapters/sections.
- **Parallel Agents**: `ParallelAgent` for concurrent chapter generation, storing results in state keys like `chapter_1_text`.

## Development Workflow
- **Local Testing**: Run agents via ADK CLI/SDK. Mock GCS if needed.
- **Deployment**: Build Docker image with pandoc. Deploy `main.py` as Cloud Run service triggered by GCS Eventarc.
- **Environment Setup**: Set `BOOK_GEN_BUCKET`, `GOOGLE_CLOUD_PROJECT`. Use service account with GCS access.

## Conventions
- **Imports**: `from google.adk.agents import LlmAgent, SequentialAgent, ParallelAgent`
- **Async Generators**: Agents yield `Event` objects with `genai_types.Content`.
- **Error Handling**: Yield system events for missing state/data. Defensive GCS URI parsing.
- **File Structure**: Agents in `custom_agents.py`, tools in `tools.py`, root in `agent.py`.

## Examples
- **State Access**: `manuscript = state.get("book_manuscript")` in `ProseNormaliserAgent`
- **GCS Upload**: `bucket.blob(object_name).upload_from_string(content, content_type="text/markdown")`
- **Pandoc Command**: `subprocess.run(["pandoc", md_path, "-o", epub_path, "--toc", "--metadata", f"title={title}"], check=True)`

Reference: `p_book_gen/custom_agents.py` for agent implementations, `main.py` for Cloud Run logic.