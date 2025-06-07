# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rmapy is an unofficial Python client for the Remarkable Cloud API. This is a fork of an archived project, maintained for personal use with a focus on programmatically extracting files from the Remarkable cloud. The original project provided full CRUD operations, but this fork emphasizes reliable file retrieval and document extraction capabilities.

## Development Commands

### Package Installation
```bash
# Install dependencies using uv (preferred for this project)
uv sync

# Install in development mode with all dependencies
uv sync --all-extras

# Install specific optional dependencies
uv sync --extra doc
uv sync --extra dev

# Legacy pip installation (if uv not available)
pip install -e .
pip install -e ".[doc]"
```

### Documentation
```bash
# Build documentation with Sphinx
cd docs/
make html

# Clean documentation build
make clean
```

### Package Building
```bash
# Build distribution packages with uv
uv build

# Legacy building (if uv not available)
python setup.py sdist bdist_wheel
```

### Running Code
```bash
# Run Python scripts in the uv environment
uv run python your_script.py

# Or activate the virtual environment
uv shell
```

## Code Architecture

### Core Components

- **`api.py`**: Central `Client` class that handles authentication and API communication with Remarkable Cloud
- **`document.py`**: Contains `Document`, `ZipDocument`, `RmPage`, and `Highlight` classes for handling document data
- **`folder.py`**: `Folder` class for managing collections/directories
- **`collections.py`**: `Collection` class that manages groups of documents and folders
- **`config.py`**: Configuration management for storing authentication tokens in `~/.rmapi`
- **`meta.py`**: Base metadata class that Document and Folder inherit from

### Authentication Flow

1. **Device Registration**: Use `register_device(code)` with a one-time code from https://my.remarkable.com/device/desktop/connect
2. **Token Renewal**: Call `renew_token()` before each session to get a fresh user token
3. **Configuration**: Tokens are automatically stored in `~/.rmapi` config file

### Key Data Structures for File Extraction

- **ZipDocument**: Critical for file extraction - handles the complex zip file format used by Remarkable Cloud containing `.content`, `.pagedata`, `.rm` files, thumbnails, and highlights
- **Collection**: Provides utilities for navigating the folder/document hierarchy to locate files for extraction
- **Meta Objects**: Both Document and Folder inherit from Meta and share common attributes like ID, Version, Parent, VissibleName

### File Extraction Workflow

1. **Authentication**: `register_device()` → `renew_token()` → `is_auth()`
2. **Discovery**: `get_meta_items()` returns a Collection of all documents/folders
3. **Navigation**: Use Collection methods like `children()` to browse hierarchy
4. **Retrieval**: `get_doc(id)` gets metadata, then `download(document)` retrieves the ZipDocument
5. **Extraction**: ZipDocument provides access to all file components (PDFs, annotations, highlights, etc.)

### Dependencies

- `requests`: HTTP client for API communication
- `pyaml==19.4.1`: Configuration file handling
- `urllib3>=1.26.5`: HTTP library (security requirement)
- `sphinx`: Documentation generation (dev dependency)

### Package Management

This project uses **uv** for fast, reliable Python package management. The configuration is defined in `pyproject.toml` with both production and development dependencies. uv provides faster dependency resolution and installation compared to traditional pip/pipenv workflows.

### File Formats

The project handles Remarkable's proprietary zip file format which contains:
- `.content`: JSON metadata about the document
- `.pagedata`: Page-specific data
- `.rm`: Binary drawing data for each page
- `.thumbnails/`: JPEG thumbnails
- `.highlights/`: JSON highlight data for EPUB documents
- `.pdf`/`.epub`: Original document files if applicable

## Development Notes

- Python 3.6+ required
- No test suite currently exists in the repository
- Package uses setuptools for distribution
- Documentation uses Sphinx with autodoc and type hints
- Project follows semantic versioning (currently v0.3.1)
- **Fork Status**: This is a personal fork of an archived project, maintained specifically for file extraction use cases
- **Primary Use Case**: Programmatic extraction of documents, annotations, and metadata from Remarkable Cloud