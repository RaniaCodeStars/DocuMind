# DocuMind — Software Requirements Specification

**Version:** 0.1  
**Status:** Initial scope validated  
**Project type:** AI-powered web application  
**Primary language:** French  
**Deployment target:** Local development and demonstration

## 1. Project Overview

DocuMind is an AI-powered web application that enables users to ask questions about PDF documents and receive answers grounded in the documents' content.

The application uses Retrieval-Augmented Generation (RAG) to retrieve relevant passages from a PDF before generating an answer with a Large Language Model (LLM).

Every answer should include references to the relevant source pages. If the available document content does not provide sufficient evidence, the application must communicate this limitation instead of inventing an answer.

The initial version is designed as a local prototype for learning, demonstration, and portfolio purposes.

## 2. Target Users

### 2.1 Primary Users

- Students who want to understand course materials, lecture notes, and academic documents.
- Users who need to retrieve specific information from long PDF documents.

### 2.2 Secondary Users

- Professionals who want to query non-confidential PDF documents.
- Developers and AI enthusiasts interested in understanding a practical RAG implementation.

### 2.3 User Access

The MVP will operate without user accounts or authentication.

Users will access the application locally. The system will not provide individual user profiles, role management, or user-specific document permissions.

Because authentication is not implemented, the prototype must not be exposed as a public multi-user service without additional security measures.

## 3. Project Scope

### 3.1 Features Included in the MVP

The initial version will support:

1. Uploading PDF documents.
2. Validating uploaded files.
3. Extracting text while preserving page numbers.
4. Splitting extracted text into manageable chunks.
5. Generating embeddings and indexing chunks in a local vector database.
6. Asking questions about a selected document in French.
7. Retrieving relevant passages from that document.
8. Generating answers grounded in retrieved passages.
9. Displaying source references with page numbers and relevant excerpts.
10. Indicating when the document does not contain sufficient information.
11. Listing uploaded documents.
12. Deleting a document and its associated indexed data.
13. Displaying clear error messages when processing fails.

### 3.2 Out of Scope

The following features are excluded from the MVP:

- User registration, login, and authentication.
- Multi-user access and document-sharing permissions.
- Cloud file storage.
- OCR for scanned or image-only PDFs.
- Support for formats other than PDF.
- Voice interaction.
- English and Arabic interfaces or multilingual answer generation.
- Queries across multiple documents simultaneously.
- Payment and subscription management.
- Enterprise administration and analytics.

These features may be considered in future versions.

## 4. Functional Requirements

### FR-01 — PDF Upload

The system shall allow users to upload PDF files through the web interface.

- The maximum file size shall initially be 10 MB.
- The system shall validate the file extension, media type, and PDF readability.
- Each document shall receive a unique identifier.
- Invalid, empty, corrupted, or unsupported files shall be rejected with an understandable error message.
- Uploaded files shall be stored in a designated local storage directory.

### FR-02 — Text Extraction

The system shall extract text from uploaded PDFs using PyMuPDF.

- Extracted text shall retain its associated page number.
- The system shall detect documents that contain no extractable text.
- Failed extraction shall not produce a document marked as successfully indexed.
- The initial version shall support text-based PDFs only.

### FR-03 — Chunking and Indexing

The system shall divide extracted text into smaller chunks suitable for semantic retrieval.

- Each chunk shall retain its document identifier and page metadata.
- The system shall generate embeddings for the chunks.
- Embeddings and associated metadata shall be stored in a local vector database.
- The system shall indicate whether document processing succeeded or failed.
- The application shall avoid treating a partially indexed document as fully ready for querying.

### FR-04 — Question Submission and Retrieval

The system shall allow users to submit questions in French about a selected document.

- Retrieval shall be restricted to the selected document.
- The system shall retrieve passages relevant to the submitted question.
- Retrieved passages shall retain their source page information.
- If no sufficiently relevant passage is found, the system shall communicate that it cannot answer reliably from the document.

### FR-05 — Answer Generation

The system shall use an LLM to generate answers based on retrieved passages.

- Answers shall be generated in French.
- The model shall be instructed to prioritize the retrieved document content.
- The system shall not present unsupported claims as established facts from the document.
- When the evidence is insufficient, the answer shall clearly state this limitation.
- Instructions found inside a PDF shall be treated as document content, not as trusted system instructions.

### FR-06 — Source References

The system shall provide source references for answers based on document content.

Each reference shall include, where available:

- The source document.
- The relevant page number.
- A short excerpt supporting the answer.

References shall be derived from the actual retrieved passages. The application shall not fabricate page numbers or source excerpts.

### FR-07 — Document Management

The system shall allow users to view and delete uploaded documents.

- The document list shall display basic metadata, such as filename and processing status.
- Deleting a document shall also remove its associated chunks and vector index entries.
- The application shall handle deletion failures without incorrectly reporting successful removal.
- The local storage structure shall prevent accidental collisions between uploaded filenames.

### FR-08 — Local Data Persistence

The system shall store uploaded PDFs and vector-index data locally.

- PDF files shall be stored in a designated local directory.
- The vector database shall use persistent local storage.
- Restarting the application shall not automatically delete successfully stored documents or their indexes.
- Temporary files created during processing shall be cleaned up when no longer needed.
- The system shall not upload PDF files to a cloud file-storage service in the MVP.

The LLM or embedding provider may still receive text when an external API is used. This must be documented clearly; local file storage does not mean that all processing remains local.

## 5. Non-Functional Requirements

### 5.1 Performance

- The application shall provide clear feedback during upload, extraction, indexing, and answer generation.
- Response time shall be measured using a small test collection of PDFs and questions.
- Initial performance targets shall be established after the first working implementation.

