🧾 Confluence Page: Document Processing & Ingestion Pipeline (RAG System)

📌 Overview
This page details the end-to-end document ingestion pipeline that supports PDF, DOCX, TXT, PPTX, and Confluence. It outlines:
High-Level Architecture (HLD)


Low-Level Design (LLD)


Input processor behavior per file type


Embedded diagrams and code-level insights



📐 High-Level Design (HLD)
Architecture Flow:
[FastAPI Layer]
    └── /upload/{type} endpoint
        └── Auth → Parse Metadata
            └── RAG Handler → Processor Router
                └── Processor (PDF/DOCX/etc.)
                    └── Clean/Chunk
                        └── PII Removal
                            └── Embedding Model
                                └── Ingestion API

FastAPI accepts file + metadata via route.


RAGHandler decides which processor to run based on file type.


Processor Modules parse content, normalize it, chunk it.


Common Module handles cleaning, PII removal, embedding.


Ingestion Module interacts with downstream vector storage APIs.



🔽 Input Type Flows (LLD)
📄 DOCX
Libraries: docx, docx2txt, BeautifulSoup


Functions: get_docx_elements, rows_by_heading, HTMLParser


Flow:


Extract text + HTML from docx


Detect headings and assign hierarchy (1.1, 1.1.1)


Extract embedded HTML tables


Split large tables by char size


Merge headers + content → chunk → clean



📄 PDF
Libraries: unstructured, pandas


Functions: partition, convert_to_csv


Flow:


Extract text blocks using partition


Normalize via CSV conversion


Group per page


Clean and chunk



📜 TXT
Libraries: open(), pandas


Flow:


Read file


Remove noise (numbers, whitespace)


Run recursive text splitting


Clean and embed



📊 PPTX
Libraries: pdf2image, Pillow, base64, LLM


Flow:


Convert PPTX to PDF → Image (per slide)


Convert each image to base64


Query LLM for slide text


On error (firewall), resize + retry



🌐 Confluence
Libraries: atlassian-python-api, BeautifulSoup


Parser: HTMLParser


Flow:


Use ConfluenceReader to fetch HTML via page ID/title


Normalize heading hierarchy


Parse content and embedded tables


Apply chunking → cleanup → PII masking




🔁 Retry Support
If ingestion or embedding fails:
The record is stored as a failed item


The /retry endpoint supports automatic re-embedding and ingestion


Failures due to ai_firewall_sre or connection timeouts are logged



🧠 Embedding Layer
Async model fetched via amodel()


Embeds clean chunk into vector using aembed_query()


Retry up to 3 times



🧱 Ingestion Layer
Ingestion class calls CreateAiDaFeatureOpsVector


Uploads document events (STARTED, COMPLETED, FAILURE)


Embeds are vectorized and pushed via payload



✅ Unit Tests
Every file has tests/test_<module>.py


Uses pytest, unittest.mock


Mocks I/O, LLM, ingestion



📎 Libraries Used
fastapi, uvicorn, pydantic


python-docx, docx2txt, bs4, unstructured


pdf2image, Pillow, pytesseract


atlassian-python-api, jproperties



📂 Output Examples
Each chunk is stored in vector DB with:
chunked_content


chunked_content_clean


ingestion_metadata


vector


usecase_id, doc_type, etc.



📬 Deployment Instructions
pip install -r requirements.txt
uvicorn app.main:app --reload
pytest tests/


For full source code, visit: GitHub Repository (TBD)

