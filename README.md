# SKOS-Notion Converter

A bidirectional converter between SKOS (Simple Knowledge Organization System) RDF vocabularies and common formats for visual editors like Notion. This tool enables knowledge engineers to manage controlled vocabularies and thesauri in a user-friendly interface while maintaining standards-compliant SKOS data.


## Features

- **SKOS → Notion**: Convert SKOS Turtle files to Notion-compatible formats (Markdown, CSV, JSON)
- **Notion → SKOS**: Convert Notion markdown exports back to SKOS Turtle
- **Language Support**: Handle multilingual labels with language preferences
- **Batch Processing**: Process entire directories of files
- **Memory Optimization**: Efficient handling of large vocabularies
- **Validation**: Comprehensive SKOS validation with error and warning detection
- **Hierarchy Preservation**: Maintains broader/narrower relationships and concept schemes
- **Metadata Support**: Preserves definitions, alternative labels, notations, and URIs
- **UUID Generation**: Automatically creates unique identifiers for new concepts
- **Circular Reference Detection**: Identifies and handles circular relationships
- **Multiple Output Formats**: Choose between CSV, Markdown, or JSON for Notion import
- **Pylint Compliant**: High-quality, well-formatted code

## Installation

### Requirements
- Python 3.6 or higher
- rdflib library

### Setup
```bash
# Clone or download the script
wget https://raw.githubusercontent.com/yourusername/skos-notion-converter/main/skos_converter.py

# Install dependencies
pip install rdflib

# Make executable (optional)
chmod +x skos_converter.py
```

## Usage

The converter uses subcommands for different conversion directions:

### Converting SKOS to Notion

#### Basic Usage
```bash
# Basic conversion to CSV (default)
python3 skos_converter.py to-notion vocabulary.ttl

# Convert to Markdown with visual formatting
python3 skos_converter.py to-notion vocabulary.ttl --format markdown

# Convert to all formats
python3 skos_converter.py to-notion vocabulary.ttl --format all
```

#### Language Support
```bash
# Prefer French labels, fallback to English then any language
python3 skos_converter.py to-notion vocabulary.ttl \
  --language fr --fallback-languages en ""

# Prefer German labels with multiple fallbacks
python3 skos_converter.py to-notion vocabulary.ttl \
  --language de --fallback-languages en fr es ""
```

#### Batch Processing
```bash
# Process all .ttl files in a directory
python3 skos_converter.py to-notion dummy.ttl \
  --batch-dir /path/to/input/directory \
  --output-dir /path/to/output/directory \
  --format all

# Batch process with language preferences
python3 skos_converter.py to-notion dummy.ttl \
  --batch-dir ./vocabularies \
  --output-dir ./notion_exports \
  --format markdown \
  --language en
```

#### Advanced Options
```bash
# Skip validation for faster processing
python3 skos_converter.py to-notion vocabulary.ttl --skip-validation

# Force conversion despite validation errors
python3 skos_converter.py to-notion vocabulary.ttl --force

# Custom output name with specific markdown style
python3 skos_converter.py to-notion vocabulary.ttl \
  --output my_vocab \
  --format markdown \
  --markdown-style headings
```

### Converting Notion to SKOS

#### Basic Usage
```bash
# Basic conversion with default namespace
python3 skos_converter.py to-skos notion_export.md

# Specify custom namespace and prefix
python3 skos_converter.py to-skos notion_export.md \
  --namespace "http://data.example.com/vocab#" \
  --prefix "ex"

# Custom output file
python3 skos_converter.py to-skos notion_export.md --output my_vocab.ttl
```

#### Batch Processing
```bash
# Process all .md files in a directory
python3 skos_converter.py to-skos dummy.md \
  --batch-dir /path/to/notion/exports \
  --output-dir /path/to/skos/files \
  --namespace "http://myorg.com/vocab#" \
  --prefix "myorg"
```

### Getting Help

```bash
# General help
python3 skos_converter.py --help

# Help for specific subcommand
python3 skos_converter.py to-notion --help
python3 skos_converter.py to-skos --help
```

## Language Support

The converter now supports multilingual SKOS vocabularies with intelligent label selection:

### Language Preferences
- **Primary language**: Specify preferred language (e.g., `--language en`)
- **Fallback languages**: Define fallback order (e.g., `--fallback-languages en fr de`)
- **No language tags**: Handles labels without language tags
- **Best match selection**: Automatically selects the best available label

### Language Tag Handling
```turtle
# Example multilingual SKOS
ex:concept1 skos:prefLabel "Dog"@en ;
           skos:prefLabel "Chien"@fr ;
           skos:prefLabel "Hund"@de .
```

With `--language fr`, the converter will prefer "Chien" over other labels.

### Language Configuration Examples
```bash
# Prefer English, fallback to any language
python3 skos_converter.py to-notion vocab.ttl --language en

# Prefer French, fallback to English, then German, then any
python3 skos_converter.py to-notion vocab.ttl \
  --language fr --fallback-languages en de ""

# No preference, use first available label
python3 skos_converter.py to-notion vocab.ttl
```

