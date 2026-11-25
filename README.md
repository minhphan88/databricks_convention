# databricks_convention
Databricks Prototyping: Using Workspaces as APIs
When building prototypes, you often need a backend API to serve your machine learning models, LLM routing logic, or even simple Python functions (like an MCP server). Instead of building and deploying a standalone web app (e.g., Flask, FastAPI), you can use Databricks out-of-the-box features to act as your API layer.
Here are the three primary ways to turn your Databricks code into an API.
1. The "Cloud Function" Approach (Real-time REST API)
If you need a fast, synchronous response (like AWS Lambda or Google Cloud Functions), Databricks Model Serving using Custom Python Models (⁠mlflow.pyfunc⁠) is the best approach.
This lets you write arbitrary Python code in a notebook, log it to MLflow, and instantly expose it as a REST API endpoint backed by Serverless compute.
Best for: Real-time ML inference, MCP (Model Context Protocol) servers, simple Python backends, and LangChain apps.
How it works:
1. Write a Python Class: Wrap your logic (or MCP server logic) inside a class that inherits from ⁠mlflow.pyfunc.PythonModel⁠.
2. Define the ⁠predict⁠ method: This method will receive the JSON payload from your API request and return the response.
3. Log the Model: Run ⁠mlflow.pyfunc.log_model()⁠ in your notebook.
4. Serve It: Go to the Databricks "Serving" tab and create an endpoint pointing to this model. Databricks automatically generates the REST API URL and manages the Serverless compute.
Note: Remember to use your naming conventions (e.g., ⁠prj123_mcp_routing_endpoint⁠).
2. The "Asynchronous Task" Approach (Jobs API)
If your app needs to trigger a long-running process (like a heavy data crunch, batch ML scoring, or complex prototype initialization) and doesn't need an immediate sub-second response, use the Databricks Jobs API with Notebooks.
Best for: Heavy backend processing, background tasks, data syncs.
How it works:
1. Set up Parameters: Use ⁠dbutils.widgets.text("param_name", "default")⁠ at the top of your notebook to accept inputs.
2. Create a Job: Turn this notebook into a Databricks Job.
3. Trigger via API: Your frontend app sends an HTTP POST request to the Databricks Jobs API (⁠/api/2.1/jobs/run-now⁠) passing the widget parameters in the JSON payload.
4. Fetch Results: Your app can either poll for completion or the notebook can write the results to a Delta table/external database when finished.
3. The "Data API" Approach (Databricks SQL Statement API)
If your prototype simply needs to query data, metrics, or ML predictions that already exist in a table, you do not need to write a custom API wrapper. You can use the Databricks SQL Statement Execution API.
Best for: Dashboards, frontend apps fetching data, reporting.
How it works:
1. Use a Serverless SQL Warehouse (e.g., ⁠prj123_app_compute⁠).
2. Your application makes an HTTP request directly to the Databricks workspace URL ⁠.../api/2.0/sql/statements/⁠.
3. Pass your raw SQL query (e.g., ⁠SELECT * FROM prj123_model_predictions WHERE user_id = X⁠) in the payload.
4. The API returns the data directly as JSON.
Best Practices for Rapid Prototyping
￼ Use Serverless: Always choose Serverless Compute for your SQL Warehouses and Model Serving endpoints. It scales down to zero, saving costs when you aren't actively testing the prototype, and spins up quickly when you do.
￼ Keep Dependencies Minimal: When using the "Cloud Function" approach (⁠mlflow.pyfunc⁠), explicitly define your ⁠pip⁠ requirements when logging the model so the endpoint boots up flawlessly.
￼ Authentication: Databricks REST APIs require authentication. For prototypes, generate a Personal Access Token (PAT) or a Service Principal token, and store it securely in your frontend app's environment variables to authenticate requests to your Databricks workspace.
￼ Consistent Naming: Tie everything back to your feature ID.
￼ Code: ⁠prj123_mcp_server_logic⁠ (Notebook)
￼ MLflow: ⁠prj123_mcp_server_model⁠ (Registered Model)
￼ API Endpoint: ⁠prj123_mcp_endpoint
⁠ (Serving Endpoint)