### 5.2 Security and Privacy

- Uploaded files shall be validated before processing.
- API keys and secrets shall be stored in environment variables and excluded from version control.
- File paths shall be controlled by the application rather than accepted directly from user input.
- The application shall not be presented as suitable for confidential documents without an appropriate security and privacy assessment.
- The local prototype shall not be exposed to untrusted network users without authentication and additional protections.
- The use of external AI APIs shall be documented, including the possibility that document text may be transmitted to a provider.

### 5.3 Reliability

- Errors during extraction, embedding, indexing, or generation shall be handled explicitly.
- Failed processing shall not leave documents falsely marked as ready.
- Deletion operations shall account for both the original file and its indexed content.

### 5.4 Maintainability

- The backend shall use a modular structure with clearly separated responsibilities.
- API routes, document processing, retrieval, and answer generation shall be implemented in separate modules or services.
- The initial architecture shall be a modular monolith rather than a microservices system.

### 5.5 Testability

- Core processing and retrieval logic shall be testable independently.
- The project shall include automated tests for important success and failure scenarios.
- RAG quality shall be evaluated using a small collection of test questions and expected source passages.

### 5.6 Usability

- The interface shall be simple and understandable.
- All user-facing interface text and generated answers shall be in French.
- Loading states, processing status, and error messages shall be visible.
- Source references shall be easy to identify and inspect.

## 6. Main Workflow

### 6.1 Document Ingestion

1. The user selects a PDF file.
2. The frontend sends the file to the backend.
3. The backend validates the file.
4. The backend saves the PDF locally and assigns a document identifier.
5. PyMuPDF extracts the text and page information.
6. The text is split into chunks.
7. The embedding model generates vector representations.
8. The chunks, embeddings, and metadata are stored in the local vector database.
9. The document is marked as ready if processing succeeds.

### 6.2 Question Answering

1. The user selects an indexed document.
2. The user submits a question in French.
3. The backend generates an embedding for the question.
4. The retrieval component searches for relevant chunks within the selected document.
5. The application checks whether the retrieved evidence is sufficient.
6. The LLM generates an answer in French using the retrieved context.
7. The backend returns the answer and its source references.
8. The frontend displays the answer and the corresponding page numbers and excerpts.

If the retrieved evidence is insufficient, the application shall communicate this instead of generating an unsupported answer.

## 7. Initial Technical Constraints

### 7.1 Frontend

- React
- TypeScript
- A responsive web interface

### 7.2 Backend

- Python
- FastAPI
- PyMuPDF for PDF text extraction

### 7.3 AI and Retrieval

- An embedding model suitable for French-language semantic retrieval.
- ChromaDB as the initial vector database.
- An LLM API for answer generation, unless a local model is selected during implementation.
- A retrieval pipeline that preserves document and page metadata.

### 7.4 Local Storage

- A dedicated local directory for uploaded PDF files.
- Persistent local storage for ChromaDB.
- Environment variables for configuration and API secrets.

The exact directory structure and storage paths shall be defined in the architecture document.

### 7.5 Development and Testing

- Git and GitHub for version control.
- pytest for backend tests.
- Docker for reproducible execution if included in the initial delivery.
- A README explaining installation, configuration, execution, and known limitations.

### 7.6 Architecture

The backend shall follow a modular monolith architecture. The initial implementation shall prioritize clear separation of responsibilities over distributed services.

## 8. Success Criteria

The MVP shall be considered successful when:

1. A valid text-based PDF can be uploaded and stored locally.
2. Text and page numbers can be extracted correctly.
3. The document can be chunked, embedded, and indexed.
4. A user can ask a question in French about the selected document.
5. The system retrieves relevant passages from that document.
6. Answers include references to real source pages and excerpts.
7. The system communicates when the document does not provide sufficient evidence.
8. Deleting a document removes both the original file and its indexed data.
9. Stored documents remain available after an application restart.
10. Automated tests cover the main processing and error-handling scenarios.
11. A small evaluation dataset is used to assess retrieval relevance and answer grounding.
12. Another developer can run the project by following the README.

## 9. Assumptions and Limitations

- The initial target is a local prototype for demonstration and portfolio purposes.
- No user accounts or authentication are required.
- The application is intended for one local user at a time.
- Only text-based PDFs are supported in the MVP.
- The supported user interaction and answer language is French.
- Uploaded files and the vector index are stored locally.
- The use of external AI APIs may involve transmitting extracted document text to a third-party provider.
- LLM-generated answers may contain errors even when source references are displayed.
- Processing time and operating costs depend on the selected models and external services.
- The prototype is not intended for confidential documents or unrestricted public deployment.
- Production deployment would require a separate assessment of authentication, access control, data protection, operational security, and scalability.

## 10. Future Improvements

Potential future enhancements include:

- User accounts and authentication.
- Cloud storage and deployment.
- Multi-user access control.
- English and Arabic support.
- OCR for scanned PDFs.
- Questions across multiple documents.
- Conversation history.
- Streaming answer generation.
- Hybrid retrieval and reranking.
- Document summarization.
- RAG evaluation dashboards.
- Monitoring, rate limiting, and production security improvements.

These improvements are not part of the current MVP.

## 11. Next Project Artifacts

The following documents shall be prepared to support the implementation:

- `UML/Use-Case-Diagram.md`
- `UML/Sequence-Diagram.md`
- `UML/Class-Diagram.md`
- `UML/Activity-Diagram.md`
- `Architecture.md`
- `RAG-Design.md`
- `API-Specification.md`
- `Database-Design.md`
- `Test-Plan.md`

Each artifact shall remain consistent with the scope and constraints defined in this specification.
