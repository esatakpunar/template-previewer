HTML and PDF Preview PRD
Project Objective
This project is a web application that allows users to input HTML templates and data sources and preview the results in both HTML and PDF formats.

User Interface Requirements
Main Page Layout
The page will be divided into two main sections vertically:

Left Panel

Right Panel

Left Panel Features
HTML Template Input Field

Large text editor

Syntax highlighting support

Highlighting of template variables

Data Source Input Field

Data entry in JSON format

Syntax highlighting support

JSON validation

Control Buttons

"Run" button

Sends the template and data source to the backend

Right Panel Features
Tabbed View

HTML Preview tab

PDF Preview tab

HTML Preview

Combined result of the template and data source

Responsive view

PDF Preview

Preview of the PDF document

Zoom in/out features

Technical Requirements
Frontend Framework: Vue.js

UI Library: Tailwind CSS

Code Editor: Monaco Editor

PDF Viewer: PDF.js

Responsive design

Modern and user-friendly interface

Dark/Light mode support

Error notification system

Detailed error handling and display

Backend Integration
API Endpoint: /report/pdfTest

Request Format:

json
Copy
Edit
{
  "template": "HTML template string",
  "dataSource": "JSON data string"
}
Response Format:

json
Copy
Edit
{
  "htmlContent": "Rendered HTML string",
  "pdfContent": "PDF byte array"
}
User Experience
Loading indicators for loading states

Toast notifications for error messages

User-friendly error management

Responsive design for mobile compatibility

Dark/Light mode toggle

Detailed error messages and notifications