Databricks Simple Naming Conventions
This guide outlines a minimal, easy-to-follow naming convention for your Databricks environment.
The Golden Rule: All assets related to a specific piece of work should be tied together using a Feature ID (e.g., a Jira ticket number like ⁠PROJ-123⁠ converted to ⁠proj123⁠).
Use ⁠snake_case⁠ (lowercase with underscores) for all Databricks assets, and ⁠kebab-case⁠ (lowercase with hyphens) for Git branches.
1. The Core Identifier (Feature ID)
Decide on a short identifier for your feature or task.
￼ Example: ⁠prj123⁠
2. Git Repos & Branches
Branch names should start with the type of work (⁠feat⁠, ⁠bug⁠, ⁠hotfix⁠), followed by the Feature ID and a brief description.
￼ Format: ⁠[type]/[feature-id]-[short-description]⁠
￼ Example: ⁠feat/prj123-customer-churn-model⁠
￼ Example: ⁠bug/prj123-fix-prompt-logging⁠
3. Workspace Assets (Notebooks & Folders)
Notebooks should clearly indicate the Feature ID and the action they perform. Keep them in a project-specific folder.
￼ Folder Format: ⁠/Workspace/Repos/[Project_Name]/[feature_id]⁠
￼ Notebook Format: ⁠[feature_id]_[action]_[target]⁠
￼ Example: ⁠prj123_train_churn_model⁠
￼ Example: ⁠prj123_evaluate_llm_prompts⁠
4. Data Assets (Databases & Tables)
For Unity Catalog or Hive Metastore, group tables under a standard database/schema, but prefix the tables with the Feature ID to isolate development work.
￼ Database / Schema: ⁠[project_name]_[environment]⁠ (e.g., ⁠customer_ml_dev⁠)
￼ Tables / Delta Tables: ⁠[feature_id]_[data_description]⁠
￼ Example: ⁠prj123_training_features⁠
￼ Example: ⁠prj123_model_predictions⁠
5. MLflow & LLMOps Assets
Tie your MLflow experiments and LLM logs directly to the Feature ID so you can trace a model back to the exact Git branch that created it.
￼ MLflow Experiment: ⁠[feature_id]_[model_name]_exp⁠
￼ Example: ⁠prj123_churn_xgboost_exp⁠
￼ Registered Model Name: ⁠[project_name]_[model_name]⁠ (Keep Feature ID out of the final production registry name, use tags instead).
￼ Example: ⁠customer_churn_model⁠ (Tag with ⁠branch: feat/prj123⁠)
￼ LLMOps Log Tables (Inference Tables / Payload Logging): ⁠[feature_id]_llm_logs⁠
￼ Example: ⁠prj123_llm_inference_logs⁠
￼ Example: ⁠prj123_prompt_eval_results⁠
6. Model Inference & Serverless Compute
When using notebook servers as inference endpoints or utilizing serverless compute for app backends, clearly label the service and its compute layer with the Feature ID to track active processes and costs.
￼ Inference Endpoint (Model Serving / Notebook): ⁠[feature_id]_[model_name]_endpoint⁠
￼ Example: ⁠prj123_churn_endpoint⁠
￼ Example: ⁠prj123_llm_routing_endpoint⁠
￼ Serverless Compute (SQL Warehouses / App Compute): ⁠[feature_id]_[process_or_app_name]_compute⁠
￼ Example: ⁠prj123_customer_app_compute⁠
￼ Example: ⁠prj123_data_sync_serverless⁠
Summary Checklist
If working on Ticket #123 for "Churn":
￼ [ ] Branch: ⁠feat/prj123-churn⁠
￼ [ ] Notebook: ⁠prj123_prep_data⁠
￼ [ ] Data Table: ⁠prj123_churn_features⁠
￼ [ ] MLflow Exp: ⁠prj123_churn_exp⁠
￼ [ ] LLM Log: ⁠prj123_llm_logs⁠
￼ [ ] Endpoint: ⁠prj123_churn_endpoint⁠
￼ [ ] Compute: ⁠prj123_churn_app_compute⁠


from __future__ import annotations

import json
import re
from collections import Counter
from dataclasses import dataclass, field
from enum import Enum
from pathlib import Path
from typing import Any, Iterator, Literal


class Engine(str, Enum):
    DOCLING = "docling"
    PYMUPDF = "pymupdf"
    PDFPLUMBER = "pdfplumber"


# ---------------------------------------------------------------------------
# Unified Hierarchical Data Model
# ---------------------------------------------------------------------------

@dataclass
class ContentBlock:
    type: Literal["paragraph", "table", "list", "code", "other"]
    content: str
    page: int | None = None

    def to_dict(self) -> dict[str, Any]:
        return {"type": self.type, "content": self.content, "page": self.page}


