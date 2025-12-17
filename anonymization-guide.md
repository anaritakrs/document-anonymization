# Sentitive Data Anonymization from PDFs - Guide

## What It Does

This process automatically finds and hides personal information in PDF documents while keeping important organizational details visible.

**Simple Process**: PDF → Convert to Images → Find Text → Detect Personal Info → Hide Sensitive Data → Create Protected PDF

## How It Works

### **Setup**

The system needs Azure credentials to work:

```python
# Put these in your k.env file
AZURE_DOCINT_ENDPOINT=your-azure-endpoint
AZURE_DOCINT_KEY=your-azure-key
```

### **What Gets Protected (NOT Hidden)**

The system is smart about what to keep visible:

- **Organizations**: Client's organization name and others related names
- **Roles**: eg. "Solicitante"
- **Legal References**: eg. "Portaria 123", "Portaria GM/MEC 456"
- **Document Dates**: All dates except birth dates
- **Most URLs**: Except specific internal links

### **What Gets Hidden**

- Names and addresses
- Phone numbers and emails  
- ID numbers (CPF, RG, etc.)
- Birth dates
- Bank account numbers
- Specific government URLs

Microsoft model in use: Detect PII - 2025-08-01-preview

## Main Components

### **Data Classes**

Simple containers for information:

```python
@dataclass
class WordBox:
    text: str        # The actual text
    bbox: tuple      # Where it appears on page (x,y coordinates)
    
@dataclass  
class EntityPII:
    category: str    # Type of info (Person, Phone, etc.)
    text: str        # The actual text found
    page: int        # Which page it's on
```

### **Processing Steps**

#### **Step 1: PDF to Images**

```python
def pdf_to_images(pdf_path):
    # Converts each PDF page into a PNG image
    # Returns list of image files
```

#### **Step 2: Read Text (OCR)**

```python  
def ocr_image_docint(image_path):
    # Uses Azure to read all text from image
    # Finds exact position of each word
```

#### **Step 3: Smart Text Processing**

The system automatically chooses the best way to process text:

- **Small documents**: Process all at once
- **Medium documents**: Break into chunks  
- **Large documents**: Use smaller chunks
- **Very large**: Emergency small chunks

#### **Step 4: Find Personal Information**

```python
def detect_pii_with_chunking(text):
    # Uses Azure AI to find personal information
    # Handles documents of any size automatically
```

#### **Step 5: Apply Protection Rules**

```python
def filter_allowed_entities(entities):
    # Decides what to hide vs. what to keep
    # Uses smart rules for Portaria, dates, etc.
```

#### **Step 6: Hide Sensitive Information**

```python
def apply_redactions_simple(image, redactions):
    # Draws black rectangles over sensitive text
    # Keeps protected information visible
```

## Protection Rules

### **Organization Protection**

```python
def is_allowed_organization(text):
    # Keeps client's name and related organizations visible
    # Case-insensitive strategy
```

### **Legal Document Protection**  

```python
def extract_portaria_words(text):
    # Automatically finds "Portaria" + numbers/codes
    # Protects legal references like "Portaria 123"
```

### **Date Protection**

- **Keep Visible**: Document dates, timestamps
- **Hide**: Birth dates only

### **URL Protection**

- **Keep Visible**: Most websites and links
- **Hide**: Internal system URLs only

## How to Use

### **1. Setup Environment**

Create a file called `k.env`:

```python
AZURE_DOCINT_ENDPOINT=https://your-service.cognitiveservices.azure.com/
AZURE_DOCINT_KEY=your-key-here
AZURE_LANGUAGE_ENDPOINT=https://your-service.cognitiveservices.azure.com/
AZURE_LANGUAGE_KEY=your-key-here
```

### **2. Run the System**

```python
# Put your PDF in the 'data' folder
PDF_FILE = 'your-document.pdf'

# Run the anonymization
results = run_simple_anonymization()

# Check if it worked
if results['success']:
    print(f"Done! Output saved to: {results['output_pdf']}")
```

### **3. What You Get**

- **Protected PDF**: Original document with sensitive info hidden
- **Statistics**: Shows what was found and what was protected
- **Logs**: Step-by-step progress information

## Easy Customization

### **Add New Protected Organizations**

```python
ALLOWED_ORGANIZATIONS.append('Your Organization Name')
```

### **Add New Protected Roles**

```python
ALLOWED_PERSONS.append('New Role Title')
```

### **Adjust Processing Speed**

```python
TEXT_SIZE_THRESHOLDS = {
    'single_call_max': 5000,  # Increase for faster processing
    'chunk_size': 3500,       # Decrease for safer processing
}
```

## What the Output Tells You

When processing finishes, you get:

- **Total pages processed**: How many pages were handled
- **Entities found**: How much personal info was detected
- **Entities protected**: How much important info was preserved  
- **Processing method**: Which strategy was used for each page

Example output:
✅ Page 1: 15 total entities found, 3 allowed (not redacted), 12 to redact
📊 TEXT PROCESSING: 8 pages single call, 2 pages chunked
🛡️ PROTECTED: 5 FNDE references, 2 Portaria references, 8 dates

## Summary

This proccessing:

1. **Automatically** finds personal information in PDFs
2. **Intelligently** protects important organizational info  
3. **Safely** hides sensitive personal details
4. **Handles** documents of any size
5. **Provides** detailed results and statistics