## Batch Processing

Process multiple files efficiently with batch processing:

### Directory Structure
```
input_directory/
├── vocab1.ttl
├── vocab2.ttl
└── vocab3.ttl

output_directory/
├── vocab1.csv
├── vocab1.md
├── vocab1.json
├── vocab2.csv
├── vocab2.md
├── vocab2.json
└── ...
```

### Batch Processing Features
- **Automatic file discovery**: Finds all `.ttl` or `.md` files
- **Progress reporting**: Shows processing status
- **Error handling**: Continues processing if individual files fail
- **Consistent output**: Applies same settings to all files
- **Memory efficient**: Processes files individually to manage memory usage

### Batch Examples
```bash
# Convert all SKOS files to all formats
python3 skos_converter.py to-notion dummy.ttl \
  --batch-dir ./skos_files \
  --output-dir ./notion_ready \
  --format all

# Convert all Notion exports to SKOS
python3 skos_converter.py to-skos dummy.md \
  --batch-dir ./notion_exports \
  --output-dir ./skos_output \
  --namespace "http://example.org/vocab#"
```

## SKOS Validation

The converter performs comprehensive validation with detailed reporting:

### Critical Errors (block conversion by default)
- **Duplicate URIs**: Same URI used for multiple resources
- **Missing labels**: Concepts without prefLabel or rdfs:label
- **Circular references**: Concepts that reference each other in loops
- **Multiple preferred labels**: More than one prefLabel per language on a concept

### Warnings (valid SKOS but worth attention)
- **Duplicate label text**: Same label used by different concepts
- **Orphan concepts**: Concepts without broader relations or top concept status
- **Missing concept schemes**: Concepts not associated with any scheme
- **Polyhierarchy**: Concepts with multiple broader concepts
- **Deep hierarchies**: Hierarchies deeper than 7 levels

### Validation Options
```bash
# Run with full validation (default)
python3 skos_converter.py to-notion vocab.ttl

# Skip validation for speed
python3 skos_converter.py to-notion vocab.ttl --skip-validation

# Continue despite errors
python3 skos_converter.py to-notion vocab.ttl --force
```

## Memory Optimization

The optimized converter includes several performance improvements:

### Caching
- **URI fragment caching**: Reuses computed URI fragments
- **Label caching**: Caches frequently accessed labels
- **Metadata caching**: Optimizes repeated metadata lookups

### Memory Management
- **Streaming processing**: Handles large files without loading everything into memory
- **Efficient data structures**: Uses optimized collections for better performance
- **Garbage collection**: Proper cleanup of temporary objects

### Performance Features
- **Batch size control**: Configurable batch sizes for processing
- **Memory limits**: Configurable memory usage limits
- **Progress reporting**: Shows processing progress for large files

## Code Quality

This version is pylint-compliant with high code quality standards:

### Code Standards
- **PEP 8 compliance**: Follows Python style guidelines
- **Type hints**: Full type annotation for better maintainability
- **Documentation**: Comprehensive docstrings for all functions
- **Error handling**: Robust error handling with detailed messages
- **Logging**: Structured logging instead of print statements

### Architecture Improvements
- **Separation of concerns**: Distinct classes for different responsibilities
- **Configuration management**: Centralized configuration system
- **Modular design**: Easy to extend and modify
- **Clean interfaces**: Well-defined APIs between components

## Command Line Options

### to-notion subcommand:
```
positional arguments:
  input_file            Input Turtle RDF file

optional arguments:
  --format {csv,markdown,json,all}
                        Output format (default: csv)
  --output OUTPUT       Output file name (without extension)
  --skip-validation     Skip SKOS validation checks
  --force              Continue conversion even if validation finds errors
  --markdown-style {headings,bullets,mixed}
                        Markdown formatting style (default: headings)
  --language LANGUAGE   Preferred language for labels (e.g., en, fr, de)
  --fallback-languages [FALLBACK_LANGUAGES ...]
                        Fallback languages in order of preference
  --batch-dir BATCH_DIR Process all .ttl files in directory
  --output-dir OUTPUT_DIR
                        Output directory for batch processing
```

### to-skos subcommand:
```
positional arguments:
  input_file           Input Notion markdown file

optional arguments:
  --output OUTPUT      Output file name (default: input_skos.ttl)
  --namespace NAMESPACE
                       Namespace URI for new concepts 
                       (default: http://example.org/vocabulary#)
  --prefix PREFIX      Namespace prefix (default: ex)
  --batch-dir BATCH_DIR
                       Process all .md files in directory
  --output-dir OUTPUT_DIR
                       Output directory for batch processing
```

## Notion Import/Export Guide

### Importing to Notion

#### From CSV:
1. In Notion, create a new page
2. Type `/table` and select "Table - Full page"
3. Click "..." menu → "Import" → "CSV"
4. Select your generated CSV file
5. Configure the database:
   - Set "Title" as the page title property
   - Use "Parent" property to create linked relations
   - Group by "Concept Scheme" or filter by "Level"
   - Add visual indicators based on "Level" for hierarchy