@dataclass
class SectionNode:
    id: str
    title: str
    level: int  # 0 = document root, 1..6 = heading depth
    page: int | None = None
    blocks: list[ContentBlock] = field(default_factory=list)
    children: list[SectionNode] = field(default_factory=list)

    def to_dict(self, include_children: bool = True) -> dict[str, Any]:
        data: dict[str, Any] = {
            "id": self.id,
            "title": self.title,
            "level": self.level,
            "page": self.page,
            "blocks": [b.to_dict() for b in self.blocks],
        }
        if include_children:
            data["children"] = [c.to_dict(include_children=True) for c in self.children]
        return data

    def to_json(self, include_children: bool = True, indent: int = 2) -> str:
        return json.dumps(
            self.to_dict(include_children=include_children),
            indent=indent,
            ensure_ascii=False,
        )

    def to_markdown(self, include_children: bool = True) -> str:
        parts: list[str] = []

        if self.level > 0 and self.title:
            hashes = "#" * min(self.level, 6)
            parts.append(f"{hashes} {self.title}")

        # Merge consecutive list blocks without extra blank lines between items
        idx = 0
        while idx < len(self.blocks):
            block = self.blocks[idx]
            if block.type == "list":
                list_lines = [block.content]
                while idx + 1 < len(self.blocks) and self.blocks[idx + 1].type == "list":
                    idx += 1
                    list_lines.append(self.blocks[idx].content)
                parts.append("\n".join(list_lines))
            else:
                parts.append(block.content)
            idx += 1

        if include_children:
            for child in self.children:
                child_md = child.to_markdown(include_children=True)
                if child_md:
                    parts.append(child_md)

        return "\n\n".join(p for p in parts if p)

    def iter_sections(self, path: tuple[str, ...] = ()) -> Iterator[tuple[tuple[str, ...], SectionNode]]:
        current_path = path + ((self.title,) if self.level > 0 else ())
        if self.level > 0 or self.blocks:
            yield current_path, self
        for child in self.children:
            yield from child.iter_sections(current_path)


@dataclass
class ParsedDocument:
    source: str
    engine: str
    root: SectionNode
    metadata: dict[str, Any] = field(default_factory=dict)

    def to_dict(self) -> dict[str, Any]:
        return {
            "source": self.source,
            "engine": self.engine,
            "metadata": self.metadata,
            "document": self.root.to_dict(include_children=True),
        }

    def to_json(self, indent: int = 2) -> str:
        return json.dumps(self.to_dict(), indent=indent, ensure_ascii=False)

    def to_markdown(self) -> str:
        """Reconstruct the entire document as Markdown."""
        return self.root.to_markdown(include_children=True)

    def get_section(self, key: str) -> SectionNode | None:
        """Find a section by hierarchical ID (e.g. '1.2') or case-insensitive title."""
        needle = key.strip().lower()
        for _, node in self.root.iter_sections():
            if node.id.lower() == needle or node.title.strip().lower() == needle:
                return node
        return None

    def to_parts(
        self,
        output_format: Literal["markdown", "json", "dict"] = "markdown",
        include_children: bool = False,
    ) -> list[dict[str, Any]]:
        """Export every section as an independent part in Markdown, JSON, or dict form."""
        parts: list[dict[str, Any]] = []
        for path, node in self.root.iter_sections():
            if output_format == "markdown":
                content: Any = node.to_markdown(include_children=include_children)
            elif output_format == "json":
                content = node.to_json(include_children=include_children)
            else:
                content = node.to_dict(include_children=include_children)

            parts.append(
                {
                    "id": node.id,
                    "title": node.title,
                    "level": node.level,
                    "path": list(path),
                    "page": node.page,
                    "content": content,
                }
            )
        return parts


# ---------------------------------------------------------------------------
# Internal Tree Builder & Markdown Helpers
# ---------------------------------------------------------------------------

class _HierarchyBuilder:
    def __init__(self, doc_title: str) -> None:
        self.root = SectionNode(id="0", title=doc_title, level=0)
        self._stack: list[SectionNode] = [self.root]

    def add_heading(self, title: str, level: int, page: int | None = None) -> SectionNode:
        clean_title = title.strip()
        level = max(1, min(6, level))
        while len(self._stack) > 1 and self._stack[-1].level >= level:
            self._stack.pop()

        parent = self._stack[-1]
        seq = len(parent.children) + 1
        node_id = str(seq) if parent.level == 0 else f"{parent.id}.{seq}"

        node = SectionNode(id=node_id, title=clean_title, level=level, page=page)
        parent.children.append(node)
        self._stack.append(node)
        return node

    def add_block(
        self,
        block_type: Literal["paragraph", "table", "list", "code", "other"],
        content: str,
        page: int | None = None,
    ) -> None:
        text = content.strip()
        if not text:
            return
        self._stack[-1].blocks.append(ContentBlock(type=block_type, content=text, page=page))


