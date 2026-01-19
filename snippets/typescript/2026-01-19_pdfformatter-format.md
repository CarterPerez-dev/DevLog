# PdfFormatter.format

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/lib/export/formatters/pdfFormatter.ts
**Language:** typescript
**Lines:** 20-126
**Complexity:** 19.0

---

## Source Code

```typescript
format(data: T[], config: ExportConfig<T>, options: ExportOptions): void {
    try {
      if (data === null || data === undefined) {
        throw new Error('Data is required for export');
      }

      if (!Array.isArray(data)) {
        throw new Error('Data must be an array');
      }

      const orientation = config.pdfOrientation ?? 'landscape';
      const pageSize = config.pdfPageSize ?? 'a4';
      const pdf = new jsPDF(orientation, 'mm', pageSize);

      const primaryColor: [number, number, number] = [99, 102, 241]; // $primary-500
      const textColor: [number, number, number] = [31, 41, 55]; // $gray-800
      const headerBg: [number, number, number] = [243, 244, 246]; // $gray-100

      const title = options.title ?? 'Data Export';
      pdf.setFontSize(18);
      pdf.setTextColor(...primaryColor);
      pdf.text(title, 14, 20);

      let yPosition = 25;
      if (
        options.description !== null &&
        options.description !== undefined &&
        options.description.length > 0
      ) {
        pdf.setFontSize(10);
        pdf.setTextColor(...textColor);
        pdf.text(options.description, 14, yPosition);
        yPosition += 5;
      }

      if (options.includeMetadata === true) {
        pdf.setFontSize(9);
        pdf.setTextColor(100, 100, 100);
        pdf.text(
          `Exported: ${format(new Date(), 'MMM dd, yyyy HH:mm')}`,
          14,
          yPosition,
        );
        pdf.text(`Records: ${String(data.length)}`, 14, yPosition + 4);
        yPosition += 10;
      }

      const tableData = this.prepareTableData(data, config);

      const headers = config.columns.map((col) => col.label);

      const columnWidths = this.calculateColumnWidths(config, orientation);

      autoTable(pdf, {
        head: [headers],
        body: tableData,
        startY: yPosition,
        theme: 'grid',
        headStyles: {
          fillColor: headerBg,
          textColor,
          fontSize: 10,
          fontStyle: 'bold',
        },
        bodyStyles: {
          fontSize: 9,
          cellPadding: 3,
        },
        alternateRowStyles: {
          fillColor: [249, 250, 251], // $gray-50
        },
        columnStyles: this.getColumnStyles(config, columnWidths),
        margin: { top: yPosition, left: 14, right: 14 },
        didDrawPage: (_hookData): void => {
          const pageCount = pdf.getNumberOfPages();
          pdf.setFontSize(8);
          pdf.setTextColor(150, 150, 150);

          for (let i = 1; i <= pageCount; i++) {
            pdf.setPage(i);
            const pageText = `Page ${String(i)} of ${String(pageCount)}`;
            const pageWidth = pdf.internal.pageSize.getWidth();
            pdf.text(
              pageText,
              pageWidth - 30,
              pdf.internal.pageSize.getHeight() - 10,
            );
          }
        },
      });

      const timestamp = format(new Date(), 'yyyy-MM-dd-HHmmss');
      const filename =
        options.filename ?? config.defaultFilename ?? `export-${timestamp}`;
      const finalFilename = filename.endsWith('.pdf') ? filename : `${filename}.pdf`;

      pdf.save(finalFilename);
    } catch (error: unknown) {
      let errorMessage = 'Failed to export data as PDF';

      if (error instanceof Error) {
        errorMessage = error.message;
      }

      throw new Error(errorMessage);
    }
  }
```

---

## Documentation

### Documentation for `PdfFormatter.format`

#### What It Does
The `format` function in the `PdfFormatter` class generates a PDF document from an array of data, using provided configuration and options. The function ensures that the input data is valid before processing, sets up the PDF document with appropriate styling, adds a title and optional description, and formats the data into a table.

#### Why It Matters
This method is essential for exporting tabular data in a structured format, making it easier to share or archive information. It handles various configurations like orientation, page size, and metadata inclusion, ensuring flexibility and customization.

#### When/Why to Use
Use this function when you need to export data from an application into a PDF document. The method is particularly useful for administrative tools where detailed reports are required. You can customize the appearance of the PDF by adjusting the `config` and `options`.

#### Patterns and Gotchas
- **Error Handling**: The function includes robust error handling, ensuring that any issues during export are caught and reported clearly.
- **Dynamic Content**: The inclusion of dynamic content like timestamps in filenames ensures that each exported file is unique and easily identifiable.
- **Customization**: While the default settings work well, users can customize almost every aspect of the PDF through the `config` and `options` parameters. However, improper configuration might lead to unexpected results or errors.

This function provides a comprehensive solution for generating PDF exports with minimal effort, making it a valuable tool in administrative applications.

---

*Generated by CodeWorm on 2026-01-19 00:42*
