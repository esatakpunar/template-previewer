# HTML and PDF Preview PRD

## Project Objective
This project is a web application that allows users to input HTML templates and data sources and preview the results in both HTML and PDF formats.

## User Interface Requirements

### Main Page Layout
The page will be divided into two main sections vertically:
- **Left Panel**
- **Right Panel**

### Left Panel Features
1. **HTML Template Input Field**
   - Large text editor
   - Syntax highlighting support
   - Highlighting of template variables

2. **Data Source Input Field**
   - Data entry in JSON format
   - Syntax highlighting support
   - JSON validation

3. **Control Buttons**
   - "Run" button
   - Sends the template and data source to the backend

### Right Panel Features
1. **Tabbed View**
   - HTML Preview tab
   - PDF Preview tab

2. **HTML Preview**
   - Combined result of the template and data source
   - Responsive view

3. **PDF Preview**
   - Preview of the PDF document
   - Zoom in/out features

## Technical Requirements
- **Frontend Framework**: Vue.js
- **UI Library**: Tailwind CSS
- **Code Editor**: Monaco Editor
- **PDF Viewer**: PDF.js
- Responsive design
- Modern and user-friendly interface
- Dark/Light mode support
- Error notification system
- Detailed error handling and display

## Backend Integration
- **API Endpoint**: `/report/pdfTest`
  
### Request Format:
```json
{
  "template": "HTML template string",
  "dataSource": "JSON data string"
}