_HEADING_RE = re.compile(r"^(#{1,6})\s+(.+?)\s*#*$")
_LIST_RE = re.compile(r"^(\s*[-*+]|\s*\d+\.)\s+")


def _ingest_markdown(builder: _HierarchyBuilder, md_text: str, page: int | None) -> None:
    """Parse Markdown text into headings and typed content blocks."""
    buf: list[str] = []
    buf_type: Literal["paragraph", "table", "list", "code", "other"] | None = None
    in_code = False

    def flush() -> None:
        nonlocal buf, buf_type
        if buf and buf_type:
            builder.add_block(buf_type, "\n".join(buf), page=page)
        buf = []
        buf_type = None

    for raw_line in md_text.splitlines():
        line = raw_line.rstrip()

        if line.lstrip().startswith("```"):
            if in_code:
                buf.append(line)
                flush()
                in_code = False
            else:
                flush()
                in_code = True
                buf_type = "code"
                buf.append(line)
            continue

        if in_code:
            buf.append(line)
            continue

        if not line.strip():
            flush()
            continue

        m = _HEADING_RE.match(line)
        if m:
            flush()
            builder.add_heading(title=m.group(2), level=len(m.group(1)), page=page)
            continue

        if line.lstrip().startswith("|") and "|" in line[1:]:
            if buf_type != "table":
                flush()
                buf_type = "table"
            buf.append(line)
            continue

        if _LIST_RE.match(line):
            flush()
            builder.add_block("list", line, page=page)
            continue

        if buf_type != "paragraph":
            flush()
            buf_type = "paragraph"
        buf.append(line)

    flush()


def _rows_to_markdown_table(rows: list[list[str | None]]) -> str:
    cleaned = [
        [("" if c is None else str(c).replace("\n", " ").replace("|", "\\|").strip()) for c in row]
        for row in rows
        if any(c is not None and str(c).strip() for c in row)
    ]
    if not cleaned:
        return ""
    width = max(len(r) for r in cleaned)
    normalised = [r + [""] * (width - len(r)) for r in cleaned]
    header = "| " + " | ".join(normalised[0]) + " |"
    sep = "| " + " | ".join("---" for _ in range(width)) + " |"
    body = ["| " + " | ".join(r) + " |" for r in normalised[1:]]
    return "\n".join([header, sep, *body])


# ---------------------------------------------------------------------------
# Engine Implementations
# ---------------------------------------------------------------------------

def _parse_with_docling(path: Path, **kwargs: Any) -> ParsedDocument:
    try:
        from docling.datamodel.base_models import InputFormat
        from docling.datamodel.pipeline_options import HeadingHierarchyOptions, PdfPipelineOptions
        from docling.document_converter import DocumentConverter, PdfFormatOption
        from docling_core.types.doc.document import ListItem, SectionHeaderItem, TableItem, TextItem
        from docling_core.types.doc.labels import DocItemLabel
    except ImportError as exc:
        raise ImportError("Docling is not installed. Run: pip install docling") from exc

    hierarchy_opts = HeadingHierarchyOptions(enabled=True)
    pipeline_opts = PdfPipelineOptions(heading_hierarchy_options=hierarchy_opts, **kwargs)
    converter = DocumentConverter(
        format_options={InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_opts)}
    )
    doc = converter.convert(str(path)).document

    builder = _HierarchyBuilder(doc_title=path.stem)
    for item, _ in doc.iterate_items():
        prov = getattr(item, "prov", None)
        page = prov[0].page_no if prov else None
        label = getattr(item, "label", None)

        if isinstance(item, SectionHeaderItem):
            builder.add_heading(item.text, level=getattr(item, "level", 1), page=page)
        elif label == DocItemLabel.TITLE and hasattr(item, "text"):
            builder.add_heading(item.text, level=1, page=page)
        elif isinstance(item, TableItem):
            builder.add_block("table", item.export_to_markdown(doc=doc), page=page)
        elif isinstance(item, ListItem):
            marker = getattr(item, "marker", "-") or "-"
            builder.add_block("list", f"{marker} {item.text.strip()}", page=page)
        elif label == DocItemLabel.CODE and hasattr(item, "text"):
            builder.add_block("code", f"```\n{item.text}\n```", page=page)
        elif isinstance(item, TextItem):
            builder.add_block("paragraph", item.text, page=page)

    return ParsedDocument(
        source=str(path),
        engine=Engine.DOCLING.value,
        root=builder.root,
        metadata={"page_count": len(doc.pages)},
    )


