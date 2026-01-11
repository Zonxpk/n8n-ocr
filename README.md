# Project 2: OCR → JSON

Automated document processing workflow that extracts data from PDF/image files (receipts, invoices, tax documents) and converts them to validated JSON format using OCR and AI.

## Features

- **Multi-format Support**: PDF, PNG, JPG, and other common image formats
- **OCR Processing**: Uses OpenAI Vision API for accurate text extraction
- **AI-powered Parsing**: GPT-4 extracts structured data from raw OCR text
- **Validation Engine**: Verifies calculations (subtotal + VAT = grand total)
- **Database Integration**: Automatically stores extracted data
- **Error Handling**: Detailed validation messages for failed documents

## Architecture

```
Document Upload → Webhook → OCR Extraction → AI Parser 
→ JSON Conversion → Validation → Database Save → Response
```

## Prerequisites

1. **n8n Instance**: Self-hosted or cloud (n8n.cloud)
2. **OpenAI API Key**: For Vision/GPT-4 access
3. **Database**: PostgreSQL, MongoDB, or cloud storage (Firebase, Supabase)
4. **File Storage**: Local filesystem or cloud (S3, Google Cloud Storage)

## Setup Instructions

### 1. Environment Variables

Create `.env` file:

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxx
DATABASE_URL=postgresql://user:password@localhost/docdb
DATABASE_TYPE=postgres
S3_BUCKET=your-bucket-name
S3_ACCESS_KEY=xxxxxxxxxxxxx
S3_SECRET_KEY=xxxxxxxxxxxxx
```

### 2. Database Schema

Create table for storing extracted documents:

```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY,
  document_type VARCHAR(50),
  merchant VARCHAR(255),
  invoice_number VARCHAR(100),
  date DATE,
  items JSONB,
  subtotal DECIMAL(10,2),
  tax_rate DECIMAL(5,4),
  tax_amount DECIMAL(10,2),
  grand_total DECIMAL(10,2),
  payment_method VARCHAR(50),
  validation_status VARCHAR(20),
  raw_ocr_text TEXT,
  uploaded_at TIMESTAMP,
  processed_at TIMESTAMP
);

CREATE INDEX idx_invoice_number ON documents(invoice_number);
CREATE INDEX idx_merchant ON documents(merchant);
CREATE INDEX idx_date ON documents(date);
```

### 3. Deploy Workflow

1. Export workflow to n8n
2. Configure webhook URL: `https://your-n8n-instance/webhook/upload`
3. Set up database connection in n8n
4. Activate workflow

### 4. Upload Test Document

```bash
curl -X POST https://your-n8n-instance/webhook/upload \
  -F "file=@receipt.jpg" \
  -F "document_type=receipt"
```

## Example Input/Output

### Input
- File: `receipt.jpg` (photo of supermarket receipt)

### Output JSON
```json
{
  "document_type": "receipt",
  "merchant": "Tesco Supermarket",
  "invoice_number": "TXN-2026-001234",
  "date": "2026-01-11",
  "items": [
    {
      "name": "Milk 2L",
      "quantity": 1,
      "unit_price": 2.50,
      "total": 2.50
    },
    {
      "name": "Bread",
      "quantity": 2,
      "unit_price": 1.20,
      "total": 2.40
    }
  ],
  "subtotal": 4.90,
  "tax_rate": 0.20,
  "tax_amount": 0.98,
  "grand_total": 5.88,
  "payment_method": "CARD",
  "validation_status": "passed"
}
```

## Validation Rules

✅ **Passed If:**
- Subtotal + Tax Amount = Grand Total (±0.01 tolerance for rounding)
- All required fields populated
- Date is in valid format
- All monetary values are positive

❌ **Failed If:**
- Math doesn't add up
- Missing critical fields (total, date, merchant)
- Date format invalid
- Negative or zero values

## Sample Outputs

See `sample-output.json` for example extracted documents.

## Supported Document Types

- 🧾 Receipt
- 📄 Invoice
- 🏦 Bank Statement
- 📋 Tax Document (ใบกำกับภาษี)
- 💳 Credit Card Statement

## Troubleshooting

| Issue | Solution |
|-------|----------|
| OCR text quality poor | Ensure image is well-lit and high-resolution (>300 DPI) |
| JSON parsing fails | Check if AI response matches expected JSON format |
| Validation fails | Review raw OCR text; document may be damaged or unclear |
| Database connection errors | Verify DATABASE_URL and network connectivity |
| API quota exceeded | Check OpenAI usage; consider batch processing during off-hours |

## Advanced Features (Optional)

- **Batch Processing**: Process multiple documents in parallel
- **Custom Templates**: Define document-specific extraction patterns
- **Expense Categorization**: Auto-categorize items (food, transport, etc.)
- **Receipt Deduplication**: Detect and prevent duplicate entries
- **Integration**: Connect to accounting software (QuickBooks, Xero)

## Files

- `workflow.json` - n8n workflow export
- `README.md` - This file
- `sample-output.json` - Example extracted JSON outputs

## References

- OpenAI Vision API: https://platform.openai.com/docs/guides/vision
- n8n Docs: https://docs.n8n.io
- GPT-4 Model Card: https://openai.com/research/gpt-4