#### From Markdown:
1. Create a new Notion page
2. Copy the entire markdown content
3. Paste into Notion using Cmd/Ctrl+Shift+V to preserve formatting
4. Optional: Convert to toggle lists:
   - Select hierarchical sections
   - Press Cmd/Ctrl+Shift+7
5. The visual indicators (▸ ▹ ◦) help show hierarchy depth

### Exporting from Notion

1. Open your Notion vocabulary page
2. Click "..." menu → "Export"
3. Choose "Markdown & CSV"
4. Include subpages: "Everything"
5. Extract the markdown file from the zip

## Example Workflows

### Workflow 1: Multilingual Vocabulary Management
```bash
# Convert multilingual SKOS with French preference
python3 skos_converter.py to-notion multilingual_vocab.ttl \
  --language fr --fallback-languages en de \
  --format markdown

# Edit in Notion with French labels
# Export and convert back preserving language
python3 skos_converter.py to-skos edited_vocab.md \
  --namespace "http://example.org/multilingual#"
```

### Workflow 2: Large-Scale Batch Processing
```bash
# Convert entire directory of vocabularies
python3 skos_converter.py to-notion dummy.ttl \
  --batch-dir ./source_vocabularies \
  --output-dir ./notion_imports \
  --format all \
  --language en

# After editing in Notion, convert back
python3 skos_converter.py to-skos dummy.md \
  --batch-dir ./notion_exports \
  --output-dir ./updated_vocabularies \
  --namespace "http://myorg.com/vocab#"
```

### Workflow 3: Quality Assurance Pipeline
```bash
# Validate all vocabularies with strict checking
for file in *.ttl; do
  echo "Validating $file..."
  python3 skos_converter.py to-notion "$file" \
    --format csv \
    --output "validated_$(basename "$file" .ttl)"
done
```

## Performance Guidelines

### For Large Vocabularies
- Use batch processing for multiple files
- Consider `--skip-validation` for initial testing
- Use CSV format for fastest processing
- Process files individually rather than concatenating

### Memory Usage
- Large vocabularies (>10MB) are handled efficiently
- Batch processing uses memory per file, not total
- Progress reporting shows memory-intensive operations

### Speed Optimization
```bash
# Fastest processing (skip validation, CSV only)
python3 skos_converter.py to-notion large_vocab.ttl \
  --format csv --skip-validation

# Balanced (validation, multiple formats)
python3 skos_converter.py to-notion vocab.ttl --format all

# Most thorough (validation, all formats, language processing)
python3 skos_converter.py to-notion vocab.ttl \
  --format all --language en --fallback-languages fr de
```

## Troubleshooting

### Common Issues:

**"Bad syntax" error when parsing Turtle**
- Check for missing periods at end of statements
- Verify all URIs are enclosed in angle brackets
- Ensure quotes are properly closed
- Validate at http://ttl.summerofcode.be/

**"Maximum recursion depth exceeded"**
- Your SKOS file has circular references
- Run validation to identify the cycle
- Fix the circular broader/narrower relationships

**Memory issues with large files**
- Use batch processing instead of single large files
- Consider `--skip-validation` for memory-intensive operations
- Process subsets of large vocabularies

**Language selection not working**
- Verify language tags in source SKOS data
- Check language code format (e.g., 'en', 'fr', not 'English')
- Use `--fallback-languages` to see all available options

**Batch processing fails**
- Ensure input and output directories exist
- Check file permissions
- Verify file extensions (.ttl for SKOS, .md for Notion)

## Limitations

1. **Complex SKOS features**: Some advanced features like collections are not yet supported
2. **Notion limitations**: 
   - Maximum 6 heading levels (deeper hierarchies use indentation)
   - Toggle list conversion must be done manually
3. **Language preservation**: Round-trip conversion may lose some language tag nuances
4. **File size**: Very large vocabularies benefit from batch processing

## Contributing

Contributions are welcome! The code is now pylint-compliant and well-structured for easy modification.

### Development Setup
```bash
# Install development dependencies
pip install rdflib pylint black

# Run pylint
pylint skos_converter.py

# Format code
black skos_converter.py
```

### Areas for Future Enhancement:
- SKOS collections and ordered collections
- Direct Notion API integration
- Additional validation rules
- Performance optimizations for very large vocabularies
- Web interface for conversions

## License

CC0

## Acknowledgments

Built with [RDFLib](https://rdflib.readthedocs.io/) for RDF processing.
Built with assistance from Claude.ai

## Version History

- 1.0.0: Initial release with bidirectional conversion
- 1.1.0: Added validation and circular reference detection
- 1.2.0: Improved hierarchy handling and visual formatting
- 1.3.0: Enhanced error handling and markdown formatting options
- 2.0.0: **Optimized version with language support, batch processing, and pylint compliance**

---

For questions or support, please open an issue on the GitHub repository.