def _parse_with_pymupdf(path: Path, **kwargs: Any) -> ParsedDocument:
    try:
        import pymupdf4llm
    except ImportError as exc:
        raise ImportError("PyMuPDF4LLM is not installed. Run: pip install pymupdf4llm") from exc

    chunks = pymupdf4llm.to_markdown(str(path), page_chunks=True, **kwargs)
    builder = _HierarchyBuilder(doc_title=path.stem)

    for idx, chunk in enumerate(chunks):
        meta = chunk.get("metadata", {})
        page = meta.get("page", idx + 1)
        _ingest_markdown(builder, chunk.get("text", ""), page=page)

    return ParsedDocument(
        source=str(path),
        engine=Engine.PYMUPDF.value,
        root=builder.root,
        metadata={"page_count": len(chunks)},
    )


def _parse_with_pdfplumber(
    path: Path,
    heading_size_delta: float = 0.8,
    max_heading_levels: int = 4,
    line_tolerance: float = 3.0,
) -> ParsedDocument:
    try:
        import pdfplumber
        from pdfplumber.utils import extract_text
    except ImportError as exc:
        raise ImportError("pdfplumber is not installed. Run: pip install pdfplumber") from exc

    pages_items: list[tuple[int, list[dict[str, Any]]]] = []
    font_weights: Counter[float] = Counter()

    with pdfplumber.open(str(path)) as pdf:
        page_count = len(pdf.pages)
        for page_idx, page in enumerate(pdf.pages, start=1):
            tables = page.find_tables()
            table_bboxes = [t.bbox for t in tables]
            elements: list[dict[str, Any]] = []

            for tbl in tables:
                md_table = _rows_to_markdown_table(tbl.extract())
                if md_table:
                    elements.append({"kind": "table", "top": tbl.bbox[1], "text": md_table})

            # Filter out characters that belong to detected tables
            non_table_chars = [
                c
                for c in page.chars
                if not any(
                    bx0 <= (c["x0"] + c["x1"]) / 2 <= bx1
                    and btop <= (c["top"] + c["bottom"]) / 2 <= bbot
                    for (bx0, btop, bx1, bbot) in table_bboxes
                )
            ]

            # Group characters into visual lines
            lines: list[list[dict[str, Any]]] = []
            for ch in sorted(non_table_chars, key=lambda c: (c["top"], c["x0"])):
                if not lines or abs(ch["top"] - lines[-1][0]["top"]) > line_tolerance:
                    lines.append([ch])
                else:
                    lines[-1].append(ch)

            for line_chars in lines:
                line_chars.sort(key=lambda c: c["x0"])
                text = (extract_text(line_chars) or "").strip()
                if not text:
                    continue
                sizes = [round(float(c["size"]), 1) for c in line_chars if c.get("size")]
                line_size = Counter(sizes).most_common(1)[0][0] if sizes else 10.0
                font_weights[line_size] += len(text)
                elements.append(
                    {
                        "kind": "line",
                        "top": line_chars[0]["top"],
                        "text": text,
                        "size": line_size,
                    }
                )

            elements.sort(key=lambda e: e["top"])
            pages_items.append((page_idx, elements))

    body_size = font_weights.most_common(1)[0][0] if font_weights else 10.0
    heading_sizes = sorted(
        (s for s in font_weights if s >= body_size + heading_size_delta),
        reverse=True,
    )[:max_heading_levels]
    size_to_level = {size: lvl for lvl, size in enumerate(heading_sizes, start=1)}

    builder = _HierarchyBuilder(doc_title=path.stem)
    for page_no, elements in pages_items:
        para_buf: list[str] = []

        def flush_para() -> None:
            if para_buf:
                builder.add_block("paragraph", " ".join(para_buf), page=page_no)
                para_buf.clear()

        for elem in elements:
            if elem["kind"] == "table":
                flush_para()
                builder.add_block("table", elem["text"], page=page_no)
            else:
                level = size_to_level.get(elem["size"])
                if level is not None and len(elem["text"]) <= 140:
                    flush_para()
                    builder.add_heading(elem["text"], level=level, page=page_no)
                else:
                    para_buf.append(elem["text"])
        flush_para()

    return ParsedDocument(
        source=str(path),
        engine=Engine.PDFPLUMBER.value,
        root=builder.root,
        metadata={"page_count": page_count, "detected_body_font_size": body_size},
    )


# ---------------------------------------------------------------------------
# Public Entry Point
# ---------------------------------------------------------------------------

def parse_pdf(
    file_path: str | Path,
    engine: Engine | Literal["docling", "pymupdf", "pdfplumber"] = Engine.DOCLING,
    **engine_kwargs: Any,
) -> ParsedDocument:
    """Parse a PDF into a flexible hierarchical ParsedDocument."""
    path = Path(file_path)
    if not path.is_file():
        raise FileNotFoundError(f"PDF not found: {path}")

    selected = Engine(engine)
    if selected is Engine.DOCLING:
        return _parse_with_docling(path, **engine_kwargs)
    if selected is Engine.PYMUPDF:
        return _parse_with_pymupdf(path, **engine_kwargs)
    if selected is Engine.PDFPLUMBER:
        return _parse_with_pdfplumber(path, **engine_kwargs)
    raise ValueError(f"Unsupported engine: {engine}")

from pdf_parser import parse_pdf

# 1. Switch engines seamlessly: "docling", "pymupdf", or "pdfplumber"
doc = parse_pdf("report.pdf", engine="docling")

# 2. Reconstruct the entire document
full_md = doc.to_markdown()
full_json = doc.to_json()

# 3. Export each section individually (ideal for RAG or per-section processing)
md_parts = doc.to_parts(output_format="markdown")
json_parts = doc.to_parts(output_format="json", include_children=False)

# 4. Grab a single branch by hierarchical ID or heading title
section = doc.get_section("1.2")  # or doc.get_section("Methodology")
if section:
    print(section.to_markdown(include_children=True))
    print(section.to_json(include_children=False))



### Scope & Guardrails
- Only answer questions related to price change requests and the information contained within the retrieved documents.
- Do not answer off-topic, general knowledge, or unrelated questions. If a user asks something outside this scope, politely decline and state that you can only assist with queries related to price change guidelines and historical price change examples.

### Document Type Context
Use the following definitions when evaluating `kira_extracted_documents_ns.document_type` to select relevant documents and answer queries:
- **guideline**: Government guidelines detailing how to manage and process a price change request.
- **TDD**: Historical examples of past price change requests completed using a standard template.

### Execution Steps
For every valid user query, follow these exact steps in order:

1. **Identify the Top 3 Relevant Documents:**
   - Evaluate `kira_extracted_documents_ns.document_type` (using the context above) and the first 1,200 characters of `kira_extracted_documents_ns.markdown_content` (e.g., using `LEFT(markdown_content, 1200)`).
   - Use `system.ai_classify` against the user query to identify and rank the top 3 most relevant documents.
   - Retain the `kira_extracted_documents_ns.file_path` values for these top 3 documents.

2. **Retrieve the Top 10 Relevant Chunks and Source Filenames:**
   - Filter `kira_extracted_documents_ns` to include only records matching the top 3 `file_path` values from Step 1.
   - Join with `kira_extracted_document` on `file_path` to retrieve the corresponding `kira_extracted_document.file_name`.
   - Evaluate `kira_extracted_documents_ns.chunk_text` against the user query to select the 10 most relevant chunks, keeping their associated `file_name`.

3. **Generate the Answer with Citations:**
   - Use only the 10 retrieved chunks from Step 2 to answer the user's query accurately and concisely.
   - Cite the exact `kira_extracted_document.file_name` inline next to each key fact or claim (e.g., `[file_name.pdf]`).
   - Include a **Sources** list at the end of the response containing the unique `file_name` values referenced.
   - If the retrieved chunks do not contain enough information to answer the query, state this clearly rather than guessing.

4. **Cache Query Data:**
   - Save the query data—including the user query, top 3 `file_path` values, top 10 retrieved chunks, referenced `file_name` values, and timestamp—into `<CACHE_TABLE_NAME>`.


qa_agent_project/
├── agents/
│   ├── rag_agent.py      # Agent 1: Governance RAG
│   └── mcp_agent.py      # Agent 2: Mock DB + FastMCP server
└── orchestrator.ipynb    # Classifier, routing, synthesis & MLflow deployment
